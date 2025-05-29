---
layout: post
title: "Understanding Bitstring Status List in the Verifiable Credentials Ecosystem (W3C v2.0)"
date: 2025-05-25
categories: [Digital Identity]
author: Oriol Canadés
version: 1.0
lang: en
permalink: /en/digital-identity/2025/05/25/bitstring-status-list/
translated_url: /es/identidad-digital/2025/05/25/bitstring-status-list/
description: "Explore the concept of Bitstring Status List in Verifiable Credentials and its implementation in the W3C v2.0 specification, enabling efficient verification of credential status."
---

*Verifiable Credentials* (VCs) have emerged as one of the core pillars of the decentralized digital identity ecosystem. As part of the W3C standard, they allow for the representation of verifiable information in a secure, private, and interoperable manner. However, as they expand into real-world scenarios, new questions arise: how can we revoke a credential without breaking privacy? How can we ensure a credential remains valid over time without unnecessarily exposing the holder?

This is where the concept of **Status List** comes into play—specifically, the use of a *BitstringStatusListCredential* as a mechanism for revocation or suspension. In this article, we explore its role in ecosystems like eIDAS2 and enterprise SSI, its architectural value, how it is queried and structured, and why it is becoming a key interoperable method.

## Why do we need a status mechanism in Verifiable Credentials?

Unlike traditional certificates, decentralized *Verifiable Credentials* do not rely on a Certification Authority (CA) or a conventional PKI for validation. This enables privacy but introduces a challenge: how can we verify whether a credential is still valid at the time of use?

Typical scenarios that require status mechanisms include:
- Early revocation (e.g., fraud detected or contract terminated).
- Temporary suspension (e.g., access blocked during maintenance).
- Passive verification by third parties (without contacting the issuer).

## What is a Bitstring Status List Credential (W3C)?

A *BitstringStatusListCredential* is a signed credential whose purpose is not to identify a subject but to list the status of up to 131,072 other credentials.

Any credential intended to be revocable includes in its `credentialStatus` field a reference to:
- `statusListCredential`: the URL of the status list.
- `statusListIndex`: the index it occupies in the bitstring.

The `encodedList` field inside the `credentialSubject` contains a 16KB binary string, compressed with GZIP and encoded in Base64URL. Each bit represents one credential:
- `0` → valid (or not yet assigned).
- `1` → revoked or suspended.

The list is public, but anonymous: no one can know which credential corresponds to which index without the original credential.

## Structure of a BitstringStatusListCredential

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
📘 [Official W3C Draft Reference](https://www.w3.org/TR/2025/REC-vc-bitstring-status-list-20250515/)

# How to verify the status of a Verifiable Credential

The verifier must:
1. Retrieve `statusListCredential` and `statusListIndex` from the Verifiable Credential.
2. Download the `statusListCredential` resource (e.g., `/credentials/status/1`).
3. Decode the `encodedList` field (base64 + gzip).
4. Read the bit at position `statusListIndex`.

Conceptual example:
```java
byte[] bitstring = decodeBase64Gzip(encodedList);
boolean isRevoked = checkBit(bitstring, statusListIndex) == 1;
```

## Example: LEARCredentialMachine

A credential of type LEARCredentialMachine (for software agents) may include this status structure:

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

This allows any verifier, without contacting the issuer, to check if that index has been marked as revoked.

## Architectural advantages

- **Decoupling**: status validation does not depend on the issuer's backend.
- **Scalability**: a single signed credential can represent the status of 130k credentials.
- **Privacy preserved**: no one knows which user corresponds to which index.
- **Fast querying**: a simple HTTP GET and bitwise operation is enough.

## Use cases
#### 1. M2M Revocation
Automatic agents or machines with delegated credentials can be easily revoked when the mandate ends.

#### 2. Temporary access
Useful in platforms with time-limited passes or event-based credentials.

#### 3. Legal compliance
Enables traceability in accordance with frameworks like eIDAS 2.0 or AML without compromising privacy.

## Best practices for implementation

- Sign the status list as a JWT.
- Use GZIP + Base64URL for the `encodedList`.
- Centralize index management.
- Do not expose the full list in frontend applications.
- Validate that bits are within valid index ranges.

## Comparison of revocation methods

| Method                    | Scalability   | Privacy   | Efficient querying   |
|---------------------------|---------------|-----------|----------------------|
| ------------------------- | ------------- | --------- | -------------------- |
| Traditional CRL           | Low           | No        | No                   |
| OCSP                      | Medium        | Partial   | Medium               |
| BitstringStatusList       | High          | Yes       | Yes                  |

## Lessons learned
1. A status list should be considered a core component in the issuer architecture. 
2. Sometimes multiple files are needed (status/1, status/2, ...). 
3. Status lists are credentials too: they must be signed, dated, and valid. 
4. Avoid exposing used indices publicly (risk of correlation).

## Beyond code: interoperability and trust
This model has been recommended by several European initiatives like EBSI, ESSIF, or Gaia-X. It enables:

- Interoperability without redefining semantics.
- Lightweight verifier integration (e.g., mobile wallets).
- Federated state validation without trusted servers.

## Conclusion

The *Bitstring Status List* model not only enables efficient and private revocation—it also provides a clear separation between:

- Credential issuance
- Status validation

By understanding this architecture, we can design more robust SSI solutions aligned with principles like privacy by design, interoperability, and scalability. Most importantly: ready for the regulatory frameworks that are coming.

Was this article useful? Follow me on [LinkedIn](https://www.linkedin.com/in/oriolcanades/) or subscribe to my newsletter for more insights on digital identity, SSI, and standards like OpenID4VC and eIDAS2.
