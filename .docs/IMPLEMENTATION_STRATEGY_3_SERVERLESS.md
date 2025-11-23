# Implementation Strategy 3: Serverless Cloud-Native Architecture

**Approach:** Event-driven, function-based architecture with managed cloud services
**Target:** Variable scale (0 to 100,000+ families)
**Development Timeline:** 7-10 months for MVP
**Team Size:** 4-8 developers

---

## Strategy Overview

Build the application using serverless functions (AWS Lambda, Google Cloud Functions, or Azure Functions) and fully managed cloud services. Pay only for what you use, auto-scale to zero, and eliminate server management entirely.

### Philosophy
- **No Server Management**: Focus 100% on business logic
- **Pay-Per-Use**: Cost scales with usage, near-zero cost when idle
- **Infinite Scalability**: Automatic scaling from 0 to millions of requests
- **Managed Services**: Use cloud provider's databases, queues, storage
- **Event-Driven**: Functions triggered by events (HTTP, database changes, schedules)

---

## High-Level Architecture (AWS Example)

```
┌─────────────────────────────────────────────────────────────────┐
│                     CLIENT LAYER                                 │
│  Next.js (SSR) on Vercel / CloudFront + S3 (Static)            │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         │ HTTPS
                         │
┌────────────────────────▼────────────────────────────────────────┐
│              AWS API Gateway (REST / HTTP API)                  │
│  - Request routing    - JWT authorization    - Throttling       │
│  - CORS handling      - Request validation   - Caching          │
└───┬──────────┬─────────┬──────────┬─────────┬──────────────────┘
    │          │         │          │         │
┌───▼────┐ ┌──▼────┐ ┌──▼─────┐ ┌──▼────┐ ┌──▼────┐
│ Auth   │ │Calendar│ │Financial│ │Vehicle│ │ Task  │
│Lambda  │ │Lambda  │ │Lambda  │ │Lambda │ │Lambda │
│Functions│ │Functions│ │Functions│ │Functions│ │Functions│
└───┬────┘ └───┬────┘ └───┬────┘ └───┬───┘ └───┬───┘
    │          │          │          │         │
    │          │          │          │         │
┌───▼──────────▼──────────▼──────────▼─────────▼───────────────┐
│                    EventBridge / SNS / SQS                    │
│  Event bus for asynchronous communication                     │
└───────────────────────┬───────────────────────────────────────┘
                        │
            ┌───────────┴────────────┐
            │                        │
┌───────────▼──────┐     ┌──────────▼──────────┐
│  DynamoDB        │     │  Aurora Serverless  │
│  (NoSQL)         │     │  PostgreSQL         │
│  - Fast reads    │     │  - Relational data  │
│  - Auto-scaling  │     │  - Auto-scaling     │
└──────────────────┘     └─────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│               Supporting AWS Services                        │
├────────────────┬─────────────────┬──────────────────────────┬┤
│  S3            │  SES (Email)    │  Step Functions          ││
│  (Documents)   │  SNS (Push)     │  (Workflows)             ││
└────────────────┴─────────────────┴──────────────────────────┴┘
```

---

## Technology Stack

### Frontend
```
Framework:          Next.js 14+ (React with SSR/SSG)
Hosting:            Vercel / AWS Amplify / CloudFront + S3
State Management:   TanStack Query + Zustand
Styling:            Tailwind CSS + shadcn/ui
Authentication:     AWS Cognito / Auth0 / Supabase Auth
```

**Why Next.js + Vercel:**
- Built-in SSR for SEO and performance
- Edge functions for API routes
- Automatic deployments
- Global CDN
- Zero configuration

### Backend (AWS Stack)

```
Compute:            AWS Lambda (Node.js, Python, Go)
API Layer:          AWS API Gateway (HTTP API or REST API)
Authentication:     AWS Cognito (user pools + identity pools)
Database (SQL):     Aurora Serverless v2 (PostgreSQL)
Database (NoSQL):   DynamoDB
Cache:              DynamoDB DAX / ElastiCache Serverless
Object Storage:     S3
Message Queue:      SQS + SNS
Event Bus:          EventBridge
Email:              SES (Simple Email Service)
SMS:                SNS
Workflows:          Step Functions
Scheduled Jobs:     EventBridge Scheduler
Monitoring:         CloudWatch + X-Ray
Secrets:            Secrets Manager
File Processing:    Lambda + S3 triggers
```

### Alternative Cloud Providers

**Google Cloud Platform (GCP):**
- Compute: Cloud Functions / Cloud Run
- API: Cloud Endpoints / API Gateway
- Database: Cloud SQL (PostgreSQL) / Firestore
- Auth: Firebase Authentication
- Storage: Cloud Storage
- Messaging: Pub/Sub
- Workflows: Cloud Workflows

