# Guía de Seguridad Profesional en GitHub Actions

> **Nota del Experto:** La automatización es una espada de doble filo. Mientras acelera tu desarrollo, un workflow mal configurado puede convertirse en una puerta trasera para atacantes. La seguridad en CI/CD no es una opción, es una base. En esta guía aprenderás a mitigar los tres pilares de riesgo: la ejecución de código malicioso (Injection), la cadena de suministro (Third-party actions) y el exceso de privilegios (Permissions).

---

## 1. Riesgos Críticos de Seguridad

Antes de blindar nuestro código, debemos entender a qué nos enfrentamos:

- **Script Injection:** Ocurre cuando un atacante usa datos que tú no controlas (como el título de una Issue o el nombre de una rama) para inyectar comandos maliciosos en tus pasos `run`.
- **Malicious Third-party Actions:** El uso de acciones de autores desconocidos puede exponer tus secretos o código, ya que esa acción tiene acceso al entorno de ejecución.
- **Permission Issues:** Otorgar permisos de "escritura" a workflows que solo necesitan "lectura" permite que, en caso de compromiso, el atacante pueda borrar repositorios o inyectar código en producción.

---

## 2. Protección contra Script Injection

El error más común es usar expresiones de GitHub directamente en un script de shell:
`run: echo "El título es ${{ github.event.issue.title }}"`

Si un atacante crea una issue con el título: `"; rm -rf / #`, tu comando se convertiría en un borrado masivo.

### ¿Cómo protegerse?

La regla de oro es **nunca confiar en el contexto del evento directamente en el shell**.

1.  **Usa Variables de Entorno:** Pasa el dato a una variable de entorno. El shell la tratará como un valor literal, no como código ejecutable.
2.  **Usa Actions específicas:** Las acciones manejan estos datos de forma segura internamente.

---

## 3. Seguridad en la Cadena de Suministro (Actions)

No todas las acciones tienen el mismo nivel de confianza:

- **Acciones Propias/Oficiales:** (Ej: `actions/checkout`). Son las más seguras, mantenidas por GitHub.
- **Verified Creators:** Acciones con el check azul. GitHub ha verificado la identidad del autor (ej: AWS, Docker, HashiCorp). Seguridad alta/intermedia.
- **Acciones no verificadas:** Cualquier usuario puede subirlas. **Peligro alto**.
  - _Protección:_ Usa el **Commit SHA** (ej: `uses: action@d12345...`) en lugar del tag de versión (`@v1`), ya que los tags pueden ser movidos por un atacante.

---

## 4. El Principio de Menor Privilegio (Permissions)

Por defecto, el `GITHUB_TOKEN` (un token temporal que GitHub crea para cada ejecución) puede tener permisos excesivos. Debemos limitarlos usando la clave `permissions`.

### `secrets.GITHUB_TOKEN`

Es una credencial generada automáticamente. Es segura porque expira cuando termina el job, pero su poder debe ser restringido manualmente en el YAML.

---

## 5. OpenID Connect (OIDC) y AWS

La forma profesional de conectar con la nube (AWS, Azure, GCP) no es usando secretos de larga duración (Access Keys), sino **OIDC**.
Esto permite que GitHub Actions "asuma un rol" en AWS temporalmente sin guardar contraseñas permanentes en GitHub.

---

## Ejemplo Práctico: Workflow Blindado y Seguro

Este ejemplo combina protección contra inyección, limitación de permisos y conexión segura a AWS mediante OIDC.

```yaml
name: Security-First Pipeline

on:
  issues:
    types: [opened] # Evento que trae datos externos (potencial inyección)

# 1. Configuración de seguridad a nivel de Workflow
# IMPORTANTE: Configuraciones de seguridad en la UI de GitHub:
# - 'Require approval for all outside collaborators' (Para evitar ejecución de código malicioso en Forks)
# - 'Read-only personal access token' (Restringir tokens personales)

permissions:
  contents: read # Solo lectura del código
  id-token: write # REQUERIDO para OpenID Connect (OIDC)
  issues: write # Permiso específico para comentar en la issue

jobs:
  secure-process:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4 # Uso de acción oficial con versión fija

      # 2. PROTECCIÓN CONTRA SCRIPT INJECTION
      - name: Handle Issue Title Safely
        env:
          # Pasamos el dato a una variable de entorno de forma segura
          # NUNCA usar ${{ github.event... }} directamente en el 'run'
          ISSUE_TITLE: ${{ github.event.issue.title }}
        run: |
          echo "Procesando issue de forma segura..."
          # Aquí el shell trata a $ISSUE_TITLE como texto plano, no ejecutable
          echo "El título es: $ISSUE_TITLE"

      # 3. CONEXIÓN SEGURA A AWS (OIDC)
      # No usamos secretos de usuario/password, usamos Roles temporales
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::1234567890:role/my-github-role
          aws-region: us-east-1

      - name: List S3 Buckets
        run: aws s3 ls # Esto funciona gracias al token temporal de OIDC


  # 4. CONSIDERACIONES ADICIONALES
  # En los settings del repo:
  # - Settings > Actions > General: "Allow GitHub Actions to create and approve pull requests"
  #   Debe estar DESACTIVADO por defecto a menos que sea estrictamente necesario
  #   para evitar que un proceso comprometido apruebe sus propios cambios maliciosos.
```

### Configuraciones de Seguridad en la Interfaz de GitHub (Resumen)

Para un control profesional, revisa estos puntos en la configuración de tu repositorio:

1.  **Workflow permissions:** Cambia el default a "Read repository contents and packages permissions".
2.  **Fork pull request workflows:** Configura "Require approval for first-time contributors" para evitar ataques de minería de criptomonedas o robo de secretos en PRs externas.
3.  **Allowed actions:** Restringe el uso solo a acciones oficiales o de creadores verificados si el entorno es crítico.
