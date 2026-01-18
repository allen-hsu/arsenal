# Component Gallery

Production-ready React Native components with distinctive design.

## Buttons

### Gradient Button

```tsx
import { LinearGradient } from 'expo-linear-gradient';
import Animated, { useAnimatedStyle, useSharedValue, withSpring } from 'react-native-reanimated';

const GradientButton = ({ label, onPress, colors = ['#6366f1', '#8b5cf6'] }) => {
  const scale = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  return (
    <Pressable
      onPressIn={() => (scale.value = withSpring(0.95))}
      onPressOut={() => (scale.value = withSpring(1))}
      onPress={onPress}
    >
      <Animated.View style={animatedStyle}>
        <LinearGradient
          colors={colors}
          start={{ x: 0, y: 0 }}
          end={{ x: 1, y: 1 }}
          className="h-14 rounded-2xl items-center justify-center px-8"
        >
          <Text className="text-white font-semibold text-lg">{label}</Text>
        </LinearGradient>
      </Animated.View>
    </Pressable>
  );
};
```

### Icon Button with Badge

```tsx
const IconButton = ({ icon: Icon, badge, onPress }) => (
  <Pressable
    onPress={onPress}
    className="relative w-12 h-12 rounded-full bg-neutral-100 dark:bg-neutral-800 items-center justify-center"
  >
    <Icon size={24} className="text-neutral-700 dark:text-neutral-200" />
    {badge > 0 && (
      <View className="absolute -top-1 -right-1 min-w-[20px] h-5 rounded-full bg-red-500 items-center justify-center px-1">
        <Text className="text-white text-xs font-bold">
          {badge > 99 ? '99+' : badge}
        </Text>
      </View>
    )}
  </Pressable>
);
```

### Pill Button Group

```tsx
const PillButtonGroup = ({ options, selected, onSelect }) => (
  <View className="flex-row bg-neutral-100 dark:bg-neutral-800 rounded-full p-1">
    {options.map((option) => (
      <Pressable
        key={option.value}
        onPress={() => onSelect(option.value)}
        className={`flex-1 py-2 px-4 rounded-full ${
          selected === option.value
            ? 'bg-white dark:bg-neutral-700 shadow-sm'
            : ''
        }`}
      >
        <Text
          className={`text-center font-medium ${
            selected === option.value
              ? 'text-neutral-900 dark:text-white'
              : 'text-neutral-500'
          }`}
        >
          {option.label}
        </Text>
      </Pressable>
    ))}
  </View>
);
```

## Cards

### Feature Card with Icon

```tsx
const FeatureCard = ({ icon: Icon, title, description, color = '#6366f1' }) => (
  <View className="bg-white dark:bg-neutral-800 rounded-3xl p-6 shadow-sm">
    <View
      style={{ backgroundColor: `${color}15` }}
      className="w-14 h-14 rounded-2xl items-center justify-center mb-4"
    >
      <Icon size={28} color={color} />
    </View>
    <Text className="text-xl font-bold text-neutral-900 dark:text-white mb-2">
      {title}
    </Text>
    <Text className="text-neutral-500 dark:text-neutral-400 leading-relaxed">
      {description}
    </Text>
  </View>
);
```

### Image Card with Overlay

```tsx
const ImageCard = ({ image, title, subtitle, onPress }) => (
  <Pressable onPress={onPress} className="overflow-hidden rounded-3xl">
    <Image source={{ uri: image }} className="w-full h-48" />
    <LinearGradient
      colors={['transparent', 'rgba(0,0,0,0.8)']}
      className="absolute inset-0 justify-end p-4"
    >
      <Text className="text-white font-bold text-xl">{title}</Text>
      <Text className="text-white/70 text-sm">{subtitle}</Text>
    </LinearGradient>
  </Pressable>
);
```

### Stats Card

```tsx
const StatsCard = ({ label, value, change, trend }) => (
  <View className="bg-white dark:bg-neutral-800 rounded-2xl p-4 flex-1">
    <Text className="text-neutral-500 dark:text-neutral-400 text-sm mb-1">
      {label}
    </Text>
    <Text className="text-2xl font-bold text-neutral-900 dark:text-white">
      {value}
    </Text>
    <View className="flex-row items-center mt-2">
      <View
        className={`flex-row items-center px-2 py-0.5 rounded-full ${
          trend === 'up' ? 'bg-green-100 dark:bg-green-900/30' : 'bg-red-100 dark:bg-red-900/30'
        }`}
      >
        {trend === 'up' ? (
          <ArrowUpIcon size={12} className="text-green-600" />
        ) : (
          <ArrowDownIcon size={12} className="text-red-600" />
        )}
        <Text
          className={`text-xs font-medium ml-0.5 ${
            trend === 'up' ? 'text-green-600' : 'text-red-600'
          }`}
        >
          {change}
        </Text>
      </View>
    </View>
  </View>
);
```

## Lists

### Avatar List Item