**Azure:**
- Compute: Azure Functions
- API: Azure API Management
- Database: Azure SQL / Cosmos DB
- Auth: Azure AD B2C
- Storage: Blob Storage
- Messaging: Service Bus / Event Grid

---

## Architecture Deep Dive

### 1. Authentication (AWS Cognito)

**Setup:**
```
User Pool:
  - Email/password authentication
  - MFA with TOTP
  - OAuth 2.0 / OIDC
  - Custom attributes (family_id, role)

Identity Pool:
  - Federated identities (Google, Apple)
  - Temporary AWS credentials
  - Fine-grained IAM permissions
```

**User Flow:**
```
1. User signs up → Cognito User Pool
2. Email verification
3. User logs in → Cognito returns JWT tokens
4. Frontend stores tokens
5. API requests include JWT in Authorization header
6. API Gateway validates JWT with Cognito
7. Lambda receives user context (user_id, family_id, role)
```

**Lambda Authorizer (Custom Logic):**
```typescript
// For complex authorization rules
export const handler = async (event) => {
  const token = event.authorizationToken;

  // Verify JWT
  const decoded = await verifyJWT(token);

  // Check additional permissions
  const user = await getUserFromDB(decoded.sub);

  if (!user.active) {
    throw new Error('Unauthorized');
  }

  // Return IAM policy
  return generatePolicy(decoded.sub, 'Allow', event.methodArn, {
    userId: decoded.sub,
    familyId: user.family_id,
    role: user.role
  });
};
```

---

### 2. API Layer (API Gateway + Lambda)

#### API Gateway Configuration

**HTTP API (Recommended):**
- Lower cost (70% cheaper than REST API)
- Better performance
- Native JWT authorizers
- CORS built-in
- WebSocket support

**REST API (More Features):**
- Request/response transformation
- API keys
- Usage plans
- Request validation
- More detailed monitoring

#### Lambda Function Structure

**Per-Resource Functions:**
```
/api/calendar/events
  GET    → listEventsFunction
  POST   → createEventFunction

/api/calendar/events/{id}
  GET    → getEventFunction
  PATCH  → updateEventFunction
  DELETE → deleteEventFunction
```

**Monolithic Lambda (Alternative):**
```
/api/calendar/*
  ALL methods → calendarServiceFunction (routes internally)
```

**Function Example:**
```typescript
// functions/calendar/createEvent.ts
import { APIGatewayProxyHandlerV2 } from 'aws-lambda';
import { DynamoDB } from '@aws-sdk/client-dynamodb';
import { EventBridge } from '@aws-sdk/client-eventbridge';

const db = new DynamoDB.DocumentClient();
const eventBridge = new EventBridge();

export const handler: APIGatewayProxyHandlerV2 = async (event) => {
  // Extract user context from authorizer
  const userId = event.requestContext.authorizer.jwt.claims.sub;
  const familyId = event.requestContext.authorizer.jwt.claims['custom:family_id'];

  // Parse request body
  const body = JSON.parse(event.body || '{}');

  // Validate input (use Zod or similar)
  const eventData = {
    id: generateId(),
    familyId,
    createdBy: userId,
    title: body.title,
    startTime: body.startTime,
    endTime: body.endTime,
    category: body.category,
    createdAt: new Date().toISOString()
  };

  // Store in DynamoDB
  await db.put({
    TableName: process.env.EVENTS_TABLE,
    Item: eventData
  }).promise();

  // Publish event to EventBridge
  await eventBridge.putEvents({
    Entries: [{
      Source: 'calendar-service',
      DetailType: 'event.created',
      Detail: JSON.stringify(eventData),
      EventBusName: process.env.EVENT_BUS_NAME
    }]
  }).promise();

  return {
    statusCode: 201,
    body: JSON.stringify({ success: true, data: eventData })
  };
};
```

---

### 3. Database Strategy

#### DynamoDB (NoSQL) - Primary Choice

**Why DynamoDB:**
- True serverless (auto-scaling)
- Single-digit millisecond latency
- Pay per request (no idle cost)
- Built-in caching (DAX)
- Global tables for multi-region
- Streams for change data capture

**Data Modeling (Single Table Design):**

