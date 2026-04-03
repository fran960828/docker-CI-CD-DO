# Guía de Despliegue Frontend: React con Docker Multi-stage y AWS ECS

> **Nota para principiantes:** En el desarrollo de aplicaciones Frontend modernas (React, Vue, Angular), existe una gran diferencia entre cómo ejecutamos la app en nuestra PC y cómo se sirve en internet. No "subimos" el código fuente; subimos el resultado de una transformación técnica que convierte miles de archivos complejos en unos pocos archivos estáticos ultrarrápidos. Esta guía te enseñará a automatizar ese proceso usando Docker de nivel profesional.

---

## 1. ¿Por qué React necesita una etapa de construcción (Build)?

Cuando programas en React, usas **JSX**, **TypeScript** y cientos de librerías en la carpeta `node_modules`. Sin embargo, los navegadores (Chrome, Firefox) solo entienden **HTML, CSS y JavaScript estándar**.

- **Minificación:** Se eliminan espacios y comentarios para que los archivos pesen menos.
- **Transpilación:** Se traduce el código moderno a versiones compatibles con navegadores antiguos.
- **Bundling:** Se agrupan cientos de archivos en uno solo para reducir las peticiones al servidor.
- **Seguridad:** No queremos enviar nuestro código fuente original al cliente, solo el "paquete" optimizado para funcionar.

---

## 2. Dockerfile Multi-stage: La Eficiencia es la Clave

Un Dockerfile "Multi-stage" nos permite usar una imagen pesada (Node.js) para construir la app y luego desecharla, quedándonos solo con una imagen ligera (Nginx) para servir los archivos.

### Ejemplo de `Dockerfile.prod`

```dockerfile
# ETAPA 1: Construcción (Build)
# Usamos una imagen de Node para tener acceso a 'npm'
FROM node:18-alpine AS build-stage

WORKDIR /app

# Copiamos solo los archivos de dependencias primero (mejor uso de caché)
COPY package*.json ./
RUN npm install

# Copiamos el resto del código y generamos la carpeta /dist o /build
COPY . .
RUN npm run build

# ETAPA 2: Producción (Servidor)
# Usamos Nginx porque es el estándar de oro para servir archivos estáticos rápido
FROM nginx:stable-alpine

# Copiamos el resultado de la etapa anterior a la carpeta que Nginx usa por defecto
# Nota: La carpeta de origen suele ser /app/dist o /app/build según tu config
COPY --from=build-stage /app/dist /usr/share/nginx/html

# Exponemos el puerto 80 (estándar web)
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

---

## 3. Construcción y Nomenclatura para Docker Hub

En producción, es vital que tu imagen local tenga el mismo nombre que tu repositorio remoto para que el comando `push` sepa a dónde ir.

```bash
# Construimos indicando explícitamente el archivo de producción
# El nombre debe seguir el patrón: usuario_dockerhub/nombre_repositorio
docker build -f Dockerfile.prod -t tu-usuario/mi-frontend-react:v1 .

# Iniciamos sesión y subimos la imagen al registro
docker login
docker push tu-usuario/mi-frontend-react:v1
```

---

## 4. Despliegue en AWS ECS: Conectando Frontend y Backend

Para que el Frontend funcione, antes de construir la imagen, debemos asegurarnos de que el código de React conozca la dirección del Backend (el DNS del Load Balancer que creaste para Django).

### A. El DNS del Backend

En tu código de React (ej. `.env.production`), debes colocar la URL pública del Load Balancer de tu Backend:
`VITE_API_URL=http://lb-backend-12345.us-east-1.elb.amazonaws.com`

### B. Configuración en la Consola de ECS

1.  **Nueva Task Definition:** Crea una tarea específica para el Frontend.
    - **Imagen:** `tu-usuario/mi-frontend-react:v1`.
    - **Memoria/CPU:** Al ser solo archivos estáticos, puedes usar lo mínimo (0.25 vCPU / 0.5 GB RAM).
2.  **Nuevo Load Balancer (ALB):**
    - Crea un ALB exclusivo para el Frontend (o usa uno existente con diferentes reglas).
    - Este ALB te dará la URL final que verán tus usuarios (ej: `http://mi-app-frontend.aws.com`).
3.  **Nuevo Servicio:** Crea el servicio dentro de tu cluster y selecciona la Task del Frontend y el Load Balancer correspondiente.

---

## 5. Resumen del Flujo de Trabajo Profesional

| Paso               | Acción                                       | Propósito                                    |
| :----------------- | :------------------------------------------- | :------------------------------------------- |
| **1. Config**      | Poner el DNS del Backend en el código React. | Que el Front sepa a quién pedirle los datos. |
| **2. Build**       | `docker build -f Dockerfile.prod`.           | Crear la versión optimizada y ligera.        |
| **3. Registry**    | `docker push` a Docker Hub.                  | Tener la imagen disponible para AWS.         |
| **4. ECS Task**    | Definir el contenedor en AWS.                | Decirle a AWS qué imagen usar.               |
| **5. ECS Service** | Lanzar y conectar con Load Balancer.         | Exponer la web al mundo de forma estable.    |

> **Recordatorio de Seguridad:** Asegúrate de que el **Security Group** de tu Frontend permita tráfico en el puerto 80 desde `0.0.0.0/0` (Todo el mundo), y que el Load Balancer del Backend permita tráfico desde el Security Group del Frontend.

Al usar este método, has separado totalmente las preocupaciones: Node.js solo trabaja un momento para construir, Nginx se encarga de servir, y el Load Balancer se encarga de que la web nunca se caiga.
