# Component Patterns

> These patterns assume you're using NativeWind (via `--nativewind` flag) with create-expo-stack. Includes i18n, RTL support, and offline state patterns.

## UI Primitives

### Button Component

```tsx
import { type VariantProps, tv } from 'tailwind-variants';
import { ActivityIndicator, Pressable, Text, View } from 'react-native';

const button = tv({
  slots: {
    container: 'flex-row items-center justify-center rounded-xl',
    label: 'font-semibold',
    indicator: '',
  },
  variants: {
    variant: {
      primary: {
        container: 'bg-primary-500',
        label: 'text-white',
        indicator: 'text-white',
      },
      secondary: {
        container: 'bg-neutral-200 dark:bg-neutral-700',
        label: 'text-neutral-900 dark:text-neutral-100',
        indicator: 'text-neutral-900 dark:text-neutral-100',
      },
      outline: {
        container: 'border border-neutral-300 bg-transparent',
        label: 'text-neutral-900 dark:text-neutral-100',
      },
      destructive: {
        container: 'bg-danger-500',
        label: 'text-white',
      },
      ghost: {
        container: 'bg-transparent',
        label: 'text-primary-500',
      },
    },
    size: {
      sm: { container: 'h-10 px-3', label: 'text-sm' },
      md: { container: 'h-12 px-4', label: 'text-base' },
      lg: { container: 'h-14 px-6', label: 'text-lg' },
    },
    disabled: {
      true: { container: 'opacity-50' },
    },
    fullWidth: {
      true: { container: 'w-full' },
    },
  },
  defaultVariants: {
    variant: 'primary',
    size: 'md',
  },
});

type ButtonVariants = VariantProps<typeof button>;

interface ButtonProps extends ButtonVariants {
  label: string;
  onPress: () => void;
  loading?: boolean;
  icon?: React.ReactNode;
  testID?: string;
}

export const Button = ({
  label,
  onPress,
  loading,
  icon,
  testID,
  variant,
  size,
  disabled,
  fullWidth,
}: ButtonProps) => {
  const styles = button({ variant, size, disabled: disabled || loading, fullWidth });

  return (
    <Pressable
      testID={testID}
      onPress={onPress}
      disabled={disabled || loading}
      className={styles.container()}
    >
      {loading ? (
        <ActivityIndicator className={styles.indicator()} />
      ) : (
        <View className="flex-row items-center gap-2">
          {icon}
          <Text className={styles.label()}>{label}</Text>
        </View>
      )}
    </Pressable>
  );
};
```

### Text Component

```tsx
import { type VariantProps, tv } from 'tailwind-variants';
import { Text as RNText, type TextProps as RNTextProps } from 'react-native';

const text = tv({
  base: 'text-neutral-900 dark:text-neutral-100 font-inter',
  variants: {
    variant: {
      h1: 'text-3xl font-bold',
      h2: 'text-2xl font-bold',
      h3: 'text-xl font-semibold',
      body: 'text-base',
      caption: 'text-sm text-neutral-500',
      error: 'text-sm text-danger-500',
    },
  },
  defaultVariants: {
    variant: 'body',
  },
});

type TextVariants = VariantProps<typeof text>;

interface TextProps extends RNTextProps, TextVariants {
  children: React.ReactNode;
}

export const Text = ({ variant, className, children, ...props }: TextProps) => (
  <RNText className={text({ variant, className })} {...props}>
    {children}
  </RNText>
);
```

### Input Component

