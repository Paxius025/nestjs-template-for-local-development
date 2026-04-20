# NestJS Template

> A backend template built with NestJS, PostgreSQL, Prisma, and Docker Compose.  
> Designed for rapid development with hot reload via Docker Compose Watch and Webpack HMR.

![NestJS](https://img.shields.io/badge/NestJS-E0234E?style=flat&logo=nestjs&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat&logo=postgresql&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=flat&logo=prisma&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-v20+-339933?style=flat&logo=node.js&logoColor=white)

---

## 🧱 Tech Stack

| Layer    | Technology         |
| -------- | ------------------ |
| Backend  | NestJS (Node.js)   |
| Database | PostgreSQL 15      |
| ORM      | Prisma             |
| Runtime  | Docker + Docker Compose |

---

## ✅ Prerequisites

Make sure you have the following installed before getting started:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (v4.24+ recommended — includes Docker Compose v2.22+ with Watch support)
- [Node.js](https://nodejs.org/) v20+ (only needed for local development outside Docker)
- [Git](https://git-scm.com/)

---

## 📁 Project Structure

```
template-nestjs/
├── docker-compose.yml        # Defines all services (backend, db)
├── .env                      # Environment variables (not committed to git)
├── .env.example              # Template for environment variables
└── backend/
    ├── Dockerfile            # Docker image for the NestJS app
    ├── nest-cli.json         # NestJS CLI config (Webpack HMR enabled)
    ├── webpack-hmr.config.js # Webpack Hot Module Replacement config
    ├── prisma/
    │   └── schema.prisma     # Database schema
    └── src/                  # Application source code
```

---

## 🚀 Getting Started

### 1. Clone the repository

```bash
git clone <your-repo-url>
cd  template-nestjs 
```

### 2. Set up environment variables

Copy the example env file and fill in your values:

```bash
cp .env.example .env
```

Open `.env` and configure the following variables:

```env
# PostgreSQL
POSTGRES_USER=your_db_user
POSTGRES_PASSWORD=your_db_password
POSTGRES_DB=your_db_name

# Prisma connection string (must match the values above)
DATABASE_URL=postgresql://your_db_user:your_db_password@db:5432/your_db_name
```

> **Note:** The hostname in `DATABASE_URL` must be `db` — this is the service name defined in `docker-compose.yml`, which Docker resolves automatically inside the container network.

### 3. Build and start the containers

```bash
docker compose up --build
```

This will:
1. Build the NestJS Docker image
2. Install all npm dependencies inside the container
3. Run `prisma generate` to generate the Prisma client
4. Start both `backend` (port 3000) and `db` (port 5432) services

Once running, the API is available at: **http://localhost:3000**

---

## ♻️ Development with Hot Reload

This project uses **Docker Compose Watch** combined with **Webpack HMR (Hot Module Replacement)** for a fast development experience — file changes are reflected inside the running container without restarting it.

### How it works

```
You edit a file on your machine
        ↓
Docker Compose Watch detects the change
        ↓
The file is synced into the running container instantly
        ↓
Webpack (inside the container) detects the updated file
        ↓
Only the changed module is reloaded — no full restart needed
```

### Start development mode with Watch

```bash
docker compose watch
```

> This is different from `docker compose up`. The `watch` command enables file sync and automatic rebuilds based on the rules defined in `docker-compose.yml`.

### Watch rules

| Path watched          | Action    | What happens                                              |
| --------------------- | --------- | --------------------------------------------------------- |
| `./backend/src/**`    | `sync`    | File is copied into `/app/src` in the container instantly |
| `./backend/package.json` | `rebuild` | The entire Docker image is rebuilt and container restarts |

- **`sync`** — Fast. No restart needed. Webpack picks up the change via polling (every 500ms).
- **`rebuild`** — Slower. Triggered only when dependencies change. Ensures `npm install` re-runs inside the container.

### Stop watching

Press `Ctrl+C` in the terminal running `docker compose watch`.

---

## 🗄️ Database Migrations

After modifying `prisma/schema.prisma`, run migrations inside the running container:

```bash
# Apply migrations to the database
docker compose exec backend npx prisma migrate dev --name <migration-name>

# Re-generate Prisma client after schema changes
docker compose exec backend npx prisma generate
```

---

## 🛠️ Useful Commands

```bash
# Start all services (no watch)
docker compose up

# Start all services and rebuild images
docker compose up --build

# Start with hot reload (development)
docker compose watch

# Stop all services
docker compose down

# Stop and remove volumes (resets the database)
docker compose down -v

# View logs from the backend service
docker compose logs -f backend

# Open a shell inside the backend container
docker compose exec backend sh

# Run a one-off command inside the backend container
docker compose exec backend <command>
```

---

## 🐛 Common Issues

### Hot reload is not detecting file changes

This is usually a filesystem event issue when running Docker on Windows. This project already handles it via `watchOptions.poll: 500` in `webpack-hmr.config.js`, so Webpack polls for changes every 500ms instead of relying on OS-level file events.

If changes still aren't detected, make sure you are running `docker compose watch`, not `docker compose up`.

### `Cannot find module` error on startup

The `node_modules` inside the container may be out of date. Rebuild the image and remove old volumes:

```bash
docker compose down -v
docker compose up --build
```

### Port 3000 or 5432 already in use

Another process is using the port. Either stop that process or change the port mapping in `docker-compose.yml`:

```yaml
ports:
  - "3001:3000"  # Change 3001 to any available port on your machine
```

### Database connection refused

Make sure the `DATABASE_URL` in your `.env` uses `db` as the hostname (not `localhost`), and that the `db` container is running:

```bash
docker compose ps
```
