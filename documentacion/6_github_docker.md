# Guía Profesional: Docker Containers y Service Containers en GitHub Actions

> **Nota del Experto:** GitHub Actions permite ejecutar tus Jobs directamente dentro de un contenedor Docker o levantar servicios adicionales (como bases de datos o sistemas de caché) durante la ejecución. Esto garantiza que tu entorno de CI sea idéntico al de producción, evitando el clásico "en mi máquina funciona". Además, el uso de **Service Containers** es la mejor práctica para realizar pruebas de integración de forma segura y efímera.

---

## 1. El Job Container (`container`)

Por defecto, los Jobs corren directamente en la máquina virtual (Runner) de GitHub. Sin embargo, puedes indicar que un Job se ejecute **dentro** de un contenedor específico.

- **¿Por qué usarlo?** Para tener todas las herramientas, lenguajes y versiones específicas ya instaladas en una imagen personalizada.
- **Sintaxis:** Se usa la keyword `container` a nivel de Job.
- **Opciones:** Puedes definir la `image`, variables de entorno (`env`), `ports`, y `volumes`.

---

## 2. Service Containers (`services`)

Los **Services** son contenedores adicionales que se ejecutan junto a tu Job principal. El caso de uso más común es levantar una base de datos (PostgreSQL, Redis, MongoDB) para que tus tests puedan interactuar con ella.

- **Efimeridad:** El servicio se inicia automáticamente al empezar el Job y **se destruye (apaga)** inmediatamente al terminar, garantizando que no queden datos residuales ni costes innecesarios.
- **Aislamiento:** Te permite probar contra una base de datos real sin riesgo de corromper los datos de producción.

---

## 3. Conectividad: ¿Cómo se comunican?

La forma en que tu código accede al servicio depende de dónde se esté ejecutando el Job:

### Caso A: El Job corre DENTRO de un contenedor (`container`)

Si el Job principal también es un contenedor, GitHub Actions los coloca a todos en la misma red de Docker.

- **Host:** Puedes usar el **nombre del servicio** definido en el YAML como hostname (ej: `postgres` o `redis`).
- **Puertos:** Se comunican directamente a través de los puertos internos del contenedor (ej: `5432`).

### Caso B: El Job corre DIRECTAMENTE en el Runner (sin `container`)

Si el Job corre sobre el sistema operativo del Runner (ej: `ubuntu-latest`), el servicio es accesible a través de la interfaz de red local.

- **Host:** Debes usar `localhost`.
- **Puertos:** Debes mapear el puerto del contenedor al puerto del host usando la sección `ports`.

---

## Ejemplo Práctico: Test de Integración con PostgreSQL

Este ejemplo muestra un flujo donde se levanta una base de datos PostgreSQL para probar una aplicación Node.js.

```yaml
name: CI con Base de Datos Dockerizada

on: [push]

jobs:
  # --- EJEMPLO 1: JOB DENTRO DE UN CONTENEDOR ---
  test-in-container:
    runs-on: ubuntu-latest
    # Definimos que este Job entero vive dentro de una imagen de Node
    container:
      image: node:20-bookworm
      env:
        NODE_ENV: test

    # Levantamos un servicio de base de datos
    services:
      postgres-db: # Este será el hostname
        image: postgres:15
        env:
          POSTGRES_PASSWORD: mysecretpassword
        # Aquí NO necesitamos mapear puertos (ports:),
        # porque estamos en la misma red Docker.

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Instalar y Probar
        run: |
          npm ci
          # Conectamos usando el ID del servicio como host
          # El puerto es el interno por defecto de Postgres
          export DATABASE_URL="postgresql://postgres:mysecretpassword@postgres-db:5432/postgres"
          npm test

  # --- EJEMPLO 2: JOB EN EL RUNNER (LOCALHOST) ---
  test-on-runner:
    runs-on: ubuntu-latest

    services:
      redis-cache:
        image: redis
        # Aquí SÍ necesitamos mapear el puerto al host (Runner)
        ports:
          - 6379:6379
        # Health checks: Asegura que el servicio esté listo antes de empezar el Job
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Test Cache
        run: |
          npm ci
          # Al estar en el Runner, conectamos a localhost
          export REDIS_URL="redis://localhost:6379"
          npm test
```

### Resumen de Conectividad para el principiante:

| Si tu Job corre en... | Accede al Service usando...            | ¿Requiere mapear `ports`?        |
| :-------------------- | :------------------------------------- | :------------------------------- |
| **Container**         | El ID del servicio (ej: `postgres-db`) | No (usa puerto interno)          |
| **Runner (OS)**       | `localhost`                            | **Sí** (mapeo `externo:interno`) |

> **Tip de Experto:** Usa siempre **Health Checks** (como se muestra en el ejemplo de Redis) en tus servicios pesados. Esto evita que tus tests empiecen a ejecutarse antes de que la base de datos haya terminado de arrancar por completo, lo cual es una causa común de fallos aleatorios en CI/CD.
