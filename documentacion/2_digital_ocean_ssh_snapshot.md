Esta es una guía técnica profesional diseñada para principiantes que desean configurar su infraestructura en DigitalOcean siguiendo las mejores prácticas de seguridad.

---

# 🚀 Guía de Configuración Profesional de Droplets en DigitalOcean

> **Nota del Experto:** Operar como usuario `root` de forma permanente es un riesgo de seguridad. Esta documentación te enseñará a blindar tu servidor desde el primer minuto, creando un entorno de trabajo seguro y escalable.

---

## 1. Conexión SSH por primera vez (Método Password)

Cuando creas un Droplet con contraseña, DigitalOcean te envía la IP por correo o la visualizas en el panel. SSH (Secure Shell) es el túnel cifrado para controlar tu Linux.

**Ejemplo:**

```bash
# Reemplaza 'tu_ip_servidor' por la IP real de tu Droplet (ej: 157.245.120.80)
ssh root@tu_ip_servidor

# El sistema te preguntará si confías en el host. Escribe 'yes'.
# Luego introduce la contraseña que definiste al crear el Droplet.
# Si es la primera vez, el servidor te obligará a cambiarla inmediatamente.
```

---

## 2. Gestión de Usuarios y Privilegios (Sudo)

Por seguridad, crearemos un usuario normal para el día a día. Usaremos `adduser` para crear el perfil y `usermod` para darle "superpoderes" (sudo).

**Ejemplo:**

```bash
# Creamos el nuevo usuario llamado 'operador'
# Te pedirá una contraseña y datos opcionales (nombre, tlf, etc.)
adduser operador

# Añadimos al usuario al grupo 'sudo' para que pueda ejecutar comandos de administrador
usermod -aG sudo operador

# Ahora 'operador' tiene permisos profesionales, pero debe usar la palabra 'sudo' antes de comandos críticos.
```

---

## 3. Configuración del Firewall (UFW)

El Firewall protege tu servidor cerrando todos los puertos excepto los que tú autorices. En Ubuntu usamos **UFW** (Uncomplicated Firewall).

### Comandos explicados:

1.  **`ufw app list`**: Muestra qué aplicaciones instaladas tienen un perfil de seguridad listo.
2.  **`ufw allow OpenSSH`**: Abre el puerto 22 para que no pierdas la conexión al activar el firewall.
3.  **`ufw allow http`**: Abre el puerto 80 (web básica).
4.  **`ufw allow https`**: Abre el puerto 443 (web segura/SSL).
5.  **`ufw enable`**: Activa el firewall.
6.  **`ufw status`**: Muestra qué puertos están abiertos actualmente.

**Ejemplo:**

```bash
# Listamos aplicaciones
sudo ufw app list

# Permitimos SSH (CRÍTICO: haz esto antes de activar ufw)
sudo ufw allow OpenSSH

# Permitimos tráfico web
sudo ufw allow http
sudo ufw allow https

# Activamos el firewall (confirma con 'y')
sudo ufw enable

# Verificamos que todo está en orden
sudo ufw status
```

---

## 4. Transición de Usuario y Seguridad por Clave Pública (SSH Keys)

Para una seguridad nivel profesional, eliminaremos el uso de contraseñas. Usaremos un par de llaves: una **privada** (en tu PC) y una **pública** (en el servidor).

**Pasos:**

1. Sal de la sesión root: `exit`.
2. Inicia sesión con tu nuevo usuario: `ssh operador@tu_ip`.
3. Crea tu llave en **tu propia computadora** (no en el servidor):

**Ejemplo (En tu PC local):**

```bash
# Generar el par de llaves (Enter a todo para valores por defecto)
ssh-keygen -t rsa -b 4096

# Copiamos el contenido de la llave pública GENERADA EN TU PC
# Normalmente está en ~/.ssh/id_rsa.pub
cat ~/.ssh/id_rsa.pub
```

**Ejemplo (En el Servidor, con el usuario 'operador'):**

```bash
# Crear el directorio oculto de configuración SSH
mkdir ~/.ssh

# Crear el archivo donde pegaremos nuestra clave pública de la PC
nano ~/.ssh/authorized_keys

# [Pega aquí el contenido que copiaste de tu PC y guarda con Ctrl+O, Enter, Ctrl+X]

# Ajustamos permisos (Vital para que SSH funcione)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

## 5. Mantenimiento de Energía: Apagar la máquina

Apagar el servidor desde la consola es preferible a hacerlo desde el botón de "Power" del panel, ya que permite al sistema cerrar procesos correctamente.

**Ejemplo:**

```bash
# Apagado inmediato y seguro
sudo shutdown -h now
```

---

## 6. Snapshots: Copias de Seguridad Manuales

Un **Snapshot** es una imagen completa del disco de tu Droplet en un momento exacto.

- **Uso:** Ideal para clonar servidores o guardar un estado antes de una actualización arriesgada.
- **Coste:** DigitalOcean cobra **$0.05 por GB al mes**. Si tu Snapshot ocupa 10GB, pagarás $0.50/mes.
- **Cómo tomarlo:** Ve a la pestaña **Snapshots** en el panel lateral de tu Droplet, ponle un nombre y pulsa "Take Snapshot". _Nota: El servidor debe estar apagado para mayor integridad._

---

## 7. Restauración, Backups y Redimensionamiento (Resize)

### Restaurar desde Snapshot

Si algo sale mal, ve a la pestaña **Snapshots**, busca tu copia y selecciona **Restore**. Esto borrará el contenido actual y dejará el servidor exactamente como estaba cuando hiciste la copia.

### Establecer Backups

A diferencia de los Snapshots manuales, los **Backups** son automáticos y semanales.

- **Coste:** 20% del valor de tu Droplet.
- **Activación:** Se puede activar al crear el Droplet o después en la pestaña **Backups**.

### Redimensionar (Resize)

Si tu proyecto crece y necesitas más potencia:

1. Apaga el Droplet.
2. Ve a **Resize**.
3. Elige entre:
   - **CPU and RAM only:** Mantiene el disco igual (permite volver atrás).
   - **Disk, CPU and RAM:** Aumenta el disco permanentemente (no podrás bajar el plan después).
4. Enciende el Droplet.

---

_Documentación generada para despliegues profesionales en DigitalOcean._