```
Table: house_management

PK (Partition Key)    | SK (Sort Key)           | Attributes
--------------------- | ----------------------- | --------------------
FAMILY#123            | METADATA                | name, settings, created_at
FAMILY#123            | USER#user-456           | email, name, role, preferences
FAMILY#123            | EVENT#2025-12-01#evt-1  | title, start, end, category
FAMILY#123            | EVENT#2025-12-15#evt-2  | title, start, end, category
FAMILY#123            | BILL#bill-789           | name, amount, due_date, status
FAMILY#123            | VEHICLE#veh-101         | make, model, year, mileage
FAMILY#123            | TASK#task-202           | title, assigned_to, due_date

USER#user-456         | PROFILE                 | email, name, family_id (GSI)
```

**Access Patterns:**
```typescript
// Get all events for a family in date range
const params = {
  TableName: 'house_management',
  KeyConditionExpression: 'PK = :pk AND SK BETWEEN :start AND :end',
  ExpressionAttributeValues: {
    ':pk': 'FAMILY#123',
    ':start': 'EVENT#2025-12-01',
    ':end': 'EVENT#2025-12-31'
  }
};

// Query by user email (GSI)
const params = {
  TableName: 'house_management',
  IndexName: 'UserEmailIndex',
  KeyConditionExpression: 'email = :email',
  ExpressionAttributeValues: {
    ':email': 'user@example.com'
  }
};
```

**Global Secondary Indexes (GSI):**
```
1. UserEmailIndex: email (PK) → quick user lookup
2. TaskAssigneeIndex: assigned_to (PK), due_date (SK) → user's tasks
3. BillDueDateIndex: family_id (PK), due_date (SK) → upcoming bills
```

#### Aurora Serverless v2 (PostgreSQL) - Alternative

**When to use:**
- Complex relational queries needed
- Strong ACID guarantees required
- Existing PostgreSQL knowledge
- Multi-table joins common

**Scaling:**
```
Aurora Serverless v2:
  - Auto-scales from 0.5 to 128 ACUs (Aurora Capacity Units)
  - Scales in seconds (not minutes)
  - Pause when idle (only charged for storage)
  - PostgreSQL compatible
```

**Connection Management:**
```typescript
// Use RDS Proxy to pool connections (Lambda creates many connections)
import { Client } from 'pg';

const client = new Client({
  host: process.env.RDS_PROXY_ENDPOINT,
  database: 'house_management',
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD
});

// Or use Data API (HTTP-based, no connection management)
import { RDSDataClient, ExecuteStatementCommand } from '@aws-sdk/client-rds-data';

const rdsData = new RDSDataClient({ region: 'us-east-1' });
const result = await rdsData.send(new ExecuteStatementCommand({
  resourceArn: process.env.DB_CLUSTER_ARN,
  secretArn: process.env.DB_SECRET_ARN,
  database: 'house_management',
  sql: 'SELECT * FROM events WHERE family_id = :familyId',
  parameters: [{ name: 'familyId', value: { stringValue: familyId } }]
}));
```

---

### 4. Event-Driven Architecture

#### EventBridge (Event Bus)

**Event Schema:**
```json
{
  "version": "0",
  "id": "event-id-123",
  "detail-type": "event.created",
  "source": "calendar-service",
  "time": "2025-11-23T10:30:00Z",
  "region": "us-east-1",
  "resources": [],
  "detail": {
    "eventId": "evt-123",
    "familyId": "fam-456",
    "title": "Soccer Practice",
    "startTime": "2025-12-01T14:00:00Z",
    "category": "activities",
    "createdBy": "user-789"
  }
}
```

**Event Rules (Subscribers):**
```json
{
  "Rules": [
    {
      "Name": "SendEventReminderRule",
      "EventPattern": {
        "source": ["calendar-service"],
        "detail-type": ["event.created", "event.updated"]
      },
      "Targets": [
        {
          "Arn": "arn:aws:lambda:...:function:scheduleReminderFunction",
          "Id": "1"
        }
      ]
    },
    {
      "Name": "BillDueReminderRule",
      "EventPattern": {
        "source": ["financial-service"],
        "detail-type": ["bill.due_soon"]
      },
      "Targets": [
        {
          "Arn": "arn:aws:lambda:...:function:sendNotificationFunction",
          "Id": "1"
        }
      ]
    }
  ]
}
```

**Lambda Event Handler:**
```typescript
// functions/notifications/sendReminder.ts
import { EventBridgeHandler } from 'aws-lambda';

export const handler: EventBridgeHandler<'event.created', EventDetail, void> = async (event) => {
  const { eventId, familyId, title, startTime } = event.detail;

  // Get family members
  const users = await getFamilyMembers(familyId);

  // Schedule reminder (e.g., 1 hour before)
  const reminderTime = new Date(startTime);
  reminderTime.setHours(reminderTime.getHours() - 1);

  for (const user of users) {
    await scheduleNotification({
      userId: user.id,
      type: 'event_reminder',
      scheduledFor: reminderTime,
      payload: { eventId, title }
    });
  }
};
```

