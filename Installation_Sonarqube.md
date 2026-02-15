Step-by-step installation guide for SonarQube on Rocky Linux.

# SonarQube Installation Guide on Rocky Linux 9.7

This repository provides a complete production-ready setup guide for installing **SonarQube Community Edition** on **Rocky Linux 9.7**.

Includes:

- Standard installation (Java + PostgreSQL)
- System tuning
- SELinux configuration
- Nginx reverse proxy with SSL (Let's Encrypt)
- Automated installation script
- Docker-based installation
- CI/CD Integration (GitHub Actions example)

---

# 📋 System Requirements

- Rocky Linux 9.7
- 4 GB RAM (Recommended)
- 2 vCPU
- 20 GB disk space
- Root or sudo privileges

---

# 🚀 OPTION 1 — STANDARD INSTALLATION (Recommended for Production)

---

## Step 1: Update System

```bash
sudo dnf update -y
sudo reboot
```

---

## Step 2: Install Java 17

```bash
sudo dnf install java-17-openjdk java-17-openjdk-devel -y
java -version
```

---

## Step 3: Install PostgreSQL

```bash
sudo dnf install postgresql-server postgresql-contrib -y
sudo postgresql-setup --initdb
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

Create database:

```bash
sudo -u postgres psql
```

```sql
CREATE DATABASE sonarqube;
CREATE USER sonar WITH ENCRYPTED PASSWORD 'StrongPassword';
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonar;
\q
```

---

## Step 4: System Tuning

### Kernel Parameters

```bash
echo "vm.max_map_count=524288" | sudo tee -a /etc/sysctl.conf
echo "fs.file-max=131072" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Limits

```bash
sudo nano /etc/security/limits.conf
```

Add:

```
sonar   -   nofile   131072
sonar   -   nproc    8192
```

---

## Step 5: Create Sonar User

```bash
sudo useradd -r -m -U -d /opt/sonarqube -s /bin/bash sonar
```

---

## Step 6: Install SonarQube

```bash
cd /opt
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.5.1.90531.zip
sudo unzip sonarqube-10.5.1.90531.zip
sudo mv sonarqube-10.5.1.90531 sonarqube
sudo chown -R sonar:sonar /opt/sonarqube
```

---

## Step 7: Configure Database

Edit:

```bash
sudo nano /opt/sonarqube/conf/sonar.properties
```

Uncomment:

```
sonar.jdbc.username=sonar
sonar.jdbc.password=StrongPassword
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
```

---

## Step 8: Create Systemd Service

```bash
sudo nano /etc/systemd/system/sonarqube.service
```

Paste:

```
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonar
Group=sonar
Restart=always
LimitNOFILE=131072
LimitNPROC=8192

[Install]
WantedBy=multi-user.target
```

Start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable sonarqube
sudo systemctl start sonarqube
```

---

# 🔥 Configure Firewall

```bash
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload
```

Access:

```
http://SERVER_IP:9000
```

---

# 🔐 SELINUX CONFIGURATION (Rocky Linux Specific)

If SELinux is enforcing:

```bash
sudo setsebool -P httpd_can_network_connect 1
sudo semanage port -a -t http_port_t -p tcp 9000
```

If semanage not installed:

```bash
sudo dnf install policycoreutils-python-utils -y
```

---

# 🌍 PRODUCTION SETUP — NGINX + SSL (HTTPS)

## Install Nginx

```bash
sudo dnf install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

## Configure Reverse Proxy

```bash
sudo nano /etc/nginx/conf.d/sonarqube.conf
```

```
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

---

## Install Let's Encrypt SSL

```bash
sudo dnf install certbot python3-certbot-nginx -y
sudo certbot --nginx -d yourdomain.com
```

Auto-renew:

```bash
sudo certbot renew --dry-run
```

---

# 🐳 OPTION 2 — DOCKER INSTALLATION

Install Docker:

```bash
sudo dnf install docker -y
sudo systemctl enable docker
sudo systemctl start docker
```

Run SonarQube container:

```bash
docker run -d --name sonarqube \
-p 9000:9000 \
-e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
sonarqube:community
```

Access:

```
http://SERVER_IP:9000
```

---

# 🤖 CI/CD INTEGRATION — GitHub Actions Example

Create:

```
.github/workflows/sonarqube.yml
```

```yaml
name: SonarQube Scan

on:
  push:
    branches:
      - main

jobs:
  sonarqube:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: SonarQube Scan
        uses: sonarsource/sonarqube-scan-action@v2
        with:
          args: >
            -Dsonar.projectKey=your_project
            -Dsonar.host.url=http://yourdomain.com
            -Dsonar.login=${{ secrets.SONAR_TOKEN }}
```

Generate token inside SonarQube:

```
My Account → Security → Generate Token
```

Add it to GitHub Secrets:

```
Settings → Secrets → Actions → New Repository Secret
```

---

# 🛑 Service Management

```bash
sudo systemctl start sonarqube
sudo systemctl stop sonarqube
sudo systemctl restart sonarqube
sudo systemctl status sonarqube
```

---

# 📊 Logs

```bash
sudo tail -f /opt/sonarqube/logs/sonar.log
```

---

# ✅ Installation Complete

You now have:

- Production-ready SonarQube
- HTTPS enabled
- Firewall configured
- SELinux configured
- Docker option available
- CI/CD integration ready

---

⭐ If this repository helped you, please consider starring it!
