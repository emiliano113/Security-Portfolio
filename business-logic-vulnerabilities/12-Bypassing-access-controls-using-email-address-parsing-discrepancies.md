# Bypassing Access Controls Using Email Address Parsing Discrepancies

**Categoría:** Business Logic Vulnerabilities
**Nivel:** Expert
**Plataforma:** PortSwigger Web Security Academy

## Descripción del lab

Aplicación que restringe el registro únicamente a usuarios con correo del dominio `ginandjuice.shop`. El objetivo es bypassear esa validación para registrarse con un correo externo, obtener acceso administrativo y eliminar al usuario `carlos`.

## Vulnerabilidad

Existe una discrepancia entre cómo el servidor valida el email durante el registro y cómo el servidor de correo lo interpreta al enviar la confirmación. La validación del servidor detecta y bloquea encodings comunes (ISO-8859-1, UTF-8), pero no reconoce **UTF-7** como una amenaza. Esto permite codificar el símbolo `@` y el espacio en UTF-7, engañando al validador para que vea `@ginandjuice.shop` como dominio válido, mientras el servidor de correo interpreta la dirección real como 
`attacker@[YOUR-EXPLOIT-SERVER-ID]`.

## Proceso de explotación

1. **Identificación de la restricción:** intenté registrarme con un correo externo (`foo@exploit-server.net`) y la aplicación lo bloqueó, indicando que solo acepta el dominio `ginandjuice.shop`.

2. **Investigación de encodings:** probé registrarme con emails codificados en ISO-8859-1 y UTF-8 — ambos fueron bloqueados. Al intentar con **UTF-7**, el servidor no generó ningún error, revelando que no reconoce ese encoding como amenaza.

3. **Explotación:** me registré con el siguiente email codificado en UTF-7: =?utf-7?q?attacker&AEA-[YOUR-EXPLOIT-SERVER-ID]&ACA-?=@ginandjuice.shop
Esto representa `attacker@[YOUR-EXPLOIT-SERVER-ID] ?=@ginandjuice.shop`, con el `@` y el espacio codificados en UTF-7. El validador del servidor vio el dominio `@ginandjuice.shop` al final y lo aprobó, pero el servidor de correo decodificó el UTF-7 y envió el email de confirmación a `attacker@[YOUR-EXPLOIT-SERVER-ID]`.

4. **Activación:** accedí al cliente de correo del lab, recibí el email de confirmación y activé la cuenta.

5. **Acceso admin:** inicié sesión, accedí al panel de administración y eliminé al usuario `carlos`, resolviendo el laboratorio.

## Conclusión

Este lab demuestra una discrepancia de parseo entre dos componentes del sistema: el validador de emails y el servidor de correo. Cuando dos partes de una aplicación interpretan el mismo dato de forma distinta, se abre una brecha explotable. La lección clave es que la validación debe realizarse sobre el dato ya normalizado y decodificado en su forma final, cubriendo todos los encodings posibles, no solo los más comunes.
