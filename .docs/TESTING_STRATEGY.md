# Testing Strategy
## Family House Management Application

**Last Updated:** November 2025
**Testing Philosophy:** Test early, test often, test at the right level

---

## Testing Pyramid

```
        /\
       /E2E\          ← 10% (Few, slow, expensive)
      /______\
     /  API   \       ← 20% (Integration tests)
    /__________\
   /   Unit     \     ← 70% (Many, fast, cheap)
  /______________\
```

**Target Coverage:**
- Unit Tests: 80%+ coverage
- Integration Tests: Critical user flows
- E2E Tests: Core happy paths only

---

## Test Types

### 1. Unit Tests
**What:** Test individual functions/components in isolation
**Tools:** Jest (backend), Vitest (frontend)
**Speed:** < 1 second per test
**When:** Write with every new function/component

### 2. Integration Tests
**What:** Test multiple units working together
**Tools:** Jest + Supertest (API), React Testing Library
**Speed:** < 5 seconds per test
**When:** Write for critical workflows

### 3. End-to-End (E2E) Tests
**What:** Test full user journeys in browser
**Tools:** Playwright / Cypress
**Speed:** 10-60 seconds per test
**When:** Write for critical user paths only

### 4. Component Tests
**What:** Test React components in isolation
**Tools:** React Testing Library + Vitest
**Speed:** 1-3 seconds per test
**When:** Write for reusable components

---

## Backend Testing

### Unit Tests

**Test Structure:**
```typescript
// server/tests/unit/services/UserService.test.ts
import { UserService } from '@/services/UserService';
import { prismaMock } from '../mocks/prisma';

describe('UserService', () => {
  let userService: UserService;

  beforeEach(() => {
    userService = new UserService();
    jest.clearAllMocks();
  });

  describe('getUserById', () => {
    it('should return user when found', async () => {
      // Arrange
      const mockUser = {
        id: 'user-123',
        email: 'test@example.com',
        name: 'Test User',
        role: 'ADULT',
        familyId: 'fam-456'
      };

      prismaMock.user.findUnique.mockResolvedValue(mockUser);

      // Act
      const result = await userService.getUserById('user-123');

      // Assert
      expect(result).toEqual(mockUser);
      expect(prismaMock.user.findUnique).toHaveBeenCalledWith({
        where: { id: 'user-123' }
      });
    });

    it('should return null when user not found', async () => {
      prismaMock.user.findUnique.mockResolvedValue(null);

      const result = await userService.getUserById('nonexistent');

      expect(result).toBeNull();
    });

    it('should throw error on database failure', async () => {
      prismaMock.user.findUnique.mockRejectedValue(
        new Error('Database connection failed')
      );

      await expect(userService.getUserById('user-123')).rejects.toThrow(
        'Database connection failed'
      );
    });
  });

  describe('createUser', () => {
    it('should hash password before storing', async () => {
      const userData = {
        email: 'new@example.com',
        password: 'PlainPassword123!',
        name: 'New User',
        familyId: 'fam-456'
      };

      prismaMock.user.findUnique.mockResolvedValue(null); // Email available
      prismaMock.user.create.mockResolvedValue({
        id: 'user-new',
        email: userData.email,
        name: userData.name,
        passwordHash: 'hashed',
        role: 'ADULT',
        familyId: userData.familyId
      });

      const result = await userService.createUser(userData);

      expect(result.passwordHash).not.toBe(userData.password);
      expect(result.email).toBe(userData.email);
    });

    it('should throw error if email already exists', async () => {
      prismaMock.user.findUnique.mockResolvedValue({
        id: 'existing-user',
        email: 'existing@example.com'
      });

      await expect(
        userService.createUser({
          email: 'existing@example.com',
          password: 'pass',
          name: 'name'
        })
      ).rejects.toThrow('Email already in use');
    });
  });
});
```

