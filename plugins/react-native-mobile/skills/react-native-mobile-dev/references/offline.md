# Offline-First Architecture

> Patterns for building reliable mobile apps that work without network connectivity. Install required packages: `@react-native-community/netinfo`, `react-native-mmkv`, `uuid`.

## Network Monitoring

### NetInfo Setup

```tsx
// src/core/services/network/monitor.ts
import NetInfo, { type NetInfoState } from '@react-native-community/netinfo';
import { useNetworkStore } from '@/store';

export type NetworkStatus = 'online' | 'offline' | 'unknown';

class NetworkMonitor {
  private unsubscribe: (() => void) | null = null;

  start(): void {
    if (this.unsubscribe) return;

    this.unsubscribe = NetInfo.addEventListener((state) => {
      this.handleNetworkChange(state);
    });

    // Initial check
    NetInfo.fetch().then(this.handleNetworkChange);
  }

  stop(): void {
    this.unsubscribe?.();
    this.unsubscribe = null;
  }

  private handleNetworkChange = (state: NetInfoState): void => {
    const status: NetworkStatus = state.isConnected === null
      ? 'unknown'
      : state.isConnected
        ? 'online'
        : 'offline';

    useNetworkStore.getState().setStatus(status, {
      type: state.type,
      isInternetReachable: state.isInternetReachable ?? false,
    });

    // Trigger sync when coming back online
    if (status === 'online') {
      useNetworkStore.getState().triggerSync();
    }
  };

  async checkConnection(): Promise<boolean> {
    const state = await NetInfo.fetch();
    return state.isConnected === true;
  }
}

export const networkMonitor = new NetworkMonitor();
```

### Network Store

```tsx
// src/store/network-store.ts
import { create } from 'zustand';
import { createSelectors } from '@/shared/utils/zustand';
import type { NetworkStatus } from '@/core/services/network';

interface NetworkDetails {
  type: string;
  isInternetReachable: boolean;
}

interface NetworkState {
  status: NetworkStatus;
  details: NetworkDetails | null;
  lastOnline: number | null;
  setStatus: (status: NetworkStatus, details?: NetworkDetails) => void;
  triggerSync: () => void;
}

const _useNetworkStore = create<NetworkState>((set, get) => ({
  status: 'unknown',
  details: null,
  lastOnline: null,

  setStatus: (status, details) => {
    set({
      status,
      details: details ?? null,
      lastOnline: status === 'online' ? Date.now() : get().lastOnline,
    });
  },

  triggerSync: () => {
    // Will be called when network comes back online
    // Sync queue will listen to this
  },
}));

export const useNetworkStore = createSelectors(_useNetworkStore);
```

### Network Hook

```tsx
// src/shared/hooks/use-network-status.ts
import { useNetworkStore } from '@/store';

export function useNetworkStatus() {
  const status = useNetworkStore.use.status();
  const details = useNetworkStore.use.details();

  return {
    isOnline: status === 'online',
    isOffline: status === 'offline',
    status,
    connectionType: details?.type,
    isInternetReachable: details?.isInternetReachable ?? false,
  };
}
```

## Offline Request Queue

### Queue Types

```tsx
// src/core/services/network/types.ts
export interface QueuedRequest {
  id: string;
  method: 'POST' | 'PUT' | 'PATCH' | 'DELETE';
  url: string;
  data: unknown;
  headers?: Record<string, string>;
  timestamp: number;
  retries: number;
  maxRetries: number;
  priority: 'high' | 'normal' | 'low';
  metadata?: {
    entityType?: string;
    entityId?: string;
    action?: string;
  };
}

export interface QueueConfig {
  maxRetries: number;
  retryDelay: number;
  maxQueueSize: number;
}
```

### Request Queue

