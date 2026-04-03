# Guía Avanzada: Despliegue Multi-contenedor en AWS ECS con Persistencia y Escalado

> **Nota para principiantes:** En esta etapa aprenderás a gestionar aplicaciones complejas que dependen de varios servicios (como una App de Node.js + una Base de Datos MongoDB). En lugar de configurar cada servidor a mano, usaremos un enfoque de "Orquestación", donde definimos cómo deben interactuar los contenedores, cuánta potencia necesitan y cómo deben sobrevivir a fallos o picos de tráfico de forma automática.

---

## 1. El Concepto de Task Multi-contenedor en ECS

En AWS ECS, la unidad mínima no es el "contenedor", sino la **Task** (Tarea). Una Task puede contener uno o varios contenedores que comparten el mismo ciclo de vida y red.

### Comunicación mediante Localhost

Una ventaja clave es que, si pones tu App y tu base de datos (MongoDB) en la **misma Task**, se pueden comunicar usando `localhost`.

- **En desarrollo:** Usabas el nombre del servicio de docker-compose (ej: `mongodb://database:27017`).
- **En producción (dentro de la misma Task):** Usas `mongodb://localhost:27017`. Esto es extremadamente rápido y seguro porque el tráfico no sale de la tarea.

---

## 2. Configuración de la Task Definition (Paso a Paso)

Para proyectos profesionales, configuramos lo siguiente en la interfaz de ECS:

1.  **Creación del Cluster:** Es el grupo lógico de recursos. Piensa en él como el "vecindario" donde vivirán tus tareas.
2.  **Definición de Tarea (Task):** Aquí añadimos los contenedores.
    - **Container de la App:** Indicamos el nombre, la imagen de Docker Hub y las **Environment Keys** (variables de entorno como `PORT`, `MONGO_URL`, etc.).
    - **Comandos:** Si tu Dockerfile usa `CMD ["npm", "start"]`, ECS lo ejecutará automáticamente al iniciar.
    - **Container de MongoDB:** Añadimos un segundo contenedor basado en la imagen oficial `mongo`.
3.  **Recursos:** Definimos la **vCPU** (ej. 1 vCPU) y la **Memoria** (ej. 2 GB) que se repartirán entre ambos contenedores.

---

## 3. Ejemplo de Configuración de Contenedores (Simulado)

```yaml
# Esta es una representación visual de cómo se configuran las variables en la interfaz de AWS
Container_1 (App_Node):
  Image: "usuario/mi-app:v1"
  PortMappings: [80]
  Environment:
    - MONGO_URL: "mongodb://localhost:27017/mi_db" # Conexión interna vía localhost
    - NODE_ENV: "production"

Container_2 (Database):
  Image: "mongo:latest"
  PortMappings: [27017] # Puerto interno de la base de datos
```

---

## 4. Servicios, Networking y Load Balancer

Un **Service** es el guardián de la Task. Si una tarea muere, el servicio lanza otra.

- **Load Balancer (ALB):** Recibe el tráfico de internet y lo reparte entre tus Tasks.
- **Healthy Path (Ruta de Salud):** Es una ruta en tu app (ej: `/health` o `/`) que el Load Balancer consulta cada pocos segundos. Si tu app responde un código `200 OK`, el Load Balancer sabe que está sana. Si falla, el Load Balancer deja de enviarle gente y pide a ECS que reinicie la tarea.

---

## 5. Escalado Automático (Auto-scaling)

El Auto-scale permite que tu infraestructura crezca o se encoja sola. Puedes configurar reglas como:

> "Si el consumo de CPU promedio de mis tareas supera el 70%, lanza 2 tareas más automáticamente".

Esto garantiza que tu aplicación no se caiga durante un Black Friday o un pico de usuarios inesperado.

---

## 6. Persistencia de Datos con EFS (Elastic File System)

Por defecto, los contenedores son "efímeros": si apagas la tarea, los datos de MongoDB desaparecen. Para evitar esto, conectamos **AWS EFS**.

- **EFS** es como un disco duro en la nube que nunca se llena y que pueden compartir varios contenedores.
- Al actualizar tu app o reiniciar la Task, el nuevo contenedor de MongoDB se "engancha" al mismo volumen de EFS y recupera todos los datos.

