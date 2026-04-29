Esta guía está diseñada para que cualquier principiante pueda transformar un servidor vacío en un host profesional capaz de servir múltiples sitios web.

> **Nota del Experto:** En el mundo del despliegue, Apache y Nginx son los "motores" que sirven tu web. Mientras que Apache es clásico y muy flexible, Nginx destaca por su velocidad y manejo de conexiones simultáneas. Aprender ambos te dará una ventaja competitiva enorme.

---

## 1. Preparación: Git y Repositorios

Git es esencial para mover tu código desde tu computadora (o GitHub) hacia el Droplet.

### Instalación de Git

Ubuntu suele traerlo, pero siempre debemos asegurar la última versión estable.

```bash
# Actualizamos los índices de paquetes del sistema
sudo apt update

# Instalamos git
sudo apt install git -y

# Verificamos la instalación
git --version
```

### Clonar un Repositorio

Para bajar tu proyecto, usamos `git clone`. Lo ideal es hacerlo en una carpeta organizada.

```bash
# Nos movemos a la carpeta de usuario o una carpeta temporal
cd ~

# Clonamos el repositorio (usa la URL de tu proyecto en GitHub)
# Esto creará una carpeta con el nombre del proyecto
git clone https://github.com/usuario/mi-proyecto-web.git
```

---

## 2. Servidor Web Apache2 y VirtualHosts

### ¿Qué es un VirtualHost?

Imagina que tu Droplet es un edificio. La dirección IP es la dirección del edificio. Un **VirtualHost** es como el número de departamento; permite que en un mismo servidor (edificio) convivan varios sitios web (departamentos) diferentes (ej: `web1.com` y `web2.com`), dirigiendo al usuario a la carpeta correcta según el dominio que escribió en su navegador.

### Configuración paso a paso (Apache)

```bash
# 1. Instalamos el servidor Apache
sudo apt install apache2 -y

# 2. Creamos la estructura de carpetas para el dominio
# El parámetro -p crea las carpetas padre si no existen
sudo mkdir -p /var/www/laveladaconquer.com/public_html

# 3. Asignamos permisos para que el servidor pueda leer los archivos
# 755 permite que el dueño escriba y el resto solo lea/ejecute
sudo chmod -R 755 /var/www/laveladaconquer.com

# 4. Creamos un archivo de prueba para verificar
echo "<h1>¡La Velada Conquer está Online!</h1>" | sudo tee /var/www/laveladaconquer.com/public_html/index.html

# 5. Creamos el archivo de configuración del VirtualHost
# Los archivos de Apache deben terminar en .conf
sudo nano /etc/apache2/sites-available/laveladaconquer.com.conf
```

**Dentro del archivo `.conf` pegamos lo siguiente:**

```apache
<VirtualHost *:80>
    ServerAdmin admin@laveladaconquer.com
    ServerName laveladaconquer.com
    ServerAlias www.laveladaconquer.com

    # Ruta absoluta donde está el index.html (Vital en proyectos complejos)
    DocumentRoot /var/www/laveladaconquer.com/public_html

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

**Activación:**

```bash
# Habilitamos el nuevo sitio
sudo a2ensite laveladaconquer.com.conf

# Deshabilitamos el sitio por defecto de Apache (opcional)
sudo a2dissite 000-default.conf

# Reiniciamos Apache para aplicar cambios
sudo systemctl restart apache2
```

---

## 3. Conexión de Dominio (DonDominio & DNS)

Para que el mundo vea tu web al escribir el nombre, debes apuntar el nombre a la IP.

1.  **En DonDominio:** Accede a la gestión de tu dominio -> **Zonas DNS**.
2.  **Registro A:** Crea o modifica el registro de tipo **A**. En "Nombre/Host" pones `@` (o déjalo vacío) y en "Valor/Destino" pegas la **IP de tu Droplet**.
3.  **Registro CNAME:** Crea uno para `www` que apunte a tu dominio principal.
4.  **Verificación:** Entra en [DNS Checker](https://dnschecker.org). Escribe tu dominio y selecciona "A". Si aparecen checks verdes con la IP de tu Droplet, la propagación ha comenzado.

---

## 4. Servidor Web Nginx (Alternativa Profesional)

Muchos desarrolladores prefieren Nginx por su eficiencia. No puedes tener Apache y Nginx funcionando en el mismo puerto (80) a la vez, así que elige uno.

### Instalación

```bash
sudo apt update
sudo apt install nginx -y
```

### VirtualHost en Nginx (Server Blocks)

En Nginx, los VirtualHosts se llaman "Server Blocks". La lógica es similar pero la sintaxis cambia.

```bash
# 1. Creamos la carpeta (si no la creamos antes en el paso de Apache)
sudo mkdir -p /var/www/mi-sitio-nginx.com/html

# 2. Creamos el archivo de configuración
sudo nano /etc/nginx/sites-available/mi-sitio-nginx.com
```

**Configuración de Nginx:**

```nginx
server {
    listen 80;
    listen [::]:80;

    root /var/www/mi-sitio-nginx.com/html;
    index index.html index.htm;

    server_name mi-sitio-nginx.com www.mi-sitio-nginx.com;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Activación en Nginx:**
A diferencia de Apache, en Nginx activamos el sitio creando un "enlace simbólico" hacia la carpeta de sitios habilitados.

```bash
# Creamos el enlace
sudo ln -s /etc/nginx/sites-available/mi-sitio-nginx.com /etc/nginx/sites-enabled/

# Verificamos que la sintaxis es correcta (Paso profesional muy importante)
sudo nginx -t

# Reiniciamos Nginx
sudo systemctl restart nginx
```

### Proyectos Complejos

En proyectos como Laravel, React o Node.js, la ruta del `index.html` (o el punto de entrada) suele estar en una subcarpeta llamada `/public` o `/dist`. Es **fundamental** que en el archivo de configuración (`.conf` o server block) la directiva `DocumentRoot` (Apache) o `root` (Nginx) apunte exactamente a esa carpeta y no a la raíz del proyecto, por seguridad y funcionamiento.
