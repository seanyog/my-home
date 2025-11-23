# API Specification
## Family House Management Application

**API Version:** v1
**Base URL:** `https://api.housemgmt.app/api/v1` (production)
**Base URL:** `http://localhost:3000/api/v1` (development)
**Protocol:** REST over HTTPS
**Authentication:** JWT Bearer Token

---

## Authentication & Authorization

### Authentication Flow

```
1. User sends credentials → POST /auth/login
2. Server validates and returns JWT access token + refresh token (cookie)
3. Client includes access token in Authorization header for all requests
4. When access token expires (15 min), client uses refresh token
5. Server issues new access token → POST /auth/refresh
```

### Request Headers
```http
Authorization: Bearer <access_token>
Content-Type: application/json
Accept: application/json
```

### Error Responses

All endpoints follow consistent error format:

```json
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
```

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `VALIDATION_ERROR` | 400 | Invalid request data |
| `UNAUTHORIZED` | 401 | Missing or invalid auth token |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `CONFLICT` | 409 | Resource already exists |
| `RATE_LIMIT` | 429 | Too many requests |
| `SERVER_ERROR` | 500 | Internal server error |
| `SERVICE_UNAVAILABLE` | 503 | External service unavailable |

---

## API Endpoints

### Authentication

#### POST /auth/register
Register a new user and create a family.

**Request:**
```json
{
  "email": "sarah@example.com",
  "password": "SecurePass123!",
  "name": "Sarah Smith",
  "familyName": "Smith Family",
  "timezone": "America/New_York"
}
```

**Response:** 201 Created
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user-123",
      "email": "sarah@example.com",
      "name": "Sarah Smith",
      "role": "ADMIN",
      "emailVerified": false
    },
    "family": {
      "id": "fam-456",
      "name": "Smith Family",
      "timezone": "America/New_York"
    },
    "message": "Verification email sent to sarah@example.com"
  },
  "meta": {
    "timestamp": "2025-11-23T10:30:00Z",
    "requestId": "req_abc123"
  }
}
```

#### POST /auth/login
Login with email and password.

**Request:**
```json
{
  "email": "sarah@example.com",
  "password": "SecurePass123!"
}
```

**Response:** 200 OK
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "user-123",
      "email": "sarah@example.com",
      "name": "Sarah Smith",
      "role": "ADMIN",
      "familyId": "fam-456"
    },
    "family": {
      "id": "fam-456",
      "name": "Smith Family"
    }
  },
  "meta": {
    "timestamp": "2025-11-23T10:30:00Z",
    "requestId": "req_abc123"
  }
}
```

Note: `refreshToken` set as HTTP-only cookie

#### POST /auth/refresh
Refresh access token using refresh token cookie.

**Request:** No body (refresh token in cookie)

**Response:** 200 OK
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

#### POST /auth/logout
Logout and invalidate refresh token.

**Response:** 200 OK
```json
{
  "success": true,
  "data": {
    "message": "Logged out successfully"
  }
}
```

#### POST /auth/forgot-password
Request password reset email.

**Request:**
```json
{
  "email": "sarah@example.com"
}
```

**Response:** 200 OK
```json
{
  "success": true,
  "data": {
    "message": "If an account exists, a password reset email has been sent"
  }
}
```

#### POST /auth/reset-password
Reset password using token from email.

**Request:**
```json
{
  "token": "reset-token-abc123",
  "password": "NewSecurePass456!"
}
```

**Response:** 200 OK

---

### Family Management

#### GET /family
Get current user's family details.

**Response:** 200 OK
```json
{
  "success": true,
  "data": {
    "id": "fam-456",
    "name": "Smith Family",
    "timezone": "America/New_York",
    "currency": "USD",
    "settings": {
      "notifications": {
        "billReminderDays": [7, 3, 1]
      }
    },
    "memberCount": 3,
    "createdAt": "2025-01-15T08:00:00Z"
  }
}
```

#### PATCH /family
Update family settings. (Admin only)

**Request:**
```json
{
  "name": "The Smith Family",
  "timezone": "America/Los_Angeles",
  "settings": {
    "notifications": {
      "billReminderDays": [7, 3, 1, 0]
    }
  }
}
```

**Response:** 200 OK

#### GET /family/members
List all family members.

