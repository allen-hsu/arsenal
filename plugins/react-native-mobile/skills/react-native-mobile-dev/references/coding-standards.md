# Coding Standards

## File & Folder Naming

### Kebab-case Convention

All files and folders use `kebab-case`:

```
# Good
login-form.tsx
use-auth.ts
api-client.ts
user-profile/

# Bad
LoginForm.tsx
useAuth.ts
apiClient.ts
UserProfile/
```

**Exception**: Native platform directories (`/android`, `/ios`) are excluded.

## Import Organization

### Import Order

```typescript
// 1. External packages
import { useQuery } from '@tanstack/react-query';
import { View } from 'react-native';

// 2. Internal aliases (@/) - configure with --import-alias
import { Button } from '@/shared/components/ui';
import { useAuth } from '@/features/auth';
import { client } from '@/core/services/api';

// 3. Relative imports
import { formatDate } from './utils';
import type { PostProps } from './types';
```

### Type Imports

Always use `import type` for type-only imports:

```typescript
// Good
import type { Post } from './types';
import { type User, fetchUser } from './api';

// Bad
import { Post } from './types';  // When Post is only a type
```

## Architecture Dependency Rules

### Layer Dependencies

```
app/      → Can import from: features/, shared/, core/, store/
features/ → Can import from: shared/, core/, store/
shared/   → Can import from: core/ only
core/     → No dependencies (base layer)
store/    → Can import from: core/, shared/
```

### Feature Isolation

Features must NOT import from each other:

```typescript
// ❌ WRONG: Cross-feature import
// In features/transactions/
import { useAuth } from '@/features/auth';

// ✅ CORRECT: Use global store
import { useAuthStore } from '@/store';

// ✅ CORRECT: Use shared components
import { Button } from '@/shared/components/ui';
```

### ESLint Configuration for Dependencies

```javascript
// .eslintrc.js
module.exports = {
  rules: {
    'import/no-restricted-paths': [
      'error',
      {
        zones: [
          // core/ cannot import from other layers
          {
            target: './src/core',
            from: './src/features',
            message: 'core/ cannot import from features/',
          },
          {
            target: './src/core',
            from: './src/shared',
            message: 'core/ cannot import from shared/',
          },
          // shared/ cannot import from features/
          {
            target: './src/shared',
            from: './src/features',
            message: 'shared/ cannot import from features/',
          },
          // Prevent cross-feature imports
          {
            target: './src/features/auth',
            from: './src/features/!(auth)/**/*',
            message: 'Features cannot import from other features',
          },
          // Add more feature restrictions as needed
        ],
      },
    ],
  },
};
```

## Function Guidelines

### Parameter Limits

Maximum **3 parameters** per function. Use objects for more:

```typescript
// Good
function createUser(options: CreateUserOptions) { }

interface CreateUserOptions {
  name: string;
  email: string;
  role: string;
  department: string;
}

// Bad
function createUser(name: string, email: string, role: string, department: string) { }
```

### Function Length

Maximum **70 lines** per function. Extract logic into smaller functions:

```typescript
// Good
function processOrder(order: Order) {
  validateOrder(order);
  const total = calculateTotal(order);
  const shipping = calculateShipping(order);
  return submitOrder({ ...order, total, shipping });
}

// Bad: 100+ line monolithic function
```

## Export Conventions

### Prefer Named Exports

```typescript
// Good
export const Button = () => { };
export function useAuth() { }

// Avoid
export default Button;
```

### Barrel Exports

Use `index.ts` for clean imports:

```typescript
// shared/components/ui/index.ts
export * from './button';
export * from './text';
export * from './input';

// Usage
import { Button, Text, Input } from '@/shared/components/ui';
```

## TypeScript Conventions

### Strict Mode

Always enable strict TypeScript:

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

### Explicit Return Types

For exported functions, prefer explicit return types:

```typescript
// Good
export function useAuth(): AuthState { }

// OK for simple components
export const Button = ({ label }: ButtonProps) => { };
```

### Avoid `any`

Use `unknown` and type guards instead:

```typescript
// Good
function processData(data: unknown) {
  if (isValidData(data)) {
    // data is now typed
  }
}

// Bad
function processData(data: any) { }
```

## Component Conventions

### Component Structure