```tsx
import { forwardRef, useState, useMemo, useCallback } from 'react';
import { TextInput, View, type TextInputProps } from 'react-native';
import { tv } from 'tailwind-variants';
import { Text } from './text';

const input = tv({
  slots: {
    container: 'mb-4',
    label: 'text-neutral-700 dark:text-neutral-300 mb-1 text-sm font-medium',
    field: 'rounded-xl border bg-neutral-50 px-4 py-3 text-base dark:bg-neutral-800',
    error: 'text-danger-500 mt-1 text-sm',
  },
  variants: {
    focused: {
      true: { field: 'border-primary-500' },
      false: { field: 'border-neutral-200 dark:border-neutral-700' },
    },
    hasError: {
      true: { field: 'border-danger-500', label: 'text-danger-500' },
    },
    disabled: {
      true: { field: 'bg-neutral-100 opacity-60' },
    },
  },
  defaultVariants: {
    focused: false,
    hasError: false,
    disabled: false,
  },
});

interface InputProps extends TextInputProps {
  label?: string;
  error?: string;
  disabled?: boolean;
}

export const Input = forwardRef<TextInput, InputProps>(
  ({ label, error, disabled, testID, ...props }, ref) => {
    const [isFocused, setIsFocused] = useState(false);

    const styles = useMemo(
      () => input({ focused: isFocused, hasError: !!error, disabled }),
      [isFocused, error, disabled]
    );

    const handleFocus = useCallback(() => setIsFocused(true), []);
    const handleBlur = useCallback(() => setIsFocused(false), []);

    return (
      <View className={styles.container()}>
        {label && (
          <Text testID={`${testID}-label`} className={styles.label()}>
            {label}
          </Text>
        )}
        <TextInput
          ref={ref}
          testID={testID}
          className={styles.field()}
          editable={!disabled}
          onFocus={handleFocus}
          onBlur={handleBlur}
          placeholderTextColor="#9CA3AF"
          {...props}
        />
        {error && (
          <Text testID={`${testID}-error`} className={styles.error()}>
            {error}
          </Text>
        )}
      </View>
    );
  }
);
```

## Controlled Form Components

### ControlledInput (React Hook Form)

```tsx
import type { Control, FieldValues, Path } from 'react-hook-form';
import { useController } from 'react-hook-form';
import { Input, type InputProps } from './input';

interface ControlledInputProps<T extends FieldValues>
  extends Omit<InputProps, 'value' | 'onChangeText'> {
  name: Path<T>;
  control: Control<T>;
}

export function ControlledInput<T extends FieldValues>({
  name,
  control,
  ...props
}: ControlledInputProps<T>) {
  const { field, fieldState } = useController({ name, control });

  return (
    <Input
      ref={field.ref}
      value={field.value}
      onChangeText={field.onChange}
      onBlur={field.onBlur}
      error={fieldState.error?.message}
      {...props}
    />
  );
}
```

### ControlledSelect

```tsx
import type { Control, FieldValues, Path } from 'react-hook-form';
import { useController } from 'react-hook-form';
import { Select, type SelectProps, type Option } from './select';

interface ControlledSelectProps<T extends FieldValues>
  extends Omit<SelectProps, 'value' | 'onSelect'> {
  name: Path<T>;
  control: Control<T>;
}

export function ControlledSelect<T extends FieldValues>({
  name,
  control,
  options,
  ...props
}: ControlledSelectProps<T>) {
  const { field, fieldState } = useController({ name, control });

  return (
    <Select
      value={field.value}
      onSelect={field.onChange}
      options={options}
      error={fieldState.error?.message}
      {...props}
    />
  );
}
```

## List Components

### FlatList with Pull-to-Refresh

```tsx
import { FlatList, RefreshControl, View } from 'react-native';
import { useCallback } from 'react';
import type { Post } from '@/features/posts/types';
import { PostCard } from './post-card';
import { EmptyState } from './empty-state';
import { ListSkeleton } from './list-skeleton';

interface PostListProps {
  posts: Post[];
  isLoading: boolean;
  isRefreshing: boolean;
  onRefresh: () => void;
  onEndReached?: () => void;
}

export const PostList = ({
  posts,
  isLoading,
  isRefreshing,
  onRefresh,
  onEndReached,
}: PostListProps) => {
  const renderItem = useCallback(
    ({ item }: { item: Post }) => <PostCard post={item} />,
    []
  );

  const keyExtractor = useCallback((item: Post) => item.id.toString(), []);

  if (isLoading) {
    return <ListSkeleton count={5} />;
  }

  return (
    <FlatList
      data={posts}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      contentContainerClassName="p-4 gap-4"
      ListEmptyComponent={<EmptyState message="No posts found" />}
      refreshControl={
        <RefreshControl refreshing={isRefreshing} onRefresh={onRefresh} />
      }
      onEndReached={onEndReached}
      onEndReachedThreshold={0.5}
    />
  );
};
```

### FlashList (High Performance)

```tsx
import { FlashList } from '@shopify/flash-list';

export const HighPerfList = ({ data }: { data: Item[] }) => (
  <FlashList
    data={data}
    renderItem={({ item }) => <ItemCard item={item} />}
    estimatedItemSize={100}  // Required for FlashList
    keyExtractor={(item) => item.id}
  />
);
```

## Modal Components

### Bottom Sheet Modal

