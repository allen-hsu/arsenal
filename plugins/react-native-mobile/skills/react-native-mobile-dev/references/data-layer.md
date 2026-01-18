# Data Layer

> These patterns are for adding data fetching and state management to your create-expo-stack project. Install required packages manually: `@tanstack/react-query`, `axios`, `zustand`, `react-native-mmkv`.

## API Client Setup

### Axios Configuration

```tsx
// src/core/services/api/client.ts
import axios, { type AxiosError, type InternalAxiosRequestConfig } from 'axios';
import { useAuthStore } from '@/store';
import { API_URL } from '@/core/config/env';

export const client = axios.create({
  baseURL: API_URL,
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor - add auth token
client.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    const token = useAuthStore.getState().token;
    if (token) {
      config.headers.Authorization = `Bearer ${token.access}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor - handle errors
client.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    if (error.response?.status === 401) {
      useAuthStore.getState().signOut();
    }
    return Promise.reject(error);
  }
);
```

### Error Handler

```tsx
// src/core/services/api/error-handler.ts
import type { AxiosError } from 'axios';
import { useAuthStore } from '@/store';
import { offlineQueue } from '@/core/services/network';

export interface ApiError {
  message: string;
  code: string;
  status: number;
  details?: Record<string, string[]>;
}

export function handleApiError(error: AxiosError): ApiError {
  const status = error.response?.status ?? 0;
  const data = error.response?.data as Record<string, unknown> | undefined;

  // Network error (no response)
  if (!error.response) {
    return {
      message: 'Network error. Please check your connection.',
      code: 'NETWORK_ERROR',
      status: 0,
    };
  }

  switch (status) {
    case 400:
      return {
        message: data?.message as string ?? 'Invalid request',
        code: 'BAD_REQUEST',
        status,
        details: data?.errors as Record<string, string[]>,
      };

    case 401:
      return {
        message: 'Session expired. Please sign in again.',
        code: 'UNAUTHORIZED',
        status,
      };

    case 403:
      return {
        message: 'You do not have permission to perform this action.',
        code: 'FORBIDDEN',
        status,
      };

    case 404:
      return {
        message: 'Resource not found.',
        code: 'NOT_FOUND',
        status,
      };

    case 422:
      return {
        message: data?.message as string ?? 'Validation error',
        code: 'VALIDATION_ERROR',
        status,
        details: data?.errors as Record<string, string[]>,
      };

    case 429:
      return {
        message: 'Too many requests. Please wait and try again.',
        code: 'RATE_LIMITED',
        status,
      };

    case 500:
    case 502:
    case 503:
      return {
        message: 'Server error. Please try again later.',
        code: 'SERVER_ERROR',
        status,
      };

    default:
      return {
        message: 'An unexpected error occurred.',
        code: 'UNKNOWN_ERROR',
        status,
      };
  }
}
```

### Token Refresh Pattern

```tsx
// src/core/services/api/interceptors.ts
import axios, { type AxiosError, type InternalAxiosRequestConfig } from 'axios';
import { client } from './client';
import { useAuthStore } from '@/store';
import { API_URL } from '@/core/config/env';

let isRefreshing = false;
let failedQueue: Array<{
  resolve: (token: string) => void;
  reject: (error: Error) => void;
}> = [];

const processQueue = (error: Error | null, token: string | null) => {
  failedQueue.forEach((promise) => {
    if (error) {
      promise.reject(error);
    } else if (token) {
      promise.resolve(token);
    }
  });
  failedQueue = [];
};

// Response interceptor with token refresh
client.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    const originalRequest = error.config as InternalAxiosRequestConfig & {
      _retry?: boolean;
    };

    // Skip if not 401 or already retried
    if (error.response?.status !== 401 || originalRequest._retry) {
      return Promise.reject(error);
    }

    // Skip refresh for auth endpoints
    if (originalRequest.url?.includes('/auth/')) {
      return Promise.reject(error);
    }

    if (isRefreshing) {
      // Queue this request while refreshing
      return new Promise((resolve, reject) => {
        failedQueue.push({
          resolve: (token: string) => {
            originalRequest.headers.Authorization = `Bearer ${token}`;
            resolve(client(originalRequest));
          },
          reject,
        });
      });
    }

    originalRequest._retry = true;
    isRefreshing = true;

    try {
      const refreshToken = useAuthStore.getState().token?.refresh;

      if (!refreshToken) {
        throw new Error('No refresh token');
      }

      // Call refresh endpoint
      const { data } = await axios.post(`${API_URL}/auth/refresh`, {
        refreshToken,
      });

      const newToken = data.token;
      useAuthStore.getState().signIn(newToken);

      // Retry original request
      originalRequest.headers.Authorization = `Bearer ${newToken.access}`;

      processQueue(null, newToken.access);

      return client(originalRequest);
    } catch (refreshError) {
      processQueue(refreshError as Error, null);
      useAuthStore.getState().signOut();
      return Promise.reject(refreshError);
    } finally {
      isRefreshing = false;
    }
  }
);
```

### React Query Provider

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
      staleTime: 1000 * 60 * 5, // 5 minutes
      gcTime: 1000 * 60 * 30,   // 30 minutes
      refetchOnWindowFocus: false,
      refetchOnReconnect: true,
      networkMode: 'offlineFirst',
    },
    mutations: {
      retry: 1,
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

## Feature API Hooks

### Basic Query Hook

```tsx
// src/features/posts/api/use-posts.ts
import { useQuery } from '@tanstack/react-query';
import { client } from '@/core/services/api';
import { handleApiError } from '@/core/services/api/error-handler';
import type { Post } from '../types';