**Mocking Prisma:**
```typescript
// server/tests/mocks/prisma.ts
import { PrismaClient } from '@prisma/client';
import { mockDeep, mockReset, DeepMockProxy } from 'jest-mock-extended';

jest.mock('@/lib/prisma', () => ({
  __esModule: true,
  prisma: mockDeep<PrismaClient>(),
}));

beforeEach(() => {
  mockReset(prismaMock);
});

export const prismaMock = prisma as unknown as DeepMockProxy<PrismaClient>;
```

### Integration Tests (API)

```typescript
// server/tests/integration/api/auth.test.ts
import request from 'supertest';
import app from '@/app';
import { prisma } from '@/lib/prisma';
import bcrypt from 'bcrypt';

describe('POST /api/v1/auth/register', () => {
  beforeEach(async () => {
    // Clean database before each test
    await prisma.user.deleteMany();
    await prisma.family.deleteMany();
  });

  it('should register new user and create family', async () => {
    const response = await request(app)
      .post('/api/v1/auth/register')
      .send({
        email: 'newuser@example.com',
        password: 'SecurePass123!',
        name: 'New User',
        familyName: 'New Family'
      });

    expect(response.status).toBe(201);
    expect(response.body.success).toBe(true);
    expect(response.body.data.user.email).toBe('newuser@example.com');
    expect(response.body.data.family.name).toBe('New Family');

    // Verify user was created in database
    const user = await prisma.user.findUnique({
      where: { email: 'newuser@example.com' }
    });
    expect(user).toBeDefined();
  });

  it('should return 409 if email already exists', async () => {
    // Create existing user
    await prisma.family.create({
      data: {
        name: 'Existing Family',
        users: {
          create: {
            email: 'existing@example.com',
            passwordHash: await bcrypt.hash('password', 10),
            name: 'Existing User',
            role: 'ADMIN'
          }
        }
      }
    });

    const response = await request(app)
      .post('/api/v1/auth/register')
      .send({
        email: 'existing@example.com',
        password: 'Password123!',
        name: 'New User',
        familyName: 'New Family'
      });

    expect(response.status).toBe(409);
    expect(response.body.success).toBe(false);
  });

  it('should return 400 for invalid email', async () => {
    const response = await request(app)
      .post('/api/v1/auth/register')
      .send({
        email: 'invalid-email',
        password: 'Password123!',
        name: 'User',
        familyName: 'Family'
      });

    expect(response.status).toBe(400);
    expect(response.body.error.code).toBe('VALIDATION_ERROR');
  });
});

describe('POST /api/v1/auth/login', () => {
  let testUser: any;

  beforeEach(async () => {
    await prisma.user.deleteMany();
    await prisma.family.deleteMany();

    // Create test user
    const family = await prisma.family.create({
      data: { name: 'Test Family' }
    });

    testUser = await prisma.user.create({
      data: {
        email: 'test@example.com',
        passwordHash: await bcrypt.hash('TestPass123!', 10),
        name: 'Test User',
        role: 'ADMIN',
        familyId: family.id,
        emailVerified: true
      }
    });
  });

  it('should login with valid credentials', async () => {
    const response = await request(app)
      .post('/api/v1/auth/login')
      .send({
        email: 'test@example.com',
        password: 'TestPass123!'
      });

    expect(response.status).toBe(200);
    expect(response.body.success).toBe(true);
    expect(response.body.data.accessToken).toBeDefined();
    expect(response.body.data.user.email).toBe('test@example.com');

    // Check refresh token cookie
    const cookies = response.headers['set-cookie'];
    expect(cookies).toBeDefined();
    expect(cookies[0]).toContain('refreshToken');
  });

  it('should return 401 for invalid password', async () => {
    const response = await request(app)
      .post('/api/v1/auth/login')
      .send({
        email: 'test@example.com',
        password: 'WrongPassword'
      });

    expect(response.status).toBe(401);
  });
});
```

