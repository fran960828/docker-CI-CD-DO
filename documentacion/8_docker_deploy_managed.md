# Guía Profesional: Despliegue Gestionado con AWS ECS y Fargate

> **Nota para principiantes:** En la guía anterior vimos cómo gestionar nosotros mismos un servidor (EC2). Ahora daremos el salto al nivel "Managed" (Gestionado). Aquí, en lugar de preocuparnos por instalar parches de Linux o Docker, delegamos esa responsabilidad a AWS mediante **ECS**. Es el equilibrio perfecto: tú mantienes el control de tu aplicación, pero AWS se encarga de que la infraestructura siempre esté disponible y sea segura.

---

## 1. Managed Remote Hosts: AWS ECS

Cuando hablamos de un **Managed Remote Host**, nos referimos a servicios donde el proveedor de la nube administra el hardware y el sistema operativo por ti.

**AWS ECS (Elastic Container Service)** es un orquestador de contenedores totalmente gestionado. A diferencia de EC2, aquí no te conectas por SSH para instalar Docker; simplemente le dices a AWS: "Aquí está mi imagen de Docker Hub, ejecútala", y AWS busca dónde ponerla.

### La Interfaz Gráfica de ECS

ECS ofrece una consola visual intuitiva que permite configurar:

- Límites de memoria y CPU.
- Variables de entorno.
- Reglas de red y seguridad.
- Registro de logs (CloudWatch) para ver qué pasa dentro del contenedor sin entrar por terminal.

---

## 2. Task Definitions y el Poder de FARGATE

Para correr un contenedor en ECS, necesitamos una "receta" llamada **Task Definition** (Definición de Tarea).

### ¿Qué es FARGATE?

Es la tecnología **Serverless** para contenedores.

- **Sin Fargate:** Tendrías que crear instancias de EC2 y unirlas a un cluster.
- **Con Fargate:** No hay servidores que gestionar. AWS reserva una "parcela" de computación exclusiva para tu contenedor.

### Configuración de Recursos

En la Task Definition, debes ser específico para no pagar de más:

- **CPU:** Se mide en unidades (vCPU). Puedes asignar desde 0.25 vCPU.
- **Memoria:** Se asigna en GB.
- **Almacenamiento:** Fargate ofrece un almacenamiento efímero para archivos temporales del contenedor.

---

## 3. Load Balancer (Equilibrador de Carga)

En producción, nunca exponemos un contenedor directamente a internet. Usamos un **Application Load Balancer (ALB)**.

**¿Para qué sirve?**

1.  **Punto de entrada único:** El usuario accede a una URL fija.
2.  **Health Checks:** Si un contenedor falla, el Load Balancer deja de enviarle tráfico y lo redirige a uno sano.
3.  **Escalabilidad:** Si tienes 10 copias de tu app, el ALB reparte las visitas entre todas de forma equitativa.

---

## 4. Ejemplo Práctico: Despliegue con ECS y Fargate

### Paso A: Preparación de la imagen (Local)

Primero, aseguramos que nuestra imagen esté en la nube para que AWS pueda leerla.

```bash
# 1. Construimos la imagen localmente
docker build -t tu-usuario/app-prod:v1 .

# 2. Iniciamos sesión en Docker Hub
docker login

# 3. Subimos la imagen (Source of Truth)
docker push tu-usuario/app-prod:v1
```

### Paso B: Creación de la Task Definition (Consola AWS)

1.  Ve a **ECS** -> **Task Definitions** -> **Create new Task Definition**.
2.  **Infraestructura:** Selecciona **AWS Fargate**.
3.  **CPU y Memoria:** Selecciona `0.5 vCPU` y `1 GB RAM` (Suficiente para empezar).
4.  **Container:** \* Name: `mi-contenedor-app`
    - Image URI: `tu-usuario/app-prod:v1` (La ruta de Docker Hub).
    - Port Mapping: `80` (o el que use tu app).

### Paso C: Crear el Cluster y el Servicio

1.  Crea un **Cluster** (un grupo lógico de tareas) con el nombre `production-cluster`.
2.  Dentro del cluster, crea un **Service**. El servicio se encarga de mantener viva la tarea.
3.  Selecciona el **Load Balancer** que creaste previamente para que gestione el tráfico hacia el servicio.

---

## 5. Cómo actualizar contenido en producción

A diferencia de EC2, aquí no entramos al servidor a hacer "pull". El flujo es declarativo:

### Flujo de actualización:

1.  **Actualiza tu código** localmente.
2.  **Build y Push:** Sube la versión `v2` a Docker Hub.
3.  **Nueva Revisión:** En la consola de AWS, crea una **nueva revisión** de la Task Definition cambiando la etiqueta de la imagen a `:v2`.
4.  **Update Service:** Ve a tu Cluster -> `clustersdefault` (o el nombre que asignaste), selecciona el servicio y dale a **Update**. Selecciona la nueva revisión de la tarea.

> **Qué sucede por detrás:** AWS lanzará la versión `v2` en paralelo. Cuando el Load Balancer confirme que la `v2` responde bien (Health Check), apagará la `v1`. **¡Cero tiempo de inactividad!**

---

## 6. Conclusión: El Compromiso de ECS

Esta forma de despliegue tiene una ventaja competitiva enorme: **Alta Disponibilidad**.

- **Lo que tú haces:** Definir cuánta CPU quieres y subir tu imagen.
- **Lo que hace AWS:** Gestionar la red, el balanceo de carga, el auto-escalado y asegurar que si un contenedor muere, otro nazca al instante.

**Riesgo:** La principal desventaja es el coste (Fargate es ligeramente más caro que EC2 puro) y la curva de aprendizaje inicial de la interfaz de AWS. Sin embargo, para un nivel profesional, la paz mental de no gestionar servidores físicos no tiene precio.