#### SQS (Queuing for Reliability)

**Use Cases:**
- Buffering between services
- Retry logic for failed operations
- Rate limiting API calls (e.g., Plaid API)

**Example: Plaid Transaction Sync Queue**
```typescript
// Producer: Add sync job to queue
import { SQS } from '@aws-sdk/client-sqs';

const sqs = new SQS();

await sqs.sendMessage({
  QueueUrl: process.env.PLAID_SYNC_QUEUE_URL,
  MessageBody: JSON.stringify({
    familyId: 'fam-123',
    accountId: 'acc-456',
    accessToken: 'encrypted-token'
  }),
  MessageAttributes: {
    priority: {
      DataType: 'Number',
      StringValue: '1'
    }
  }
});

// Consumer: Lambda triggered by SQS
export const handler: SQSHandler = async (event) => {
  for (const record of event.Records) {
    const { familyId, accountId, accessToken } = JSON.parse(record.body);

    try {
      await syncPlaidTransactions(accessToken, accountId);
      // Message auto-deleted on success
    } catch (error) {
      // Message goes to DLQ after max retries
      console.error('Sync failed:', error);
      throw error; // Triggers retry
    }
  }
};
```

**Dead Letter Queue (DLQ):**
- Failed messages after max retries
- Alarm on DLQ depth
- Manual investigation and replay

---

### 5. Background Jobs & Scheduled Tasks

#### EventBridge Scheduler

**Cron-based Schedules:**
```json
{
  "Name": "DailyBillReminderCheck",
  "ScheduleExpression": "cron(0 9 * * ? *)",
  "Target": {
    "Arn": "arn:aws:lambda:...:function:checkBillReminders",
    "RoleArn": "arn:aws:iam::...:role/EventBridgeRole"
  }
}
```

**One-Time Scheduled Events:**
```typescript
// Schedule a reminder for specific date/time
import { Scheduler } from '@aws-sdk/client-scheduler';

const scheduler = new Scheduler();

await scheduler.createSchedule({
  Name: `reminder-${eventId}`,
  ScheduleExpression: `at(${reminderTime.toISOString()})`,
  FlexibleTimeWindow: { Mode: 'OFF' },
  Target: {
    Arn: process.env.SEND_NOTIFICATION_FUNCTION_ARN,
    RoleArn: process.env.SCHEDULER_ROLE_ARN,
    Input: JSON.stringify({
      userId: 'user-123',
      type: 'event_reminder',
      eventId: 'evt-456'
    })
  }
});
```

#### Step Functions (Complex Workflows)

**Use Case: Plaid Account Linking Flow**

```json
{
  "Comment": "Plaid account linking workflow",
  "StartAt": "ExchangeToken",
  "States": {
    "ExchangeToken": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:function:exchangePlaidToken",
      "Next": "FetchAccounts"
    },
    "FetchAccounts": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:function:fetchPlaidAccounts",
      "Next": "StoreAccounts"
    },
    "StoreAccounts": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:function:storeAccounts",
      "Next": "SyncInitialTransactions"
    },
    "SyncInitialTransactions": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:function:syncTransactions",
      "Retry": [
        {
          "ErrorEquals": ["PlaidRateLimitError"],
          "IntervalSeconds": 60,
          "MaxAttempts": 3,
          "BackoffRate": 2.0
        }
      ],
      "Catch": [
        {
          "ErrorEquals": ["States.ALL"],
          "Next": "NotifyLinkingFailed"
        }
      ],
      "End": true
    },
    "NotifyLinkingFailed": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:function:sendNotification",
      "End": true
    }
  }
}
```

**Benefits:**
- Visual workflow editor
- Built-in retry and error handling
- Long-running workflows (up to 1 year)
- Human approval steps
- Parallel execution

---

### 6. External Integrations

#### Plaid Integration

