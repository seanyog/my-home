# Implementation Strategy 1: Monolithic Full-Stack Application

**Approach:** Single unified application with modular architecture
**Target:** Small to medium scale (up to 10,000 families)
**Development Timeline:** 6-9 months for MVP
**Team Size:** 4-6 developers

---

## Strategy Overview

Build a traditional monolithic web application where all components (frontend, backend, database) are tightly integrated but internally organized into clear modules. This approach prioritizes speed of development, ease of deployment, and simplicity of operations.

### Philosophy
- **Simplicity First:** Single codebase, single deployment unit
- **Faster Time to Market:** No microservice coordination overhead
- **Cost Effective:** Single server can handle significant load
- **Developer Friendly:** Easier debugging, simpler local development

---

## Technology Stack

### Frontend
```
Framework:          React 18+ with TypeScript
State Management:   Zustand (lightweight, simpler than Redux)
UI Library:         shadcn/ui + Tailwind CSS
Routing:            React Router v6
Data Fetching:      TanStack Query (React Query)
Forms:              React Hook Form + Zod validation
Charts:             Recharts / Chart.js
Calendar UI:        FullCalendar.io
Build Tool:         Vite
Testing:            Vitest + React Testing Library
```

**Why React?**
- Largest ecosystem and community
- Excellent TypeScript support
- Rich component libraries
- Strong talent pool for hiring
- Meta (Facebook) backing for long-term support

**Why Zustand over Redux?**
- Minimal boilerplate
- Excellent TypeScript inference
- Smaller bundle size
- Simpler learning curve
- Sufficient for this app's complexity

### Backend
```
Runtime:            Node.js 20 LTS
Framework:          Express.js 4.x
Language:           TypeScript
API Style:          RESTful JSON API
Validation:         Zod (shared with frontend)
Authentication:     Passport.js + JWT
File Upload:        Multer
Background Jobs:    BullMQ + Redis
ORM:                Prisma
Testing:            Jest + Supertest
```

**Why Node.js + Express?**
- JavaScript/TypeScript end-to-end (code sharing)
- Excellent async I/O for handling external API calls
- Large ecosystem (npm packages)
- Good performance for I/O-bound operations
- WebSocket support via Socket.io
- Team can work across frontend and backend

**Why Prisma ORM?**
- Type-safe database access
- Excellent TypeScript support
- Migrations built-in
- Intuitive query API
- Great DX (developer experience)

### Database
```
Primary Database:   PostgreSQL 15+
Cache:              Redis 7+
Object Storage:     AWS S3 / MinIO (self-hosted option)
Search (optional):  PostgreSQL Full-Text Search
```

### Infrastructure
```
Web Server:         NGINX (reverse proxy + static file serving)
Process Manager:    PM2 (Node.js process management)
Hosting Options:
  - Single VPS: DigitalOcean Droplet / Linode / Hetzner
  - Managed: Render / Railway / Fly.io
  - Cloud: AWS EC2 + RDS + ElastiCache
Monitoring:         PM2 monitoring / Sentry for errors
Logging:            Winston / Pino to files + LogRotate
```

---

## Application Architecture

### Directory Structure