**Response:** 200 OK
```json
{
  "success": true,
  "data": [
    {
      "id": "user-123",
      "name": "Sarah Smith",
      "email": "sarah@example.com",
      "role": "ADMIN",
      "avatarUrl": "https://cdn.example.com/avatars/user-123.jpg",
      "lastActive": "2025-11-23T10:25:00Z"
    },
    {
      "id": "user-456",
      "name": "Michael Smith",
      "email": "michael@example.com",
      "role": "ADULT",
      "avatarUrl": null,
      "lastActive": "2025-11-22T18:00:00Z"
    }
  ]
}
```

#### POST /family/members/invite
Invite a new family member. (Admin only)

**Request:**
```json
{
  "email": "emma@example.com",
  "role": "TEEN"
}
```

**Response:** 201 Created
```json
{
  "success": true,
  "data": {
    "message": "Invitation sent to emma@example.com",
    "expiresAt": "2025-11-30T10:30:00Z"
  }
}
```

#### DELETE /family/members/:userId
Remove a family member. (Admin only)

**Response:** 200 OK

---

### Calendar Events

#### GET /calendar/events
List calendar events with optional filters.

**Query Parameters:**
- `startDate` (ISO 8601): Filter events starting after this date
- `endDate` (ISO 8601): Filter events ending before this date
- `category` (string): Filter by category
- `attendee` (userId): Filter by attendee

**Example:** `GET /calendar/events?startDate=2025-12-01&endDate=2025-12-31`

**Response:** 200 OK
```json
{
  "success": true,
  "data": [
    {
      "id": "evt-123",
      "title": "Soccer Practice",
      "description": "Weekly soccer practice",
      "startTime": "2025-12-01T14:00:00Z",
      "endTime": "2025-12-01T15:30:00Z",
      "allDay": false,
      "category": "ACTIVITIES",
      "location": "Community Soccer Field",
      "attendees": ["user-123", "user-789"],
      "reminders": [
        { "minutesBefore": 60, "type": "email" }
      ],
      "recurrence": {
        "frequency": "weekly",
        "interval": 1,
        "byDay": ["MO"]
      },
      "externalSource": null,
      "createdBy": {
        "id": "user-123",
        "name": "Sarah Smith"
      },
      "createdAt": "2025-11-20T10:00:00Z",
      "updatedAt": "2025-11-20T10:00:00Z"
    }
  ],
  "meta": {
    "total": 15,
    "timestamp": "2025-11-23T10:30:00Z"
  }
}
```

#### POST /calendar/events
Create a new calendar event.

**Request:**
```json
{
  "title": "Dentist Appointment",
  "description": "Annual checkup for Emma",
  "startTime": "2025-12-05T10:00:00Z",
  "endTime": "2025-12-05T11:00:00Z",
  "allDay": false,
  "category": "MEDICAL",
  "location": "Dr. Jones Dental",
  "attendees": ["user-789"],
  "reminders": [
    { "minutesBefore": 1440, "type": "email" },
    { "minutesBefore": 60, "type": "push" }
  ]
}
```

**Response:** 201 Created
```json
{
  "success": true,
  "data": {
    "id": "evt-456",
    "title": "Dentist Appointment",
    ...
  }
}
```

#### GET /calendar/events/:id
Get a specific event.

**Response:** 200 OK

#### PATCH /calendar/events/:id
Update an event. (Creator or admin only)

**Request:**
```json
{
  "startTime": "2025-12-05T14:00:00Z",
  "endTime": "2025-12-05T15:00:00Z"
}
```

**Response:** 200 OK

#### DELETE /calendar/events/:id
Delete an event. (Creator or admin only)

**Response:** 200 OK

#### POST /calendar/sync
Trigger manual calendar sync from external sources.

**Response:** 202 Accepted
```json
{
  "success": true,
  "data": {
    "message": "Sync started",
    "jobId": "job-123"
  }
}
```

#### GET /calendar/integrations
List connected calendar integrations.

**Response:** 200 OK
```json
{
  "success": true,
  "data": [
    {
      "id": "int-123",
      "provider": "GOOGLE_CALENDAR",
      "externalCalendarId": "primary",
      "lastSync": "2025-11-23T09:00:00Z",
      "syncStatus": "SUCCESS",
      "enabled": true
    }
  ]
}
```

