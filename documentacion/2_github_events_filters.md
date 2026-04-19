# Guía Avanzada de GitHub Actions: Control de Eventos y Filtros

> **Nota del Experto:** Dominar GitHub Actions no es solo saber ejecutar comandos, sino saber **cuándo** ejecutarlos. Por defecto, un evento como `pull_request` dispara el flujo en situaciones que quizás no necesites (como cambiar el título de la PR). El uso de `types` y `filters` es lo que diferencia a un principiante de un profesional, permitiendo ahorrar minutos de ejecución y evitar ruidos innecesarios en el pipeline.

---

## 1. Activity Types y la Keyword `types`

Cada evento en GitHub puede tener múltiples "sub-eventos" llamados **Activity Types**. Si solo pones `on: pull_request`, GitHub asume unos tipos por defecto, pero puedes ser mucho más específico.

- **Pull Request Activity Types:** Por defecto, una PR se dispara cuando es `opened`, `reopened` o `synchronize` (cuando subes nuevos commits a la rama de la PR).
- **Tipos comunes:** \* `closed`: Útil para limpiar entornos de prueba.
  - `labeled`: Para disparar flujos si alguien añade una etiqueta específica.
  - `review_requested`: Para notificar a sistemas externos.

Para configurarlos, usamos la keyword `types`:

```yaml
on:
  pull_request:
    types: [opened, reopened, synchronize, labeled]
```

---

## 2. Event Filters: `branches` y `paths`

Los filtros permiten que el workflow solo se ejecute si los cambios ocurren en ciertos archivos o ramas. Se aplican principalmente a eventos de código como `push` y `pull_request`.

### Filtros de Ramas (`branches` / `branches-ignore`)

- **`branches`**: El workflow solo corre si el evento ocurre en las ramas listadas.
- **`branches-ignore`**: El workflow corre en todas las ramas **excepto** en las listadas.
- _Nota:_ No puedes usar ambos en el mismo evento.

### Filtros de Rutas (`paths` / `paths-ignore`)

- **`paths`**: Ideal para monorepos. Si solo cambias archivos en la carpeta `/docs`, ¿para qué ejecutar los tests de `/backend`?
- **`paths-ignore`**: Útil para ignorar cambios en el `README.md` o archivos de documentación que no afectan la lógica del programa.

---

## 3. Seguridad y Pull Requests de Forks

Por seguridad, los **Pull Requests que provienen de un Fork** (repositorios externos que no controlas) deben manejarse con extrema precaución.

> **Advertencia Profesional:** Nunca permitas que un workflow con acceso a secretos de producción se ejecute automáticamente ante una PR de un fork. Un atacante podría enviar código malicioso que lea tus secretos (como tokens de AWS o bases de datos) y los envíe a su propio servidor. Por defecto, GitHub limita los permisos en estos casos, pero es una buena práctica filtrar o requerir aprobación manual.

---

## 4. Control desde el Commit (Skip & Cancel)

A veces haces un cambio pequeño (corregir una errata) y no quieres gastar minutos de servidor. GitHub permite "saltar" la ejecución si incluyes palabras específicas entre corchetes en el mensaje de tu commit:

- `[skip ci]`
- `[ci skip]`
- `[no ci]`
- `[skip actions]`
- `[actions skip]`

Si el mensaje del commit contiene cualquiera de estas cadenas, GitHub Actions ignorará ese push.

---

## Ejemplo Práctico: Workflow con Control de Flujo Profesional

Este ejemplo muestra cómo filtrar ejecuciones para ahorrar recursos y asegurar que solo probamos lo que realmente ha cambiado.

```yaml
name: Workflow Optimizado de CI

on:
  # Configuración detallada para Push
  push:
    # Solo se dispara si el push es a la rama principal
    branches:
      - main
    # PERO, no se dispara si solo cambiamos la documentación
    paths-ignore:
      - "docs/**"
      - "*.md"

  # Configuración detallada para Pull Request
  pull_request:
    # Definimos tipos de actividad específicos
    # No queremos que corra si solo se editó el comentario de la PR
    types: [opened, synchronize, reopened]

    # Solo si el destino de la PR es la rama develop
    branches:
      - develop

    # Solo si hay cambios en la carpeta del servidor
    paths:
      - "src/server/**"
      - "package.json"

jobs:
  test:
    name: Running Sensitive Tests
    runs-on: ubuntu-latest

    # Verificación de seguridad básica para forks
    # Solo ejecutamos si el autor no es un bot o validamos el origen
    if: github.event.pull_request.head.repo.full_name == github.repository

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Install Dependencies
        run: npm ci

      - name: Run Tests
        run: npm test

      - name: Log Success
        # Usamos expresiones para mostrar información del evento
        run: |
          echo "Evento disparado por: ${{ github.event_name }}"
          echo "Rama origen: ${{ github.head_ref }}"
```

### Resumen de uso de este ejemplo:

1.  **Filtro de archivos:** Si subes un cambio al archivo `docs/manual.md`, este workflow **no** se activará gracias a `paths-ignore`.
2.  **Filtro de ramas:** Si creas una rama llamada `fix-typo` y haces push, no pasará nada a menos que sea `main`.
3.  **Seguridad:** El `if` en el job `test` asegura que si alguien hace un fork de tu proyecto y envía una PR, el código no se ejecute automáticamente (protegiendo tus recursos y entorno).
4.  **Omisión manual:** Si haces un commit con el mensaje `docs: corregir ortografía [skip ci]`, GitHub ignorará el evento por completo.