```
house-management-app/
├── client/                          # Frontend React application
│   ├── src/
│   │   ├── components/              # Reusable UI components
│   │   │   ├── ui/                  # Base UI components (shadcn)
│   │   │   ├── calendar/            # Calendar-specific components
│   │   │   ├── financial/           # Financial components
│   │   │   ├── vehicle/             # Vehicle components
│   │   │   └── layout/              # Layout components
│   │   ├── pages/                   # Page components (routes)
│   │   │   ├── Dashboard.tsx
│   │   │   ├── Calendar.tsx
│   │   │   ├── Finances/
│   │   │   ├── Vehicles/
│   │   │   └── Settings.tsx
│   │   ├── hooks/                   # Custom React hooks
│   │   ├── services/                # API client functions
│   │   ├── stores/                  # Zustand state stores
│   │   ├── types/                   # TypeScript types (shared)
│   │   ├── utils/                   # Utility functions
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── public/
│   ├── index.html
│   ├── vite.config.ts
│   ├── tsconfig.json
│   └── package.json
│
├── server/                          # Backend Node.js application
│   ├── src/
│   │   ├── api/                     # API routes
│   │   │   ├── auth/                # Authentication routes
│   │   │   ├── calendar/            # Calendar endpoints
│   │   │   ├── financial/           # Financial endpoints
│   │   │   ├── vehicles/            # Vehicle endpoints
│   │   │   ├── tasks/               # Task endpoints
│   │   │   └── index.ts             # Route aggregation
│   │   ├── services/                # Business logic layer
│   │   │   ├── CalendarService.ts
│   │   │   ├── FinancialService.ts
│   │   │   ├── PlaidService.ts
│   │   │   ├── VehicleService.ts
│   │   │   ├── NotificationService.ts
│   │   │   └── DocumentService.ts
│   │   ├── jobs/                    # Background job definitions
│   │   │   ├── syncFinancialAccounts.ts
│   │   │   ├── syncCalendars.ts
│   │   │   ├── sendReminders.ts
│   │   │   └── index.ts
│   │   ├── middleware/              # Express middleware
│   │   │   ├── auth.ts              # JWT verification
│   │   │   ├── errorHandler.ts
│   │   │   ├── validation.ts
│   │   │   └── rbac.ts              # Role-based access control
│   │   ├── lib/                     # External service clients
│   │   │   ├── plaid.ts
│   │   │   ├── s3.ts
│   │   │   ├── email.ts
│   │   │   └── calendar-providers/
│   │   ├── utils/                   # Utility functions
│   │   ├── types/                   # TypeScript types
│   │   ├── config/                  # Configuration
│   │   │   └── index.ts             # Environment variables
│   │   ├── app.ts                   # Express app setup
│   │   └── server.ts                # Server entry point
│   ├── prisma/
│   │   ├── schema.prisma            # Database schema
│   │   ├── migrations/
│   │   └── seed.ts                  # Database seeding
│   ├── tests/
│   ├── tsconfig.json
│   └── package.json
│
├── shared/                          # Shared code between client/server
│   ├── types/                       # Shared TypeScript interfaces
│   ├── validation/                  # Shared Zod schemas
│   └── constants/                   # Shared constants
│
├── docker/                          # Docker configuration
│   ├── Dockerfile.dev
│   ├── Dockerfile.prod
│   └── docker-compose.yml
│
├── scripts/                         # Utility scripts
│   ├── setup.sh
│   ├── deploy.sh
│   └── backup.sh
│
├── .github/
│   └── workflows/                   # CI/CD pipelines
│       ├── test.yml
│       └── deploy.yml
│
├── docs/                            # Additional documentation
├── README.md
└── package.json                     # Root package.json (workspace)
```

### Modular Monolith Pattern

**Principle:** Organize code into clear domain modules within the monolith, making it easier to extract into microservices later if needed.

**Module Boundaries:**
1. **Authentication Module** - User management, login, permissions
2. **Calendar Module** - Events, syncing, reminders
3. **Financial Module** - Accounts, transactions, bills, budgets
4. **Vehicle Module** - Vehicles, maintenance, costs
5. **Household Module** - Tasks, documents, shopping lists
6. **Notification Module** - Email, push, SMS notifications

**Communication:**
- Modules communicate via well-defined service interfaces
- No direct database access across modules
- Shared types via the `shared/` directory

---

## Database Design

### Prisma Schema (Simplified)

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Family {
  id          String   @id @default(uuid())
  name        String
  timezone    String   @default("UTC")
  currency    String   @default("USD")
  settings    Json     @default("{}")
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  users              User[]
  calendarEvents     CalendarEvent[]
  financialAccounts  FinancialAccount[]
  bills              Bill[]
  vehicles           Vehicle[]
  tasks              Task[]
  documents          Document[]

  @@index([createdAt])
}

model User {
  id           String   @id @default(uuid())
  email        String   @unique
  passwordHash String
  name         String
  role         UserRole
  avatarUrl    String?
  preferences  Json     @default("{}")
  lastLogin    DateTime?
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  familyId String
  family   Family @relation(fields: [familyId], references: [id], onDelete: Cascade)

  createdEvents CalendarEvent[] @relation("EventCreator")
  assignedTasks Task[]

  @@index([familyId])
  @@index([email])
}