#### POST /calendar/integrations
Connect a new calendar integration.

**Request:**
```json
{
  "provider": "GOOGLE_CALENDAR",
  "authorizationCode": "google-auth-code-123"
}
```

**Response:** 201 Created

#### DELETE /calendar/integrations/:id
Disconnect a calendar integration.

**Response:** 200 OK

---

### Financial Accounts

#### GET /financial/accounts
List all linked financial accounts.

**Response:** 200 OK
```json
{
  "success": true,
  "data": [
    {
      "id": "acc-123",
      "institutionName": "Chase Bank",
      "accountType": "CHECKING",
      "accountName": "Primary Checking",
      "accountMask": "****1234",
      "currentBalance": 5420.50,
      "availableBalance": 5420.50,
      "currency": "USD",
      "lastSynced": "2025-11-23T06:00:00Z",
      "syncStatus": "SUCCESS"
    },
    {
      "id": "acc-456",
      "institutionName": "Ally Bank",
      "accountType": "SAVINGS",
      "accountName": "High Yield Savings",
      "accountMask": "****5678",
      "currentBalance": 8500.00,
      "availableBalance": 8500.00,
      "currency": "USD",
      "lastSynced": "2025-11-23T06:00:00Z",
      "syncStatus": "SUCCESS"
    }
  ],
  "meta": {
    "totalBalance": 13920.50
  }
}
```

#### POST /financial/accounts/link
Create Plaid Link token to initiate account linking.

**Response:** 200 OK
```json
{
  "success": true,
  "data": {
    "linkToken": "link-sandbox-abc123...",
    "expiration": "2025-11-23T11:00:00Z"
  }
}
```

#### POST /financial/accounts/exchange
Exchange Plaid public token for access token and link accounts.

**Request:**
```json
{
  "publicToken": "public-sandbox-abc123..."
}
```

**Response:** 201 Created
```json
{
  "success": true,
  "data": {
    "accounts": [
      {
        "id": "acc-789",
        "institutionName": "Wells Fargo",
        "accountType": "CHECKING",
        "accountName": "Everyday Checking",
        "accountMask": "****9012",
        "currentBalance": 2150.75
      }
    ],
    "message": "Accounts linked successfully. Syncing transactions..."
  }
}
```

#### GET /financial/accounts/:id
Get detailed account information.

**Response:** 200 OK

#### DELETE /financial/accounts/:id
Unlink a financial account.

**Response:** 200 OK

#### POST /financial/accounts/:id/sync
Trigger manual sync for a specific account.

**Response:** 202 Accepted
```json
{
  "success": true,
  "data": {
    "message": "Sync started",
    "jobId": "job-456"
  }
}
```

#### GET /financial/transactions
List transactions with optional filters.

**Query Parameters:**
- `accountId` (string): Filter by account
- `startDate` (ISO 8601): Filter transactions after this date
- `endDate` (ISO 8601): Filter transactions before this date
- `category` (string): Filter by category
- `page` (number): Page number (default: 1)
- `limit` (number): Items per page (default: 20, max: 100)

**Example:** `GET /financial/transactions?accountId=acc-123&startDate=2025-11-01&limit=50`

**Response:** 200 OK
```json
{
  "success": true,
  "data": [
    {
      "id": "txn-123",
      "accountId": "acc-123",
      "amount": -85.42,
      "date": "2025-11-22",
      "description": "Whole Foods Market",
      "category": "Groceries",
      "subcategory": "Food & Drink",
      "merchantName": "Whole Foods",
      "pending": false
    },
    {
      "id": "txn-456",
      "accountId": "acc-123",
      "amount": 2500.00,
      "date": "2025-11-21",
      "description": "Payroll Deposit",
      "category": "Income",
      "subcategory": "Paycheck",
      "merchantName": null,
      "pending": false
    }
  ],
  "pagination": {
    "page": 1,
    "perPage": 20,
    "total": 143,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": false
  }
}
```

#### GET /financial/summary
Get financial summary (total balance, spending trends).

**Query Parameters:**
- `period` (string): "week", "month", "year" (default: "month")

