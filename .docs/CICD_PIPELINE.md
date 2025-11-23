# CI/CD Pipeline Design
## Family House Management Application

**Last Updated:** November 2025
**Platform:** GitHub Actions (can be adapted for GitLab CI, Jenkins, etc.)

---

## Pipeline Overview

```
┌──────────────┐
│ Code Commit  │
└──────┬───────┘
       │
       ▼
┌──────────────────────────────────────┐
│ Continuous Integration (CI)          │
├──────────────────────────────────────┤
│ 1. Lint & Format Check               │
│ 2. TypeScript Type Check             │
│ 3. Unit Tests                        │
│ 4. Integration Tests                 │
│ 5. Build                             │
│ 6. E2E Tests (on feature branches)   │
└──────┬───────────────────────────────┘
       │
       ▼ (if main/develop branch)
┌──────────────────────────────────────┐
│ Continuous Deployment (CD)           │
├──────────────────────────────────────┤
│ 1. Build Docker Images               │
│ 2. Deploy to Staging                 │
│ 3. Smoke Tests                       │
│ 4. Deploy to Production (main only)  │
│ 5. Post-Deploy Verification          │
└──────────────────────────────────────┘
```

---

## GitHub Actions Workflows

### 1. Pull Request CI Workflow

**File:** `.github/workflows/pr-checks.yml`

```yaml
name: PR Checks

on:
  pull_request:
    branches: [main, develop]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint-and-format:
    name: Lint & Format Check
    runs-on: ubuntu-latest
    timeout-minutes: 5

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint backend
        run: cd server && npm run lint

      - name: Lint frontend
        run: cd client && npm run lint

      - name: Check Prettier formatting
        run: npm run format:check

  type-check:
    name: TypeScript Type Check
    runs-on: ubuntu-latest
    timeout-minutes: 5

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci

      - name: Type check backend
        run: cd server && npm run type-check

      - name: Type check frontend
        run: cd client && npm run type-check

  backend-unit-tests:
    name: Backend Unit Tests
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: cd server && npm ci

      - name: Run unit tests
        run: cd server && npm run test:unit

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./server/coverage/lcov.info
          flags: backend-unit
          fail_ci_if_error: false

  backend-integration-tests:
    name: Backend Integration Tests
    runs-on: ubuntu-latest
    timeout-minutes: 15

    services:
      postgres:
        image: postgres:15-alpine
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: house_management_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: cd server && npm ci

      - name: Generate Prisma Client
        run: cd server && npx prisma generate

      - name: Run migrations
        run: cd server && npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://postgres:testpass@localhost:5432/house_management_test

      - name: Run integration tests
        run: cd server && npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:testpass@localhost:5432/house_management_test
          REDIS_URL: redis://localhost:6379
          JWT_SECRET: test-jwt-secret
          ENCRYPTION_KEY: test-encryption-key-32-bytes!!

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./server/coverage/lcov.info
          flags: backend-integration

  frontend-tests:
    name: Frontend Tests
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: cd client && npm ci

      - name: Run tests
        run: cd client && npm run test

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./client/coverage/lcov.info
          flags: frontend

  build:
    name: Build Check
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build backend
        run: cd server && npm run build

      - name: Build frontend
        run: cd client && npm run build
        env:
          VITE_API_URL: https://api.staging.housemgmt.app

  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    timeout-minutes: 20

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Run E2E tests
        run: npm run test:e2e

      - name: Upload Playwright report
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30

  security-scan:
    name: Security Scan
    runs-on: ubuntu-latest
    timeout-minutes: 5

    steps:
      - uses: actions/checkout@v4

      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/node@master
        continue-on-error: true
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

      - name: Run npm audit
        run: |
          cd server && npm audit --audit-level=moderate
          cd ../client && npm audit --audit-level=moderate
```

---

### 2. Deploy to Staging Workflow

**File:** `.github/workflows/deploy-staging.yml`

