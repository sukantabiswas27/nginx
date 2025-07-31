
# 🌐 NGINX Setup for Auriva Project (Thinksurfmedia)

This document outlines the NGINX configuration used in the **Auriva** project, including static file serving, reverse proxy, and SSL via Certbot.

---

## 📁 Static Frontend Hosting

### 🔹 Serving Angular/React/Static Files

```nginx
server {
    listen 80;
    root /var/www/auriva/dist;

    server_name auriva.thinksurfmedia.iny;

    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 🔹 Serve index.html for all routes (SPA behavior)

```nginx
server {
    listen 80;
    root /var/www/auriva/dist;

    server_name auriva.thinksurfmedia.iny;

    index index.html index.htm;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

---

## 🔁 Reverse Proxy Setup (Node.js Backend)

### 🔹 Proxy to Localhost:5000 via IP Address

```nginx
server {
    listen 80;

    server_name 98.70.32.60;

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

## 🔐 SSL with Certbot + Reverse Proxy

### 🔹 Backend Upstream Setup

```nginx
upstream backend {
    server localhost:5000;
}
```

### 🔹 HTTPS with Certbot

```nginx
server {
    server_name dev.thinksurfmedia.in;

    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/dev.thinksurfmedia.in/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/dev.thinksurfmedia.in/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}
```

### 🔹 Redirect HTTP to HTTPS

```nginx
server {
    if ($host = dev.thinksurfmedia.in) {
        return 301 https://$host$request_uri;
    }

    listen 80;
    server_name dev.thinksurfmedia.in;
    return 404;
}
```

---

## 🔧 Certbot Installation & SSL Setup

📌 Follow official instructions: [Certbot for NGINX on Ubuntu (Snap)](https://certbot.eff.org/instructions?ws=nginx&os=snap&tab=standard)

### ✅ Steps

```bash
# 1. Install snapd
sudo apt update
sudo apt install snapd

# 2. Install Certbot
sudo snap install --classic certbot

# 3. Link Certbot binary
sudo ln -s /snap/bin/certbot /usr/bin/certbot

# 4. Run Certbot for NGINX
sudo certbot --nginx
```

---

## ✅ Status

- [x] Static Frontend on `/var/www/auriva/dist`
- [x] Reverse proxy to backend (Node.js)
- [x] SSL via Certbot for dev domain
- [x] Automatic HTTPS redirection

---

> 🛠 Maintained by Thinksurfmedia DevOps Team
