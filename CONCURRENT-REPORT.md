# 🔍 Incident Report — Northstone Holding VPS Deployment

**Date:** 21 May 2026  
**Project:** northstoneholding.com  
**Server:** VPS @ `187.77.153.122`  
**Reporter:** Antigravity AI  

---

## 📋 Interview Summary

> *What went wrong, how was it diagnosed, how was it fixed, and what commands were used — step by step.*

---

## ❓ Q1: What was the problem?

When we tried to deploy the Northstone Holding static website to the VPS and start Nginx, **Nginx crashed immediately on startup** with the following error:

```
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
nginx: [emerg] still could not bind()
Job for nginx.service failed because the control process exited with error code.
```

Nginx could not start because **ports 80 and 443 were already occupied** by another process on the server.

---

## ❓ Q2: How was it diagnosed?

We ran a series of diagnostic commands to understand what was using the ports.

### Step 1 — Check which processes are listening on ports 80 and 443:
```bash
ss -tulpn | grep -E '80|443'
```

**Output revealed:**
```
tcp  LISTEN  0  4096  0.0.0.0:443  users:(("docker-proxy",...))
tcp  LISTEN  0  4096  0.0.0.0:80   users:(("docker-proxy",...))
```

The culprit was `docker-proxy` — a Docker container was already bound to both ports.

### Step 2 — Identify the running Docker container:
```bash
docker ps
```

**Output:**
```
CONTAINER ID   IMAGE            COMMAND              CREATED   STATUS    PORTS
c1f1e5e4c621   caddy:2-alpine   "caddy run ..."      ...       Up        0.0.0.0:80->80, 0.0.0.0:443->443
0d8a669a7c55   kira-ai-os-web   "docker-entrypoint"  ...       Up        3000/tcp
```

**Root cause confirmed:** The `kira-ai-os` project was running a Dockerized **Caddy** reverse proxy on ports 80 and 443. Nginx cannot also bind to these ports on the same host — it's a hard conflict.

### Step 3 — Check Nginx status to confirm the failure:
```bash
systemctl status nginx
```

**Output confirmed:** Nginx failed to start due to the port conflict.

### Step 4 — Check which ports are free:
```bash
ss -tulpn | grep 8080
```

**Output:** Empty — port `8080` was completely free and available.

---

## ❓ Q3: What was the fix?

We implemented a **two-layer architecture** that eliminates the conflict:

```
Internet (HTTP/HTTPS)
        │
        ▼
  Caddy Container          ← ports 80 / 443 (Docker, already running)
  (kira-ai-os project)
        │
        ├──── kiraos.io ──────────────► Next.js app (container port 3000)
        │
        └──── northstoneholding.com ──► Nginx on HOST port 8080
                                              │
                                              ▼
                                    /var/www/northstone/
                                    (static HTML + assets)
```

### Fix Part 1 — Configure Nginx on port 8080 (not 80)

We created a new Nginx server block that listens only on port `8080`, avoiding the conflict entirely:

```nginx
server {
    listen 8080;
    listen [::]:8080;
    server_name northstoneholding.com www.northstoneholding.com;

    root /var/www/northstone;
    index index.html;

    location / {
        try_files $uri $uri/ /404-error.html;
    }

    error_page 404 /404-error.html;
    location = /404-error.html { internal; }

    location ~* \.(?:css|js|jpg|jpeg|gif|png|ico|svg|woff|woff2|ttf)$ {
        expires 1M;
        access_log off;
        add_header Cache-Control "public";
    }

    location ~ /\. { deny all; }
}
```

### Fix Part 2 — Add northstoneholding.com to the Caddy reverse proxy

We updated the `Caddyfile` in `kira-ai-os` to forward `northstoneholding.com` traffic through Caddy (on port 443 with auto SSL) down to Nginx on port 8080:

```caddy
kiraos.io, www.kiraos.io {
    reverse_proxy web:3000
}

northstoneholding.com, www.northstoneholding.com {
    reverse_proxy 187.77.153.122:8080
}
```

