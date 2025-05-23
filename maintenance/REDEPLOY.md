### Clean the old deployment

```
cd ~/snipe-it                # or wherever the previous project lives

# 1-a. Stop stack, delete its containers & custom images
docker compose down --rmi all     # keeps named volumes by default

# 1-b. (Optional) nuke data volumes too – **irreversible**
# docker volume rm snipe-it_db_data snipe-it_storage
#
# 1-c. (Optional) purge dangling images/networks/containers
docker system prune -f            # leaves running containers alone
```

At this point Docker is empty except for the two data volumes (if you kept them).

### 1 Refresh or clone the repo

```
cd ~
rm -rf snipe-it                 # toss stale working copy
git clone https://github.com/graham-mitchell-vcs/snipe-it.git
cd snipe-it                     # repo root - contains the big Dockerfile
```

### 2 Create the runtime .env

```
cp .env.docker .env
nano .env
```

Modify / add only these lines:
```
MYSQL_ROOT_PASSWORD=changeme
MYSQL_DATABASE=snipeit
MYSQL_USER=snipeit
MYSQL_PASSWORD=changeme

APP_URL=http://assets.cehq.io
APP_TIMEZONE=Australia/Sydney

PHP_UPLOAD_LIMIT=8
```

Generate a fresh Laravel key once:

```
docker compose run --rm app php artisan key:generate --show
```

### 4 Compose files

Create (or update) docker-compose.override.yml in the same directory:

```
services:
  app:
    build: .                  # uses the repo’s Dockerfile
    image: assets-snipeit:latest
    ports:
      - "80:80"               # host 80 → container 80
```

### 5 Build and launch

```
docker compose up -d --build     # first run builds image; later runs reuse cache
docker compose logs -f           # wait for “Startup finished”

Open http://assets.cehq.io/ – you should see the Snipe-IT web installer (or your existing data if you preserved the volumes).
```