---

## Ejemplo: Flujo Completo de Despliegue Profesional

### Paso 1: Configurar el Volumen EFS (Consola AWS)

1. Creamos un sistema de archivos EFS.
2. En la **Task Definition**, añadimos un "Volume" de tipo EFS.
3. En la configuración del contenedor de MongoDB, hacemos un **Mount Point**:
   - _Container Path:_ `/data/db` (Donde Mongo guarda los datos).
   - _Source Volume:_ El volumen EFS que creamos.

### Paso 2: Creación del Servicio con Load Balancer

```bash
# Conceptualmente, esto es lo que ocurre al activar el servicio:
1. ECS lanza la Task (App + Mongo).
2. Mongo empieza a escribir en EFS (Persistencia).
3. La App se conecta a Mongo en 'localhost:27017'.
4. El Load Balancer revisa el path '/health'.
5. Si '/health' responde 200, el tráfico empieza a fluir hacia la App.
```

### Paso 3: Actualización y Mantenimiento

Cuando quieras subir cambios:

1. Subes la nueva imagen a Docker Hub.
2. Actualizas la Task Definition.
3. El Servicio hace un "Rolling Update": levanta la nueva versión, espera a que el **Healthy Path** dé el visto bueno y luego apaga la vieja.

---

## Resumen de Ventajas Profesionales

| Característica           | Beneficio en Producción                                                        |
| :----------------------- | :----------------------------------------------------------------------------- |
| **Multi-container Task** | Latencia casi cero entre App y DB usando `localhost`.                          |
| **Healthy Path**         | Evita que los usuarios vean errores; el tráfico solo va a contenedores listos. |
| **Auto-scaling**         | Ahorro de costos (menos tareas de noche) y fiabilidad (más tareas en picos).   |
| **EFS Integration**      | Seguridad total de que tus datos no se borrarán al reiniciar servicios.        |

# Guía Profesional: De Contenedores de Base de Datos a Servicios Gestionados (MongoDB Atlas)

> **Nota para principiantes:** Hasta ahora, hemos aprendido a correr MongoDB dentro de un contenedor en AWS ECS. Aunque esto funciona para proyectos pequeños o de aprendizaje, en el mundo profesional "correr tu propia base de datos" en un contenedor es una tarea de alto riesgo. Esta guía te explicará por qué las empresas prefieren pagar por servicios gestionados y cómo migrar tu arquitectura para que sea escalable y segura.

---

## 1. El Problema de "Auto-gestionar" MongoDB en Contenedores

Correr una imagen de `mongo` en una Task de ECS parece sencillo, pero en producción surgen desafíos críticos:

- **Escalabilidad Vertical y Horizontal:** Si tu base de datos se queda sin RAM durante un pico de tráfico, el contenedor morirá. Configurar un "Cluster de Réplica" (varios nodos de Mongo sincronizados) manualmente en Docker es extremadamente complejo.
- **Disponibilidad:** Si el nodo donde corre tu contenedor falla, tu base de datos queda fuera de línea hasta que ECS levante otra. Esos segundos o minutos de caída pueden costar mucho dinero.
- **Backups y Recuperación:** Tú eres el responsable de programar scripts que saquen copias de seguridad, las suban a S3 y verifiquen que no estén corruptas.
- **Rendimiento en Picos:** Optimizar el motor de almacenamiento de Mongo dentro de las limitaciones de CPU/RAM de un contenedor Fargate suele dar resultados mediocres comparado con hardware dedicado.

**La Solución: Managed Database Services (DBaaS)**
Servicios como **MongoDB Atlas** o **AWS RDS** eliminan estas preocupaciones. Ellos se encargan de los parches de seguridad, los backups automáticos, la alta disponibilidad (siempre hay 3 copias de tu dato) y el escalado con un solo clic.

---

## 2. Migración a MongoDB Atlas: Cambios en la App

Cuando dejas de usar el contenedor de Mongo local, tu aplicación de Node.js ya no se conecta a `localhost`. Ahora se conecta a una **URL de conexión en la nube**.

### Cambio en el código (Node.js + Mongoose)

**Antes (Desarrollo/Contenedor):**

