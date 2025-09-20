# Apache + Cloudflare Tunnel: Multi-Site Hosting Project

This project demonstrates how to host **multiple static websites** on a single Ubuntu server using **Apache VirtualHosts** and **Cloudflare Tunnel**.  
I used my domain **mytixe.me** (from Namecheap) and configured it to work securely over HTTPS with Cloudflare.

---

##  What I Learned
- Installing and configuring Apache on Ubuntu  
- Hosting static websites with HTML + CSS  
- Configuring **VirtualHosts** to host multiple sites (main + blog) on one server  
- Connecting a custom domain from **Namecheap → Cloudflare → Apache**  
- Enabling **HTTPS automatically with Cloudflare Tunnel**  
- Creating a **custom 404 error page**  

---

##  Project Structure
```
apache-multisite-cloudflare-tunnel/
├── README.md
├── apache/
│   ├── mywebsite.conf
│   ├── blog.conf
├── cloudflared/
│   └── config.yml
├── sites/
│   ├── mytixe/
│   │   └── index.html
│   └── blog/
│       └── index.html
└── screenshots/   (optional: add screenshots here)
```

---

##  Live Demo
- [https://mytixe.me](https://mytixe.me) → Main site  
- [https://blog.mytixe.me](https://blog.mytixe.me) → Blog site  

---

##  Setup Steps

### 1. Install Apache
```bash
sudo apt update
sudo apt install apache2 -y
```

### 2. Create Website Folders
```bash
sudo mkdir -p /var/www/mytixe
sudo mkdir -p /var/www/blog

echo "<h1>Welcome to Mytixe 🚀</h1>" | sudo tee /var/www/mytixe/index.html
echo "<h1>Welcome to the Blog 📝</h1>" | sudo tee /var/www/blog/index.html
```

### 3. Configure Apache VirtualHosts

#### File: `/etc/apache2/sites-available/mywebsite.conf`
```apache
<VirtualHost *:80>
    ServerName mytixe.me
    ServerAlias www.mytixe.me
    DocumentRoot /var/www/mytixe

    <Directory /var/www/mytixe>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/mytixe_error.log
    CustomLog ${APACHE_LOG_DIR}/mytixe_access.log combined
</VirtualHost>
```

#### File: `/etc/apache2/sites-available/blog.conf`
```apache
<VirtualHost *:80>
    ServerName blog.mytixe.me
    DocumentRoot /var/www/blog

    <Directory /var/www/blog>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/blog_error.log
    CustomLog ${APACHE_LOG_DIR}/blog_access.log combined
</VirtualHost>
```

Enable the sites:
```bash
sudo a2ensite mywebsite.conf
sudo a2ensite blog.conf
sudo systemctl reload apache2
```

---

### 4. Install Cloudflare Tunnel
```bash
curl -fsSL https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb -o cloudflared.deb
sudo dpkg -i cloudflared.deb
cloudflared --version
```

Authenticate with Cloudflare:
```bash
cloudflared tunnel login
```

Create tunnel:
```bash
cloudflared tunnel create mytixe-tunnel
```

---

### 5. Configure Tunnel

#### File: `/etc/cloudflared/config.yml`
```yaml
tunnel: <TUNNEL-ID>
credentials-file: /etc/cloudflared/<TUNNEL-ID>.json

ingress:
  - hostname: mytixe.me
    service: http://localhost:80
  - hostname: www.mytixe.me
    service: http://localhost:80
  - hostname: blog.mytixe.me
    service: http://localhost:80
  - service: http_status:404
```

Route DNS:
```bash
cloudflared tunnel route dns mytixe-tunnel mytixe.me
cloudflared tunnel route dns mytixe-tunnel www.mytixe.me
cloudflared tunnel route dns mytixe-tunnel blog.mytixe.me
```

Start tunnel as a service:
```bash
sudo cloudflared service install
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

---

### 6. Test
- Open [https://mytixe.me](https://mytixe.me) → shows main site  
- Open [https://blog.mytixe.me](https://blog.mytixe.me) → shows blog site  

---