```typescript
// functions/financial/linkAccount.ts
import { Configuration, PlaidApi, PlaidEnvironments } from 'plaid';

const plaidClient = new PlaidApi(new Configuration({
  basePath: PlaidEnvironments[process.env.PLAID_ENV],
  baseOptions: {
    headers: {
      'PLAID-CLIENT-ID': process.env.PLAID_CLIENT_ID,
      'PLAID-SECRET': process.env.PLAID_SECRET
    }
  }
}));

export const createLinkToken = async (userId: string) => {
  const response = await plaidClient.linkTokenCreate({
    user: { client_user_id: userId },
    client_name: 'Family House Management',
    products: ['transactions'],
    country_codes: ['US'],
    language: 'en'
  });

  return response.data.link_token;
};

export const exchangePublicToken = async (publicToken: string) => {
  const response = await plaidClient.itemPublicTokenExchange({
    public_token: publicToken
  });

  // Store access token in Secrets Manager
  await storeSecret(`plaid-token-${response.data.item_id}`, response.data.access_token);

  return response.data.item_id;
};

// Webhook handler for Plaid events
export const webhookHandler: APIGatewayProxyHandlerV2 = async (event) => {
  const { webhook_type, webhook_code, item_id } = JSON.parse(event.body || '{}');

  if (webhook_type === 'TRANSACTIONS' && webhook_code === 'DEFAULT_UPDATE') {
    // Queue sync job
    await sqs.sendMessage({
      QueueUrl: process.env.PLAID_SYNC_QUEUE_URL,
      MessageBody: JSON.stringify({ itemId: item_id })
    });
  }

  return { statusCode: 200, body: 'OK' };
};
```

#### Calendar Integration (Google Calendar)

```typescript
// functions/calendar/syncGoogle.ts
import { google } from 'googleapis';
import { OAuth2Client } from 'google-auth-library';

export const syncGoogleCalendar = async (userId: string, familyId: string) => {
  // Retrieve user's OAuth tokens from Secrets Manager
  const tokens = await getSecret(`google-calendar-${userId}`);

  const oauth2Client = new OAuth2Client(
    process.env.GOOGLE_CLIENT_ID,
    process.env.GOOGLE_CLIENT_SECRET
  );
  oauth2Client.setCredentials(JSON.parse(tokens));

  const calendar = google.calendar({ version: 'v3', auth: oauth2Client });

  const response = await calendar.events.list({
    calendarId: 'primary',
    timeMin: new Date().toISOString(),
    maxResults: 100,
    singleEvents: true,
    orderBy: 'startTime'
  });

  const events = response.data.items || [];

  // Batch write to DynamoDB
  const batch = events.map(event => ({
    PutRequest: {
      Item: {
        PK: `FAMILY#${familyId}`,
        SK: `EVENT#${event.start?.dateTime || event.start?.date}#${event.id}`,
        title: event.summary,
        startTime: event.start?.dateTime || event.start?.date,
        endTime: event.end?.dateTime || event.end?.date,
        externalSource: 'google_calendar',
        externalId: event.id
      }
    }
  }));

  await db.batchWrite({
    RequestItems: {
      [process.env.DYNAMODB_TABLE]: batch
    }
  }).promise();
};
```

---

### 7. File Storage & Processing

#### S3 + Lambda Triggers

**Document Upload Flow:**
```
1. Frontend gets pre-signed URL from Lambda
2. Frontend uploads directly to S3
3. S3 triggers Lambda on upload
4. Lambda processes file (virus scan, OCR, thumbnail)
5. Lambda stores metadata in DynamoDB
```

**Implementation:**
```typescript
// functions/documents/getUploadUrl.ts
import { S3 } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';

const s3 = new S3();

export const handler: APIGatewayProxyHandlerV2 = async (event) => {
  const familyId = event.requestContext.authorizer.jwt.claims['custom:family_id'];
  const { fileName, fileType } = JSON.parse(event.body || '{}');

  const key = `families/${familyId}/documents/${generateId()}-${fileName}`;

  const command = new PutObjectCommand({
    Bucket: process.env.DOCUMENTS_BUCKET,
    Key: key,
    ContentType: fileType
  });

  const uploadUrl = await getSignedUrl(s3, command, { expiresIn: 300 });

  return {
    statusCode: 200,
    body: JSON.stringify({ uploadUrl, key })
  };
};

// functions/documents/processUpload.ts (S3 trigger)
import { S3Handler } from 'aws-lambda';
import { Textract } from '@aws-sdk/client-textract';

const textract = new Textract();

export const handler: S3Handler = async (event) => {
  for (const record of event.Records) {
    const bucket = record.s3.bucket.name;
    const key = record.s3.object.key;

    // Extract text with Textract (OCR)
    const textractResult = await textract.detectDocumentText({
      Document: {
        S3Object: { Bucket: bucket, Name: key }
      }
    });

    const extractedText = textractResult.Blocks
      ?.filter(block => block.BlockType === 'LINE')
      .map(block => block.Text)
      .join('\n');

    // Store metadata in DynamoDB
    const [, familyId, , documentId] = key.split('/');

    await db.put({
      TableName: process.env.DYNAMODB_TABLE,
      Item: {
        PK: `FAMILY#${familyId}`,
        SK: `DOCUMENT#${documentId}`,
        name: record.s3.object.key.split('/').pop(),
        storageKey: key,
        fileSize: record.s3.object.size,
        extractedText,
        uploadedAt: new Date().toISOString()
      }
    }).promise();
  }
};
```

---

### 8. Notifications

#### SES (Email)

```typescript
// functions/notifications/sendEmail.ts
import { SES } from '@aws-sdk/client-ses';

