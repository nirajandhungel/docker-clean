# Safe Clean-weekly after majoe deployment 

```bash
docker container prune -f
docker builder prune -a -f
docker image prune -f
```



# 🧨 FULL VPS DOCKER WIPE (safe order)

Run these step by step:

## 1. Stop everything

```bash
docker stop $(docker ps -aq)
```

## 2. Remove all containers

```bash
docker rm -f $(docker ps -aq)
```

## 3. Remove all images

```bash
docker rmi -f $(docker images -q)
```

## 4. Remove all volumes (⚠️ DB + Redis data deleted)

```bash
docker volume rm $(docker volume ls -q)
```

If some fail, ignore or force clean:

```bash
docker volume prune -f
```

## 5. Remove all networks (except default)

```bash
docker network prune -f
```

## 6. Remove build cache

```bash
docker builder prune -a -f
```

## 7. Final system cleanup

```bash
docker system prune -a --volumes -f
```

## 🧼 OPTIONAL (extra clean logs)

```bash
sudo rm -rf /var/lib/docker/containers/*/*-json.log
```

## 🚀 VERIFY CLEAN STATE

After everything:

```bash
docker ps -a
```

Should show:

```
No containers
```

---

## 🚀 DEPLOY

```bash
git pull
pnpm vps:up
```

---

## ✅ 1. Prisma DB Sync (migrate / push)

```bash
docker compose --env-file .env -f infrastructure/docker-compose.vps.yml run --rm migrate \
  npx prisma db push --schema=packages/database/prisma/schema.prisma --accept-data-loss
```

## ✅ 2. Seed Admin User

```bash
docker compose --env-file .env -f infrastructure/docker-compose.vps.yml run --rm \
  admin-api node apps/api-admin/dist/scripts/seed-admin.js
```

## ✅ 3. Seed Config (IMPORTANT: needs volume mount)

```bash
docker compose --env-file .env -f infrastructure/docker-compose.vps.yml run --rm \
  -v $(pwd)/seed-config.local.json:/seed-config.local.json \
  admin-api node apps/api-admin/dist/scripts/seed-config.js
```