```tsx
import { forwardRef, useCallback, useImperativeHandle, useMemo } from 'react';
import { View } from 'react-native';
import BottomSheet, {
  BottomSheetBackdrop,
  BottomSheetView,
  type BottomSheetBackdropProps,
} from '@gorhom/bottom-sheet';

interface BottomSheetModalProps {
  children: React.ReactNode;
  snapPoints?: string[];
}

export interface BottomSheetModalRef {
  open: () => void;
  close: () => void;
}

export const BottomSheetModal = forwardRef<BottomSheetModalRef, BottomSheetModalProps>(
  ({ children, snapPoints = ['50%'] }, ref) => {
    const bottomSheetRef = useRef<BottomSheet>(null);

    useImperativeHandle(ref, () => ({
      open: () => bottomSheetRef.current?.expand(),
      close: () => bottomSheetRef.current?.close(),
    }));

    const renderBackdrop = useCallback(
      (props: BottomSheetBackdropProps) => (
        <BottomSheetBackdrop {...props} disappearsOnIndex={-1} appearsOnIndex={0} />
      ),
      []
    );

    return (
      <BottomSheet
        ref={bottomSheetRef}
        index={-1}
        snapPoints={snapPoints}
        enablePanDownToClose
        backdropComponent={renderBackdrop}
      >
        <BottomSheetView className="flex-1 p-4">
          {children}
        </BottomSheetView>
      </BottomSheet>
    );
  }
);

// Usage
const modalRef = useRef<BottomSheetModalRef>(null);
<BottomSheetModal ref={modalRef}>
  <Text>Modal Content</Text>
</BottomSheetModal>
<Button label="Open" onPress={() => modalRef.current?.open()} />
```

## Image Components

### Optimized Image with expo-image

```tsx
import { Image as ExpoImage, type ImageProps } from 'expo-image';
import { cssInterop } from 'nativewind';

// Enable className support
cssInterop(ExpoImage, { className: 'style' });

interface OptimizedImageProps extends ImageProps {
  className?: string;
}

const blurhash = 'L6PZfSi_.AyE_3t7t7R**0o#DgR4';

export const Image = ({ className, placeholder = blurhash, ...props }: OptimizedImageProps) => (
  <ExpoImage
    className={className}
    placeholder={placeholder}
    contentFit="cover"
    transition={200}
    {...props}
  />
);

// Preload images
export const preloadImages = (urls: string[]) => ExpoImage.prefetch(urls);
```

## Loading States

### Skeleton Component

```tsx
import { useEffect } from 'react';
import Animated, {
  useAnimatedStyle,
  useSharedValue,
  withRepeat,
  withTiming,
} from 'react-native-reanimated';
import { View } from 'react-native';

interface SkeletonProps {
  width?: number | string;
  height?: number;
  className?: string;
}

export const Skeleton = ({ width = '100%', height = 20, className }: SkeletonProps) => {
  const opacity = useSharedValue(0.3);

  useEffect(() => {
    opacity.value = withRepeat(withTiming(0.7, { duration: 800 }), -1, true);
  }, []);

  const animatedStyle = useAnimatedStyle(() => ({
    opacity: opacity.value,
  }));

  return (
    <Animated.View
      style={[{ width, height }, animatedStyle]}
      className={`rounded-md bg-neutral-200 dark:bg-neutral-700 ${className}`}
    />
  );
};

// Card Skeleton
export const CardSkeleton = () => (
  <View className="rounded-xl bg-white p-4 dark:bg-neutral-800">
    <Skeleton height={200} className="mb-3 rounded-lg" />
    <Skeleton width="70%" height={24} className="mb-2" />
    <Skeleton width="90%" height={16} />
  </View>
);
```

## Error States

### Error Boundary

```tsx
import { Component, type ErrorInfo, type ReactNode } from 'react';
import { View } from 'react-native';
import { Text } from './ui/text';
import { Button } from './ui/button';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('ErrorBoundary caught:', error, errorInfo);
    // Log to error reporting service
  }

  handleRetry = () => {
    this.setState({ hasError: false, error: undefined });
  };

  render() {
    if (this.state.hasError) {
      return (
        this.props.fallback || (
          <View className="flex-1 items-center justify-center p-4">
            <Text variant="h2" className="mb-2">Something went wrong</Text>
            <Text variant="caption" className="mb-4 text-center">
              {this.state.error?.message}
            </Text>
            <Button label="Try Again" onPress={this.handleRetry} />
          </View>
        )
      );
    }

    return this.props.children;
  }
}
```

