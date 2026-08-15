# 📺 Makari Islamic TV

A complete, production-ready Islamic streaming platform dedicated to the lectures, tafsir, and teachings of **Malam Ibrahim Makari**.

---

## ✨ Features

| Feature | Status |
|---|---|
| Splash Screen + Onboarding | ✅ |
| Home Page (Featured, Trending, Live, Categories) | ✅ |
| Tafsir Section | ✅ |
| Audio Player (Stream, Download, Speed, Sleep Timer) | ✅ |
| Video Player (YouTube embed + Direct stream) | ✅ |
| Live Streaming | ✅ |
| 10 Content Categories | ✅ |
| Full-text Search | ✅ |
| Favorites System | ✅ |
| Downloads (offline) | ✅ |
| Push Notifications | ✅ |
| Islamic Library (Books/PDFs) | ✅ |
| Prayer Times (GPS-based) | ✅ |
| Qibla Direction Compass | ✅ |
| User Authentication (Register/Login) | ✅ |
| Admin Dashboard | ✅ |
| Dark Mode + Light Mode | ✅ |
| Hausa + English Language | ✅ |
| Mobile-first Responsive Design | ✅ |
| Skeleton Loaders + Shimmer Effects | ✅ |

---

## 🛠️ Tech Stack

- **Frontend:** HTML5, CSS3, Vanilla JavaScript (Mobile-first PWA)
- **Backend:** Python FastAPI
- **Database:** SQLite (dev) / PostgreSQL (production)
- **Auth:** JWT (python-jose + bcrypt)
- **Media:** Static file serving + YouTube embed + Direct stream
- **Deployment:** Render / Railway

---

## 🚀 Quick Start

### 1. Backend Setup

```bash
cd backend

# Install dependencies
pip install -r requirements.txt

# Copy env file
cp .env.example .env

# Seed database with demo content
python seed.py

# Run the server
uvicorn main:app --reload --port 8000
```

API will be live at: `http://localhost:8000`
API Docs: `http://localhost:8000/docs`

### 2. Frontend

Simply open `frontend/index.html` in a browser, or serve it:

```bash
cd frontend
python -m http.server 3000
# Open http://localhost:3000
```

The frontend auto-detects the API at `localhost:8000` in development.

---

## 🔐 First Admin Account

There is no default/hardcoded admin account — every deployment used to share
the same public `admin@makariilamictv.com` / `admin123` login, which was a
serious security hole and has been removed.

To create your own admin, set these two environment variables before starting
the server (locally in `.env`, or in the Render dashboard for production):

```
ADMIN_EMAIL=you@example.com
ADMIN_PASSWORD=a-strong-password-min-8-chars
```

The app creates the admin account automatically on startup if it doesn't
already exist. You can also trigger it on demand by calling
`GET /api/auth/setup` (or `/api/admin/setup`) after setting the env vars —
useful if you added them after the server was already running.

---

## 📁 Project Structure

```
makari-islamic-tv/
├── frontend/
│   └── index.html          # Complete single-page app
├── backend/
│   ├── main.py             # FastAPI entry point
│   ├── database.py         # DB connection + session
│   ├── models.py           # All SQLAlchemy models
│   ├── auth_utils.py       # JWT + password utilities
│   ├── seed.py             # Demo data seeder
│   ├── requirements.txt
│   ├── .env.example
│   └── routers/
│       ├── auth.py
│       ├── users.py
│       ├── lectures.py
│       ├── videos.py
│       ├── audio.py
│       ├── categories.py
│       ├── search.py
│       ├── favorites.py
│       ├── downloads.py
│       ├── live.py
│       ├── library.py
│       ├── notifications.py
│       ├── admin.py
│       └── prayer.py
└── render.yaml
```

---

## 🌐 API Endpoints

### Auth
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/auth/register` | Register new user |
| POST | `/api/auth/login` | Login |
| GET | `/api/auth/me` | Current user |

### Lectures
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/lectures/` | All lectures (paginated) |
| GET | `/api/lectures/featured` | Featured lectures |
| GET | `/api/lectures/trending` | Trending |
| GET | `/api/lectures/latest` | Latest |
| GET | `/api/lectures/most-viewed` | Most viewed |
| GET | `/api/lectures/{id}` | Single lecture |
| POST | `/api/lectures/` | Create (admin) |
| PUT | `/api/lectures/{id}` | Update (admin) |
| DELETE | `/api/lectures/{id}` | Delete (admin) |

### Media
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/videos/` | Video list |
| POST | `/api/videos/` | Attach video by URL (admin) |
| POST | `/api/videos/upload` | Upload video file (admin) |
| GET | `/api/audio/` | Audio list |
| POST | `/api/audio/` | Attach audio by URL (admin) |
| POST | `/api/audio/upload` | Upload audio file (admin) |

### Categories
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/categories/` | All categories |
| GET | `/api/categories/{slug}/lectures` | Lectures by category |
| POST | `/api/categories/` | Create (admin) |

### Live
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/live/` | All streams |
| GET | `/api/live/active` | Currently live |
| POST | `/api/live/` | Create stream (admin) |
| PATCH | `/api/live/{id}/toggle` | Go live/offline |

### Library
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/library/books` | All books |
| GET | `/api/library/books/{id}` | Single book |
| POST | `/api/library/books/upload` | Upload PDF |

