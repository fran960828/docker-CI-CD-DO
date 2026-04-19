# Guía Maestra de GitHub Actions: Lógica, Estrategias y Reutilización

> **Nota del Experto:** Dominar la lógica condicional y la reutilización es lo que separa un flujo de trabajo lineal y rígido de un sistema de CI/CD profesional y resiliente. En esta guía aprenderás a manejar errores, ejecutar pruebas en múltiples entornos simultáneamente y crear flujos modulares que pueden ser compartidos entre múltiples proyectos.

---

## 1. Condicionales y Manejo de Errores

GitHub Actions nos permite controlar el flujo mediante la sentencia `if`. Esta puede aplicarse tanto a **Jobs** como a **Steps**.

### Uso de `if`, `outcome` y `conclusion`

Cuando un step tiene un `id`, podemos consultar su resultado:

- **outcome:** El resultado del step antes de considerar `continue-on-error`.
- **conclusion:** El resultado final del step.
- **Valores de comparación:** `success`, `failure`, `cancelled`, `skipped`.

### Funciones de Estado

Por defecto, si un paso falla, los siguientes no se ejecutan. Para cambiar esto usamos:

- `success()`: (Por defecto) Corre si los pasos anteriores tuvieron éxito.
- `failure()`: Corre solo si un paso anterior falló. Ideal para notificaciones de error.
- `always()`: Corre siempre, incluso si hubo cancelaciones o fallos. Útil para limpieza de datos.
- `cancelled()`: Corre solo si el workflow fue detenido manualmente.

### `continue-on-error` vs `if`

- **`continue-on-error: true`**: El step puede fallar, pero el Job se marcará como "pasado" (verde). Útil para tests experimentales o linters que no deben bloquear el despliegue.
- **`if: failure()`**: Se usa para ejecutar un paso específico _porque_ algo falló (ej: enviar un Slack).

---

## 2. Estrategia de Matriz (`matrix`)

La keyword `strategy` con `matrix` permite ejecutar un mismo Job múltiples veces con diferentes configuraciones (versiones de Node, SO, etc.) en paralelo.

- **`include`**: Añade una combinación específica o una variable extra a una combinación existente.
- **`exclude`**: Elimina una combinación específica que no queremos probar (ej: Node 14 en Windows).

---

## 3. Workflows Reutilizables (Reusable Workflows)

Permiten llamar a un flujo de trabajo desde otro, evitando duplicar código.

- **Trigger:** Se activan con `on: workflow_call`.
- **`inputs`**: Parámetros que el flujo hijo recibe del padre (`description`, `required`, `default`).
- **`secrets`**: Deben pasarse explícitamente o usar `secrets: inherit`.
- **`outputs`**: Valores que el flujo hijo devuelve al padre.

---

## Ejemplo Práctico: Workflow de Pruebas y Despliegue Modular

### Parte A: El Workflow Reutilizable (`.github/workflows/reusable-deploy.yml`)

```yaml
name: Reusable Deploy

on:
  workflow_call:
    # Definimos qué necesita este flujo para funcionar
    inputs:
      environment:
        required: true
        type: string
        description: "Entorno de destino"
    secrets:
      deploy_token:
        required: true

    # Definimos qué devuelve este flujo
    outputs:
      status:
        description: "Estado del despliegue"
        value: ${{ jobs.deploy.outputs.deploy_status }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    outputs:
      deploy_status: ${{ steps.set-output.outputs.result }}
    steps:
      - name: Desplegar
        run: echo "Desplegando en ${{ inputs.environment }} con token ${{ secrets.deploy_token }}"

      - id: set-output
        run: echo "result=success" >> $GITHUB_OUTPUT
```

---

### Parte B: Workflow Principal (`.github/workflows/main.yml`)

```yaml
name: Main Pipeline Profesional

on:
  push:
    branches: [main]

jobs:
  # --- USO DE MATRIX ---
  tests:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20]
        # Excluimos una combinación específica
        exclude:
          - os: windows-latest
            node-version: 18
        # Añadimos una variable extra solo para una combinación
        include:
          - os: ubuntu-latest
            node-version: 22
            experimental: true

    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4

      # CONDICIONAL DE CACHE
      - name: Cache dependencies
        id: cache-npm
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

      - name: Install
        # cache-hit devuelve true si encontró una coincidencia exacta
        if: steps.cache-npm.outputs.cache-hit != 'true'
        run: npm ci

      - name: Run Tests
        id: run-tests
        # Si falla, el job sigue adelante pero este step se marca rojo
        continue-on-error: true
        run: npm test

      # CONDICIONAL BASADO EN EL STEP DE TEST
      - name: Report Error
        # Solo corre si los tests fallaron (outcome)
        if: steps.run-tests.outcome == 'failure'
        run: echo "Los tests fallaron, pero continuamos porque es experimental."

  # --- JOB CONDICIONAL Y REUTILIZABLE ---
  deploy:
    # Solo corre si 'tests' terminó con éxito y estamos en main
    needs: [tests]
    if: github.ref == 'refs/heads/main' && success()

    # Llamada al workflow reutilizable
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: "production"
    secrets:
      deploy_token: ${{ secrets.PROD_TOKEN }}

  # --- JOB DE NOTIFICACIÓN FINAL ---
  notify:
    needs: [deploy]
    runs-on: ubuntu-latest
    # Se ejecuta siempre, pase lo que pase con los jobs anteriores
    if: always()
    steps:
      - name: Final Summary
        run: |
          echo "Estado del despliegue: ${{ needs.deploy.outputs.status }}"
          echo "Pipeline finalizado."
```

---

### Puntos Clave para el Aprendiz:

1.  **`cache-hit`**: Es un booleano. Si es `'true'`, significa que GitHub encontró exactamente la misma carpeta y no necesitas descargarla de nuevo.
2.  **`outcome` vs `conclusion`**: Si usas `continue-on-error: true`, el `outcome` será `failure`, pero la `conclusion` será `success`. Usa `outcome` para decisiones lógicas internas.
3.  **Lógica en Reutilizables**: Al usar `workflow_call`, el flujo hijo se comporta como si sus pasos estuvieran escritos directamente en el padre, pero manteniendo su propio aislamiento de variables.
4.  **`always()`**: Es vital para pasos de limpieza (ej. borrar base de datos temporal) o logs de auditoría que deben ejecutarse incluso si el usuario cancela el workflow.