```tsx
const AvatarListItem = ({ avatar, name, subtitle, trailing, onPress }) => (
  <Pressable
    onPress={onPress}
    className="flex-row items-center py-3 px-4 active:bg-neutral-50 dark:active:bg-neutral-800"
  >
    <Image source={{ uri: avatar }} className="w-12 h-12 rounded-full" />
    <View className="flex-1 ml-3">
      <Text className="font-semibold text-neutral-900 dark:text-white">
        {name}
      </Text>
      <Text className="text-neutral-500 dark:text-neutral-400 text-sm">
        {subtitle}
      </Text>
    </View>
    {trailing}
  </Pressable>
);
```

### Settings Row

```tsx
const SettingsRow = ({ icon: Icon, label, value, onPress, showArrow = true }) => (
  <Pressable
    onPress={onPress}
    className="flex-row items-center py-4 px-4 bg-white dark:bg-neutral-800"
  >
    <View className="w-10 h-10 rounded-xl bg-primary-100 dark:bg-primary-900/30 items-center justify-center">
      <Icon size={20} className="text-primary-600 dark:text-primary-400" />
    </View>
    <Text className="flex-1 ml-3 text-neutral-900 dark:text-white font-medium">
      {label}
    </Text>
    {value && (
      <Text className="text-neutral-500 dark:text-neutral-400 mr-2">
        {value}
      </Text>
    )}
    {showArrow && <ChevronRightIcon size={20} className="text-neutral-400" />}
  </Pressable>
);
```

## Inputs

### Search Bar

```tsx
const SearchBar = ({ value, onChangeText, placeholder = 'Search...' }) => {
  const [focused, setFocused] = useState(false);

  return (
    <View
      className={`flex-row items-center px-4 h-12 rounded-2xl ${
        focused
          ? 'bg-white dark:bg-neutral-800 border-2 border-primary-500'
          : 'bg-neutral-100 dark:bg-neutral-800'
      }`}
    >
      <SearchIcon size={20} className="text-neutral-400" />
      <TextInput
        value={value}
        onChangeText={onChangeText}
        placeholder={placeholder}
        placeholderTextColor="#9ca3af"
        onFocus={() => setFocused(true)}
        onBlur={() => setFocused(false)}
        className="flex-1 ml-3 text-neutral-900 dark:text-white"
      />
      {value.length > 0 && (
        <Pressable onPress={() => onChangeText('')}>
          <XCircleIcon size={20} className="text-neutral-400" />
        </Pressable>
      )}
    </View>
  );
};
```

### OTP Input

```tsx
const OTPInput = ({ length = 6, value, onChange }) => {
  const inputRefs = useRef([]);

  const handleChange = (text, index) => {
    const newValue = value.split('');
    newValue[index] = text;
    onChange(newValue.join(''));

    if (text && index < length - 1) {
      inputRefs.current[index + 1]?.focus();
    }
  };

  const handleKeyPress = (e, index) => {
    if (e.nativeEvent.key === 'Backspace' && !value[index] && index > 0) {
      inputRefs.current[index - 1]?.focus();
    }
  };

  return (
    <View className="flex-row justify-center gap-3">
      {Array.from({ length }).map((_, index) => (
        <TextInput
          key={index}
          ref={(ref) => (inputRefs.current[index] = ref)}
          value={value[index] || ''}
          onChangeText={(text) => handleChange(text.slice(-1), index)}
          onKeyPress={(e) => handleKeyPress(e, index)}
          keyboardType="number-pad"
          maxLength={1}
          className={`w-12 h-14 rounded-xl text-center text-2xl font-bold ${
            value[index]
              ? 'bg-primary-50 dark:bg-primary-900/30 border-2 border-primary-500'
              : 'bg-neutral-100 dark:bg-neutral-800 border border-neutral-200 dark:border-neutral-700'
          } text-neutral-900 dark:text-white`}
        />
      ))}
    </View>
  );
};
```

## Navigation

### Bottom Tab Bar

```tsx
const TabBar = ({ tabs, activeTab, onTabPress }) => {
  const insets = useSafeAreaInsets();

  return (
    <View
      style={{ paddingBottom: insets.bottom }}
      className="flex-row bg-white dark:bg-neutral-900 border-t border-neutral-100 dark:border-neutral-800"
    >
      {tabs.map((tab) => {
        const isActive = activeTab === tab.key;
        return (
          <Pressable
            key={tab.key}
            onPress={() => onTabPress(tab.key)}
            className="flex-1 items-center py-2"
          >
            <View
              className={`p-2 rounded-xl ${
                isActive ? 'bg-primary-100 dark:bg-primary-900/30' : ''
              }`}
            >
              <tab.icon
                size={24}
                className={
                  isActive
                    ? 'text-primary-600 dark:text-primary-400'
                    : 'text-neutral-400'
                }
              />
            </View>
            <Text
              className={`text-xs mt-1 ${
                isActive
                  ? 'text-primary-600 dark:text-primary-400 font-medium'
                  : 'text-neutral-400'
              }`}
            >
              {tab.label}
            </Text>
          </Pressable>
        );
      })}
    </View>
  );
};
```

