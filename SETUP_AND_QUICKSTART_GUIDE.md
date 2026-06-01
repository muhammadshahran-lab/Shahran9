# Riphah UMS - Setup & Quick Start Guide

Complete step-by-step guide to setup and run the Riphah UMS locally.

---

## 📋 Prerequisites

### System Requirements

```
Windows 10/11 or macOS 10.15+ or Ubuntu 20.04+
RAM: 8GB minimum (16GB recommended)
Disk Space: 20GB available
```

### Software Requirements

Install in this order:

#### 1. Git
```bash
# Windows (using winget)
winget install Git.Git

# macOS
brew install git

# Linux (Ubuntu)
sudo apt update && sudo apt install git
```

#### 2. Node.js 18+
```bash
# Windows (using winget)
winget install OpenJS.NodeJS

# macOS
brew install node

# Linux
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

#### 3. Docker & Docker Compose
```bash
# Windows
winget install Docker.DockerDesktop

# macOS
brew install --cask docker

# Linux (Ubuntu)
sudo apt update
sudo apt install docker.io docker-compose
```

#### 4. PostgreSQL CLI (optional but useful)
```bash
# Windows
winget install PostgreSQL.PostgreSQL

# macOS
brew install postgresql

# Linux
sudo apt install postgresql-client
```

#### 5. NestJS CLI
```bash
npm install -g @nestjs/cli
```

#### 6. Prisma CLI (will be installed via npm)
```bash
npm install @prisma/cli -D
```

---

## 🚀 Installation Steps

### Step 1: Clone Repository

```bash
# Clone the main repository
git clone https://github.com/riphah/ums.git
cd ums

# Or if setting up locally
mkdir -p ~/projects/riphah-ums
cd ~/projects/riphah-ums
git init
```

### Step 2: Install Node Dependencies

```bash
# Install root dependencies
npm install

# If using workspaces, install all:
npm install --workspaces

# Or individually:
cd backend/nest
npm install

cd ../../frontend
npm install

cd ../
```

### Step 3: Environment Configuration

Copy environment templates:

```bash
# Backend environment
cp backend/nest/.env.example backend/nest/.env

# Frontend environment
cp frontend/.env.example frontend/.env

# Root environment
cp .env.example .env
```

Edit `backend/nest/.env`:

```env
# DATABASE
DATABASE_URL="postgresql://postgres:password@localhost:5432/riphah_ums?schema=public"

# JWT
JWT_SECRET="your-super-secret-key-min-32-chars-long-here"
JWT_EXPIRATION="3600"

# SERVER
NODE_ENV="development"
PORT=4000

# REDIS
REDIS_URL="redis://localhost:6379"

# MONGODB (for logs/analytics)
MONGODB_URL="mongodb://localhost:27017/riphah_ums"

# AI Services
AI_SERVICE_URL="http://localhost:8000"
AI_SERVICE_KEY="ai-secret-key"

# Email (optional)
SMTP_HOST="smtp.gmail.com"
SMTP_PORT="587"
SMTP_USER="your-email@gmail.com"
SMTP_PASS="your-app-password"

# JWT Refresh Token
JWT_REFRESH_SECRET="your-refresh-secret-min-32-chars"
JWT_REFRESH_EXPIRATION="604800"
```

### Step 4: Start Docker Services

```bash
# Start PostgreSQL, Redis, MongoDB
docker-compose up -d

# Verify services are running
docker-compose ps

# Check logs if needed
docker-compose logs -f

# Stop services (when done)
docker-compose down

# Stop and remove volumes (reset databases)
docker-compose down -v
```

### Step 5: Setup Database

```bash
cd backend/nest

# Generate Prisma Client
npm run prisma:generate

# Run migrations (creates tables)
npm run prisma:migrate -- --name initial

# Seed initial data (roles, permissions, admin user)
npm run prisma:seed

# Open Prisma Studio (visual editor)
npm run prisma:studio
```

### Step 6: Start Backend Server

```bash
cd backend/nest

# Development mode (auto-reload)
npm run start:dev

