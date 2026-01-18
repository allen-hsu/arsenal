# Testing Architecture

> Comprehensive testing setup for React Native apps using Jest and React Native Testing Library. Packages are included with create-expo-stack.

## Directory Structure

```
src/
├── __tests__/                    # Global test utilities
│   ├── setup.ts                  # Jest setup file
│   ├── mocks/                    # Global mocks
│   │   ├── api.ts                # API client mock
│   │   ├── navigation.ts         # Expo Router mock
│   │   ├── stores.ts             # Zustand stores mock
│   │   ├── i18n.ts               # i18next mock
│   │   └── async-storage.ts      # MMKV mock
│   └── utils/                    # Test utilities
│       ├── render.tsx            # Custom render with providers
│       ├── factories.ts          # Test data factories
│       └── matchers.ts           # Custom matchers
│
└── features/
    └── [feature]/
        └── __tests__/            # Feature-specific tests
            ├── components/       # Component tests
            │   └── login-form.test.tsx
            ├── hooks/            # Hook tests
            │   └── use-auth.test.ts
            └── api/              # API hook tests
                └── use-login.test.ts
```

## Jest Configuration

### jest.config.js

```javascript
// jest.config.js
module.exports = {
  preset: 'jest-expo',
  setupFilesAfterEnv: ['<rootDir>/src/__tests__/setup.ts'],
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?)|expo(nent)?|@expo(nent)?/.*|@expo-google-fonts/.*|react-navigation|@react-navigation/.*|@unimodules/.*|unimodules|sentry-expo|native-base|react-native-svg)',
  ],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '^@/../locales/(.*)$': '<rootDir>/locales/$1',
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/__tests__/**',
    '!src/**/types/**',
  ],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
  },
  testMatch: ['**/__tests__/**/*.test.{ts,tsx}'],
};
```

### Test Setup

```tsx
// src/__tests__/setup.ts
import '@testing-library/react-native/extend-expect';
import { cleanup } from '@testing-library/react-native';

// Mock native modules
jest.mock('react-native/Libraries/Animated/NativeAnimatedHelper');

// Mock expo modules
jest.mock('expo-font');
jest.mock('expo-asset');
jest.mock('expo-constants', () => ({
  default: { expoConfig: { name: 'TestApp' } },
}));

// Mock MMKV
jest.mock('react-native-mmkv', () => require('./__tests__/mocks/async-storage').mockMMKV);

// Mock i18n
jest.mock('@/core/i18n', () => require('./__tests__/mocks/i18n').mockI18n);

// Mock navigation
jest.mock('expo-router', () => require('./__tests__/mocks/navigation').mockRouter);

// Cleanup after each test
afterEach(() => {
  cleanup();
  jest.clearAllMocks();
});

// Silence console warnings in tests
const originalWarn = console.warn;
console.warn = (...args) => {
  if (args[0]?.includes?.('ReactDOM.render is no longer supported')) return;
  originalWarn(...args);
};
```

## Mocks

### API Mock

