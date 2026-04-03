¡Hola de nuevo! Como experto en Docker, entramos en la fase de **optimización y herramientas de utilidad**. Hasta ahora hemos construido aplicaciones completas, pero a veces Docker se usa simplemente como una "herramienta de bolsillo" o para servir contenido de forma ultra eficiente.

Aquí tienes la documentación detallada sobre Utility Containers, comandos avanzados de Dockerfile y el servidor Nginx.

---

# 🛠️ Guía Avanzada: Utility Containers, Nginx y Volúmenes Optimizados

> **Nota de Experto:** En un flujo de trabajo profesional, no siempre queremos que un contenedor "viva" para siempre. A veces solo queremos que entre, ejecute un comando (como instalar una librería) y muera. Además, aprenderemos a separar la lógica de nuestra app del servidor que entrega los archivos (Nginx).

---

## 1. Utility Containers (Contenedores de Utilidad)

Un **Utility Container** es un contenedor que no aloja tu aplicación, sino un entorno de herramientas (como Node, PHP, Python o Composer). Se usa para ejecutar comandos sin tener que instalar nada en tu sistema local.

- **Uso:** Ejecutar `npm install` o `django-admin startproject` dentro de un contenedor para que los archivos se generen en tu host a través de un bind mount.

### Ejemplo: Usar Node solo para instalar dependencias

```bash
# Ejecutamos node solo para inicializar un proyecto (npm init)
# --rm: Se borra al terminar
# -v: Sincroniza los archivos generados con nuestra carpeta local
docker run --rm -v "$(pwd):/app" -w /app node npm init -y
```

---

## 2. CMD vs. ENTRYPOINT

Ambas instrucciones definen qué comando se ejecuta al arrancar el contenedor, pero se comportan de forma distinta ante los argumentos que pasamos por consola.

| Instrucción    | Comportamiento                                                                                                           |
| :------------- | :----------------------------------------------------------------------------------------------------------------------- |
| **CMD**        | Es el comando por defecto. Si pasas un argumento al hacer `docker run`, **CMD se ignora por completo**.                  |
| **ENTRYPOINT** | Es el comando "fijo". Si pasas argumentos al hacer `docker run`, estos se **añaden** al final del comando de Entrypoint. |

### Ejemplo: Creando una herramienta de CLI

```dockerfile
FROM node:18-alpine
WORKDIR /app
# Si usamos ENTRYPOINT, el contenedor se comporta como un ejecutable
ENTRYPOINT ["npm"]
# CMD actúa como el argumento por defecto si el usuario no pone nada
CMD ["help"]
```

- Si ejecutas `docker run mi-utilidad`, ejecutará `npm help`.
- Si ejecutas `docker run mi-utilidad install`, ejecutará `npm install` (el "install" se anexa al entrypoint).

---

## 3. Servidor Nginx: ¿Qué es y para qué sirve?

**Nginx** es un servidor web de alto rendimiento, proxy inverso y balanceador de carga.

- **¿Para qué se emplea?** Principalmente para servir contenido estático (HTML, CSS, JS de React/Angular) o para actuar como "puerta de entrada" (Proxy) que redirige el tráfico a diferentes microservicios.
- **¿Qué aporta?** Seguridad, velocidad extrema manejando múltiples conexiones y una configuración sencilla para certificados SSL o compresión de archivos.

### Ejemplo: Servir una App de React con Nginx

```dockerfile
# Fase 1: Build (Utility stage)
FROM node:18 AS build-stage
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build # Genera la carpeta /dist o /build

# Fase 2: Producción con Nginx
FROM nginx:stable-alpine
# Copiamos solo el resultado del build al directorio que Nginx usa por defecto
COPY --from=build-stage /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## 4. Modificadores de Volúmenes: `ro` y `delegated`

Cuando montamos volúmenes, podemos añadir "flags" para mejorar la seguridad o el rendimiento.

### A. Read-Only (`:ro`)

Indica que el contenedor puede leer los archivos pero **jamás modificarlos**. Es vital para la seguridad.

```bash
# El contenedor puede leer el código pero no borrarlo ni alterarlo
docker run -v "$(pwd):/app:ro" mi-imagen
```

### B. Delegated (`:delegated`)

Se usa principalmente en Docker Desktop (Mac/Windows). Indica que el estado del contenedor es el "maestro" y que los cambios no necesitan sincronizarse instantáneamente en el host.

- **Ventaja:** Mejora muchísimo la velocidad de escritura/lectura en proyectos con miles de archivos pequeños (como `node_modules`).
- **Inconveniente:** Puede haber un pequeño retraso (milisegundos) en ver el cambio reflejado en tu carpeta local.

### Ejemplo de uso combinado:

```bash
# Código fuente como lectura (seguridad) y node_modules optimizado (rendimiento)
docker run -d \
  -v "$(pwd):/app:ro" \
  -v /app/node_modules:delegated \
  mi-app-profesional
```

---

## 5. Resumen de comandos para esta sección

| Tarea                   | Comando                                                                      |
| :---------------------- | :--------------------------------------------------------------------------- |
| **Utility run**         | `docker run --rm -v $(pwd):/app -w /app imagen comando`                      |
| **Entrypoint override** | `docker run --entrypoint /bin/sh imagen` (Para saltar el entrypoint)         |
| **Nginx logs**          | `docker logs <nginx-container-id>`                                           |
| **Check config**        | `docker exec <id-nginx> nginx -t` (Valida si el config de nginx es correcto) |

---

### Verificación de flujo

1.  Hemos usado un **Utility Container** para generar nuestra app.
2.  Hemos configurado el **Dockerfile** con un **ENTRYPOINT** para que sea una herramienta flexible.
3.  Hemos empaquetado el resultado en un servidor **Nginx** para producción.
4.  Hemos protegido nuestros archivos con el flag **ro** y optimizado el acceso con **delegated**.

¿Te gustaría que profundizáramos en los **Multi-stage Builds** para ver cómo reducir drásticamente el tamaño de tus imágenes de Nginx de 1GB a apenas 20MB?
