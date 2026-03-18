# Node.js / MERN Application Deployment Guide on VPS (Ubuntu)

This guide provides step-by-step instructions to deploy Node.js or MERN applications on a VPS, including **performance optimization**, **PM2 process management**, **NGINX reverse proxy**, and **SSL setup**. It also includes notes for deploying a **Next.js frontend without a custom server**.

## 1. Connect to VPS
```bash
ssh root@your_vps_ip
type your password
```

## 2. Update & Upgrade Packages
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git ufw
```

## 3. Install Node.js (Latest LTS)
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
\. "$HOME/.nvm/nvm.sh"
nvm install 24
node -v
npm -v
```

## 4. Install PM2
```bash
npm install pm2 -g
```

## 5. Setup Firewall (UFW)
```bash
sudo ufw allow OpenSSH
sudo ufw allow http
sudo ufw allow https
sudo ufw enable
```

## 6. Clone Your Application
```bash
mkdir /var/www
cd /var/www
git clone https://github.com/yourusername/your-node-app.git
cd your-node-app
```

## 7. Setup Environment Variables
Create a .env file in your project root:
```bash
nano .env
```
Add database URLs, API keys, secrets, and other environment variables.

## 8. Install and build your application
```bash
npm install
npm run build
```

## 9. Start Application with PM2
Node.js (Backend / Frontend (Node based)
```bash
pm2 start npm --name myapp -- start
pm2 save
pm2 startup
```

Default port: 3000 <br />
Optional clustering for high traffic:
```bash
pm2 start npm --name "frontend" -- start -- -p 3000
```

## 10. Setup NGINX Reverse Proxy
```bash
sudo apt install nginx -y
sudo nano /etc/nginx/sites-available/myapp
```
Example configuration:
```bash
server {
    listen 80;
    server_name your-domain.com;
   
    client_max_body_size 50M;

    location / {
        proxy_pass http://localhost:applicationPort;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
Enable the site and reload NGINX:
```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

## 11. Add SSL with Let’s Encrypt
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
sudo systemctl enable certbot.timer
```

## 12. Performance Optimization
PM2 Cluster Mode
```bash
pm2 start npm --name myApp -- start -i max
```
Enable Gzip & Static Caching (configured in NGINX above)
Node.js Compression Middleware
```bash
const compression = require("compression");
app.use(compression());
```
Optimize database queries, indexes, and consider Redis caching for heavy reads.

## 13. Monitoring
View Logs:
```bash
pm2 logs myapp
```
Monitor app and resource usage:
```bash
pm2 monit
```

## 14. Optional CI/CD Deployment
Automate deployment with GitHub Actions or GitLab CI/CD:
```bash
git pull origin main
npm install
npm run build
pm2 restart myapp
```

## 15. Final Checklist
```bash
Node.js installed & app running under PM2

NGINX reverse proxy configured

SSL enabled with Let’s Encrypt

Firewall enabled

Logs & monitoring active

Performance optimizations applied (gzip, caching, clustering)
```

Notes:

```bash
This guide works for MERN stack apps, backend APIs (Express/Node.js), and frontend apps (Next.js without custom server).

For Next.js frontend, using npm start is sufficient if no custom server is needed.

For high traffic apps, consider PM2 cluster mode and NGINX caching.
```
