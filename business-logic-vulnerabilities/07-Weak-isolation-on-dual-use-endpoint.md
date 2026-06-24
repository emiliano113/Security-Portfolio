# Weak Isolation on Dual-Use Endpoint

**Categoría:** Business Logic Vulnerabilities
**Nivel:** Practitioner
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Aplicación con un endpoint de cambio de contraseña que hace suposiciones incorrectas sobre el nivel de privilegio del usuario en base a los parámetros que recibe. El objetivo es acceder a la cuenta del administrador y eliminar al usuario `carlos`.

## Vulnerabilidad

El endpoint `POST /my-account/change-password` utiliza el parámetro `username` para determinar qué cuenta se modifica, y el parámetro `current-password` para verificar la identidad del solicitante. El fallo está en que la aplicación no valida correctamente la presencia de `current-password`: si se elimina ese parámetro de la petición, el servidor igual procesa el cambio de contraseña sin exigir verificación previa. Esto permite modificar la contraseña de cualquier cuenta, incluyendo la del administrador.

## Proceso de explotación

1. Inicié sesión con las credenciales provistas (`wiener:peter`) y accedí a la página de cambio de contraseña.
2. Con Burp Suite interceptando el tráfico, realicé un cambio de contraseña normal y capturé la solicitud `POST /my-account/change-password`. La envié a Burp Repeater para analizarla.
3. En Repeater, eliminé por completo el parámetro `current-password` de la petición y cambié el valor del parámetro `username` de `wiener` a `administrator`.
4. Envié la solicitud modificada con una nueva contraseña de mi elección. El servidor la procesó sin exigir la contraseña actual.
5. Cerré sesión e inicié sesión nuevamente con `administrator` y la contraseña que acababa de establecer.
6. Accedí al panel de administración y eliminé al usuario `carlos`, resolviendo el laboratorio.

## Conclusión

Este lab muestra un caso de **aislamiento débil en un endpoint de doble uso**: el mismo endpoint sirve tanto para usuarios comunes como para administradores, pero la lógica de autorización depende de parámetros que el cliente puede manipular o eliminar. La lección clave es que los controles de seguridad críticos (como verificar la identidad antes de cambiar una contraseña) nunca deben depender de que un parámetro esté presente en la petición del cliente — si ese parámetro falta, el servidor debe rechazar la operación, no ignorar el control.
