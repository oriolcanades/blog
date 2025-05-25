---
layout: post
title: "Cómo garantizar la emisión diferida de credenciales usando Refresh Tokens"
date: 2025-04-27
categories: [Identidad Digital]
author: Oriol Canadés
version: 1.0
lang: es
permalink: /es/identidad-digital/2025/04/27/credencial-diferida-usando-refresh-token/
translated_url: /en/digital-identity/2025/04/27/deferred-credential-using-refresh-token/
description: "Descubre cómo reforzar la resiliencia en OID4VCI usando Refresh Tokens en flujos Pre-Authorized Code para gestionar emisiones diferidas."
---

A lo largo de mi trabajo en proyectos de identidad digital, me he encontrado con un reto recurrente: ¿cómo garantizar la emisión de credenciales cuando el proceso no puede completarse de inmediato?

Trabajando con flujos de emisión basados en la especificación [OpenID for Verifiable Credential Issuance (OID4VCI)](https://openid.github.io/OpenID4VCI/openid-4-verifiable-credential-issuance-wg-draft.html), el flujo Pre-Authorized Code facilita escenarios donde el Wallet no necesita una autorización interactiva tradicional. Sin embargo, en situaciones de emisión diferida, donde intervienen validaciones manuales o procesos asincrónicos, surge una pregunta crítica: **¿Qué ocurre si el Access Token caduca antes de que la credencial esté lista para ser entregada?**

En esta publicación quiero compartir cómo abordamos este desafío, qué soluciones exploramos y por qué el uso de Refresh Tokens en flujos Pre-Authorized Code se convierte en una pieza clave para garantizar la resiliencia en la emisión de credenciales verificables.

## Un poco de contexto

Cuando un Credential Issuer no puede emitir la credencial inmediatamente, responde al Wallet con un transaction_id, permitiendo que el Wallet recupere la credencial más adelante a través del Deferred Credential Endpoint. Sin embargo, la especificación establece que:

- El Access Token es obligatorio para acceder al Deferred Credential Endpoint.
- Hasta donde he podido analizar, no existe un mecanismo estandarizado para renovar el Access Token en el flujo Pre-Authorized Code.

Aunque opcional, la documentación del OID4VCI permite emitir un Refresh Token en el Token Endpoint. Implementarlo de forma adecuada resulta crucial en escenarios de emisión diferida.

Vamos a ver en qué casos de uso sería interesante poder usar un Refresh Token en el flujo Pre-Authorized Code.

### 1. Firma manual por parte de un responsable (Onboarding de empleados)

En procesos de incorporación, un responsable de RRHH puede validar y firmar manualmente la credencial de un nuevo empleado. Este proceso puede tardar horas. Si el Access Token caduca y no existe un Refresh Token, el empleado debería reiniciar el proceso de emisión completo, lo que podría ser frustrante y poco eficiente.

### 2. Procesos asincrónicos en universidades o administraciones

La emisión de diplomas o certificados académicos suele requerir validaciones múltiples, firmas institucionales y procesos que pueden durar días. Sin un Refresh Token, los estudiantes podrían perder la posibilidad de recibir su credencial de forma transparente.

### 3. Problemas de infraestructura o errores operativos

Por diversos motivos (problemas de infraestructura, errores en la firma, caídas de red), el proceso de emisión puede no completarse de inmediato. Si el Access Token expira durante este período, sin un Refresh Token, el Wallet perdería la posibilidad de recuperar la credencial.

## ¿Cómo usamos el Refresh Token en Pre-Authorized Code?

Aunque el flujo Pre-Authorized Code no incluye un Authorization Request, OAuth 2.0 ([RFC6749](https://datatracker.ietf.org/doc/html/rfc6749)) permite emitir un Refresh Token. Para implementarlo correctamente:

1. Emitir un Access Token de vida corta (5–10 minutos).
2. Emitir un Refresh Token válido por 7, 30 o 90 días.
3. Asociar el Refresh Token al transaction_id.
4. Permitir la renovación de Access Token sólo si:
    - El transaction_id sigue vigente.
    - La credencial no ha sido emitida.

Veamos como sería el flujo de trabajo:

![Deferred Credential Using Refresh Token](/assets/img/posts/deferred-credential-using-refresh-token.png)

Tras recibir un Credential Response con `transaction_id`:

1. El Wallet almacena:
    - `access_token`, `refresh_token`, `expires_in`, `transaction_id`, timestamps.

2. Intenta inmediatamente recuperar la credencial:
    - Si recibe `issuance_pending`, espera según el `interval` indicado.

3. Antes de cada intento:
    - Verifica si el Access Token está vigente.
    - Si ha caducado y tiene un Refresh Token válido, solicita uno nuevo.

4. Si el Refresh Token caduca:
    - Limpia la sesión y el usuario deberá solicitar una nueva Credential Offer a la entidad emisora.

## Riesgos y consideraciones de seguridad

- **Almacenamiento seguro:** El Wallet debe cifrar los Refresh Tokens.
- **Vinculación segura:** Asociar explícitamente cada Refresh Token a un transaction_id único.
- **Reutilización controlada:** Evitar permitir múltiples usos indebidos del mismo Refresh Token.
- **Revocación:** El Issuer debe invalidar Refresh Tokens tras emitir la credencial.
- **Monitorización:** Se deben registrar usos anómalos de Refresh Token.
- **Limitación de intentos:** Prevenir abusos o denegaciones de servicio.

## Algunas recomendaciones si se quiere implementar esta solución

- Controlar la validez de transaction_id antes de permitir renovaciones.
- Diseñar Wallets para gestionar expiración de tokens de forma automática.
- Proporcionar logs claros y auditables de los reintentos.

## Notas finales

Implementar el Refresh Token en flujos Pre-Authorized Code es clave para garantizar la resiliencia y usabilidad de los sistemas de emisión de credenciales verificables. No se trata solo de cumplir con la especificación, sino de construir confianza en los procesos de identidad digital.

Los sistemas empresariales modernos necesitan soportar flujos asincrónicos, interrupciones y tiempos de espera variables sin impactar la experiencia del usuario. La emisión diferida robusta es una pieza fundamental para lograrlo.

**¿Te has enfrentado a retos de emisión diferida en tus proyectos de identidad digital? ¡Me encantaría conocer tu experiencia!**