# Or production mode
npm run build
npm run start

# Server should start on http://localhost:4000
```

### Step 7: Verify Backend

```bash
# In another terminal, test the API
curl http://localhost:4000/api/health

# Should return: {"status":"ok"}

# Check Swagger docs
# Open in browser: http://localhost:4000/api/docs
```

### Step 8: Start Frontend (Optional)

```bash
cd frontend

# Development mode
npm run dev

# Frontend runs on http://localhost:3000
```

---

## ✅ Verification Checklist

After setup, verify everything works:

### 1. Docker Services Running
```bash
docker-compose ps
# All containers should be "Up"
```

### 2. PostgreSQL Connection
```bash
# Connection string: postgresql://postgres:password@localhost:5432/riphah_ums
psql postgresql://postgres:password@localhost:5432/riphah_ums

# Then list tables
\dt
# Should show ~50 tables from schema
```

### 3. Backend Server Running
```bash
# Should return healthy status
curl http://localhost:4000/api/health

# Response: {"status":"ok"}
```

### 4. API Documentation
Open browser: http://localhost:4000/api/docs
- Should show Swagger UI with all endpoints
- Try "Try it out" on GET /api/health

### 5. Database Seeded
```bash
# Check if admin user exists (should be created by seed)
# curl http://localhost:4000/api/users

# Or in Prisma Studio:
npm run prisma:studio
# Navigate to "users" table, should see admin@university.edu
```

### 6. Roles & Permissions
```bash
# Check roles endpoint
curl http://localhost:4000/api/roles \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"

# Or via Swagger UI with authentication
```

---

## 🔐 Authentication Setup

### Default Admin User (created by seed)

```
Email: superadmin@university.edu
Password: SuperAdmin@123
```

### Get JWT Token

```bash
# Login
curl -X POST http://localhost:4000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "superadmin@university.edu",
    "password": "SuperAdmin@123"
  }'

# Response contains:
# {
#   "access_token": "eyJhbGc...",
#   "refresh_token": "eyJhbGc...",
#   "user": { ... }
# }

# Copy the access_token for authorized requests
```

### Use Token in API Calls

```bash
# Use in Authorization header
curl http://localhost:4000/api/users \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"

# Or in Swagger UI:
# Click "Authorize" button -> Paste token -> Close -> Try out endpoints
```

---

## 🛠️ Development Workflow

### Common Commands

```bash
# Backend
cd backend/nest

# Start dev server
npm run start:dev

# Run tests
npm run test
npm run test:e2e

# Run linter
npm run lint

# Format code
npm run format

# Build for production
npm run build

# Database operations
npm run prisma:generate
npm run prisma:migrate -- --name "description"
npm run prisma:seed
npm run prisma:studio

# View logs
npm run prisma:doctor
```

### Frontend
```bash
cd frontend

# Start dev server
npm run dev

# Build
npm run build

# Test
npm run test

# Lint
npm run lint
```

### Database Migrations

```bash
# After changing schema in backend/nest/prisma/schema.prisma

cd backend/nest

# Create a new migration
npm run prisma:migrate -- --name add_new_table

# Apply pending migrations
npm run prisma:migrate deploy

# Reset database (WARNING: deletes all data)
npm run prisma:migrate reset

# Check migration status
npx prisma migrate status
```

### Adding New Modules

```bash
# Generate new NestJS module
cd backend/nest/src
nest generate module {module-name}
nest generate controller {module-name}
nest generate service {module-name}

# Example: Add Admission module
nest generate module admission
nest generate controller admission
nest generate service admission

# Then implement controller and service following patterns
```

---

## 🐛 Troubleshooting

### Issue: "Cannot connect to PostgreSQL"

```bash
# Check if Docker service is running
docker-compose ps

# If not running, start it
docker-compose up -d

# Check PostgreSQL logs
docker-compose logs postgres

# Reset PostgreSQL
docker-compose down -v
docker-compose up -d postgres
```

### Issue: "Port 5432 already in use"

```bash
# Find process using port
lsof -i :5432  # macOS/Linux
netstat -ano | findstr :5432  # Windows

