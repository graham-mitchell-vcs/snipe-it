# Snipe-IT on Docker (assets.cehq.io) — **Ops Quick-Sheet**

> **Stack recap**  
> * Ubuntu 22.04 host  
> * Custom image built from this repo’s root **Dockerfile**  
> * `docker-compose.yml` + override expose **port 80**  
> * Data volumes → `snipe-it_db_data`, `snipe-it_storage` (never erased unless you do it)  
> * Runtime config → `.env` (kept out of Git except non-secret defaults)

---

## 1  Routine code update (clean & reproducible)

```bash
# SSH into the server

# Pull latest code
cd ~/snipe-it
git pull

# Rebuild image & restart stack
docker compose up -d --build

# (Optional) tail logs
docker compose logs -f --tail=50

## 2 Emergency live patch

Only for urgent production fixes — the change lives only in this container until you capture it.

# SSH into the server
cd ~/snipe-it

# Enter the running app container
docker compose exec app bash

# Edit & clear cache
nano /var/www/html/path/to/file.php
php artisan optimize:clear
exit

## 3 Copy that patch back to Git (make it permanent)

# Inside ~/snipe-it on the host
CID=$(docker compose ps -q app)

# Copy modified file(s) from container → working tree
docker cp "$CID":/var/www/html/path/to/file.php ./path/to/file.php

# Commit & push
git add path/to/file.php
git commit -m "Hot-fix: <description>"
git push

# Bake fix into fresh image
docker compose up -d --build

## 4 Handy maintenance commands

| Purpose                                              | Command                                                      |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| Stop stack (keep volumes & images)                   | `docker compose down`                                        |
| Remove containers **and** images (keep DB + uploads) | `docker compose down --rmi all`                              |
| **Full wipe** (containers + images + volumes)        | `docker compose down -v --rmi all && docker system prune -a` |
| DB shell                                             | `docker compose exec db mariadb -u snipeit -p`               |
| Run Artisan                                          | `docker compose exec app php artisan <cmd>`                  |
| Check health / ports                                 | `docker compose ps`                                          |

## 5 Secrets

.env holds real passwords / keys — never commit it (already in .gitignore).

After editing .env, run docker compose up -d to apply changes.