```tsx
// src/__tests__/mocks/api.ts
import type { AxiosInstance, AxiosRequestConfig } from 'axios';

type MockResponse<T = unknown> = {
  data: T;
  status: number;
};

class MockApiClient {
  private responses: Map<string, MockResponse> = new Map();

  mockGet<T>(url: string, response: T, status = 200): void {
    this.responses.set(`GET:${url}`, { data: response, status });
  }

  mockPost<T>(url: string, response: T, status = 200): void {
    this.responses.set(`POST:${url}`, { data: response, status });
  }

  mockPut<T>(url: string, response: T, status = 200): void {
    this.responses.set(`PUT:${url}`, { data: response, status });
  }

  mockDelete<T>(url: string, response: T, status = 200): void {
    this.responses.set(`DELETE:${url}`, { data: response, status });
  }

  mockError(method: string, url: string, error: { status: number; message: string }): void {
    this.responses.set(`${method}:${url}`, {
      data: { message: error.message },
      status: error.status,
    });
  }

  reset(): void {
    this.responses.clear();
  }

  private getResponse(method: string, url: string): MockResponse {
    const key = `${method}:${url}`;
    const response = this.responses.get(key);

    if (!response) {
      throw new Error(`No mock found for ${key}`);
    }

    if (response.status >= 400) {
      const error = new Error(response.data?.message || 'Request failed');
      (error as any).response = { status: response.status, data: response.data };
      throw error;
    }

    return response;
  }

  get = jest.fn(async (url: string) => this.getResponse('GET', url));
  post = jest.fn(async (url: string) => this.getResponse('POST', url));
  put = jest.fn(async (url: string) => this.getResponse('PUT', url));
  patch = jest.fn(async (url: string) => this.getResponse('PATCH', url));
  delete = jest.fn(async (url: string) => this.getResponse('DELETE', url));
}

export const mockApiClient = new MockApiClient();

export const mockClient = {
  get: mockApiClient.get,
  post: mockApiClient.post,
  put: mockApiClient.put,
  patch: mockApiClient.patch,
  delete: mockApiClient.delete,
  interceptors: {
    request: { use: jest.fn() },
    response: { use: jest.fn() },
  },
} as unknown as AxiosInstance;

// Mock the client module
jest.mock('@/core/services/api/client', () => ({
  client: mockClient,
}));
```

### Navigation Mock

```tsx
// src/__tests__/mocks/navigation.ts
const mockPush = jest.fn();
const mockReplace = jest.fn();
const mockBack = jest.fn();
const mockSetParams = jest.fn();

export const mockRouter = {
  useRouter: () => ({
    push: mockPush,
    replace: mockReplace,
    back: mockBack,
    setParams: mockSetParams,
  }),

  useLocalSearchParams: jest.fn(() => ({})),

  useSegments: jest.fn(() => []),

  usePathname: jest.fn(() => '/'),

  Link: ({ children, href, ...props }: any) => children,

  Redirect: ({ href }: { href: string }) => null,

  Stack: {
    Screen: ({ children }: any) => children,
  },

  Tabs: {
    Screen: ({ children }: any) => children,
  },

  Slot: ({ children }: any) => children,

  // Test helpers
  __mocks: {
    push: mockPush,
    replace: mockReplace,
    back: mockBack,
    setParams: mockSetParams,
    reset: () => {
      mockPush.mockClear();
      mockReplace.mockClear();
      mockBack.mockClear();
      mockSetParams.mockClear();
    },
  },
};
```

### Store Mock

```tsx
// src/__tests__/mocks/stores.ts
import { create } from 'zustand';

// Create resettable store for testing
type StoreState<T> = T & { __reset: () => void };

export function createTestStore<T extends object>(
  initialState: T
): StoreState<T> {
  const store = create<StoreState<T>>()((set) => ({
    ...initialState,
    __reset: () => set(initialState as StoreState<T>),
  }));

  return store.getState();
}

// Mock auth store
export const mockAuthStore = {
  token: null as string | null,
  status: 'idle' as 'idle' | 'signIn' | 'signOut',
  signIn: jest.fn((token: string) => {
    mockAuthStore.token = token;
    mockAuthStore.status = 'signIn';
  }),
  signOut: jest.fn(() => {
    mockAuthStore.token = null;
    mockAuthStore.status = 'signOut';
  }),
  reset: () => {
    mockAuthStore.token = null;
    mockAuthStore.status = 'idle';
    mockAuthStore.signIn.mockClear();
    mockAuthStore.signOut.mockClear();
  },
};

// Mock network store
export const mockNetworkStore = {
  status: 'online' as 'online' | 'offline' | 'unknown',
  setStatus: jest.fn((status: 'online' | 'offline' | 'unknown') => {
    mockNetworkStore.status = status;
  }),
  reset: () => {
    mockNetworkStore.status = 'online';
    mockNetworkStore.setStatus.mockClear();
  },
};

jest.mock('@/store', () => ({
  useAuthStore: Object.assign(
    (selector?: (state: typeof mockAuthStore) => any) =>
      selector ? selector(mockAuthStore) : mockAuthStore,
    {
      getState: () => mockAuthStore,
      use: {
        token: () => mockAuthStore.token,
        status: () => mockAuthStore.status,
      },
    }
  ),
  useNetworkStore: Object.assign(
    (selector?: (state: typeof mockNetworkStore) => any) =>
      selector ? selector(mockNetworkStore) : mockNetworkStore,
    {
      getState: () => mockNetworkStore,
      use: {
        status: () => mockNetworkStore.status,
      },
    }
  ),
}));
```

