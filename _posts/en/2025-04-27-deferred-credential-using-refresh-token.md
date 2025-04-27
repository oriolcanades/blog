---
layout: post
title: "How to Ensure Deferred Credential Issuance Using Refresh Tokens"
date: 2025-04-27
categories: [Digital Identity]
author: Oriol Canadés
version: 1.0
lang: en
permalink: /en/digital-identity/2025/04/27/deferred-credential-using-refresh-token/
translated_url: /es/identidad-digital/2025/04/27/credencial-diferida-usando-refresh-token/
description: "Learn how to enhance resilience in OID4VCI by using Refresh Tokens in Pre-Authorized Code flows for deferred credential issuance scenarios."
---

Throughout my work on digital identity projects, I have encountered a recurring challenge: **how can we guarantee credential issuance when the process cannot be completed immediately?**

Working with issuance flows based on the [OpenID for Verifiable Credential Issuance (OID4VCI)](https://openid.github.io/OpenID4VCI/openid-4-verifiable-credential-issuance-wg-draft.html) specification, 
the Pre-Authorized Code flow facilitates scenarios where the Wallet does not require an interactive authorization step. However, in deferred issuance scenarios involving manual validations or asynchronous processes, 
a critical question arises:  
**What happens if the Access Token expires before the credential is ready to be delivered?**

In this post, I want to share how we addressed this challenge, the solutions we explored, and why using Refresh Tokens in Pre-Authorized Code flows becomes a key piece to ensure resilience in verifiable credential issuance.

## A bit of context

When a Credential Issuer cannot issue a credential immediately, it responds to the Wallet with a transaction_id, 
allowing the Wallet to retrieve the credential later via the Deferred Credential Endpoint. 
However, the specification establishes that:

- The Access Token is required to access the Deferred Credential Endpoint.
- As far as I have analyzed, there is no standardized mechanism to renew the Access Token in the Pre-Authorized Code flow.

Although optional, the OID4VCI documentation allows the issuance of a Refresh Token at the Token Endpoint. 
Implementing it properly becomes crucial in deferred issuance scenarios.

Let's look at the use cases where it would be beneficial to leverage a Refresh Token in the Pre-Authorized Code flow.

### 1. Manual signature by a responsible party (Employee onboarding)

During employee onboarding processes, an HR manager may need to manually validate and sign the credential for a new employee. This process could take hours. If the Access Token expires and no Refresh Token is available, the employee would need to restart the entire issuance process, leading to frustration and inefficiency.

### 2. Asynchronous processes in universities or government agencies

Issuing diplomas or academic certificates often requires multiple validations, institutional signatures, and processes that can take days. Without a Refresh Token, students could lose the opportunity to receive their credentials smoothly.

### 3. Infrastructure issues or operational errors

Various factors (infrastructure problems, signing errors, network outages) can prevent the immediate completion of credential issuance. If the Access Token expires during this time and no Refresh Token is available, the Wallet would lose the ability to recover the credential.

## How do we use the Refresh Token in Pre-Authorized Code flows?

Although the Pre-Authorized Code flow does not include an Authorization Request, OAuth 2.0 ([RFC6749](https://datatracker.ietf.org/doc/html/rfc6749)) allows the issuance of a Refresh Token. To implement it correctly:

1. Issue a short-lived Access Token (5–10 minutes).
2. Issue a Refresh Token valid for 7, 30, or 90 days.
3. Associate the Refresh Token to the specific transaction_id.
4. Allow Access Token renewal only if:
   - The `transaction_id` is still valid.
   - The credential has not been issued yet.

Let's visualize the workflow:

![Deferred Credential Using Refresh Token](/assets/img/posts/deferred-credential-using-refresh-token.png)

Upon receiving a Credential Response with a `transaction_id`:

1. The Wallet stores:
   - `access_token`, `refresh_token`, `expires_in`, `transaction_id`, timestamps.

2. It immediately attempts to retrieve the credential:
   - If it receives `issuance_pending`, it waits according to the indicated `interval`.

3. Before each retrieval attempt:
   - It checks whether the Access Token is still valid.
   - If it has expired and a valid Refresh Token exists, it uses it to obtain a new Access Token.

4. If the Refresh Token expires:
   - The session is cleared, and the user must request a new Credential Offer from the Issuer.

## Risks and security considerations

- **Secure storage:** The Wallet must encrypt stored Refresh Tokens.
- **Secure linkage:** Explicitly associate each Refresh Token to a unique `transaction_id`.
- **Controlled reuse:** Prevent multiple unauthorized uses of the same Refresh Token.
- **Revocation:** The Issuer must invalidate the Refresh Token once the credential is issued.
- **Monitoring:** Anomalous usage of Refresh Tokens must be logged and analyzed.
- **Attempt limitation:** Prevent abuse or denial-of-service scenarios.

## Recommendations if you plan to implement this solution

- Validate the `transaction_id` before allowing Access Token renewal.
- Design Wallets capable of automatically managing token expiration.
- Provide clear and auditable logs for reattempts and Refresh Token usage.

## Final notes

Implementing Refresh Tokens in Pre-Authorized Code flows is crucial to ensure the resilience and usability of verifiable credential issuance systems. It is not just about complying with the specification, but about building trust in digital identity processes.

Modern enterprise systems must support asynchronous flows, interruptions, and variable waiting times without impacting user experience. Robust deferred issuance is a fundamental piece to achieve this.

**Have you faced challenges with deferred credential issuance in your digital identity projects? I'd love to hear about your experiences!**