### Background Job Testing

```typescript
// server/tests/unit/jobs/billReminders.test.ts
import { checkBillReminders } from '@/jobs/billReminders';
import { prismaMock } from '../mocks/prisma';
import { NotificationService } from '@/services/NotificationService';

jest.mock('@/services/NotificationService');

describe('Bill Reminders Job', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should send reminders for bills due in 7 days', async () => {
    const sevenDaysFromNow = new Date();
    sevenDaysFromNow.setDate(sevenDaysFromNow.getDate() + 7);

    const mockBills = [
      {
        id: 'bill-123',
        name: 'Electric Bill',
        amount: 150,
        dueDate: sevenDaysFromNow,
        familyId: 'fam-456',
        family: {
          users: [
            { id: 'user-123', email: 'user@example.com' }
          ]
        }
      }
    ];

    prismaMock.bill.findMany.mockResolvedValue(mockBills);

    await checkBillReminders();

    expect(NotificationService.sendBillReminder).toHaveBeenCalledWith({
      bill: mockBills[0],
      daysUntilDue: 7
    });
  });
});
```

---

## Frontend Testing

### Component Tests

```typescript
// client/tests/components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from '@/components/Button';

describe('Button', () => {
  it('should render with children text', () => {
    render(<Button>Click Me</Button>);

    expect(screen.getByText('Click Me')).toBeInTheDocument();
  });

  it('should call onClick when clicked', () => {
    const handleClick = jest.fn();

    render(<Button onClick={handleClick}>Click Me</Button>);

    fireEvent.click(screen.getByText('Click Me'));

    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('should apply variant className', () => {
    render(<Button variant="primary">Primary</Button>);

    const button = screen.getByText('Primary');
    expect(button).toHaveClass('btn-primary');
  });

  it('should be disabled when disabled prop is true', () => {
    render(<Button disabled>Disabled</Button>);

    const button = screen.getByText('Disabled');
    expect(button).toBeDisabled();
  });
});
```

### Hook Tests

```typescript
// client/tests/hooks/useAuth.test.ts
import { renderHook, act, waitFor } from '@testing-library/react';
import { useAuth } from '@/hooks/useAuth';

// Mock API calls
jest.mock('@/services/apiClient');

describe('useAuth', () => {
  it('should start with loading state', () => {
    const { result } = renderHook(() => useAuth());

    expect(result.current.loading).toBe(true);
    expect(result.current.user).toBeNull();
  });

  it('should login user successfully', async () => {
    const { result } = renderHook(() => useAuth());

    await act(async () => {
      await result.current.login('test@example.com', 'password');
    });

    await waitFor(() => {
      expect(result.current.user).toBeDefined();
      expect(result.current.user?.email).toBe('test@example.com');
      expect(result.current.loading).toBe(false);
    });
  });

  it('should handle login error', async () => {
    const { result } = renderHook(() => useAuth());

    // Mock API to return error
    jest.spyOn(apiClient, 'post').mockRejectedValue(
      new Error('Invalid credentials')
    );

    await act(async () => {
      await result.current.login('test@example.com', 'wrongpass');
    });

    await waitFor(() => {
      expect(result.current.error).toBe('Invalid credentials');
      expect(result.current.user).toBeNull();
    });
  });
});
```

### Integration Tests (User Flows)

