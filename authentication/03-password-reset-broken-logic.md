# Password Reset Broken Logic

**Categoría:** Authentication
**Nivel:** Apprentice
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Aplicación con una funcionalidad de restablecimiento de contraseña vulnerable. 
El objetivo es resetear la contraseña de `carlos` y acceder a su cuenta.

## Vulnerabilidad

El servidor no valida el token de restablecimiento (`temp-forgot-password-token`) 
al momento de procesar la nueva contraseña. Esto significa que el token puede ser 
eliminado o dejado vacío y el servidor igual procesa el cambio, confiando únicamente 
en el parámetro `username` para determinar qué cuenta modificar.

## Proceso de explotación

1. Hice clic en **"Forgot your password?"** con mi propio usuario (`wiener`) para 
iniciar el flujo de reset. Accedí al cliente de correo del lab, abrí el email de 
restablecimiento y completé el cambio de contraseña.
2. Con Burp corriendo, analicé el HTTP History y encontré la request
3. `POST /forgot-password?temp-forgot-password-token` que se envía al completar 
el formulario. La envié a **Burp Repeater**.
4. En Repeater, eliminé el valor del parámetro `temp-forgot-password-token`
5. tanto en la URL como en el cuerpo de la request, dejándolo vacío. Envié la request y 
el servidor la procesó correctamente, confirmando que el token no es validado.
6. Cambié el parámetro `username` de `wiener` a `carlos`, establecí una nueva
7. contraseña de mi elección y envié la request. El servidor aceptó el cambio.
8. Inicié sesión con `carlos` y la nueva contraseña, accediendo a su cuenta y 
resolviendo el laboratorio.

## Conclusión

Este lab demuestra una falla crítica en la lógica del restablecimiento de contraseña: el token de reset existe para verificar que quien solicita el cambio 
es el dueño legítimo de la cuenta, pero si el servidor no lo valida al momento de aplicar el cambio, el token pierde todo su propósito. El parámetro `username` 
en el cuerpo de la request nunca debería ser controlable por el cliente — el servidor debe derivar la cuenta a modificar exclusivamente del token validado, 
no de un campo que el atacante puede manipular libremente.
