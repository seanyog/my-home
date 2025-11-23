# Coding Standards & Conventions
## Family House Management Application

**Last Updated:** November 2025
**Applies To:** All developers on the project

---

## General Principles

### SOLID Principles
- **S**ingle Responsibility: Each module/function does one thing well
- **O**pen/Closed: Open for extension, closed for modification
- **L**iskov Substitution: Subtypes must be substitutable for their base types
- **I**nterface Segregation: Many specific interfaces > one general interface
- **D**ependency Inversion: Depend on abstractions, not concretions

### DRY (Don't Repeat Yourself)
- Extract repeated code into reusable functions/components
- Use constants for magic numbers and repeated strings
- Create utility functions for common operations

### KISS (Keep It Simple, Stupid)
- Prefer simple solutions over clever ones
- Write code for readability, not brevity
- Avoid premature optimization

### YAGNI (You Aren't Gonna Need It)
- Don't build features "just in case"
- Implement only what's needed for current requirements
- Refactor when actual needs emerge

---

## TypeScript Standards

### Type Safety

**Always use TypeScript - Never `any`**

❌ **Bad:**
```typescript
const fetchData = async (url: string): Promise<any> => {
  const response = await fetch(url);
  return response.json();
};
```

✅ **Good:**
```typescript
interface UserResponse {
  id: string;
  name: string;
  email: string;
}

const fetchUser = async (url: string): Promise<UserResponse> => {
  const response = await fetch(url);
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  return response.json();
};
```

**Use `unknown` instead of `any` when type is truly unknown**

```typescript
const parseJson = (jsonString: string): unknown => {
  return JSON.parse(jsonString);
};

// Type guard to safely use the result
const isUser = (obj: unknown): obj is UserResponse => {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    'name' in obj &&
    'email' in obj
  );
};
```

### Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Variables | camelCase | `const userProfile = {}` |
| Constants | UPPER_SNAKE_CASE | `const MAX_RETRY_ATTEMPTS = 3` |
| Functions | camelCase | `function calculateTotal() {}` |
| Classes | PascalCase | `class UserService {}` |
| Interfaces | PascalCase | `interface UserData {}` |
| Types | PascalCase | `type UserId = string` |
| Enums | PascalCase | `enum UserRole {}` |
| Files (components) | PascalCase | `UserProfile.tsx` |
| Files (utilities) | camelCase | `dateUtils.ts` |
| Private properties | _camelCase | `private _cache = new Map()` |

### File Naming

```
client/src/
  components/
    UserProfile.tsx          // PascalCase for components
    Button.tsx
  hooks/
    useAuth.ts               // camelCase with 'use' prefix
    useCalendar.ts
  utils/
    dateUtils.ts             // camelCase for utilities
    apiClient.ts
  types/
    user.types.ts            // camelCase with .types suffix
    calendar.types.ts
  services/
    AuthService.ts           // PascalCase for service classes
    CalendarService.ts

server/src/
  api/
    routes/
      userRoutes.ts          // camelCase with descriptive suffix
      calendarRoutes.ts
    controllers/
      UserController.ts      // PascalCase for classes
      CalendarController.ts
  services/
    UserService.ts
    PlaidService.ts
  utils/
    encryption.ts            // camelCase
    validation.ts
```

### Interfaces vs Types

**Use Interfaces for:**
- Object shapes
- Extending/implementing
- Declaration merging (if needed)

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

interface Admin extends User {
  permissions: string[];
}
```

**Use Types for:**
- Unions and intersections
- Tuples
- Utility types
- Type aliases

```typescript
type UserId = string;
type UserRole = 'ADMIN' | 'ADULT' | 'TEEN' | 'CHILD';
type ApiResponse<T> = { success: true; data: T } | { success: false; error: string };
```

---

## React/Frontend Standards

### Component Structure

**Functional Components with Hooks (always)**

```typescript
// components/UserProfile.tsx
import { useState, useEffect } from 'react';
import type { User } from '@/types/user.types';

interface UserProfileProps {
  userId: string;
  onUpdate?: (user: User) => void;
}