### i18n Mock

```tsx
// src/__tests__/mocks/i18n.ts
const t = jest.fn((key: string, options?: Record<string, unknown>) => {
  if (options?.count !== undefined) {
    return `${key}:${options.count}`;
  }
  return key;
});

export const mockI18n = {
  useTranslation: (ns?: string | string[]) => ({
    t,
    i18n: {
      language: 'en',
      changeLanguage: jest.fn(),
    },
  }),

  // Direct export
  t,

  // Reset for tests
  __reset: () => {
    t.mockClear();
  },
};
```

### Storage Mock

```tsx
// src/__tests__/mocks/async-storage.ts
const storage = new Map<string, string>();

export const mockMMKV = {
  MMKV: jest.fn().mockImplementation(() => ({
    set: jest.fn((key: string, value: string) => {
      storage.set(key, value);
    }),
    getString: jest.fn((key: string) => storage.get(key)),
    delete: jest.fn((key: string) => storage.delete(key)),
    contains: jest.fn((key: string) => storage.has(key)),
    clearAll: jest.fn(() => storage.clear()),
  })),

  // Test helper
  __storage: storage,
  __reset: () => storage.clear(),
};
```

## Test Utilities

### Custom Render

```tsx
// src/__tests__/utils/render.tsx
import { render as rtlRender, type RenderOptions } from '@testing-library/react-native';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import type { ReactElement, ReactNode } from 'react';

interface WrapperProps {
  children: ReactNode;
}

interface CustomRenderOptions extends Omit<RenderOptions, 'wrapper'> {
  // Add custom options here
  initialRoute?: string;
  preloadedState?: Record<string, unknown>;
}

function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
        gcTime: 0,
      },
      mutations: {
        retry: false,
      },
    },
  });
}

export function render(
  ui: ReactElement,
  options: CustomRenderOptions = {}
) {
  const { initialRoute, preloadedState, ...renderOptions } = options;
  const queryClient = createTestQueryClient();

  function Wrapper({ children }: WrapperProps) {
    return (
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    );
  }

  return {
    ...rtlRender(ui, { wrapper: Wrapper, ...renderOptions }),
    queryClient,
  };
}

// Re-export everything from testing-library
export * from '@testing-library/react-native';
export { render };
```

### Test Data Factories

```tsx
// src/__tests__/utils/factories.ts
import { v4 as uuid } from 'uuid';

// User factory
export interface UserData {
  id: string;
  email: string;
  name: string;
  createdAt: string;
}

export function createUser(overrides: Partial<UserData> = {}): UserData {
  return {
    id: uuid(),
    email: `user-${Date.now()}@test.com`,
    name: 'Test User',
    createdAt: new Date().toISOString(),
    ...overrides,
  };
}

// Post factory
export interface PostData {
  id: string;
  title: string;
  body: string;
  userId: string;
  createdAt: string;
  updatedAt: string;
}

export function createPost(overrides: Partial<PostData> = {}): PostData {
  return {
    id: uuid(),
    title: 'Test Post',
    body: 'Test post body content',
    userId: uuid(),
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
    ...overrides,
  };
}

// Create multiple items
export function createMany<T>(
  factory: (overrides?: Partial<T>) => T,
  count: number,
  overrides?: Partial<T>
): T[] {
  return Array.from({ length: count }, () => factory(overrides));
}

// Usage:
// createMany(createPost, 5)
// createMany(createUser, 3, { role: 'admin' })
```

