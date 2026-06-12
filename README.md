# ComfiCurry — Deployment & Local Development

ComfiCurry is made up of three applications plus this orchestration repo:

| Repo | What it is | Local port |
|---|---|---|
| [comficurryapi](https://github.com/COMFICURRY/comficurryapi) | Spring Boot 3 / Java 17 backend API (PostgreSQL) | 8080 |
| [comficurryaaminapp](https://github.com/COMFICURRY/comficurryaaminapp) | Admin web app (React / Create React App) | 9090 |
| [comficurryapp](https://github.com/COMFICURRY/comficurryapp) | Customer ordering web app (React / Parcel, LINE LIFF) | 9091 |
| comficurrydocker | docker-compose files (this repo) | — |

## Run the whole stack locally

1. Install Docker (Docker Desktop, or `colima` + `docker` on macOS).
2. Clone all four repos **side by side** in one folder:

   ```sh
   mkdir comficurry && cd comficurry
   gh repo clone COMFICURRY/comficurryapi
   gh repo clone COMFICURRY/comficurryaaminapp
   gh repo clone COMFICURRY/comficurryapp
   gh repo clone COMFICURRY/comficurrydocker
   ```

3. Start everything:

   ```sh
   docker compose -f comficurrydocker/docker-compose.dev.yml up -d --build
   ```

4. Open:
   - API: http://localhost:8080
   - Admin app: http://localhost:9090
   - Customer app: http://localhost:9091
   - Postgres: `localhost:5432`, user `postgres` / password `root`, database `comficurry`

The database starts empty; Hibernate creates the schema when the API boots.
Register an admin user via `POST /auth/signup` (role `admin`) to log into the
admin app.

## Configuration / secrets

The API reads all secrets from environment variables (see
`comficurryapi/.env.example`):

| Variable | Purpose |
|---|---|
| `SPRING_DATASOURCE_URL` / `_USERNAME` / `_PASSWORD` | PostgreSQL connection |
| `LINE_CHANNEL_TOKEN` / `LINE_CHANNEL_SECRET` | LINE Messaging API (only needed when testing LINE features) |
| `JWT_SECRET_KEY` | JWT signing key (defaults to a dev-only key) |

Never commit real values. For production deployments set these in the host
platform (Render dashboard, etc.).

## Other files in this repo

- `docker-compose.yml` / `docker-compose1.yml` — the original consultants'
  files. They expect the app repos as git submodules inside this repo and
  contain hardcoded credentials; superseded by `docker-compose.dev.yml` for
  local work. Kept for reference until production deployment is reworked.
