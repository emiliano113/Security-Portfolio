# Username Enumeration via Different Responses

**Categoría:** Authentication
**Nivel:** Apprentice
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Aplicación de login vulnerable a enumeración de usuarios y fuerza bruta de 
contraseñas. El objetivo es identificar un usuario válido y su contraseña usando 
las wordlists provistas, luego acceder a su cuenta.

## Vulnerabilidad

La aplicación devuelve mensajes de error distintos según si el usuario existe o no:
- Usuario inválido → `Invalid username`
- Usuario válido pero contraseña incorrecta → `Incorrect password`

Esta diferencia en las respuestas permite enumerar usuarios válidos antes de 
intentar el brute force de contraseñas, reduciendo significativamente el espacio 
de ataque.

## Proceso de explotación

1. Con Burp corriendo, intenté un login con credenciales inválidas y capturé la 
request `POST /login` en el HTTP History. La envié a **Burp Intruder**.
2. Marqué el parámetro `username` como posición de payload (dejando la contraseña 
fija) y configuré el ataque tipo **Sniper** con la wordlist de usuarios candidatos.
3. Al finalizar el ataque, analicé la columna **Length** — una respuesta tenía una 
longitud diferente al resto, indicando un mensaje distinto. Confirmé que esa 
respuesta contenía `Incorrect password` en lugar de `Invalid username`, revelando 
un usuario válido.
4. Con el usuario identificado, repetí el proceso en Intruder pero esta vez marcando 
el parámetro `password` como posición de payload y cargué la wordlist de contraseñas.
5. Al finalizar, identifiqué la respuesta con código **302** (redirección exitosa) 
en lugar de 200, lo que indicaba un login exitoso.
6. Inicié sesión con las credenciales encontradas, resolviendo el laboratorio.

## Conclusión

Este lab demuestra cómo las diferencias en los mensajes de error de una aplicación 
pueden filtrar información sensible. Cuando el servidor responde distinto según si 
el usuario existe o no, un atacante puede enumerar usuarios válidos antes de atacar 
las contraseñas. La buena práctica es devolver siempre el mismo mensaje genérico 
independientemente de si el usuario existe o si la contraseña es incorrecta, 
eliminando así cualquier pista que facilite la enumeración.