### Custom Matchers

```tsx
// src/__tests__/utils/matchers.ts
import { expect } from '@jest/globals';

expect.extend({
  toBeDisabled(received) {
    const pass = received.props.disabled === true ||
      received.props.accessibilityState?.disabled === true;

    return {
      pass,
      message: () =>
        pass
          ? `expected element not to be disabled`
          : `expected element to be disabled`,
    };
  },

  toHaveAccessibilityLabel(received, expected) {
    const label = received.props.accessibilityLabel;
    const pass = label === expected;

    return {
      pass,
      message: () =>
        pass
          ? `expected element not to have accessibility label "${expected}"`
          : `expected element to have accessibility label "${expected}", but got "${label}"`,
    };
  },
});

declare global {
  namespace jest {
    interface Matchers<R> {
      toBeDisabled(): R;
      toHaveAccessibilityLabel(expected: string): R;
    }
  }
}
```

## Component Testing

### Basic Component Test

```tsx
// src/features/auth/components/__tests__/login-form.test.tsx
import { render, screen, fireEvent, waitFor } from '@/__tests__/utils/render';
import { LoginForm } from '../widgets/login-form';

describe('LoginForm', () => {
  const mockOnSubmit = jest.fn();

  beforeEach(() => {
    mockOnSubmit.mockClear();
  });

  it('renders all form fields', () => {
    render(<LoginForm onSubmit={mockOnSubmit} />);

    expect(screen.getByTestId('email-input')).toBeTruthy();
    expect(screen.getByTestId('password-input')).toBeTruthy();
    expect(screen.getByTestId('login-button')).toBeTruthy();
  });

  it('shows validation errors for empty fields', async () => {
    render(<LoginForm onSubmit={mockOnSubmit} />);

    fireEvent.press(screen.getByTestId('login-button'));

    await waitFor(() => {
      expect(screen.getByText('errors.validation.required')).toBeTruthy();
    });

    expect(mockOnSubmit).not.toHaveBeenCalled();
  });

  it('shows error for invalid email', async () => {
    render(<LoginForm onSubmit={mockOnSubmit} />);

    fireEvent.changeText(screen.getByTestId('email-input'), 'invalid-email');
    fireEvent.changeText(screen.getByTestId('password-input'), 'password123');
    fireEvent.press(screen.getByTestId('login-button'));

    await waitFor(() => {
      expect(screen.getByText('errors.validation.email')).toBeTruthy();
    });
  });

  it('calls onSubmit with valid data', async () => {
    render(<LoginForm onSubmit={mockOnSubmit} />);

    fireEvent.changeText(screen.getByTestId('email-input'), 'test@example.com');
    fireEvent.changeText(screen.getByTestId('password-input'), 'password123');
    fireEvent.press(screen.getByTestId('login-button'));

    await waitFor(() => {
      expect(mockOnSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123',
      });
    });
  });

  it('disables submit button while loading', () => {
    render(<LoginForm onSubmit={mockOnSubmit} isLoading />);

    const button = screen.getByTestId('login-button');
    expect(button.props.disabled).toBe(true);
  });
});
```

### Component with API Hook