**Response:** 200 OK
```json
{
  "success": true,
  "data": {
    "totalBalance": 13920.50,
    "checking": 5420.50,
    "savings": 8500.00,
    "creditCards": -750.00,
    "netWorth": 13170.50,
    "spending": {
      "period": "month",
      "total": 3240.00,
      "byCategory": [
        { "category": "Groceries", "amount": 680.00 },
        { "category": "Utilities", "amount": 450.00 },
        { "category": "Gas", "amount": 280.00 }
      ]
    },
    "income": {
      "period": "month",
      "total": 5000.00
    }
  }
}
```

---

### Bills

#### GET /bills
List bills with optional filters.

**Query Parameters:**
- `status` (string): "PENDING", "PAID", "OVERDUE"
- `startDate` (ISO 8601): Due date after
- `endDate` (ISO 8601): Due date before

**Example:** `GET /bills?status=PENDING&startDate=2025-11-01&endDate=2025-12-31`

**Response:** 200 OK
```json
{
  "success": true,
  "data": [
    {
      "id": "bill-123",
      "name": "Electric Bill - PG&E",
      "category": "UTILITIES",
      "amount": 150.00,
      "currency": "USD",
      "dueDate": "2025-12-15",
      "isRecurring": true,
      "recurrence": {
        "frequency": "monthly",
        "dayOfMonth": 15
      },
      "status": "PENDING",
      "autoPay": true,
      "paymentAccount": {
        "id": "acc-123",
        "name": "Primary Checking"
      },
      "daysUntilDue": 22
    }
  ]
}
```

#### POST /bills
Create a new bill.

**Request:**
```json
{
  "name": "Internet - Comcast",
  "category": "UTILITIES",
  "amount": 80.00,
  "dueDate": "2025-12-20",
  "isRecurring": true,
  "recurrence": {
    "frequency": "monthly",
    "dayOfMonth": 20
  },
  "paymentAccountId": "acc-123",
  "autoPay": false,
  "notes": "Account #12345678"
}
```

**Response:** 201 Created

#### GET /bills/:id
Get a specific bill.

**Response:** 200 OK

#### PATCH /bills/:id
Update a bill.

**Response:** 200 OK

#### DELETE /bills/:id
Delete a bill.

**Response:** 200 OK

#### POST /bills/:id/mark-paid
Mark a bill as paid.

**Request:**
```json
{
  "paidDate": "2025-11-22",
  "paidAmount": 150.00,
  "notes": "Paid via auto-pay"
}
```

**Response:** 200 OK
```json
{
  "success": true,
  "data": {
    "id": "bill-123",
    "status": "PAID",
    "paidDate": "2025-11-22",
    "paidAmount": 150.00,
    "nextOccurrence": {
      "dueDate": "2026-01-15"
    }
  }
}
```

---

### Vehicles

#### GET /vehicles
List all family vehicles.

**Response:** 200 OK
```json
{
  "success": true,
  "data": [
    {
      "id": "veh-123",
      "name": "Family SUV",
      "make": "Toyota",
      "model": "Highlander",
      "year": 2022,
      "color": "Blue",
      "vin": "1HGBH41JXMN109186",
      "licensePlate": "ABC1234",
      "currentMileage": 25000,
      "photoUrl": "https://cdn.example.com/vehicles/veh-123.jpg",
      "insurance": {
        "provider": "State Farm",
        "policyNumber": "SF-123456",
        "renewalDate": "2026-03-15",
        "annualCost": 1800
      },
      "registration": {
        "expirationDate": "2026-03-31",
        "renewalCost": 120
      },
      "upcomingMaintenance": [
        {
          "type": "Oil Change",
          "dueDate": "2026-02-20",
          "dueMileage": 27500
        }
      ]
    }
  ]
}
```

#### POST /vehicles
Add a new vehicle.

**Request:**
```json
{
  "name": "Family SUV",
  "make": "Toyota",
  "model": "Highlander",
  "year": 2022,
  "color": "Blue",
  "vin": "1HGBH41JXMN109186",
  "licensePlate": "ABC1234",
  "currentMileage": 25000,
  "purchaseDate": "2022-03-15",
  "purchasePrice": 45000,
  "insurance": {
    "provider": "State Farm",
    "policyNumber": "SF-123456",
    "renewalDate": "2026-03-15",
    "annualCost": 1800
  },
  "registration": {
    "expirationDate": "2026-03-31",
    "renewalCost": 120
  }
}
```

