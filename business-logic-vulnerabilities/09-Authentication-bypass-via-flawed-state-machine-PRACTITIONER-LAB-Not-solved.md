# Authentication Bypass via Flawed State Machine

**Categoría:** Business Logic Vulnerabilities
**Nivel:** Practitioner
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Aplicación con un proceso de login multifase que incluye un paso intermedio de selección de rol. El objetivo es bypassear ese paso para acceder al panel de administración y eliminar al usuario `carlos`.

## Vulnerabilidad

El servidor asigna el rol de `administrator` de forma provisional inmediatamente después del login exitoso, antes de que el usuario complete el paso de selección de rol. Si ese paso intermedio nunca se ejecuta, el servidor mantiene el rol asignado por defecto (administrador), permitiendo al atacante heredar privilegios elevados sin haberlos solicitado explícitamente.

## Proceso de explotación

1. Intenté acceder directamente a `/admin` agregándolo al final de la URL del lab, pero el servidor lo rechazó sin estar autenticado.
2. Con Burp Suite interceptando el tráfico, inicié el proceso de login con las credenciales provistas (`wiener:peter`).
3. Tras el `POST /login` exitoso, el servidor generó la solicitud `GET /role-selector` para que el usuario elija su rol. En lugar de forwardearla, la **dropeé** directamente desde Burp, evitando que el navegador enviara esa restricción de privilegios.
4. Navegué manualmente a `/admin`. Esta vez el servidor me dio acceso completo, ya que mi sesión mantenía el rol de administrador asignado por defecto tras el login.
5. Desde el panel de administración, eliminé al usuario `carlos`, resolviendo el laboratorio.

## Conclusión

Este lab demuestra una falla de omisión de autenticación multifase: el error de diseño está en que el servidor asigna el rol de `administrator` de forma provisional inmediatamente después del login exitoso, antes de que el usuario elija su rol real. Al interceptar y descartar la solicitud del selector de roles, el navegador nunca envía la restricción de privilegios, permitiendo al atacante navegar directamente a `/admin` manteniendo los permisos heredados por defecto. Los flujos de autenticación multifase deben validar que **todos** los pasos fueron completados antes de establecer el rol definitivo del usuario, nunca asumir un rol privilegiado como estado intermedio.
