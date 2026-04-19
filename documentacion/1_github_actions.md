# Guía Profesional de GitHub Actions para Principiantes

> **Nota del Experto:** GitHub Actions es el motor de automatización de GitHub que permite implementar flujos de **Integración Continua y Despliegue Continuo (CI/CD)**. Imagina que tienes un robot asistente que, cada vez que subes código, se encarga de revisarlo, probarlo y subirlo al servidor por ti. Esta guía desglosa los componentes técnicos para que pases de cero a experto en la configuración de flujos automatizados.

---

## 1. Conceptos Fundamentales (Key Components)

Para entender GitHub Actions, debemos visualizar una jerarquía de contenedores:

- **Workflows:** Es el proceso automatizado total (el archivo `.yml`). Está asociado a un repositorio y se dispara mediante un **Event** (como un push).
- **Jobs:** Son grupos de tareas que se ejecutan dentro de un Workflow. Por defecto, los jobs corren en **paralelo** para ahorrar tiempo, aunque pueden configurarse para ser secuenciales o condicionales. Cada job define su propio **Runner** (el servidor donde vive).
- **Steps:** Son las tareas individuales dentro de un job. Se ejecutan en **orden estricto** (secuencial). Un step puede ejecutar un comando de consola (`run`) o una **Action** (un bloque de código pre-hecho).

---

## 2. Configuración y Estructura del Archivo `.yml`

### ¿Dónde vive el código?

Para que GitHub reconozca tus flujos, los archivos deben estar en la ruta específica: `.github/workflows/nombre-del-archivo.yml`.

### Anatomía del YAML

- **name:** El nombre descriptivo que verás en la pestaña "Actions".
- **on:** Define los **triggers** (eventos).
- **runs-on:** Define el **Runner** (entorno de ejecución).
  - `ubuntu-latest`: El más común, rápido y económico.
  - `windows-latest` / `macos-latest`: Úsalos solo si tu app depende específicamente de esos SO.

---

## 3. Eventos, Actions y Sintaxis Avanzada

### Events (Triggers)

Los eventos más comunes son:

- `push`: Al subir código.
- `pull_request`: Al abrir o actualizar una propuesta de cambio.
- `workflow_dispatch`: Permite ejecutar el workflow **manualmente** mediante un botón en la interfaz de GitHub.

### Actions vs Workflows

No los confundas: El **Workflow** es el archivo completo. Una **Action** es una aplicación reusable que simplifica un step (como `actions/checkout`). Se invocan con la palabra clave `uses`.

### Uso de `uses`, `v`, `@` y `with`

Cuando ves `actions/checkout@v4`:

- **@**: Separa el nombre de la acción de su versión.
- **v4**: Indica la versión mayor. Es vital fijar versiones para que cambios externos no rompan tu CI/CD.
- **with**: Se usa para pasar parámetros o variables de configuración a esa Action específica.

---

## 4. Buenas Prácticas y Comandos CI

### `npm ci` vs `npm install`

En entornos de CI, siempre usa **`npm ci`**. A diferencia de `install`, `ci` (Clean Install) es más rápido, no modifica el `package-lock.json` y garantiza que las dependencias sean exactamente las mismas que en tu entorno local.

### Secuencialidad con `needs`

Si el Job B depende de que el Job A termine con éxito (ejemplo: no despliegues si los tests fallan), usamos `needs: [job_a]`.

### Expressions y Contextos

GitHub permite usar lógica mediante `${{ <expression> }}`. Los contextos más usados son:

- `github`: Información sobre el repo y el evento (`github.ref`, `github.actor`).
- `secrets`: Para manejar datos sensibles (tokens, contraseñas).
- `matrix`: Para correr el mismo job en múltiples versiones de software.

---

## Ejemplo Práctico: Workflow Profesional de Integración

Este ejemplo simula un flujo real donde primero se testea una aplicación Node.js y, si todo va bien, se simula un despliegue.

```yaml
# 1. El nombre que aparecerá en la pestaña Actions de GitHub
name: Pipeline de Integración Continua v1

# 2. Eventos que disparan el workflow
on:
  push:
    branches: [main, develop] # Se dispara al hacer push a estas ramas
  pull_request:
    branches: [main] # Se dispara al abrir PR hacia main
  workflow_dispatch: # Permite ejecución manual desde la web

jobs:
  # --- PRIMER JOB: PRUEBAS ---
  unit-tests:
    name: Ejecución de Tests Unitarios
    runs-on: ubuntu-latest # Usamos Linux por eficiencia y costo

    steps:
      # 'uses' descarga una Action oficial para clonar nuestro código en el Runner
      # @v4 indica la versión de la acción
      - name: Clonar el código del repositorio
        uses: actions/checkout@v4

      # Configuramos el entorno de Node.js
      - name: Configurar Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20" # Definimos la versión de Node mediante 'with'

      # 'npm ci' es la mejor práctica para entornos de CI/CD
      - name: Instalar dependencias limpias
        run: npm ci

      # 'run' permite ejecutar múltiples comandos usando el pipe '|'
      - name: Linter y Tests
        run: |
          npm run lint
          npm test
        # Podemos usar condicionales: este paso solo corre si estamos en una rama específica
        if: github.event_name == 'push'

  # --- SEGUNDO JOB: DESPLIEGUE ---
  deploy:
    name: Despliegue a Producción
    runs-on: ubuntu-latest
    # IMPORTANTE: Este job solo iniciará si 'unit-tests' termina con éxito
    needs: [unit-tests]
    # Condicional: Solo despliega si estamos en la rama 'main'
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Descargar código
        uses: actions/checkout@v4

      - name: Simular Despliegue
        run: echo "Desplegando la aplicación de ${{ github.actor }}..."
        # Usamos expresiones ${{ }} para acceder al contexto 'github'
```

### Resumen de uso en GitHub:

1.  Entra en tu repositorio de GitHub.
2.  Haz clic en la pestaña **Actions**.
3.  Si no tienes workflows, verás opciones para crear uno. Haz clic en "set up a workflow yourself".
4.  Pega el código anterior, nómbralo `main.yml` y haz commit.
5.  ¡Listo! GitHub detectará automáticamente el archivo y comenzará la ejecución según los eventos definidos.