const POSTS_KEY = ['posts'];

async function fetchPosts(): Promise<Post[]> {
  try {
    const { data } = await client.get<Post[]>('/posts');
    return data;
  } catch (error) {
    throw handleApiError(error as any);
  }
}

export function usePosts() {
  return useQuery({
    queryKey: POSTS_KEY,
    queryFn: fetchPosts,
  });
}
```

### Query with Parameters

```tsx
// src/features/posts/api/use-post.ts
import { useQuery } from '@tanstack/react-query';
import { client } from '@/core/services/api';
import { handleApiError } from '@/core/services/api/error-handler';
import type { Post } from '../types';

async function fetchPost(id: number): Promise<Post> {
  try {
    const { data } = await client.get<Post>(`/posts/${id}`);
    return data;
  } catch (error) {
    throw handleApiError(error as any);
  }
}

export function usePost(id: number) {
  return useQuery({
    queryKey: ['posts', id],
    queryFn: () => fetchPost(id),
    enabled: !!id,
  });
}
```

### Infinite Query (Pagination)

```tsx
// src/features/posts/api/use-infinite-posts.ts
import { useInfiniteQuery } from '@tanstack/react-query';
import { client } from '@/core/services/api';
import { handleApiError } from '@/core/services/api/error-handler';
import type { Post, PaginatedResponse } from '../types';

interface PostsParams {
  page: number;
  limit: number;
}

async function fetchPosts({ page, limit }: PostsParams): Promise<PaginatedResponse<Post>> {
  try {
    const { data } = await client.get<PaginatedResponse<Post>>('/posts', {
      params: { page, limit },
    });
    return data;
  } catch (error) {
    throw handleApiError(error as any);
  }
}

export function useInfinitePosts(limit = 20) {
  return useInfiniteQuery({
    queryKey: ['posts', 'infinite', limit],
    queryFn: ({ pageParam = 1 }) => fetchPosts({ page: pageParam, limit }),
    initialPageParam: 1,
    getNextPageParam: (lastPage) =>
      lastPage.hasMore ? lastPage.page + 1 : undefined,
  });
}

// Usage in component
const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfinitePosts();
const posts = data?.pages.flatMap((page) => page.items) ?? [];
```

### Query with Request Cancellation

```tsx
// src/features/search/api/use-search.ts
import { useQuery } from '@tanstack/react-query';
import { client } from '@/core/services/api';
import type { SearchResult } from '../types';

async function searchItems(
  query: string,
  signal?: AbortSignal
): Promise<SearchResult[]> {
  const { data } = await client.get<SearchResult[]>('/search', {
    params: { q: query },
    signal, // Pass abort signal
  });
  return data;
}

export function useSearch(query: string) {
  return useQuery({
    queryKey: ['search', query],
    queryFn: ({ signal }) => searchItems(query, signal),
    enabled: query.length >= 2,
    staleTime: 1000 * 60, // Cache for 1 minute
  });
}
```

## Mutation Hooks

### Basic Mutation with Error Handling

```tsx
// src/features/posts/api/use-create-post.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { client } from '@/core/services/api';
import { handleApiError, type ApiError } from '@/core/services/api/error-handler';
import type { Post, CreatePostInput } from '../types';

async function createPost(input: CreatePostInput): Promise<Post> {
  try {
    const { data } = await client.post<Post>('/posts', input);
    return data;
  } catch (error) {
    throw handleApiError(error as any);
  }
}

