# ComfiCurry — Prototype Deployment on One Free VM (Path B)

Goal: run the **whole stack on a single server** so the CEO can see the
end-to-end flow — admin adds the menu, a (guest) customer orders, the admin
reviews the order and the payment slip.

> This is a throwaway prototype: the API is open, the order cut-off is off,
> guest login is on. Fine for a demo with fake data — not a real launch.

---

## 1. Get a free VM
Any small Linux VM works. Free options:
- **Oracle Cloud — Always Free** (best value: ARM, up to 4 vCPU / 24 GB RAM, free forever). Pick an **Ubuntu 22.04** "Ampere (ARM)" instance.
- **Google Cloud** `e2-micro` free tier, or any other free VM.

Open these inbound ports on the VM (cloud firewall **and** the OS firewall):
`22` (SSH), `80` (customer app), `8090` (admin app), `8080` (API).
Leave Postgres (`5432`) closed — it's internal to the compose network.

## 2. Install Docker
```bash
sudo apt-get update && sudo apt-get install -y docker.io docker-compose-plugin git
sudo usermod -aG docker $USER && newgrp docker   # run docker without sudo
```

## 3. Clone the four repos as siblings
```bash
mkdir -p ~/comficurry && cd ~/comficurry
git clone https://github.com/COMFICURRY/comficurryapi.git
git clone https://github.com/COMFICURRY/comficurryaaminapp.git -b dev
git clone https://github.com/COMFICURRY/comficurryapp.git
git clone https://github.com/COMFICURRY/comficurrydocker.git
```
(Customer + API default branches are correct; the admin app's default is `dev`.)

## 4. Configure
```bash
cd ~/comficurry/comficurrydocker
cp .env.prod.example .env
nano .env
```
Set **`PUBLIC_API_URL`** to the VM's public address + `:8080`, e.g.
`http://203.0.113.10:8080`. This is **baked into the apps at build time**, so it
must be the address the CEO's browser will reach (the VM's public IP or a
domain) — not `localhost`. Also set a real `POSTGRES_PASSWORD` and
`JWT_SECRET_KEY`.

## 5. Build & run
```bash
docker compose -f docker-compose.prod.yml up -d --build
```
First build takes a few minutes (Maven + two React builds). Check status:
```bash
docker compose -f docker-compose.prod.yml ps
docker compose -f docker-compose.prod.yml logs -f app-server   # watch the API boot
```
On boot the API auto-creates the schema and seeds delivery zones + order
statuses.

## 6. URLs (replace HOST with the VM IP/domain)
- **Customer app:** `http://HOST/` — opens in **guest mode**, order without LINE.
- **Admin app:** `http://HOST:8090/`
- **API:** `http://HOST:8080/`

## 7. Demo the end-to-end flow
1. **Admin** (`:8090`) → log in → add **Meal times / types**, **Options**
   (dishes), **Carbs**, and **Combo sets**; mark them **Active**. (Zones are
   already seeded.)
2. **Customer** (`/`) → welcome page → **Order Lunch/Dinner** → pick a combo,
   add options, choose an address (zone auto-fills) → pay by **QR Code** and
   **upload a slip**, or **COD**.
3. **Admin** → **Payment Review** → see the slip → **Verify** (or Reject).
   **Orders / Kitchen / Packing / Daily Report** show the order; **Subscribers**
   shows balances.

## Notes & caveats
- **First admin user:** if the `user` table is empty, create one via the admin
  **Register** screen (or `POST /auth/signup`).
- **Persistence:** Postgres and uploaded slips/photos are on named volumes
  (`db-data`, `uploads`), so they survive restarts. `docker compose ... down -v`
  wipes them.
- **Rotate secrets first:** the old LINE/Neon credentials are still in git
  history — rotate them before exposing anything you care about.
- **HTTPS / one domain (optional):** to serve everything under one HTTPS domain,
  put Caddy or Nginx in front (e.g. `app.` → customer, `admin.` → admin, `api.`
  → API) and set `PUBLIC_API_URL` to the `api.` URL. Not required for a quick
  IP-based demo.
- **Before a real launch:** remove guest login (`REACT_APP_GUEST`), set
  `ORDERING_ENFORCE_WINDOW=true`, wire real LINE/LIFF login, and complete
  Security Phase 2 (lock down the API).