### Favorites & Downloads
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/favorites/` | User's favorites |
| POST | `/api/favorites/` | Add favorite |
| DELETE | `/api/favorites/{id}` | Remove favorite |
| DELETE | `/api/favorites/lecture/{lecture_id}` | Remove favorite by lecture id |
| GET | `/api/downloads/` | User's download records |
| POST | `/api/downloads/` | Start/register a download |
| PATCH | `/api/downloads/{id}` | Update progress/status |
| DELETE | `/api/downloads/{id}` | Remove a download record |

### Watch History
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/users/history` | Recently watched lectures |
| POST | `/api/lectures/{id}/watch` | Record/update watch progress |

### Prayer
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/prayer/times?lat=X&lng=Y` | Prayer times |
| GET | `/api/prayer/qibla?lat=X&lng=Y` | Qibla direction |

### Admin
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/admin/stats` | Dashboard stats |
| GET | `/api/admin/users` | All users |
| PATCH | `/api/admin/users/{id}/toggle` | Enable/disable user |
| PATCH | `/api/admin/users/{id}/role` | Change user role |

### Notifications
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/notifications/` | User notifications |
| GET | `/api/notifications/unread-count` | Unread count (for the bell badge) |
| POST | `/api/notifications/send` | Broadcast (admin) |
| PATCH | `/api/notifications/{id}/read` | Mark one read |
| PATCH | `/api/notifications/read-all` | Mark all read |

### Search
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/search/?q=QUERY` | Full-text search |

---

## ☁️ Deploy to Render

1. Push code to GitHub
2. Use the provided root `render.yaml` (Render → New → Blueprint) for
   automatic setup of the web service + PostgreSQL database, **or** create
   the pieces manually:
   - New **Web Service**, root directory `backend/`
   - Build command: `./build.sh`
   - Start command: `uvicorn main:app --host 0.0.0.0 --port $PORT`
   - Health check path: `/health`
3. Set these environment variables on the service:
   - `SECRET_KEY` — generate with `python -c "import secrets; print(secrets.token_hex(32))"` (or let Render auto-generate it via the blueprint)
   - `DATABASE_URL` — from your Render PostgreSQL instance
   - `ENVIRONMENT=production`
   - `ADMIN_EMAIL` / `ADMIN_PASSWORD` — see "First Admin Account" above
   - `ALLOWED_ORIGINS` — your deployed frontend URL(s), comma-separated
4. For the frontend: create a **Static Site** pointing to `frontend/`

> **Media storage note:** Render's free web-service plan does not support
> persistent disks, so anything saved to `backend/media/` (uploaded videos,
> audio, PDFs) is lost on every redeploy/restart. For production, either
> upgrade to a paid instance with a persistent disk, or — recommended —
> use object storage (Cloudflare R2, AWS S3, Backblaze B2) instead of local
> disk for uploaded media.

---

## 🚂 Deploy to Railway

```bash
# Install Railway CLI
npm i -g @railway/cli

# Login and deploy backend
cd backend
railway login
railway init
railway up

# Set env vars
railway variables set SECRET_KEY=your-secret-key
railway variables set DATABASE_URL=postgresql://...
```

---

## 🎨 Design System

| Token | Value |
|---|---|
| Primary Green | `#1e7e3c` |
| Gold Accent | `#c9a227` |
| Background | `#0b1410` |
| Card BG | `#141f19` |
| Font Display | Amiri (Arabic-style) |
| Font Body | Outfit |

---

## 📱 PWA Support

PWA support is already fully implemented — installable manifest with real
icons (`frontend/icons/`), a versioned service worker (`frontend/sw.js`)
with network-first HTML and offline fallback, and an install-prompt banner
(`frontend/pwa.js`). Just deploy `frontend/` over HTTPS and it's installable
as-is; nothing further to configure.

## 🤖 Turn it into an Android App

See [`docs/ANDROID.md`](docs/ANDROID.md) for the full step-by-step guide —
a GitHub Actions workflow (`.github/workflows/build-android.yml`) builds a
signed Android app bundle from this PWA using Google's Bubblewrap tool. A
few one-time, interactive steps (choosing a package name, generating a
signing key) can't be automated sight-unseen, but the guide walks through
exactly what to do and why each step matters.

---

## ⚠️ Known Limitations (require external services to complete)

- **Media persistence:** uploaded files are lost on redeploy on Render's free
  plan (see the deploy section above) — use object storage in production.
- **Push notifications:** the service worker has a `push` event handler, but
  there is no VAPID key pair, subscription endpoint, or push-sending backend
  wired up. In-app notifications (bell icon) work fully; OS-level push when
  the app is closed does not, and would need Firebase Cloud Messaging or
  `pywebpush` + a subscription table added.
- **Database migrations:** the app uses `Base.metadata.create_all()`, which
  only creates missing tables — it will not alter existing columns/indexes
  on an already-deployed database. For schema changes after your first
  deploy, add [Alembic](https://alembic.sqlalchemy.org/) migrations rather
  than relying on `create_all()`.
- **PDF reading:** `openBook()` opens the PDF in a new browser tab using the
  browser's native viewer — there's no in-app reader.

---

## 📄 License

Built for **Makari Islamic TV** — All rights reserved.

---

*Alhamdulillah — May Allah bless this project and make it beneficial to the Ummah.*
