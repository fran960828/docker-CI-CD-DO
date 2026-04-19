# Guía Definitiva: Creación de Custom Actions Profesionales

> **Nota del Experto:** Las **Custom Actions** son la unidad de reutilización más potente de GitHub Actions. Mientras que un Workflow Reutilizable comparte un archivo `.yml` completo, una Action permite crear "piezas de Lego" personalizadas que puedes conectar en cualquier Step. Se usan para simplificar flujos complejos, estandarizar procesos en toda una empresa o cuando las acciones del Marketplace no cubren exactamente tus necesidades técnicas.

---

## 1. Tipos de Custom Actions: ¿Cuál elegir?

Existen tres formas de construir una Action. La elección depende de la complejidad y el entorno:

| Tipo           | Ventajas                                                                          | Inconvenientes                                                         | ¿Cuándo usarla?                                                             |
| :------------- | :-------------------------------------------------------------------------------- | :--------------------------------------------------------------------- | :-------------------------------------------------------------------------- |
| **Composite**  | La más sencilla. Agrupa varios comandos `run` o `uses` en un solo paso.           | Limitada en lógica compleja (bucles, condicionales avanzados).         | Para scripts de shell repetitivos o agrupar pasos comunes.                  |
| **JavaScript** | Ejecución rápida (corre directo en el Runner). Acceso total a las APIs de GitHub. | Requiere node_modules (hay que compilarla o subir las dependencias).   | Para lógica de negocio compleja que interactúa con la API de GitHub.        |
| **Docker**     | Entorno 100% controlado. Puedes usar cualquier lenguaje (Python, Go, Rust).       | Más lenta (debe construir o descargar la imagen cada vez). Solo Linux. | Cuando necesitas herramientas de SO específicas o lenguajes que no sean JS. |

---

## 2. Disponibilidad: Local vs. Pública

- **Acciones Locales:** Solo para el repositorio actual. Se guardan en una carpeta (ej. `.github/actions/mi-accion`). No necesitan versionado estricto pero no son visibles para otros proyectos.
- **Acciones Globales:** Viven en su propio repositorio público. Cualquier persona o repositorio puede usarlas mediante la sintaxis `usuario/repo@version`.

---

## 3. Anatomía de una Composite Action Local

Para crear una acción local, la estructura de archivos recomendada es:
`.github/actions/nombre-de-tu-accion/action.yml`

### El archivo `action.yml`:

Este archivo define la interfaz de tu acción.

- **name / description:** Identificadores.
- **inputs:** Parámetros que recibe (ej. un token, una versión).
- **outputs:** Datos que devuelve al workflow.
- **runs:** Define que es `using: 'composite'`.
- **steps:** La lista de comandos. **Importante:** Cada step `run` requiere la clave `shell: bash` (o el shell que prefieras).

---

## 4. Inputs y Outputs: La Comunicación

### Inputs

Se definen en el bloque `inputs`. Para usarlos dentro de la acción, empleamos la expresión `${{ inputs.nombre_del_input }}`. En el workflow principal, se pasan mediante la keyword `with`.

### Outputs

Se definen en el bloque `outputs`. Para que funcionen, un step de la acción debe escribir en el archivo `$GITHUB_OUTPUT`. El workflow principal los accede mediante `${{ steps.id_del_paso.outputs.nombre_del_output }}`.

---

## Ejemplo Práctico: Action de "Saludo y Auditoría"

Vamos a crear una acción que reciba un nombre, imprima un saludo y devuelva la hora actual.

### Paso 1: Definición de la Action

Archivo: `.github/actions/hello-audit/action.yml`

```yaml
# Definición de la lógica de la Action
name: "Hello Audit"
description: "Saluda al usuario y genera un timestamp de auditoría"

# 1. Definimos las entradas (Inputs)
inputs:
  who-to-greet:
    description: "Nombre de la persona a saludar"
    required: true
    default: "Mundo"

# 2. Definimos las salidas (Outputs)
outputs:
  time:
    description: "La hora exacta en la que se ejecutó el saludo"
    # Mapeamos el output de la action al output de un step específico
    value: ${{ steps.get-time.outputs.current_time }}

runs:
  using: "composite" # Indicamos que es una suma de pasos
  steps:
    - name: Saludar
      run: echo "Hola ${{ inputs.who-to-greet }}!"
      shell: bash # Obligatorio en composite actions

    - name: Generar Time
      id: get-time # ID necesario para que el output lo localice
      run: |
        # Escribimos el valor en el archivo especial de GitHub
        echo "current_time=$(date)" >> $GITHUB_OUTPUT
      shell: bash
```

---

### Paso 2: Uso de la Action en el Workflow

Archivo: `.github/workflows/main.yml`

```yaml
name: Test Custom Action

on: [push]

jobs:
  test-job:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      # USO DE ACTION LOCAL:
      # Se usa la ruta relativa a la carpeta de la acción
      - name: Ejecutar saludo local
        id: saludo # ID para capturar el output después
        uses: ./.github/actions/hello-audit
        with:
          who-to-greet: "Gemini User" # Pasamos el input

      # USO DEL OUTPUT:
      - name: Mostrar el resultado
        run: echo "La acción terminó a las ${{ steps.saludo.outputs.time }}"

      # USO DE ACTION EXTERNA (Ejemplo de varios repos):
      # Si estuviera en otro repo se usaría: uses: usuario/repo-action@v1
```

---

### Resumen de Implementación Profesional

1.  **Inputs:** Úsalos para parametrizar. Evita "hardcodear" valores como nombres de carpetas o entornos.
2.  **Outputs:** Son vitales si tu acción genera un ID de despliegue, una URL o un resultado de análisis que el siguiente Job necesita.
3.  **Shell:** En las Composite Actions, no olvides poner `shell: bash` en **cada** paso `run`. Si no lo pones, la acción fallará porque GitHub no sabrá qué intérprete usar en el Runner.
4.  **Local vs Global:** Si la lógica es solo para tu proyecto, mantenla local (`./.github/...`). Si ves que la vas a copiar en 3 repositorios diferentes, muévela a un repositorio propio para seguir el principio **DRY** (Don't Repeat Yourself).