```typescript
// client/tests/integration/LoginFlow.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import App from '@/App';
import { server } from '../mocks/server';
import { rest } from 'msw';

describe('Login Flow', () => {
  it('should login and redirect to dashboard', async () => {
    render(
      <BrowserRouter>
        <App />
      </BrowserRouter>
    );

    // Navigate to login page
    fireEvent.click(screen.getByText('Login'));

    // Fill in form
    fireEvent.change(screen.getByLabelText('Email'), {
      target: { value: 'test@example.com' }
    });
    fireEvent.change(screen.getByLabelText('Password'), {
      target: { value: 'password123' }
    });

    // Submit
    fireEvent.click(screen.getByRole('button', { name: 'Login' }));

    // Wait for redirect
    await waitFor(() => {
      expect(screen.getByText('Dashboard')).toBeInTheDocument();
    });
  });

  it('should show error for invalid credentials', async () => {
    // Mock API to return error
    server.use(
      rest.post('/api/v1/auth/login', (req, res, ctx) => {
        return res(
          ctx.status(401),
          ctx.json({
            success: false,
            error: { message: 'Invalid credentials' }
          })
        );
      })
    );

    render(
      <BrowserRouter>
        <App />
      </BrowserRouter>
    );

    fireEvent.click(screen.getByText('Login'));

    fireEvent.change(screen.getByLabelText('Email'), {
      target: { value: 'test@example.com' }
    });
    fireEvent.change(screen.getByLabelText('Password'), {
      target: { value: 'wrongpass' }
    });

    fireEvent.click(screen.getByRole('button', { name: 'Login' }));

    await waitFor(() => {
      expect(screen.getByText('Invalid credentials')).toBeInTheDocument();
    });
  });
});
```

### MSW (Mock Service Worker) Setup

```typescript
// client/tests/mocks/server.ts
import { setupServer } from 'msw/node';
import { rest } from 'msw';

export const handlers = [
  rest.post('/api/v1/auth/login', (req, res, ctx) => {
    return res(
      ctx.json({
        success: true,
        data: {
          accessToken: 'mock-token',
          user: {
            id: 'user-123',
            email: 'test@example.com',
            name: 'Test User'
          }
        }
      })
    );
  }),

  rest.get('/api/v1/calendar/events', (req, res, ctx) => {
    return res(
      ctx.json({
        success: true,
        data: []
      })
    );
  })
];

export const server = setupServer(...handlers);

// Setup/teardown
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

---

## E2E Testing

### Playwright Setup

```typescript
// e2e/tests/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test('should register new user', async ({ page }) => {
    await page.goto('http://localhost:5173');

    // Click register
    await page.click('text=Sign Up');

    // Fill form
    await page.fill('[name="email"]', 'newuser@example.com');
    await page.fill('[name="password"]', 'SecurePass123!');
    await page.fill('[name="name"]', 'New User');
    await page.fill('[name="familyName"]', 'New Family');

    // Submit
    await page.click('button[type="submit"]');

    // Verify success message
    await expect(page.locator('text=Verification email sent')).toBeVisible();
  });

  test('should login existing user', async ({ page }) => {
    await page.goto('http://localhost:5173/login');

    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'TestPass123!');

    await page.click('button[type="submit"]');

    // Verify redirect to dashboard
    await expect(page).toHaveURL('http://localhost:5173/dashboard');
    await expect(page.locator('text=Dashboard')).toBeVisible();
  });
});

test.describe('Calendar', () => {
  test.beforeEach(async ({ page }) => {
    // Login before each test
    await page.goto('http://localhost:5173/login');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'TestPass123!');
    await page.click('button[type="submit"]');
    await page.waitForURL('**/dashboard');
  });

  test('should create new event', async ({ page }) => {
    await page.goto('http://localhost:5173/calendar');

    // Click new event button
    await page.click('text=New Event');

    // Fill form
    await page.fill('[name="title"]', 'Doctor Appointment');
    await page.selectOption('[name="category"]', 'MEDICAL');
    await page.fill('[name="startTime"]', '2025-12-15T10:00');
    await page.fill('[name="endTime"]', '2025-12-15T11:00');

    // Submit
    await page.click('button:has-text("Save Event")');

    // Verify event appears on calendar
    await expect(page.locator('text=Doctor Appointment')).toBeVisible();
  });
});
```

### Playwright Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e/tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',

  use: {
    baseURL: 'http://localhost:5173',
    trace: 'on-first-retry',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],

  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
  },
});
```

