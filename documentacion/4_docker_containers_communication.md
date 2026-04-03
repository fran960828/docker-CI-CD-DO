Esta guía representa el "siguiente nivel" en tu formación como experto en Docker. Aquí aprenderás a orquestar una arquitectura **MERN (Mongo, Express, React, Node)** completa, aplicando persistencia, redes seguras y flujos de desarrollo en tiempo real.

---

# 🚀 Orquestación de Stack Completo con Docker

> **Nota de Arquitecto:** En un entorno profesional, no solo corremos contenedores aislados; creamos un **ecosistema**. La clave aquí es la **Sincronización**: que el código cambie en vivo (Bind Mounts), que los datos no se borren (Named Volumes) y que los servicios se descubran entre sí (Networking).

---

## 1. La Red (Networking)

Para que nuestro Frontend, Backend y Base de Datos se hablen, necesitan un lenguaje común y un cable virtual que los una.

```bash
# Creamos una red tipo bridge para que los 3 contenedores se vean por su NOMBRE
docker network create mern-network
```

---

## 2. Persistencia y Seguridad: MongoDB

Un contenedor de base de datos sin volumen es un peligro (pierdes los datos al apagarlo). Además, en producción, siempre usamos credenciales.

### Conceptos aplicados:

- **Named Volume:** `mongo-data` para que los datos vivan en el host.
- **Variables de Entorno:** `MONGO_INITDB_ROOT_USERNAME` y `PASSWORD`.

```bash
# Creamos el contenedor de Mongo
docker run -d \
  --name mongodb \
  --network mern-network \
  -v mongo-data:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=secret \
  mongo
```

---

## 3. Backend: Node.js con Desarrollo en Vivo

El Backend necesita conectarse a Mongo usando las credenciales anteriores. La URL de conexión profesional sería:
`mongodb://admin:secret@mongodb:27017/course-goals?authSource=admin`

### El Dockerfile de Node (Optimizado)

```dockerfile
FROM node:18
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 80
# Usamos nodemon para que el contenedor reinicie el proceso al detectar cambios en el código
CMD ["npm", "run", "dev"]
```

### Ejecución con Triple Volumen (Patrón Pro)

```bash
docker run -d \
  --name backend \
  --network mern-network \
  -v backend-logs:/app/logs \
  -v "$(pwd):/app" \
  -v /app/node_modules \
  -e MONGO_URL=mongodb://admin:secret@mongodb:27017/db?authSource=admin \
  --rm \
  backend-image
```

- **`backend-logs`**: Named volume para guardar logs.
- **`$(pwd):/app`**: Bind mount para que tus cambios en VS Code lleguen al contenedor.
- **`/app/node_modules`**: Anonymous volume para que Docker use sus propios módulos de Linux y no los de tu PC.

---

## 4. Frontend: React y el Browser

React es especial: aunque vive en un contenedor, se ejecuta en el **navegador del usuario**. Por eso, no puede usar nombres de contenedores para hablar con el backend; debe usar la IP de tu máquina o `localhost` porque el browser está fuera de la red interna de Docker.

### El Dockerfile de React

```dockerfile
FROM node:18
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

### Ejecución con Bind Mount

```bash
docker run -d \
  --name frontend \
  --network mern-network \
  -p 3000:3000 \
  -v "$(pwd):/app" \
  -v /app/node_modules \
  --rm \
  frontend-image
```

- **Nota sobre el puerto:** Aquí el puerto es vital porque tú accederás vía `http://localhost:3000`.

---

## 5. El archivo .dockerignore

Es fundamental para no "ensuciar" la imagen y hacer que el `docker build` sea instantáneo.

**Crea un archivo llamado `.dockerignore` en la raíz de tus proyectos:**

```text
node_modules
access.log
Dockerfile
.git
.env
```

- **¿Por qué?** Si copias `node_modules` de tu Windows a un contenedor Linux, lo más probable es que la aplicación falle por incompatibilidad de binarios.

---

## 6. Resumen de Flujo de Trabajo con Nodemon

En tus archivos `package.json`, asegúrate de tener configurado nodemon:

```json
"scripts": {
  "dev": "nodemon app.js"
}
```

**Ventaja:** Gracias al **Bind Mount** (`-v $(pwd):/app`), cuando guardas un archivo en tu editor:

1.  El archivo se actualiza instantáneamente dentro del contenedor.
2.  **Nodemon** detecta el cambio dentro del contenedor.
3.  Reinicia el servidor automáticamente sin que tengas que hacer `docker stop` y `docker run`.

---

## 7. Tabla Comparativa de Volúmenes en este Ejemplo

| Tipo             | Sintaxis              | Propósito                                              |
| :--------------- | :-------------------- | :----------------------------------------------------- |
| **Named Volume** | `mongo-data:/data/db` | Persistir la DB a largo plazo.                         |
| **Bind Mount**   | `$(pwd):/app`         | Sincronizar código fuente en tiempo real.              |
| **Anonymous**    | `/app/node_modules`   | Evitar conflictos de librerías entre Host y Container. |

---

### ¿Cuál es el siguiente paso?

Ahora que sabes conectar todo manualmente, te darás cuenta de que escribir estos comandos de 10 líneas en la consola es tedioso.

**¿Te gustaría que te enseñe a convertir todo este despliegue en un único archivo `docker-compose.yml` para levantarlo con un solo comando (`docker-compose up`)?**
