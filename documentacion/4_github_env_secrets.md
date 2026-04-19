# Guía Profesional: Variables de Entorno, Secrets y Environments en GitHub Actions

> **Nota del Experto:** En el mundo del CI/CD, la gestión de la configuración es tan crítica como el código mismo. Una mala gestión de credenciales puede comprometer toda tu infraestructura. Esta guía te enseñará a separar la configuración (Variables de Entorno) de la información sensible (Secrets) y cómo aplicar capas de seguridad profesional mediante el uso de "Environments".

---

## 1. Variables de Entorno (Environment Variables)

Las variables de entorno (`env`) permiten almacenar información que tus scripts o aplicaciones necesitan, pero que no quieres escribir directamente en el código.

### Ámbitos de definición (Scopes)

Puedes definir variables en tres niveles de jerarquía:

1.  **Workflow Level:** Disponibles para todos los jobs del archivo. Se colocan al mismo nivel que `jobs:` y `on:`.
2.  **Job Level:** Disponibles solo para los steps de ese job específico.
3.  **Step Level:** Disponibles únicamente para un paso concreto.

### Sintaxis en Linux (Runner)

Dado que la mayoría de los runners usan Linux, para acceder a una variable de entorno dentro de un comando `run`, usamos el símbolo de dólar: `$NOMBRE_VARIABLE` o `${NOMBRE_VARIABLE}`.

---

## 2. Variables vs. Secrets

Es fundamental entender la diferencia para no exponer datos en los logs de GitHub:

- **Environment Variables:** Ideales para datos no sensibles (nombres de carpetas, versiones de software, URLs públicas). Son visibles en los archivos YAML y en los logs.
- **Secrets:** Ideales para datos sensibles (contraseñas, API Keys, Tokens). GitHub los **encripta** y, si intentas imprimirlos en consola, aparecerán como asteriscos `***`. Se acceden mediante el contexto `${{ secrets.NOMBRE_DEL_SECRET }}`.

---

## 3. GitHub Environments (Entornos de Repositorio)

Los **Environments** son una característica de nivel profesional que permite agrupar secrets y variables bajo un nombre (ej: `Production`, `Staging`).

Para usarlos, empleamos la keyword `environment: nombre_del_entorno` dentro de un job. Esto habilita funcionalidades avanzadas:

- **Reviewers:** El workflow se pausa hasta que una persona autorizada apruebe la ejecución.
- **Wait timer:** Retrasa la ejecución por un tiempo determinado.
- **Deployment branches:** Solo permite que el job se ejecute si el push viene de una rama específica (ej. `main`).

---

## Ejemplo Práctico: Despliegue Seguro a Base de Datos

Este ejemplo muestra cómo se heredan las variables y cómo se protegen los datos de acceso a una base de datos utilizando un entorno de producción.

```yaml
name: Despliegue con Seguridad Avanzada

# 1. Variables a nivel de WORKFLOW (Disponibles en todo el archivo)
env:
  NODE_VERSION: "20"
  APP_NAME: "mi-sistema-erp"

on:
  push:
    branches: [main]

jobs:
  deploy-production:
    # 2. Uso de ENVIRONMENT para cargar reglas de protección y secrets específicos
    # Esto buscará los secrets dentro de 'Production' en la configuración del repo
    environment: Production

    runs-on: ubuntu-latest

    # 3. Variables a nivel de JOB
    env:
      DB_HOST: "db.produccion.empresa.com"

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Configurar Entorno
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }} # Acceso a var de workflow

      - name: Ejecutar script de migración
        # 4. Variables a nivel de STEP y uso de SECRETS
        env:
          # Accedemos al secret guardado en el Environment 'Production'
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
          # Definimos una variable de paso
          LOG_LEVEL: "verbose"
        run: |
          # Sintaxis de Linux para usar las variables definidas arriba
          echo "Iniciando despliegue de $APP_NAME..."
          echo "Conectando al servidor: $DB_HOST"

          # GitHub ocultará automáticamente el valor de DB_PASSWORD en los logs
          # Aunque intentemos imprimirlo, veremos '***'
          node scripts/migrate.js --user admin --pass "$DB_PASSWORD" --log "$LOG_LEVEL"

      - name: Paso final de confirmación
        # Si intentamos usar $LOG_LEVEL aquí, fallará porque era de nivel STEP
        run: echo "Finalizado despliegue en host $DB_HOST"
```

### Resumen de acceso a datos:

| Tipo             | Definición en YAML     | Acceso en YAML/Actions | Acceso en Linux (Shell) |
| :--------------- | :--------------------- | :--------------------- | :---------------------- |
| **Variable Env** | `env: VAR: 'valor'`    | `${{ env.VAR }}`       | `$VAR`                  |
| **Secret**       | Configuración Repo/Env | `${{ secrets.KEY }}`   | (Pasar a `env` primero) |
| **Contexto**     | Automático             | `${{ github.actor }}`  | N/A                     |

> **Consejo de Experto:** Para mayor seguridad, nunca uses secrets directamente en el comando `run`. Asígnalos primero a una variable de entorno dentro del `step` (como hicimos con `DB_PASSWORD`) y luego usa la variable de Linux. Esto reduce el riesgo de exposición accidental.
