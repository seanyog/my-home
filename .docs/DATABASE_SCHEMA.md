# Database Schema Design
## Family House Management Application - Monolithic Strategy

**Database:** PostgreSQL 15
**ORM:** Prisma
**Version:** 1.0

---

## Complete Prisma Schema

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
  previewFeatures = ["fullTextSearch"]
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ============================================================================
// CORE MODELS
// ============================================================================

model Family {
  id        String   @id @default(uuid())
  name      String
  timezone  String   @default("America/New_York")
  currency  String   @default("USD")
  locale    String   @default("en-US")

  // JSON settings
  settings  Json     @default("{}")
  // Example structure:
  // {
  //   "notifications": {
  //     "billReminderDays": [7, 3, 1],
  //     "maintenanceReminders": true
  //   },
  //   "calendar": {
  //     "defaultView": "month",
  //     "weekStartsOn": 0
  //   }
  // }

  // Subscription info (for future use)
  subscriptionPlan   String   @default("free") // "free", "premium"
  subscriptionStatus String   @default("active") // "active", "trial", "cancelled"

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  users              User[]
  calendarEvents     CalendarEvent[]
  financialAccounts  FinancialAccount[]
  transactions       Transaction[]
  bills              Bill[]
  vehicles           Vehicle[]
  tasks              Task[]
  documents          Document[]
  integrations       Integration[]

  @@index([createdAt])
  @@map("families")
}

model User {
  id           String   @id @default(uuid())
  email        String   @unique
  passwordHash String
  name         String
  role         UserRole @default(ADULT)

  // Profile
  avatarUrl    String?
  phone        String?

  // Preferences (JSON)
  preferences  Json     @default("{}")
  // Example structure:
  // {
  //   "notificationChannels": ["email", "push"],
  //   "calendarView": "month",
  //   "theme": "light",
  //   "language": "en"
  // }

  // Email verification
  emailVerified Boolean  @default(false)
  verificationToken String?
  verificationTokenExpiry DateTime?

  // Password reset
  resetToken       String?
  resetTokenExpiry DateTime?

  // Activity tracking
  lastLogin    DateTime?
  lastActive   DateTime?

  // Timestamps
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  // Foreign keys
  familyId     String
  family       Family   @relation(fields: [familyId], references: [id], onDelete: Cascade)

  // Relations
  createdEvents    CalendarEvent[] @relation("EventCreator")
  assignedTasks    Task[]          @relation("TaskAssignee")
  createdTasks     Task[]          @relation("TaskCreator")
  notifications    Notification[]

  @@index([familyId])
  @@index([email])
  @@index([lastActive])
  @@map("users")
}

enum UserRole {
  ADMIN    // Full access, can manage family
  ADULT    // Standard access, can create/edit
  TEEN     // Limited access, can view most, edit own
  CHILD    // Very limited access
}

// ============================================================================
// CALENDAR MODELS
// ============================================================================

model CalendarEvent {
  id          String   @id @default(uuid())
  title       String
  description String?

  // Date/Time
  startTime   DateTime
  endTime     DateTime
  allDay      Boolean  @default(false)
  timezone    String?  // Event-specific timezone

  // Categorization
  category    EventCategory @default(OTHER)
  color       String?       // Hex color override

  // Location
  location    String?
  locationLat Float?
  locationLng Float?

  // Attendees (array of user IDs as JSON)
  attendees   Json     @default("[]")
  // Example: ["user-id-1", "user-id-2"]

  // Reminders (JSON array)
  reminders   Json     @default("[]")
  // Example: [
  //   { "minutesBefore": 60, "type": "email" },
  //   { "minutesBefore": 15, "type": "push" }
  // ]

  // Recurrence (JSON)
  recurrence  Json?
  // Example: {
  //   "frequency": "weekly",
  //   "interval": 1,
  //   "byDay": ["MO", "WE"],
  //   "until": "2026-12-31"
  // }
  recurringEventId String? // Parent event for recurring series

  // External sync
  externalSource   String?  // "google_calendar", "outlook", "ical"
  externalId       String?
  externalCalendarId String?

  // Metadata
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // Foreign keys
  familyId    String
  family      Family   @relation(fields: [familyId], references: [id], onDelete: Cascade)

  createdById String
  createdBy   User     @relation("EventCreator", fields: [createdById], references: [id], onDelete: Cascade)

  @@index([familyId, startTime])
  @@index([familyId, endTime])
  @@index([externalId])
  @@index([recurringEventId])
  @@map("calendar_events")
}

