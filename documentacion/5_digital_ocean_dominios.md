Esta documentación final cierra el ciclo de despliegue profesional. Aquí entenderás cómo la identidad digital (dominio), la infraestructura (hosting) y la comunicación (correo) se entrelazan para formar un proyecto completo y serio.

---

# 🌐 Identidad Digital: Dominio, Hosting, DNS y Correo Profesional

> **Nota del Experto:** Muchos principiantes cometen el error de pensar que comprar un dominio incluye automáticamente el correo o la web. En el entorno profesional de DigitalOcean, estos son servicios que solemos separar para obtener mayor escalabilidad y seguridad.

---

## 1. La Trinidad de la Web: Dominio, Hosting y Correo

Para entender estos conceptos, usemos la analogía de una **oficina física**:

- **Dominio (La Dirección):** Es el nombre único que la gente escribe para encontrarte (ej: `tuempresa.com`). Es equivalente a la dirección postal "Calle Falsa 123". No es el edificio, es solo el indicador de dónde está.
- **Hosting / Droplet (El Edificio):** Es el espacio físico (servidor) donde guardas tus archivos, bases de datos y código. Es el local que alquilas en DigitalOcean para que tu web "viva" allí.
- **Correo (El Buzón):** Es el servicio que gestiona la correspondencia. Aunque el hosting puede enviar correos, a nivel profesional se usan plataformas externas (como Google Workspace o Zoho) para asegurar que tus correos no lleguen a la carpeta de SPAM.

---

## 2. El Dominio y la Zona DNS (El Traductor)

El **DNS (Domain Name System)** es lo que conecta tu dominio con tu Droplet. Los servidores no entienden nombres como `google.com`, solo entienden IPs (como `157.245.120.80`).

### ¿Para qué sirve la Zona DNS?

Es una tabla de registros que le dice al mundo:

1.  **Registro A:** "Si alguien busca mi web, mándalo a esta IP de DigitalOcean".
2.  **Registro MX:** "Si alguien me envía un email, mándalo a estos servidores de correo".
3.  **Registro TXT:** "Usa esto para verificar que soy el dueño del dominio".

### Cómo vincular un Dominio a un Virtual Host (Ejemplo Profesional)

Imagina que ya configuraste tu Virtual Host en el paso anterior (Apache o Nginx). Ahora falta que el dominio "llame" a la puerta de ese servidor.

**Paso 1: Configuración en el Registrador (DonDominio/GoDaddy/DigitalOcean)**
Debes añadir un registro tipo **A** en la zona DNS.

```text
Tipo: A
Nombre: @ (o deja vacío)
Valor: 157.245.120.80 (La IP de tu Droplet)
TTL: 3600
```

**Paso 2: Asegurar que el Virtual Host responde a ese nombre**
En tu Droplet, el archivo de configuración debe tener el `ServerName` correcto.

```apache
# Ejemplo en Apache (/etc/apache2/sites-available/misitio.conf)
<VirtualHost *:80>
    # Este nombre DEBE coincidir con el registro DNS que creaste
    ServerName tusitio.com
    ServerAlias www.tusitio.com

    DocumentRoot /var/www/misitio/public_html
    # ... resto de la configuración
</VirtualHost>
```

---

## 3. Empleo de una Plataforma de Correo Profesional

**Nunca uses tu propio Droplet para gestionar correos importantes.** ¿Por qué? Porque si tu servidor es marcado como SPAM, tus correos dejarán de llegar. Lo profesional es usar servicios como **Zoho Mail (Gratis/Pago)** o **Google Workspace**.

### Cómo configurar el correo mediante DNS (Registros MX)

Para que tu dominio reciba correos, debes ir a tu Zona DNS y configurar los registros **MX (Mail Exchange)**.

**Ejemplo de configuración para Zoho Mail:**

```bash
# 1. Accedes al panel de DNS de DigitalOcean o tu registrador.
# 2. Borras cualquier registro MX existente.
# 3. Añades los nuevos registros (estos te los da el proveedor de correo):

Prioridad: 10  |  Valor: mx.zoho.eu
Prioridad: 20  |  Valor: mx2.zoho.eu
Prioridad: 50  |  Valor: mx3.zoho.eu

# 4. Añades un registro TXT para el SPF (Evita que otros suplanten tu identidad)
# Este registro dice: "Solo los servidores de Zoho tienen permiso para enviar correos de @tusitio.com"
Tipo: TXT
Valor: v=spf1 include:zoho.eu ~all
```

---

## Ejemplo de Flujo Completo (Checklist Profesional)

Si estás lanzando un sitio hoy, este es el orden que yo, como experto, te recomiendo seguir:

1.  **Compra el Dominio:** En un registrador (ej: DonDominio).
2.  **Crea el Droplet:** En DigitalOcean.
3.  **Apunta el DNS:** Crea un registro **A** en la zona DNS que apunte a la IP del Droplet.
4.  **Configura el Virtual Host:** En el servidor (Ubuntu + Nginx/Apache), asegúrate de que el `ServerName` sea el dominio comprado.
5.  **Verifica con DNS Checker:** Asegúrate de que el dominio ya resuelve a tu IP.
6.  **Configura el Correo:** Crea una cuenta en Zoho/Google y añade los registros **MX** en la misma zona DNS donde pusiste el registro A.
7.  **Instala SSL (Opcional pero Obligatorio):** Usa `Certbot` para que tu sitio tenga el candado verde (`https`).

```bash
# Comando rápido para instalar SSL una vez el dominio apunte a tu IP
sudo apt install certbot python3-certbot-apache
sudo certbot --apache -d tusitio.com -d www.tusitio.com
# ¡Y listo! Tu dominio, hosting y seguridad están vinculados.
```

¡Felicidades! Con estas cuatro guías, has pasado de no saber qué es un VPS a entender cómo se gestiona una infraestructura profesional completa.