```tsx
// src/features/posts/components/__tests__/post-list.test.tsx
import { render, screen, waitFor } from '@/__tests__/utils/render';
import { mockApiClient } from '@/__tests__/mocks/api';
import { createMany, createPost } from '@/__tests__/utils/factories';
import { PostList } from '../widgets/post-list';

describe('PostList', () => {
  beforeEach(() => {
    mockApiClient.reset();
  });

  it('renders loading state initially', () => {
    mockApiClient.mockGet('/posts', []);

    render(<PostList />);

    expect(screen.getByTestId('loading-skeleton')).toBeTruthy();
  });

  it('renders posts after loading', async () => {
    const posts = createMany(createPost, 3);
    mockApiClient.mockGet('/posts', posts);

    render(<PostList />);

    await waitFor(() => {
      expect(screen.getByText(posts[0].title)).toBeTruthy();
      expect(screen.getByText(posts[1].title)).toBeTruthy();
      expect(screen.getByText(posts[2].title)).toBeTruthy();
    });
  });

  it('renders empty state when no posts', async () => {
    mockApiClient.mockGet('/posts', []);

    render(<PostList />);

    await waitFor(() => {
      expect(screen.getByText('common:empty.no_posts')).toBeTruthy();
    });
  });

  it('renders error state on API failure', async () => {
    mockApiClient.mockError('GET', '/posts', {
      status: 500,
      message: 'Server error',
    });

    render(<PostList />);

    await waitFor(() => {
      expect(screen.getByText('errors.network.server')).toBeTruthy();
    });
  });
});
```

## Hook Testing

### Custom Hook Test

```tsx
// src/features/auth/hooks/__tests__/use-auth-state.test.ts
import { renderHook, act, waitFor } from '@testing-library/react-native';
import { mockAuthStore } from '@/__tests__/mocks/stores';
import { useAuthState } from '../use-auth-state';

describe('useAuthState', () => {
  beforeEach(() => {
    mockAuthStore.reset();
  });

  it('returns unauthenticated state initially', () => {
    const { result } = renderHook(() => useAuthState());

    expect(result.current.isAuthenticated).toBe(false);
    expect(result.current.user).toBeNull();
  });

  it('returns authenticated state after sign in', () => {
    mockAuthStore.signIn('test-token');

    const { result } = renderHook(() => useAuthState());

    expect(result.current.isAuthenticated).toBe(true);
  });

  it('returns unauthenticated state after sign out', () => {
    mockAuthStore.signIn('test-token');
    mockAuthStore.signOut();

    const { result } = renderHook(() => useAuthState());

    expect(result.current.isAuthenticated).toBe(false);
  });
});
```

### API Hook Test

```tsx
// src/features/posts/api/__tests__/use-create-post.test.tsx
import { renderHook, waitFor } from '@testing-library/react-native';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { mockApiClient } from '@/__tests__/mocks/api';
import { createPost } from '@/__tests__/utils/factories';
import { useCreatePost } from '../use-create-post';

function createWrapper() {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false }, mutations: { retry: false } },
  });

  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}

describe('useCreatePost', () => {
  beforeEach(() => {
    mockApiClient.reset();
  });

  it('creates a post successfully', async () => {
    const newPost = createPost({ title: 'New Post' });
    mockApiClient.mockPost('/posts', newPost);

    const { result } = renderHook(() => useCreatePost(), {
      wrapper: createWrapper(),
    });

    result.current.mutate({ title: 'New Post', body: 'Post body' });

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });

    expect(result.current.data).toEqual(newPost);
  });

  it('handles error correctly', async () => {
    mockApiClient.mockError('POST', '/posts', {
      status: 400,
      message: 'Validation error',
    });

    const { result } = renderHook(() => useCreatePost(), {
      wrapper: createWrapper(),
    });

    result.current.mutate({ title: '', body: '' });

    await waitFor(() => {
      expect(result.current.isError).toBe(true);
    });

    expect(result.current.error?.message).toBe('Validation error');
  });
});
```

## Integration Testing

### Screen Integration Test

