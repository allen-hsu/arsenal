# Animation Patterns

Meaningful motion that enhances UX without distracting. Using React Native Reanimated.

## Core Principles

1. **Purpose**: Every animation should have a reason
2. **Performance**: Use `useAnimatedStyle` and worklets
3. **Subtlety**: Most animations should be 200-400ms
4. **Consistency**: Use the same easing across your app

## Spring Configurations

```tsx
// Natural, bouncy feel
const springBouncy = {
  damping: 10,
  stiffness: 100,
  mass: 0.5,
};

// Responsive, controlled
const springResponsive = {
  damping: 15,
  stiffness: 150,
  mass: 0.5,
};

// Gentle, slow settle
const springGentle = {
  damping: 20,
  stiffness: 80,
  mass: 1,
};

// Quick snap
const springSnappy = {
  damping: 20,
  stiffness: 300,
  mass: 0.5,
};
```

## Timing Configurations

```tsx
import { Easing } from 'react-native-reanimated';

// Standard easing
const easeOut = Easing.bezier(0.25, 0.1, 0.25, 1);
const easeInOut = Easing.bezier(0.42, 0, 0.58, 1);

// Expressive
const easeOutBack = Easing.bezier(0.34, 1.56, 0.64, 1);
const easeOutExpo = Easing.bezier(0.16, 1, 0.3, 1);

// Duration standards
const DURATION = {
  instant: 100,
  fast: 200,
  normal: 300,
  slow: 500,
  emphasis: 800,
};
```

## Micro-Interactions

### Press Feedback

```tsx
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
} from 'react-native-reanimated';
import { Gesture, GestureDetector } from 'react-native-gesture-handler';

const PressableScale = ({ children, onPress, scale = 0.95 }) => {
  const pressed = useSharedValue(1);

  const gesture = Gesture.Tap()
    .onBegin(() => {
      pressed.value = withSpring(scale, springResponsive);
    })
    .onFinalize(() => {
      pressed.value = withSpring(1, springResponsive);
      runOnJS(onPress)?.();
    });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: pressed.value }],
  }));

  return (
    <GestureDetector gesture={gesture}>
      <Animated.View style={animatedStyle}>{children}</Animated.View>
    </GestureDetector>
  );
};
```

### Toggle Switch

```tsx
const AnimatedSwitch = ({ value, onValueChange }) => {
  const translateX = useSharedValue(value ? 22 : 2);

  useEffect(() => {
    translateX.value = withSpring(value ? 22 : 2, springResponsive);
  }, [value]);

  const thumbStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: translateX.value }],
  }));

  const trackStyle = useAnimatedStyle(() => ({
    backgroundColor: interpolateColor(
      translateX.value,
      [2, 22],
      ['#d1d5db', '#6366f1']
    ),
  }));

  return (
    <Pressable onPress={() => onValueChange(!value)}>
      <Animated.View style={[styles.track, trackStyle]}>
        <Animated.View style={[styles.thumb, thumbStyle]} />
      </Animated.View>
    </Pressable>
  );
};
```

### Like Button Pop

```tsx
const LikeButton = ({ liked, onPress }) => {
  const scale = useSharedValue(1);

  const handlePress = () => {
    if (!liked) {
      // Pop effect on like
      scale.value = withSequence(
        withSpring(1.3, { damping: 5 }),
        withSpring(1, springResponsive)
      );
    }
    onPress();
  };

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  return (
    <Pressable onPress={handlePress}>
      <Animated.View style={animatedStyle}>
        <HeartIcon filled={liked} color={liked ? '#ef4444' : '#9ca3af'} />
      </Animated.View>
    </Pressable>
  );
};
```

## List Animations

### Staggered Entry

```tsx
import Animated, { FadeIn, FadeOut, Layout } from 'react-native-reanimated';

const AnimatedListItem = ({ index, children }) => (
  <Animated.View
    entering={FadeIn.delay(index * 50).springify().damping(15)}
    exiting={FadeOut.duration(200)}
    layout={Layout.springify()}
  >
    {children}
  </Animated.View>
);

// Usage
{items.map((item, index) => (
  <AnimatedListItem key={item.id} index={index}>
    <ItemCard item={item} />
  </AnimatedListItem>
))}
```

### Swipe to Delete

```tsx
const SwipeableRow = ({ children, onDelete }) => {
  const translateX = useSharedValue(0);
  const rowHeight = useSharedValue(80);
  const opacity = useSharedValue(1);

  const gesture = Gesture.Pan()
    .activeOffsetX([-10, 10])
    .onUpdate((e) => {
      translateX.value = Math.max(-100, Math.min(0, e.translationX));
    })
    .onEnd(() => {
      if (translateX.value < -80) {
        // Delete
        translateX.value = withTiming(-400, { duration: 200 });
        rowHeight.value = withTiming(0, { duration: 200 });
        opacity.value = withTiming(0, { duration: 200 }, () => {
          runOnJS(onDelete)();
        });
      } else {
        translateX.value = withSpring(0, springResponsive);
      }
    });

  const rowStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: translateX.value }],
    height: rowHeight.value,
    opacity: opacity.value,
  }));

  return (
    <View className="relative overflow-hidden">
      {/* Delete background */}
      <View className="absolute inset-y-0 right-0 w-24 bg-red-500 items-center justify-center">
        <TrashIcon color="white" />
      </View>

      <GestureDetector gesture={gesture}>
        <Animated.View style={rowStyle}>{children}</Animated.View>
      </GestureDetector>
    </View>
  );
};
```

