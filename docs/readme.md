# Despliegue de Flask con Gunicorn y Nginx

Guía práctica para desplegar una aplicación Flask usando Gunicorn como servidor de aplicaciones y Nginx como proxy inverso.

## Requisitos

- Sistema Debian/Ubuntu
- Python 3 y pip
- Nginx instalado
- Permisos de sudo

## Instalación inicial

```bash
# Actualizar sistema e instalar dependencias
sudo apt-get update && sudo apt-get install -y python3-pip
pip3 install pipenv python-dotenv

# Añadir pipenv al PATH
export PATH=$PATH:/home/$USER/.local/bin
```

## Configuración del proyecto

### 1. Estructura de directorios

```bash
sudo mkdir -p /var/www/app
sudo chown -R $USER:www-data /var/www/app
sudo chmod -R 775 /var/www/app
```

### 2. Variables de entorno

Crea el archivo `/var/www/app/.env`:

```
FLASK_APP=wsgi.py
FLASK_ENV=production
```

### 3. Instalar dependencias

```bash
cd /var/www/app
pipenv shell
pipenv install flask gunicorn
```

### 4. Aplicación Flask

**application.py:**
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return '<h1>App desplegada</h1>'
```

**wsgi.py:**
```python
from application import app

if __name__ == '__main__':
   app.run(debug=False)
```

### 5. Prueba local

```bash
# Desarrollo
flask run --host '0.0.0.0'

# Producción
gunicorn --workers 4 --bind 0.0.0.0:5000 wsgi:app
```

## Servicio systemd

Primero, obtén la ruta de Gunicorn:
```bash
which gunicorn
```

Crea `/etc/systemd/system/flask_app.service` (ajusta las rutas según tu instalación):

```ini
[Unit]
Description=Servicio Flask con Gunicorn
After=network.target

[Service]
User=vagrant
Group=www-data
Environment="PATH=/home/vagrant/.local/share/virtualenvs/app-XXXXX/bin"
WorkingDirectory=/var/www/app
ExecStart=/home/vagrant/.local/share/virtualenvs/app-XXXXX/bin/gunicorn --workers 3 --bind unix:/var/www/app/app.sock wsgi:app

[Install]
WantedBy=multi-user.target
```

Activa el servicio:
```bash
sudo systemctl daemon-reload
sudo systemctl enable flask_app
sudo systemctl start flask_app
```

## Configuración de Nginx

Crea `/etc/nginx/sites-available/app.conf`:

```nginx
server {
  listen 80;
  server_name app.izv www.app.izv;

  access_log /var/log/nginx/app.access.log;
  error_log /var/log/nginx/app.error.log;

  location / {
    include proxy_params;
    proxy_pass http://unix:/var/www/app/app.sock;
  }
}
```

Activa la configuración:
```bash
sudo ln -s /etc/nginx/sites-available/app.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### Configurar hosts localmente

Añade a `/etc/hosts` (Linux) o `C:\Windows\System32\drivers\etc\hosts` (Windows):
```
192.168.X.X app.izv www.app.izv
```

## Verificación

Accede a http://app.izv/ y deberías ver tu aplicación funcionando.

---

## Ampliación: Desplegar otro proyecto

Para desplegar el proyecto de ejemplo de Azure:

```bash
cd /var/www
git clone https://github.com/Azure-Samples/msdocs-python-flask-webapp-quickstart
cd msdocs-python-flask-webapp-quickstart

pipenv shell
pipenv install -r requirements.txt
gunicorn --workers 4 --bind 0.0.0.0:5000 wsgi:app
```

Luego sigue los mismos pasos de configuración de systemd y Nginx, ajustando las rutas correspondientes.
