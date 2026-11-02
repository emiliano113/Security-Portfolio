# Username Enumeration via Subtly Different Responses

**Categoría:** Authentication
**Nivel:** Practitioner
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Aplicación de login sutilmente vulnerable a enumeración de usuarios y fuerza bruta 
de contraseñas, a diferencia del lab anterior donde la diferencia entre respuestas 
era obvia. El objetivo es identificar un usuario válido, descubrir su contraseña 
y acceder a su cuenta.

## Vulnerabilidad

Todas las respuestas del login muestran el mismo mensaje genérico 
("Invalid username or password"), pero existe una diferencia mínima y no visible 
a simple vista: en el caso de un usuario válido, el mensaje de error tiene un 
**espacio extra al final** en lugar de un punto. Esta diferencia sutil en el texto 
permite distinguir usuarios válidos de inválidos usando la función de extracción 
de datos de Burp Intruder.

## Proceso de explotación

1. Con Burp corriendo, envié un login con usuario y contraseña inválidos, capturé 
la request `POST /login` y la mandé a **Burp Intruder**, marcando el parámetro 
`username` como posición de payload con la wordlist de usuarios candidatos.
2. En la pestaña **Settings** del Intruder, usé la función **Grep - Extract** para 
que Burp capturara automáticamente el texto exacto del mensaje de error en cada 
respuesta, agregándolo como columna extra en los resultados.
3. Al finalizar el ataque, ordené los resultados por esa columna y detecté que uno 
de los mensajes era ligeramente distinto a los demás: tenía un espacio al final en 
lugar del punto final, revelando el usuario válido.
4. Repetí el proceso con ese usuario fijo, marcando esta vez el parámetro `password` 
como posición de payload y cargando la wordlist de contraseñas.
5. Al finalizar, identifiqué la respuesta con código **302** (login exitoso) en 
lugar de 200.
6. Inicié sesión con el usuario y contraseña encontrados, resolviendo el laboratorio.

## Conclusión

Este lab muestra que no alcanza con unificar el mensaje de error visible para 
prevenir la enumeración de usuarios: cualquier diferencia mínima en la respuesta 
(un espacio, un carácter, el tiempo de respuesta) puede ser detectada y explotada 
con herramientas como Grep-Extract de Burp. La única forma robusta de evitar este 
tipo de fuga es asegurar que la respuesta sea **byte por byte idéntica** para 
usuarios válidos e inválidos, sin ninguna variación de texto, formato o timing.