## Screen Transitions

### Shared Element

```tsx
// List screen
<Link href={`/detail/${item.id}`} asChild>
  <Pressable>
    <Animated.Image
      sharedTransitionTag={`image-${item.id}`}
      source={{ uri: item.image }}
      className="w-full h-48 rounded-xl"
    />
  </Pressable>
</Link>

// Detail screen
<Animated.Image
  sharedTransitionTag={`image-${id}`}
  source={{ uri: item.image }}
  className="w-full h-80"
/>
```

### Hero Header Collapse

```tsx
const AnimatedHeader = ({ scrollY, title }) => {
  const headerHeight = useAnimatedStyle(() => ({
    height: interpolate(
      scrollY.value,
      [0, 100],
      [200, 60],
      Extrapolation.CLAMP
    ),
  }));

  const titleStyle = useAnimatedStyle(() => ({
    fontSize: interpolate(
      scrollY.value,
      [0, 100],
      [32, 18],
      Extrapolation.CLAMP
    ),
    opacity: interpolate(
      scrollY.value,
      [0, 50],
      [1, 0.8],
      Extrapolation.CLAMP
    ),
  }));

  return (
    <Animated.View style={headerHeight} className="bg-white dark:bg-neutral-900">
      <Animated.Text style={titleStyle} className="font-bold">
        {title}
      </Animated.Text>
    </Animated.View>
  );
};
```

## Loading States

### Skeleton Pulse

```tsx
const Skeleton = ({ width, height }) => {
  const opacity = useSharedValue(0.3);

  useEffect(() => {
    opacity.value = withRepeat(
      withTiming(0.7, { duration: 800 }),
      -1,
      true
    );
  }, []);

  const animatedStyle = useAnimatedStyle(() => ({
    opacity: opacity.value,
  }));

  return (
    <Animated.View
      style={[{ width, height }, animatedStyle]}
      className="rounded-lg bg-neutral-200 dark:bg-neutral-700"
    />
  );
};
```

### Progress Ring

```tsx
import Svg, { Circle } from 'react-native-svg';

const AnimatedCircle = Animated.createAnimatedComponent(Circle);

const ProgressRing = ({ progress, size = 60, strokeWidth = 4 }) => {
  const radius = (size - strokeWidth) / 2;
  const circumference = 2 * Math.PI * radius;

  const strokeDashoffset = useAnimatedStyle(() => ({
    strokeDashoffset: circumference * (1 - progress.value),
  }));

  return (
    <Svg width={size} height={size}>
      {/* Background */}
      <Circle
        cx={size / 2}
        cy={size / 2}
        r={radius}
        stroke="#e5e7eb"
        strokeWidth={strokeWidth}
        fill="none"
      />
      {/* Progress */}
      <AnimatedCircle
        cx={size / 2}
        cy={size / 2}
        r={radius}
        stroke="#6366f1"
        strokeWidth={strokeWidth}
        fill="none"
        strokeDasharray={circumference}
        strokeLinecap="round"
        animatedProps={strokeDashoffset}
        transform={`rotate(-90 ${size / 2} ${size / 2})`}
      />
    </Svg>
  );
};
```

## Gestures

### Pull to Refresh

```tsx
const PullToRefresh = ({ onRefresh, children }) => {
  const translateY = useSharedValue(0);
  const isRefreshing = useSharedValue(false);

  const gesture = Gesture.Pan()
    .onUpdate((e) => {
      if (e.translationY > 0 && !isRefreshing.value) {
        translateY.value = Math.min(e.translationY * 0.5, 100);
      }
    })
    .onEnd(() => {
      if (translateY.value > 60) {
        isRefreshing.value = true;
        translateY.value = withSpring(60);
        runOnJS(onRefresh)().then(() => {
          isRefreshing.value = false;
          translateY.value = withSpring(0);
        });
      } else {
        translateY.value = withSpring(0);
      }
    });

  const contentStyle = useAnimatedStyle(() => ({
    transform: [{ translateY: translateY.value }],
  }));

  const indicatorStyle = useAnimatedStyle(() => ({
    opacity: interpolate(translateY.value, [0, 60], [0, 1]),
    transform: [
      { rotate: `${interpolate(translateY.value, [0, 60], [0, 360])}deg` },
    ],
  }));

  return (
    <View>
      <Animated.View style={indicatorStyle} className="absolute top-4 self-center">
        <RefreshIcon />
      </Animated.View>
      <GestureDetector gesture={gesture}>
        <Animated.View style={contentStyle}>{children}</Animated.View>
      </GestureDetector>
    </View>
  );
};
```

## Performance Tips

1. **Use worklets** for animation logic
2. **Avoid re-renders** during animation
3. **Use `useAnimatedStyle`** instead of inline styles
4. **Batch updates** with `runOnJS`
5. **Use `cancelAnimation`** on unmount

```tsx
useEffect(() => {
  return () => {
    cancelAnimation(sharedValue);
  };
}, []);
```
