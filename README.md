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

1️⃣ Seed the Admin User
Run this command. You can change the password to whatever you want:

bash
ADMIN_SEED_PASSWORD="YourSecurePassword123" \
docker compose -f infrastructure/docker-compose.vps.yml run --rm admin-api \
node apps/api-admin/dist/scripts/seed-admin.js
2️⃣ Seed the App Config
This initializes your application settings:

bash
docker compose -f infrastructure/docker-compose.vps.yml run --rm admin-api \
node apps/api-admin/dist/scripts/seed-config.js
3️⃣ Verify the Database (Optional)
If you want to see the new admin user in the database:

bash
docker exec -it futsmandu-db psql -U futsmandu -d futsmandu -c "SELECT email, role, is_active FROM admins;"
