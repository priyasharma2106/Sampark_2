# Sampark Deployment Checklist (Local + Supabase + Render + Vercel)

Follow this in order. Do not skip steps.

## 1) Local backend env (`Backend/.env`)

Copy example and set real values:

```powershell
Set-Location d:/Sampark/Sampark/Backend
Copy-Item .env.example .env
```

Required keys:

```env
DATABASE_URL=postgresql://postgres:<YOUR_DB_PASSWORD>@db.<PROJECT_REF>.supabase.co:5432/postgres
SECRET_KEY=<LONG_RANDOM_SECRET_FOR_JWT>
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=10080
HOST=0.0.0.0
PORT=8000
CORS_ORIGINS=http://localhost:5173,https://<your-vercel-domain>.vercel.app,https://<your-custom-domain>
```

## 2) Frontend local env (`frontend/.env`)

```powershell
Set-Location d:/Sampark/Sampark/frontend
Copy-Item .env.example .env
```

```env
VITE_API_URL=http://127.0.0.1:8000
```

## 3) Supabase: where to get DB link + secrets

### A) DB connection string (`DATABASE_URL`)
1. Open Supabase project dashboard.
2. Go to **Project Settings → Database**.
3. Under **Connection string**, choose **URI**.
4. Copy URI and replace `[YOUR-PASSWORD]` with your DB password.

Format (direct connection):

```text
postgresql://postgres:<DB_PASSWORD>@db.<PROJECT_REF>.supabase.co:5432/postgres
```

### B) If password unknown/reset
1. Go to **Project Settings → Database**.
2. Click **Reset database password** (or change password option).
3. Save the new password in your password manager.
4. Update `DATABASE_URL` everywhere (local `.env`, Render env).

### C) Supabase API keys (optional for this project currently)
- Go to **Project Settings → API**.
- You will see:
  - `anon` public key (safe for frontend usage when needed)
  - `service_role` secret key (backend only, never expose)

> Current Sampark backend uses direct Postgres connection (`DATABASE_URL`) and JWT `SECRET_KEY`.
> It does not currently require Supabase JS client keys unless you add Supabase API usage.

## 4) Generate backend `SECRET_KEY` (JWT signing key)

This is **not** your Supabase API key.

```powershell
d:/Sampark/.venv/Scripts/python.exe -c "import secrets; print(secrets.token_urlsafe(64))"
```

Paste output into:

```env
SECRET_KEY=<PASTE_VALUE>
```

## 5) Run DB migration + seed

```powershell
Set-Location d:/Sampark/Sampark/Backend
d:/Sampark/.venv/Scripts/python.exe -m alembic upgrade head
d:/Sampark/.venv/Scripts/python.exe seed_data.py
```

Expected seed login:
- `admin / admin123`
- `ramesh.kumar / password123`
- `sita.devi / password123`
- `mohan.singh / password123`
- `block.officer / officer123`

## 6) Run locally

Backend:

```powershell
Set-Location d:/Sampark/Sampark/Backend
d:/Sampark/.venv/Scripts/python.exe -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

Frontend:

```powershell
Set-Location d:/Sampark/Sampark/frontend
npm run dev -- --host 127.0.0.1 --port 5173
```

## 7) Render backend env vars

Set in Render service environment:
- `DATABASE_URL` = Supabase Postgres URI
- `SECRET_KEY` = same JWT secret used for backend
- `ALGORITHM=HS256`
- `ACCESS_TOKEN_EXPIRE_MINUTES=10080`
- `CORS_ORIGINS=https://<your-vercel-domain>.vercel.app,https://<custom-domain>,http://localhost:5173`

Then redeploy.

## 8) Vercel frontend env vars

Set in Vercel project:
- `VITE_API_URL=https://<your-render-backend-domain>`

Redeploy frontend.

## 9) Final verification

1. Open frontend URL.
2. Login with `admin / admin123`.
3. Confirm requests go to Render API (not localhost).
4. Confirm no CORS error in browser console.
5. Confirm survey create/read works.

## 10) Common failure map

- **401 invalid credentials**: DB not seeded or wrong DB URL.
- **Network error from frontend**: `VITE_API_URL` missing/wrong.
- **CORS blocked**: missing frontend domain in `CORS_ORIGINS`.
- **Backend boot fails**: missing `DATABASE_URL` or `SECRET_KEY`.
