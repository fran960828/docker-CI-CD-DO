Esta es la guía definitiva para desplegar Django. He desglosado cada comando y archivo de configuración para que entiendas no solo **qué** estás escribiendo, sino **por qué** es necesario.

> **Nota del Experto:** Desplegar Django es como armar un reloj. Tenemos la maquinaria (Python/Gunicorn), la esfera que el cliente ve (Nginx) y la base que sostiene todo (Postgres). Si una pieza falla, el reloj se detiene. Sigue cada paso con atención a los nombres de las carpetas.

---

## 1. Preparación del Sistema (La Base)

Antes de subir tu código, necesitamos que el Droplet entienda Python y sepa comunicarse con bases de datos.

```bash
# Actualizamos la lista de software disponible para tener lo último.
sudo apt update

# Instalamos:
# python3-pip: El gestor de paquetes de Python.
# python3-dev: Cabeceras necesarias para compilar extensiones de Python.
# libpq-dev: Necesario para que Python pueda hablar con PostgreSQL.
# postgresql: El motor de base de datos.
# nginx: El servidor que recibirá las visitas de internet.
sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl -y

# Instalamos virtualenv globalmente para crear entornos aislados.
sudo pip3 install virtualenv
```

---

## 2. Creación del Entorno de Trabajo

Es vital que cada proyecto tenga su propia "caja" (entorno virtual) para que las librerías de un proyecto no rompan las de otro.

```bash
# 1. Crea la carpeta de tu proyecto.
# MODIFICAR: Cambia 'mi_proyecto' por el nombre de tu web.
mkdir ~/mi_proyecto && cd ~/mi_proyecto

# 2. Creamos el entorno virtual llamado 'env'.
virtualenv env

# 3. Activamos el entorno.
# IMPORTANTE: Verás que tu terminal ahora empieza con (env).
# Esto significa que todo lo que instales ahora se quedará SOLO en esta carpeta.
source env/bin/activate

# 4. Instalamos Django y las herramientas de producción.
# gunicorn: El servidor que ejecutará tu código Python.
# psycopg2-binary: El conector entre Django y Postgres.
pip install django gunicorn psycopg2-binary
```

---

## 3. Configuración de Django para Producción

Debes modificar el archivo `settings.py` de tu proyecto Django para que sea seguro.

```python
# MODIFICACIONES EN settings.py:

# 1. Desactiva el modo de depuración (¡NUNCA lo dejes en True en el Droplet!)
DEBUG = False

# 2. Pon tu dominio y la IP de tu Droplet.
# MODIFICAR: Cambia por tus datos reales.
ALLOWED_HOSTS = ['tusitio.com', 'www.tusitio.com', 'tu_ip_del_droplet']

# 3. Configura la base de datos (Usa los datos que creaste en la guía de Postgres).
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'nombre_db',
        'USER': 'usuario_db',
        'PASSWORD': 'password_db',
        'HOST': 'localhost',
        'PORT': '',
    }
}

# 4. Indica dónde se guardarán los archivos estáticos (CSS, Imágenes).
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
```

---

## 4. Gunicorn: El corazón de la aplicación

Gunicorn ejecutará tu Django en segundo plano. Para que sea profesional, lo configuramos como un **servicio del sistema** que se inicie solo al encender el Droplet.

### Paso A: El Socket

Es un "punto de encuentro" entre Nginx y Gunicorn.
`sudo nano /etc/systemd/system/gunicorn.socket`

```ini
[Unit]
Description=gunicorn socket

[Socket]
# Crea un archivo temporal en el sistema para la comunicación
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

### Paso B: El Servicio

Aquí es donde le decimos al sistema cómo ejecutar tu app.
`sudo nano /etc/systemd/system/gunicorn.service`

```ini
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
# MODIFICAR: 'operador' debe ser el nombre de tu usuario de Ubuntu.
User=operador
Group=www-data
# MODIFICAR: Ruta exacta a la carpeta de tu proyecto.
WorkingDirectory=/home/operador/mi_proyecto
# MODIFICAR: Ruta al ejecutable de gunicorn dentro de tu entorno virtual.
# Y cambia 'mi_proyecto.wsgi' por el nombre de la carpeta de tu app (donde esté wsgi.py).
ExecStart=/home/operador/mi_proyecto/env/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          mi_proyecto.wsgi:application

[Install]
WantedBy=multi-user.target
```

---

## 5. Nginx: El Escudo Exterior

Nginx recibirá a los usuarios y decidirá: si piden una imagen, la sirve él; si piden la web, se la pide a Gunicorn.

`sudo nano /etc/nginx/sites-available/mi_proyecto`

```nginx
server {
    listen 80;
    # MODIFICAR: Tus dominios reales.
    server_name tusitio.com www.tusitio.com;

    location = /favicon.ico { access_log off; log_not_found off; }

    # MODIFICAR: Ruta a la carpeta static que configuraste en settings.py.
    location /static/ {
        root /home/operador/mi_proyecto;
    }

    # El resto de peticiones se envían al socket de Gunicorn.
    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```

---

## 6. Activación y Comprobación

Este es el momento de la verdad. Debemos activar los servicios y abrir los muros del servidor.

```bash
# 1. Activamos el socket de Gunicorn
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket

# 2. Enlazamos la configuración de Nginx para activarla
sudo ln -s /etc/nginx/sites-available/mi_proyecto /etc/nginx/sites-enabled

# 3. Comprobamos que no haya errores de escritura en Nginx
sudo nginx -t

# 4. Reiniciamos Nginx para que lea los cambios
sudo systemctl restart nginx

# 5. Abrimos el Firewall para que el mundo pueda entrar
sudo ufw allow 'Nginx Full'
```

### ¿Qué debes modificar siempre?

1.  **Nombres de usuario:** Asegúrate de no usar `root` en los archivos de servicio, usa el usuario que creaste (ej: `operador`).
2.  **Rutas (Paths):** Si tu proyecto está en `/var/www/` en lugar de `/home/user/`, cámbialo en todos los archivos.
3.  **WSGI:** En el comando de Gunicorn, `mi_proyecto.wsgi` se refiere al archivo `wsgi.py` que está dentro de la carpeta interior de tu proyecto Django. Si tu carpeta se llama `tienda`, sería `tienda.wsgi:application`.
