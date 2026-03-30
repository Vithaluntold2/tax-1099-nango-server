# TaxBlitz Nango — Self-Hosted on Railway

Self-hosted Nango server running at `https://taxblitz-nango-production.up.railway.app`.

---

## One-Time Setup

### 1. Create the Railway project

```bash
railway init taxblitz-nango --region sjc
```

### 2. Create a Railway PostgreSQL database

```bash
railway add --plugin postgresql --name taxblitz-nango-db --region sjc --initial-cluster-size 1 --vm-size shared-cpu-1x --volume-size 10
# Attach it — Railway injects DATABASE_URL automatically:
railway connect postgresql taxblitz-nango-db -a taxblitz-nango
# Then rename to NANGO_DB_URL:
railway variables set NANGO_DB_URL="$(railway connect postgresql -a taxblitz-nango-db --print-url)" -a taxblitz-nango
```

### 3. Create an Upstash Redis instance

```bash
railway add redis --name taxblitz-nango-redis --region sjc
# Note the connection URL shown, then:
railway variables set NANGO_REDIS_URL="<redis_url>" -a taxblitz-nango
```

### 4. Generate and set secrets

```bash
NANGO_SECRET=$(openssl rand -hex 32)
NANGO_PUBLIC=$(uuidgen | tr '[:upper:]' '[:lower:]')
ENCRYPT=$(openssl rand -hex 16)

railway variables set \
  NANGO_SECRET_KEY="$NANGO_SECRET" \
  NANGO_PUBLIC_KEY="$NANGO_PUBLIC" \
  ENCRYPT_KEY="$ENCRYPT" \
  -a taxblitz-nango
```

### 5. Deploy

```bash
cd nango-server
railway up --detach --remote-only
```

### 6. Verify

```bash
curl https://taxblitz-nango-production.up.railway.app/healthcheck
```

---

## Configure TaxBlitz backend & frontend

Set these two env vars on your **backend** Railway service:

```bash
railway variables set \
  NANGO_BASE_URL="https://taxblitz-nango-production.up.railway.app" \
  NANGO_SECRET_KEY="$NANGO_SECRET" \
  NANGO_PUBLIC_KEY="$NANGO_PUBLIC" \
  -a taxblitz-backend
```

The frontend reads `host` from the `/api/v1/nango/config` endpoint automatically.

---

## Deploy sync scripts after setup

```bash
cd nango-integrations
NANGO_HOSTPORT=https://taxblitz-nango-production.up.railway.app \
NANGO_SECRET_KEY=<your_secret_key> \
npx nango deploy
```

---

## Redeploy (updates)

```bash
cd nango-server && railway up --detach --remote-only
```