export function useCreatePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createPost,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
    onError: (error: ApiError) => {
      // Error is already transformed by handleApiError
      console.error('Create post failed:', error.message);
    },
  });
}
```

### Mutation with Offline Queue

```tsx
// src/features/posts/api/use-create-post.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { v4 as uuid } from 'uuid';
import { client } from '@/core/services/api';
import { offlineQueue } from '@/core/services/network';
import { useNetworkStatus } from '@/shared/hooks';
import { handleApiError } from '@/core/services/api/error-handler';
import type { Post, CreatePostInput } from '../types';

export function useCreatePost() {
  const queryClient = useQueryClient();
  const { isOnline } = useNetworkStatus();

  return useMutation({
    mutationFn: async (input: CreatePostInput) => {
      if (!isOnline) {
        // Queue for later sync
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
          _pending: true,
        } as Post & { _pending: boolean };
      }

      try {
        const { data } = await client.post<Post>('/posts', input);
        return data;
      } catch (error) {
        throw handleApiError(error as any);
      }
    },

    onMutate: async (input) => {
      await queryClient.cancelQueries({ queryKey: ['posts'] });
      const previousPosts = queryClient.getQueryData<Post[]>(['posts']);

      // Optimistically add
      const optimisticPost: Post = {
        id: uuid(),
        ...input,
        createdAt: new Date().toISOString(),
      };

      queryClient.setQueryData<Post[]>(['posts'], (old) => [
        optimisticPost,
        ...(old ?? []),
      ]);

      return { previousPosts };
    },

    onError: (_error, _input, context) => {
      if (context?.previousPosts) {
        queryClient.setQueryData(['posts'], context.previousPosts);
      }
    },

    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
  });
}
```

### Optimistic Update

```tsx
// src/features/posts/api/use-update-post.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { client } from '@/core/services/api';
import { handleApiError } from '@/core/services/api/error-handler';
import type { Post, UpdatePostInput } from '../types';

interface UpdatePostParams {
  id: number;
  data: UpdatePostInput;
}

async function updatePost({ id, data }: UpdatePostParams): Promise<Post> {
  try {
    const { data: result } = await client.patch<Post>(`/posts/${id}`, data);
    return result;
  } catch (error) {
    throw handleApiError(error as any);
  }
}

export function useUpdatePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: updatePost,
    onMutate: async ({ id, data }) => {
      await queryClient.cancelQueries({ queryKey: ['posts', id] });
      const previousPost = queryClient.getQueryData<Post>(['posts', id]);

      queryClient.setQueryData<Post>(['posts', id], (old) =>
        old ? { ...old, ...data } : undefined
      );

      return { previousPost };
    },
    onError: (_error, { id }, context) => {
      if (context?.previousPost) {
        queryClient.setQueryData(['posts', id], context.previousPost);
      }
    },
    onSettled: (_data, _error, { id }) => {
      queryClient.invalidateQueries({ queryKey: ['posts', id] });
    },
  });
}
```

### Delete Mutation

```tsx
// src/features/posts/api/use-delete-post.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { client } from '@/core/services/api';
import { handleApiError } from '@/core/services/api/error-handler';

async function deletePost(id: number): Promise<void> {
  try {
    await client.delete(`/posts/${id}`);
  } catch (error) {
    throw handleApiError(error as any);
  }
}

export function useDeletePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: deletePost,
    onSuccess: (_data, id) => {
      queryClient.removeQueries({ queryKey: ['posts', id] });
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
  });
}
```

## Global State (Zustand)

### Auth Store

```tsx
// src/store/auth-store.ts
import { create } from 'zustand';
import { storage } from '@/core/services/storage';
import { createSelectors } from '@/shared/utils/zustand';

const TOKEN_KEY = 'auth-token';

export interface TokenType {
  access: string;
  refresh: string;
}

interface AuthState {
  token: TokenType | null;
  status: 'idle' | 'signIn' | 'signOut';
  signIn: (token: TokenType) => void;
  signOut: () => void;
  hydrate: () => void;
}

const _useAuthStore = create<AuthState>((set, get) => ({
  token: null,
  status: 'idle',

  signIn: (token) => {
    storage.set(TOKEN_KEY, JSON.stringify(token));
    set({ token, status: 'signIn' });
  },

  signOut: () => {
    storage.delete(TOKEN_KEY);
    set({ token: null, status: 'signOut' });
  },

  hydrate: () => {
    try {
      const storedToken = storage.getString(TOKEN_KEY);
      if (storedToken) {
        const token = JSON.parse(storedToken) as TokenType;
        get().signIn(token);
      } else {
        get().signOut();
      }
    } catch {
      get().signOut();
    }
  },
}));

export const useAuthStore = createSelectors(_useAuthStore);