# Kill process
kill -9 <PID>  # macOS/Linux
taskkill /PID <PID> /F  # Windows

# Or use different port in docker-compose.yml
```

### Issue: "JWT_SECRET too short"

```bash
# Generate a long random string
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# Add to .env
JWT_SECRET="output-from-above"
```

### Issue: "Module not found" errors

```bash
# Clear node_modules and reinstall
rm -rf node_modules package-lock.json
npm install

# Or use npm cache clean
npm cache clean --force
npm install
```

### Issue: "Cannot find Prisma client"

```bash
# Regenerate Prisma client
npm run prisma:generate

# Or manually
npx prisma generate
```

### Issue: "Port 4000 already in use"

```bash
# Option 1: Change port in backend/nest/.env
PORT=4001

# Option 2: Kill existing process
lsof -i :4000
kill -9 <PID>

# Option 3: Use different port
npm run start:dev -- --port 4001
```

### Issue: "Seed script fails"

```bash
# Check seed file
cat backend/nest/prisma/seed.js

# Run seed manually
npm run prisma:seed

# Or with debugging
npm run prisma:seed -- --debug

# If still fails, run migrations first
npm run prisma:migrate deploy
npm run prisma:seed
```

---

## 📚 Project Structure

```
riphah-ums/
├── backend/
│   ├── nest/                 # Main NestJS application
│   │   ├── src/
│   │   │   ├── main.ts
│   │   │   ├── app.module.ts
│   │   │   ├── auth/
│   │   │   ├── users/
│   │   │   ├── roles/
│   │   │   └── ...
│   │   ├── prisma/
│   │   │   ├── schema.prisma
│   │   │   └── seed.js
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── .env
│   │
│   └── ai-services/          # Python FastAPI (optional)
│       ├── app.py
│       └── requirements.txt
│
├── frontend/                 # Next.js application
│   ├── app/
│   ├── components/
│   ├── lib/
│   ├── package.json
│   └── .env
│
├── docker-compose.yml        # Docker services
├── .env                      # Root environment
├── package.json              # Root package (workspaces)
├── README.md
└── docs/
    ├── IMPLEMENTATION_ROADMAP.md
    ├── RIPHAH_UMS_MASTER_DOCUMENT.md
    ├── NESTJS_MODULE_GENERATION_GUIDE.md
    └── SETUP_GUIDE.md
```

---

## 🎓 Key Files to Review

1. **[RIPHAH_UMS_MASTER_DOCUMENT.md](RIPHAH_UMS_MASTER_DOCUMENT.md)** - Complete project overview
2. **[IMPLEMENTATION_ROADMAP.md](IMPLEMENTATION_ROADMAP.md)** - 20-week timeline
3. **[NESTJS_MODULE_GENERATION_GUIDE.md](NESTJS_MODULE_GENERATION_GUIDE.md)** - Module patterns
4. **[backend/nest/README.md](backend/nest/README.md)** - Backend specific
5. **[docker-compose.yml](docker-compose.yml)** - Infrastructure config

---

## 📞 Getting Help

### Documentation
- Prisma Docs: https://www.prisma.io/docs
- NestJS Docs: https://docs.nestjs.com
- PostgreSQL: https://www.postgresql.org/docs
- Docker: https://docs.docker.com

### Common Issues Repo
Check the GitHub issues page for solutions to common problems

### Contact
- Tech Lead: [Contact Info]
- Backend Lead: [Contact Info]
- DevOps: [Contact Info]

---

## ✨ Next Steps After Setup

1. ✅ Verify all services are running
2. ✅ Confirm database is seeded
3. ✅ Test authentication (get JWT token)
4. ✅ Explore API documentation
5. 📖 Review IMPLEMENTATION_ROADMAP.md
6. 🚀 Start implementing Admission Module
7. 📝 Create first database migration

---

**Setup Completed!** 🎉

You now have a fully functional Riphah UMS development environment. 

Start with: `npm run dev`

---

**Last Updated**: June 1, 2026  
**Version**: 1.0.0
