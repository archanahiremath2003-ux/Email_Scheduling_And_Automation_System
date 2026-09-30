# 🚀 Email Scheduler & Dispatcher

A high-performance, resilient, and production-grade email scheduling dashboard built with **Node.js**, **Express**, **TypeScript**, **React**, **Vite**, **Tailwind CSS**, **BullMQ**, **Redis**, and **Prisma** (PostgreSQL/SQLite).

---

## ✨ Features & Architecture

### 1. 🔐 Google Authentication
- Direct Google OAuth 2.0 Sign-In.
- Users authenticate directly using their Google accounts.
- Integrates with Google's OAuth authorization servers and verifies credentials.

### 2. 📝 Compose Email & Template Engine
- Personalized subject & body with dynamic tags: `{{name}}`, `{{company}}`, `{{email}}`.
- Live preview tab to visualize rendered HTML emails before scheduling.
- Sender name and email customization.

### 3. 📁 CSV Upload & Duplicate Prevention
- Drag-and-drop CSV upload with auto-detection of `email`, `name`, and `company` columns.
- **Deduplication Engine**:
  - Automatically identifies and filters intra-file duplicate email addresses.
  - Implements a unique database constraint `dedupKey = campaignId:recipientEmail` to ensure no recipient is ever emailed twice within the same campaign.
- Built-in **"Download Sample CSV"** button with realistic sample recipients.

### 4. ⏰ Flexible Scheduling & Staggered Pacing
- **Send Immediately**: Pushes directly into the BullMQ queue with configured pacing.
- **Schedule for Later**: Interactive date-time picker with quick presets (+15m, +1h, Tomorrow).
- **Stagger Delay**: Configurable interval (e.g., 2s, 5s, 10s) between consecutive emails to prevent spam filters.

### 5. ⚡ Hourly Rate Limiting & Slack Webhook Alerts
- Configurable hourly sending limit (e.g., 50 or 100 emails/hour).
- Uses Redis atomic counters combined with persistent database tracking across hour windows (`YYYY-MM-DDTHH`).
- When the limit is reached:
  - Jobs are automatically paused or delayed until the next hour window.
  - An instant **Slack Incoming Webhook alert** is fired with campaign details, current hour volume, and resume timestamp.
  - Alert suppression prevents duplicate notification spam during the same window.
  - Dedicated **"Send Test Slack Notification"** button in Settings.

### 6. 🔄 Server Restart Survival (100% Durability)
- BullMQ delayed and waiting jobs are persisted on disk via Redis RDB/AOF.
- On startup, the backend's **`RecoveryService`** automatically queries the database for any pending or scheduled emails.
- If any job was interrupted, missing, or scheduled during downtime, it recalculates remaining delay and safely re-enqueues it with pacing.

### 7. 📬 Ethereal SMTP Sandbox & Live Preview Links
- Real test emails sent through **Ethereal SMTP**.
- Automatically saves and displays a **"View in Ethereal"** preview link for every dispatched email.
- Credentials persist across restarts.

### 8. 🔍 Elasticsearch Search with Database Fallback
- Searches subject, recipient email, recipient name, and email body using Elasticsearch `multi_match` with fuzziness.
- **Resilient Fallback**: If an Elasticsearch cluster is not running at `http://localhost:9200`, the system automatically falls back to full-text database queries without failing.

### 9. 📊 Bull Board Queue Monitor
- Full **Bull Board UI** mounted at `/admin/queues`.
- Also embedded as an interactive queue monitor page inside the React dashboard with metrics (Active, Waiting, Delayed, Completed, Failed) and Pause/Resume controls.

---

## 🛠️ Project Structure

```text
├── backend/
│   ├── prisma/
│   │   ├── schema.prisma          # SQLite schema (default zero-config)
│   │   └── schema.postgres.prisma # PostgreSQL production schema
│   ├── src/
│   │   ├── config/redis.ts        # Redis client & auto-spawner
│   │   ├── controllers/           # Auth, Campaign, Email, Queue, Settings
│   │   ├── routes/                # Express API routes
│   │   ├── services/
│   │   │   ├── db.ts              # Prisma client singleton
│   │   │   ├── emailService.ts    # Ethereal SMTP & preview links
│   │   │   ├── elasticsearchService.ts # Elasticsearch + DB fallback
│   │   │   ├── queueService.ts    # BullMQ campaign scheduling
│   │   │   ├── rateLimiterService.ts # Hourly rate limiter & Slack alerts
│   │   │   ├── recoveryService.ts # Server restart reconciliation
│   │   │   └── slackService.ts    # Slack incoming webhook alerts
│   │   ├── workers/
│   │   │   └── emailWorker.ts     # BullMQ email dispatch worker
│   │   ├── bullBoard.ts           # Bull Board express adapter
│   │   └── server.ts              # Express application bootstrap
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── components/            # Sidebar, RescheduleModal
│   │   ├── context/               # AuthContext (Google OAuth)
│   │   ├── pages/
│   │   │   ├── ComposePage.tsx    # Compose, CSV upload, schedule, pacing
│   │   │   ├── ScheduledEmailsPage.tsx # Scheduled queue, search, reschedule
│   │   │   ├── SentEmailsPage.tsx # Sent history, Ethereal preview URLs
│   │   │   ├── QueueStatusPage.tsx # Bull Board embed & live counters
│   │   │   ├── SettingsPage.tsx   # Slack webhook & Ethereal test
│   │   │   └── LoginPage.tsx      # Google login screen
│   │   ├── api.ts                 # Axios client
│   │   ├── App.tsx                # Routing & layout
│   │   └── main.tsx
│   └── package.json
├── bin/redis/                     # Bundled portable Redis 8.10.1 for Windows
├── docker-compose.yml             # Docker stack for Postgres, Redis, Elasticsearch
└── package.json                   # Root orchestrator scripts
```

