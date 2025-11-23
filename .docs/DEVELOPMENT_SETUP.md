# Development Environment Setup
## Family House Management Application

**Last Updated:** November 2025
**Target OS:** macOS, Linux, Windows (WSL2)

---

## Prerequisites

### Required Software

| Software | Version | Purpose |
|----------|---------|---------|
| Node.js | 20 LTS | Backend runtime |
| npm | 10+ | Package manager |
| PostgreSQL | 15+ | Database |
| Redis | 7+ | Cache & job queue |
| Git | 2.40+ | Version control |
| Docker | 24+ (optional) | Containerization |
| VS Code | Latest (recommended) | IDE |

---

## Installation Steps

### 1. Install Node.js

**macOS (Homebrew):**
```bash
brew install node@20
```

**Linux (nvm - recommended):**
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install 20
nvm use 20
```

**Windows (WSL2):**
```bash
# Use nvm for Linux instructions above
```

**Verify Installation:**
```bash
node --version  # Should show v20.x.x
npm --version   # Should show 10.x.x
```

### 2. Install PostgreSQL

**Option A: Docker (Recommended for Development)**
```bash
docker run --name house-mgmt-postgres \
  -e POSTGRES_PASSWORD=devpassword \
  -e POSTGRES_DB=house_management_dev \
  -p 5432:5432 \
  -d postgres:15-alpine
```

**Option B: Native Installation**

**macOS:**
```bash
brew install postgresql@15
brew services start postgresql@15
createdb house_management_dev
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install postgresql-15
sudo systemctl start postgresql
sudo -u postgres createdb house_management_dev
```

**Verify Installation:**
```bash
psql -U postgres -d house_management_dev -c "SELECT version();"
```

### 3. Install Redis

**Option A: Docker (Recommended)**
```bash
docker run --name house-mgmt-redis \
  -p 6379:6379 \
  -d redis:7-alpine
```

**Option B: Native Installation**

**macOS:**
```bash
brew install redis
brew services start redis
```

**Linux:**
```bash
sudo apt install redis-server
sudo systemctl start redis-server
```

**Verify Installation:**
```bash
redis-cli ping  # Should return "PONG"
```

### 4. Install Git
```bash
# macOS
brew install git

# Linux
sudo apt install git

# Configure
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### 5. Install VS Code (Optional but Recommended)
Download from: https://code.visualstudio.com/

**Recommended Extensions:**
- ESLint
- Prettier - Code Formatter
- Prisma
- GitLens
- Thunder Client (API testing)
- Error Lens
- Auto Rename Tag
- Tailwind CSS IntelliSense

---

## Project Setup

### 1. Clone Repository
```bash
git clone <repository-url>
cd house-management-app
```

### 2. Install Dependencies

**Root Dependencies (if using workspaces):**
```bash
npm install
```

**Or install separately:**
```bash
# Frontend
cd client
npm install

# Backend
cd ../server
npm install
```

### 3. Environment Variables

#### Server Environment (.env)

Create `server/.env`:
```env
# Server
NODE_ENV=development
PORT=3000

# Database
DATABASE_URL="postgresql://postgres:devpassword@localhost:5432/house_management_dev?schema=public"

# Redis
REDIS_URL="redis://localhost:6379"

# JWT
JWT_SECRET="your-super-secret-jwt-key-change-in-production"
JWT_REFRESH_SECRET="your-super-secret-refresh-key-change-in-production"
JWT_ACCESS_EXPIRY="15m"
JWT_REFRESH_EXPIRY="7d"

# Encryption
ENCRYPTION_KEY="32-byte-encryption-key-for-aes256" # 32 bytes exactly

# Email (SendGrid)
SENDGRID_API_KEY="SG.your-sendgrid-api-key"
FROM_EMAIL="noreply@housemgmt.app"
FRONTEND_URL="http://localhost:5173"

# Plaid (Sandbox)
PLAID_CLIENT_ID="your-plaid-client-id"
PLAID_SECRET="your-plaid-sandbox-secret"
PLAID_ENV="sandbox"

# Google Calendar
GOOGLE_CLIENT_ID="your-google-oauth-client-id"
GOOGLE_CLIENT_SECRET="your-google-oauth-client-secret"
GOOGLE_REDIRECT_URI="http://localhost:3000/api/v1/calendar/integrations/google/callback"

# AWS S3 (for file uploads - optional for initial dev)
AWS_REGION="us-east-1"
AWS_ACCESS_KEY_ID="your-aws-access-key"
AWS_SECRET_ACCESS_KEY="your-aws-secret-key"
AWS_S3_BUCKET="house-mgmt-dev"

# Monitoring (optional)
SENTRY_DSN=""
```

#### Client Environment (.env)

Create `client/.env`:
```env
# API
VITE_API_URL=http://localhost:3000/api/v1

# Environment
VITE_ENV=development

# Feature Flags (optional)
VITE_ENABLE_GOOGLE_CALENDAR=true
VITE_ENABLE_PLAID=true
```