```javascript
// Conexión a la base de datos interna de la Task de ECS
const mongoURI = "mongodb://localhost:27017/mi_proyecto";
```

**Ahora (Producción con Atlas):**

```javascript
// Conexión a la nube. Los datos sensibles van en variables de entorno.
// El formato SRV permite que Atlas gestione la disponibilidad automáticamente.
const mongoURI = process.env.MONGO_ATLAS_URI;

mongoose
  .connect(mongoURI)
  .then(() => console.log("Conectado a MongoDB Atlas (Cloud)"))
  .catch((err) => console.error("Error de conexión", err));
```

---

## 3. Ajustes en la Infraestructura de Producción (AWS ECS)

Al mover la base de datos a Atlas, nuestra arquitectura en AWS se vuelve más "limpia" y ligera.

### ¿Qué eliminamos?

1.  **El Contenedor de Mongo:** Ya no necesitamos la imagen `mongo` en nuestra Task Definition. Esto ahorra CPU y RAM en nuestra factura de AWS.
2.  **AWS EFS (Elastic File System):** Como los datos ya no viven en nuestros contenedores sino en Atlas, ya no necesitamos montar volúmenes persistentes.
3.  **Group Permissions:** Nos olvidamos de configurar permisos de lectura/escritura de carpetas de Linux para los volúmenes, eliminando una fuente común de errores.

### ¿Qué añadimos?

- **Variables de Entorno (Environment Keys):** En la Task Definition de ECS, añadimos la clave `MONGO_ATLAS_URI` con el valor que nos proporciona el panel de Atlas.
- **IP Whitelisting:** En el panel de MongoDB Atlas, debemos permitir el acceso desde la IP pública de nuestro servicio de AWS o usar un "VPC Peering" para máxima seguridad.

---

## 4. Ejemplo Comparativo: Task Definition

### Caso A: Estructura Local/ECS (Todo en uno)

_No recomendado para producción crítica._

```yaml
# Task Definition antigua
Containers:
  - Name: "my-node-app"
    Image: "usuario/app:v1"
  - Name: "mongodb-container"
    Image: "mongo:latest"
    MountPoints:
      - SourceVolume: "my-efs-storage" # Necesitábamos EFS
        ContainerPath: "/data/db"
```

### Caso B: Estructura Profesional (App + Atlas)

_Estándar de la industria._

```yaml
# Task Definition limpia
Containers:
  - Name: "my-node-app"
    Image: "usuario/app:v2"
    Environment:
      - Name: "MONGO_ATLAS_URI"
        Value: "mongodb+srv://admin:password@cluster0.mongodb.net/prod_db"
# Ya no hay contenedor de Mongo aquí.
# Ya no hay volúmenes EFS.
```

---

## 5. ¿Cuándo usar cada opción? (Control vs Responsabilidad)

| Situación            | Usar Imagen Docker (Self-hosted)                   | Usar Managed Service (Atlas/RDS)                 |
| :------------------- | :------------------------------------------------- | :----------------------------------------------- |
| **Desarrollo Local** | **Sí.** Es gratis y rápido de levantar.            | No es necesario, pero podrías.                   |
| **Prototipo / MVP**  | **Tal vez.** Si el presupuesto es cero.            | **Recomendado.** Evitas perder datos por fallos. |
| **Producción Real**  | **No.** El riesgo de pérdida de datos es muy alto. | **Obligatorio.** Por backups y seguridad.        |
| **Coste**            | Bajo (solo pagas la RAM de la Task).               | Medio/Alto (pagas por el servicio gestionado).   |

### Resumen de la Transición

1.  **Crea un cluster gratuito** en [MongoDB Atlas](https://www.mongodb.com/cloud/atlas).
2.  **Obtén la Connection String**.
3.  **Actualiza tu código Node.js** para usar variables de entorno.
4.  **Limpia tu AWS ECS:** Borra el contenedor de Mongo y el volumen EFS de la Task Definition.
5.  **Despliega:** Tu App ahora es "stateless" (sin estado), lo que facilita enormemente el **Auto-scaling**.

¿Te gustaría que te explicara cómo configurar el Firewall (IP Access List) en Atlas para que solo tu App de AWS pueda entrar?
