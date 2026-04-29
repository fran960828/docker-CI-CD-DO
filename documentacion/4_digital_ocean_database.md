Esta guía técnica te llevará paso a paso por la configuración profesional de bases de datos en DigitalOcean. Aprenderás a asegurar el motor de base de datos y a gestionar accesos granulares, algo vital para evitar filtraciones de datos.

---

# 🗄️ Guía Profesional de Bases de Datos: MySQL y PostgreSQL

> **Nota del Experto:** Instalar una base de datos es sencillo, pero **asegurarla** es lo que separa a un principiante de un profesional. Nunca dejes una base de datos con acceso abierto a todo el mundo (`0.0.0.0/0`) y siempre utiliza contraseñas robustas.

---

## 1. MySQL: Instalación y Seguridad

MySQL es el sistema de gestión de bases de datos relacionales más popular del mundo.

### Paso 1: Instalación

```bash
# Actualizamos los repositorios e instalamos el paquete del servidor
sudo apt update
sudo apt install mysql-server -y

# Verificamos que el servicio esté corriendo
sudo systemctl status mysql
```

### Paso 2: mysql_secure_installation

Este script elimina configuraciones inseguras que vienen por defecto.

1. **VALIDATE PASSWORD COMPONENT:** Te pregunta si quieres forzar contraseñas seguras. (Recomendado: Yes).
2. **Remove anonymous users:** Elimina usuarios sin nombre que permiten entrar a cualquiera. (Recomendado: Yes).
3. **Disallow root login remotely:** Prohíbe que el usuario root entre desde fuera del servidor. (Recomendado: Yes).
4. **Remove test database:** Borra una base de datos pública de ejemplo. (Recomendado: Yes).
5. **Reload privilege tables:** Aplica los cambios inmediatamente. (Recomendado: Yes).

### Paso 3: Acceso y cambio de contraseña Root

En las versiones modernas de Ubuntu, el root de MySQL usa el plugin `auth_socket` (se entra con `sudo` desde la terminal), pero a veces necesitamos ponerle una contraseña clásica.

```bash
# Accedemos a la terminal de MySQL
sudo mysql

# Cambiamos la contraseña del root (reemplaza 'TuClaveSegura123!')
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'TuClaveSegura123!';

# Refrescamos permisos y salimos
FLUSH PRIVILEGES;
exit;
```

---

## 2. Gestión de Usuarios y Accesos Remotos (MySQL)

### Crear DB y Usuario con permisos

```sql
-- Entramos a MySQL (ahora pedirá contraseña)
mysql -u root -p

-- 1. Crear la base de datos
CREATE DATABASE mi_proyecto_db;

-- 2. Crear un usuario que solo puede entrar desde la IP de tu casa/oficina
-- Reemplaza '203.0.113.0' por tu IP pública real
CREATE USER 'dev_user'@'203.0.113.0' IDENTIFIED BY 'PasswordDificil456!';

-- 3. Darle todos los privilegios sobre esa base de datos específica
GRANT ALL PRIVILEGES ON mi_proyecto_db.* TO 'dev_user'@'203.0.113.0';

-- 4. Aplicar cambios
FLUSH PRIVILEGES;
EXIT;
```

### Permitir conexión externa (Bind Address)

Por defecto, MySQL solo escucha peticiones de la propia máquina (`127.0.0.1`). Debemos cambiarlo para que escuche a internet.

```bash
# Editamos el archivo de configuración
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

# Buscamos la línea 'bind-address' y la cambiamos a:
# bind-address = 0.0.0.0  (Esto significa que escucha en todas las IPs del Droplet)

# Reiniciamos el servicio
sudo systemctl restart mysql

# IMPORTANTE: Abre el puerto 3306 en el Firewall de DigitalOcean o en UFW
sudo ufw allow from 203.0.113.0 to any port 3306
```

---

## 3. PostgreSQL: La alternativa avanzada

PostgreSQL es conocido por su robustez y cumplimiento de estándares SQL.

### Instalación

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
```

### Gestión de Usuarios y DB

Postgres usa el concepto de "roles". Por defecto, el usuario administrador es `postgres`.

```bash
# Entramos como el usuario del sistema 'postgres' y abrimos su consola (psql)
sudo -u postgres psql

-- 1. Cambiar contraseña del administrador de la DB
ALTER USER postgres WITH PASSWORD 'NuevaClavePostgres';

-- 2. Crear base de datos y usuario
CREATE DATABASE proyecto_postgre;
CREATE USER usuario_pro WITH PASSWORD 'ClavePro123';

-- 3. Asignar privilegios
GRANT ALL PRIVILEGES ON DATABASE proyecto_postgre TO usuario_pro;

-- Salir de la consola
\q
```

### Permitir conexión externa en PostgreSQL

Esto requiere dos pasos distintos a MySQL:

**Paso 1: Configurar escucha**

```bash
sudo nano /etc/postgresql/14/main/postgresql.conf
# (La versión 14 puede variar según tu Ubuntu)

# Buscamos 'listen_addresses' y lo cambiamos a:
listen_addresses = '*'
```

**Paso 2: Configurar autenticación de red (HBA)**

```bash
sudo nano /etc/postgresql/14/main/pg_hba.conf

# Añadimos al final una línea para tu IP específica
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    proyecto_postgre usuario_pro     203.0.113.0/32          scram-sha-256

# Reiniciamos para aplicar
sudo systemctl restart postgresql
```

> **Consejo final:** Siempre que abras una base de datos al exterior, asegúrate de que el usuario que creas tiene permisos **solo** para su base de datos y no para todo el sistema, y que tu firewall (UFW) solo permita tu IP específica.
