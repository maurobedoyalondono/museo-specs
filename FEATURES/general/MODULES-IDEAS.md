# MODULES IMPLEMENTATION.

## Flow
to be implemented ste by step as AUTH-PLAN.md

## Systems
Apply to /web, /api and /infrastructure/mirgations/db

## Core Principes
For CLAUDE.md in each system we have core principles need to study and follow

## Rules
The need to be self contained. They can not just be along the core components.
They might live in something like
- modules/

For instance, for payment, all payment modules must live inside:
- src/modules/payment/mercado-pago

## Configuration
Initially we can use .env to determine of a Module is active.

For Payment we may have several active modules, just one by default (to be displayed initially in checkckout page, if more active, then they can be also used by selecting them)

## logger
for /api need to use logger

### Payment

#### Mercado Pago by Mercado Libre

Initial module implementation will be MercadoPago by Mercado Libre using checkout api
https://www.mercadopago.com.co/developers/en/docs/checkout-api/overview

#### Reponse Codes

Para probar diferentes resultados de pago, completa el estado deseado en el nombre del titular de la tarjeta:

Estado de pago	Descripción	Documento de identidad
APRO
Pago aprobado
123456789
OTHE
Rechazado por error general
123456789
CONT
Pendiente de pago
-
CALL
Rechazado con validación para autorizar
-
FUND
Rechazado por importe insuficiente
-
SECU
Rechazado por código de seguridad inválido
-
EXPI
Rechazado debido a un problema de fecha de vencimiento
-
FORM
Rechazado debido a un error de formulario
-

#### Config
Public Key
Access Token

#### Usage
We just want to use Credit Cards, we dont want to store them


#### Additional resources

https://www.mercadopago.com.co/developers/en/reference/payments/_payments/post