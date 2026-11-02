# 2FA Broken Logic

**Categoría:** Authentication
**Nivel:** Practitioner
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Aplicación con autenticación de dos factores vulnerable debido a una falla en 
la lógica de verificación. El objetivo es acceder a la cuenta de `carlos` 
sin conocer su código 2FA real.

## Vulnerabilidad

El parámetro `verify` de la request `GET /login2` determina sobre qué cuenta 
se genera y valida el código 2FA, y este parámetro puede ser modificado 
libremente por el cliente. Esto permite generar un código 2FA para la cuenta 
de la víctima mientras se usa la sesión propia, y luego hacer fuerza bruta 
sobre ese código sin que la cuenta de la víctima se bloquee ni se alerte.

## Proceso de explotación

1. Inicié sesión con mis propias credenciales (`wiener:peter`) y analicé el 
flujo de verificación 2FA en Burp. Identifiqué que la request 
`GET /login2` usa el parámetro `verify` para determinar de qué usuario se 
trata.
2. Cerré sesión. Envié la request `GET /login2` a Burp Repeater y cambié el 
valor del parámetro `verify` a `carlos`, generando así un código 2FA 
temporal para su cuenta.
3. Fui a la página de login, ingresé usuario y contraseña, y en la pantalla 
de verificación 2FA envié un código inválido a propósito, para capturar la 
request `POST /login2`.
4. Envié esa request a **Burp Intruder**, configuré el parámetro `verify` 
en `carlos` de forma fija y marqué el parámetro `mfa-code` como posición de 
payload para hacer fuerza bruta.
5. Ejecuté el ataque y busqué la respuesta con código **302** (verificación 
exitosa), identificando así el código correcto.
6. Con el código encontrado, accedí a la cuenta de `carlos` desde el 
navegador y entré a "My account", resolviendo el laboratorio.

## Conclusión

Este lab muestra que un parámetro que determina de quién es la identidad en 
un proceso de verificación 2FA nunca debe ser modificable por el cliente. 
Al permitir cambiar el valor de `verify`, la aplicación permite generar y 
atacar por fuerza bruta el código de 2FA de cualquier usuario sin que su 
propia sesión se vea comprometida ni se dispare ningún control de seguridad 
en la cuenta real de la víctima. La identidad del usuario en un paso de 
verificación debe derivarse siempre de la sesión autenticada del servidor, 
nunca de un parámetro enviado por el cliente.