enum EventCategory {
  SCHOOL
  ACTIVITIES
  MEDICAL
  CELEBRATION
  WORK
  PERSONAL
  OTHER
}

model Integration {
  id          String   @id @default(uuid())
  provider    IntegrationProvider

  // OAuth credentials (encrypted)
  accessToken     String?
  refreshToken    String?
  tokenExpiry     DateTime?

  // Provider-specific IDs
  externalUserId  String?
  externalCalendarId String?

  // Sync status
  lastSync        DateTime?
  syncStatus      SyncStatus @default(PENDING)
  syncError       String?

  // Settings
  enabled         Boolean  @default(true)
  syncFrequency   Int      @default(15) // minutes

  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  // Foreign keys
  familyId    String
  family      Family   @relation(fields: [familyId], references: [id], onDelete: Cascade)

  @@index([familyId, provider])
  @@map("integrations")
}

enum IntegrationProvider {
  GOOGLE_CALENDAR
  OUTLOOK_CALENDAR
  ICAL
}

enum SyncStatus {
  SUCCESS
  PENDING
  ERROR
  DISABLED
}

// ============================================================================
// FINANCIAL MODELS
// ============================================================================

model FinancialAccount {
  id              String   @id @default(uuid())

  // Account details
  institutionName String
  institutionId   String?
  accountType     AccountType
  accountName     String
  accountMask     String   // Last 4 digits

  // Balances
  currentBalance  Decimal  @db.Decimal(12, 2)
  availableBalance Decimal? @db.Decimal(12, 2)
  currency        String   @default("USD")

  // Plaid integration
  plaidItemId     String?
  plaidAccountId  String?
  plaidAccessToken String? // Encrypted

  // Sync status
  lastSynced      DateTime?
  syncStatus      SyncStatus @default(PENDING)
  syncError       String?

  // Account status
  isActive        Boolean  @default(true)

  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  // Foreign keys
  familyId    String
  family      Family   @relation(fields: [familyId], references: [id], onDelete: Cascade)

  // Relations
  transactions Transaction[]
  bills        Bill[]        @relation("BillPaymentAccount")

  @@index([familyId])
  @@index([plaidItemId])
  @@index([plaidAccountId])
  @@unique([familyId, plaidAccountId])
  @@map("financial_accounts")
}

enum AccountType {
  CHECKING
  SAVINGS
  CREDIT_CARD
  INVESTMENT
  LOAN
  OTHER
}

model Transaction {
  id          String   @id @default(uuid())

  // Transaction details
  amount      Decimal  @db.Decimal(12, 2)
  date        DateTime
  description String
  category    String?
  subcategory String?

  // Merchant info
  merchantName String?

  // Status
  pending     Boolean  @default(false)

  // Plaid data
  plaidTransactionId String? @unique
  plaidCategoryId    String?

  // User overrides
  userCategory       String?
  userNotes          String?

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // Foreign keys
  accountId   String
  account     FinancialAccount @relation(fields: [accountId], references: [id], onDelete: Cascade)

  familyId    String
  family      Family   @relation(fields: [familyId], references: [id], onDelete: Cascade)

  @@index([accountId, date])
  @@index([familyId, date])
  @@index([plaidTransactionId])
  @@index([category])
  @@map("transactions")
}