```tsx
// src/core/services/network/queue.ts
import { v4 as uuid } from 'uuid';
import { storage } from '@/core/services/storage';
import { client } from '@/core/services/api';
import { useSyncStore } from '@/store';
import type { QueuedRequest, QueueConfig } from './types';

const QUEUE_KEY = 'offline-queue';

const DEFAULT_CONFIG: QueueConfig = {
  maxRetries: 3,
  retryDelay: 1000,
  maxQueueSize: 100,
};

class OfflineQueue {
  private config: QueueConfig;
  private isProcessing = false;

  constructor(config: Partial<QueueConfig> = {}) {
    this.config = { ...DEFAULT_CONFIG, ...config };
  }

  // Add request to queue
  async enqueue(
    request: Omit<QueuedRequest, 'id' | 'timestamp' | 'retries'>
  ): Promise<string> {
    const queue = this.getQueue();

    if (queue.length >= this.config.maxQueueSize) {
      throw new Error('Offline queue is full');
    }

    const queuedRequest: QueuedRequest = {
      ...request,
      id: uuid(),
      timestamp: Date.now(),
      retries: 0,
      maxRetries: request.maxRetries ?? this.config.maxRetries,
    };

    queue.push(queuedRequest);
    this.saveQueue(queue);

    useSyncStore.getState().updatePendingCount(queue.length);

    return queuedRequest.id;
  }

  // Remove request from queue
  dequeue(id: string): void {
    const queue = this.getQueue().filter((req) => req.id !== id);
    this.saveQueue(queue);
    useSyncStore.getState().updatePendingCount(queue.length);
  }

  // Process all queued requests
  async processQueue(): Promise<void> {
    if (this.isProcessing) return;

    this.isProcessing = true;
    useSyncStore.getState().setSyncing(true);

    const queue = this.getQueue();
    const results: Array<{ id: string; success: boolean; error?: string }> = [];

    // Sort by priority and timestamp
    const sorted = this.sortByPriority(queue);

    for (const request of sorted) {
      try {
        await this.processRequest(request);
        this.dequeue(request.id);
        results.push({ id: request.id, success: true });
      } catch (error) {
        const shouldRetry = this.handleFailure(request, error);
        results.push({
          id: request.id,
          success: false,
          error: error instanceof Error ? error.message : 'Unknown error',
        });

        if (!shouldRetry) {
          this.dequeue(request.id);
        }
      }
    }

    this.isProcessing = false;
    useSyncStore.getState().setSyncing(false);
    useSyncStore.getState().setLastSyncResult(results);
  }

  private async processRequest(request: QueuedRequest): Promise<void> {
    const { method, url, data, headers } = request;

    await client.request({
      method,
      url,
      data,
      headers,
    });
  }

  private handleFailure(request: QueuedRequest, error: unknown): boolean {
    const queue = this.getQueue();
    const index = queue.findIndex((req) => req.id === request.id);

    if (index === -1) return false;

    queue[index].retries += 1;

    if (queue[index].retries >= queue[index].maxRetries) {
      // Max retries reached, will be removed
      return false;
    }

    this.saveQueue(queue);
    return true;
  }

  private sortByPriority(queue: QueuedRequest[]): QueuedRequest[] {
    const priorityOrder = { high: 0, normal: 1, low: 2 };

    return [...queue].sort((a, b) => {
      const priorityDiff = priorityOrder[a.priority] - priorityOrder[b.priority];
      if (priorityDiff !== 0) return priorityDiff;
      return a.timestamp - b.timestamp;
    });
  }

  private getQueue(): QueuedRequest[] {
    const data = storage.getString(QUEUE_KEY);
    return data ? JSON.parse(data) : [];
  }

  private saveQueue(queue: QueuedRequest[]): void {
    storage.set(QUEUE_KEY, JSON.stringify(queue));
  }

  // Get current queue status
  getStatus() {
    const queue = this.getQueue();
    return {
      pending: queue.length,
      items: queue,
    };
  }

  // Clear all queued requests
  clear(): void {
    this.saveQueue([]);
    useSyncStore.getState().updatePendingCount(0);
  }
}

export const offlineQueue = new OfflineQueue();
```

### Sync Store

