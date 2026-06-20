# ComfiCurry — Free Prototype on Render (no credit card)

Everything on **one platform, one dashboard, no credit card**: API + Postgres +
both apps. This replaces the Oracle VM (Path B) since Render's free tier needs
no card. (Vercel/Supabase can host the two frontends + DB, but **not** the Java
API — so Render is the simplest single home for all of it.)

> Prototype only: guest login is on, the order cut-off is off, and the API is
> open. Great for a demo with fake data — not a real launch.

## Option 1 — One-click Blueprint (easiest)
1. Push `render.yaml` (in this repo) to a branch on GitHub if it isn't already.
2. In Render: **New → Blueprint**, pick the repo with `render.yaml`, **Apply**.
   It creates: a free Postgres, the API (Docker), and the two static sites —
   with the API URL and CORS list already wired between them.
3. Wait for all four to go live (first build ~5–10 min). Done.

URLs (if the names weren't already taken):
- Customer: `https://comficurry-proto-customer.onrender.com` (guest login)
- Admin: `https://comficurry-proto-admin.onrender.com`
- API: `https://comficurry-proto-api.onrender.com`

> If Render had to rename any service (the `*.onrender.com` name was taken),
> update `REACT_APP_API_URL` on both static sites and `CORS_ALLOWED_ORIGINS` on
> the API to the real URLs, then redeploy those services.

## Option 2 — Manual (if you prefer clicking, or the Blueprint errors)
Create four things in the Render dashboard (all **Free** plan, same region):

1. **PostgreSQL** → name `comficurry-proto-db`. Note its connection info.
2. **Web Service** → connect `comficurryapi`, runtime **Docker**. Env vars:
   - `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD` → from the DB above
   - `SPRING_JPA_HIBERNATE_DDL_AUTO=update`
   - `JWT_SECRET_KEY=<any long random string>`
   - `ORDERING_ENFORCE_WINDOW=false`
   - `LINE_CHANNEL_TOKEN`, `LINE_CHANNEL_SECRET` = `prototype-placeholder` (or real)
   - `CORS_ALLOWED_ORIGINS` = the customer + admin URLs (set after step 3/4), comma-separated
   - Deploy, note the API URL.
3. **Static Site** → connect `comficurryapp` (branch `main`). Build:
   `npm install && npm run build`, Publish dir: `dist`. Env:
   `REACT_APP_API_URL=<API URL>`, `REACT_APP_GUEST=true`. Add a **Rewrite Rule**:
   source `/*` → destination `/index.html` (so client-side routes work).
4. **Static Site** → connect `comficurryaaminapp` (branch `dev`). Build:
   `npm install && npm run build`, Publish dir: `build`. Env:
   `REACT_APP_API_URL=<API URL>`. Same `/*` → `/index.html` rewrite.
5. Go back to the API service, set `CORS_ALLOWED_ORIGINS` to the two static-site
   URLs, and redeploy it.

## Demo the end-to-end flow
1. **Admin** → log in (create the first user via **Register** if the `user`
   table is empty) → add Meal times/types, **Options** (dishes), **Carbs**,
   **Combo sets**; mark them **Active** (zones are auto-seeded).
2. **Customer** → opens in **guest mode** → welcome → Order Lunch/Dinner → pick
   combo + options → address (zone auto-fills) → pay by **QR Code + upload slip**
   or **COD**.
3. **Admin** → **Payment Review** verify the slip; **Orders / Kitchen / Packing /
   Daily Report / Subscribers** show the activity.

## Good to know
- **Warm it up before the demo:** free services sleep after ~15 min idle; open
  the API URL once so the first real request isn't slow.
- **Slips are ephemeral** on Render's free disk (lost on redeploy). Fine for a
  live demo; for persistence later, move uploads to object storage.
- **Free Postgres expires after 90 days** — prototype-only.
- **Before a real launch:** remove guest login, set `ORDERING_ENFORCE_WINDOW=true`,
  wire real LINE/LIFF, complete Security Phase 2, and rotate the leaked secrets.