### 4. Database Setup

**Generate Prisma Client:**
```bash
cd server
npx prisma generate
```

**Run Migrations:**
```bash
npx prisma migrate dev --name init
```

**Seed Database (Optional):**
```bash
npx prisma db seed
```

**Prisma Studio (Database GUI):**
```bash
npx prisma studio
# Opens at http://localhost:5555
```

### 5. Start Development Servers

**Option A: Using Separate Terminals**

Terminal 1 - Backend:
```bash
cd server
npm run dev
# Server starts at http://localhost:3000
```

Terminal 2 - Frontend:
```bash
cd client
npm run dev
# Vite dev server at http://localhost:5173
```

Terminal 3 - Background Workers:
```bash
cd server
npm run worker
# BullMQ workers for background jobs
```

**Option B: Using Docker Compose (Full Stack)**

Create `docker-compose.dev.yml`:
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: devpassword
      POSTGRES_DB: house_management_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

Start services:
```bash
docker-compose -f docker-compose.dev.yml up -d
```

---

## Development Workflow

### Daily Workflow

1. **Pull Latest Changes:**
```bash
git pull origin develop
```

2. **Install New Dependencies:**
```bash
cd client && npm install
cd ../server && npm install
```

3. **Run Database Migrations:**
```bash
cd server
npx prisma migrate dev
```

4. **Start Servers:**
```bash
# Terminal 1
cd server && npm run dev

# Terminal 2
cd client && npm run dev

# Terminal 3
cd server && npm run worker
```

5. **Access Application:**
- Frontend: http://localhost:5173
- Backend API: http://localhost:3000
- API Docs: http://localhost:3000/api-docs
- Prisma Studio: http://localhost:5555 (if running)

### Git Workflow

```bash
# Create feature branch
git checkout -b feature/calendar-sync

# Make changes, commit frequently
git add .
git commit -m "Add calendar sync functionality"

# Push to remote
git push origin feature/calendar-sync

# Create PR on GitHub/GitLab
```

### Database Workflows

**Create New Migration:**
```bash
cd server
npx prisma migrate dev --name add_document_table
```

**Reset Database (WARNING: Deletes all data):**
```bash
npx prisma migrate reset
```

**Update Prisma Client After Schema Changes:**
```bash
npx prisma generate
```

**View Database:**
```bash
npx prisma studio
```

---

## Testing

### Run Tests

**Backend Tests:**
```bash
cd server

# Unit tests
npm run test

# Integration tests
npm run test:integration

# E2E tests
npm run test:e2e

# Coverage report
npm run test:coverage
```

**Frontend Tests:**
```bash
cd client

# Unit tests
npm run test

# Component tests
npm run test:components

# Coverage
npm run test:coverage
```

### Test Database

Use separate test database:
```env
# server/.env.test
DATABASE_URL="postgresql://postgres:devpassword@localhost:5432/house_management_test?schema=public"
```

Setup test database:
```bash
DATABASE_URL="postgresql://..." npx prisma migrate deploy
```

---

## Debugging

### VS Code Launch Configuration

Create `.vscode/launch.json`:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Backend",
      "skipFiles": ["<node_internals>/**"],
      "program": "${workspaceFolder}/server/src/server.ts",
      "preLaunchTask": "tsc: build - server/tsconfig.json",
      "outFiles": ["${workspaceFolder}/server/dist/**/*.js"],
      "env": {
        "NODE_ENV": "development"
      }
    },
    {
      "type": "chrome",
      "request": "launch",
      "name": "Debug Frontend (Chrome)",
      "url": "http://localhost:5173",
      "webRoot": "${workspaceFolder}/client",
      "sourceMapPathOverrides": {
        "webpack:///./src/*": "${webRoot}/src/*"
      }
    }
  ]
}
```

### Debug Backend API
```typescript
// Add breakpoints in VS Code
// Start with F5 or Run > Start Debugging
```

### Debug Frontend
```javascript
// Browser DevTools
// React DevTools extension
// Redux DevTools extension
```

---

## Troubleshooting

### Common Issues

**Issue: Port 3000 already in use**
```bash
# Find and kill process
lsof -ti:3000 | xargs kill -9

# Or use different port
PORT=3001 npm run dev
```

**Issue: Database connection refused**
```bash
# Check PostgreSQL is running
pg_isready -h localhost -p 5432

# Restart PostgreSQL
# macOS
brew services restart postgresql@15

# Linux
sudo systemctl restart postgresql

# Docker
docker restart house-mgmt-postgres
```

**Issue: Redis connection refused**
```bash
# Check Redis is running
redis-cli ping

# Restart Redis
# macOS
brew services restart redis

# Linux
sudo systemctl restart redis

# Docker
docker restart house-mgmt-redis
```

**Issue: Prisma Client not generated**
```bash
cd server
npx prisma generate
```

**Issue: Migration failed**
```bash
# Reset and reapply migrations
npx prisma migrate reset
npx prisma migrate dev
```

**Issue: npm install fails**
```bash
# Clear cache
npm cache clean --force