**Response:** 201 Created

#### GET /vehicles/:id
Get detailed vehicle information.

**Response:** 200 OK

#### PATCH /vehicles/:id
Update vehicle information.

**Response:** 200 OK

#### DELETE /vehicles/:id
Delete a vehicle.

**Response:** 200 OK

#### GET /vehicles/:id/maintenance
List maintenance history for a vehicle.

**Response:** 200 OK
```json
{
  "success": true,
  "data": [
    {
      "id": "maint-123",
      "vehicleId": "veh-123",
      "serviceType": "Oil Change",
      "date": "2025-11-20",
      "mileage": 24500,
      "cost": 75.00,
      "serviceProvider": "Quick Lube Express",
      "providerPhone": "(555) 123-4567",
      "notes": "Full synthetic oil, new oil filter",
      "receiptUrl": "https://cdn.example.com/receipts/maint-123.pdf",
      "nextServiceDue": {
        "date": "2026-02-20",
        "mileage": 27500,
        "type": "Oil Change"
      }
    }
  ]
}
```

#### POST /vehicles/:id/maintenance
Log a maintenance record.

**Request:**
```json
{
  "serviceType": "Oil Change",
  "date": "2025-11-20",
  "mileage": 24500,
  "cost": 75.00,
  "serviceProvider": "Quick Lube Express",
  "providerPhone": "(555) 123-4567",
  "notes": "Full synthetic oil",
  "nextServiceDue": {
    "date": "2026-02-20",
    "mileage": 27500
  }
}
```

**Response:** 201 Created

#### GET /vehicles/:id/costs
Get cost analytics for a vehicle.

**Query Parameters:**
- `year` (number): Filter by year (default: current year)

**Response:** 200 OK
```json
{
  "success": true,
  "data": {
    "year": 2025,
    "totalCost": 2450.00,
    "breakdown": {
      "maintenance": 500.00,
      "insurance": 1800.00,
      "registration": 120.00,
      "fuel": 0 // Not tracked in MVP
    },
    "costPerMile": 0.098,
    "milesDriven": 25000,
    "maintenanceHistory": [
      { "month": "November", "cost": 75.00 },
      { "month": "August", "cost": 50.00 }
    ]
  }
}
```

---

### Tasks

#### GET /tasks
List tasks with optional filters.

**Query Parameters:**
- `status` (string): "TODO", "IN_PROGRESS", "COMPLETED"
- `assigneeId` (string): Filter by assignee
- `priority` (string): "LOW", "MEDIUM", "HIGH", "URGENT"

**Response:** 200 OK
```json
{
  "success": true,
  "data": [
    {
      "id": "task-123",
      "title": "Pick up groceries",
      "description": "Get items from shopping list",
      "assignedTo": {
        "id": "user-456",
        "name": "Michael Smith"
      },
      "createdBy": {
        "id": "user-123",
        "name": "Sarah Smith"
      },
      "dueDate": "2025-11-23T18:00:00Z",
      "priority": "MEDIUM",
      "status": "TODO",
      "checklist": [
        { "item": "Milk", "completed": false },
        { "item": "Bread", "completed": false }
      ],
      "createdAt": "2025-11-23T14:00:00Z",
      "hoursUntilDue": 4
    }
  ]
}
```

#### POST /tasks
Create a new task.

**Request:**
```json
{
  "title": "Pick up groceries",
  "description": "Get items from shopping list",
  "assignedToId": "user-456",
  "dueDate": "2025-11-23T18:00:00Z",
  "priority": "MEDIUM",
  "checklist": [
    { "item": "Milk", "completed": false },
    { "item": "Bread", "completed": false }
  ]
}
```

**Response:** 201 Created

#### GET /tasks/:id
Get a specific task.

**Response:** 200 OK

#### PATCH /tasks/:id
Update a task.

**Request:**
```json
{
  "status": "IN_PROGRESS",
  "checklist": [
    { "item": "Milk", "completed": true },
    { "item": "Bread", "completed": false }
  ]
}
```

**Response:** 200 OK

#### DELETE /tasks/:id
Delete a task.

**Response:** 200 OK

#### POST /tasks/:id/complete
Mark task as completed.