---

## Test Data Management

### Factories

```typescript
// server/tests/factories/userFactory.ts
import { faker } from '@faker-js/faker';
import bcrypt from 'bcrypt';

export const userFactory = {
  build: (overrides = {}) => ({
    id: faker.string.uuid(),
    email: faker.internet.email(),
    passwordHash: bcrypt.hashSync('password', 10),
    name: faker.person.fullName(),
    role: 'ADULT',
    familyId: faker.string.uuid(),
    emailVerified: true,
    createdAt: new Date(),
    updatedAt: new Date(),
    ...overrides
  })
};

export const familyFactory = {
  build: (overrides = {}) => ({
    id: faker.string.uuid(),
    name: `${faker.person.lastName()} Family`,
    timezone: 'America/New_York',
    currency: 'USD',
    createdAt: new Date(),
    updatedAt: new Date(),
    ...overrides
  })
};
```

**Usage:**
```typescript
const testUser = userFactory.build({ email: 'specific@example.com' });
const testFamily = familyFactory.build({ name: 'Smith Family' });
```

---

## Test Coverage

### Measuring Coverage

**Backend:**
```bash
npm run test:coverage
```

**Frontend:**
```bash
npm run test:coverage
```

### Coverage Requirements

**Minimum Thresholds:**
```json
// jest.config.js
{
  "coverageThreshold": {
    "global": {
      "branches": 70,
      "functions": 80,
      "lines": 80,
      "statements": 80
    }
  }
}
```

### What to Prioritize for Coverage

1. ✅ **High Priority (Aim for 90%+)**
   - Business logic (services)
   - Utility functions
   - Authentication/authorization
   - Data validation
   - Critical user workflows

2. ⚠️ **Medium Priority (Aim for 70%+)**
   - Controllers/route handlers
   - React components
   - Custom hooks
   - API integrations

3. 🔵 **Lower Priority**
   - Simple getters/setters
   - Configuration files
   - Type definitions
   - Mock data

---

## Continuous Integration

### Test Pipeline

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  backend-tests:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: house_management_test
        ports:
          - 5432:5432

      redis:
        image: redis:7
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20

      - name: Install dependencies
        run: cd server && npm ci

      - name: Run migrations
        run: cd server && npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://postgres:testpass@localhost:5432/house_management_test

      - name: Run unit tests
        run: cd server && npm run test

      - name: Run integration tests
        run: cd server && npm run test:integration

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./server/coverage/lcov.info

  frontend-tests:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20

      - name: Install dependencies
        run: cd client && npm ci

      - name: Run tests
        run: cd client && npm run test

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./client/coverage/lcov.info

  e2e-tests:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright
        run: npx playwright install --with-deps

      - name: Run E2E tests
        run: npm run test:e2e

      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
```

---

## Best Practices

### General

1. **Write tests first (TDD)** for complex logic
2. **Keep tests fast** - unit tests < 1s, integration < 5s
3. **One assertion per test** (when possible)
4. **Test behavior, not implementation**
5. **Mock external dependencies** (APIs, databases)
6. **Use descriptive test names** - should read like documentation
7. **Clean up after tests** - reset database, clear mocks
8. **Don't test framework code** - trust React, Prisma, etc.
9. **Test edge cases** - null, empty, max values
10. **Keep tests isolated** - no dependencies between tests

### Anti-Patterns

❌ **Don't:**
- Test internal implementation details
- Have tests that depend on execution order
- Share state between tests
- Use production database for tests
- Have flaky tests that sometimes fail
- Test generated code (Prisma client)
- Write tests just to hit coverage

✅ **Do:**
- Test public interfaces
- Make tests independent
- Use test database
- Mock external services
- Fix failing tests immediately
- Focus on critical paths
- Write meaningful tests

---

**Follow this strategy to maintain high code quality and catch bugs early!**
