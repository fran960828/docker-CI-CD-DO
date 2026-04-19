rol: Experto en github actions CI/CD consulta: Haz una documentación para una persona principiante con una explicación en forma de comentario al inicio y tras la explicación emplea un ejemplo sencillo para que quede más claro los siguientes conceptos:

- Explicación introductoría sobre problemas de seguridad relacionados con script injection, Malicious third-part actions y Permission Issues.
- Explicación a profundidad de script injection, como identificar vulnerabilidades y como escribir nuestro archivo yml para que este protegido frente a estos ataques ya sea a través de actions o utilizando variables de entorno.
- Explicación a profundidad del uso de actions, de las propias que represental la mayor seguridad, a las verificadas por github (seguridad intermedia), al uso de acciones no verificadas y como protegerte.
- Explicación a profundida de los permisos, usando la key permissions para establecer para un job deteminado que puede hacer (ej:issues:write).
- Explicación y uso de secrets.GITHUB_TOKEN.
- Explicación de otras consideraciones de seguridad como permitir a github actions crear y aprobar pull request.
- Explica a modo introductoría las configuraciones más importantes que puedes controlar en github relacionadas con la seguridad.
- Explicación introductoría al uso de OpenId Connect, explicando como usar permissions con id-token y contents y luego un step para obtener AWS credentials
  con uses aws-actions/configure-aws-credentials with role-to-assume y aws-region

Especificaciones:-La documentación debe contener la explicación detallada de todo lo necesario para el uso de los conceptos a nivel profesional-Los ejemplos deben estar explicados con comentarios sobre lo que hacen en cada paso -El formato de entrega será markdown. Verificación:Revisa el contenido de la consulta para obtener el resultado deseado, recuerda que lo más importante es que los ejemplos estén bien explicados , tomate el tiempo necesario para obtener el mejor resultado.

genera el markdown de la documentación para descargar pero no te dejes nada de lo que has desarrollado en el primer prompt.

genera un serie de ejercicios para practicar todos estos conceptos, estos ejercicios deben diferir de los que has puesto de ejemplo y aumentar progresivamente de dificultad hasta el punto de alcanzar un uso profesional y fluido. Genera el número de ejercicios que consideres necesario para alcanzar un buen nivel de dominio

Documenta el contenido del video realizando un explicación a detalle cada uno de los ejemplos en forma de comentario tipo py y
luego poniendo el ejemplo en formato ejecutable. Las explicaciones deben ir dirigidas a un público principiante y todo debe ir
en formato py

A continuación te paso la guia completa sobre CI avanzado, quiero que la estructures siguiendo el mismo formato y le adiciones todos los comandos que hemos visto por muy simples y obvios que parezcan explicando para que sirven cada uno de ellos, al final lo que quiero es tener un paso a paso muy claro que me sirva de base de proyectos.
