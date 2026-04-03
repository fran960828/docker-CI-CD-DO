¡Hola de nuevo! Como experto en Docker, entramos ahora en uno de los temas que más confusión suele generar, pero que es el "pegamento" de cualquier arquitectura moderna: **Networking**.

En Docker, los contenedores son islas. La red es lo que construye los puentes entre esas islas, tu máquina local y el mundo exterior (APIs). Aquí tienes la documentación técnica detallada para dominar la comunicación entre contenedores.

---

# 🌐 Guía Avanzada de Docker Networking

> **Nota de Experto:** Un error común es pensar que los contenedores pueden hablarse entre sí usando `localhost`. En Docker, `localhost` dentro de un contenedor se refiere **al contenedor mismo**. Para conectar piezas, necesitamos entender las tres direcciones de comunicación: Hacia afuera (Internet), hacia el Host (tu PC) y entre contenedores (Redes internas).

---

## 1. Tipos de Comunicación (Cross-Container)

Existen tres escenarios principales de comunicación para un contenedor:

### A. Comunicación con APIs Públicas

Un contenedor, por defecto, tiene acceso a Internet. No requiere configuración especial para consumir una API externa (como la de Google Maps o Stripe).

```bash
# Ejemplo: Un contenedor de Node que hace un fetch a una API externa
docker run --name app-api node-app
# El contenedor resolverá DNS externos automáticamente.
```

### B. Comunicación con el Host (Tu PC)

Si tienes una base de datos instalada directamente en tu Windows/Mac/Linux (fuera de Docker) y quieres que tu contenedor se conecte a ella, no puedes usar `localhost`.

- **Solución:** Usar el dominio especial `host.docker.internal`.

```javascript
// En tu código Node.js, en lugar de:
// mongoose.connect('mongodb://localhost:27017/mydb');

// Debes usar:
mongoose.connect("mongodb://host.docker.internal:27017/mydb");
```

### C. Comunicación entre Contenedores

Es la comunicación entre dos procesos que viven en contenedores distintos dentro de la misma máquina.

---

## 2. El método "Manual" (IP Address)

Antes de usar redes, la gente solía buscar la IP interna del contenedor. Aunque funciona, **no se recomienda en producción** porque las IPs cambian si el contenedor se reinicia.

### Ejemplo con MongoDB:

```bash
# 1. Corremos un contenedor de Mongo
docker run -d --name mi-mongo mongo

# 2. Obtenemos su IP interna (buscamos la sección "IPAddress")
docker inspect mi-mongo

# 3. Supongamos que la IP es 172.17.0.2. En nuestro código pondríamos:
# mongoose.connect('mongodb://172.17.0.2:27017/mi-db');

# 4. Reconstruimos la imagen de nuestra app y la corremos
docker build -t mi-app .
docker run --name mi-app-container mi-app
```

---

## 3. El método Profesional: Docker Networks 🚀

Las redes de Docker permiten que los contenedores se descubran por **nombre**, ignorando las IPs dinámicas.

### Creación de una Network

```bash
# Creamos una red de tipo bridge (puente)
docker network create mi-red-app
```

### Conectar contenedores a la Red

Para que se hablen, ambos deben estar en la misma red. Además, usaremos el **nombre del contenedor** como dominio.

```bash
# 1. Corremos la DB en la red
docker run -d \
  --name mongodb-cont \
  --network mi-red-app \
  mongo

# 2. Corremos nuestra App en la misma red
docker run -d \
  --name mi-web-app \
  --network mi-red-app \
  mi-imagen-app
```

> **¡Importante!:** En el código de tu aplicación, la cadena de conexión ahora sería:
> `mongodb://mongodb-cont:27017/mi-db` (Docker traduce "mongodb-cont" a la IP correcta automáticamente).

---

## 4. Tipos de Drivers de Red

Al crear una red (`docker network create --driver <tipo>`), puedes elegir el comportamiento:

| Driver      | Descripción                                                               | Caso de uso                                                 |
| :---------- | :------------------------------------------------------------------------ | :---------------------------------------------------------- |
| **bridge**  | El valor por defecto. Crea una red privada interna en el host.            | Aplicaciones estándar con varios contenedores.              |
| **host**    | Elimina el aislamiento entre el contenedor y el host (usa la red del PC). | Aplicaciones que requieren máximo rendimiento de red.       |
| **none**    | El contenedor no tiene red. Aislamiento total.                            | Tareas de computación críticas que no deben tocar internet. |
| **overlay** | Conecta varios demonios de Docker en distintos servidores físicos.        | Docker Swarm o clusters de servidores.                      |

---

## 5. Resumen de comandos de Networking

```bash
# Listar todas las redes disponibles
docker network ls

# Inspeccionar una red (ver qué contenedores están conectados)
docker network inspect mi-red-app

# Conectar un contenedor que ya está corriendo a una red
docker network connect mi-red-app nombre-contenedor

# Desconectar un contenedor de una red
docker network disconnect mi-red-app nombre-contenedor

# Borrar una red (no debe tener contenedores conectados)
docker network rm mi-red-app
```

---

## Ejemplo Práctico Final

Imagina que tienes una aplicación que necesita una base de datos Mongo. Este sería el flujo de trabajo profesional:

```bash
# 1. Preparar el entorno
docker network create social-network

# 2. Lanzar la base de datos con un nombre claro
docker run -d --name db-mongo --network social-network mongo

# 3. Lanzar la aplicación conectándola a la misma red
# Nota: La app en su código interno apunta a "db-mongo"
docker run -d \
  --name web-app \
  --network social-network \
  -p 8080:3000 \
  mi-usuario/web-app:v1

# 4. Verificar que están hablando
docker network inspect social-network
```

¿Te gustaría que pasáramos a ver **Docker Compose**? Es la herramienta que permite escribir todo este despliegue (redes, volúmenes y contenedores) en un solo archivo YAML para no tener que escribir tantos comandos en la consola.