```tsx
// src/store/sync-store.ts
import { create } from 'zustand';
import { createSelectors } from '@/shared/utils/zustand';

interface SyncResult {
  id: string;
  success: boolean;
  error?: string;
}

interface SyncState {
  isSyncing: boolean;
  pendingCount: number;
  lastSyncTime: number | null;
  lastSyncResult: SyncResult[] | null;
  setSyncing: (syncing: boolean) => void;
  updatePendingCount: (count: number) => void;
  setLastSyncResult: (results: SyncResult[]) => void;
}

const _useSyncStore = create<SyncState>((set) => ({
  isSyncing: false,
  pendingCount: 0,
  lastSyncTime: null,
  lastSyncResult: null,

  setSyncing: (isSyncing) => set({ isSyncing }),

  updatePendingCount: (pendingCount) => set({ pendingCount }),

  setLastSyncResult: (results) => set({
    lastSyncResult: results,
    lastSyncTime: Date.now(),
  }),
}));

export const useSyncStore = createSelectors(_useSyncStore);
```

## Sync Mechanism

### Auto-Sync on Reconnect

```tsx
// src/core/services/network/sync.ts
import { offlineQueue } from './queue';
import { useNetworkStore, useSyncStore } from '@/store';

class SyncManager {
  private unsubscribe: (() => void) | null = null;

  start(): void {
    // Subscribe to network status changes
    this.unsubscribe = useNetworkStore.subscribe(
      (state) => state.status,
      (status, prevStatus) => {
        // Coming back online
        if (prevStatus === 'offline' && status === 'online') {
          this.sync();
        }
      }
    );

    // Initial sync if online
    if (useNetworkStore.getState().status === 'online') {
      this.sync();
    }
  }

  stop(): void {
    this.unsubscribe?.();
    this.unsubscribe = null;
  }

  async sync(): Promise<void> {
    const { pendingCount } = useSyncStore.getState();

    if (pendingCount === 0) return;

    try {
      await offlineQueue.processQueue();
    } catch (error) {
      console.error('Sync failed:', error);
    }
  }

  // Manual sync trigger
  async forceSync(): Promise<void> {
    await this.sync();
  }
}

export const syncManager = new SyncManager();
```

### Network Provider

```tsx
// src/core/providers/network-provider.tsx
import { useEffect, type ReactNode } from 'react';
import { networkMonitor } from '@/core/services/network/monitor';
import { syncManager } from '@/core/services/network/sync';

interface NetworkProviderProps {
  children: ReactNode;
}

export const NetworkProvider = ({ children }: NetworkProviderProps) => {
  useEffect(() => {
    // Start network monitoring
    networkMonitor.start();
    syncManager.start();

    return () => {
      networkMonitor.stop();
      syncManager.stop();
    };
  }, []);

  return <>{children}</>;
};
```

## Optimistic UI Pattern

### Optimistic Mutation Hook

```tsx
// src/features/posts/api/use-create-post.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { v4 as uuid } from 'uuid';
import { client } from '@/core/services/api';
import { offlineQueue } from '@/core/services/network';
import { useNetworkStatus } from '@/shared/hooks';
import type { Post, CreatePostInput } from '../types';

export function useCreatePost() {
  const queryClient = useQueryClient();
  const { isOnline } = useNetworkStatus();

  return useMutation({
    mutationFn: async (input: CreatePostInput) => {
      if (!isOnline) {
        // Queue for later
        const tempId = uuid();
        await offlineQueue.enqueue({
          method: 'POST',
          url: '/posts',
          data: input,
          priority: 'normal',
          maxRetries: 3,
          metadata: {
            entityType: 'post',
            entityId: tempId,
            action: 'create',
          },
        });

        // Return optimistic response
        return {
          id: tempId,
          ...input,
          createdAt: new Date().toISOString(),
          _pending: true, // Mark as pending
        } as Post & { _pending: boolean };
      }

      const { data } = await client.post<Post>('/posts', input);
      return data;
    },

    onMutate: async (input) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['posts'] });

      // Snapshot previous value
      const previousPosts = queryClient.getQueryData<Post[]>(['posts']);

      // Optimistically add new post
      const optimisticPost: Post = {
        id: uuid(),
        ...input,
        createdAt: new Date().toISOString(),
      };

      queryClient.setQueryData<Post[]>(['posts'], (old) => [
        optimisticPost,
        ...(old ?? []),
      ]);

      return { previousPosts, optimisticPost };
    },

    onError: (_error, _input, context) => {
      // Rollback on error
      if (context?.previousPosts) {
        queryClient.setQueryData(['posts'], context.previousPosts);
      }
    },

    onSettled: () => {
      // Always refetch after error or success
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
  });
}
```

