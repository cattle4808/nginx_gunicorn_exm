# Django Deployment Template with Nginx and Gunicorn

This repository provides a comprehensive guide and setup files for deploying Django applications using **Gunicorn** as the application server and **Nginx** as a reverse proxy.

---

## Features

- Gunicorn systemd service configuration
- Nginx reverse proxy configuration
- SSL support with Let's Encrypt
- Easy-to-follow deployment process

---

## Quick Start

1. Clone this repository:
   ```bash
   git clone <repository_url>
   ```

2. Follow the [Deployment Guide](https://github.com/cattle4808/nginx_gunicorn_exm/blob/main/nginx_gunicorn_settings_for_django.md) for step-by-step instructions.

3. Customize the provided systemd and Nginx configuration files for your project:
   - Update paths in `gunicorn.service`.
   - Set your domain or IP in the Nginx configuration.

4. Test and reload:
   ```bash
   systemctl daemon-reload
   systemctl restart gunicorn.service
   systemctl restart nginx
   ```

---

## Repository Structure

```plaintext
.
├── gunicorn.service       # Template for Gunicorn systemd service
├── gunicorn.socket        # Template for Gunicorn socket
├── nginx/                 # Nginx configurations (sites-available and sites-enabled)
├── README.md              # This file
└── deployment_guide.md    # Full deployment instructions
```

---

## Related Resources

- [Detailed Deployment Guide](https://github.com/cattle4808/nginx_gunicorn_exm/blob/main/nginx_gunicorn_settings_for_django.md)
- [Gunicorn Documentation](https://docs.gunicorn.org/en/stable/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Django Deployment Checklist](https://docs.djangoproject.com/en/stable/howto/deployment/checklist/)

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

## Contributing

Contributions are welcome! Please submit a pull request or open an issue to suggest improvements or report bugs.