## i18n Integrated Components

### Localized Button

```tsx
import { type VariantProps, tv } from 'tailwind-variants';
import { ActivityIndicator, Pressable, Text, View } from 'react-native';
import { useTranslation } from '@/core/i18n';

const button = tv({
  slots: {
    container: 'flex-row items-center justify-center rounded-xl',
    label: 'font-semibold',
  },
  variants: {
    variant: {
      primary: { container: 'bg-primary-500', label: 'text-white' },
      secondary: { container: 'bg-neutral-200 dark:bg-neutral-700', label: 'text-neutral-900 dark:text-neutral-100' },
    },
    size: {
      sm: { container: 'h-10 px-3', label: 'text-sm' },
      md: { container: 'h-12 px-4', label: 'text-base' },
      lg: { container: 'h-14 px-6', label: 'text-lg' },
    },
    disabled: {
      true: { container: 'opacity-50' },
    },
  },
  defaultVariants: {
    variant: 'primary',
    size: 'md',
  },
});

type ButtonVariants = VariantProps<typeof button>;

interface LocalizedButtonProps extends ButtonVariants {
  // Use translation key instead of raw label
  labelKey: string;
  labelParams?: Record<string, unknown>;
  namespace?: string;
  onPress: () => void;
  loading?: boolean;
  testID?: string;
}

export const LocalizedButton = ({
  labelKey,
  labelParams,
  namespace = 'common',
  onPress,
  loading,
  testID,
  variant,
  size,
  disabled,
}: LocalizedButtonProps) => {
  const { t } = useTranslation(namespace);
  const styles = button({ variant, size, disabled: disabled || loading });

  return (
    <Pressable
      testID={testID}
      onPress={onPress}
      disabled={disabled || loading}
      className={styles.container()}
    >
      {loading ? (
        <ActivityIndicator color="white" />
      ) : (
        <Text className={styles.label()}>{t(labelKey, labelParams)}</Text>
      )}
    </Pressable>
  );
};

// Usage
<LocalizedButton
  labelKey="buttons.submit"
  onPress={handleSubmit}
/>

<LocalizedButton
  labelKey="login.submit"
  namespace="auth"
  onPress={handleLogin}
/>
```

### Localized Input

```tsx
import { forwardRef, useState, useMemo, useCallback } from 'react';
import { TextInput, View, type TextInputProps } from 'react-native';
import { tv } from 'tailwind-variants';
import { useTranslation } from '@/core/i18n';
import { Text } from './text';

const input = tv({
  slots: {
    container: 'mb-4',
    label: 'text-neutral-700 dark:text-neutral-300 mb-1 text-sm font-medium',
    field: 'rounded-xl border bg-neutral-50 px-4 py-3 text-base dark:bg-neutral-800',
    error: 'text-danger-500 mt-1 text-sm',
  },
  variants: {
    focused: {
      true: { field: 'border-primary-500' },
      false: { field: 'border-neutral-200 dark:border-neutral-700' },
    },
    hasError: {
      true: { field: 'border-danger-500' },
    },
  },
});

interface LocalizedInputProps extends Omit<TextInputProps, 'placeholder'> {
  labelKey: string;
  placeholderKey?: string;
  errorKey?: string;
  errorParams?: Record<string, unknown>;
  namespace?: string;
  disabled?: boolean;
}

export const LocalizedInput = forwardRef<TextInput, LocalizedInputProps>(
  ({ labelKey, placeholderKey, errorKey, errorParams, namespace = 'common', disabled, testID, ...props }, ref) => {
    const { t } = useTranslation(namespace);
    const { t: tErrors } = useTranslation('errors');
    const [isFocused, setIsFocused] = useState(false);

    const styles = useMemo(
      () => input({ focused: isFocused, hasError: !!errorKey }),
      [isFocused, errorKey]
    );

    return (
      <View className={styles.container()}>
        <Text testID={`${testID}-label`} className={styles.label()}>
          {t(labelKey)}
        </Text>
        <TextInput
          ref={ref}
          testID={testID}
          className={styles.field()}
          editable={!disabled}
          onFocus={() => setIsFocused(true)}
          onBlur={() => setIsFocused(false)}
          placeholder={placeholderKey ? t(placeholderKey) : undefined}
          placeholderTextColor="#9CA3AF"
          {...props}
        />
        {errorKey && (
          <Text testID={`${testID}-error`} className={styles.error()}>
            {tErrors(errorKey, errorParams)}
          </Text>
        )}
      </View>
    );
  }
);

// Usage
<LocalizedInput
  labelKey="login.email_label"
  placeholderKey="login.email_placeholder"
  errorKey={errors.email?.message}
  namespace="auth"
  testID="email-input"
/>
```

