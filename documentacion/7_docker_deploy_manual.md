# Guía Profesional: Despliegue de Aplicaciones con Docker en AWS EC2

> **Nota para principiantes:** Esta documentación detalla la transición de un entorno local (donde priorizamos la comodidad) a un entorno de producción (donde priorizamos la seguridad, estabilidad y escalabilidad). Aprenderás no solo a "hacer que funcione", sino a entender por qué ciertos métodos de desarrollo son peligrosos en la nube y cómo usar AWS para tener el control total de tu infraestructura.

---

## 1. Conceptos Fundamentales: Desarrollo vs. Producción

En desarrollo, usamos **Bind Mounts** para ver cambios en tiempo real, pero en producción esto es un error crítico. La imagen debe ser la **"única fuente de la verdad"**.

- **Inmutabilidad:** Usamos la instrucción `COPY` en el Dockerfile para tomar un "snapshot" del código. Así, la imagen contiene todo lo necesario y no depende de archivos externos del servidor host.
- **Etapas de construcción (Multi-stage):** Apps como React necesitan compilarse. No enviamos el código fuente, sino el resultado del "build" para que la imagen sea ligera.
- **Escalabilidad:** En local, un solo `docker-compose` levanta todo. En producción profesional, la base de datos suele ir en un host y la app en otro para evitar que una caída de uno afecte al otro.
- **Control vs. Responsabilidad:** Al usar una instancia propia (EC2), tienes control total, pero tú eres responsable de la seguridad y los parches. Si prefieres menos responsabilidad, podrías contratar servicios "Serverless" (como AWS Fargate), pero pierdes control técnico.

---

## 2. Infraestructura en la Nube: AWS EC2

Aunque existen hostings como DigitalOcean, Azure o Google Cloud, **AWS EC2 (Elastic Compute Cloud)** es el estándar. Te permite alquilar servidores virtuales donde tienes acceso total como administrador (root).

### Pasos para el despliegue en AWS EC2

#### A. Inicio y Lanzamiento de Instancia

1.  Inicia sesión en la consola de AWS.
2.  En la barra de búsqueda, escribe **EC2**.
3.  Haz clic en **Launch Instance**.
4.  **Configuración:** Selecciona un nombre para tu servidor y elige la Amazon Machine Image (AMI) de **Ubuntu** (muy recomendada por su compatibilidad con Docker).
5.  Selecciona el tipo de instancia `t2.micro` (disponible en la **capa gratuita**).

#### B. Redes y Seguridad (VPC y Security Groups)

AWS crea por defecto una **VPC** (nube privada virtual). Lo importante aquí es el **Security Group** (tu firewall):

- Por defecto, solo permite el puerto 22 (SSH).
- Para que el mundo vea tu app, debes configurar las **Inbound Rules**: añade una regla para permitir tráfico HTTP (puerto 80) o el puerto que use tu contenedor (ej. 8080) desde la dirección `0.0.0.0/0` (Cualquier lugar).

#### C. Key Pair (La llave de entrada)

Antes de lanzar, crea un **Key Pair**:

1.  Asigna un nombre (ej. `mi-llave-aws`).
2.  Descarga el archivo `.pem`. **¡No lo pierdas!** Sin él no podrás entrar al servidor.
3.  Haz clic en **Launch Instance**.

---

## 3. Conexión y Configuración del Servidor

Para conectar, usa tu terminal (Linux/Mac) o PowerShell/PuTTY (Windows):

```bash
# 1. Cambia los permisos de la llave para que sea privada (Solo Linux/Mac)
chmod 400 mi-llave-aws.pem

# 2. Conéctate vía SSH usando el DNS público que te da AWS
# ssh -i "llave.pem" usuario@ip-publica
ssh -i "mi-llave-aws.pem" ubuntu@ec2-18-200-10-1.compute-1.amazonaws.com
```

### Instalación de Docker en la instancia

Una vez dentro del servidor de AWS, ejecuta:

```bash
# Actualizar los paquetes del sistema
sudo apt-get update

# Instalar Docker
sudo apt-get install docker.io -y

# Iniciar el servicio de Docker
sudo systemctl start docker
sudo systemctl enable docker

# Dar permisos a tu usuario para usar docker sin 'sudo'
sudo usermod -aG docker ${USER}
# (Debes cerrar sesión y volver a entrar para que este cambio surta efecto)
```

---

## 4. Estrategias de Despliegue de la Aplicación

Existen dos formas principales de llevar tu código al servidor:

1.  **Directo (Push de código):** Subes tu código fuente al servidor vía Git y haces `docker build` allí mismo. Es más lento y consume recursos del servidor remoto.
2.  **Docker Hub (Recomendado):** Construyes la imagen en tu PC local, la subes a la "nube de imágenes" (Docker Hub) y en el servidor remoto solo haces un `pull`.

---

## Ejemplo Práctico: Despliegue Profesional

### Paso 1: El Dockerfile Inmutable (Local)

Este archivo asegura que la imagen es la única fuente de verdad y usa etapas para React.

```dockerfile
# ETAPA 1: Construcción (Build)
FROM node:18-alpine AS build-stage
WORKDIR /app
# Copiamos archivos de dependencias
COPY package*.json ./
RUN npm install
# Capturamos el snapshot del código (Inmutabilidad)
COPY . .
RUN npm run build

# ETAPA 2: Producción (Servidor ligero)
FROM nginx:stable-alpine
# Copiamos solo el resultado de la etapa anterior (Eficiencia)
COPY --from=build-stage /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Paso 2: Flujo de Actualización (Ciclo de Vida)

Si haces un cambio en tu código, el proceso profesional es:

```bash
# 1. Construir nueva versión en LOCAL
docker build -t tu-usuario/mi-app:v2 .

# 2. Subir a Docker Hub
docker push tu-usuario/mi-app:v2

# 3. En el servidor AWS (vía SSH)
docker pull tu-usuario/mi-app:v2
docker stop mi-contenedor-viejo
docker rm mi-contenedor-viejo
# Correr el nuevo contenedor
docker run -d --name mi-app-v2 -p 80:80 tu-usuario/mi-app:v2
```

---

## 5. Advertencia Final: El Riesgo del Control Total

Desplegar en AWS EC2 instalando Docker manualmente te da **control total**, pero recuerda el equilibrio:

> **Riesgos:** Si no configuras bien las reglas de entrada (Security Groups), dejas puertos abiertos a ataques. Si no actualizas el sistema operativo de la instancia, podrías sufrir brechas de seguridad. Además, si el tráfico sube de golpe, una sola instancia de EC2 podría colapsar si no configuras un "Auto Scaling Group".

**Conclusión:** Este método es excelente para aprender y para proyectos donde necesitas optimizar cada céntimo y recurso técnico, siempre que asumas la responsabilidad de mantener la "salud" del servidor.
