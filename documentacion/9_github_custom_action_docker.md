# Guía Profesional: Creación de Docker Actions con Python y Boto3

> **Nota del Experto:** Las **Docker Actions** son la opción más robusta cuando necesitas un control total sobre el entorno de ejecución. A diferencia de las JavaScript Actions (que dependen del runtime de Node.js del Runner), una Docker Action empaqueta su propio Sistema Operativo, lenguajes y librerías. Esto permite usar Python con herramientas potentes como `boto3` para interactuar con AWS, garantizando que la acción funcionará exactamente igual independientemente de las actualizaciones del Runner de GitHub.

---

## 1. ¿Por qué usar Docker para una Action?

- **Entorno Aislado:** Puedes usar cualquier versión de Python y cualquier dependencia de sistema (como compiladores de C o librerías gráficas) sin preocuparte por lo que hay instalado en el Runner.
- **Consistencia:** "Funciona en mi máquina y funciona en GitHub" porque la máquina _es_ el contenedor.
- **Flexibilidad:** Ideal para scripts complejos de Ciencia de Datos, DevOps o integraciones con nubes como AWS usando el SDK oficial.

---

## 2. Componentes de la Action (Estructura de Archivos)

Para esta acción de despliegue a S3, necesitaremos cuatro archivos dentro de la carpeta de nuestra acción (ej. `.github/actions/python-s3-deploy/`):

### A. El archivo de dependencias (`requirements.txt`)

Aquí listamos las librerías necesarias para que Python hable con AWS.

```text
boto3==1.28.0
botocore==1.31.0
```

### B. El script de lógica (`main.py`)

Adaptamos la lógica de JavaScript a Python. Usamos `os` para leer las variables que GitHub Actions nos pasa automáticamente como variables de entorno.

```python
import os
import boto3
from botocore.config import Config

def run():
    # 1. GitHub Actions pasa los inputs como variables de entorno con el prefijo INPUT_
    # y en mayúsculas. Ej: 'bucket' se convierte en 'INPUT_BUCKET'
    bucket_name = os.environ.get('INPUT_BUCKET')
    region = os.environ.get('INPUT_BUCKET-REGION')
    dist_folder = os.environ.get('INPUT_DIST-FOLDER')

    # Configuración de AWS desde los secretos que el workflow inyectará
    # (Boto3 busca automáticamente AWS_ACCESS_KEY_ID y AWS_SECRET_ACCESS_KEY)
    my_config = Config(region_name=region)
    s3 = boto3.resource('s3', config=my_config)

    try:
        print(f"--- Iniciando despliegue en bucket: {bucket_name} ---")

        # 2. Lógica para subir archivos (equivalente a s3 sync)
        for root, dirs, files in os.walk(dist_folder):
            for file in files:
                local_path = os.path.join(root, file)
                # Calculamos la ruta relativa para el bucket
                relative_path = os.path.relpath(local_path, dist_folder)

                print(f"Subiendo: {relative_path}")
                s3.Bucket(bucket_name).upload_file(local_path, relative_path)

        # 3. Establecer el Output para GitHub Actions
        # Para enviar un output en Docker, debemos escribir en el archivo definido en $GITHUB_OUTPUT
        output_url = f"http://{bucket_name}.s3-website-{region}.amazonaws.com"

        with open(os.environ['GITHUB_OUTPUT'], 'a') as fh:
            print(f"website-url={output_url}", file=fh)

    except Exception as e:
        print(f"Error: {str(e)}")
        exit(1) # Salida con error para marcar el paso como fallido

if __name__ == "__main__":
    run()
```

### C. El archivo de configuración de la imagen (`Dockerfile`)

Define cómo se construye el contenedor que ejecutará el script.

```dockerfile
# 1. Imagen base de Python ligera
FROM python:3.11-slim

# 2. Copiamos los archivos de nuestra acción al contenedor
COPY requirements.txt /requirements.txt
COPY main.py /main.py

# 3. Instalamos las dependencias
RUN pip install --no-cache-dir -r /requirements.txt

# 4. Definimos qué archivo se ejecutará al iniciar el contenedor
ENTRYPOINT ["python", "/main.py"]
```

### D. El manifiesto de la Action (`action.yml`)

Le dice a GitHub que esta acción no es de JavaScript ni Composite, sino de Docker.

```yaml
name: "Python S3 Deploy Docker"
description: "Despliega archivos a S3 usando Python y Boto3 dentro de Docker"

inputs:
  bucket:
    description: "Nombre del bucket"
    required: true
  bucket-region:
    description: "Región de AWS"
    required: true
    default: "us-east-1"
  dist-folder:
    description: "Carpeta local con los archivos"
    required: true

outputs:
  website-url:
    description: "URL del sitio desplegado"

runs:
  using: "docker"
  image: "Dockerfile" # GitHub construirá la imagen usando nuestro Dockerfile
```

---

## 3. Uso en el Workflow Principal

La implementación es idéntica a las anteriores, pero GitHub se encargará de hacer un `docker build` y `docker run` por debajo.

```yaml
name: Deploy con Python Docker Action

on: [push]

jobs:
  deploy-job:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Build Project
        run: mkdir dist && echo "<h1>Hola desde Python Docker Action</h1>" > dist/index.html

      # Invocamos nuestra Docker Action
      - name: Deploy a S3
        id: deploy-s3
        uses: ./.github/actions/python-s3-deploy
        with:
          bucket: "mi-bucket-python-2026"
          bucket-region: "us-east-1"
          dist-folder: "dist"
        env:
          # Boto3 leerá estas variables automáticamente dentro del contenedor
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      - name: Resultado
        run: echo "Despliegue completado en ${{ steps.deploy-s3.outputs.website-url }}"
```

---

## Resumen Técnico Profesional

1.  **Manejo de Inputs:** En Docker, GitHub Actions convierte los inputs en variables de entorno con el prefijo `INPUT_` y los nombres en mayúsculas. Python los lee con `os.environ`.
2.  **Escritura de Outputs:** A diferencia de `core.setOutput` en JS, en Docker/Python debes añadir la cadena `nombre=valor` al archivo cuya ruta está en la variable de entorno `$GITHUB_OUTPUT`.
3.  **Rendimiento:** Ten en cuenta que las Docker Actions tardan un poco más en iniciar porque GitHub debe construir la imagen (salvo que uses una imagen ya pre-construida en Docker Hub).
4.  **Seguridad:** Al usar `boto3`, las credenciales de AWS pasadas por `env` son capturadas por el SDK de Python de forma segura sin que queden registradas en los logs.