---

## 📦 How to Share This Source Code with a Friend

When sending this project as a ZIP file or via Git, make sure to **exclude bulky dependency folders**:
- **DO NOT include**: `node_modules/`, `frontend/dist/`, `.git/` (if sending a ZIP), or local `.env` with private keys.
- **DO include**:
  - `backend/` (source code, prisma schemas, package.json, `.env.example`)
  - `frontend/` (source code, package.json, vite.config.ts)
  - `bin/redis/` (contains portable Redis for Windows, no installation needed)
  - `docker-compose.yml`, `package.json`, `README.md`

Your friend will be able to install dependencies and run the project in minutes using the steps below.

---

## 🛠️ Prerequisites

Before running the project, ensure you have:
1. **Node.js** (v18 or v20+ recommended) — [Download Node.js](https://nodejs.org/)
2. **npm** (comes with Node.js)
3. **Redis**:
   - **Windows**: No installation needed! The project includes a portable Redis binary in `bin/redis/`.
   - **macOS**: `brew install redis && brew services start redis`
   - **Linux**: `sudo apt install redis-server && sudo systemctl start redis`
   - **Docker (Any OS)**: `docker compose up -d redis`

---

## 🚀 Step-by-Step Manual Setup

### Step 1: Open the Project
Open a terminal in the project root directory (`website/`).

---

### Step 2: Configure Environment Variables
Navigate to the `backend/` directory and ensure a `.env` file exists:

```bash
cd backend
cp .env.example .env
```
*(On Windows PowerShell, use `Copy-Item .env.example .env`)*

> [!NOTE]
> The default `.env` uses **SQLite** (`DATABASE_URL="file:./dev.db"`), which requires **zero database installation**. You can start testing immediately!

---

### Step 3: Install Dependencies & Prepare Database

#### Quick 1-Command Setup (from root):
```bash
# From project root:
npm run setup
```
*This command automatically installs root, backend, and frontend dependencies, and pushes the Prisma database schema.*

#### Or Manual Terminal-by-Terminal Setup:
1. **Backend setup:**
   ```bash
   cd backend
   npm install
   npx prisma db push
   ```
2. **Frontend setup:**
   ```bash
   cd ../frontend
   npm install
   ```

---

### Step 4: Start Redis Server

- **Option A (Windows with bundled Redis - easiest)**:
  Open a new terminal window:
  ```powershell
  cd bin/redis
  .\redis-server.exe
  ```
  *(Or from root: `npm run start:redis`)*

- **Option B (macOS/Linux native Redis)**:
  ```bash
  redis-server
  ```

- **Option C (Docker - any platform)**:
  ```bash
  docker compose up -d redis
  ```

---

### Step 5: Start the Backend Server

Open a new terminal window:
```bash
cd backend
npm run dev
```

You should see:
```text
[Redis] Connected to Redis at 127.0.0.1:6379
[Database] Prisma connected to dev.db
[Recovery] Reconciled pending scheduled emails
[Server] 🚀 Server running at http://localhost:5000
```

---

### Step 6: Start the Frontend Application

Open another terminal window:
```bash
cd frontend
npm run dev
```

The frontend will start at: **http://localhost:3000**

---

## 🔑 How to Log In

1. Open your browser and go to: **`http://localhost:3000`**
2. **Instant Test Access**:
   - Click the **"Demo Login (For Test)"** button.
   - You will instantly be logged in as `Test User` without any Google credentials needed!
3. **Real Google Sign-In (Optional)**:
   - Click **"Sign in with Google"**.
   - If not yet configured, enter your Google Cloud OAuth Client ID and Secret in the modal to authenticate directly through Google.

---

## 🧪 Testing Core Features

1. **Compose & Staggered Dispatch**:
   - Go to **Compose Email**.
   - Click **"Download Sample CSV"** to get a formatted list of test recipients.
   - Upload the CSV, write a subject and message using tags like `Hello {{name}}`.
   - Set pacing delay (e.g. 2 seconds) and an hourly limit (e.g. 50 emails/hr).
   - Click **"Schedule Campaign"**.
2. **View Real Test Emails in Ethereal**:
   - Switch to the **"Sent Emails"** tab.
   - Dispatched emails will have a **"View in Ethereal"** button that opens the real rendered email in your browser sandbox.
3. **Monitor Live Queue (Bull Board)**:
   - Go to **Queue Status** in the sidebar, or visit **`http://localhost:5000/admin/queues`** to view active BullMQ jobs in real time.
4. **Hourly Rate Limit & Slack Alert**:
   - Go to **Settings**, paste an incoming Slack Webhook URL, and test the notification!