### Pending Indicator Component

```tsx
// src/shared/components/feedback/pending-indicator.tsx
import { View } from 'react-native';
import { CloudOff, RefreshCw } from 'lucide-react-native';
import Animated, { useAnimatedStyle, withRepeat, withTiming } from 'react-native-reanimated';
import { useSyncStore } from '@/store';

interface PendingIndicatorProps {
  isPending?: boolean;
}

export const PendingIndicator = ({ isPending }: PendingIndicatorProps) => {
  const isSyncing = useSyncStore.use.isSyncing();

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [
      {
        rotate: isSyncing
          ? withRepeat(withTiming('360deg', { duration: 1000 }), -1)
          : '0deg',
      },
    ],
  }));

  if (!isPending) return null;

  return (
    <Animated.View style={animatedStyle}>
      {isSyncing ? (
        <RefreshCw size={16} className="text-primary-500" />
      ) : (
        <CloudOff size={16} className="text-warning-500" />
      )}
    </Animated.View>
  );
};
```

### List Item with Pending State

```tsx
// src/features/posts/components/widgets/post-card.tsx
import { View } from 'react-native';
import { Text } from '@/shared/components/ui';
import { PendingIndicator } from '@/shared/components/feedback';
import type { Post } from '../../types';

interface PostCardProps {
  post: Post & { _pending?: boolean };
}

export const PostCard = ({ post }: PostCardProps) => (
  <View
    className={`rounded-xl p-4 bg-white dark:bg-neutral-800 ${
      post._pending ? 'opacity-70' : ''
    }`}
  >
    <View className="flex-row items-center justify-between">
      <Text variant="h3">{post.title}</Text>
      <PendingIndicator isPending={post._pending} />
    </View>
    <Text variant="body" className="mt-2">{post.body}</Text>
  </View>
);
```

## Conflict Resolution

### Version-Based Conflict Detection

```tsx
// src/core/services/network/conflict.ts
import type { QueuedRequest } from './types';

interface VersionedEntity {
  id: string;
  version: number;
  updatedAt: string;
}

type ConflictStrategy = 'client-wins' | 'server-wins' | 'merge' | 'ask-user';

interface ConflictResult<T> {
  resolved: boolean;
  data: T;
  strategy: ConflictStrategy;
}

export async function resolveConflict<T extends VersionedEntity>(
  clientData: T,
  serverData: T,
  strategy: ConflictStrategy = 'server-wins'
): Promise<ConflictResult<T>> {
  // No conflict if versions match
  if (clientData.version === serverData.version) {
    return { resolved: true, data: clientData, strategy };
  }

  switch (strategy) {
    case 'client-wins':
      return {
        resolved: true,
        data: { ...clientData, version: serverData.version + 1 },
        strategy,
      };

    case 'server-wins':
      return { resolved: true, data: serverData, strategy };

    case 'merge':
      // Merge non-conflicting fields
      const merged = mergeObjects(clientData, serverData);
      return {
        resolved: true,
        data: { ...merged, version: serverData.version + 1 },
        strategy,
      };

    case 'ask-user':
      // Return unresolved, UI should handle
      return { resolved: false, data: serverData, strategy };

    default:
      return { resolved: true, data: serverData, strategy: 'server-wins' };
  }
}

function mergeObjects<T extends object>(client: T, server: T): T {
  // Simple merge: prefer client values for changed fields
  return { ...server, ...client };
}
```

### Conflict UI Component

