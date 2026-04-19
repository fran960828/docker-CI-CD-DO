# Guía Profesional: Creación de Custom Actions con JavaScript y Despliegue en AWS S3

> **Nota del Experto:** Las **JavaScript Actions** son la forma más robusta y profesional de extender GitHub Actions. A diferencia de las Composite Actions, permiten usar todo el ecosistema de Node.js, manejar errores de forma granular y utilizar las librerías oficiales del kit de herramientas de GitHub (`actions/toolkit`). En esta guía, construiremos una acción que despliega archivos a un bucket de AWS S3, integrando seguridad, automatización y lógica de scripting.

---

## 1. Preparación del Entorno (Node.js)

Para crear una acción de JavaScript, primero debemos inicializar un proyecto de Node.js.

- **`npm init -y`**: Crea un archivo `package.json` con valores por defecto. Es el "DNI" de nuestro proyecto donde se gestionan las dependencias.
- **Instalación de dependencias críticas**:
  - **`@actions/core`**: La librería fundamental para leer inputs, escribir outputs y mostrar mensajes en los logs.
  - **`@actions/github`**: Permite interactuar con la API de GitHub (acceder a PRs, issues, etc.).
  - **`@actions/exec`**: Facilita la ejecución de comandos de consola (como el CLI de AWS) de forma síncrona y con manejo de logs.

---

## 2. Configuración de AWS S3

Antes de programar, necesitamos un destino en la nube. Un **Bucket S3** es un contenedor virtual en AWS para almacenar archivos.

1.  Ve a la consola de **AWS S3** y pulsa "Create bucket".
2.  Dale un nombre único (ej: `mi-proyecto-deploy-2026`).
3.  Desactiva "Block all public access" si quieres que los archivos sean legibles vía URL (opcional según el caso).
4.  En el servicio **IAM**, crea un usuario con permisos `AmazonS3FullAccess` y genera las **Access Keys** (ID y Secret). Estas son las llaves que usaremos en los secretos de GitHub.

---

## 3. Desarrollo de la Acción (Lógica y Metadatos)

### El archivo `action.yml` (Definición)

Es el puente entre GitHub y tu código. Aquí definimos que usaremos Node.js y cuáles son los parámetros de entrada.

```yaml
name: "Despliegue a AWS S3"
description: "Sube archivos a un bucket de S3 usando JavaScript"

# 1. Definimos los parámetros de entrada
inputs:
  bucket:
    description: "Nombre del bucket de destino"
    required: true
  bucket-region:
    description: "Región de AWS donde reside el bucket"
    required: true
    default: "us-east-1"
  dist-folder:
    description: "Carpeta local que contiene los archivos a subir"
    required: true

# 2. Definimos la salida (Output)
outputs:
  website-url:
    description: "URL pública del bucket (si está configurado como sitio)"

# 3. Configuramos la ejecución
runs:
  using: "node20" # Usamos el motor de Node.js más reciente disponible
  main: "main.js" # Archivo de entrada de la lógica
```

### El archivo `main.js` (Lógica)

Aquí es donde ocurre la magia. Usamos `core.getInput` para leer el YAML y `exec.exec` para hablar con AWS.

```javascript
const core = require("@actions/core");
const exec = require("@actions/exec");

async function run() {
  try {
    // 1. Obtención de inputs definidos en el action.yml
    const bucket = core.getInput("bucket", { required: true });
    const region = core.getInput("bucket-region", { required: true });
    const distFolder = core.getInput("dist-folder", { required: true });

    // 2. Uso de core.notice para enviar un mensaje informativo al log de GitHub
    core.notice(
      `Iniciando despliegue de la carpeta ${distFolder} en el bucket ${bucket}...`
    );

    // 3. Ejecución del comando AWS CLI usando exec.exec
    // Este comando sincroniza la carpeta local con el bucket remoto
    await exec.exec(
      `aws s3 sync ${distFolder} s3://${bucket} --region ${region}`
    );

    // 4. Construcción dinámica de la URL de salida
    const url = `http://${bucket}.s3-website-${region}.amazonaws.com`;

    // 5. Establecemos el output para que el workflow pueda usarlo después
    core.setOutput("website-url", url);
  } catch (error) {
    // Si algo falla, marcamos el job como fallido y mostramos el error
    core.setFailed(`La acción falló con el error: ${error.message}`);
  }
}

run();
```

---

## 4. Implementación en el Workflow Principal

Para usar nuestra acción, debemos pasarle los secretos de AWS. **Nunca escribas las llaves directamente en el código.**

```yaml
name: Despliegue de Producción

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Clonar código
        uses: actions/checkout@v4

      - name: Compilar proyecto
        run: npm ci && npm run build # Asumimos que genera una carpeta 'dist'

      # EJECUCIÓN DE NUESTRA CUSTOM ACTION
      - name: Subir a S3
        id: s3-upload # ID necesario para recuperar el output
        uses: ./.github/actions/mi-js-action # Ruta a la carpeta de la acción
        with:
          bucket: "mi-proyecto-deploy-2026"
          bucket-region: "eu-west-1"
          dist-folder: "dist"
        env:
          # AWS CLI busca estas variables automáticamente para autenticarse
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      # USO DEL OUTPUT GENERADO POR LA ACTION
      - name: Mostrar URL del sitio
        run: echo "Tu sitio está vivo en ${{ steps.s3-upload.outputs.website-url }}"
```

---

### Resumen de Flujo de Datos

| Concepto    | Rol en la Acción               | Rol en el Workflow                    |
| :---------- | :----------------------------- | :------------------------------------ |
| **Inputs**  | `core.getInput('nombre')`      | Se pasan mediante `with:`             |
| **Secrets** | Usados por el comando `aws s3` | Se pasan mediante `env:`              |
| **Outputs** | `core.setOutput('key', val)`   | Se acceden vía `steps.id.outputs.key` |

> **Tip Profesional:** Cuando desarrolles JavaScript Actions, recuerda que los archivos dentro de `node_modules` no suelen subirse al repositorio. Para que tu acción funcione en GitHub sin tener que instalar dependencias, es común usar herramientas como `@vercel/ncc` para compilar todo tu código y dependencias en un único archivo `index.js`.
