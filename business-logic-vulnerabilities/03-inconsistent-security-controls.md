# Inconsistent Security Controls

**Categoría:** Business Logic Vulnerabilities
**Nivel:** Apprentice
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Aplicación que otorga acceso administrativo a usuarios registrados con un correo de dominio corporativo. El objetivo es acceder al panel `/admin` y eliminar al usuario `carlos` sin tener originalmente un correo válido de ese dominio.

## Vulnerabilidad

La aplicación valida el dominio del correo corporativo únicamente durante el registro inicial. Al modificar el email desde la sección "Mi cuenta", el sistema no vuelve a exigir esa verificación, y el cambio de dominio alcanza para elevar privilegios de forma automática. Es un control de acceso defectuoso por **inconsistencia**: la misma validación se aplica en un punto de entrada pero no en otro.

## Proceso de explotación

1. **Descubrimiento de la ruta oculta:** usando la herramienta de Content Discovery de Burp, mapeé la aplicación y encontré la ruta `/admin`, no enlazada en la navegación normal. Al intentar acceder, el sistema rechazó el ingreso pero reveló una pista: esa zona es exclusiva para la organización `DontWannaCry`.
2. **Registro legítimo:** me registré con el dominio temporal provisto por PortSwigger (`@your-email-id.web-security-academy.net`), accedí al cliente de correo del lab y verifiqué la cuenta haciendo clic en el link de confirmación.
3. **Explotación del flujo de cambio de email:** una vez con la cuenta básica activa, fui a la sección de cambio de correo electrónico y lo modifiqué por uno con el dominio corporativo (`loquesea@dontwannacry.com`). Como el sistema no exige verificar esta nueva dirección, el cambio se aplicó de inmediato y actualizó mi rol en la sesión activa.
4. **Acceso y ejecución:** con la sesión ya reconocida como perteneciente al dominio autorizado, volví a `/admin`, obtuve acceso completo al panel y eliminé al usuario `carlos`, resolviendo el laboratorio.

## Conclusión

Este lab demuestra que un control de seguridad aplicado en un solo punto de entrada (el registro) no sirve de nada si existen otros caminos —como editar el perfil— que llevan al mismo resultado sin pasar por esa validación. Cualquier modificación de datos críticos durante una sesión activa debe pasar por el mismo nivel de verificación que el proceso original, incluyendo re-autenticación o validación explícita del nuevo valor. Además, quedó claro el valor de mapear rutas no enlazadas con herramientas de Content Discovery, ya que muchas veces revelan funcionalidad sensible que no aparece en la navegación estándar.