enum UserRole {
  ADMIN
  ADULT
  TEEN
  CHILD
}

model CalendarEvent {
  id          String   @id @default(uuid())
  title       String
  description String?
  startTime   DateTime
  endTime     DateTime
  allDay      Boolean  @default(false)
  category    EventCategory
  location    String?
  attendees   Json     @default("[]")
  reminders   Json     @default("[]")
  recurrence  Json?

  familyId  String
  family    Family @relation(fields: [familyId], references: [id], onDelete: Cascade)

  createdById String
  createdBy   User   @relation("EventCreator", fields: [createdById], references: [id])

  externalSource   String?  // "google_calendar", "outlook", etc.
  externalId       String?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([familyId, startTime])
  @@index([externalId])
}

enum EventCategory {
  SCHOOL
  ACTIVITIES
  MEDICAL
  CELEBRATION
  WORK
  OTHER
}

model FinancialAccount {
  id              String   @id @default(uuid())
  institutionName String
  accountType     AccountType
  accountName     String
  accountMask     String
  currentBalance  Decimal  @db.Decimal(12, 2)
  availableBalance Decimal? @db.Decimal(12, 2)
  currency        String   @default("USD")

  plaidItemId     String?
  plaidAccountId  String?

  lastSynced   DateTime?
  syncStatus   SyncStatus @default(PENDING)
  syncError    String?
  isActive     Boolean    @default(true)

  familyId String
  family   Family @relation(fields: [familyId], references: [id], onDelete: Cascade)

  transactions Transaction[]

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([familyId])
  @@index([plaidAccountId])
}

enum AccountType {
  CHECKING
  SAVINGS
  CREDIT_CARD
  INVESTMENT
}

enum SyncStatus {
  SUCCESS
  PENDING
  ERROR
}

model Transaction {
  id          String   @id @default(uuid())
  amount      Decimal  @db.Decimal(12, 2)
  date        DateTime
  description String
  category    String?
  pending     Boolean  @default(false)

  plaidTransactionId String? @unique

  accountId String
  account   FinancialAccount @relation(fields: [accountId], references: [id], onDelete: Cascade)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([accountId, date])
  @@index([plaidTransactionId])
}

model Bill {
  id          String   @id @default(uuid())
  name        String
  category    BillCategory
  amount      Decimal  @db.Decimal(10, 2)
  currency    String   @default("USD")
  dueDate     DateTime
  isRecurring Boolean  @default(false)
  recurrence  Json?

  paymentAccountId String?
  autoPay          Boolean @default(false)

  status    BillStatus @default(PENDING)
  paidDate  DateTime?
  notes     String?

  familyId String
  family   Family @relation(fields: [familyId], references: [id], onDelete: Cascade)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([familyId, dueDate])
  @@index([status])
}

enum BillCategory {
  UTILITIES
  INSURANCE
  SUBSCRIPTION
  MORTGAGE
  LOAN
  OTHER
}

enum BillStatus {
  PENDING
  PAID
  OVERDUE
}

model Vehicle {
  id            String   @id @default(uuid())
  name          String
  make          String
  model         String
  year          Int
  vin           String?
  licensePlate  String?
  color         String?
  purchaseDate  DateTime?
  purchasePrice Decimal?  @db.Decimal(10, 2)
  currentMileage Int?

  insurance    Json?    // { provider, policyNumber, renewalDate, annualCost }
  registration Json?    // { expirationDate, renewalCost }

  familyId String
  family   Family @relation(fields: [familyId], references: [id], onDelete: Cascade)

  maintenanceRecords MaintenanceRecord[]

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([familyId])
}

model MaintenanceRecord {
  id              String   @id @default(uuid())
  serviceType     String
  date            DateTime
  mileage         Int?
  cost            Decimal  @db.Decimal(10, 2)
  serviceProvider String?
  notes           String?
  receiptUrl      String?

  nextServiceDue Json?    // { date, mileage }

  vehicleId String
  vehicle   Vehicle @relation(fields: [vehicleId], references: [id], onDelete: Cascade)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([vehicleId, date])
}

