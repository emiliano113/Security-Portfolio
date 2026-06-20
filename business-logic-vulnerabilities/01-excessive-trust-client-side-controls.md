# Excessive Trust in Client-Side Controls

**Categoría:** Business Logic Vulnerabilities
**Nivel:** Apprentice
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Tienda online donde se intenta comprar un artículo (chaqueta de cuero) que supera el crédito disponible en la cuenta. El objetivo es completar la compra sin tener fondos suficientes.

## Vulnerabilidad

La aplicación confía en el cliente para informar el precio del producto, en lugar de calcularlo del lado del servidor en base al ID del artículo. Esto permite manipular el valor antes de que llegue al backend.

## Proceso de explotación

1. Con Burp Suite interceptando el tráfico, intenté agregar el artículo al carrito y analicé la solicitud `POST /cart` en el historial de Proxy.
2. Detecté que la petición incluía un parámetro `price` enviado directamente desde el cliente.
3. Envié la solicitud a Burp Repeater y modifiqué el valor de `price` por un monto dentro de mi crédito disponible.
4. Reenvié la solicitud modificada, actualicé el carrito y confirmé que el precio había cambiado según lo que yo había definido.
5. Completé la compra con el precio alterado, resolviendo el laboratorio.

## Conclusión

El lab muestra un caso clásico de **confianza excesiva en datos del cliente**: cualquier valor que viaje en una request HTTP (precios, cantidades, roles, permisos) puede ser modificado por el usuario. La validación de datos sensibles a la lógica de negocio siempre debe hacerse en el servidor, nunca asumir que lo que llega del cliente es confiable.
