# Color Palettes

Curated color palettes for distinctive mobile app designs. Avoid defaults—choose a palette that reflects your app's personality.

## Bold & Vibrant

### Coral Energy
Best for: Social apps, lifestyle, entertainment

```javascript
module.exports = {
  primary: {
    50: '#fff5f5',
    100: '#ffe0e0',
    200: '#ffc7c7',
    300: '#ffa3a3',
    400: '#ff7070',
    500: '#ff4757',  // Hero
    600: '#ed2d3f',
    700: '#c71f30',
    800: '#a41c2a',
    900: '#881b28',
  },
  accent: '#5352ed',
  background: {
    light: '#ffffff',
    dark: '#1a1a2e',
  },
};
```

### Electric Indigo
Best for: Fintech, productivity, professional

```javascript
module.exports = {
  primary: {
    50: '#eef2ff',
    100: '#e0e7ff',
    200: '#c7d2fe',
    300: '#a5b4fc',
    400: '#818cf8',
    500: '#6366f1',  // Hero
    600: '#4f46e5',
    700: '#4338ca',
    800: '#3730a3',
    900: '#312e81',
  },
  accent: '#06ffa5',
  background: {
    light: '#fafafe',
    dark: '#0f0f23',
  },
};
```

### Sunset Gradient
Best for: Creative apps, photo/video, social

```javascript
module.exports = {
  gradient: {
    start: '#f857a6',
    end: '#ff5858',
    // Or multi-stop
    stops: ['#f857a6', '#ff6b6b', '#feca57'],
  },
  primary: '#f857a6',
  accent: '#5f27cd',
  background: {
    light: '#ffffff',
    dark: '#16162a',
  },
};
```

## Minimal & Refined

### Ink & Paper
Best for: Reading apps, notes, documentation

```javascript
module.exports = {
  primary: {
    50: '#f8f8f8',
    100: '#f0f0f0',
    200: '#e4e4e4',
    300: '#d1d1d1',
    400: '#b4b4b4',
    500: '#9a9a9a',
    600: '#818181',
    700: '#6a6a6a',
    800: '#5a5a5a',
    900: '#1a1a1a',  // Hero (text)
  },
  accent: '#0066cc',
  background: {
    light: '#fffef9',  // Warm paper
    dark: '#1a1a1a',
  },
};
```

### Stone & Sage
Best for: Wellness, meditation, eco-friendly

```javascript
module.exports = {
  primary: {
    50: '#f6f7f6',
    100: '#e3e6e3',
    200: '#c7cdc7',
    300: '#a3afa3',
    400: '#7d8c7d',
    500: '#5f705f',  // Hero
    600: '#4a594a',
    700: '#3d493d',
    800: '#333d33',
    900: '#2b332b',
  },
  accent: '#d4a574',  // Warm terracotta
  background: {
    light: '#faf9f7',
    dark: '#1c1f1c',
  },
};
```

## Luxury & Premium

### Obsidian Gold
Best for: Banking, luxury brands, premium services

```javascript
module.exports = {
  primary: {
    50: '#fdf9f0',
    100: '#faf0d7',
    200: '#f4dfae',
    300: '#ecc97b',
    400: '#e4b04d',
    500: '#d4952b',  // Gold hero
    600: '#bc7720',
    700: '#9c5a1c',
    800: '#80481e',
    900: '#6a3c1c',
  },
  neutral: {
    50: '#f7f7f7',
    100: '#e3e3e3',
    200: '#c8c8c8',
    300: '#a4a4a4',
    400: '#818181',
    500: '#666666',
    600: '#515151',
    700: '#434343',
    800: '#2d2d2d',
    900: '#1a1a1a',
    950: '#0d0d0d',  // Deep black
  },
  background: {
    light: '#fafafa',
    dark: '#0d0d0d',
  },
};
```

### Midnight Blue
Best for: Analytics, dashboards, data apps

```javascript
module.exports = {
  primary: {
    50: '#eef5ff',
    100: '#d9e8ff',
    200: '#bcd7ff',
    300: '#8ebeff',
    400: '#599aff',
    500: '#3374ff',
    600: '#1a52f5',
    700: '#133de1',
    800: '#1633b6',
    900: '#18308f',
    950: '#0f1d57',  // Deep midnight
  },
  accent: '#00d9ff',  // Cyan highlight
  background: {
    light: '#f8fafc',
    dark: '#0a0f1a',
  },
};
```

## Playful & Fun

### Candy Pop
Best for: Games, kids apps, casual social

```javascript
module.exports = {
  primary: '#ff6b9d',
  secondary: '#c44dff',
  accent: '#00d4aa',
  highlight: '#ffd93d',
  background: {
    light: '#fff8fa',
    dark: '#1a1025',
  },
  gradients: {
    primary: ['#ff6b9d', '#c44dff'],
    accent: ['#00d4aa', '#00b4d8'],
  },
};
```

### Neon Night
Best for: Music apps, nightlife, gaming

```javascript
module.exports = {
  primary: '#ff00ff',     // Magenta
  secondary: '#00ffff',   // Cyan
  accent: '#ffff00',      // Yellow
  background: {
    light: '#1a1a2e',
    dark: '#0a0a14',
  },
  glow: {
    magenta: 'rgba(255, 0, 255, 0.5)',
    cyan: 'rgba(0, 255, 255, 0.5)',
  },
};
```

## Semantic Colors

Always include these for consistency:

```javascript
const semantic = {
  success: {
    light: '#10b981',
    dark: '#34d399',
    bg: {
      light: '#ecfdf5',
      dark: '#064e3b',
    },
  },
  warning: {
    light: '#f59e0b',
    dark: '#fbbf24',
    bg: {
      light: '#fffbeb',
      dark: '#78350f',
    },
  },
  error: {
    light: '#ef4444',
    dark: '#f87171',
    bg: {
      light: '#fef2f2',
      dark: '#7f1d1d',
    },
  },
  info: {
    light: '#3b82f6',
    dark: '#60a5fa',
    bg: {
      light: '#eff6ff',
      dark: '#1e3a8a',
    },
  },
};
```

## Tailwind Config Example

```javascript
// tailwind.config.js
const colors = require('./src/components/ui/colors');

module.exports = {
  theme: {
    extend: {
      colors: {
        primary: colors.primary,
        accent: colors.accent,
        neutral: colors.neutral,
        success: colors.semantic.success,
        warning: colors.semantic.warning,
        danger: colors.semantic.error,
      },
    },
  },
};
```

## Dark Mode Strategy

### Option 1: Invert Primary
```javascript
// Light mode: dark primary on light bg
// Dark mode: light primary on dark bg
primary: {
  light: '#1a1a1a',  // Used in light mode
  dark: '#ffffff',   // Used in dark mode
}
```

### Option 2: Shift Brightness
```javascript
// Shift the scale for dark mode
// Light mode uses 500-900
// Dark mode uses 100-500
<View className="bg-primary-600 dark:bg-primary-400" />
```

### Option 3: Complementary Accent
```javascript
// Different accent color for dark mode
accent: {
  light: '#6366f1',  // Indigo
  dark: '#818cf8',   // Lighter indigo
}
```