model Task {
  id          String   @id @default(uuid())
  title       String
  description String?
  dueDate     DateTime?
  priority    Priority @default(MEDIUM)
  status      TaskStatus @default(TODO)
  isRecurring Boolean  @default(false)
  recurrence  Json?

  assignedToId String?
  assignedTo   User?   @relation(fields: [assignedToId], references: [id], onDelete: SetNull)

  familyId String
  family   Family @relation(fields: [familyId], references: [id], onDelete: Cascade)

  completedAt DateTime?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([familyId, status])
  @@index([assignedToId])
}

enum Priority {
  LOW
  MEDIUM
  HIGH
  URGENT
}

enum TaskStatus {
  TODO
  IN_PROGRESS
  COMPLETED
  CANCELLED
}

model Document {
  id          String   @id @default(uuid())
  name        String
  category    DocumentCategory
  storagePath String
  fileSize    Int
  mimeType    String

  expirationDate DateTime?
  tags           String[]
  notes          String?

  familyId String
  family   Family @relation(fields: [familyId], references: [id], onDelete: Cascade)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([familyId, category])
  @@index([expirationDate])
}

enum DocumentCategory {
  FINANCIAL
  MEDICAL
  LEGAL
  HOME
  VEHICLE
  OTHER
}
```

---

## API Design

### RESTful Endpoint Structure

```
Authentication:
POST   /api/auth/register              - Register new family
POST   /api/auth/login                 - Login user
POST   /api/auth/logout                - Logout user
POST   /api/auth/refresh               - Refresh access token
POST   /api/auth/forgot-password       - Request password reset
POST   /api/auth/reset-password        - Reset password

Family Management:
GET    /api/family                     - Get family details
PATCH  /api/family                     - Update family settings
GET    /api/family/members             - List family members
POST   /api/family/members             - Invite family member
DELETE /api/family/members/:userId     - Remove family member

Calendar:
GET    /api/calendar/events            - List events (with date range filter)
POST   /api/calendar/events            - Create event
GET    /api/calendar/events/:id        - Get event details
PATCH  /api/calendar/events/:id        - Update event
DELETE /api/calendar/events/:id        - Delete event
POST   /api/calendar/sync              - Trigger calendar sync
GET    /api/calendar/integrations      - List connected calendars
POST   /api/calendar/integrations      - Connect external calendar
DELETE /api/calendar/integrations/:id  - Disconnect calendar

Financial:
GET    /api/financial/accounts         - List all accounts
POST   /api/financial/accounts/link    - Link new account (Plaid Link)
GET    /api/financial/accounts/:id     - Get account details
DELETE /api/financial/accounts/:id     - Remove account
POST   /api/financial/accounts/:id/sync - Force sync account
GET    /api/financial/transactions     - List transactions (with filters)
GET    /api/financial/summary          - Get financial summary
GET    /api/financial/net-worth        - Calculate net worth

Bills:
GET    /api/bills                      - List bills
POST   /api/bills                      - Create bill
GET    /api/bills/:id                  - Get bill details
PATCH  /api/bills/:id                  - Update bill
DELETE /api/bills/:id                  - Delete bill
POST   /api/bills/:id/pay              - Mark bill as paid

Vehicles:
GET    /api/vehicles                   - List vehicles
POST   /api/vehicles                   - Add vehicle
GET    /api/vehicles/:id               - Get vehicle details
PATCH  /api/vehicles/:id               - Update vehicle
DELETE /api/vehicles/:id               - Delete vehicle
GET    /api/vehicles/:id/maintenance   - List maintenance records
POST   /api/vehicles/:id/maintenance   - Add maintenance record
GET    /api/vehicles/:id/costs         - Get cost analytics

Tasks:
GET    /api/tasks                      - List tasks
POST   /api/tasks                      - Create task
GET    /api/tasks/:id                  - Get task details
PATCH  /api/tasks/:id                  - Update task
DELETE /api/tasks/:id                  - Delete task
POST   /api/tasks/:id/complete         - Mark task complete