```tsx
// src/features/auth/__tests__/login-screen.integration.test.tsx
import { render, screen, fireEvent, waitFor } from '@/__tests__/utils/render';
import { mockApiClient } from '@/__tests__/mocks/api';
import { mockRouter } from '@/__tests__/mocks/navigation';
import { mockAuthStore } from '@/__tests__/mocks/stores';
import { LoginScreen } from '../components/screens/login-screen';

describe('LoginScreen Integration', () => {
  beforeEach(() => {
    mockApiClient.reset();
    mockRouter.__mocks.reset();
    mockAuthStore.reset();
  });

  it('completes full login flow', async () => {
    // Setup API response
    mockApiClient.mockPost('/auth/login', {
      token: { access: 'test-token', refresh: 'refresh-token' },
      user: { id: '1', email: 'test@example.com' },
    });

    render(<LoginScreen />);

    // Fill form
    fireEvent.changeText(
      screen.getByTestId('email-input'),
      'test@example.com'
    );
    fireEvent.changeText(
      screen.getByTestId('password-input'),
      'password123'
    );

    // Submit
    fireEvent.press(screen.getByTestId('login-button'));

    // Verify loading state
    expect(screen.getByTestId('login-button').props.disabled).toBe(true);

    // Verify success
    await waitFor(() => {
      expect(mockAuthStore.signIn).toHaveBeenCalledWith({
        access: 'test-token',
        refresh: 'refresh-token',
      });
    });

    // Verify navigation
    expect(mockRouter.__mocks.replace).toHaveBeenCalledWith('/');
  });

  it('shows error message on login failure', async () => {
    mockApiClient.mockError('POST', '/auth/login', {
      status: 401,
      message: 'Invalid credentials',
    });

    render(<LoginScreen />);

    fireEvent.changeText(
      screen.getByTestId('email-input'),
      'test@example.com'
    );
    fireEvent.changeText(
      screen.getByTestId('password-input'),
      'wrongpassword'
    );
    fireEvent.press(screen.getByTestId('login-button'));

    await waitFor(() => {
      expect(screen.getByText('errors.auth.invalid_credentials')).toBeTruthy();
    });

    expect(mockAuthStore.signIn).not.toHaveBeenCalled();
    expect(mockRouter.__mocks.replace).not.toHaveBeenCalled();
  });
});
```

## Test Commands

```bash
# Run all tests
pnpm test

# Run tests in watch mode
pnpm test --watch

# Run tests for specific file
pnpm test login-form.test.tsx

# Run tests matching pattern
pnpm test --testPathPattern="auth"

# Run with coverage
pnpm test --coverage

# Update snapshots
pnpm test -u
```

## Best Practices

### 1. Test IDs Convention

```tsx
// Pattern: [component]-[element]
testID="login-button"
testID="email-input"
testID="email-input-error"
testID="post-list"
testID="post-card-123"  // With ID for dynamic elements
```

### 2. Arrange-Act-Assert Pattern

```tsx
it('should update user profile', async () => {
  // Arrange
  const user = createUser();
  mockApiClient.mockPut(`/users/${user.id}`, { ...user, name: 'Updated' });

  render(<ProfileForm user={user} />);

  // Act
  fireEvent.changeText(screen.getByTestId('name-input'), 'Updated');
  fireEvent.press(screen.getByTestId('save-button'));

  // Assert
  await waitFor(() => {
    expect(screen.getByText('Updated')).toBeTruthy();
  });
});
```

### 3. Avoid Implementation Details

```tsx
// ❌ Bad: Testing implementation
expect(component.state.isLoading).toBe(true);

// ✅ Good: Testing behavior
expect(screen.getByTestId('loading-spinner')).toBeTruthy();
```

### 4. Reset Mocks Between Tests

```tsx
beforeEach(() => {
  mockApiClient.reset();
  mockAuthStore.reset();
  mockRouter.__mocks.reset();
});
```

### 5. Use Factories for Test Data

```tsx
// ❌ Bad: Hardcoded data
const post = { id: '1', title: 'Test', body: 'Body' };

// ✅ Good: Use factories
const post = createPost({ title: 'Custom Title' });
```