### Empty State with i18n

```tsx
import { View } from 'react-native';
import { useTranslation } from '@/core/i18n';
import { Text } from './ui/text';
import { Button } from './ui/button';
import { InboxIcon } from 'lucide-react-native';

interface EmptyStateProps {
  titleKey: string;
  descriptionKey?: string;
  actionKey?: string;
  namespace?: string;
  onAction?: () => void;
  icon?: React.ReactNode;
}

export const EmptyState = ({
  titleKey,
  descriptionKey,
  actionKey,
  namespace = 'common',
  onAction,
  icon,
}: EmptyStateProps) => {
  const { t } = useTranslation(namespace);

  return (
    <View className="flex-1 items-center justify-center p-8">
      {icon ?? <InboxIcon size={48} className="text-neutral-400 mb-4" />}
      <Text variant="h3" className="text-center mb-2">
        {t(titleKey)}
      </Text>
      {descriptionKey && (
        <Text variant="caption" className="text-center mb-4">
          {t(descriptionKey)}
        </Text>
      )}
      {actionKey && onAction && (
        <Button label={t(actionKey)} onPress={onAction} variant="primary" />
      )}
    </View>
  );
};

// Usage
<EmptyState
  titleKey="empty.no_posts"
  descriptionKey="empty.no_posts_description"
  actionKey="empty.create_first"
  onAction={() => router.push('/create')}
/>
```

## RTL-Aware Components

### RTL Container

```tsx
import { View, type ViewProps } from 'react-native';
import { I18nManager } from 'react-native';

interface RTLContainerProps extends ViewProps {
  children: React.ReactNode;
  reverse?: boolean;
}

export const RTLContainer = ({ children, reverse = false, className, ...props }: RTLContainerProps) => {
  const isRTL = I18nManager.isRTL;
  const shouldReverse = reverse ? !isRTL : isRTL;

  return (
    <View
      className={`flex-row ${shouldReverse ? 'flex-row-reverse' : ''} ${className ?? ''}`}
      {...props}
    >
      {children}
    </View>
  );
};

// Usage
<RTLContainer className="items-center gap-3">
  <Icon />
  <Text>{t('label')}</Text>
</RTLContainer>
```

### Directional Icon

```tsx
import { View } from 'react-native';
import { I18nManager } from 'react-native';
import { ChevronRight, ChevronLeft } from 'lucide-react-native';

interface DirectionalIconProps {
  size?: number;
  color?: string;
  direction?: 'forward' | 'back';
}

export const DirectionalIcon = ({
  size = 24,
  color = '#000',
  direction = 'forward',
}: DirectionalIconProps) => {
  const isRTL = I18nManager.isRTL;

  // In RTL, "forward" means left, "back" means right
  const showLeft = direction === 'forward' ? isRTL : !isRTL;

  return showLeft ? (
    <ChevronLeft size={size} color={color} />
  ) : (
    <ChevronRight size={size} color={color} />
  );
};

// Usage - will automatically flip in RTL
<DirectionalIcon direction="forward" />
```

### RTL-Aware List Item

```tsx
import { View, Pressable } from 'react-native';
import { I18nManager } from 'react-native';
import { Text } from './ui/text';
import { DirectionalIcon } from './directional-icon';

interface ListItemProps {
  icon: React.ReactNode;
  label: string;
  value?: string;
  onPress?: () => void;
  showChevron?: boolean;
}

export const ListItem = ({ icon, label, value, onPress, showChevron = true }: ListItemProps) => {
  const isRTL = I18nManager.isRTL;

  return (
    <Pressable
      onPress={onPress}
      className={`flex-row items-center p-4 bg-white dark:bg-neutral-800 ${
        isRTL ? 'flex-row-reverse' : ''
      }`}
    >
      <View className={isRTL ? 'ml-3' : 'mr-3'}>{icon}</View>
      <View className="flex-1">
        <Text className={isRTL ? 'text-right' : 'text-left'}>{label}</Text>
      </View>
      {value && (
        <Text variant="caption" className={isRTL ? 'ml-2' : 'mr-2'}>
          {value}
        </Text>
      )}
      {showChevron && <DirectionalIcon direction="forward" color="#9CA3AF" />}
    </Pressable>
  );
};
```

