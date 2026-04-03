¡Excelente continuación! Has tocado el corazón de Docker: **la persistencia de datos y la configuración dinámica**. Un contenedor por naturaleza es efímero (si se borra, sus datos mueren con él), por lo que entender volúmenes y variables de entorno es lo que separa a un novato de un profesional.

A continuación, presento la guía completa integrando los conceptos de la primera parte con estos nuevos pilares avanzados.

---

# 🐳 Guía Maestra de Docker: Volúmenes, Datos y Configuración Profesional

> **Nota de Experto:** En Docker, el código es estático pero los datos son vivos. Esta guía te enseñará a gestionar la información para que, aunque destruyas mil veces tus contenedores, tus bases de datos y configuraciones sigan intactas.

---

## 1. Tipos de Datos en el Ecosistema Docker

En una arquitectura profesional, clasificamos los datos según su ciclo de vida:

1.  **Datos de Imagen (Código y Entorno):** Son de solo lectura. Se definen en el `Dockerfile` (código fuente, librerías instaladas).
2.  **Datos Temporales (Capa de Contenedor):** Datos que el contenedor escribe mientras corre (logs temporales, caché). Si el contenedor se borra, estos datos **desaparecen**.
3.  **Datos Permanentes (Volúmenes):** Datos que deben sobrevivir al contenedor (bases de datos, imágenes subidas por usuarios). Se almacenan fuera de la capa de unión del contenedor.

---

## 2. Volúmenes vs. Bind Mounts

Para persistir datos en el **Host** (tu PC/Servidor), tenemos dos estrategias principales:

| Característica    | **Volumes (Named)**                                                    | **Bind Mounts**                                             |
| :---------------- | :--------------------------------------------------------------------- | :---------------------------------------------------------- |
| **Ubicación**     | Gestionada por Docker (en una carpeta interna del sistema).            | Tú eliges la ruta exacta en tu disco duro.                  |
| **Uso Ideal**     | Persistencia de bases de datos y datos de producción.                  | Desarrollo en tiempo real (Hot Reload).                     |
| **Ventaja**       | Más seguros, fáciles de respaldar y migrar vía CLI.                    | Permite ver cambios en el código sin reconstruir la imagen. |
| **Inconveniente** | No puedes ver los archivos fácilmente desde tu explorador de archivos. | Dependen de la estructura de carpetas de tu PC.             |

---

## 3. Los Volúmenes en Profundidad

### Named Volumes (Volúmenes con Nombre)

Son la forma recomendada de persistir datos. Docker crea una carpeta en su área de gestión y le asigna un nombre.

```bash
# Ejemplo: Crear un contenedor de base de datos con un volumen nombrado
# -v nombre_volumen:ruta_dentro_del_contenedor
docker run -d --name mi-db -v db-data:/var/lib/mysql mysql
```

- **¿Por qué no Anonymous Volumes?** Un volumen anónimo (ej. `-v /app/data`) se crea sin nombre. Cuando borras el contenedor, Docker pierde la referencia fácil a ese volumen, lo que dificulta mucho recuperar los datos o reutilizarlos en otro contenedor.

### Comandos de Gestión de Volúmenes

```bash
docker volume create mi_volumen     # Crear manualmente
docker volume ls                   # Listar todos
docker volume inspect mi_volumen    # Ver ruta real en el host
docker volume rm mi_volumen        # Borrar uno específico
docker volume prune                # ¡Cuidado! Borra todos los volúmenes sin usar
```

---

## 4. Bind Mounts para Desarrollo (El truco del experto)

Si estás desarrollando, quieres que al cambiar tu código en VS Code, el contenedor se actualice al instante.

### El comando "Combo" de desarrollo:

```bash
docker run -d \
  -p 3000:80 \
  --name mi-app-dev \
  -v "$(pwd):/app" \
  -v /app/node_modules \
  --rm \
  mi-imagen-node
```

**Explicación del ejemplo:**

1.  `-v "$(pwd):/app"`: Es el **Bind Mount**. Mapea tu carpeta actual al contenedor. Tu código local "sobrescribe" el del contenedor.
2.  `-v /app/node_modules`: Es un **Volumen Anónimo**. Docker prioriza la ruta más específica. Esto evita que tu carpeta local `node_modules` (que podría ser de Windows/Mac) sobrescriba la del contenedor (Linux), protegiendo las dependencias instaladas dentro de la imagen.

> **Configuración en Docker Desktop:** En Windows/Mac, asegúrate en `Settings > Resources > File Sharing` que la ruta de tu proyecto tenga permiso para ser compartida.

---

## 5. Volúmenes Read-Only (`:ro`)

A veces quieres que el contenedor pueda leer archivos de tu PC pero no modificarlos ni borrarlos.

```bash
# Añadimos :ro al final del mapeo del volumen
docker run -v "$(pwd):/app:ro" mi-app
```

- **Uso:** Evita que un error en el código o un atacante modifique los archivos originales de tu servidor.

---

## 6. Configuración: ARG vs ENV

### ARG (Build-time)

Variables que solo existen mientras se construye la imagen.

```dockerfile
ARG VERSION=1.0
RUN echo "Construyendo versión ${VERSION}"
```

Uso en consola: `docker build --build-arg VERSION=2.0 .`

### ENV (Runtime)

Variables que permanecen dentro del contenedor mientras corre.

```dockerfile
ENV PORT=80
EXPOSE $PORT
```

---

## 7. Manejo de Variables de Entorno (.env)

No es seguro poner contraseñas en el Dockerfile. Usamos archivos `.env`.

### Formas de pasar variables:

1.  **En el comando:** `docker run -e DB_USER=admin mi-app`
2.  **Vía archivo .env:**
    ```bash
    # Suponiendo que tienes un archivo llamado .variables.env
    docker run --env-file ./.variables.env mi-app
    ```

---

## 8. El archivo .dockerignore

Igual que `.gitignore`, evita que archivos innecesarios suban a la imagen, reduciendo su tamaño y mejorando la seguridad.
**Qué incluir:**

- `node_modules` (se deben instalar dentro).
- `.git`
- `Dockerfile`
- Archivos de logs o secretos locales.

---

## 9. Resumen de comandos de limpieza y borrado

Para mantener tu sistema limpio de "basura" de Docker:

```bash
# Borrar un contenedor específico
docker rm nombre_contenedor

# Borrar una imagen específica
docker rmi id_imagen

# Borrar contenedores detenidos automáticamente al parar
docker run --rm mi-imagen

# LIMPIEZA TOTAL (Uso profesional)
docker system prune -a  # Borra TODO lo que no se esté usando (contenedores, imágenes, redes)
```

---

## Ejemplo Final Integrado (Flujo Profesional)

1.  **Creamos la imagen:** `docker build -t mi-web-app:v1 .`
2.  **Subimos a Docker Hub:**
    ```bash
    docker login
    docker tag mi-web-app:v1 miusuario/mi-web-app:v1
    docker push miusuario/mi-web-app:v1
    ```
3.  **Corremos en producción con persistencia:**
    ```bash
    docker run -d \
      --name app-produccion \
      -p 80:3000 \
      -v datos-usuarios:/app/uploads \
      --env-file ./.env.prod \
      --restart always \
      miusuario/mi-web-app:v1
    ```

¿Te gustaría que te genere un archivo **docker-compose.yml** para orquestar varios contenedores (App + Base de Datos) de forma más sencilla?
