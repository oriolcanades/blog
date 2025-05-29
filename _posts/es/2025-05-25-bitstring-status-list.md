---
layout: post
title: "Entendiendo Bitstring Status List en el Ecosistema de Verifiable Credentials (W3C v2.0)"
date: 2025-05-25
categories: [Identidad Digital]
author: Oriol Canadés
version: 1.0
lang: es
permalink: /es/identidad-digital/2025/05/25/bitstring-status-list/
translated_url: /en/digital-identity/2025/05/25/bitstring-status-list/
description: "Explora el concepto de Bitstring Status List en Verifiable Credentials y su implementación en la especificación W3C v2.0, facilitando la verificación eficiente del estado de las credenciales."
---

Las *Verifiable Credentials* (VC) han emergido como uno de los pilares fundamentales del ecosistema de identidad digital descentralizada. Como parte del estándar definido por el W3C, permiten representar información verificable de forma segura, privada, interoperable y controlada. Sin embargo, su crecimiento en escenarios reales ha puesto sobre la mesa nuevas necesidades: ¿cómo revocar una credencial sin romper su privacidad?, ¿cómo garantizar que una credencial sigue siendo válida a lo largo del tiempo sin exponer innecesariamente a su portador?

Aquí es donde entra en juego el concepto de **Status List** —en concreto, el uso de una *BitstringStatusListCredential* como mecanismo de revocación o suspensión. En este artículo exploraremos su papel en ecosistemas como eIDAS2 o SSI empresarial, su valor arquitectónico, cómo se consulta y se representa, y por qué está ganando relevancia como método interoperable.

## ¿Por qué necesitamos un mecanismo de estado en Verifiable Credentials?

A diferencia de los certificados tradicionales, las *Verifiable Credentials* descentralizadas no dependen de una CA (Autoridad de Certificación) ni de una PKI tradicional para su validación. Esto permite privacidad, pero plantea un reto: ¿cómo saber si una credencial es válida en el momento de su uso?

Algunos casos típicos que requieren mecanismos de estado son:
- Revocación anticipada (ej. se detecta fraude o finaliza un contrato).
- Suspensión temporal (ej. acceso bloqueado por mantenimiento).
- Verificación pasiva por parte de terceros (sin contactar al emisor).

## ¿Qué es una Bitstring Status List Credential (W3C)?

Una *BitstringStatusListCredential* es una credencial firmada cuyo propósito no es identificar a un sujeto, sino listar el estado de hasta 131.072 otras credenciales.

Cada credencial que desee ser revocable, incluirá en su campo `credentialStatus` una referencia a:
- `statusListCredential`: la URL de la lista.
- `statusListIndex`: el índice que ocupa en el bitstring.

El campo `encodedList` dentro del `credentialSubject` contiene una cadena binaria de 16KB comprimida con GZIP y codificada en Base64URL. Cada bit representa una credencial:
- `0` → válida (o no asignada aún).
- `1` → revocada o suspendida.

La lista es pública, pero anónima: nadie puede saber a quién corresponde un índice si no conoce la credencial original.

## Estructura de una BitstringStatusListCredential