```yaml
name: Deploy to Staging

on:
  push:
    branches: [develop]

jobs:
  build-and-deploy:
    name: Build & Deploy to Staging
    runs-on: ubuntu-latest
    timeout-minutes: 20

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build backend
        run: cd server && npm run build

      - name: Build frontend
        run: cd client && npm run build
        env:
          VITE_API_URL: https://api.staging.housemgmt.app

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push backend image
        uses: docker/build-push-action@v5
        with:
          context: ./server
          push: true
          tags: ghcr.io/${{ github.repository }}/backend:staging-${{ github.sha }}
          cache-from: type=registry,ref=ghcr.io/${{ github.repository }}/backend:staging-latest
          cache-to: type=inline

      - name: Build and push frontend image
        uses: docker/build-push-action@v5
        with:
          context: ./client
          push: true
          tags: ghcr.io/${{ github.repository }}/frontend:staging-${{ github.sha }}
          cache-from: type=registry,ref=ghcr.io/${{ github.repository }}/frontend:staging-latest
          cache-to: type=inline

      - name: Deploy to staging server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.STAGING_SERVER_HOST }}
          username: ${{ secrets.STAGING_SERVER_USER }}
          key: ${{ secrets.STAGING_SERVER_SSH_KEY }}
          script: |
            cd /opt/house-mgmt
            docker-compose pull
            docker-compose up -d
            docker system prune -af

      - name: Run database migrations
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.STAGING_SERVER_HOST }}
          username: ${{ secrets.STAGING_SERVER_USER }}
          key: ${{ secrets.STAGING_SERVER_SSH_KEY }}
          script: |
            cd /opt/house-mgmt
            docker-compose exec -T backend npx prisma migrate deploy

      - name: Run smoke tests
        run: |
          sleep 10  # Wait for services to be ready
          npm run test:smoke -- --url=https://staging.housemgmt.app

      - name: Notify deployment success
        uses: 8398a7/action-slack@v3
        if: success()
        with:
          status: custom
          custom_payload: |
            {
              text: "✅ Staging deployment successful",
              attachments: [{
                color: 'good',
                text: `Commit: ${{ github.sha }}\nBranch: develop\nURL: https://staging.housemgmt.app`
              }]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

      - name: Notify deployment failure
        uses: 8398a7/action-slack@v3
        if: failure()
        with:
          status: custom
          custom_payload: |
            {
              text: "❌ Staging deployment failed",
              attachments: [{
                color: 'danger',
                text: `Commit: ${{ github.sha }}\nBranch: develop`
              }]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

### 3. Deploy to Production Workflow

**File:** `.github/workflows/deploy-production.yml`

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]
    tags:
      - 'v*'

