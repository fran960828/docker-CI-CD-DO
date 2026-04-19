# Guía de Gestión de Datos en GitHub Actions: Artifacts, Outputs y Caching

> **Nota del Experto:** En un entorno CI/CD profesional, los Jobs suelen ejecutarse en máquinas virtuales totalmente aisladas. Esto significa que si el `Job A` compila un archivo, el `Job B` no podrá verlo a menos que usemos mecanismos de transferencia. Aquí es donde entran los **Artifacts** (para archivos pesados), los **Outputs** (para variables pequeñas) y el **Caching** (para acelerar la ejecución reutilizando dependencias).

---

## 1. Intercambio de Archivos con Artifacts

Los **Job Artifacts** son archivos (como binarios, carpetas `dist` o reportes de test) que se guardan en los servidores de GitHub tras finalizar un Job. Se usan para persistir datos después de que el Runner se destruye o para pasar datos entre Jobs.



### Upload: `actions/upload-artifact@v4`
Permite "subir" archivos al almacenamiento de GitHub.
* **name:** Es el identificador del paquete (puedes elegirlo tú).
* **path:** La ruta del archivo o carpeta que quieres salvar.

### Download: `actions/download-artifact@v4`
Permite "bajar" esos archivos en un Job distinto.
* **Requisito:** El Job que descarga debe tener la keyword `needs: [nombre_del_job_anterior]` para asegurar que el archivo ya existe.

---

## 2. Comunicación de Variables con Job Outputs

A diferencia de los Artifacts (que son archivos), los **Outputs** se usan para pasar pequeñas cadenas de texto o valores entre Jobs.

### El flujo del Output (Explicación técnica)
Para que un Job comparta un dato, necesitamos tres niveles de configuración:
1.  **El ID del Step:** El paso que genera el dato debe tener un `id` único (ej. `publish`).
2.  **La escritura en `$GITHUB_OUTPUT`:** Se usa un comando de shell para escribir el par clave=valor en un archivo especial del sistema.
3.  **Declaración del Job Output:** Al nivel del Job, se debe exponer ese valor para que otros Jobs lo vean.

**¿Puedes elegir el nombre a voluntad?** ¡Sí! En `script-file: ${{ steps.publish.outputs.script-file }}`, el nombre a la izquierda del colon es el nombre "público" que verá el resto del workflow, y el de la derecha es el nombre que capturaste en el step.

---

## 3. Optimización con Caching Dependencies

El **Caching** no sirve para pasar datos entre jobs, sino para **ahorrar tiempo**. En lugar de descargar los 500MB de `node_modules` en cada ejecución, los guardamos en una memoria especial de GitHub.

### `actions/cache@v3`
* **path:** Carpeta que queremos cachear (ej. `~/.npm`).
* **key:** Una huella digital única. Si el archivo `package-lock.json` cambia, la función `hashFiles` generará una llave distinta, invalidando la caché anterior y forzando una descarga limpia.

---

## Ejemplo Práctico Profesional

Este workflow compila una aplicación, extrae el nombre de un archivo generado dinámicamente y lo pasa a un job de despliegue, todo mientras usa caché para ir más rápido.

```yaml
name: Flujo de Gestión de Datos Avanzado

on:
  push:
    branches: [main]

jobs:
  # --- JOB DE CONSTRUCCIÓN ---
  build:
    runs-on: ubuntu-latest
    # Declaramos qué valores de este job estarán disponibles para otros
    outputs:
      # 'script-file' es el nombre que elegimos a voluntad
      # Lo tomamos del step con id 'publish'
      script-file: ${{ steps.publish.outputs.selected-file }}

    steps:
      - name: Checkout código
        uses: actions/checkout@v4

      # CONFIGURACIÓN DE CACHÉ
      - name: Cachear dependencias de Node
        uses: actions/cache@v3
        with:
          path: ~/.npm # Ruta donde npm guarda su caché
          # La 'key' cambia si el package-lock.json cambia
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-

      - name: Instalar dependencias
        run: npm ci

      - name: Compilar proyecto
        run: npm run build # Esto genera una carpeta 'dist/'

      # EXTRACCIÓN DEL OUTPUT (Detalle técnico solicitado)
      - name: Publicar nombre de archivo JS
        id: publish # ID necesario para referenciarlo en el 'output' del Job
        # Este comando busca un archivo .js en assets y lo guarda en la variable $GITHUB_OUTPUT
        run: |
          FILE_NAME=$(find dist/assets/*.js -type f -execdir echo {} \;)
          echo "selected-file=$FILE_NAME" >> $GITHUB_OUTPUT

      # CREACIÓN DE ARTIFACT
      - name: Guardar carpeta dist
        uses: actions/upload-artifact@v4
        with:
          name: dist-files # Nombre de la carpeta en GitHub
          path: dist       # Subimos toda la carpeta compilada

  # --- JOB DE DESPLIEGUE ---
  deploy:
    # Necesita que 'build' termine y acceso a sus outputs
    needs: build
    runs-on: ubuntu-latest
    steps:
      # DESCARGA DE ARTIFACT
      - name: Obtener archivos compilados
        uses: actions/download-artifact@v4
        with:
          name: dist-files # Debe coincidir con el nombre definido arriba

      # USO DEL JOB OUTPUT
      - name: Mostrar nombre del archivo recibido
        # Accedemos mediante: needs.<job_id>.outputs.<output_name>
        run: echo "El archivo detectado en el build fue ${{ needs.build.outputs.script-file }}"

      - name: Simular Despliegue
        run: ls -R
```

### Diferencias Clave para no olvidar:

| Característica | **Artifacts** | **Job Outputs** | **Caching** |
| :--- | :--- | :--- | :--- |
| **Tipo de dato** | Archivos y carpetas. | Pequeños Strings (variables). | Dependencias (node_modules, etc). |
| **Propósito** | Compartir resultados de compilación. | Compartir IDs, nombres o versiones. | Ahorrar tiempo de instalación. |
| **Persistencia** | Se pueden descargar desde la UI de GitHub. | Solo existen durante el workflow. | Se guardan por 7 días (o hasta 10GB). |

> **Tip Pro:** Siempre usa la versión más reciente de las acciones (`@v4` en lugar de `@v3` si está disponible) para obtener mejoras de rendimiento y seguridad en la carga de archivos.