model Bill {
  id          String   @id @default(uuid())

  // Bill details
  name        String
  category    BillCategory @default(OTHER)
  amount      Decimal  @db.Decimal(10, 2)
  currency    String   @default("USD")

  // Due date
  dueDate     DateTime

  // Recurrence
  isRecurring Boolean  @default(false)
  recurrence  Json?
  // Example: {
  //   "frequency": "monthly",
  //   "dayOfMonth": 15,
  //   "monthsInterval": 1
  // }

  // Payment
  paymentAccountId String?
  paymentAccount   FinancialAccount? @relation("BillPaymentAccount", fields: [paymentAccountId], references: [id], onDelete: SetNull)
  autoPay          Boolean  @default(false)

  // Status
  status      BillStatus @default(PENDING)
  paidDate    DateTime?
  paidAmount  Decimal?  @db.Decimal(10, 2)

  // Notes
  notes       String?

  // Reminders sent
  reminders   Json     @default("[]")
  // Example: [
  //   { "type": "7_days", "sentAt": "2025-12-08T09:00:00Z" },
  //   { "type": "3_days", "sentAt": "2025-12-12T09:00:00Z" }
  // ]

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // Foreign keys
  familyId    String
  family      Family   @relation(fields: [familyId], references: [id], onDelete: Cascade)

  @@index([familyId, dueDate])
  @@index([familyId, status])
  @@index([status, dueDate])
  @@map("bills")
}

enum BillCategory {
  UTILITIES
  INSURANCE
  SUBSCRIPTION
  MORTGAGE
  LOAN
  CREDIT_CARD
  TAX
  OTHER
}

enum BillStatus {
  PENDING
  PAID
  OVERDUE
  CANCELLED
}

// ============================================================================
// VEHICLE MODELS
// ============================================================================

model Vehicle {
  id            String   @id @default(uuid())

  // Basic info
  name          String   // "Family SUV"
  make          String
  model         String
  year          Int
  color         String?

  // Identifiers
  vin           String?
  licensePlate  String?

  // Current status
  currentMileage Int?

  // Purchase info
  purchaseDate  DateTime?
  purchasePrice Decimal?  @db.Decimal(10, 2)

  // Insurance (JSON)
  insurance     Json?
  // Example: {
  //   "provider": "State Farm",
  //   "policyNumber": "SF-123456",
  //   "renewalDate": "2026-03-15",
  //   "annualCost": 1800
  // }

  // Registration (JSON)
  registration  Json?
  // Example: {
  //   "expirationDate": "2026-03-31",
  //   "renewalCost": 120
  // }

  // Photo
  photoUrl      String?

  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  // Foreign keys
  familyId      String
  family        Family   @relation(fields: [familyId], references: [id], onDelete: Cascade)

  // Relations
  maintenanceRecords MaintenanceRecord[]

  @@index([familyId])
  @@map("vehicles")
}

model MaintenanceRecord {
  id              String   @id @default(uuid())

  // Service details
  serviceType     String   // "Oil Change", "Tire Rotation", etc.
  date            DateTime
  mileage         Int?
  cost            Decimal  @db.Decimal(10, 2)

  // Service provider
  serviceProvider String?
  providerPhone   String?
  providerAddress String?

  // Notes and receipt
  notes           String?
  receiptUrl      String?

  // Next service (JSON)
  nextServiceDue  Json?
  // Example: {
  //   "date": "2026-05-01",
  //   "mileage": 27500,
  //   "type": "Oil Change"
  // }

  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  // Foreign keys
  vehicleId       String
  vehicle         Vehicle  @relation(fields: [vehicleId], references: [id], onDelete: Cascade)

  @@index([vehicleId, date])
  @@index([vehicleId, mileage])
  @@map("maintenance_records")
}

// ============================================================================
// TASK MODELS
// ============================================================================