const ses = new SES();

export const sendEmail = async (to: string, subject: string, body: string) => {
  await ses.sendEmail({
    Source: process.env.FROM_EMAIL,
    Destination: { ToAddresses: [to] },
    Message: {
      Subject: { Data: subject },
      Body: { Html: { Data: body } }
    }
  });
};

// Template-based emails
export const sendTemplatedEmail = async (to: string, template: string, data: any) => {
  await ses.sendTemplatedEmail({
    Source: process.env.FROM_EMAIL,
    Destination: { ToAddresses: [to] },
    Template: template,
    TemplateData: JSON.stringify(data)
  });
};
```

#### SNS (Push Notifications)

```typescript
// functions/notifications/sendPush.ts
import { SNS } from '@aws-sdk/client-sns';

const sns = new SNS();

export const sendPushNotification = async (userId: string, message: string) => {
  // Get user's device endpoints from DynamoDB
  const endpoints = await getUserDeviceEndpoints(userId);

  for (const endpoint of endpoints) {
    await sns.publish({
      TargetArn: endpoint.arn,
      Message: JSON.stringify({
        default: message,
        APNS: JSON.stringify({ aps: { alert: message } }),
        GCM: JSON.stringify({ data: { message } })
      }),
      MessageStructure: 'json'
    });
  }
};
```

---

## Infrastructure as Code (AWS CDK)

### Project Structure

```
infrastructure/
  ├── bin/
  │   └── app.ts                 # CDK app entry point
  ├── lib/
  │   ├── stacks/
  │   │   ├── auth-stack.ts      # Cognito, IAM
  │   │   ├── api-stack.ts       # API Gateway, Lambda
  │   │   ├── database-stack.ts  # DynamoDB, Aurora
  │   │   ├── storage-stack.ts   # S3, CloudFront
  │   │   └── monitoring-stack.ts # CloudWatch, X-Ray
  │   ├── constructs/
  │   │   ├── lambda-function.ts # Reusable Lambda construct
  │   │   └── api-endpoint.ts    # API + Lambda combo
  │   └── config/
  │       ├── dev.ts
  │       ├── staging.ts
  │       └── prod.ts
  ├── package.json
  └── tsconfig.json
```

### Example Stack (API + Lambda)

```typescript
// lib/stacks/calendar-stack.ts
import * as cdk from 'aws-cdk-lib';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as apigateway from 'aws-cdk-lib/aws-apigatewayv2';
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';
import { Construct } from 'constructs';

export class CalendarStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // DynamoDB Table
    const table = new dynamodb.Table(this, 'EventsTable', {
      partitionKey: { name: 'PK', type: dynamodb.AttributeType.STRING },
      sortKey: { name: 'SK', type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
      encryption: dynamodb.TableEncryption.AWS_MANAGED,
      pointInTimeRecovery: true
    });

    // Lambda Layer (shared dependencies)
    const dependenciesLayer = new lambda.LayerVersion(this, 'DependenciesLayer', {
      code: lambda.Code.fromAsset('layers/dependencies'),
      compatibleRuntimes: [lambda.Runtime.NODEJS_20_X]
    });

    // Lambda Functions
    const listEventsFunction = new lambda.Function(this, 'ListEventsFunction', {
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: 'listEvents.handler',
      code: lambda.Code.fromAsset('functions/calendar'),
      layers: [dependenciesLayer],
      environment: {
        TABLE_NAME: table.tableName
      },
      timeout: cdk.Duration.seconds(10),
      memorySize: 512
    });

    const createEventFunction = new lambda.Function(this, 'CreateEventFunction', {
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: 'createEvent.handler',
      code: lambda.Code.fromAsset('functions/calendar'),
      layers: [dependenciesLayer],
      environment: {
        TABLE_NAME: table.tableName,
        EVENT_BUS_NAME: 'house-management-events'
      }
    });

    // Grant permissions
    table.grantReadData(listEventsFunction);
    table.grantReadWriteData(createEventFunction);

    // API Gateway
    const api = new apigateway.HttpApi(this, 'CalendarApi', {
      apiName: 'calendar-service',
      corsPreflight: {
        allowOrigins: ['https://app.example.com'],
        allowMethods: [apigateway.CorsHttpMethod.GET, apigateway.CorsHttpMethod.POST],
        allowHeaders: ['Authorization', 'Content-Type']
      }
    });

    // Cognito Authorizer
    const authorizer = new apigateway.HttpJwtAuthorizer('CognitoAuthorizer', {
      jwtAudience: [process.env.COGNITO_CLIENT_ID!],
      jwtIssuer: `https://cognito-idp.${this.region}.amazonaws.com/${process.env.COGNITO_USER_POOL_ID}`
    });

    // Routes
    api.addRoutes({
      path: '/events',
      methods: [apigateway.HttpMethod.GET],
      integration: new apigateway.HttpLambdaIntegration('ListEventsIntegration', listEventsFunction),
      authorizer
    });

    api.addRoutes({
      path: '/events',
      methods: [apigateway.HttpMethod.POST],
      integration: new apigateway.HttpLambdaIntegration('CreateEventIntegration', createEventFunction),
      authorizer
    });

    // Outputs
    new cdk.CfnOutput(this, 'ApiEndpoint', {
      value: api.apiEndpoint
    });
  }
}
```

**Deploy:**
```bash
cd infrastructure
npm install
cdk deploy --all --profile prod
```

---

## Monitoring & Observability

### CloudWatch Metrics

**Automatic Metrics:**
- Lambda: Invocations, Duration, Errors, Throttles
- API Gateway: Count, Latency, 4xx/5xx errors
- DynamoDB: ConsumedReadCapacity, ConsumedWriteCapacity, UserErrors

**Custom Metrics:**
```typescript
import { CloudWatch } from '@aws-sdk/client-cloudwatch';