```typescript
// 1. Imports
import { View, Text } from 'react-native';
import type { ComponentProps } from './types';

// 2. Types/Interfaces
interface ButtonProps {
  label: string;
  onPress: () => void;
}

// 3. Styled variants (if using tailwind-variants)
const buttonStyles = tv({
  base: 'rounded-xl px-4 py-3',
  variants: { }
});

// 4. Component
export const Button = ({ label, onPress }: ButtonProps) => {
  // 4a. Hooks
  const [isPressed, setIsPressed] = useState(false);

  // 4b. Derived state / memoization
  const styles = useMemo(() => buttonStyles({ pressed: isPressed }), [isPressed]);

  // 4c. Handlers
  const handlePress = useCallback(() => {
    setIsPressed(true);
    onPress();
  }, [onPress]);

  // 4d. Render
  return (
    <Pressable onPress={handlePress} className={styles}>
      <Text>{label}</Text>
    </Pressable>
  );
};
```

### Prop Naming

```typescript
// Event handlers: on[Event]
onPress, onSubmit, onChange

// Boolean props: is/has/should prefix
isDisabled, hasError, shouldValidate

// Render props: render[Thing]
renderItem, renderHeader
```

## i18n Conventions

### Translation Key Naming

Use hierarchical, descriptive keys:

```json
{
  "feature": {
    "screen": {
      "element": "Text"
    }
  }
}
```

Example:

```json
{
  "auth": {
    "login": {
      "title": "Welcome Back",
      "email_label": "Email",
      "submit": "Sign In"
    }
  }
}
```

### Namespace Usage

```typescript
// Use feature-specific namespace
const { t } = useTranslation('auth');
t('login.title');

// For common strings, use common namespace
const { t: tCommon } = useTranslation('common');
tCommon('buttons.submit');
```

### No Hardcoded Strings

```typescript
// ❌ Bad
<Text>Welcome back!</Text>
<Button label="Submit" />

// ✅ Good
<Text>{t('auth:login.title')}</Text>
<Button label={t('common:buttons.submit')} />
```

## Testing Conventions

### Test File Location

Place tests next to source files or in `__tests__/` directory:

```
features/auth/
├── components/
│   └── widgets/
│       └── login-form.tsx
└── __tests__/
    └── components/
        └── login-form.test.tsx
```

### Test Naming

```typescript
describe('LoginForm', () => {
  it('should validate email format', () => { });
  it('should call onSubmit with form data', () => { });
  it('should show error message for invalid input', () => { });
});
```

### Test IDs

Use consistent `testID` naming:

```typescript
// Pattern: [component]-[element]
testID="login-button"
testID="email-input"
testID="error-message"

// With dynamic IDs
testID={`post-card-${post.id}`}

// In component
<Button testID="login-button" />
<Input testID="email-input" />
```

### Test File Naming

```
[component-name].test.tsx     # Component tests
[hook-name].test.ts           # Hook tests
[function-name].test.ts       # Utility tests
```

## Error Handling Conventions

### API Error Handling

Always use the centralized error handler:

```typescript
// Good
import { handleApiError } from '@/core/services/api/error-handler';

async function fetchData() {
  try {
    const { data } = await client.get('/endpoint');
    return data;
  } catch (error) {
    throw handleApiError(error as AxiosError);
  }
}

// Bad
async function fetchData() {
  const { data } = await client.get('/endpoint'); // Unhandled error
  return data;
}
```

### Error Display

Use translated error messages:

```typescript
const { t } = useTranslation('errors');

// Display error
{error && <Text>{t(`network.${error.code}`)}</Text>}
```

## ESLint Configuration Summary

```javascript
module.exports = {
  rules: {
    // Naming
    'unicorn/filename-case': ['error', { case: 'kebabCase' }],

    // Functions
    'max-params': ['error', 3],
    'max-lines-per-function': ['error', 70],

    // Imports
    '@typescript-eslint/consistent-type-imports': ['warn', {
      prefer: 'type-imports',
      fixStyle: 'inline-type-imports',
    }],
    'simple-import-sort/imports': 'error',
    'simple-import-sort/exports': 'error',

    // Unused code
    'unused-imports/no-unused-imports': 'error',
    'unused-imports/no-unused-vars': ['error', {
      argsIgnorePattern: '^_',
    }],

    // Cycles
    'import/no-cycle': ['error', { maxDepth: '∞' }],

    // i18n (if using eslint-plugin-i18next)
    'i18next/no-literal-string': ['warn', {
      markupOnly: true,
      ignoreAttribute: ['testID', 'className', 'accessibilityRole'],
    }],
  },
};
```