model Task {
  id          String   @id @default(uuid())

  // Task details
  title       String
  description String?

  // Assignment
  assignedToId String?
  assignedTo   User?    @relation("TaskAssignee", fields: [assignedToId], references: [id], onDelete: SetNull)

  createdById  String
  createdBy    User     @relation("TaskCreator", fields: [createdById], references: [id], onDelete: Cascade)

  // Scheduling
  dueDate     DateTime?
  priority    Priority @default(MEDIUM)
  status      TaskStatus @default(TODO)

  // Recurrence
  isRecurring Boolean  @default(false)
  recurrence  Json?
  // Example: {
  //   "frequency": "weekly",
  //   "dayOfWeek": 6,
  //   "time": "16:00"
  // }

  // Completion
  completedAt DateTime?
  completedBy String?

  // Linked data
  linkedShoppingListId String?

  // Checklist (JSON array)
  checklist   Json?
  // Example: [
  //   { "item": "Milk", "completed": true },
  //   { "item": "Bread", "completed": false }
  // ]

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // Foreign keys
  familyId    String
  family      Family   @relation(fields: [familyId], references: [id], onDelete: Cascade)

  @@index([familyId, status])
  @@index([familyId, dueDate])
  @@index([assignedToId])
  @@map("tasks")
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

// ============================================================================
// DOCUMENT MODELS
// ============================================================================

model Document {
  id          String   @id @default(uuid())

  // File details
  name        String
  category    DocumentCategory

  // Storage
  storagePath String   // S3 key or file path
  fileSize    Int      // bytes
  mimeType    String

  // Metadata
  expirationDate DateTime?
  tags        String[]
  notes       String?

  // OCR results
  extractedText String?
  ocrProcessed  Boolean @default(false)

  // Uploader
  uploadedById  String

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // Foreign keys
  familyId    String
  family      Family   @relation(fields: [familyId], references: [id], onDelete: Cascade)

  @@index([familyId, category])
  @@index([expirationDate])
  @@map("documents")
}

enum DocumentCategory {
  FINANCIAL
  MEDICAL
  LEGAL
  HOME
  VEHICLE
  INSURANCE
  TAX
  OTHER
}

// ============================================================================
// NOTIFICATION MODELS
// ============================================================================

model Notification {
  id          String   @id @default(uuid())

  // Notification details
  type        NotificationType
  title       String
  message     String

  // Target
  userId      String
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  // Channels
  channels    String[] // ["email", "push", "sms"]

  // Status
  read        Boolean  @default(false)
  readAt      DateTime?

  // Delivery
  sentAt      DateTime?
  deliveryStatus String @default("pending") // "pending", "sent", "failed"

  // Action link
  actionUrl   String?

  // Related entity
  relatedEntityType String? // "event", "bill", "task", "vehicle"
  relatedEntityId   String?

  createdAt   DateTime @default(now())

  @@index([userId, read])
  @@index([userId, createdAt])
  @@map("notifications")
}

enum NotificationType {
  EVENT_REMINDER
  BILL_DUE
  TASK_ASSIGNED
  TASK_COMPLETED
  MAINTENANCE_DUE
  ACCOUNT_SYNC_ERROR
  FAMILY_INVITATION
  SYSTEM_ANNOUNCEMENT
}

// ============================================================================
// BACKGROUND JOB TRACKING (Optional - BullMQ handles most of this)
// ============================================================================

model JobLog {
  id          String   @id @default(uuid())

  jobType     String   // "financial_sync", "calendar_sync", "send_reminders"
  status      String   // "pending", "processing", "completed", "failed"

  // Metadata
  familyId    String?
  userId      String?

  // Results
  startedAt   DateTime?
  completedAt DateTime?
  error       String?
  result      Json?

  createdAt   DateTime @default(now())

  @@index([jobType, status])
  @@index([familyId])
  @@map("job_logs")
}
```

---

## Indexes Strategy

### High-Traffic Queries
```sql
-- Frequently queried together
CREATE INDEX idx_calendar_family_date ON calendar_events(family_id, start_time, end_time);
CREATE INDEX idx_bills_family_status ON bills(family_id, status, due_date);
CREATE INDEX idx_tasks_assignee_status ON tasks(assigned_to_id, status, due_date);

-- Foreign key indexes (auto-created by Prisma, but documented here)
CREATE INDEX idx_users_family ON users(family_id);
CREATE INDEX idx_transactions_account ON transactions(account_id);

-- Full-text search (for future)
CREATE INDEX idx_transactions_description ON transactions USING gin(to_tsvector('english', description));
```

### Unique Constraints
```sql
-- Prevent duplicate Plaid accounts
UNIQUE (family_id, plaid_account_id)

-- Prevent duplicate emails
UNIQUE (email)

-- Prevent duplicate Plaid transactions
UNIQUE (plaid_transaction_id)
```

---

## Data Validation Rules

### At Database Level
```sql
-- Check constraints
ALTER TABLE vehicles ADD CONSTRAINT year_valid CHECK (year >= 1900 AND year <= 2100);
ALTER TABLE maintenance_records ADD CONSTRAINT cost_positive CHECK (cost >= 0);
ALTER TABLE bills ADD CONSTRAINT amount_positive CHECK (amount >= 0);
```

### At Application Level (Prisma/Zod)
- Email format validation
- Password strength (min 8 chars, mixed case, number)
- Phone number format
- URL validation
- Date range validation (start < end for events)
- Timezone validation

---

## Encryption Strategy

### Encrypted Fields
```typescript
// server/src/utils/encryption.ts
import crypto from 'crypto';

const ENCRYPTION_KEY = process.env.ENCRYPTION_KEY; // 32 bytes
const IV_LENGTH = 16;

export function encrypt(text: string): string {
  const iv = crypto.randomBytes(IV_LENGTH);
  const cipher = crypto.createCipheriv('aes-256-cbc', Buffer.from(ENCRYPTION_KEY), iv);
  let encrypted = cipher.update(text);
  encrypted = Buffer.concat([encrypted, cipher.final()]);
  return iv.toString('hex') + ':' + encrypted.toString('hex');
}

export function decrypt(text: string): string {
  const textParts = text.split(':');
  const iv = Buffer.from(textParts.shift()!, 'hex');
  const encryptedText = Buffer.from(textParts.join(':'), 'hex');
  const decipher = crypto.createDecipheriv('aes-256-cbc', Buffer.from(ENCRYPTION_KEY), iv);
  let decrypted = decipher.update(encryptedText);
  decrypted = Buffer.concat([decrypted, decipher.final()]);
  return decrypted.toString();
}
```

**Encrypted Fields:**
- `User.passwordHash` (bcrypt, not reversible)
- `FinancialAccount.plaidAccessToken` (AES-256)
- `Integration.accessToken` (AES-256)
- `Integration.refreshToken` (AES-256)

---

## Migration Strategy

### Initial Migration
```bash
# Create migration
npx prisma migrate dev --name init

# Apply to production
npx prisma migrate deploy
```

### Seeding (Development/Testing)
```typescript
// prisma/seed.ts
import { PrismaClient } from '@prisma/client';
import bcrypt from 'bcrypt';

const prisma = new PrismaClient();

async function main() {
  // Create test family
  const family = await prisma.family.create({
    data: {
      name: 'Smith Family',
      timezone: 'America/New_York',
      users: {
        create: [
          {
            email: 'sarah@smith.com',
            passwordHash: await bcrypt.hash('password123', 10),
            name: 'Sarah Smith',
            role: 'ADMIN',
            emailVerified: true
          },
          {
            email: 'michael@smith.com',
            passwordHash: await bcrypt.hash('password123', 10),
            name: 'Michael Smith',
            role: 'ADULT',
            emailVerified: true
          },
          {
            email: 'emma@smith.com',
            passwordHash: await bcrypt.hash('password123', 10),
            name: 'Emma Smith',
            role: 'TEEN',
            emailVerified: true
          }
        ]
      }
    }
  });

  console.log('Seeded family:', family);
}

main()
  .catch(e => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

---

## Backup Strategy

### Automated Backups
```bash
# Daily backup script (cron job)
#!/bin/bash
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups"
DB_NAME="house_management"

pg_dump $DATABASE_URL | gzip > $BACKUP_DIR/backup_$TIMESTAMP.sql.gz

# Keep only last 30 days
find $BACKUP_DIR -name "backup_*.sql.gz" -mtime +30 -delete
```

### Point-in-Time Recovery
- Enable WAL (Write-Ahead Logging) on PostgreSQL
- Configure continuous archiving
- Managed databases (RDS, Cloud SQL) handle this automatically

---

## Performance Considerations

### Connection Pooling
```typescript
// server/src/lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma = globalForPrisma.prisma || new PrismaClient({
  log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  datasources: {
    db: {
      url: process.env.DATABASE_URL
    }
  }
});

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;

// Configure connection pool
// DATABASE_URL=postgresql://user:pass@localhost:5432/db?connection_limit=20&pool_timeout=20
```

### Query Optimization
```typescript
// BAD: N+1 query
const families = await prisma.family.findMany();
for (const family of families) {
  const users = await prisma.user.findMany({ where: { familyId: family.id } });
}

// GOOD: Use includes
const families = await prisma.family.findMany({
  include: {
    users: true
  }
});

// GOOD: Use select for specific fields
const users = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
    email: true
  }
});
```

---

## Data Retention Policy

### Active Data
- All user data retained indefinitely while account is active

### Soft Deletes
- Completed tasks: Retained for 90 days, then archived
- Paid bills: Retained for 7 years (tax purposes)
- Old transactions: Retained indefinitely (financial records)

### Hard Deletes
- User account deletion: 30-day grace period, then full purge
- Family deletion: Immediate cascade delete of all related data
- Unverified accounts: Deleted after 7 days

### GDPR Compliance
```typescript
// Implement right to be forgotten
async function deleteUserData(userId: string) {
  await prisma.$transaction([
    // Anonymize user's contributions
    prisma.task.updateMany({
      where: { createdById: userId },
      data: { createdById: 'deleted-user' }
    }),
    // Delete personal data
    prisma.notification.deleteMany({ where: { userId } }),
    prisma.user.delete({ where: { id: userId } })
  ]);
}
```

---

## Schema Evolution Plan

### Future Additions (Phase 2+)
```prisma
// Budget tracking
model Budget {
  id        String   @id @default(uuid())
  familyId  String
  category  String
  amount    Decimal  @db.Decimal(10, 2)
  period    String   // "monthly", "annual"
  startDate DateTime
  endDate   DateTime?
}

// Shopping lists
model ShoppingList {
  id       String @id @default(uuid())
  familyId String
  name     String
  items    Json   // Array of { item, quantity, completed }
}

// Household maintenance
model MaintenanceTask {
  id          String   @id @default(uuid())
  familyId    String
  title       String
  frequency   String   // "monthly", "quarterly", "annually"
  lastDone    DateTime?
  nextDue     DateTime?
}
```

---

## Testing Data

### Test Database Setup
```bash
# Create test database
createdb house_management_test

# Run migrations
DATABASE_URL="postgresql://..." npx prisma migrate deploy

# Seed test data
DATABASE_URL="postgresql://..." npx prisma db seed
```

### Integration Test Cleanup
```typescript
// tests/setup.ts
import { prisma } from '../src/lib/prisma';

beforeEach(async () => {
  // Clean database before each test
  await prisma.notification.deleteMany();
  await prisma.task.deleteMany();
  await prisma.maintenanceRecord.deleteMany();
  await prisma.vehicle.deleteMany();
  await prisma.bill.deleteMany();
  await prisma.transaction.deleteMany();
  await prisma.financialAccount.deleteMany();
  await prisma.calendarEvent.deleteMany();
  await prisma.user.deleteMany();
  await prisma.family.deleteMany();
});

afterAll(async () => {
  await prisma.$disconnect();
});
```

---

**This schema provides a solid foundation for the MVP while allowing for future expansion.**
