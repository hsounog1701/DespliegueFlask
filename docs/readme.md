# Despliegue de Flask con Gunicorn y Nginx

Este repositorio contiene la documentación para desplegar una aplicación Flask utilizando Gunicorn como servidor de aplicaciones y Nginx como proxy inverso.

## Requisitos previos

- Sistema Debian/Ubuntu
- Python 3
- Nginx
- Permisos de administrador

## Instalación de dependencias

```bash
# Actualizar el sistema e instalar pip
sudo apt-get update && sudo apt-get install -y python3-pip

# Instalar pipenv
pip3 install pipenv

# Verificar instalación
PATH=$PATH:/home/$USER/.local/bin
pipenv --version

# Instalar python-dotenv
pip3 install python-dotenv
```

## Configuración del proyecto

### 1. Crear estructura de directorios

```bash
# Crear directorio del proyecto
sudo mkdir -p /var/www/app

# Cambiar permisos
sudo chown -R $USER:www-data /var/www/app
sudo chmod -R 775 /var/www/app
```

### 2. Configurar variables de entorno

Crear archivo `/var/www/app/.env`:

```bash
FLASK_APP=wsgi.py
FLASK_ENV=production
```

### 3. Crear entorno virtual e instalar dependencias

```bash
cd /var/www/app
pipenv shell
pipenv install flask gunicorn
```

### 4. Crear la aplicación Flask

**Archivo `/var/www/app/application.py`:**

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    '''Index page route'''
    return '<h1>App desplegada</h1>'
```

**Archivo `/var/www/app/wsgi.py`:**

```python
from application import app

if __name__ == '__main__':
   app.run(debug=False)
```

### 5. Probar la aplicación

```bash
# Con Flask (desarrollo)
flask run --host '0.0.0.0'

# Con Gunicorn (producción)
gunicorn --workers 4 --bind 0.0.0.0:5000 wsgi:app
```

## Configuración de Gunicorn como servicio

### 1. Obtener ruta de Gunicorn

```bash
which gunicorn
# Ejemplo: /home/vagrant/.local/share/virtualenvs/app-1lvW3LzD/bin/gunicorn
```

### 2. Crear servicio systemd

Crear archivo `/etc/systemd/system/flask_app.service`:

```ini
[Unit]
Description=flask app service - App con flask y Gunicorn
After=network.target

[Service]
User=vagrant
Group=www-data
Environment="PATH=/home/vagrant/.local/share/virtualenvs/app-1lvW3LzD/bin"
WorkingDirectory=/var/www/app
ExecStart=/home/vagrant/.local/share/virtualenvs/app-1lvW3LzD/bin/gunicorn --workers 3 --bind unix:/var/www/app/app.sock wsgi:app

[Install]
WantedBy=multi-user.target
```

**⚠️ Nota:** Ajustar las rutas según tu instalación.

### 3. Iniciar el servicio

```bash
sudo systemctl daemon-reload
sudo systemctl enable flask_app
sudo systemctl start flask_app
```

## Configuración de Nginx

### 1. Crear configuración del sitio

Crear archivo `/etc/nginx/sites-available/app.conf`:

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

### 2. Activar el sitio

```bash
# Crear enlace simbólico
sudo ln -s /etc/nginx/sites-available/app.conf /etc/nginx/sites-enabled/

# Verificar configuración
sudo nginx -t

# Reiniciar Nginx
sudo systemctl restart nginx
sudo systemctl status nginx
```

### 3. Configurar hosts (local)

Editar `/etc/hosts` (Linux) o `C:\Windows\System32\drivers\etc\hosts` (Windows):

```
192.168.X.X app.izv www.app.izv
```

## Verificación

Acceder desde el navegador a: `http://app.izv/`

Deberías ver: **App desplegada**

---

## Tarea de ampliación

Repetir el proceso con el repositorio:
```
https://github.com/Azure-Samples/msdocs-python-flask-webapp-quickstart
```

### Pasos específicos:

```bash
# Clonar repositorio
cd /var/www
git clone https://github.com/Azure-Samples/msdocs-python-flask-webapp-quickstart

# Navegar al directorio
cd msdocs-python-flask-webapp-quickstart

# Crear entorno virtual e instalar dependencias
pipenv shell
pipenv install -r requirements.txt

# Iniciar Gunicorn
gunicorn --workers 4 --bind 0.0.0.0:5000 wsgi:app
```

El resto del proceso es idéntico, solo ajusta los paths en la configuración de systemd y nginx.

---

## Tecnologías utilizadas

- **Python** - Lenguaje de programación
- **Flask** - Framework web ligero
- **Gunicorn** - Servidor WSGI para Python
- **Nginx** - Servidor web y proxy inverso
- **Pipenv** - Gestor de entornos virtuales

## Estructura del proyecto

```
/var/www/app/
├── .env
├── application.py
├── wsgi.py
└── Pipfile
```

## Referencias

- [Flask Documentation](https://flask.palletsprojects.com/)
- [Gunicorn Documentation](https://gunicorn.org/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Pipenv Documentation](https://pipenv.pypa.io/)

## Autor

Higor De Souza Nogueira Eufrasio - 2026