¡Hola\! Bienvenido al mundo del despliegue en la nube. Como experto en DigitalOcean, he preparado esta guía técnica estructurada para que pases de cero a tener una infraestructura profesional.

> **Nota para principiantes:** DigitalOcean se basa en la simplicidad. A diferencia de otros proveedores más complejos, aquí todo está diseñado para que el desarrollador se centre en el código y no solo en la administración de sistemas.

---

## 1\. Glosario de Infraestructura Cloud

Conceptos fundamentales para entender cómo se conectan las piezas de tu servidor.

- **VPS (Virtual Private Server):** Un servidor físico dividido en varios servidores virtuales. Tienes control total (root) sobre tu parcela.
- **Droplet:** Es el nombre comercial que DigitalOcean da a sus VPS.
- **Kubernetes (DOKS):** Sistema para automatizar el despliegue y escalado de aplicaciones en contenedores (Docker).
- **VPC (Virtual Private Cloud):** Una red privada aislada donde tus Droplets pueden comunicarse entre sí de forma segura sin salir a internet.
- **Cloud Firewalls:** Barreras de seguridad gratuitas que filtran el tráfico (puertos) antes de que llegue a tu servidor.
- **Load Balancers:** Distribuyen el tráfico entrante entre varios Droplets para evitar saturaciones.
- **DNS:** Sistema que traduce nombres de dominio (https://www.google.com/search?q=google.com) en direcciones IP.
- **DDoS Protection:** Capa de seguridad que mitiga ataques que intentan tumbar tu web por exceso de tráfico.
- **Spaces (Object Storage):** Almacenamiento ilimitado para archivos estáticos (imágenes, vídeos), similar a Google Drive pero para apps.
- **Volumes (Block Storage):** "Discos duros" extra que puedes conectar y desconectar de tus Droplets.
- **Managed Databases:** Bases de datos configuradas y mantenidas por DigitalOcean (tú solo te preocupas de los datos).
- **Backups:** Copias de seguridad automáticas semanales.
- **Snapshots:** "Fotos" manuales del estado actual de tu servidor para clonarlo o restaurarlo en el futuro.
- **Floating IPs:** IPs estáticas que puedes mover de un Droplet a otro instantáneamente.

---

## 2\. Tipos de VPS (Droplets) y Cuándo Usarlos

| Tipo                  | Uso Ideal                                 | Descripción                                                   |
| :-------------------- | :---------------------------------------- | :------------------------------------------------------------ |
| **Basic**             | Blogs, staging, apps pequeñas.            | Opción económica con CPUs compartidas.                        |
| **General Purpose**   | E-commerce, servidores web con tráfico.   | Equilibrio entre CPU dedicada y RAM (4GB RAM por cada vCPU).  |
| **CPU-Optimized**     | Procesamiento de vídeo, machine learning. | Para tareas que requieren cálculos intensos y rapidez de CPU. |
| **Memory-Optimized**  | Bases de datos grandes, caché (Redis).    | Prioriza la memoria RAM (8GB por cada vCPU).                  |
| **Storage-Optimized** | Grandes bases de datos NoSQL, Logs.       | Incluye discos NVMe de altísima velocidad y capacidad.        |

---

## 3\. Precios y Facturación

DigitalOcean utiliza un modelo de **pago por uso**:

- **Cobro por hora:** Si creas un Droplet de 6$/mes y lo borras a las 10 horas, solo pagas esas 10 horas (aprox $0.009/hr).
- **Límite mensual:** Hay un precio máximo mensual. Nunca pagarás más de ese límite aunque el mes tenga 31 días.
- **Recursos apagados:** ¡Ojo\! Un Droplet **apagado sigue cobrando**, ya que reserva espacio en disco y memoria. Para dejar de pagar, debes **Destruirlo**.

---

## 4\. Primeros Pasos: Cuenta y Proyecto

1.  **Registro:** Crea tu cuenta en [digitalocean.com](https://www.digitalocean.com). Requiere tarjeta de crédito o PayPal para verificar identidad.
2.  **Proyecto:** Al entrar, crea un "New Project". Sirve para organizar tus recursos (ej: Proyecto "Tienda Online" separado de "Blog Personal").

---

## 5\. Creación de un Droplet (Paso a Paso)

Para crear tu primer servidor, sigue este flujo en el panel:

1.  **Choose Image:** Selecciona "OS" para una instalación limpia (Ubuntu 22.04 es el estándar) o "Marketplace" si quieres algo preinstalado (WordPress, Docker).
2.  **Choose Size:** Elige el tipo de Droplet (Basic, etc.).
3.  **Add Block Storage:** Si necesitas más de los 25GB que trae el plan básico, añade un Volumen aquí.
4.  **Datacenter Region:** Elige la más cercana a tus usuarios (ej: New York, Frankfurt).
5.  **Authentication:** \* **SSH Keys (Recomendado):** Más seguro. Usas una llave criptográfica.
    - **Password:** Menos seguro, pero más fácil para empezar.
6.  **Add-ons:**
    - **Backups:** Actívalo (cuesta un 20% extra del valor del Droplet).
    - **Monitoring:** Gratuito. Instala un agente para ver métricas avanzadas.

---

## 6\. Gestión Avanzada del Droplet

### Networking e IPs

- **IPv4:** Tu dirección pública para acceder desde internet.
- **IPv6:** El nuevo estándar de direcciones IP (opcional).
- **Private IP:** Dirección para hablar con otros Droplets en tu VPC sin coste de ancho de banda.

### Gráficos y Monitoreo (Graphs)

En la pestaña **Graphs** verás:

- **Bandwidth:** Cuántos datos entran/salen de tu servidor.
- **CPU Usage:** Si el procesador está al límite.
- **Disk I/O:** Velocidad de escritura/lectura del disco.

### Acceso y Control

- **Access:** Si pierdes tu contraseña o llave SSH, puedes usar la **Console** (un terminal web) o hacer un **Reset Root Password**.
- **Power:** Botón para **Power Off** (apagado seguro) o **Power Cycle** (reinicio forzado).
- **Resize:** Permite subir de plan (ej: pasar de 1GB a 2GB de RAM). _Nota: Ampliar CPU/RAM es fácil, pero reducir el disco es casi imposible sin recrear el Droplet._

### Seguridad y Recuperación

- **Firewall:** Configura reglas para que solo se pueda entrar por el puerto 80 (HTTP) y 443 (HTTPS).
- **Snapshots:** Haz uno antes de hacer un cambio crítico en tu servidor.
- **History:** Registro de todas las acciones realizadas en el Droplet.
- **Destroy / Rebuild:** \* **Destroy:** Borra el Droplet para siempre (deja de cobrar).
  - **Rebuild:** Reinstala el sistema operativo desde cero pero **mantiene la misma dirección IP**.
- **Recovery:** Si el sistema no arranca, puedes iniciar desde una "ISO de recuperación" para intentar salvar tus datos.

---

## Ejemplo Práctico: Despliegue de un Servidor Web

A continuación, simulamos el proceso de configuración inicial de un Droplet Ubuntu para un principiante:

```bash
# ---------------------------------------------------------
# EJEMPLO DE CONFIGURACIÓN INICIAL PROFESIONAL (UBUNTU)
# ---------------------------------------------------------

# 1. Acceso al Droplet por primera vez (reemplaza con tu IP)
# El sistema te pedirá cambiar la contraseña si usaste Password
ssh root@123.456.78.90

# 2. Actualizar el sistema (Paso obligatorio por seguridad)
# 'apt update' descarga la lista de versiones, 'upgrade' las instala
apt update && apt upgrade -y

# 3. Crear un volumen de almacenamiento extra (Simulación)
# Si añadiste un 'Volume' de 10GB, DigitalOcean lo detecta como un disco nuevo
# Formateamos el nuevo disco (Solo se hace la primera vez)
mkfs.ext4 /dev/sda

# 4. Configurar el Firewall (UFW - Uncomplicated Firewall)
# Permitimos SSH para no quedarnos fuera del servidor
ufw allow ssh
# Permitimos tráfico web estándar
ufw allow http
ufw allow https
# Activamos el firewall
ufw enable

# 5. Instalar un servidor web sencillo (Nginx)
apt install nginx -y

# 6. Verificar que el servidor funciona
# Al poner tu IP en el navegador, deberías ver "Welcome to nginx"
systemctl status nginx

# ---------------------------------------------------------
# CONSEJO PRO: Antes de instalar tu App real, ve al panel
# de DigitalOcean y crea un "Snapshot".
# Si rompes algo, puedes volver a este punto exacto en 2 minutos.
# ---------------------------------------------------------
```

### Resumen de Flujo de Trabajo Profesional

1.  **Crea el Droplet** (Selecciona región y tipo).
2.  **Configura Networking** (Asigna un Firewall en el panel).
3.  **Conecta un Volumen** (Si vas a guardar muchos datos).
4.  **Habilita Backups** (Vital para producción).
5.  **Monitorea Graphs** (Revisa una vez a la semana que el CPU no esté al 100%).
