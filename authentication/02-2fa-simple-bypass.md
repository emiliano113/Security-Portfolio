# 2FA Simple Bypass

**Categoría:** Authentication
**Nivel:** Apprentice
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Aplicación con autenticación de dos factores (2FA) que puede ser bypasseada 
sin necesidad del código de verificación. El objetivo es acceder a la cuenta 
de `carlos` sin tener acceso a su código 2FA.

## Vulnerabilidad

La aplicación autentica parcialmente al usuario después del primer factor 
(usuario y contraseña), antes de que complete el segundo factor (código 2FA). 
El servidor no verifica que el paso del 2FA fue completado antes de permitir 
el acceso a páginas protegidas como `/my-account`.

## Proceso de explotación

1. Inicié sesión con mis propias credenciales (`wiener:peter`) para entender 
el flujo normal: después del login, el servidor envía un código 2FA al email 
y redirige a la pantalla de verificación. Anoté la URL de mi página de cuenta 
(`/my-account`).
2. Cerré sesión e inicié sesión con las credenciales de la víctima 
(`carlos:montoya`). El servidor solicitó el código 2FA.
3. En lugar de ingresar el código, modifiqué manualmente la URL en el navegador 
cambiando `/login2` por `/my-account`.
4. El servidor me dio acceso directo a la cuenta de Carlos sin exigir el código 
2FA, resolviendo el laboratorio.

## Conclusión

Este lab demuestra que implementar 2FA no es suficiente si el flujo de 
autenticación no valida correctamente que todos los pasos fueron completados. 
Al asignar una sesión parcialmente autenticada después del primer factor, el 
servidor permite que un atacante con credenciales válidas saltee el segundo 
factor simplemente navegando directamente a una URL protegida. El servidor 
debe verificar el estado completo de la autenticación antes de conceder acceso 
a cualquier recurso protegido.
