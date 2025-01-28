# Шаблон настройки Nginx и Gunicorn для Django-проектов

## 1. Настройка Gunicorn

Создаём и настраиваем файл `gunicorn.service` для работы через systemd:

### **gunicorn-def.service**

```ini
[Unit]
Description=Gunicorn service for Django project
After=network.target
Requires=gunicorn-def.socket

[Service]
User=root
Group=root
WorkingDirectory=/path/to/your/project
ExecStart=/path/to/your/project/.venv/bin/gunicorn \
    --access-logfile - \
    --workers 3 \
    --bind unix:/run/gunicorn-def.sock \
    your_project_name.wsgi:application
Restart=always

[Install]
WantedBy=multi-user.target
```

- **User и Group**: Указывают пользователя и группу, под которыми будет работать Gunicorn.
- **WorkingDirectory**: Путь к корню вашего Django-проекта.
- **ExecStart**: Полный путь к Gunicorn внутри виртуального окружения с параметрами запуска.
- **--bind unix:/run/gunicorn-def.sock**: Используем Unix-сокет для взаимодействия с Nginx.

После сохранения:

```bash
systemctl daemon-reload
systemctl start gunicorn-def.service
systemctl enable gunicorn-def.service
```

---

## 2. Настройка сокета Gunicorn

Создаём файл `gunicorn-def.socket` для взаимодействия через сокет:

### **gunicorn-def.socket**

```ini
[Unit]
Description=Gunicorn socket for Django project

[Socket]
ListenStream=/run/gunicorn-def.sock
SocketUser=www-data
SocketGroup=www-data
SocketMode=0660

[Install]
WantedBy=sockets.target
```

После сохранения:

```bash
systemctl daemon-reload
systemctl start gunicorn-def.socket
systemctl enable gunicorn-def.socket
```

Проверить, создаётся ли сокет:

```bash
ls -l /run/gunicorn-def.sock
```

---

## 3. Настройка Nginx

Создаём конфигурационный файл для Nginx в `/etc/nginx/sites-available/<your_project>`:

### **nginx.conf для сайта**

```nginx
server {
    listen 80;
    server_name your_domain_or_ip;

    location / {
        proxy_pass http://unix:/run/gunicorn-def.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    error_log /var/log/nginx/your_project_error.log;
    access_log /var/log/nginx/your_project_access.log;
}
```

1. **proxy_pass**: Указывает, что запросы будут перенаправляться через сокет Gunicorn.
2. **proxy_set_header**: Передаёт важные заголовки от клиента для корректной работы.
3. **server_name**: Укажите ваш домен или IP-адрес.

---

## 4. Подключение конфигурации Nginx

Создаём символическую ссылку, чтобы Nginx видел конфигурацию:

```bash
ln -s /etc/nginx/sites-available/<your_project> /etc/nginx/sites-enabled/
```

Проверяем конфигурацию Nginx:

```bash
nginx -t
```

Перезапускаем Nginx:

```bash
systemctl restart nginx
```

---

## 5. Проверка работы

1. Убедитесь, что Gunicorn работает:
   ```bash
   systemctl status gunicorn-def.service
   ```

2. Убедитесь, что Nginx работает и обслуживает сайт:
   ```bash
   systemctl status nginx
   ```

3. Откройте браузер и зайдите по вашему домену или IP.

---

## 6. Дополнительно: Настройка SSL

Для HTTPS установите бесплатный сертификат Let's Encrypt:

```bash
apt install certbot python3-certbot-nginx
certbot --nginx -d your_domain -d www.your_domain
```

Автообновление сертификатов:

```bash
systemctl enable certbot.timer
```