jobs:
  build-and-deploy:
    name: Build & Deploy to Production
    runs-on: ubuntu-latest
    timeout-minutes: 30
    environment: production

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run all tests
        run: npm run test:all

      - name: Build backend
        run: cd server && npm run build

      - name: Build frontend
        run: cd client && npm run build
        env:
          VITE_API_URL: https://api.housemgmt.app

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract version from tag
        id: version
        run: |
          if [[ $GITHUB_REF == refs/tags/* ]]; then
            echo "VERSION=${GITHUB_REF#refs/tags/}" >> $GITHUB_OUTPUT
          else
            echo "VERSION=${{ github.sha }}" >> $GITHUB_OUTPUT
          fi

      - name: Build and push backend image
        uses: docker/build-push-action@v5
        with:
          context: ./server
          push: true
          tags: |
            ghcr.io/${{ github.repository }}/backend:${{ steps.version.outputs.VERSION }}
            ghcr.io/${{ github.repository }}/backend:latest
          cache-from: type=registry,ref=ghcr.io/${{ github.repository }}/backend:latest
          cache-to: type=inline

      - name: Build and push frontend image
        uses: docker/build-push-action@v5
        with:
          context: ./client
          push: true
          tags: |
            ghcr.io/${{ github.repository }}/frontend:${{ steps.version.outputs.VERSION }}
            ghcr.io/${{ github.repository }}/frontend:latest
          cache-from: type=registry,ref=ghcr.io/${{ github.repository }}/frontend:latest
          cache-to: type=inline

      - name: Create database backup
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.PROD_SERVER_HOST }}
          username: ${{ secrets.PROD_SERVER_USER }}
          key: ${{ secrets.PROD_SERVER_SSH_KEY }}
          script: |
            timestamp=$(date +%Y%m%d_%H%M%S)
            docker exec house-mgmt-postgres pg_dump -U postgres house_management \
              | gzip > /backups/pre-deploy-$timestamp.sql.gz

      - name: Deploy to production server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.PROD_SERVER_HOST }}
          username: ${{ secrets.PROD_SERVER_USER }}
          key: ${{ secrets.PROD_SERVER_SSH_KEY }}
          script: |
            cd /opt/house-mgmt
            docker-compose pull
            docker-compose up -d --no-deps backend frontend
            docker system prune -af

      - name: Run database migrations
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.PROD_SERVER_HOST }}
          username: ${{ secrets.PROD_SERVER_USER }}
          key: ${{ secrets.PROD_SERVER_SSH_KEY }}
          script: |
            cd /opt/house-mgmt
            docker-compose exec -T backend npx prisma migrate deploy

      - name: Health check
        run: |
          sleep 15
          response=$(curl -s -o /dev/null -w "%{http_code}" https://api.housemgmt.app/health)
          if [ $response -ne 200 ]; then
            echo "Health check failed with status $response"
            exit 1
          fi
          echo "Health check passed"

      - name: Rollback on failure
        if: failure()
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.PROD_SERVER_HOST }}
          username: ${{ secrets.PROD_SERVER_USER }}
          key: ${{ secrets.PROD_SERVER_SSH_KEY }}
          script: |
            cd /opt/house-mgmt
            docker-compose down
            docker-compose up -d

      - name: Notify deployment success
        uses: 8398a7/action-slack@v3
        if: success()
        with:
          status: custom
          custom_payload: |
            {
              text: "🚀 Production deployment successful",
              attachments: [{
                color: 'good',
                text: `Version: ${{ steps.version.outputs.VERSION }}\nURL: https://housemgmt.app`
              }]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

      - name: Notify deployment failure
        uses: 8398a7/action-slack@v3
        if: failure()
        with:
          status: custom
          custom_payload: |
            {
              text: "🚨 Production deployment FAILED - Rollback initiated",
              attachments: [{
                color: 'danger',
                text: `Version: ${{ steps.version.outputs.VERSION }}\n@channel Please investigate immediately`
              }]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

## Docker Configuration

### Backend Dockerfile

**File:** `server/Dockerfile`

```dockerfile
# Build stage
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
COPY prisma ./prisma/

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Generate Prisma client
RUN npx prisma generate

# Build TypeScript
RUN npm run build

# Production stage
FROM node:20-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./
COPY prisma ./prisma/

# Install production dependencies only
RUN npm ci --omit=dev

# Copy built application
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules/.prisma ./node_modules/.prisma

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

USER nodejs

EXPOSE 3000

CMD ["node", "dist/server.js"]
```

### Frontend Dockerfile

**File:** `client/Dockerfile`

```dockerfile
# Build stage
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine

# Copy built assets
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose (Production)

**File:** `docker-compose.yml`

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/ssl:/etc/nginx/ssl
      - ./client/dist:/usr/share/nginx/html
    depends_on:
      - backend
    restart: unless-stopped

  backend:
    image: ghcr.io/yourorg/house-mgmt/backend:latest
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - JWT_SECRET=${JWT_SECRET}
      - PLAID_CLIENT_ID=${PLAID_CLIENT_ID}
      - PLAID_SECRET=${PLAID_SECRET}
    depends_on:
      - postgres
      - redis
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  worker:
    image: ghcr.io/yourorg/house-mgmt/backend:latest
    command: npm run worker
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    depends_on:
      - postgres
      - redis
    restart: unless-stopped

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=house_management
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
  redis_data:
```

---

## Deployment Strategies

### Blue-Green Deployment

```yaml
# Deploy new version (green)
docker-compose -f docker-compose.green.yml up -d

# Run smoke tests
npm run test:smoke -- --url=https://green.housemgmt.app

# If successful, switch traffic
# Update load balancer to point to green

# If failed, rollback
docker-compose -f docker-compose.blue.yml up -d
```

### Canary Deployment

```yaml
# Deploy canary (5% of traffic)
# Update load balancer to send 5% traffic to new version

# Monitor metrics for 15 minutes

# If healthy, increase to 25%
# Monitor for 15 minutes

# If healthy, increase to 50%
# Monitor for 15 minutes

# If healthy, deploy to 100%

# If any issues, rollback to 0%
```

---

## Monitoring & Alerting

### Health Check Endpoint

```typescript
// server/src/api/routes/healthRoutes.ts
import { Router } from 'express';
import { prisma } from '@/lib/prisma';
import { redis } from '@/lib/redis';

const router = Router();

router.get('/health', async (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    version: process.env.npm_package_version,
    uptime: process.uptime(),
    dependencies: {
      database: 'unknown',
      redis: 'unknown',
    }
  };

  try {
    // Check database
    await prisma.$queryRaw`SELECT 1`;
    health.dependencies.database = 'healthy';
  } catch (error) {
    health.dependencies.database = 'unhealthy';
    health.status = 'degraded';
  }

  try {
    // Check Redis
    await redis.ping();
    health.dependencies.redis = 'healthy';
  } catch (error) {
    health.dependencies.redis = 'unhealthy';
    health.status = 'degraded';
  }

  const statusCode = health.status === 'healthy' ? 200 : 503;
  res.status(statusCode).json(health);
});

export default router;
```

### Uptime Monitoring

**Services:**
- UptimeRobot (free tier)
- Pingdom
- StatusCake

**Configure:**
- Check every 5 minutes
- Alert via email/SMS/Slack
- Monitor:
  - https://api.housemgmt.app/health
  - https://housemgmt.app

---

## Database Migrations in CI/CD

### Migration Strategy

```yaml
# In deployment workflow
- name: Backup database
  run: pg_dump $DATABASE_URL > backup.sql

- name: Run migrations
  run: npx prisma migrate deploy
  env:
    DATABASE_URL: ${{ secrets.DATABASE_URL }}

- name: Verify migration
  run: |
    # Run a test query to verify schema
    psql $DATABASE_URL -c "SELECT * FROM _prisma_migrations;"

- name: Rollback on failure
  if: failure()
  run: psql $DATABASE_URL < backup.sql
```

### Migration Best Practices

1. **Never drop columns in production** - deprecate and remove later
2. **Always make migrations backward compatible**
3. **Test migrations on copy of production data**
4. **Have rollback plan ready**
5. **Run migrations before deploying new code**

---

## Secrets Management

### GitHub Secrets (Required)

```
STAGING_SERVER_HOST
STAGING_SERVER_USER
STAGING_SERVER_SSH_KEY

PROD_SERVER_HOST
PROD_SERVER_USER
PROD_SERVER_SSH_KEY

DATABASE_URL (production)
JWT_SECRET
JWT_REFRESH_SECRET
ENCRYPTION_KEY

PLAID_CLIENT_ID
PLAID_SECRET

GOOGLE_CLIENT_ID
GOOGLE_CLIENT_SECRET

SENDGRID_API_KEY

AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY

SLACK_WEBHOOK_URL
SNYK_TOKEN
```

---

## Cost Optimization

### Caching

```yaml
# Cache dependencies
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

# Cache Docker layers
- uses: docker/build-push-action@v5
  with:
    cache-from: type=registry,ref=myimage:latest
    cache-to: type=inline
```

### Concurrency Controls

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

---

## Disaster Recovery Plan

### Automated Backups

```bash
#!/bin/bash
# scripts/backup.sh

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups"

# Backup database
docker exec house-mgmt-postgres pg_dump -U postgres house_management \
  | gzip > $BACKUP_DIR/db_backup_$TIMESTAMP.sql.gz

# Backup to S3
aws s3 cp $BACKUP_DIR/db_backup_$TIMESTAMP.sql.gz \
  s3://house-mgmt-backups/database/

# Keep only last 30 days locally
find $BACKUP_DIR -name "db_backup_*.sql.gz" -mtime +30 -delete
```

**Cron:** Run daily at 2 AM

---

**This CI/CD pipeline ensures reliable, automated deployments with safety checks at every stage!**