Documents:
GET    /api/documents                  - List documents
POST   /api/documents                  - Upload document
GET    /api/documents/:id              - Get document metadata
GET    /api/documents/:id/download     - Download document file
PATCH  /api/documents/:id              - Update document metadata
DELETE /api/documents/:id              - Delete document

Notifications:
GET    /api/notifications              - List user notifications
PATCH  /api/notifications/:id/read     - Mark as read
DELETE /api/notifications/:id          - Dismiss notification
```

### API Response Format

```typescript
// Success Response
{
  "success": true,
  "data": { /* response data */ },
  "meta": {
    "timestamp": "2025-11-23T10:30:00Z",
    "requestId": "req_abc123"
  }
}

// Error Response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  },
  "meta": {
    "timestamp": "2025-11-23T10:30:00Z",
    "requestId": "req_abc123"
  }
}

// Paginated Response
{
  "success": true,
  "data": [ /* array of items */ ],
  "pagination": {
    "page": 1,
    "perPage": 20,
    "total": 150,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": false
  },
  "meta": {
    "timestamp": "2025-11-23T10:30:00Z",
    "requestId": "req_abc123"
  }
}
```

---

## Authentication & Authorization Flow

### JWT-Based Authentication

```typescript
// Login Flow
1. User submits email + password
2. Server validates credentials (bcrypt compare)
3. Server generates:
   - Access Token (JWT, 15-min expiry)
   - Refresh Token (JWT, 7-day expiry, stored in httpOnly cookie)
4. Client stores access token (memory/sessionStorage)
5. Client uses access token in Authorization header

// Token Refresh Flow
1. Access token expires
2. Client sends refresh token (automatic via cookie)
3. Server validates refresh token
4. Server issues new access token
5. Client continues with new token

// Logout Flow
1. Client calls /api/auth/logout
2. Server clears refresh token cookie
3. Client clears access token from memory
```

### Authorization with RBAC

```typescript
// Middleware example (server/src/middleware/auth.ts)
interface AuthRequest extends Request {
  user?: {
    userId: string;
    familyId: string;
    role: UserRole;
  };
}

function requireAuth(req: AuthRequest, res: Response, next: NextFunction) {
  // Verify JWT from Authorization header
  // Attach user info to req.user
}

