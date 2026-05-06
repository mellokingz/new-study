# Nexus Study 🚀

AI-powered study app — flashcards, quizzes, summaries, spaced repetition, and a personal AI tutor.

## Stack

| Layer | Tech | Deployed on |
|---|---|---|
| Frontend | Next.js 14 + Tailwind | Vercel |
| Backend | Node.js + Express | Railway |
| Database | PostgreSQL | Railway (plugin) |
| AI | Anthropic Claude | — |
| CI/CD | GitHub Actions | — |

---

## Repository layout

```
nexus-study/                  ← monorepo root
├── .github/
│   └── workflows/
│       ├── ci.yml            ← runs on every PR (lint, typecheck, tests)
│       └── deploy.yml        ← runs on merge to main (Vercel + Railway)
├── frontend/                 ← Next.js app
│   ├── src/
│   │   ├── app/              ← App Router pages
│   │   │   ├── auth/         ← login, register
│   │   │   ├── dashboard/
│   │   │   ├── notes/
│   │   │   ├── flashcards/
│   │   │   ├── quiz/
│   │   │   ├── summary/
│   │   │   ├── mastery/
│   │   │   └── chat/
│   │   └── lib/
│   │       ├── api.ts        ← typed API client
│   │       └── auth-context.tsx
│   ├── vercel.json
│   └── package.json
├── backend/                  ← Express API
│   ├── src/
│   │   ├── controllers/
│   │   ├── routes/
│   │   ├── services/         ← AI + file processing
│   │   ├── middleware/
│   │   └── utils/
│   ├── prisma/schema.prisma
│   ├── Dockerfile
│   ├── railway.toml
│   └── package.json
└── package.json              ← workspace root
```

---

## Local development

### Prerequisites
- Node.js 20+
- Docker (for PostgreSQL)

### 1. Clone & install

```bash
git clone https://github.com/YOUR_USERNAME/nexus-study.git
cd nexus-study
npm install
```

### 2. Start PostgreSQL

```bash
docker compose -f backend/docker-compose.yml up -d
```

### 3. Configure environment

```bash
# Backend
cp backend/.env.example backend/.env
# Edit backend/.env — set ANTHROPIC_API_KEY and JWT_SECRET

# Frontend
cp frontend/.env.example frontend/.env.local
# NEXT_PUBLIC_API_URL=http://localhost:4000/api  (already set)
```

### 4. Set up database

```bash
cd backend
npx prisma generate
npx prisma db push
node prisma/seed.js    # loads demo data
cd ..
```

### 5. Run both servers

```bash
npm run dev
# Backend:  http://localhost:4000
# Frontend: http://localhost:3000
```

Demo login: `demo@nexusstudy.com` / `demo1234`

---

## Deploying to GitHub → Vercel + Railway

### Step 1 — Push to GitHub

```bash
git init
git add .
git commit -m "feat: initial Nexus Study MVP"
git remote add origin https://github.com/YOUR_USERNAME/nexus-study.git
git push -u origin main
```

### Step 2 — Deploy backend on Railway

1. Go to [railway.app](https://railway.app) → **New Project**
2. Select **Deploy from GitHub repo** → choose `nexus-study`
3. Set **Root directory**: `backend`
4. Add a **PostgreSQL plugin** (Railway creates and links `DATABASE_URL` automatically)
5. Add environment variables in Railway dashboard:

```
NODE_ENV=production
JWT_SECRET=<generate: node -e "console.log(require('crypto').randomBytes(48).toString('hex'))">
ANTHROPIC_API_KEY=sk-ant-...
FRONTEND_URL=https://nexus-study.vercel.app
```

6. Copy your Railway service URL (e.g. `https://nexus-study-backend.up.railway.app`)

### Step 3 — Deploy frontend on Vercel

1. Go to [vercel.com](https://vercel.com) → **Add New Project**
2. Import `nexus-study` from GitHub
3. Set **Root directory**: `frontend`
4. Framework: **Next.js** (auto-detected)
5. Add environment variable:
   - `NEXT_PUBLIC_API_URL` = `https://nexus-study-backend.up.railway.app/api`
6. Deploy → copy your Vercel domain

### Step 4 — Update CORS in Railway

Go back to Railway and update:
```
FRONTEND_URL=https://your-app.vercel.app
```

### Step 5 — Add GitHub Secrets for auto-deploy

In your GitHub repo → **Settings → Secrets and variables → Actions**, add:

| Secret | Where to find it |
|---|---|
| `VERCEL_TOKEN` | vercel.com → Settings → Tokens |
| `VERCEL_ORG_ID` | vercel.com → Settings → General → Team ID |
| `VERCEL_PROJECT_ID` | Vercel project → Settings → General → Project ID |
| `RAILWAY_TOKEN` | railway.app → Account → Tokens |
| `NEXT_PUBLIC_API_URL` | Your Railway backend URL + `/api` |

After adding secrets, every push to `main` will:
1. Run CI (lint + typecheck + tests)
2. Deploy frontend to Vercel
3. Deploy backend to Railway
4. Run a health check

---

## Environment variables reference

### Backend (`backend/.env`)

| Variable | Required | Description |
|---|---|---|
| `DATABASE_URL` | ✅ | PostgreSQL connection string |
| `JWT_SECRET` | ✅ | 48+ random chars |
| `ANTHROPIC_API_KEY` | ✅ | Your Anthropic key |
| `FRONTEND_URL` | ✅ | Vercel domain (for CORS) |
| `PORT` | — | Default 4000 |
| `NODE_ENV` | — | `development` or `production` |
| `AI_RATE_LIMIT_MAX` | — | AI calls/hr per user (default 20) |

### Frontend (`frontend/.env.local`)

| Variable | Required | Description |
|---|---|---|
| `NEXT_PUBLIC_API_URL` | ✅ | Railway backend URL + `/api` |

---

## API routes quick reference

```
POST   /api/auth/register
POST   /api/auth/login
GET    /api/auth/me

GET    /api/notes
POST   /api/notes                  ← paste text
POST   /api/notes/upload           ← file upload (PDF/TXT/MD)
GET    /api/notes/:id/status       ← poll AI generation

GET    /api/flashcards/sets
GET    /api/flashcards/due
POST   /api/flashcards/cards/:id/rate

GET    /api/quiz
POST   /api/quiz/:id/attempt

GET    /api/summary/note/:noteId

POST   /api/chat/message
POST   /api/chat/explain

GET    /api/progress/dashboard
GET    /api/progress/activity
```

---

## Branch strategy

```
main      ← production (auto-deploys)
develop   ← integration branch
feature/* ← feature branches (PR → develop)
```

PRs to `main` or `develop` trigger CI checks automatically.
