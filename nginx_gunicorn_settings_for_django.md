# Nginx and Gunicorn Configuration Template for Django Projects

## 1. Gunicorn Setup

### **gunicorn.service**
Create the file `/etc/systemd/system/gunicorn.service` and add the following:

```ini
[Unit]
Description=Gunicorn service for Django project
After=network.target
Requires=gunicorn.socket

[Service]
User=www-data
Group=www-data
WorkingDirectory=/path/to/your/project
ExecStart=/path/to/your/project/.venv/bin/gunicorn \
    --access-logfile - \
    --workers 3 \
    --bind unix:/run/gunicorn.sock \
    project_name.wsgi:application
Restart=always

[Install]
WantedBy=multi-user.target
```

### Start and Enable Gunicorn
```bash
systemctl daemon-reload
systemctl start gunicorn.service
systemctl enable gunicorn.service
```

---

## 2. Gunicorn Socket Setup

### **gunicorn.socket**
Create the file `/etc/systemd/system/gunicorn.socket`:

```ini
[Unit]
Description=Gunicorn socket for Django project

[Socket]
ListenStream=/run/gunicorn.sock
SocketUser=www-data
SocketGroup=www-data
SocketMode=0660

[Install]
WantedBy=sockets.target
```

### Start and Enable the Socket
```bash
systemctl daemon-reload
systemctl start gunicorn.socket
systemctl enable gunicorn.socket
```

### Verify the Socket
```bash
ls -l /run/gunicorn.sock
```

---

## 3. Nginx Configuration

### **Nginx Site Configuration**
Create the file `/etc/nginx/sites-available/<project_name>`:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    location / {
        proxy_pass http://unix:/run/gunicorn.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    error_log /var/log/nginx/<project_name>_error.log;
    access_log /var/log/nginx/<project_name>_access.log;
}
```

### Enable the Configuration
1. Create a symbolic link:
   ```bash
   ln -s /etc/nginx/sites-available/<project_name> /etc/nginx/sites-enabled/
   ```

2. Test the configuration:
   ```bash
   nginx -t
   ```

3. Restart Nginx:
   ```bash
   systemctl restart nginx
   ```

---

## 4. Enable SSL with Let's Encrypt

1. Install Certbot:
   ```bash
   apt install certbot python3-certbot-nginx
   ```

2. Obtain and configure the certificate:
   ```bash
   certbot --nginx -d example.com -d www.example.com
   ```

3. Enable auto-renewal of certificates:
   ```bash
   systemctl enable certbot.timer
   ```

---

## 5. Testing the Setup

1. Check Gunicorn status:
   ```bash
   systemctl status gunicorn.service
   ```

2. Check Nginx status:
   ```bash
   systemctl status nginx
   ```

3. Open your website in a browser (HTTP/HTTPS).

---
