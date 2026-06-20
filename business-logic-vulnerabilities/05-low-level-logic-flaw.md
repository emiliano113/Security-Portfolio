# Low-Level Logic Flaw

**Categoría:** Business Logic Vulnerabilities
**Nivel:** Practitioner
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Tienda online donde el objetivo es comprar una chaqueta de cuero cuyo precio supera ampliamente el saldo disponible en la cuenta, explotando un fallo de bajo nivel en el manejo numérico del carrito.

## Vulnerabilidad

El campo `quantity` del carrito no valida límites superiores razonables y se procesa como un entero de 32 bits sin protección frente a **integer overflow**. Al superar el valor máximo representable (2^32), el número se desborda y pasa a interpretarse como negativo, lo que rompe por completo la lógica de cálculo del total del carrito.

## Proceso de explotación

1. Usando Burp Intruder, automaticé el envío de peticiones para agregar chaquetas al carrito de a 99 unidades por petición (modificando el parámetro `quantity` a 99 en cada request).
2. Repetí el envío de forma sucesiva hasta que la cantidad acumulada superó el límite de 2^32, momento en el que el valor del total se desbordó y pasó a mostrarse como un número negativo.
3. Una vez en negativo, calculé cuántas peticiones adicionales de chaquetas eran necesarias para acercar ese saldo negativo lo más posible a cero, ajustando la cantidad enviada en cada petición.
4. Agregué un producto adicional al carrito para terminar de inclinar el balance hacia un valor positivo y favorable, suficiente para cubrir el costo de la chaqueta original.
5. Finalicé la compra con "Place order", resolviendo el laboratorio.

## Conclusión

Este lab expone un problema clásico de **integer overflow**: cuando una aplicación no limita la cantidad máxima permitida en un campo numérico, un atacante puede forzar ese valor más allá del límite que el tipo de dato puede representar, provocando que se desborde y tome un valor inesperado (en este caso, negativo). La lección de bajo nivel es clave: la validación de rangos no solo debe evitar valores negativos en el ingreso del usuario, sino también prevenir que valores positivos acumulados excedan los límites del tipo de dato usado para representarlos en el servidor.
