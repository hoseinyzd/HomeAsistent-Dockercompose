# Home Assistant with Nginx Reverse Proxy

این پروژه شامل راه‌اندازی Home Assistant با استفاده از Docker Compose و Nginx به عنوان reverse proxy است.

## ساختار پروژه

```
homeasistant/
├── homeasistent/
│   └── docker-compose.yml      # فایل Docker Compose برای Home Assistant
├── nginx/
│   ├── docker-compose.yml      # فایل Docker Compose برای Nginx
│   └── nginx.conf              # تنظیمات Nginx
└── README.md                   # این فایل
```

## ویژگی‌ها

- ✅ Home Assistant بدون expose کردن پورت (فقط از طریق شبکه Docker)
- ✅ Nginx reverse proxy روی پورت 3290
- ✅ اتصال بین کانتینرها از طریق نام کانتینر
- ✅ پورت 80 هاست آزاد برای استفاده‌های دیگر
- ✅ پشتیبانی از WebSocket برای Home Assistant

## پیش‌نیازها

- Docker
- Docker Compose

## راه‌اندازی

### 1. راه‌اندازی Home Assistant

```bash
cd homeasistent
docker compose up -d
```

این دستور:
- شبکه `homeasistant-network` را ایجاد می‌کند
- کانتینر Home Assistant را راه‌اندازی می‌کند
- Volume برای ذخیره تنظیمات ایجاد می‌کند

### 2. تنظیمات Reverse Proxy در Home Assistant

بعد از راه‌اندازی Home Assistant، باید تنظیمات reverse proxy را فعال کنید:

```bash
docker exec homeasistant sh -c 'cat >> /config/configuration.yaml << EOF

# Reverse proxy configuration
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 172.18.0.0/16
EOF
'
```

سپس Home Assistant را restart کنید:

```bash
docker restart homeasistant
```

**نکته:** اگر IP کانتینر nginx متفاوت است، می‌توانید IP آن را با دستور زیر پیدا کنید:

```bash
docker inspect nginx | grep IPAddress
```

و آن را به `trusted_proxies` اضافه کنید.

### 3. راه‌اندازی Nginx

```bash
cd nginx
docker compose up -d
```

## دسترسی

بعد از راه‌اندازی، می‌توانید از طریق آدرس زیر به Home Assistant دسترسی داشته باشید:

```
http://YOUR_SERVER_IP:3290
```

## دستورات مفید

### بررسی وضعیت کانتینرها

```bash
docker ps | grep -E "homeasistant|nginx"
```

### بررسی لاگ‌ها

```bash
# لاگ Home Assistant
docker logs homeasistant --tail 50

# لاگ Nginx
docker logs nginx --tail 50
```

### Restart کانتینرها

```bash
docker restart homeasistant
docker restart nginx
```

### بررسی شبکه

```bash
docker network inspect homeasistant-network
```

### بررسی اتصال بین کانتینرها

```bash
docker exec nginx curl -I http://homeasistant:8123
```

## تنظیمات

### تغییر پورت Nginx

برای تغییر پورت nginx، فایل `nginx/docker-compose.yml` را ویرایش کنید:

```yaml
ports:
  - "YOUR_PORT:3290"  # پورت هاست:پورت کانتینر
```

و فایل `nginx/nginx.conf` را هم ویرایش کنید:

```nginx
server {
    listen 3290;  # پورت داخل کانتینر
    ...
}
```

### تغییر تنظیمات Nginx

فایل `nginx/nginx.conf` را ویرایش کنید و سپس nginx را restart کنید:

```bash
docker restart nginx
```

## عیب‌یابی

### خطای 400 Bad Request

اگر خطای 400 دریافت می‌کنید، بررسی کنید که:

1. تنظیمات reverse proxy در Home Assistant فعال باشد
2. IP کانتینر nginx در `trusted_proxies` باشد
3. Home Assistant restart شده باشد

### بررسی لاگ‌ها برای خطا

```bash
docker logs homeasistant | grep -i "reverse proxy\|forwarded"
docker logs nginx | grep -i error
```

## ساختار شبکه

```
┌─────────────────┐
│   Host (3290)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Nginx Container│
│  (Port 3290)    │
└────────┬────────┘
         │
         │ homeasistant-network
         │
         ▼
┌─────────────────┐
│ Home Assistant  │
│  (Port 8123)    │
│  (No exposed)   │
└─────────────────┘
```

## نکات مهم

- پورت 80 هاست درگیر نمی‌شود و می‌توانید از آن برای سرویس‌های دیگر استفاده کنید
- اتصال بین کانتینرها از طریق نام کانتینر (`homeasistant`) انجام می‌شود
- تمام تنظیمات Home Assistant در volume `homeasistant-config` ذخیره می‌شود
- برای backup، می‌توانید volume را backup کنید

## لایسنس

این پروژه برای استفاده شخصی است.

