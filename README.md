# TaxBlitz Nango — Self-Hosted on Fly.io

Self-hosted Nango server running at `https://taxblitz-nango.fly.dev`.

---

## One-Time Setup

### 1. Create the Fly app
```bash
fly apps create taxblitz-nango --region sjc
```

### 2. Create a Fly Postgres cluster
```bash
fly postgres create --name taxblitz-nango-db --region sjc --initial-cluster-size 1 --vm-size shared-cpu-1x --volume-size 10
# Attach it — Fly injects DATABASE_URL automatically:
fly postgres attach taxblitz-nango-db -a taxblitz-nango
# Then rename to NANGO_DB_URL:
fly secrets set NANGO_DB_URL="$(fly postgres connect -a taxblitz-nango-db --print-url)" -a taxblitz-nango
```

### 3. Create an Upstash Redis instance
```bash
fly ext redis create --name taxblitz-nango-redis --region sjc
# Note the connection URL shown, then:
fly secrets set NANGO_REDIS_URL="<redis_url>" -a taxblitz-nango
```

### 4. Generate and set secrets
```bash
NANGO_SECRET=$(openssl rand -hex 32)
NANGO_PUBLIC=$(uuidgen | tr '[:upper:]' '[:lower:]')
ENCRYPT=$(openssl rand -hex 16)

fly secrets set \
  NANGO_SECRET_KEY="$NANGO_SECRET" \
  NANGO_PUBLIC_KEY="$NANGO_PUBLIC" \
  ENCRYPT_KEY="$ENCRYPT" \
  -a taxblitz-nango
```

### 5. Deploy
```bash
cd nango-server
fly deploy --remote-only
```

### 6. Verify
```bash
curl https://taxblitz-nango.fly.dev/healthcheck
```

---

## Configure TaxBlitz backend & frontend

Set these two env vars on your **backend** Fly app:
```bash
fly secrets set \
  NANGO_BASE_URL="https://taxblitz-nango.fly.dev" \
  NANGO_SECRET_KEY="$NANGO_SECRET" \
  NANGO_PUBLIC_KEY="$NANGO_PUBLIC" \
  -a taxblitz-backend
```

The frontend reads `host` from the `/api/v1/nango/config` endpoint automatically.

---

## Deploy sync scripts after setup

```bash
cd nango-integrations
NANGO_HOSTPORT=https://taxblitz-nango.fly.dev \
NANGO_SECRET_KEY=<your_secret_key> \
npx nango deploy
```

---

## Redeploy (updates)
```bash
cd nango-server && fly deploy --remote-only
```
