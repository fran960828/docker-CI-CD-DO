¡Felicidades! Has llegado a la "joya de la corona" de Docker. Si hasta ahora te parecía tedioso escribir comandos larguísimos en la terminal, **Docker Compose** es la solución definitiva. Es la herramienta que permite definir y correr aplicaciones multi-contenedor de forma elegante y automatizada.

Aquí tienes la guía profesional para dominar el archivo `docker-compose.yaml`.

---

# 🏗️ Guía Maestra: Docker Compose (Orquestación Simplificada)

> **Nota de Arquitecto:** Docker Compose es una herramienta para definir aplicaciones que necesitan más de un contenedor (ej. App + DB). En lugar de ejecutar 10 comandos de `docker run` con redes y volúmenes manuales, escribes un archivo de configuración `.yaml` y levantas todo el sistema con un solo comando: `docker-compose up`.

---

## 1. Estructura y Terminología Base

Un archivo `docker-compose.yaml` sigue una jerarquía estricta basada en indentaciones (espacios).

### Conceptos Clave:

- **version**: Define la versión de la especificación de Compose (actualmente se suele omitir o usar '3.8').
- **services**: Aquí definimos nuestros contenedores. Cada servicio es un contenedor.
- **networks**: Por defecto, Compose crea una red para todos los servicios del archivo, así que no es obligatorio declararla, pero es buena práctica.
- **volumes (Top-level)**: Los _named volumes_ (volúmenes con nombre) deben declararse al final, al mismo nivel que `services`.

### Ejemplo: El servicio de Base de Datos (MongoDB)

```yaml
version: "3.8" # Versión de la sintaxis de Docker Compose

services: # Inicio de la definición de contenedores
  mongodb: # Nombre que elegimos para el servicio (y su dominio en la red)
    image: mongo # Imagen oficial que descargará de Docker Hub
    volumes:
      - data:/data/db # Named volume para persistencia (definido abajo)
    env_file:
      - ./.env # Carga variables desde un archivo externo (seguro)
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin # Variable directa (menos segura)
    # Nota: No necesitamos declarar la red, Compose crea una por defecto
    # llamada 'proyecto_default' donde todos se ven.

volumes: # Declaración obligatoria de los named volumes usados arriba
  data: # El nombre coincide con el del servicio mongodb
```

---

## 2. Construcción Personalizada (El Backend)

Cuando no usamos una imagen oficial (como Mongo), sino nuestro propio código, usamos la instrucción `build`.

### Terminología de Build:

- **context**: La ruta a la carpeta donde está el código (normalmente `./backend`).
- **dockerfile**: El nombre del archivo (si es simplemente `Dockerfile`, se puede omitir).
- **depends_on**: Indica que este contenedor necesita que otro (la DB) esté listo antes de arrancar.

### Ejemplo: Servicio Node.js

```yaml
backend:
  build:
    context: ./backend # Ruta donde está el código y el Dockerfile
    dockerfile: Dockerfile # Nombre del archivo de configuración
  ports:
    - "80:80" # Mapeo de puertos (Host:Contenedor)
  volumes:
    - logs:/app/logs # 1. Named Volume (Persistencia de logs)
    - ./backend:/app # 2. Bind Mount (Sincronización de código en vivo)
    - /app/node_modules # 3. Anonymous Volume (Protege los módulos internos)
  depends_on:
    - mongodb # No arranca hasta que 'mongodb' esté creado
  environment:
    - DB_URL=mongodb://admin:secret@mongodb:27017/db?authSource=admin
```

---

## 3. El Frontend (Interactividad y Sincronización)

El frontend (React/Vue) suele requerir una terminal interactiva para que el servidor de desarrollo no se detenga inmediatamente.

### Configuración Especial:

- **stdin_open (True)**: Mantiene la entrada estándar abierta (equivalente al `-i` de `docker run`).
- **tty (True)**: Simula una terminal real (equivalente al `-t` de `docker run`).

### Ejemplo: Servicio React

```yaml
frontend:
  build:
    context: ./frontend
  ports:
    - "3000:3000"
  volumes:
    - ./frontend:/app # Bind mount para ver cambios en el navegador al editar
    - /app/node_modules # Evita que los módulos del host rompan el contenedor
  stdin_open: true # Permite interacción con el proceso de React
  tty: true # Asigna una terminal virtual
  depends_on:
    - backend # El frontend depende de que el API esté lista
```

---

## 4. El archivo Completo Integrado

Aquí tienes cómo se vería tu infraestructura profesional completa en un solo archivo:

```yaml
version: "3.8"

services:
  mongodb:
    image: mongo
    volumes:
      - data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=secret

  backend:
    build: ./backend # Atajo si el archivo se llama Dockerfile
    ports:
      - "80:80"
    volumes:
      - logs:/app/logs
      - ./backend:/app
      - /app/node_modules
    depends_on:
      - mongodb
    env_file:
      - ./.env

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    stdin_open: true
    tty: true
    depends_on:
      - backend

volumes: # Declaración de volúmenes con nombre
  data:
  logs:
```

---

## 5. Comandos Esenciales de Compose

Una vez tienes tu archivo `docker-compose.yaml` listo, estos son los comandos que usarás a diario:

| Comando                  | Acción                                                                |
| :----------------------- | :-------------------------------------------------------------------- |
| `docker-compose up`      | Crea la red, volúmenes y arranca todos los contenedores.              |
| `docker-compose up -d`   | Arranca todo en segundo plano (Detached mode).                        |
| `docker-compose down`    | Detiene y **borra** los contenedores y redes (pero no los volúmenes). |
| `docker-compose down -v` | Detiene todo y **borra también los volúmenes** (limpieza total).      |
| `docker-compose build`   | Re-construye las imágenes si has hecho cambios en los Dockerfiles.    |

---

### ¿Cuál es el siguiente paso?

Has dominado el flujo completo: desde una imagen simple hasta una arquitectura multi-contenedor persistente y reactiva.

**¿Te gustaría que te explicara cómo llevar esto a un entorno de producción real, o quizás prefieres aprender sobre los "Multi-stage Builds" para hacer que tus imágenes pesen 10 veces menos?**