export const UserProfile: React.FC<UserProfileProps> = ({ userId, onUpdate }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        setUser(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Unknown error');
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div className="user-profile">
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};
```

### Custom Hooks

```typescript
// hooks/useAuth.ts
import { useState, useEffect } from 'react';
import type { User } from '@/types/user.types';

interface UseAuthReturn {
  user: User | null;
  loading: boolean;
  error: string | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
}

export const useAuth = (): UseAuthReturn => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const login = async (email: string, password: string) => {
    // Implementation
  };

  const logout = async () => {
    // Implementation
  };

  return { user, loading, error, login, logout };
};
```

### Props Destructuring

✅ **Good:**
```typescript
const Button: React.FC<ButtonProps> = ({ children, onClick, variant = 'primary' }) => {
  return (
    <button onClick={onClick} className={`btn btn-${variant}`}>
      {children}
    </button>
  );
};
```

❌ **Bad:**
```typescript
const Button: React.FC<ButtonProps> = (props) => {
  return (
    <button onClick={props.onClick} className={`btn btn-${props.variant}`}>
      {props.children}
    </button>
  );
};
```

### Component Organization

```typescript
// 1. Imports (grouped)
import React, { useState, useEffect } from 'react';  // React
import { useNavigate } from 'react-router-dom';      // Third-party
import { Button } from '@/components/Button';         // Internal components
import { useAuth } from '@/hooks/useAuth';            // Internal hooks
import { formatDate } from '@/utils/dateUtils';       // Utilities
import type { User } from '@/types/user.types';       // Types
import './UserProfile.css';                           // Styles

// 2. Types/Interfaces (if component-specific)
interface UserProfileProps {
  userId: string;
}

// 3. Constants (if component-specific)
const DEFAULT_AVATAR = '/images/default-avatar.png';

// 4. Component
export const UserProfile: React.FC<UserProfileProps> = ({ userId }) => {
  // 4a. Hooks
  const navigate = useNavigate();
  const { user } = useAuth();

  // 4b. State
  const [isEditing, setIsEditing] = useState(false);

  // 4c. Effects
  useEffect(() => {
    // ...
  }, [userId]);

  // 4d. Event handlers
  const handleEdit = () => {
    setIsEditing(true);
  };

  // 4e. Render helpers (if needed)
  const renderAvatar = () => {
    return <img src={user?.avatarUrl || DEFAULT_AVATAR} alt={user?.name} />;
  };

  // 4f. Main render
  return (
    <div className="user-profile">
      {renderAvatar()}
      {/* ... */}
    </div>
  );
};

// 5. Sub-components (if tightly coupled)
const UserBadge: React.FC<{ role: string }> = ({ role }) => {
  return <span className="badge">{role}</span>;
};
```

### Conditional Rendering

✅ **Good:**
```typescript
// Simple condition
{isLoading && <Spinner />}

// Ternary for if/else
{user ? <UserProfile user={user} /> : <LoginPrompt />}

// Complex conditions - extract to function
const renderContent = () => {
  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!data) return <EmptyState />;
  return <DataDisplay data={data} />;
};

return <div>{renderContent()}</div>;
```

❌ **Avoid:**
```typescript
// Nested ternaries
{isLoading ? <Spinner /> : error ? <ErrorMessage /> : data ? <DataDisplay /> : <EmptyState />}
```

---

## Backend/Node.js Standards

### API Route Structure

```typescript
// server/src/api/routes/userRoutes.ts
import { Router } from 'express';
import { UserController } from '../controllers/UserController';
import { requireAuth, requireRole } from '@/middleware/auth';
import { validateRequest } from '@/middleware/validation';
import { updateUserSchema } from '@/validation/userSchemas';

const router = Router();
const userController = new UserController();

// Public routes
router.post('/register', validateRequest(registerSchema), userController.register);
router.post('/login', validateRequest(loginSchema), userController.login);

// Protected routes
router.get('/me', requireAuth, userController.getCurrentUser);
router.patch('/me', requireAuth, validateRequest(updateUserSchema), userController.updateProfile);

// Admin routes
router.get('/', requireAuth, requireRole('ADMIN'), userController.listUsers);
router.delete('/:id', requireAuth, requireRole('ADMIN'), userController.deleteUser);

export default router;
```

### Controller Pattern

```typescript
// server/src/api/controllers/UserController.ts
import { Request, Response, NextFunction } from 'express';
import { UserService } from '@/services/UserService';
import { ApiError } from '@/utils/ApiError';

export class UserController {
  private userService: UserService;

  constructor() {
    this.userService = new UserService();
  }

  getCurrentUser = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const userId = req.user!.id;  // Set by auth middleware
      const user = await this.userService.getUserById(userId);

      if (!user) {
        throw new ApiError(404, 'User not found');
      }

      res.json({
        success: true,
        data: user
      });
    } catch (error) {
      next(error);  // Pass to error handling middleware
    }
  };

  updateProfile = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const userId = req.user!.id;
      const updates = req.body;

      const user = await this.userService.updateUser(userId, updates);

      res.json({
        success: true,
        data: user
      });
    } catch (error) {
      next(error);
    }
  };
}
```

### Service Layer

```typescript
// server/src/services/UserService.ts
import { prisma } from '@/lib/prisma';
import type { User, Prisma } from '@prisma/client';
import { ApiError } from '@/utils/ApiError';
import bcrypt from 'bcrypt';

