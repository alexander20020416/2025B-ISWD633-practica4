# Reflexión sobre los Aprendizajes

## Conocimientos Previos

Antes de realizar esta práctica, tenía conocimientos básicos sobre Docker y sabía crear contenedores simples, pero no entendía bien cómo funcionaban las políticas de reinicio ni por qué eran importantes. Pensaba que simplemente con crear y ejecutar un contenedor era suficiente.

## Principales Aprendizajes

Durante la práctica aprendí varias cosas importantes. Primero, entendí que las políticas de reinicio son fundamentales para mantener servicios en producción. Antes no sabía que existía la diferencia entre `always` y `unless-stopped`, y ahora me queda claro que si detengo manualmente un contenedor con `unless-stopped`, Docker respeta esa decisión incluso después de reiniciarse. Esto es útil cuando estoy trabajando en desarrollo y no quiero que un contenedor se levante automáticamente.

También aprendí sobre el sistema de capas de Docker. Al principio no entendía por qué la segunda construcción de la imagen era mucho más rápida que la primera. Ahora sé que Docker cachea las capas que no cambian y solo reconstruye lo necesario. Esto me ayudará a optimizar mis Dockerfiles en el futuro, poniendo las instrucciones que cambian frecuentemente al final del archivo.

Otro aprendizaje importante fue sobre los healthchecks. No sabía que Docker podía monitorear automáticamente si un servicio está funcionando correctamente. Configurar el healthcheck con curl me ayudó a entender cómo Docker determina si un contenedor está "healthy" o no, y ahora puedo usar esto para crear aplicaciones más robustas.

## Problemas Encontrados y Soluciones

En cuanto a los problemas que enfrenté, el principal fue cuando intenté construir la imagen de Apache con CentOS 7. Me salió un error diciendo que no podía conectarse a los repositorios de CentOS. Después de investigar descubrí que CentOS 7 llegó al final de su vida útil y tuve que modificar el Dockerfile para usar los repositorios archivados (vault.centos.org). Esto me enseñó que las tecnologías evolucionan y hay que estar preparado para adaptar el código.

Otro problema fue cuando olvidé instalar curl en la imagen y el healthcheck fallaba. Al principio no entendía por qué, pero luego me di cuenta que las imágenes base no incluyen todas las herramientas y hay que instalar lo que necesitas. Aprendí a reconstruir la imagen con `--no-cache` para forzar la instalación de curl.

También tuve confusión con los límites de memoria. Cuando vi que tenía que calcular cuánta memoria swap estaba disponible, al inicio no entendía la fórmula. Ahora me queda claro que el parámetro `--memory-swap` incluye tanto RAM como swap, y para saber la swap disponible simplemente resto la RAM del total.

## Aplicación Práctica

Lo más valioso de esta práctica fue entender que Docker no es solo para "empaquetar aplicaciones". Es una herramienta completa para gestionar el ciclo de vida de los contenedores con políticas de reinicio, límites de recursos y monitoreo de salud. Esto me va a servir mucho cuando trabaje en proyectos reales donde la disponibilidad y el uso eficiente de recursos son importantes.

## Beneficio para mi Formación Profesional

Para mi formación profesional, estos conocimientos son fundamentales porque en la industria se usan contenedores en producción y necesito saber cómo configurarlos correctamente. Ahora entiendo mejor por qué las empresas usan Docker y cómo puedo aplicarlo en proyectos futuros de manera profesional.

Específicamente, los aprendizajes que considero más valiosos son:

- Comprender cómo funcionan las políticas de reinicio y cuándo usar cada una
- Saber optimizar la construcción de imágenes aprovechando el sistema de capas
- Poder configurar healthchecks para monitorear servicios automáticamente
- Entender cómo limitar recursos (CPU y memoria) para evitar que un contenedor consuma todo el sistema
- Solucionar problemas comunes como repositorios obsoletos o dependencias faltantes

Estos conocimientos me dan una base sólida para trabajar con contenedores en entornos profesionales y me preparan para tecnologías relacionadas como Kubernetes que también se usan en la industria.
