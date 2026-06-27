# Infinite Money Logic Flaw

**Categoría:** Business Logic Vulnerabilities
**Nivel:** Practitioner
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Tienda online con un sistema de gift cards y cupones de descuento. El objetivo es explotar 
una falla en el flujo de compra para acumular crédito indefinidamente y poder comprar 
la chaqueta de cuero.

## Vulnerabilidad

La aplicación permite comprar gift cards de $10 con un cupón de descuento del 30% 
(`SIGNUP30`), obteniendo cada una por $7. Al canjear esa gift card se recuperan $10 
completos, generando una ganancia neta de $3 por ciclo. La falla está en que no existe 
ningún control que limite la cantidad de veces que este ciclo puede repetirse: sin rate 
limiting, sin captcha, y sin restricción sobre la reutilización del cupón en combinación 
con el canje de gift cards.

## Proceso de explotación

### 1. Reconocimiento manual (HTTP History)
Usando el historial del Proxy de Burp, analicé el flujo completo de compra manual: 
agregar gift card al carrito, aplicar el cupón `SIGNUP30`, hacer checkout, obtener el 
código de la gift card en la confirmación de orden, y canjearlo en `POST /gift-card`. 
Identifiqué que el ciclo completo generaba $3 de ganancia neta por iteración.

### 2. Grabación de la macro (Burp Macros)
En **Settings → Sessions → Session handling rules**, grabé una macro con la siguiente 
secuencia de 5 peticiones consecutivas:
- `POST /cart` — agregar gift card al carrito
- `POST /cart/coupon` — aplicar el cupón de descuento
- `POST /cart/checkout` — completar la compra
- `GET /cart/order-confirmation?order-confirmed=true` — obtener confirmación con el 
código de la gift card
- `POST /gift-card` — canjear la gift card

### 3. Extracción dinámica del código (Custom Parameter)
En el editor de la macro, configuré un parámetro personalizado llamado `gift-card` que 
extraía automáticamente el código dinámico de la gift card del HTML de respuesta del 
`GET /order-confirmation`, y lo inyectaba en la petición siguiente (`POST /gift-card`). 
Esto permitió que Burp manejara los códigos cambiantes en cada ciclo sin intervención manual.

### 4. Vinculación con Intruder (Session Handling Rule)
Creé una regla de sesión para que la macro se ejecutara en cada petición lanzada por 
Intruder, convirtiendo cada disparo del ataque en un ciclo completo de compra y canje.

### 5. Automatización (Burp Intruder + Resource Pool)
Envié la petición `GET /my-account` a Burp Intruder, configuré el tipo de payload como 
**Null payloads** con **412 ejecuciones**, y asigné el ataque a un Resource Pool con 
**máximo 1 request simultánea** para garantizar que el servidor procesara cada ciclo en 
orden cronológico sin condiciones de carrera. Al finalizar el ataque, el crédito acumulado 
fue suficiente para comprar la chaqueta, resolviendo el laboratorio.

## Conclusión

Este lab combina una falla de lógica de negocio con ausencia de controles de 
automatización. El ciclo comprar-canjear generaba ganancia neta por diseño, y sin rate 
limiting ni captcha, fue trivial automatizarlo con Burp Macros e Intruder. La lección es 
doble: primero, cualquier flujo donde se intercambian valores monetarios debe auditarse 
para detectar ciclos de ganancia neta; segundo, los endpoints críticos deben tener 
controles que detecten y limiten el abuso automatizado, independientemente de si la lógica 
individual de cada transacción parece válida.