// Export actions for use outside React
export const signIn = (token: TokenType) => _useAuthStore.getState().signIn(token);
export const signOut = () => _useAuthStore.getState().signOut();
export const hydrateAuth = () => _useAuthStore.getState().hydrate();
```

### Create Selectors Utility

```tsx
// src/shared/utils/zustand.ts
import type { StoreApi, UseBoundStore } from 'zustand';

type WithSelectors<S> = S extends { getState: () => infer T }
  ? S & { use: { [K in keyof T]: () => T[K] } }
  : never;

export const createSelectors = <S extends UseBoundStore<StoreApi<object>>>(
  _store: S
) => {
  const store = _store as WithSelectors<typeof _store>;
  store.use = {};

  for (const k of Object.keys(store.getState())) {
    (store.use as Record<string, () => unknown>)[k] = () =>
      store((s) => s[k as keyof typeof s]);
  }

  return store;
};

// Usage:
// const status = useAuthStore.use.status();  // Only re-renders when status changes
// const token = useAuthStore.use.token();    // Only re-renders when token changes
```

### Theme Store

```tsx
// src/store/theme-store.ts
import { create } from 'zustand';
import { createJSONStorage, persist } from 'zustand/middleware';
import { zustandStorage } from '@/core/services/storage';
import { createSelectors } from '@/shared/utils/zustand';

type Theme = 'light' | 'dark' | 'system';

interface ThemeState {
  theme: Theme;
  setTheme: (theme: Theme) => void;
}

const _useThemeStore = create<ThemeState>()(
  persist(
    (set) => ({
      theme: 'system',
      setTheme: (theme) => set({ theme }),
    }),
    {
      name: 'theme-storage',
      storage: createJSONStorage(() => zustandStorage),
    }
  )
);

export const useThemeStore = createSelectors(_useThemeStore);
```

### Store Barrel Export

```tsx
// src/store/index.ts
export * from './auth-store';
export * from './theme-store';
export * from './network-store';
export * from './sync-store';
```

## Storage (MMKV)

### Storage Setup

```tsx
// src/core/services/storage/index.ts
import { MMKV } from 'react-native-mmkv';
import type { StateStorage } from 'zustand/middleware';

export const storage = new MMKV();

// Zustand storage adapter
export const zustandStorage: StateStorage = {
  setItem: (name, value) => storage.set(name, value),
  getItem: (name) => storage.getString(name) ?? null,
  removeItem: (name) => storage.delete(name),
};

// Typed storage helpers
export function getStorageItem<T>(key: string): T | null {
  const value = storage.getString(key);
  if (!value) return null;
  try {
    return JSON.parse(value) as T;
  } catch {
    return null;
  }
}

export function setStorageItem<T>(key: string, value: T): void {
  storage.set(key, JSON.stringify(value));
}
```

## Types

### Shared API Types

```tsx
// src/shared/types/api.ts
export interface PaginatedResponse<T> {
  items: T[];
  page: number;
  limit: number;
  total: number;
  hasMore: boolean;
}

export interface ApiError {
  message: string;
  code: string;
  status: number;
  details?: Record<string, string[]>;
}
```

### Feature Types

```tsx
// src/features/posts/types/index.ts
export interface Post {
  id: number;
  title: string;
  body: string;
  userId: number;
  createdAt: string;
  updatedAt: string;
}

export interface CreatePostInput {
  title: string;
  body: string;
}

export interface UpdatePostInput {
  title?: string;
  body?: string;
}
```

## Usage Example

```tsx
// app/(tabs)/feed.tsx
import { View } from 'react-native';
import { usePosts, useCreatePost } from '@/features/posts';
import { PostList } from '@/features/posts/components';
import { CreatePostForm } from '@/features/posts/components';
import { ErrorBoundary, OfflineBanner } from '@/shared/components/feedback';
import { useTranslation } from '@/core/i18n';
import type { CreatePostInput } from '@/features/posts/types';

export default function FeedScreen() {
  const { t } = useTranslation();
  const { data: posts, isLoading, refetch, isRefetching, error } = usePosts();
  const createPost = useCreatePost();

  const handleCreatePost = async (data: CreatePostInput) => {
    await createPost.mutateAsync(data);
  };

  return (
    <ErrorBoundary>
      <View className="flex-1">
        <OfflineBanner />
        <CreatePostForm
          onSubmit={handleCreatePost}
          isLoading={createPost.isPending}
          error={createPost.error?.message}
        />
        <PostList
          posts={posts ?? []}
          isLoading={isLoading}
          isRefreshing={isRefetching}
          onRefresh={refetch}
          error={error?.message}
        />
      </View>
    </ErrorBoundary>
  );
}
```
