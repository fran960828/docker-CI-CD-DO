Esta guía técnica está diseñada para elevar tu nivel de gestión en DigitalOcean utilizando **CloudPanel**, un panel de control moderno, ligero y extremadamente rápido, optimizado para aplicaciones PHP y Python (Django).

> **Nota del Experto:** CloudPanel simplifica la gestión de Nginx y Vhosts, pero recuerda que bajo el capó seguimos usando la arquitectura que hemos aprendido: Nginx como Proxy Inverso y Gunicorn como servidor de aplicaciones. La seguridad aquí es clave, especialmente al exponer paneles de administración.

---

## 1. Reinicio del Droplet: Soft vs Hard Reboot

Reiniciar es una tarea común, pero existen dos formas de hacerlo según el estado de tu servidor.

- **Soft Reboot (Recomendado):** Se hace desde la consola. Ordena al sistema cerrar procesos, guardar datos en disco y reiniciarse de forma segura.
- **Hard Reboot (Power Cycle):** Se hace desde el panel de DigitalOcean. Es equivalente a tirar del cable. Solo debe usarse si el servidor no responde a comandos.

**Ejemplo de reinicio seguro por consola:**

```bash
# Comando para reiniciar el sistema de forma ordenada
sudo reboot

# Una vez ejecutado, perderás la conexión SSH.
# Debes esperar unos 30-60 segundos para volver a conectar.
```

---

## 2. Instalación de CloudPanel en Ubuntu

CloudPanel requiere una instalación limpia de Ubuntu. Antes de lanzarlo, debemos preparar el terreno con las dependencias necesarias.

**Ejemplo de instalación profesional:**

```bash
# 1. Actualizamos el sistema a las últimas versiones estables
sudo apt update && sudo apt upgrade -y

# 2. Instalamos dependencias básicas (Curl y soporte para software externo)
sudo apt install curl wget gnupg2 software-properties-common -y

# 3. Ejecutamos el instalador oficial de CloudPanel para la versión de Python (Django)
# El instalador configurará automáticamente el stack de Nginx y bases de datos.
curl -sS https://installer.cloudpanel.io/ce/v2/install.sh | sudo bash
```

---

## 3. Seguridad del Panel (Puerto 8443) y Firewall

CloudPanel utiliza el puerto **8443** para su interfaz web. Dejarlo abierto a todo el mundo es un riesgo. Configuraremos el Cloud Firewall de DigitalOcean para que **solo tu IP** pueda verlo.

### Pasos en el Panel de DigitalOcean:

1.  Ve a **Networking** -> **Firewalls** -> **Create Firewall**.
2.  **Inbound Rules:**
    - **SSH:** Puerto 22 (Tu IP).
    - **HTTP/HTTPS:** Puertos 80/443 (All IPv4/IPv6).
    - **CloudPanel Admin:** Añade una regla personalizada para el puerto **8443**. En el campo "Source", borra "All" y busca "My IP".
3.  **Apply to Droplets:** Busca el nombre de tu Droplet y selecciónalo.

---

## 4. Despliegue de Django con CloudPanel (Proxy Inverso)

CloudPanel facilita la creación de sitios, pero para Django usaremos la opción de **Reverse Proxy**. Esto significa que Nginx recibirá la petición y la pasará a Gunicorn.

### Paso A: Crear el sitio en CloudPanel

1.  Accede a `https://tu-ip:8443`.
2.  **Add Site** -> **Create a Python Site** (o **Reverse Proxy Site** si ya tienes Gunicorn corriendo).
3.  Ingresa tu dominio (ej: `misitio.com`) y el **Application Port** (normalmente el `8000` que usa Gunicorn).

### Paso B: Configuración del entorno (Gunicorn y Usuarios)

CloudPanel crea un usuario específico para cada sitio por seguridad. Debes usar ese usuario para clonar tu proyecto.

```bash
# 1. Cambiamos al usuario del sitio creado por CloudPanel (ej: clp-user)
sudo su - clp-user

# 2. Clonamos y preparamos el entorno virtual (como vimos en guías anteriores)
cd htdocs/misitio.com
virtualenv env
source env/bin/activate
pip install django gunicorn

# 3. Configuramos Gunicorn como servicio (Systemd)
# NOTA: Sigue los pasos de la guía anterior pero asegurándote de que:
# User=clp-user
# Group=www-data
# ExecStart apunta al socket o puerto 8000.
```

### Paso C: Nginx y SSL en CloudPanel

CloudPanel gestiona Nginx de forma visual.

1.  En el panel del sitio, ve a la pestaña **Vhost**. CloudPanel ya tendrá una configuración base de Proxy Inverso.
2.  **Configuración de Proxy:** Asegúrate de que la línea `proxy_pass http://127.0.0.1:8000;` apunte al puerto donde corre tu Gunicorn.
3.  **SSL:** Ve a la pestaña **SSL Store** dentro del sitio en CloudPanel, selecciona **Let's Encrypt** y haz clic en **Install**. CloudPanel se encargará de renovarlo automáticamente.

### Paso D: Permisos y Firewall Interno

- **Permisos:** Asegúrate de que los archivos estáticos tengan permisos de lectura para el grupo `www-data` para que Nginx pueda servirlos.
- **Firewall:** Si usas el firewall de DigitalOcean, asegúrate de que el puerto 80 y 443 estén abiertos para todos, pero el puerto 8000 (Gunicorn) no necesita estar abierto al exterior, ya que Nginx se comunica con él internamente.

**Verificación Final:**
Al visitar `https://misitio.com`, Nginx recibirá la petición, verificará el certificado SSL gestionado por CloudPanel, y pasará la carga a Gunicorn, quien ejecutará tu código Django de forma eficiente.
