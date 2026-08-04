# Authentication Bypass via Encryption Oracle

**Categoría:** Business Logic Vulnerabilities
**Nivel:** Practitioner
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Aplicación con una cookie cifrada `stay-logged-in` que gestiona la sesión persistente del usuario. El objetivo es forjar una cookie válida para la cuenta `administrator` sin conocer la clave de cifrado, usando la propia aplicación como oráculo.

## Vulnerabilidad

La aplicación expone involuntariamente dos operaciones criptográficas accesibles al 
usuario:
- El parámetro `email` del formulario de comentarios **cifra** cualquier dato arbitrario y devuelve el resultado en la cookie `notification`.
- La cookie `notification` en las requests GET **descifra** su contenido y lo refleja en el mensaje de error de la página.

Esto convierte a la aplicación en un **oráculo de cifrado**: permite cifrar y descifrar datos controlados por el atacante, usando las mismas claves que protegen la cookie `stay-logged-in`.

## Proceso de explotación

1. **Identificación del oráculo:** inicié sesión con `wiener:peter` activando "Stay logged in" y publiqué un comentario con un email inválido. Analizando el HTTP History de Burp, identifiqué que la cookie `stay-logged-in` está cifrada y que el mensaje de error refleja el contenido descifrado de la cookie `notification`. Envié el `POST /post/comment` y el `GET /post?postId=x` a Burp Repeater, renombrando las pestañas **"encrypt"** y **"decrypt"** respectivamente.

2. **Descifrado de la cookie propia:** en la pestaña "decrypt", reemplacé el valor de la cookie `notification` por el valor de mi cookie `stay-logged-in` y envié la request. El servidor devolvió el contenido descifrado en texto plano, revelando el formato: `wiener:timestamp`. Copié el timestamp.

3. **Cifrado del valor objetivo:** en la pestaña "encrypt", cambié el parámetro `email` a `administrator:timestamp` (usando el timestamp obtenido) y envié la request. Copié la nueva cookie `notification` de la respuesta y la envié a la pestaña "decrypt" para verificar su contenido. Observé que el servidor agrega automáticamente el prefijo de 23 caracteres `"Invalid email address: "` a cualquier valor cifrado por esa vía.

4. **Eliminación del prefijo (padding):** para remover esos 23 caracteres del texto cifrado, necesitaba que el total de bytes a eliminar fuera múltiplo de 16 (requerimiento del algoritmo de cifrado en bloques). Agregué 9 caracteres de relleno (`xxxxxxxxx`) al inicio del valor en la pestaña "encrypt", quedando: xxxxxxxxxadministrator:timestamp esto hizo que el prefijo basura ocupara exactamente 32 bytes (2 bloques de 16), 
facilitando su eliminación limpia.

5. **Manipulación en Burp Decoder:** envié el nuevo ciphertext a **Burp Decoder**, lo decodifiqué con URL-decode y luego Base64-decode. En la pestaña **Hex**, eliminé los primeros 32 bytes correspondientes al prefijo basura. Re-encodeé el resultado (Base64 → URL-encode) y lo pegué como valor de la cookie `notification` en la pestaña "decrypt". La respuesta confirmó que el contenido descifrado era únicamente `administrator:timestamp`, sin ningún prefijo.

6. **Forja de sesión:** desde el HTTP History, envié la request `GET /` a Repeater. Eliminé la cookie `session` por completo y reemplacé el valor de `stay-logged-in` por el ciphertext forjado. Al enviar la request, el servidor me autenticó como `administrator`.

7. **Ejecución:** accedí a `/admin` y navegué a `/admin/delete?username=carlos`, resolviendo el laboratorio.

## Conclusión

Este lab demuestra el riesgo de exponer operaciones criptográficas al usuario sin restricciones. Cuando una aplicación permite cifrar y descifrar datos arbitrarios usando sus propias claves internas, cualquier mecanismo de seguridad basado en esas claves queda comprometido. La cookie `stay-logged-in` era técnicamente segura en términos de algoritmo, pero la existencia del oráculo hizo irrelevante esa seguridad. Los endpoints que manejan criptografía deben validar estrictamente qué datos pueden ser procesados y por quién.