const cloudwatch = new CloudWatch();

await cloudwatch.putMetricData({
  Namespace: 'HouseManagement',
  MetricData: [{
    MetricName: 'EventsCreated',
    Value: 1,
    Unit: 'Count',
    Timestamp: new Date(),
    Dimensions: [
      { Name: 'FamilyId', Value: familyId },
      { Name: 'Category', Value: category }
    ]
  }]
});
```

### X-Ray (Distributed Tracing)

```typescript
import AWSXRay from 'aws-xray-sdk-core';
import AWS from 'aws-sdk';

// Wrap AWS SDK
const dynamodb = AWSXRay.captureAWSClient(new AWS.DynamoDB.DocumentClient());

// Custom subsegments
const subsegment = AWSXRay.getSegment()?.addNewSubsegment('PlaidAPICall');
try {
  const result = await plaidClient.transactionsGet(...);
  subsegment?.close();
  return result;
} catch (error) {
  subsegment?.addError(error);
  subsegment?.close();
  throw error;
}
```

**Service Map:**
- Visual representation of service dependencies
- Latency and error rates per service
- Identify bottlenecks

### Alarms

```typescript
// CDK: Create CloudWatch Alarm
import * as cloudwatch from 'aws-cdk-lib/aws-cloudwatch';
import * as actions from 'aws-cdk-lib/aws-cloudwatch-actions';
import * as sns from 'aws-cdk-lib/aws-sns';

const topic = new sns.Topic(this, 'AlarmTopic');

const errorAlarm = new cloudwatch.Alarm(this, 'LambdaErrorAlarm', {
  metric: myFunction.metricErrors(),
  threshold: 10,
  evaluationPeriods: 2,
  treatMissingData: cloudwatch.TreatMissingData.NOT_BREACHING
});

errorAlarm.addAlarmAction(new actions.SnsAction(topic));
```

---

## Cost Optimization

### Lambda Cost Optimization

**Right-Size Memory:**
- More memory = faster execution = lower cost (sometimes)
- Use Lambda Power Tuning tool to find optimal size

**Reduce Cold Starts:**
- Keep functions warm with EventBridge pings (for critical paths)
- Use Provisioned Concurrency for predictable traffic

**Bundle Optimization:**
- Use esbuild to minimize bundle size
- Smaller bundles = faster cold starts

**Reuse Connections:**
```typescript
// Initialize outside handler (reused across invocations)
const db = new DynamoDB.DocumentClient();
const s3 = new S3();

