¡Hola! Como experto en despliegue de agentes, he preparado esta guía técnica estructurada para que cualquier principiante pueda escalar desde conceptos básicos hasta la creación de herramientas personalizadas profesionales.

---

### 📝 Introducción al Despliegue de IA

> **Comentario del experto:** El mundo de la IA ha evolucionado de simples chats que responden preguntas (Modelos de Lenguaje) a entidades capaces de realizar tareas (Agentes). La clave de un buen despliegue no es solo que la IA "sepa" cosas, sino cómo interactúa con el usuario, qué herramientas tiene a su disposición y cómo limitamos su "creatividad" para que sea confiable. En esta guía aprenderás a diferenciar estas tecnologías y a dominar **Pickaxe**, una de las plataformas más potentes para crear aplicaciones de IA sin saber programar.

---

## 1. Asistentes vs. Agentes de IA

La diferencia radica principalmente en la **autonomía** y la **capacidad de ejecución**.

- **Asistente:** Es una IA reactiva. Tú le haces una pregunta y ella te da una respuesta basada en sus datos. Ejemplo: ChatGPT estándar.
- **Agente:** Es una IA proactiva. Tiene un objetivo (meta) y puede usar herramientas (navegar por internet, ejecutar código, conectar con otras apps) para cumplirlo por sí misma.

**Ejemplo:**

- _Asistente:_ "Escríbeme un correo para un cliente." (Solo escribe).
- _Agente:_ "Envía un correo al cliente X con el presupuesto adjunto y agéndame una reunión si responde." (Actúa y ejecuta).

---

## 2. Gems (Google Gemini)

Los **Gems** son versiones personalizadas del modelo Gemini de Google. Sirven para crear expertos en temas específicos sin tener que repetir las instrucciones cada vez.

**Cómo crear uno:**

1. Ve a la sección de "Gems" en la interfaz de Gemini.
2. Haz clic en **"Nuevo Gem"**.
3. Dale un nombre y escribe las instrucciones (el sistema te ayudará a refinarlas).
4. Haz clic en **Crear**.

---

## 3. GPTs (OpenAI)

Los **GPTs** son versiones personalizadas de ChatGPT diseñadas para tareas específicas o empresas.

- **Para qué sirven:** Para automatizar flujos de trabajo, analizar datos específicos o actuar como un tutor personalizado.
- **Cómo crear uno:** En ChatGPT, ve a "Explore GPTs" -> "Create".
- **Recomendación:** Puedes usar **BOB GPT** (creado por el profesor), un asistente optimizado para guiarte en tareas complejas y ayudarte a redactar prompts avanzados.

---

## 4. Diferencias: GPT vs. Pickaxe

Es vital entender dónde vive cada herramienta para elegir la correcta:

| Característica   | GPT (OpenAI)                      | Pickaxe                                         |
| :--------------- | :-------------------------------- | :---------------------------------------------- |
| **Interacción**  | Dentro de la web/app de ChatGPT.  | En tu propia web o una URL personalizada.       |
| **Programación** | "GPT Builder" (lenguaje natural). | Dashboard de Pickaxe (configuración avanzada).  |
| **Cerebro**      | GPT-4o (Exclusivo).               | Multimodelo (GPT-4o, Claude 3.5, Gemini, etc.). |

---

## 5. Studio vs. Pickaxes (En Pickaxe.it)

Dentro de la plataforma Pickaxe, existen dos niveles de organización:

- **Pickaxes:** Son los asistentes/agentes individuales que creas.
- **Studio:** Es el contenedor o "marca blanca". Sirve para agrupar varios Pickaxes en una sola página web profesional con tu propio logo y dominio.

---

## 6. Creación de un nuevo Pickaxe (Paso a Paso)

Al crear un Pickaxe, configuramos el "corazón" de la IA:

1.  **Prompt (System Instructions):** Usa **BOB GPT** para generar un prompt detallado que defina el rol, tono y límites de tu IA.
2.  **Modelo:** Elige el "cerebro" (ej. GPT-4o para razonamiento complejo).
3.  **Capacidades:** Activa interruptores para permitirle:
    - Adjuntar archivos.
    - Generar imágenes (DALL-E).
    - Escribir/Ejecutar código.
