# Inconsistent Handling of Exceptional Input

**Categoría:** Business Logic Vulnerabilities
**Nivel:** Practitioner
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Aplicación con registro de usuarios que otorga acceso administrativo a empleados del dominio `dontwannacry.com`. El objetivo es acceder al panel de administración y eliminar al usuario `carlos`.

## Vulnerabilidad

La aplicación trunca los emails almacenados a 255 caracteres sin considerar el impacto sobre la validación de dominio. Esto permite diseñar un email que, tras el truncado, quede con `@dontwannacry.com` como dominio efectivo, obteniendo privilegios administrativos sin pertenecer realmente a esa organización.

## Proceso de explotación

1. Usé **Content Discovery** de Burp para encontrar la ruta `/admin`, que confirmó ser exclusiva para el dominio `DontWannaCry`.
2. Me registré con un email largo para verificar que el servidor truncaba a 255 caracteres.
3. Calculé la longitud exacta para que al truncarse, el dominio resultante fuera 
`@dontwannacry.com`, usando el formato:very-long-string@dontwannacry.com.YOUR-EMAIL-ID.web-security-academy.net
4. Verifiqué la cuenta desde el cliente de correo del lab, inicié sesión y obtuve acceso al panel de administración.
5. Eliminé al usuario `carlos`, resolviendo el laboratorio.

## Conclusión

El truncado de strings aplicado después de la validación puede romper la lógica de control de acceso. La aplicación valida el email completo al registrarse, pero almacena una versión truncada que ya no refleja el dominio real. Los datos siempre deben validarse sobre su forma final, tal como serán almacenados y usados por el sistema.