### Floating Action Button

```tsx
const FAB = ({ icon: Icon, onPress, extended, label }) => (
  <Pressable
    onPress={onPress}
    className={`absolute bottom-6 right-6 bg-primary-500 shadow-lg shadow-primary-500/30 items-center justify-center ${
      extended ? 'flex-row px-6 py-4 rounded-full' : 'w-14 h-14 rounded-full'
    }`}
  >
    <Icon size={24} color="white" />
    {extended && (
      <Text className="text-white font-semibold ml-2">{label}</Text>
    )}
  </Pressable>
);
```

## Feedback

### Toast Notification

```tsx
const Toast = ({ message, type = 'info', visible }) => {
  const translateY = useSharedValue(-100);

  useEffect(() => {
    translateY.value = withSpring(visible ? 0 : -100);
  }, [visible]);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateY: translateY.value }],
  }));

  const colors = {
    success: 'bg-green-500',
    error: 'bg-red-500',
    warning: 'bg-amber-500',
    info: 'bg-primary-500',
  };

  return (
    <Animated.View
      style={animatedStyle}
      className={`absolute top-safe left-4 right-4 ${colors[type]} rounded-2xl p-4 flex-row items-center shadow-lg`}
    >
      <Text className="flex-1 text-white font-medium">{message}</Text>
    </Animated.View>
  );
};
```

### Empty State

```tsx
const EmptyState = ({ icon: Icon, title, description, action }) => (
  <View className="flex-1 items-center justify-center p-8">
    <View className="w-20 h-20 rounded-full bg-neutral-100 dark:bg-neutral-800 items-center justify-center mb-6">
      <Icon size={40} className="text-neutral-400" />
    </View>
    <Text className="text-xl font-bold text-neutral-900 dark:text-white text-center mb-2">
      {title}
    </Text>
    <Text className="text-neutral-500 dark:text-neutral-400 text-center mb-6 max-w-xs">
      {description}
    </Text>
    {action && (
      <Pressable
        onPress={action.onPress}
        className="bg-primary-500 px-6 py-3 rounded-full"
      >
        <Text className="text-white font-semibold">{action.label}</Text>
      </Pressable>
    )}
  </View>
);
```

## Modals

### Action Sheet

```tsx
import BottomSheet, { BottomSheetView } from '@gorhom/bottom-sheet';

const ActionSheet = ({ options, onSelect, onClose }) => {
  const bottomSheetRef = useRef(null);

  return (
    <BottomSheet
      ref={bottomSheetRef}
      index={0}
      snapPoints={['40%']}
      enablePanDownToClose
      onClose={onClose}
      backgroundStyle={{ backgroundColor: '#1c1917' }}
      handleIndicatorStyle={{ backgroundColor: '#57534e' }}
    >
      <BottomSheetView className="p-4">
        {options.map((option, index) => (
          <Pressable
            key={index}
            onPress={() => {
              onSelect(option);
              bottomSheetRef.current?.close();
            }}
            className="flex-row items-center py-4 border-b border-neutral-800"
          >
            {option.icon && (
              <option.icon size={24} className="text-neutral-400 mr-4" />
            )}
            <Text
              className={`flex-1 text-lg ${
                option.destructive ? 'text-red-500' : 'text-white'
              }`}
            >
              {option.label}
            </Text>
          </Pressable>
        ))}
      </BottomSheetView>
    </BottomSheet>
  );
};
```

### Confirmation Dialog

```tsx
const ConfirmDialog = ({ visible, title, message, onConfirm, onCancel }) => (
  <Modal visible={visible} transparent animationType="fade">
    <Pressable
      onPress={onCancel}
      className="flex-1 bg-black/50 items-center justify-center p-6"
    >
      <Pressable className="bg-white dark:bg-neutral-800 rounded-3xl p-6 w-full max-w-sm">
        <Text className="text-xl font-bold text-neutral-900 dark:text-white text-center mb-2">
          {title}
        </Text>
        <Text className="text-neutral-500 dark:text-neutral-400 text-center mb-6">
          {message}
        </Text>
        <View className="flex-row gap-3">
          <Pressable
            onPress={onCancel}
            className="flex-1 py-3 rounded-xl bg-neutral-100 dark:bg-neutral-700"
          >
            <Text className="text-center font-semibold text-neutral-900 dark:text-white">
              Cancel
            </Text>
          </Pressable>
          <Pressable
            onPress={onConfirm}
            className="flex-1 py-3 rounded-xl bg-red-500"
          >
            <Text className="text-center font-semibold text-white">
              Delete
            </Text>
          </Pressable>
        </View>
      </Pressable>
    </Pressable>
  </Modal>
);
```