## Offline-Aware Components

### Offline Banner

```tsx
import { View } from 'react-native';
import Animated, { FadeInUp, FadeOutUp } from 'react-native-reanimated';
import { WifiOff, RefreshCw } from 'lucide-react-native';
import { Text } from './ui/text';
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
            <Text className="text-white font-medium">{t('status.offline')}</Text>
          </>
        ) : isSyncing ? (
          <>
            <RefreshCw size={16} color="white" />
            <Text className="text-white font-medium">{t('status.syncing')}</Text>
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

### Pending Indicator

```tsx
import { View } from 'react-native';
import { CloudOff, RefreshCw } from 'lucide-react-native';
import Animated, { useAnimatedStyle, withRepeat, withTiming } from 'react-native-reanimated';
import { useSyncStore } from '@/store';

interface PendingIndicatorProps {
  isPending?: boolean;
  size?: number;
}

export const PendingIndicator = ({ isPending, size = 16 }: PendingIndicatorProps) => {
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
        <RefreshCw size={size} className="text-primary-500" />
      ) : (
        <CloudOff size={size} className="text-warning-500" />
      )}
    </Animated.View>
  );
};
```

### Offline-Aware Button

```tsx
import { ActivityIndicator, Pressable, Text, View } from 'react-native';
import { WifiOff } from 'lucide-react-native';
import { useNetworkStatus } from '@/shared/hooks';
import { useTranslation } from '@/core/i18n';

interface OfflineAwareButtonProps {
  labelKey: string;
  offlineLabelKey?: string;
  namespace?: string;
  onPress: () => void;
  loading?: boolean;
  disabled?: boolean;
  requiresNetwork?: boolean;
}

export const OfflineAwareButton = ({
  labelKey,
  offlineLabelKey = 'buttons.save_offline',
  namespace = 'common',
  onPress,
  loading,
  disabled,
  requiresNetwork = false,
}: OfflineAwareButtonProps) => {
  const { t } = useTranslation(namespace);
  const { isOffline } = useNetworkStatus();

  const isDisabled = disabled || loading || (requiresNetwork && isOffline);
  const showOfflineIndicator = isOffline && !requiresNetwork;

  return (
    <Pressable
      onPress={onPress}
      disabled={isDisabled}
      className={`flex-row items-center justify-center rounded-xl h-12 px-4 ${
        isDisabled ? 'bg-neutral-300' : 'bg-primary-500'
      }`}
    >
      {loading ? (
        <ActivityIndicator color="white" />
      ) : (
        <View className="flex-row items-center gap-2">
          {showOfflineIndicator && <WifiOff size={16} color="white" />}
          <Text className="text-white font-semibold">
            {showOfflineIndicator ? t(offlineLabelKey) : t(labelKey)}
          </Text>
        </View>
      )}
    </Pressable>
  );
};

// Usage
<OfflineAwareButton
  labelKey="buttons.submit"
  onPress={handleSubmit}
  loading={isLoading}
/>

// For actions that require network
<OfflineAwareButton
  labelKey="buttons.refresh"
  onPress={handleRefresh}
  requiresNetwork
/>
```

### Card with Pending State

```tsx
import { View, Pressable } from 'react-native';
import { Text } from './ui/text';
import { PendingIndicator } from './pending-indicator';

interface CardWithPendingProps<T> {
  item: T & { _pending?: boolean };
  renderContent: (item: T) => React.ReactNode;
  onPress?: () => void;
}

export function CardWithPending<T>({
  item,
  renderContent,
  onPress,
}: CardWithPendingProps<T>) {
  return (
    <Pressable
      onPress={onPress}
      disabled={item._pending}
      className={`rounded-xl p-4 bg-white dark:bg-neutral-800 ${
        item._pending ? 'opacity-70' : ''
      }`}
    >
      <View className="absolute top-2 right-2">
        <PendingIndicator isPending={item._pending} />
      </View>
      {renderContent(item)}
    </Pressable>
  );
}

// Usage
<CardWithPending
  item={post}
  onPress={() => router.push(`/posts/${post.id}`)}
  renderContent={(p) => (
    <>
      <Text variant="h3">{p.title}</Text>
      <Text variant="body">{p.body}</Text>
    </>
  )}
/>
```
