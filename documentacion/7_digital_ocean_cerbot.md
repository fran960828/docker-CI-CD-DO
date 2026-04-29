Esta es la guía final para securizar tu infraestructura. En el mundo profesional, un sitio sin **SSL (HTTPS)** no solo es inseguro, sino que es penalizado por buscadores y navegadores, mostrando advertencias de "Sitio no seguro".

> **Nota del Experto:** **Certbot** es la herramienta estándar de la industria que interactúa con **Let's Encrypt** para emitir certificados gratuitos. Lo configuraremos usando **Snap**, que es el método recomendado oficialmente por los desarrolladores de Certbot para asegurar que siempre tengas la versión más actualizada y con los parches de seguridad al día.

---

# 🛡️ Guía Profesional: Instalación de SSL con Certbot y Nginx

Para que estos pasos funcionen, **tu dominio ya debe estar apuntando a la IP de tu Droplet** (Registro A en DNS) y tu configuración de Nginx debe tener el `server_name` correctamente definido.

---

## 1. Preparación del Entorno: Snapd

**Snapd** es un gestor de paquetes (similar a `apt`) que permite instalar aplicaciones "empaquetadas" con todas sus dependencias. Es el motor que usaremos para correr Certbot.

```bash
# 1. Actualizamos los repositorios del sistema
sudo apt update

# 2. Instalamos snapd
# Ubuntu suele traerlo por defecto, pero este comando asegura que esté presente y activo
sudo apt install snapd -y

# 3. Aseguramos que el núcleo de snap esté actualizado a la última versión
sudo snap install core; sudo snap refresh core
```

---

## 2. Limpieza de Instalaciones Antiguas

Es muy común que Ubuntu traiga una versión obsoleta de Certbot instalada vía `apt`. Para evitar conflictos de software y errores en la emisión del certificado, debemos eliminarla.

```bash
# Eliminamos cualquier rastro de certbot instalado previamente por el gestor apt
# Esto no borrará tus certificados si ya tenías alguno, solo el programa antiguo
sudo apt remove certbot -y
```

---

## 3. Instalación de Certbot y Enlace Simbólico

Instalaremos Certbot mediante Snap para garantizar la compatibilidad total con los protocolos de renovación automática.

```bash
# 1. Instalamos Certbot usando el comando snap
sudo snap install --classic certbot

# 2. Creamos un enlace simbólico (Symlink)
# Esto permite que el sistema reconozca el comando 'certbot' desde cualquier carpeta
# vinculando la ruta de instalación de snap con la ruta de ejecución estándar del sistema
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

---

## 4. Obtención y Configuración del Certificado

Aquí es donde ocurre la "magia". Certbot leerá tu configuración de Nginx, validará que el dominio te pertenece y modificará los archivos de configuración para activar el HTTPS automáticamente.

```bash
# Ejecutamos el plugin de certbot para nginx
# --nginx indica que queremos que modifique automáticamente nuestros VirtualHosts
sudo certbot --nginx
```

### ¿Qué pasará durante la ejecución?

Certbot te hará las siguientes preguntas:

1.  **Email:** Para notificarte si un certificado está por caducar (importante por seguridad).
2.  **Términos de servicio:** Debes aceptar con una `A`.
3.  **Newsletter:** Opcional (puedes decir que no con `N`).
4.  **Selección de Dominio:** Te mostrará una lista de dominios encontrados en tus archivos de Nginx. **Pulsa Enter** para seleccionarlos todos o escribe los números separados por comas.

**Resultado:** Certbot generará las llaves criptográficas, las instalará y reiniciará Nginx por ti. ¡Tu sitio ya tendrá el candado verde!

---

## 5. La Renovación Automática (Ciclo de 3 Meses)

Los certificados de Let's Encrypt son gratuitos pero duran **90 días**. La ventaja de usar el método Snap es que Certbot crea un "Timer" (una tarea programada) en el sistema que se ejecuta dos veces al día.

- **¿Cómo funciona?** El sistema revisa si a tu certificado le faltan menos de 30 días para expirar. Si es así, se conecta a Let's Encrypt, lo renueva y reinicia Nginx de forma totalmente invisible para ti.

### Comprobación Profesional

Como experto, nunca asumas que la renovación funciona; compruébalo manualmente una vez:

```bash
# Simulamos un proceso de renovación (Dry Run)
# Esto hace todo el proceso pero sin descargar un certificado real
# Si este comando termina sin errores (SUCCESS), la renovación automática está garantizada
sudo certbot renew --dry-run
```

---

## Resumen de Comandos Rápidos

| Acción                | Comando                               |
| :-------------------- | :------------------------------------ |
| **Instalar**          | `sudo snap install --classic certbot` |
| **Obtener SSL**       | `sudo certbot --nginx`                |
| **Forzar Renovación** | `sudo certbot renew`                  |
| **Ver Certificados**  | `sudo certbot certificates`           |

Con este último paso, tu infraestructura en **DigitalOcean** ha pasado de ser un simple servidor a una plataforma de despliegue profesional, segura y lista para producción. ¡Excelente trabajo!