# Remove node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

---

## Development Tools

### Recommended CLI Tools

```bash
# Install globally
npm install -g typescript ts-node nodemon prisma
```

### API Testing

**Thunder Client (VS Code):**
- Install extension in VS Code
- Import collection from `docs/thunder-client/`

**Postman:**
- Import collection from `docs/postman/house-management-api.json`

**cURL Examples:**

```bash
# Register
curl -X POST http://localhost:3000/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "Test123!",
    "name": "Test User",
    "familyName": "Test Family"
  }'

# Login
curl -X POST http://localhost:3000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "Test123!"
  }'

# Get calendar events (with auth)
curl http://localhost:3000/api/v1/calendar/events \
  -H "Authorization: Bearer <your-access-token>"
```

### Database Inspection

**psql (PostgreSQL CLI):**
```bash
psql -U postgres -d house_management_dev

# List tables
\dt

# Describe table
\d calendar_events

# Run query
SELECT * FROM users LIMIT 5;

# Exit
\q
```

**Redis CLI:**
```bash
redis-cli

# List all keys
KEYS *

# Get value
GET key_name

# Monitor commands
MONITOR

# Exit
quit
```

---

## External Service Setup

### SendGrid (Email)

1. Create account: https://sendgrid.com/
2. Verify sender email
3. Create API key (Settings > API Keys)
4. Add to `.env`: `SENDGRID_API_KEY=SG.xxx`
5. Test email sending:
```bash
curl -X POST http://localhost:3000/api/v1/test/email \
  -H "Content-Type: application/json" \
  -d '{"to": "your-email@example.com"}'
```

### Plaid (Financial Data)

1. Create account: https://plaid.com/
2. Get credentials from dashboard
3. Add to `.env`:
```env
PLAID_CLIENT_ID=your-client-id
PLAID_SECRET=your-sandbox-secret
PLAID_ENV=sandbox
```
4. Test integration:
```bash
# Create link token
curl -X POST http://localhost:3000/api/v1/financial/accounts/link \
  -H "Authorization: Bearer <token>"
```

### Google Calendar API

1. Go to: https://console.cloud.google.com/
2. Create new project
3. Enable Google Calendar API
4. Create OAuth 2.0 credentials
5. Add authorized redirect URI: `http://localhost:3000/api/v1/calendar/integrations/google/callback`
6. Add to `.env`:
```env
GOOGLE_CLIENT_ID=your-client-id
GOOGLE_CLIENT_SECRET=your-client-secret
```

### AWS S3 (File Storage - Optional)

1. Create AWS account
2. Create S3 bucket: `house-mgmt-dev`
3. Create IAM user with S3 permissions
4. Add credentials to `.env`:
```env
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_S3_BUCKET=house-mgmt-dev
```

Or use MinIO for local development:
```bash
docker run -p 9000:9000 -p 9001:9001 \
  -e "MINIO_ROOT_USER=minioadmin" \
  -e "MINIO_ROOT_PASSWORD=minioadmin" \
  quay.io/minio/minio server /data --console-address ":9001"
```

---

## Performance Optimization Tips

### Backend
- Enable Prisma query logging in development
- Use database indexes appropriately
- Profile slow queries with `EXPLAIN ANALYZE`
- Monitor BullMQ queue depths

### Frontend
- Use React DevTools Profiler
- Enable Vite's HMR for fast refresh
- Lazy load routes and components
- Use React Query DevTools

---

## IDE Configuration

### VS Code Settings

Create `.vscode/settings.json`:
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,
  "eslint.workingDirectories": [
    "./client",
    "./server"
  ],
  "[prisma]": {
    "editor.defaultFormatter": "Prisma.prisma"
  }
}
```

### Prettier Configuration

Create `.prettierrc`:
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false
}
```

### ESLint Configuration

Already configured in `client/.eslintrc.json` and `server/.eslintrc.json`

---

## Hot Tips

1. **Use npm scripts in package.json** for common tasks
2. **Create shell aliases** for frequently used commands
3. **Use Prisma Studio** to inspect database during development
4. **Enable TypeScript strict mode** for better type safety
5. **Use environment-specific `.env` files** (.env.development, .env.test)
6. **Keep dependencies updated** with `npm outdated`
7. **Use debugger statements** instead of console.log
8. **Commit `.env.example`** files (without secrets)
9. **Use Docker for consistency** across team members
10. **Document any dev setup issues** you encounter

---

## Next Steps

After setup is complete:

1. **Review coding standards:** Read `CODING_STANDARDS.md`
2. **Understand database schema:** Review `DATABASE_SCHEMA.md`
3. **Study API spec:** Read `API_SPECIFICATION.md`
4. **Pick first task:** Check `MVP_FEATURE_BREAKDOWN.md`
5. **Create feature branch:** Start development!

---

**Setup complete! Ready to start development. 🚀**
