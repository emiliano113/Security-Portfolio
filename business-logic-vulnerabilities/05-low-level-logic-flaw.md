# Low-Level Logic Flaw

**Categoría:** Business Logic Vulnerabilities
**Nivel:** Practitioner
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Tienda online donde el objetivo es comprar una chaqueta de cuero ($1337.00) cuyo precio supera ampliamente el saldo disponible en la cuenta, explotando un fallo de bajo nivel en el manejo numérico del carrito.

## Vulnerabilidad

El campo `quantity` del carrito se procesa en el servidor como un entero de 32 bits con signo, sin validar un límite superior razonable. El precio se almacena internamente en centavos (la chaqueta equivale a `133700`). Al acumular una cantidad suficiente de unidades, el total supera el valor máximo representable por ese tipo de dato (2.147.483.647, es decir 2³¹-1) y se produce un **integer overflow**: el valor se desborda y pasa abruptamente al mínimo posible (-2.147.483.648), volviéndose negativo.

## Proceso de explotación

1. Inicié sesión e intenté comprar la chaqueta de cuero; el pedido fue rechazado por crédito insuficiente. Analizando el historial de Proxy en Burp, identifiqué la solicitud `POST /cart` y la envié a Burp Repeater.
2. Dado que el campo `quantity` solo admite hasta 2 dígitos por solicitud, envié la petición a Burp Intruder y configuré el parámetro `quantity` en `99` (el máximo posible por request).
3. En la pestaña **Payloads**, usé el tipo **Null payloads** configurado para generar solicitudes de forma continua (**"Continue indefinitely"**), e inicié el ataque para sumar unidades de a 99 repetidamente.
4. Mientras el ataque corría, fui monitoreando el total del carrito. En un momento el precio saltó abruptamente a un número negativo grande y comenzó a acercarse a cero — señal de que se había superado el límite del entero de 32 bits.
5. Vacié el carrito y repetí un ataque de Intruder similar, pero esta vez configurando el payload para generar exactamente **323 payloads**, y agregué la solicitud a un **Resource Pool** con el número máximo de solicitudes simultáneas limitado a **1** (para evitar condiciones de carrera y asegurar que el conteo final fuera predecible).
6. Al finalizar el ataque, envié una única solicitud de **47 chaquetas** vía Repeater. El total del carrito quedó en **-$1221.96**.
7. Usando Repeater, agregué la cantidad necesaria de otro artículo para que el total final del carrito quedara entre $0 y $100.
8. Confirmé el pedido (**"Place order"**), resolviendo el laboratorio.

## Conclusión

Este lab expone un problema clásico de **integer overflow**: cuando una aplicación no limita la cantidad máxima permitida en un campo numérico, un atacante puede forzar ese valor más allá del límite que el tipo de dato puede representar (en este caso, un entero de 32 bits con signo), provocando que se desborde y "dé la vuelta" hacia el extremo opuesto del rango. La lección de bajo nivel es clave: la validación de rangos no solo debe evitar valores negativos en el ingreso del usuario, sino también prevenir que valores positivos acumulados excedan los límites del tipo de dato usado para representarlos en el servidor. Además, el uso de un Resource Pool con concurrencia limitada fue importante para que el ataque diera un resultado numérico predecible y no afectado por race conditions.
