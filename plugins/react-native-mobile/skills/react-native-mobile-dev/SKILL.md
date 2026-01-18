---
name: react-native-mobile-dev
description: Mobile app development with React Native and Expo using production-ready patterns. Use when (1) user is building a mobile app, (2) user asks about mobile architecture or best practices, (3) user needs help with React Native components, navigation, state management, or data fetching, (4) user mentions "mobile", "React Native", "Expo", "RN", "iOS app", or "Android app", (5) user wants to create a new React Native app.
---

# Mobile Development (React Native + Expo)

Expert guidance for building production-ready mobile applications using React Native and Expo with modern best practices.

## Create New App

Use **create-expo-stack** (rn.new) to scaffold new React Native apps with your preferred configuration.

### Quick Start (Recommended)

Interactive CLI that lets you choose your tech stack:

```bash
# Simplest way
npx rn-new@latest

# Or with package managers
pnpm create expo-stack
npm create expo-stack
yarn create expo-stack
bun create expo-stack
```

The CLI will prompt you to select:
- Navigation framework (Expo Router / React Navigation)
- Navigation type (Stack / Tabs / Drawer)
- Styling solution (NativeWind / Unistyles / Tamagui / Restyle / StyleSheet)
- Backend services (Firebase / Supabase / None)
- Additional features (i18n, import aliases, etc.)

### One-liner with Preset Options

For production-ready setup matching our tech stack:

```bash
# Full production setup with Expo Router + NativeWind + i18n
pnpm create expo-stack MyApp --expo-router --tabs --nativewind --i18next --import-alias

cd MyApp
pnpm ios      # iOS simulator
pnpm android  # Android emulator
```

### Available Options

| Category | Options |
|----------|---------|
| **Package Manager** | `--npm`, `--yarn`, `--pnpm`, `--bun` |
| **Navigation** | `--expo-router`, `--react-navigation` |
| **Nav Type** | `--tabs`, `--drawer` (default: stack) |
| **Styling** | `--nativewind`, `--unistyles`, `--tamagui`, `--restyle`, `--stylesheet` |
| **Backend** | `--firebase`, `--supabase` |
| **Features** | `--i18next`, `--import-alias` |
| **Other** | `-d/--default`, `--no-git`, `--no-install` |

### Prerequisites

- Node.js LTS
- pnpm (`npm install -g pnpm`)
- Watchman (macOS/Linux)
- Xcode (iOS) / Android Studio (Android)

### Post-setup

1. Open project in VS Code/Cursor and install recommended extensions (ESLint, Prettier, Tailwind CSS IntelliSense)
2. Update `app.config.ts` with your app name, bundle ID, etc.
3. Configure environment variables in `.env`
4. Run `eas init` to set up EAS Build

> **Docs:** https://docs.rn.new/ | **GitHub:** https://github.com/roninoss/create-expo-stack

---

## Tech Stack

Based on create-expo-stack (Expo SDK 54):

| Category         | Technology                                         |
| ---------------- | -------------------------------------------------- |
| Framework        | React Native 0.81 + React 19 + Expo SDK 54         |
| Navigation       | Expo Router 6 (file-based routing with typed routes) |
| Styling          | NativeWind 4 (Tailwind CSS) or Unistyles 3         |
| State Management | Zustand (add manually)                             |
| Data Fetching    | TanStack Query (React Query) + Axios (add manually) |
| Forms            | React Hook Form + Zod (add manually)               |
| Storage          | MMKV (recommended) or AsyncStorage                 |
| i18n             | i18next + react-i18next (use `--i18next` flag)     |
| Offline Support  | NetInfo + Custom Queue (add manually)              |
| Authentication   | Supabase or Firebase (optional flags)              |
| Testing          | Jest + React Native Testing Library                |

## Project Structure

Follow the recommended structure from [references/project-structure.md](references/project-structure.md).

```
├── app/                    # Expo Router screens and layouts
├── src/
│   ├── core/               # Core infrastructure (base layer)
│   │   ├── config/         # Environment variables, constants
│   │   ├── services/       # API client, storage, network, logger
│   │   │   ├── api/        # HTTP client, interceptors, error handler
│   │   │   ├── network/    # NetInfo monitor, offline queue, sync
│   │   │   └── storage/    # MMKV wrapper
│   │   ├── i18n/           # i18next config, RTL support
│   │   └── providers/      # Theme, Query, i18n, Network providers
│   ├── features/           # Feature modules (self-contained)
│   │   └── [feature]/      # screens/, widgets/, api/, hooks/, store/
│   ├── shared/             # Shared components, hooks, utils
│   │   ├── components/     # ui/, forms/, layout/, feedback/
│   │   └── hooks/          # useDebounce, useMounted, etc.
│   ├── store/              # Global state (auth, theme, network)
│   └── __tests__/          # Test setup, mocks, utilities
├── assets/                 # Static assets (images, fonts)
└── locales/                # i18n JSON files (common, errors, features)
```

### Architecture Layers

```
app/      → features/, shared/, core/, store/
features/ → shared/, core/, store/ (NO cross-feature imports)
shared/   → core/ only
core/     → No dependencies (base layer)
store/    → core/, shared/
```

## Coding Standards

Refer to [references/coding-standards.md](references/coding-standards.md) for detailed conventions.

### Key Rules

1. **File naming**: Use `kebab-case` for all files and folders (e.g., `login-form.tsx`)
2. **Imports**: Use `@/` alias for absolute imports (configure with `--import-alias`)
3. **Type imports**: Use `import type` for type-only imports
4. **Function limits**: Max 3 parameters, max 70 lines per function
5. **Exports**: Prefer named exports over default exports

## Component Development

### UI Components with NativeWind