function requireRole(...roles: UserRole[]) {
  return (req: AuthRequest, res: Response, next: NextFunction) => {
    if (!req.user || !roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}

// Usage in routes
router.delete('/family/members/:userId',
  requireAuth,
  requireRole('ADMIN'),
  deleteFamilyMember
);
```

---

## Background Jobs

### Job Queue with BullMQ

```typescript
// server/src/jobs/index.ts
import { Queue, Worker } from 'bullmq';
import Redis from 'ioredis';

const connection = new Redis(process.env.REDIS_URL);

// Define Queues
export const financialSyncQueue = new Queue('financial-sync', { connection });
export const calendarSyncQueue = new Queue('calendar-sync', { connection });
export const reminderQueue = new Queue('reminders', { connection });

// Job Producers
export async function scheduleFinancialSync(familyId: string) {
  await financialSyncQueue.add(
    'sync-accounts',
    { familyId },
    {
      repeat: { pattern: '0 */6 * * *' } // Every 6 hours
    }
  );
}

// Job Consumers
const financialWorker = new Worker('financial-sync', async (job) => {
  const { familyId } = job.data;
  // Sync all accounts for this family via Plaid
  await FinancialService.syncAllAccounts(familyId);
}, { connection });

const reminderWorker = new Worker('reminders', async (job) => {
  const { type, data } = job.data;

  if (type === 'bill-due') {
    await NotificationService.sendBillReminder(data);
  } else if (type === 'maintenance-due') {
    await NotificationService.sendMaintenanceReminder(data);
  }
}, { connection });
```

### Scheduled Jobs

```typescript
// Daily job to check for upcoming bills and send reminders
async function checkBillReminders() {
  const upcomingBills = await prisma.bill.findMany({
    where: {
      status: 'PENDING',
      dueDate: {
        gte: new Date(),
        lte: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // 7 days ahead
      }
    },
    include: { family: { include: { users: true } } }
  });

  for (const bill of upcomingBills) {
    const daysUntilDue = Math.ceil(
      (bill.dueDate.getTime() - Date.now()) / (24 * 60 * 60 * 1000)
    );

    if ([7, 3, 1].includes(daysUntilDue)) {
      await reminderQueue.add('bill-due', {
        type: 'bill-due',
        data: { bill, daysUntilDue }
      });
    }
  }
}

// Run daily at 9 AM
reminderQueue.add('check-bill-reminders', {}, {
  repeat: { pattern: '0 9 * * *' }
});
```

---

## External Integrations

### Plaid Integration

```typescript
// server/src/lib/plaid.ts
import { Configuration, PlaidApi, PlaidEnvironments } from 'plaid';

const configuration = new Configuration({
  basePath: PlaidEnvironments[process.env.PLAID_ENV],
  baseOptions: {
    headers: {
      'PLAID-CLIENT-ID': process.env.PLAID_CLIENT_ID,
      'PLAID-SECRET': process.env.PLAID_SECRET,
    },
  },
});

export const plaidClient = new PlaidApi(configuration);

// Create Link Token for frontend
export async function createLinkToken(userId: string) {
  const response = await plaidClient.linkTokenCreate({
    user: { client_user_id: userId },
    client_name: 'Family House Management',
    products: ['transactions', 'auth'],
    country_codes: ['US'],
    language: 'en',
  });

  return response.data.link_token;
}

// Exchange public token for access token
export async function exchangePublicToken(publicToken: string) {
  const response = await plaidClient.itemPublicTokenExchange({
    public_token: publicToken,
  });

  return {
    accessToken: response.data.access_token,
    itemId: response.data.item_id,
  };
}

// Fetch accounts
export async function getAccounts(accessToken: string) {
  const response = await plaidClient.accountsGet({
    access_token: accessToken,
  });

  return response.data.accounts;
}

// Sync transactions
export async function syncTransactions(
  accessToken: string,
  startDate: string,
  endDate: string
) {
  const response = await plaidClient.transactionsGet({
    access_token: accessToken,
    start_date: startDate,
    end_date: endDate,
  });

  return response.data.transactions;
}
```

### Google Calendar Integration

```typescript
// server/src/lib/calendar-providers/google.ts
import { google } from 'googleapis';

export async function setupGoogleCalendar(user: User) {
  const oauth2Client = new google.auth.OAuth2(
    process.env.GOOGLE_CLIENT_ID,
    process.env.GOOGLE_CLIENT_SECRET,
    process.env.GOOGLE_REDIRECT_URI
  );

  // Store oauth2Client tokens in user preferences
  oauth2Client.setCredentials({
    access_token: user.googleAccessToken,
    refresh_token: user.googleRefreshToken,
  });

  return google.calendar({ version: 'v3', auth: oauth2Client });
}

export async function syncGoogleCalendar(user: User, familyId: string) {
  const calendar = await setupGoogleCalendar(user);

  const response = await calendar.events.list({
    calendarId: 'primary',
    timeMin: new Date().toISOString(),
    maxResults: 100,
    singleEvents: true,
    orderBy: 'startTime',
  });

  const events = response.data.items || [];

  // Upsert events into our database
  for (const event of events) {
    await prisma.calendarEvent.upsert({
      where: { externalId: event.id },
      update: {
        title: event.summary,
        startTime: new Date(event.start.dateTime || event.start.date),
        endTime: new Date(event.end.dateTime || event.end.date),
        // ... other fields
      },
      create: {
        familyId,
        createdById: user.id,
        title: event.summary,
        // ... other fields
        externalSource: 'google_calendar',
        externalId: event.id,
      },
    });
  }
}
```

---

## Deployment Strategy

### Single Server Deployment (Simple, Cost-Effective)

**Infrastructure:**
- VPS: 4 vCPU, 8 GB RAM, 160 GB SSD (e.g., DigitalOcean $48/month)
- Managed PostgreSQL: Small instance ($15/month)
- Managed Redis: Small instance ($15/month)
- Total: ~$80-100/month for 1,000-5,000 families

**Server Setup:**
```
┌──────────────────────────────────────────┐
│  DigitalOcean Droplet / Linode VPS      │
├──────────────────────────────────────────┤
│                                          │
│  ┌────────────────────────────────────┐ │
│  │ NGINX (Port 80/443)                │ │
│  │ - SSL Termination                  │ │
│  │ - Static file serving              │ │
│  │ - Reverse proxy to Node.js         │ │
│  └──────────────┬─────────────────────┘ │
│                 │                        │
│  ┌──────────────▼─────────────────────┐ │
│  │ Node.js App (PM2)                  │ │
│  │ - Express API server               │ │
│  │ - 4 instances (cluster mode)       │ │
│  │ - Auto-restart on crash            │ │
│  └────────────────────────────────────┘ │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │ BullMQ Workers (PM2)               │ │
│  │ - Background job processing        │ │
│  │ - 2 worker instances               │ │
│  └────────────────────────────────────┘ │
│                                          │
└──────────────────────────────────────────┘
         │                    │
         │                    │
    ┌────▼─────┐       ┌─────▼──────┐
    │PostgreSQL│       │   Redis    │
    │ Managed  │       │  Managed   │
    │   DB     │       │   Cache    │
    └──────────┘       └────────────┘
```

**Deployment Process:**
```bash
# 1. Build frontend
cd client && npm run build

# 2. Build backend
cd server && npm run build

# 3. Upload to server (rsync or git pull)
rsync -avz --exclude node_modules ./dist user@server:/var/www/app/

# 4. Install dependencies on server
ssh user@server "cd /var/www/app && npm ci --production"

# 5. Run migrations
ssh user@server "cd /var/www/app/server && npx prisma migrate deploy"

# 6. Restart PM2
ssh user@server "pm2 restart all"
```

### Docker-Based Deployment (More Portable)

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./client/dist:/usr/share/nginx/html
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
    depends_on:
      - api

  api:
    build:
      context: ./server
      dockerfile: Dockerfile
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      - postgres
      - redis
    restart: unless-stopped

  worker:
    build:
      context: ./server
      dockerfile: Dockerfile
    command: npm run worker
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    depends_on:
      - postgres
      - redis
    restart: unless-stopped

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=house_management
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

---

## Development Workflow

### Local Development Setup

```bash
# 1. Clone repository
git clone <repo-url>
cd house-management-app

# 2. Install dependencies
npm install  # Root (if using workspaces)
cd client && npm install
cd ../server && npm install

# 3. Setup environment variables
cp server/.env.example server/.env
# Edit .env with your credentials

# 4. Start PostgreSQL and Redis (Docker)
docker-compose -f docker-compose.dev.yml up -d

# 5. Run database migrations
cd server && npx prisma migrate dev

# 6. Start development servers
# Terminal 1: Frontend
cd client && npm run dev  # Vite dev server on localhost:5173

# Terminal 2: Backend API
cd server && npm run dev  # Node dev server on localhost:3000

# Terminal 3: Background workers
cd server && npm run worker
```

### Git Workflow

```
main (protected)
  ├── develop (integration branch)
  │    ├── feature/calendar-sync
  │    ├── feature/plaid-integration
  │    └── feature/vehicle-management
  └── hotfix/critical-bug
```

**Process:**
1. Create feature branch from `develop`
2. Develop feature with commits
3. Create PR to `develop`
4. Code review + automated tests
5. Merge to `develop`
6. Periodic releases from `develop` to `main`

---

## Testing Strategy

### Unit Tests
```typescript
// Example: server/tests/services/CalendarService.test.ts
import { CalendarService } from '@/services/CalendarService';
import { prismaMock } from './mocks/prisma';

describe('CalendarService', () => {
  it('should create a calendar event', async () => {
    const eventData = {
      familyId: 'family-123',
      title: 'Soccer Practice',
      startTime: new Date('2025-12-01T14:00:00Z'),
      endTime: new Date('2025-12-01T15:30:00Z'),
      category: 'ACTIVITIES',
    };

    prismaMock.calendarEvent.create.mockResolvedValue({
      id: 'event-123',
      ...eventData,
    });

    const result = await CalendarService.createEvent(eventData);

    expect(result.id).toBe('event-123');
    expect(result.title).toBe('Soccer Practice');
  });
});
```

### Integration Tests
```typescript
// Example: server/tests/api/calendar.test.ts
import request from 'supertest';
import app from '@/app';

describe('POST /api/calendar/events', () => {
  it('should create event with valid auth', async () => {
    const token = await getAuthToken(); // Helper function

    const response = await request(app)
      .post('/api/calendar/events')
      .set('Authorization', `Bearer ${token}`)
      .send({
        title: 'Dentist Appointment',
        startTime: '2025-12-05T10:00:00Z',
        endTime: '2025-12-05T11:00:00Z',
        category: 'MEDICAL',
      });

    expect(response.status).toBe(201);
    expect(response.body.data.title).toBe('Dentist Appointment');
  });
});
```

### E2E Tests (Optional)
- Playwright or Cypress for full user flows
- Test critical paths: login, create event, link bank account

---

## Monitoring & Maintenance

### Application Monitoring
- PM2 monitoring for process health
- Sentry for error tracking
- Custom logging with Winston/Pino
- Uptime monitoring (UptimeRobot)

### Database Maintenance
- Weekly VACUUM ANALYZE (PostgreSQL)
- Monthly index rebuilding
- Automated backups (daily)
- Monitor query performance (pg_stat_statements)

### Performance Optimization
- Database query optimization (indexes, EXPLAIN ANALYZE)
- API response caching (Redis)
- Static asset caching (NGINX)
- Image optimization
- Code splitting in frontend

---

## Pros & Cons of Monolithic Approach

### Advantages ✅
- **Faster Development**: Single codebase, no inter-service communication overhead
- **Easier Debugging**: All code in one place, simple stack traces
- **Lower Cost**: Single server deployment, fewer infrastructure components
- **Simpler Deployment**: One deployment unit, no orchestration needed
- **ACID Transactions**: Database transactions across all features
- **Strong Consistency**: No eventual consistency challenges
- **Developer Experience**: Easy local setup, straightforward testing

### Disadvantages ❌
- **Scaling Limitations**: Can only scale vertically initially (bigger server)
- **Technology Lock-in**: Entire app tied to one stack (Node.js)
- **Deployment Risk**: Single deployment affects entire application
- **Team Coordination**: Harder for large teams to work independently
- **Resource Coupling**: Heavy background job can affect API performance
- **Potential for Spaghetti Code**: Requires discipline to maintain modularity

### When to Choose This Strategy
- ✅ Small to medium-scale application (< 10,000 families)
- ✅ Small development team (2-6 developers)
- ✅ Tight budget constraints
- ✅ Fast time-to-market priority
- ✅ Uncertain scale/feature requirements
- ✅ MVP or early-stage product

---

## Migration Path to Microservices

If the application outgrows the monolith, the modular structure allows for gradual extraction:

**Phase 1:** Extract Background Jobs
- Move workers to separate service
- Communicate via Redis queue (already in place)

**Phase 2:** Extract Notification Service
- Standalone notification microservice
- API calls from main app

**Phase 3:** Extract Financial Service (most complex integrations)
- Separate service handling all Plaid interactions
- Expose REST API to main app

**Phase 4:** Continue based on bottlenecks and team structure

---

## Estimated Timeline

**Month 1-2: Foundation**
- Project setup, CI/CD, infrastructure
- Database schema and migrations
- Authentication system
- Basic frontend shell

**Month 3-4: Core Features**
- Calendar module (CRUD + basic sync)
- Financial account linking (Plaid)
- Vehicle management

**Month 5-6: Advanced Features**
- Bill tracking and reminders
- Transaction categorization
- Task management
- Background job system

**Month 7-8: Polish & Launch**
- Document storage
- Notifications (email, push)
- Mobile responsive optimization
- Security audit
- Beta testing
- Production deployment

**Month 9+: Post-Launch**
- User feedback iteration
- Performance optimization
- Additional integrations
- Advanced analytics

---

## Conclusion

The monolithic approach provides a pragmatic, cost-effective path to building the Family House Management application. It prioritizes speed of development, simplicity of operations, and developer experience while maintaining the flexibility to scale and evolve as the product grows.

**Best for:** Teams that value simplicity, rapid iteration, and want to validate the product-market fit before investing in more complex architectures.