4.  **Ice Breakers:** Son las burbujas de texto iniciales (ej. "¿Cómo puedo ayudarte hoy?") para que el usuario no empiece de cero.
5.  **Introducción:** El mensaje de bienvenida que explica qué hace la IA.
6.  **Model Reminder:** Instrucciones invisibles que se añaden al final de cada prompt del usuario para asegurar que la IA nunca olvide sus reglas básicas (ej: "Siempre responde en español y sé breve").

---

## 7. Edición y Ajustes Técnicos

Aquí personalizamos la experiencia visual y el consumo de recursos:

- **Avatar y Placeholder:** Sube una imagen para la IA y define el texto gris que aparece en la barra de escritura.
- **Tokens (Input/Output):** \* _Input:_ Cuánta info puede leer de una vez.
  - _Output:_ Longitud máxima de la respuesta.
  - _Memoria:_ Cuántos mensajes previos recordará de la conversación actual.

---

## 8. Training Dialogue y Creatividad

- **Training Dialogue:** Introduces ejemplos reales de "Pregunta del usuario" -> "Respuesta ideal de la IA". Esto obliga a la IA a seguir un patrón específico.
- **Creatividad (Temperature):** \* _Baja (0.1 - 0.3):_ Respuestas deterministas y serias (ideal para datos técnicos).
  - _Alta (0.7 - 1.0):_ Respuestas originales y variadas (ideal para marketing).

---

## 9. Knowledge (Base de Conocimientos)

Es la "mochila" de información del agente.

1.  **Subir archivos:** PDF, .txt, .csv.
2.  **Importancia:** Ajustas cuánto debe priorizar el documento frente a su conocimiento general.
3.  **Contexto del documento:** Una breve nota explicándole a la IA qué hay en ese archivo (ej: "Este es el manual de ventas 2024").

---

## 10. Actions (Herramientas Externas)

Las **Actions** convierten al asistente en un agente. Puedes conectar:

- **Google Search:** Para que la IA busque información en tiempo real.
- **Image Gen:** Para crear visuales bajo demanda.
- **Integraciones:** Enviar datos a hojas de cálculo o CRMs.

---

## 11. Diseño de la Web y Tokens Gratis

Pickaxe permite personalizar el front-end:

- **Diseño:** Cambia colores, tipografía y disposición de los elementos para que parezca tu propio producto de software.
- **Tokens gratuitos:** Puedes configurar que cada usuario nuevo tenga, por ejemplo, **10 mensajes gratis** antes de que tengan que pagar o registrarse, ideal para captar leads.

---

# 🚀 EJEMPLO PRÁCTICO: "Agente de Soporte Técnico"

A continuación, un ejemplo comentado de cómo configurarías este agente en Pickaxe:

```markdown
### CONFIGURACIÓN DEL AGENTE DE SOPORTE

# 1. EL PROMPT (Generado por BOB GPT)

# "Eres un experto en soporte técnico de servidores. Tu tono es profesional y calmado.

# Si no sabes la respuesta basándote en el 'Knowledge', di que escalarás el ticket."

# 2. MODEL REMINDER (Lo que el usuario no ve)

# "Recordatorio: Nunca menciones que eres una IA de OpenAI o Pickaxe. Di que eres el Asistente de IT de la empresa."

# 3. TRAINING DIALOGUE (Entrenamiento de ejemplo)

# User: "No puedo entrar al correo"

# Assistant: "Entiendo. Por favor, confirma si has verificado que tu contraseña no ha expirado en el portal corporativo."

# 4. KNOWLEDGE (Archivo cargado)

# Documento: "manual_servidores_v2.pdf"

# Contexto: "Este manual contiene todos los códigos de error posibles del servidor central."

# 5. AJUSTES DE TOKENS Y CREATIVIDAD

# Output Tokens: 500 (para respuestas concisas)

# Creatividad: 0.2 (Queremos precisión técnica, no inventos)

# 6. ACTIONS ACTIVADAS

# - Internet Search: Activado (Para buscar errores de software conocidos en foros)

# - Code Interpreter: Desactivado (No lo necesitamos para soporte básico)
```
