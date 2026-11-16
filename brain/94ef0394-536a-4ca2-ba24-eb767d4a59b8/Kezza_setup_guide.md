# Kezza Mobile — Full Setup Guide (macOS)

> [!IMPORTANT]
> **Your current system status:**
> - ✅ Python 3.9.6 — installed
> - ❌ Node.js / npm — NOT installed
> - ❌ PostgreSQL — NOT installed
> - ❌ Homebrew — NOT installed

---

## STEP 1 — Install Homebrew (macOS Package Manager)

Homebrew is used to install Node.js and PostgreSQL easily.

Open **Terminal** and run:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After install, follow the printed instructions to add brew to your PATH (usually for Apple Silicon Macs):
```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Verify:
```bash
brew --version
```

---

## STEP 2 — Install Node.js (for the Mobile App)

```bash
brew install node
```

Verify:
```bash
node --version    # should print v20.x or higher
npm --version     # should print 10.x or higher
```

---

## STEP 3 — Install PostgreSQL (Database)

```bash
brew install postgresql@16
```

Add PostgreSQL to your PATH:
```bash
echo 'export PATH="/opt/homebrew/opt/postgresql@16/bin:$PATH"' >> ~/.zprofile
source ~/.zprofile
```

Start PostgreSQL service:
```bash
brew services start postgresql@16
```

Verify it's running:
```bash
psql --version
psql postgres -c "SELECT version();"
```

---

## STEP 4 — Create the Kezza Database & User

```bash
psql postgres
```

Inside the `psql` prompt, run:
```sql
CREATE DATABASE kezza_db;
CREATE USER kezza_user WITH PASSWORD 'kezza_pass123';
GRANT ALL PRIVILEGES ON DATABASE kezza_db TO kezza_user;
\q
```

> [!TIP]
> You can choose any username and password — just use them consistently in the `.env` file below.

---

## STEP 5 — Configure Backend Environment

Create the file `backend/.env`:
```bash
cd /Users/ravi/.gemini/antigravity/scratch/Kezza_mobile/backend
```

Create the `.env` file with these contents:
```
DB_HOST=localhost
DB_PORT=5432
DB_NAME=kezza_db
DB_USER=kezza_user
DB_PASSWORD=kezza_pass123
```

---

## STEP 6 — Install Python Dependencies (Backend)

> [!NOTE]
> It is best practice to use a **virtual environment** to isolate Python packages.

```bash
cd /Users/ravi/.gemini/antigravity/scratch/Kezza_mobile/backend

# Create virtual environment
python3 -m venv .venv

# Activate it
source .venv/bin/activate

# Upgrade pip
pip install --upgrade pip

# Install all backend dependencies
pip install -r requirements.txt
```

Verify FastAPI is installed:
```bash
python3 -c "import fastapi; print(fastapi.__version__)"
```

---

## STEP 7 — Run Database Migrations

This creates all the tables (`users`, `patients`, `agents`, `branches`, `referrals`) in PostgreSQL:

```bash
# Still inside backend/ with .venv activated
alembic upgrade head
```

Verify tables were created:
```bash
psql -U kezza_user -d kezza_db -c "\dt"
```

---

## STEP 8 — Start the Backend API

```bash
cd /Users/ravi/.gemini/antigravity/scratch/Kezza_mobile/backend
source .venv/bin/activate
uvicorn app.main:app --reload
```

✅ API will be live at: **http://localhost:8000**

Test it in your browser:
- http://localhost:8000 → App info
- http://localhost:8000/health → Health check
- http://localhost:8000/health/db → DB connection status
- http://localhost:8000/docs → Swagger UI (auto-generated API docs)

---

## STEP 9 — Install Frontend Dependencies

Open a **new terminal tab**:

```bash
cd /Users/ravi/.gemini/antigravity/scratch/Kezza_mobile
npm install
```

---

## STEP 10 — Start the Mobile App

```bash
# In the Kezza_mobile root directory
npx expo start
```

You'll see a QR code and options:
- Press `i` → Open in **iOS Simulator** (requires Xcode)
- Press `a` → Open in **Android Emulator** (requires Android Studio)
- Press `w` → Open in **Web Browser** (easiest to test)
- Scan QR code with **Expo Go** app on your phone

> [!TIP]
> The quickest way to test without a simulator is:
> 1. Install **Expo Go** from the App Store / Play Store on your phone
> 2. Connect your phone to the same Wi-Fi as your Mac
> 3. Scan the QR code from `npx expo start`

---

## Summary Checklist

```
 STEP 1 — Install Homebrew
 STEP 2 — Install Node.js        (npm install + npx expo start)
 STEP 3 — Install PostgreSQL      (brew install postgresql@16)
 STEP 4 — Create DB & User        (psql postgres → CREATE DATABASE...)
 STEP 5 — Create backend/.env     (DB credentials)
 STEP 6 — Install Python deps     (pip install -r requirements.txt)
 STEP 7 — Run migrations          (alembic upgrade head)
 STEP 8 — Start Backend API       (uvicorn app.main:app --reload)
 STEP 9 — npm install             (frontend dependencies)
 STEP 10 — npx expo start         (run the mobile app)
```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `brew: command not found` | Re-run Step 1 and add brew to PATH |
| `psql: command not found` | Add PostgreSQL to PATH (Step 3) |
| `Missing environment variables` | Create `backend/.env` with DB credentials (Step 5) |
| `alembic: command not found` | Activate virtual environment: `source .venv/bin/activate` |
| `Connection refused` on `/health/db` | Make sure PostgreSQL service is running: `brew services start postgresql@16` |
| `npm: command not found` | Re-install Node.js (Step 2) |
| Expo QR code not working | Make sure phone and Mac are on the same Wi-Fi network |