export class UserService {
  async getUserById(id: string): Promise<User | null> {
    return prisma.user.findUnique({
      where: { id },
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        avatarUrl: true,
        // Exclude passwordHash
      }
    });
  }

  async createUser(data: Prisma.UserCreateInput): Promise<User> {
    // Validate email doesn't exist
    const existing = await prisma.user.findUnique({
      where: { email: data.email }
    });

    if (existing) {
      throw new ApiError(409, 'Email already in use');
    }

    // Hash password
    const passwordHash = await bcrypt.hash(data.password, 10);

    // Create user
    return prisma.user.create({
      data: {
        ...data,
        passwordHash,
      }
    });
  }

  async updateUser(id: string, updates: Partial<User>): Promise<User> {
    return prisma.user.update({
      where: { id },
      data: updates
    });
  }
}
```

### Error Handling

**Custom Error Class:**
```typescript
// server/src/utils/ApiError.ts
export class ApiError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public code?: string
  ) {
    super(message);
    this.name = 'ApiError';
    Error.captureStackTrace(this, this.constructor);
  }
}
```

**Error Middleware:**
```typescript
// server/src/middleware/errorHandler.ts
import { Request, Response, NextFunction } from 'express';
import { ApiError } from '@/utils/ApiError';
import { Prisma } from '@prisma/client';

export const errorHandler = (
  error: Error,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  console.error('Error:', error);

  // Handle known API errors
  if (error instanceof ApiError) {
    return res.status(error.statusCode).json({
      success: false,
      error: {
        code: error.code || 'API_ERROR',
        message: error.message
      }
    });
  }

  // Handle Prisma errors
  if (error instanceof Prisma.PrismaClientKnownRequestError) {
    if (error.code === 'P2002') {
      return res.status(409).json({
        success: false,
        error: {
          code: 'DUPLICATE_ENTRY',
          message: 'Resource already exists'
        }
      });
    }
  }

  // Handle validation errors (from Zod)
  if (error.name === 'ZodError') {
    return res.status(400).json({
      success: false,
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Invalid request data',
        details: error.errors
      }
    });
  }

  // Default error
  res.status(500).json({
    success: false,
    error: {
      code: 'SERVER_ERROR',
      message: 'An unexpected error occurred'
    }
  });
};
```

### Async/Await Best Practices

✅ **Good:**
```typescript
const processUsers = async () => {
  try {
    const users = await fetchUsers();
    const processed = await Promise.all(
      users.map(user => processUser(user))
    );
    return processed;
  } catch (error) {
    logger.error('Failed to process users:', error);
    throw error;
  }
};
```

❌ **Bad:**
```typescript
const processUsers = async () => {
  const users = await fetchUsers();

  // Serial execution - slow!
  const processed = [];
  for (const user of users) {
    processed.push(await processUser(user));
  }

  return processed;
};
```

---

## Database/Prisma Standards

### Query Optimization

✅ **Good - Use includes:**
```typescript
const family = await prisma.family.findUnique({
  where: { id: familyId },
  include: {
    users: true,
    calendarEvents: {
      where: {
        startTime: { gte: new Date() }
      },
      take: 10
    }
  }
});
```

❌ **Bad - N+1 query:**
```typescript
const family = await prisma.family.findUnique({ where: { id: familyId } });
const users = await prisma.user.findMany({ where: { familyId } });
const events = await prisma.calendarEvent.findMany({ where: { familyId } });
```

### Transactions

```typescript
const createEventWithReminder = async (eventData, reminderData) => {
  return await prisma.$transaction(async (tx) => {
    const event = await tx.calendarEvent.create({
      data: eventData
    });

    await tx.notification.create({
      data: {
        ...reminderData,
        relatedEntityId: event.id
      }
    });

    return event;
  });
};
```

---

## Testing Standards

### Test Structure (AAA Pattern)

```typescript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create a new user with valid data', async () => {
      // Arrange
      const userData = {
        email: 'test@example.com',
        password: 'Test123!',
        name: 'Test User',
        familyId: 'family-123'
      };

      // Act
      const user = await userService.createUser(userData);

      // Assert
      expect(user).toBeDefined();
      expect(user.email).toBe(userData.email);
      expect(user.passwordHash).not.toBe(userData.password);
    });

    it('should throw error if email already exists', async () => {
      // Arrange
      const userData = {
        email: 'existing@example.com',
        password: 'Test123!',
        name: 'Test User'
      };

      await userService.createUser(userData);

      // Act & Assert
      await expect(userService.createUser(userData)).rejects.toThrow('Email already in use');
    });
  });
});
```

### Test Naming

```typescript
// Pattern: should [expected behavior] when [condition]
it('should return 401 when token is missing');
it('should create event when all required fields provided');
it('should send reminder email when bill is due in 7 days');
```

---

## Comments & Documentation

### When to Comment

✅ **Good reasons to comment:**
- Complex algorithms that aren't self-evident
- Business logic that requires context
- Workarounds for bugs in third-party libraries
- Public API documentation

✅ **Good comment:**
```typescript
/**
 * Calculates the next bill due date based on recurrence pattern.
 * Handles edge cases like:
 * - Month-end dates (e.g., January 31 → February 28)
 * - Leap years
 * - Daylight saving time transitions
 */