export const handler = async (event) => {
  // Use initialized clients
};
```

### DynamoDB Cost Optimization

**On-Demand vs Provisioned:**
- On-Demand: Unpredictable traffic, pay per request
- Provisioned: Predictable traffic, cheaper at scale

**DynamoDB Streams:**
- Process changes asynchronously
- Cheaper than polling

**TTL (Time To Live):**
- Auto-delete old data (e.g., old notifications)
- No cost for deletions

### API Gateway Cost Optimization

**HTTP API vs REST API:**
- HTTP API is 70% cheaper
- Use unless you need REST API features

**Caching:**
- Cache frequent queries (reduces Lambda invocations)

### Overall Cost Estimates

**Small Scale (1,000 families, moderate usage):**
```
Lambda:             $20/month
API Gateway:        $10/month
DynamoDB:           $25/month (on-demand)
S3:                 $5/month
Cognito:            $0 (< 50k MAU)
CloudWatch:         $10/month
SES (email):        $5/month
EventBridge:        $3/month
Step Functions:     $5/month
────────────────────────────
Total: ~$83/month
```

**Medium Scale (10,000 families):**
```
Lambda:             $200/month
API Gateway:        $80/month
DynamoDB:           $300/month
S3:                 $30/month
Cognito:            $275/month (55k MAU)
CloudWatch:         $50/month
SES:                $30/month
EventBridge:        $20/month
Step Functions:     $30/month
────────────────────────────
Total: ~$1,015/month
```

**Cost Scales with Usage:** True serverless - if no one uses the app, cost is near $0.

---

## Pros & Cons of Serverless

### Advantages ✅

- **No Server Management:** Zero infrastructure to maintain
- **Auto-Scaling:** Handles 0 to millions of requests automatically
- **Pay-Per-Use:** Only pay for actual usage, not idle servers
- **High Availability:** Built into cloud services (99.9%+ SLA)
- **Faster Development:** Focus on code, not infrastructure
- **Built-in Security:** Managed services handle patching
- **Global Scale:** Deploy to multiple regions easily
- **Cost-Effective at Low Scale:** Near-zero cost when idle

### Disadvantages ❌

- **Vendor Lock-In:** Heavily tied to cloud provider (AWS, GCP, Azure)
- **Cold Starts:** Initial request latency (100-500ms for Node.js)
- **Debugging Complexity:** Distributed tracing required
- **Local Development:** Harder to replicate locally
- **Timeout Limits:** Lambda max 15 minutes
- **Stateless Only:** No in-memory state across requests
- **Learning Curve:** New paradigm, many services to learn
- **Cost at High Scale:** Can be expensive at massive scale (negotiate enterprise pricing)

### When to Choose This Strategy

- ✅ Variable/unpredictable traffic patterns
- ✅ Want to minimize operational overhead
- ✅ Small team without DevOps expertise
- ✅ Rapid prototyping and iteration
- ✅ Cost efficiency at low usage is priority
- ✅ Already using AWS/GCP/Azure
- ✅ Global user base (multi-region deployment easy)
- ❌ Predictable high traffic (may be cheaper with VMs)
- ❌ Need complete infrastructure control
- ❌ Concerned about vendor lock-in
- ❌ Team has no cloud experience

---

## Migration Strategy

### From Monolith to Serverless

**Phase 1: Frontend Migration**
1. Deploy React app to Vercel/Amplify
2. Keep backend as monolith initially
3. Use CloudFront for caching

**Phase 2: API Gateway**
1. Add API Gateway in front of monolith
2. Proxy all requests to monolith initially
3. Gradually route endpoints to Lambda

**Phase 3: Extract Functions**
1. Start with simple, stateless endpoints
2. Create Lambda for each route
3. Migrate database to DynamoDB/Aurora Serverless
4. Update API Gateway routes

**Phase 4: Event-Driven**
1. Introduce EventBridge
2. Decouple services with events
3. Add async processing (queues)

---

## Estimated Timeline

**Month 1-2: Foundation**
- AWS account setup, IAM, CDK
- Cognito user authentication
- API Gateway + first Lambda functions
- DynamoDB schema design
- Frontend deployment (Vercel)

**Month 3-5: Core Features**
- Calendar Lambda functions + DynamoDB
- Plaid integration (Financial Lambdas)
- Vehicle management
- EventBridge for notifications

**Month 6-8: Advanced Features**
- Document upload (S3 + Textract)
- Bill tracking and reminders
- Task management
- Email notifications (SES)

**Month 9-10: Polish & Launch**
- Step Functions for complex workflows
- Monitoring and alarms
- Security audit
- Load testing
- Beta testing
- Production launch

---

## Conclusion

The serverless architecture offers the best operational simplicity and cost efficiency for variable workloads. It's ideal for teams that want to focus on building features rather than managing infrastructure.

**Best for:**
- Startups wanting fast iteration
- Teams without dedicated DevOps
- Applications with variable traffic
- Prototypes and MVPs
- Budget-conscious projects

**Consider alternatives if:**
- You need complete infrastructure control
- Vendor lock-in is unacceptable
- Team lacks cloud expertise
- Predictable high-scale workloads (VMs may be cheaper)

**Hybrid Approach:** Start serverless for MVP, migrate specific components to containers/VMs if economics change at scale.