Use Tailwind CSS classes with NativeWind. For complex variants, use `tailwind-variants`:

```tsx
import { tv } from "tailwind-variants";
import { Pressable, Text } from "react-native";

const button = tv({
  base: "rounded-xl px-4 py-3 items-center justify-center",
  variants: {
    variant: {
      primary: "bg-primary-500",
      secondary: "bg-neutral-200 dark:bg-neutral-700",
      destructive: "bg-danger-500",
    },
    disabled: {
      true: "opacity-50",
    },
  },
  defaultVariants: {
    variant: "primary",
  },
});

type ButtonProps = {
  label: string;
  variant?: "primary" | "secondary" | "destructive";
  disabled?: boolean;
  onPress: () => void;
};

export const Button = ({ label, variant, disabled, onPress }: ButtonProps) => (
  <Pressable
    className={button({ variant, disabled })}
    onPress={onPress}
    disabled={disabled}
  >
    <Text className="text-white font-semibold">{label}</Text>
  </Pressable>
);
```

### Forms with React Hook Form + Zod

```tsx
import { zodResolver } from "@hookform/resolvers/zod";
import { useForm } from "react-hook-form";
import * as z from "zod";

const schema = z.object({
  email: z.string().email("Invalid email"),
  password: z.string().min(6, "Min 6 characters"),
});

type FormData = z.infer<typeof schema>;

export const LoginForm = ({
  onSubmit,
}: {
  onSubmit: (data: FormData) => void;
}) => {
  const { control, handleSubmit } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  return (
    <View>
      <ControlledInput control={control} name="email" label="Email" />
      <ControlledInput
        control={control}
        name="password"
        label="Password"
        secureTextEntry
      />
      <Button label="Login" onPress={handleSubmit(onSubmit)} />
    </View>
  );
};
```

## State Management

### Zustand Store Pattern

```tsx
import { create } from "zustand";
import { createSelectors } from "@/lib/utils";

interface AuthState {
  token: string | null;
  status: "idle" | "signIn" | "signOut";
  signIn: (token: string) => void;
  signOut: () => void;
}

const _useAuth = create<AuthState>((set) => ({
  token: null,
  status: "idle",
  signIn: (token) => set({ token, status: "signIn" }),
  signOut: () => set({ token: null, status: "signOut" }),
}));

// Use createSelectors for optimized re-renders
export const useAuth = createSelectors(_useAuth);

// Export actions for use outside React
export const signIn = (token: string) => _useAuth.getState().signIn(token);
export const signOut = () => _useAuth.getState().signOut();
```

## Data Fetching

### React Query Pattern

```tsx
// api/posts/use-posts.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { client } from "../common/client";
import type { Post } from "./types";

export const usePosts = () => {
  return useQuery<Post[]>({
    queryKey: ["posts"],
    queryFn: () => client.get("/posts").then((res) => res.data),
  });
};

export const useCreatePost = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: Omit<Post, "id">) => client.post("/posts", data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["posts"] });
    },
  });
};
```

## Navigation (Expo Router)

### File-based Routing

```
app/
├── _layout.tsx        # Root layout
├── index.tsx          # Home screen (/)
├── login.tsx          # Login screen (/login)
├── (tabs)/            # Tab group
│   ├── _layout.tsx    # Tab layout
│   ├── home.tsx       # /home tab
│   └── profile.tsx    # /profile tab
└── [id].tsx           # Dynamic route (/:id)
```

### Protected Routes

```tsx
// app/_layout.tsx
import { useAuth } from "@/lib/auth";
import { Redirect, Stack } from "expo-router";

export default function RootLayout() {
  const status = useAuth.use.status();

  if (status === "signOut") {
    return <Redirect href="/login" />;
  }

  return <Stack />;
}
```

## Environment Variables

Use Zod for validation in `env.js`:

```javascript
const client = z.object({
  APP_ENV: z.enum(["development", "staging", "production"]),
  API_URL: z.string().url(),
  VERSION: z.string(),
});

// Access via @env alias
import Env from "@env";
console.log(Env.API_URL);
```

## Testing

### Component Testing

```tsx
import { render, screen, fireEvent } from "@/lib/test-utils";
import { LoginForm } from "./login-form";

describe("LoginForm", () => {
  it("validates email format", async () => {
    const onSubmit = jest.fn();
    render(<LoginForm onSubmit={onSubmit} />);

    fireEvent.changeText(screen.getByTestId("email-input"), "invalid");
    fireEvent.press(screen.getByTestId("login-button"));

    expect(await screen.findByText("Invalid email")).toBeTruthy();
    expect(onSubmit).not.toHaveBeenCalled();
  });
});
```

## Quick Commands

```bash
# Development
pnpm start                    # Start Expo dev server
pnpm ios                      # Run on iOS simulator
pnpm android                  # Run on Android emulator

# Testing
pnpm test                     # Run tests
pnpm lint                     # Run ESLint
pnpm typecheck               # Run TypeScript check

# Building
APP_ENV=staging pnpm build:ios      # Build iOS (staging)
APP_ENV=production pnpm build:android  # Build Android (production)
```

## Resources

### Architecture & Patterns
- **Project Structure**: [references/project-structure.md](references/project-structure.md)
- **Coding Standards**: [references/coding-standards.md](references/coding-standards.md)
- **Component Patterns**: [references/component-patterns.md](references/component-patterns.md)

### Data & Services
- **Data Layer**: [references/data-layer.md](references/data-layer.md)
- **Offline Support**: [references/offline.md](references/offline.md)

### Internationalization & Testing
- **i18n Guide**: [references/i18n.md](references/i18n.md)
- **Testing Guide**: [references/testing.md](references/testing.md)