const calculateNextDueDate = (currentDate: Date, recurrence: Recurrence): Date => {
  // Implementation
};
```

❌ **Bad comment (states the obvious):**
```typescript
// Increment counter
counter++;

// Get user by ID
const user = await prisma.user.findUnique({ where: { id } });
```

### JSDoc for Public APIs

```typescript
/**
 * Fetches a user by their ID.
 *
 * @param userId - The unique identifier of the user
 * @returns The user object if found, null otherwise
 * @throws {ApiError} When userId is invalid format
 *
 * @example
 * ```typescript
 * const user = await getUserById('user-123');
 * ```
 */
export async function getUserById(userId: string): Promise<User | null> {
  // Implementation
}
```

---

## Git Commit Standards

### Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, semicolons, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks (dependencies, build, etc.)
- `perf`: Performance improvements

**Examples:**
```
feat(calendar): add Google Calendar sync integration

Implements OAuth flow and periodic sync for Google Calendar events.
Users can now import their Google Calendar events into the family calendar.

Closes #42
```

```
fix(auth): prevent race condition in token refresh

Adds mutex lock to prevent multiple simultaneous token refresh requests
when multiple API calls fail with 401 at the same time.

Fixes #78
```

```
refactor(api): extract validation middleware

Moves request validation logic from individual routes into reusable
middleware functions to reduce code duplication.
```

---

## Security Best Practices

### Never Log Sensitive Data

❌ **Bad:**
```typescript
console.log('User logged in:', { email, password, token });
```

✅ **Good:**
```typescript
logger.info('User logged in', { userId: user.id });
```

### Validate All Input

```typescript
import { z } from 'zod';

const createEventSchema = z.object({
  title: z.string().min(1).max(200),
  startTime: z.string().datetime(),
  endTime: z.string().datetime(),
  category: z.enum(['SCHOOL', 'ACTIVITIES', 'MEDICAL', 'OTHER'])
});

// In route handler
const validated = createEventSchema.parse(req.body);
```

### Use Parameterized Queries

Prisma handles this automatically, but if using raw SQL:

✅ **Good:**
```typescript
const users = await prisma.$queryRaw`
  SELECT * FROM users WHERE email = ${email}
`;
```

❌ **Bad (SQL injection vulnerability):**
```typescript
const users = await prisma.$queryRawUnsafe(
  `SELECT * FROM users WHERE email = '${email}'`
);
```

---

## Performance Best Practices

### Debounce User Input

```typescript
import { debounce } from 'lodash';

const SearchBar = () => {
  const handleSearch = debounce((query: string) => {
    // API call
  }, 300);

  return <input onChange={(e) => handleSearch(e.target.value)} />;
};
```

### Memoize Expensive Calculations

```typescript
import { useMemo } from 'react';

const ExpensiveComponent = ({ data }) => {
  const processedData = useMemo(() => {
    return complexCalculation(data);
  }, [data]);

  return <div>{processedData}</div>;
};
```

### Use React Query for Data Fetching

```typescript
import { useQuery } from '@tanstack/react-query';

const useCalendarEvents = (startDate: string, endDate: string) => {
  return useQuery({
    queryKey: ['events', startDate, endDate],
    queryFn: () => fetchEvents(startDate, endDate),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};
```

---

## Code Review Checklist

Before submitting a PR, ensure:

- [ ] Code follows all style guidelines
- [ ] All tests pass
- [ ] New code has tests (>80% coverage)
- [ ] No console.log statements (use logger)
- [ ] No commented-out code
- [ ] Type safety maintained (no `any`)
- [ ] Error handling implemented
- [ ] Loading states implemented (for UI)
- [ ] Edge cases considered
- [ ] Documentation updated (if needed)
- [ ] Commit messages follow convention
- [ ] No merge conflicts

---

**Follow these standards to maintain code quality and team productivity!**