```tsx
// src/shared/components/feedback/conflict-resolver.tsx
import { View } from 'react-native';
import { Text, Button } from '@/shared/components/ui';
import { BottomSheetModal, type BottomSheetModalRef } from './bottom-sheet-modal';
import { useTranslation } from '@/core/i18n';

interface ConflictResolverProps<T> {
  localData: T;
  serverData: T;
  onResolve: (data: T, strategy: 'local' | 'server') => void;
  renderDiff: (local: T, server: T) => React.ReactNode;
  modalRef: React.RefObject<BottomSheetModalRef>;
}

export function ConflictResolver<T>({
  localData,
  serverData,
  onResolve,
  renderDiff,
  modalRef,
}: ConflictResolverProps<T>) {
  const { t } = useTranslation();

  return (
    <BottomSheetModal ref={modalRef} snapPoints={['60%']}>
      <View className="p-4">
        <Text variant="h2">{t('sync.conflict_title')}</Text>
        <Text variant="caption" className="mt-1">
          {t('sync.conflict_description')}
        </Text>

        <View className="mt-4">
          {renderDiff(localData, serverData)}
        </View>

        <View className="flex-row gap-3 mt-6">
          <Button
            label={t('sync.keep_local')}
            variant="outline"
            onPress={() => onResolve(localData, 'local')}
            fullWidth
          />
          <Button
            label={t('sync.use_server')}
            variant="primary"
            onPress={() => onResolve(serverData, 'server')}
            fullWidth
          />
        </View>
      </View>
    </BottomSheetModal>
  );
}
```

## Offline Banner

```tsx
// src/shared/components/feedback/offline-banner.tsx
import { View } from 'react-native';
import Animated, { FadeInUp, FadeOutUp } from 'react-native-reanimated';
import { WifiOff, RefreshCw } from 'lucide-react-native';
import { Text } from '@/shared/components/ui';
import { useNetworkStatus } from '@/shared/hooks';
import { useSyncStore } from '@/store';
import { useTranslation } from '@/core/i18n';

export const OfflineBanner = () => {
  const { t } = useTranslation();
  const { isOffline } = useNetworkStatus();
  const pendingCount = useSyncStore.use.pendingCount();
  const isSyncing = useSyncStore.use.isSyncing();

  if (!isOffline && pendingCount === 0) return null;

  return (
    <Animated.View
      entering={FadeInUp}
      exiting={FadeOutUp}
      className="bg-warning-500 px-4 py-2"
    >
      <View className="flex-row items-center justify-center gap-2">
        {isOffline ? (
          <>
            <WifiOff size={16} color="white" />
            <Text className="text-white font-medium">
              {t('status.offline')}
            </Text>
          </>
        ) : isSyncing ? (
          <>
            <RefreshCw size={16} color="white" />
            <Text className="text-white font-medium">
              {t('status.syncing')}
            </Text>
          </>
        ) : (
          <Text className="text-white font-medium">
            {t('status.pending_changes', { count: pendingCount })}
          </Text>
        )}
      </View>
    </Animated.View>
  );
};
```

## Usage in App Layout

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';
import { View } from 'react-native';
import { OfflineBanner } from '@/shared/components/feedback';
import { AppProviders } from '@/core/providers';

export default function RootLayout() {
  return (
    <AppProviders>
      <View className="flex-1">
        <OfflineBanner />
        <Stack />
      </View>
    </AppProviders>
  );
}
```

## React Query Offline Configuration

```tsx
// src/core/providers/query-provider.tsx
import { QueryClient, QueryClientProvider, onlineManager } from '@tanstack/react-query';
import NetInfo from '@react-native-community/netinfo';
import type { ReactNode } from 'react';

// Sync React Query online state with NetInfo
onlineManager.setEventListener((setOnline) => {
  return NetInfo.addEventListener((state) => {
    setOnline(!!state.isConnected);
  });
});

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 2,
      staleTime: 1000 * 60 * 5,
      gcTime: 1000 * 60 * 30,
      refetchOnWindowFocus: false,
      refetchOnReconnect: true,
      // Keep data when offline
      networkMode: 'offlineFirst',
    },
    mutations: {
      retry: 1,
      // Pause mutations when offline, resume when online
      networkMode: 'offlineFirst',
    },
  },
});

export const QueryProvider = ({ children }: { children: ReactNode }) => (
  <QueryClientProvider client={queryClient}>
    {children}
  </QueryClientProvider>
);

export { queryClient };
```