Caddy automatically obtained **Let's Encrypt SSL certificates** for all four domains.

---

## ❓ Q4: What commands were used — in exact order?

### 🔎 Diagnosis
```bash
# 1. Check what's on ports 80/443
sshpass -p '...' ssh root@187.77.153.122 "ss -tulpn | grep -E '80|443'"

# 2. List all running Docker containers
sshpass -p '...' ssh root@187.77.153.122 "docker ps"

# 3. Read the Caddyfile to understand routing
sshpass -p '...' ssh root@187.77.153.122 "cat /var/www/kira-ai-os/Caddyfile"

# 4. Check Nginx service status
sshpass -p '...' ssh root@187.77.153.122 "systemctl status nginx"

# 5. Check if port 8080 is free
sshpass -p '...' ssh root@187.77.153.122 "ss -tulpn | grep 8080"
```

### 📁 Local Repository Restructuring
```bash
# Move all static files into a clean public/ directory
mkdir -p public
mv assets public/
mv *.html public/
mkdir -p public/assets/images
mv northstone.png public/assets/images/
```

### 🚀 Deploy Static Files to VPS
```bash
# Sync local public/ folder to /var/www/northstone/ on VPS
sshpass -p '...' rsync -avz --delete --exclude='.DS_Store' \
  -e "ssh -o StrictHostKeyChecking=no" \
  ./public/ root@187.77.153.122:/var/www/northstone/
```

### 🖥️ VPS — Configure Nginx on Port 8080
```bash
# Write Nginx config remotely via SSH heredoc
sshpass -p '...' ssh root@187.77.153.122 "cat << 'EOF' > /etc/nginx/sites-available/northstoneholding.com
server { listen 8080; ... }
EOF"

# Enable the site
ln -sf /etc/nginx/sites-available/northstoneholding.com /etc/nginx/sites-enabled/

# Remove the default site (was trying to bind to port 80 — conflict)
rm -f /etc/nginx/sites-enabled/default

# Test config syntax
nginx -t

# Restart Nginx
systemctl restart nginx
```

### 🔒 VPS — Update Caddy to Route northstoneholding.com
```bash
# Write updated Caddyfile with northstone block
sshpass -p '...' ssh root@187.77.153.122 "cat << 'EOF' > /var/www/kira-ai-os/Caddyfile
kiraos.io, www.kiraos.io {
    reverse_proxy web:3000
}

northstoneholding.com, www.northstoneholding.com {
    reverse_proxy 187.77.153.122:8080
}
EOF"

# Restart the Caddy Docker container to reload config
cd /var/www/kira-ai-os
docker compose up -d --force-recreate caddy
```

### ✅ Verification
```bash
# Check Nginx is listening on port 8080
ss -tulpn | grep 8080

# Confirm local response from Nginx
curl -I http://localhost:8080

# Confirm public HTTPS is live for both domains
curl -I https://northstoneholding.com
curl -I https://kiraos.io
```

---

## ✅ Results

| Check | Result |
|---|---|
| Nginx listening on port 8080 | ✅ Confirmed |
| `http://localhost:8080` on VPS | `HTTP/1.1 200 OK` |
| `https://northstoneholding.com` | `HTTP/2 200` via Caddy → Nginx |
| `https://kiraos.io` | `HTTP/2 200` via Caddy → Next.js |
| Let's Encrypt SSL for northstone | ✅ Issued automatically by Caddy |
| Port conflict with Docker | ✅ Resolved — Nginx runs on 8080 only |

---

## 📁 Files Changed

| File | Change |
|---|---|
| `public/` | **New directory** — all HTML + assets moved here |
| `deploy.sh` | Updated rsync source from `./` to `./public/` |
| `domain.md` | Rewritten — documents Nginx 8080 + Caddy proxy architecture |
| `REPORT.md` | **This file** — full incident report |
| `kira-ai-os/Caddyfile` | Added `northstoneholding.com` reverse proxy block |
