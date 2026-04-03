¡Hola! Qué gusto saludarte. Como experto en Docker, me encanta ver a gente sumergiéndose en el mundo de la contenerización. Docker no es solo una herramienta, es un cambio de paradigma que elimina el clásico "en mi máquina funciona".

Aquí tienes una guía profesional, estructurada y clara para dominar los fundamentos desde cero.

---

# 🐳 Guía Profesional de Docker: De Cero a Image Master

> **Nota para principiantes:** Docker es una plataforma que permite empaquetar una aplicación y todas sus dependencias (librerías, sistema operativo, configuración) en una unidad estandarizada llamada **contenedor**. Imagina que es un "paquete autoejecutable" que garantiza que el software corra exactamente igual en tu laptop, en el servidor de un cliente o en la nube.

---

## 1. Imágenes vs. Contenedores

Para entender Docker, hay que entender esta dualidad:

- **Imagen:** Es una plantilla de **solo lectura**. Contiene el código, las librerías y el entorno. Es como una "receta" de cocina o el instalador de un programa.
- **Contenedor:** Es una **instancia ejecutable** de una imagen. Es la receta ya cocinada. Puedes crear muchos contenedores (platos) a partir de una sola imagen (receta).

---

## 2. Primeros pasos con Node.js

Docker te permite usar tecnologías sin instalarlas directamente en tu sistema.

### Crear un contenedor básico

```bash
# Descarga la imagen de node y crea un contenedor que ejecuta node
docker run node

# Listar todos los contenedores (los que están corriendo y los que no)
# Verás que el de node aparece con status 'Exited' porque terminó su tarea
docker ps -a
```

### Modo Interactivo

Si quieres entrar a la consola de Node para escribir código:

```bash
# -i: interactivo, -t: terminal (TTY)
docker run -it node
# Esto te abrirá el REPL de Node directamente en tu terminal.
```

---

## 3. Anatomía de una Custom Image (Dockerfile)

Cuando quieres "contenerizar" **tu propia** aplicación, creas un archivo llamado `Dockerfile`.

### Conceptos clave del Dockerfile:

- **FROM:** La imagen base (ej. `node:18`).
- **WORKDIR:** El directorio de trabajo dentro del contenedor (como un `cd`).
- **COPY:** Copia archivos de tu PC al contenedor.
- **RUN:** Ejecuta comandos durante la construcción (ej. `npm install`).
- **CMD:** El comando que se ejecuta **solo cuando el contenedor inicia**.
- **EXPOSE:** Documenta en qué puerto escucha la app.

### Ejemplo de creación:

Imagina que tienes una app sencilla en Node.

```dockerfile
# 1. Usar una imagen base de Node
FROM node:18

# 2. Definir dónde vivirá la app dentro del contenedor
WORKDIR /app

# 3. Copiar el archivo de dependencias primero (optimiza capas)
COPY package.json .

# 4. Instalar dependencias
RUN npm install

# 5. Copiar el resto del código
COPY . .

# 6. Informar que la app usa el puerto 80
EXPOSE 80

# 7. Comando para arrancar la app
CMD ["node", "app.js"]
```

---

## 4. Construcción y Puertos

### Construir la imagen

```bash
# El punto '.' indica que el Dockerfile está en la carpeta actual
docker build .

# Construir con un nombre y etiqueta (tag) personalizado
docker build -t mi-app-node:1.0 .
```

### Ejecutar con mapeo de puertos

Un contenedor es una isla aislada. Para acceder a él desde tu navegador, usamos el flag `-p`.

```bash
# Mapea el puerto 3000 de tu PC al 80 del contenedor
docker run -p 3000:80 <imageId_o_nombre>

# Detener el contenedor
docker stop <containerName>
```

---

## 5. Capas de Imagen y Read-Only

Las imágenes de Docker son **inmutables (Read-Only)**. Se componen de **Layers (capas)**. Cada comando en el Dockerfile (`FROM`, `RUN`, `COPY`) crea una capa nueva.

- Si cambias una línea al final del Dockerfile, Docker reutiliza las capas anteriores (caché), lo que hace que las builds sean rapidísimas.
- Cuando un contenedor corre, Docker añade una pequeña "capa de escritura" temporal encima de la imagen de solo lectura.

---

## 6. Gestión de Imágenes y Contenedores

### Comandos de Imágenes

| Comando                        | Descripción                                      |
| :----------------------------- | :----------------------------------------------- |
| `docker images`                | Lista todas las imágenes en tu PC.               |
| `docker tag image:v1 image:v2` | Crea un alias/etiqueta para una imagen.          |
| `docker image inspect <id>`    | Muestra detalles técnicos (JSON) de la imagen.   |
| `docker rmi <id>`              | Borra una imagen específica.                     |
| `docker image prune`           | Borra todas las imágenes que no se están usando. |

### Comandos de Contenedores

| Comando               | Descripción                                                   |
| :-------------------- | :------------------------------------------------------------ |
| `docker ps`           | Muestra contenedores activos.                                 |
| `docker ps -a`        | Muestra todos (activos e inactivos).                          |
| `docker --help`       | Tu mejor amigo: manual de ayuda.                              |
| `docker rm <name>`    | Borra un contenedor (debe estar detenido).                    |
| `docker inspect <id>` | Ver configuración detallada del contenedor (IPs, rutas, etc). |

---

## 7. Ciclo de Vida y Modos de Ejecución

### Run vs. Start

- **`docker run`**: Crea un contenedor nuevo a partir de una imagen y lo arranca.
- **`docker start <name>`**: Arranca un contenedor que ya existía pero estaba apagado.

### Attached vs. Detached

- **Attached (por defecto en `run`)**: Ves los logs en tiempo real en tu terminal. Si cierras la terminal, el proceso suele detenerse.
- **Detached (`-d`)**: El contenedor corre en segundo plano.
  - `docker start` por defecto es detached. Para verlo: `docker start -a <name>`.
  - Para ver logs de un contenedor en segundo plano: `docker logs -f <name>`.

---

## 8. Trucos Profesionales de Limpieza y Copia

### Limpieza automática

Si solo quieres probar algo y no quieres que el contenedor ocupe espacio al terminar:

```bash
# --rm borra el contenedor automáticamente al detenerse
docker run --rm node
```

### Copiar archivos (docker cp)

Puedes mover archivos entre tu PC y un contenedor en ejecución:

```bash
# De Local a Contenedor
docker cp archivo.txt mi_contenedor:/app/archivo.txt

# De Contenedor a Local
docker cp mi_contenedor:/app/logs.txt ./logs.txt
```

---

## 9. Compartir en Docker Hub

Docker Hub es el "GitHub" de las imágenes.

1.  **Login:** `docker login` (introduce usuario y contraseña).
2.  **Crear Repositorio:** En la web de Docker Hub, crea un repo llamado `mi-usuario/mi-app`.
3.  **Tagging:** Tu imagen local debe llamarse igual que el repo.
    ```bash
    # Renombrar/Etiquetar
    docker tag mi-app-local:latest mi-usuario/mi-app:v1
    ```
4.  **Push:** Enviar a la nube.
    ```bash
    docker push mi-usuario/mi-app:v1
    ```
5.  **Pull:** Descargar en cualquier otra máquina.
    ```bash
    docker pull mi-usuario/mi-app:v1
    ```

---

¿Te gustaría que profundizáramos en cómo gestionar bases de datos con Docker utilizando **Volúmenes** para no perder la información al borrar el contenedor?
