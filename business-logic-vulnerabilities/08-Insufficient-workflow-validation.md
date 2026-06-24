# Insufficient Workflow Validation

**Categoría:** Business Logic Vulnerabilities
**Nivel:** Practitioner
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Tienda online con un flujo de compra que asume incorrectamente que los pasos del proceso se ejecutan siempre en el orden correcto. El objetivo es comprar una chaqueta de cuero sin tener crédito suficiente para hacerlo.

## Vulnerabilidad

La aplicación no valida que el flujo de compra se haya completado correctamente antes de confirmar un pedido. El endpoint de confirmación de orden (`GET /cart/order-confirmation?order-confirmation=true`) puede ser invocado directamente, sin pasar por el paso de pago, y el servidor lo procesa como una compra válida sin verificar si el cobro realmente se realizó.

## Proceso de explotación

1. Inicié sesión y compré un artículo que sí podía pagar con el crédito disponible, completando el flujo normal de compra.
2. Analizando el historial del Proxy en Burp, identifiqué que al confirmar una orden, el servidor redirige a `GET /cart/order-confirmation?order-confirmation=true`. Envié esa solicitud a Burp Repeater.
3. Agregué la chaqueta de cuero al carrito, artículo que superaba mi crédito disponible.
4. Sin pasar por el paso de pago, reenvié desde Repeater la solicitud de confirmación de orden que había guardado anteriormente.
5. El servidor procesó la confirmación como válida, completando la compra de la chaqueta sin descontar el costo del crédito, resolviendo el laboratorio.

## Conclusión

Este lab ilustra una falla en la **validación del flujo de trabajo**: la aplicación asume que si el usuario llega al endpoint de confirmación, el pago ya fue procesado correctamente. Al no verificar el estado real del pago antes de confirmar la orden, cualquier usuario puede saltear el paso de cobro invocando directamente el endpoint de confirmación. Los flujos críticos de negocio deben validar en el servidor que cada paso previo fue completado y autorizado, no asumir que el orden de las peticiones fue el esperado.