**Response:** 200 OK
```json
{
  "success": true,
  "data": {
    "id": "task-123",
    "status": "COMPLETED",
    "completedAt": "2025-11-23T17:45:00Z",
    "completedBy": "user-456"
  }
}
```

---

### Dashboard

#### GET /dashboard
Get unified dashboard data.

**Response:** 200 OK
```json
{
  "success": true,
  "data": {
    "today": {
      "events": [
        {
          "id": "evt-123",
          "title": "Soccer Practice",
          "startTime": "2025-11-23T14:00:00Z",
          "endTime": "2025-11-23T15:30:00Z",
          "category": "ACTIVITIES"
        }
      ],
      "tasks": [
        {
          "id": "task-123",
          "title": "Pick up groceries",
          "dueDate": "2025-11-23T18:00:00Z",
          "assignedTo": { "id": "user-456", "name": "Michael" }
        }
      ]
    },
    "thisWeek": {
      "billsDue": [
        {
          "id": "bill-123",
          "name": "Electric Bill",
          "amount": 150.00,
          "dueDate": "2025-11-28",
          "daysUntilDue": 5
        }
      ],
      "upcomingEvents": 8,
      "tasksDue": 3
    },
    "financial": {
      "totalBalance": 13920.50,
      "monthlySpending": 3240.00,
      "monthlyBudget": 4000.00,
      "budgetUsedPercent": 81
    },
    "vehicles": [
      {
        "id": "veh-123",
        "name": "Family SUV",
        "upcomingMaintenance": [
          {
            "type": "Oil Change",
            "dueDate": "2026-02-20",
            "daysUntilDue": 89
          }
        ]
      }
    ]
  }
}
```

---

### User Profile

#### GET /users/me
Get current user's profile.

**Response:** 200 OK
```json
{
  "success": true,
  "data": {
    "id": "user-123",
    "email": "sarah@example.com",
    "name": "Sarah Smith",
    "role": "ADMIN",
    "avatarUrl": "https://cdn.example.com/avatars/user-123.jpg",
    "phone": null,
    "preferences": {
      "notificationChannels": ["email", "push"],
      "calendarView": "month",
      "theme": "light"
    },
    "family": {
      "id": "fam-456",
      "name": "Smith Family"
    },
    "lastLogin": "2025-11-23T10:00:00Z"
  }
}
```

#### PATCH /users/me
Update current user's profile.

**Request:**
```json
{
  "name": "Sarah Johnson",
  "phone": "+1-555-123-4567",
  "preferences": {
    "notificationChannels": ["email", "push", "sms"],
    "theme": "dark"
  }
}
```

**Response:** 200 OK

#### POST /users/me/avatar
Upload profile avatar.

**Request:** `multipart/form-data`
```
file: <image file>
```

**Response:** 200 OK
```json
{
  "success": true,
  "data": {
    "avatarUrl": "https://cdn.example.com/avatars/user-123.jpg"
  }
}
```

#### POST /users/me/change-password
Change password.

**Request:**
```json
{
  "currentPassword": "OldPass123!",
  "newPassword": "NewSecurePass456!"
}
```

**Response:** 200 OK

---

## Rate Limiting

### Limits (per user)

| Endpoint Group | Limit | Window |
|----------------|-------|--------|
| Auth (login, register) | 5 requests | 15 minutes |
| General API | 100 requests | 1 minute |
| Financial sync | 10 requests | 1 hour |
| File uploads | 20 requests | 1 hour |

### Rate Limit Headers
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1638360000
```

### Rate Limit Exceeded Response
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT",
    "message": "Too many requests. Please try again later.",
    "retryAfter": 42
  }
}
```

---

## Webhooks (Future)

### Plaid Webhook
```
POST /webhooks/plaid
```

Receives transaction updates from Plaid.

---

## API Versioning

- Current version: `v1`
- Version specified in URL: `/api/v1/...`
- Breaking changes require new version
- Deprecated endpoints supported for 6 months

---

## Development Tools

### Postman Collection
Available at: `docs/postman/house-management-api.json`

### OpenAPI Spec
Available at: `docs/openapi.yaml`

### API Documentation (Swagger)
Available at: `http://localhost:3000/api-docs` (development)

---

**This API provides a comprehensive interface for all MVP features while maintaining REST best practices and extensibility.**