```json
{
  "@context": ["https://www.w3.org/ns/credentials/v2"],
  "type": ["VerifiableCredential", "BitstringStatusListCredential"],
  "id": "http://issuer.127.0.0.1.nip.io/credentials/status/1",
  "issuer": "did:elsi:VATES-B12345678",
  "validFrom": "2025-05-25T00:00:00Z",
  "validUntil": "2025-08-25T00:00:00Z",
  "credentialSubject": {
    "type"         : "BitstringStatusList",
    "statusPurpose": "revocation",
    "encodedList"  : "uH4sIAAAAAAAA_-3OMQEAAAgDoEWzf...AAqAU8U2NYAEAAAA"
  }
}
```
📘 [Referencia oficial al draft del W3C](https://www.w3.org/TR/2025/REC-vc-bitstring-status-list-20250515/)

## Cómo verificar el estado de una Verifiable Credential

El verifier debe:
1. Obtener el valor `statusListCredential` y `statusListIndex` de la credencial verificable.
2. Descargar el recurso `statusListCredential` (por ejemplo: `/credentials/status/1).
3. Decodificar el campo encodedList (base64 + gzip).
4. Leer el bit en la posición statusListIndex.

Ejemplo conceptual:
```java
byte[] bitstring = decodeBase64Gzip(encodedList);
boolean isRevoked = checkBit(bitstring, statusListIndex) == 1;
```

## Ejemplo: LEARCredentialMachine

Una credencial de tipo LEARCredentialMachine (para agentes software) puede incluir esta estructura de estado:

```json
{
    "@context": ["https://www.w3.org/ns/credentials/v2"],
    "type": ["VerifiableCredential", "LEARCredentialMachine"],
    "id": "urn:uuid:68422e47-5d69-4e0b-8a49-34990f2f76a2",
    "issuer": "did:elsi:VATES-B12345678",
    "validFrom": "2025-05-25T00:00:00Z",
    "validUntil": "2025-08-25T00:00:00Z",
    "credentialSubject": {
      "mandate": {
        "id": "urn:uuid:a2ece539-e199-4be9-a781-3a11f4a25ad9",
        "mandator": {
          "id": "did:elsi:VATFR-A12345678",
          "organization": "ACME, S.L.",
          "country": "FR",
          "commonName": "JEAN MARTIN - CNI 880692310285",
          "serialNumber": "880692310285"
        },
        "mandatee": {
          "id": "did:key:zDnaexS1hEocz1R51ZXakcUPXWZSzkVEBJAEz9fHtxjfqZRhN",
          "domain": "dpas.acme.io",
          "ipAddress": "195.70.63.244"
        }
      }
    },
    "credentialStatus": {
      "id": "http://issuer.trustservice.com/credentials/status/1#94567",
      "type": "BitstringStatusListEntry",
      "statusPurpose": "revocation",
      "statusListIndex": "94567",
      "statusListCredential": "http://issuer.trustservice.com/credentials/status/1"
    }
}
```

Esto permite que cualquier verificador, sin contactar al emisor, consulte si ese índice fue marcado como revocado.

## Ventajas arquitectónicas

- **Desacoplamiento**: la verificación del estado no depende del backend emisor.
- **Escalabilidad**: con 1 credencial firmada se controlan 130k credenciales.
- **Privacidad preservada**: nadie sabe qué índice corresponde a qué usuario.
- **Velocidad de consulta**: basta un GET HTTP y unas operaciones bit a bit.

## Casos de uso

#### 1. Revocación M2M
Agentes automáticos o máquinas con credenciales delegadas pueden ser revocados fácilmente cuando finaliza un mandato.

#### 2. Accesos temporales
Útil en plataformas con pases temporales o credenciales ligadas a eventos.

#### 3. Cumplimiento legal
Permite trazabilidad conforme a marcos como eIDAS 2.0 o AML sin comprometer privacidad.

## Buenas prácticas de implementación

- Firmar la lista como JWT.
- Utilizar GZIP + Base64URL para encodedList.
- Centralizar la gestión del índice.
- No compartir listas completas en frontend.
- Validar que los bits estén en rangos válidos.

## Comparativa de métodos de revocación

| Método              | Escalabilidad | Privacidad | Consulta eficiente |
|---------------------|---------------|------------|--------------------|
| CRL tradicional     | Baja          | No         | No                 |
| OCSP                | Media         | Parcial    | Media              |
| BitstringStatusList | Alta          | Sí         | Sí                 |

## Lecciones aprendidas

1. Una status list debe considerarse un componente core en el emisor. 
2. A veces, varios ficheros son necesarios (status/1, status/2, ...). 
3. Las listas también son credenciales: deben firmarse, tener fechas, y ser válidas. 
4. Evita exponer los índices usados públicamente (riesgo de correlación).

## Más allá del código: interoperabilidad y confianza

Este modelo ha sido recomendado por múltiples iniciativas europeas como EBSI, ESSIF o Gaia-X. Permite:
- Interoperar sin redefinir semánticas.
- Integrar verificadores livianos (por ejemplo, wallets móviles).
- Federar la validación del estado sin servidores de confianza.

## Conclusión

El modelo *Bitstring Status List* no solo resuelve la revocación de forma eficiente y privada, sino que permite una separación clara entre:

- Emisión de credenciales
- Validación de estado

Al entender esta arquitectura, podemos diseñar soluciones SSI más sólidas, alineadas con los principios de privacidad por diseño, interoperabilidad y escalabilidad. Y lo más importante: listas para los marcos normativos que están por llegar.

¿Te ha parecido útil? Sígueme en [LinkedIn](https://www.linkedin.com/in/oriolcanades/) o suscríbete a mi newsletter para más análisis sobre identidad digital, SSI y estándares como OpenID4VC o eIDAS2.
