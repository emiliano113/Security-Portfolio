# Flawed Enforcement of Business Rules

**Categoría:** Business Logic Vulnerabilities
**Nivel:** Apprentice
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Tienda online que ofrece cupones de descuento, con la regla de negocio implícita de que cada cupón debería poder aplicarse una única vez. El objetivo es conseguir un descuento mayor al previsto para poder comprar un artículo fuera de presupuesto.

## Vulnerabilidad

La aplicación no lleva un control adecuado de qué cupones ya fueron aplicados al carrito durante la sesión. Esto permite reutilizar los mismos códigos de forma repetida, acumulando descuentos por encima del límite que la lógica de negocio pretendía permitir.

## Proceso de explotación

1. Apliqué el primer cupón de descuento disponible al carrito.
2. Apliqué el segundo cupón disponible a continuación.
3. Volví a aplicar el primer cupón nuevamente, y la aplicación lo aceptó sin restricción.
4. Repetí el proceso intercalando ambos cupones (cupón 1 → cupón 2 → cupón 1 → cupón 2...) de forma sucesiva.
5. Cada repetición sumó un nuevo descuento al total, hasta alcanzar un saldo suficiente para cubrir el artículo objetivo.
6. Completé la compra, resolviendo el laboratorio.

## Conclusión

Este lab muestra que no alcanza con que un cupón sea válido en términos de formato o existencia: la aplicación también debe llevar un registro de **cuántas veces** y **en qué contexto** ya fue utilizado dentro de una misma sesión o cuenta. La falta de control de estado sobre acciones que deberían ser de uso único es una falla común de lógica de negocio, y suele explotarse simplemente repitiendo la acción tantas veces como el sistema lo permita.
