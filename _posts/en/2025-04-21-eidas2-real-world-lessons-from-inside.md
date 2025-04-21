---
layout: post
title: "Implementing eIDAS2: real-world lessons from the inside"
date: 2025-04-21
categories: [Digital Identity]
author: Oriol Canadés
version: 1.1
lang: en
---

# Introduction

For the past four years, I’ve been designing and building technical solutions related to digital identity. We’ve implemented services that act as **Credential Issuers**, tools that validate those credentials (**Verifiers**), and wallets where users can store and present them. All of this happened before **eIDAS2** was officially approved by the European Union, and built on the foundations laid by the team I joined — the same team that had developed one of the wallets used during the pandemic.

Recently, I sat down to re-read the eIDAS2 regulation, approved in April 2024, to assess how it affects the solutions we've designed and built. That’s why I want to share some lessons learned and reflections on what it really means to implement eIDAS2 from the inside. This article is not a legal analysis or a step-by-step technical guide. It’s an honest account of what works, what’s still unclear, and what no one tells you until you're actually building it.

Let’s start from the beginning.

## What is eIDAS and what changes with eIDAS2?

The **eIDAS Regulation** 910/2014 was the EU’s first attempt to establish a common framework for electronic identification and trust services. It allowed documents to be digitally signed with legal validity and laid the groundwork for recognizing citizens across Member States. But it had significant limitations: its adoption was voluntary, focused heavily on the public sector, and implementation varied widely. As someone who’s worked on multiple EU projects, I can confirm that even today, some countries still lack electronic identification systems for legal entities that comply with eIDAS.

The **eIDAS2 Regulation** 2024/1183 is a much more ambitious evolution. It introduces the **European Digital Identity Wallet (EUDI Wallet)**, requiring each EU Member State to provide at least one. It also expands its use into the private sector and allows users to store and share attributes — such as university degrees, driving licenses, or legal representations — under their exclusive control.

In short, while eIDAS opened the door to digital signing, eIDAS2 proposes a full-fledged infrastructure for European digital identity.

## Being “eIDAS2 compliant” isn’t what you think

One of the most common misconceptions I see is that using JWTs or implementing login with Verifiable Credentials (VCs) makes your system eIDAS2-compliant. Nothing could be further from the truth.

eIDAS2 requires that:
- Identities and attributes are issued by **Qualified Trust Service Providers**, subject to formal certification.
- Attributes are **electronically signed**, ideally with **Qualified Electronic Signatures** for independent validation.
- Users have full control over what they share and with whom, through mechanisms like **Selective Disclosure**.
- Data is protected through **logical separation of functions**, and ideally also **physical separation** in critical components like the wallet.

And that’s just the beginning. Technically, many flows change fundamentally. We’re no longer just talking about access tokens, but about **Verifiable Presentations (VPs)** that encapsulate digitally signed **Verifiable Credentials (VCs)**, using formats like *jwt_vc_json*. Identity is no longer just about individuals — it now includes **legal entities**, **devices**, **organizations**, and even **delegated powers** between them.

## Is everything resolved in eIDAS2?

Not at all. There are still many open areas:
- **Attribute schemas** are not standardized yet and depend on future delegated acts.
- UI-ready verification tools are still scarce or fragmented.
- There is no clear standard for **enterprise wallets** or B2B environments.
- Coordination between regulators, developers, and end users is still slow.
- Many international specifications are in draft form or subject to change.
- Interoperability between different credential formats and wallets remains a major challenge.
- Trust and audit frameworks are still vague and mostly defined at the national level.

But we are at a historic turning point. For the first time, Europe has defined a legal, technical, and ethical framework that can reshape how we manage digital identity. Now it’s up to architects, developers, and product leaders to turn that vision into working software. Several initiatives are already underway, mostly focused on citizen-facing services. The four major Large Scale Pilot projects are:

- **POTENTIAL**: promotes innovation and collaboration in key sectors like public services, banking, telecom, mobile driver’s licenses, e-signatures, and health.
- **EU Digital Wallet Consortium (EWC)**: focuses on Digital Travel Credentials (DTCs), building on the EU reference wallet application.
- **NOBID**: a group of Nordic and Baltic countries, along with Italy and Germany, piloting wallet-based payment authorizations by end users.
- **DC4EU**: led by the European Commission, this project aims to deliver an interoperable infrastructure for public and private cross-border services.

## Conclusion

Implementing eIDAS2 isn’t just about upgrading your login system. It’s about rethinking how you represent your users, your services, your machines. It’s about understanding that identities are no longer just things you verify — they are living entities that interact, sign, delegate, and prove things by themselves.

This journey wasn’t one we did alone. We asked, we built, we failed, and we tried again. Everything we’ve learned is now helping shape new models of authentication, authorization, and trust in Europe’s digital landscape.

If you're just starting with eIDAS2, here’s my only piece of advice: **don’t start with the wallet**. Start by understanding the **flows**, the **delegable powers**, the **qualified credentials**, the **technical roles** defined by the regulation — and let the technology adapt to you, not the other way around.

## References

- [eIDAS Regulation 910/2014](https://eur-lex.europa.eu/legal-content/EN/ALL/?uri=celex:32014R0910)
- [eIDAS2 Regulation 2024/1183](https://eur-lex.europa.eu/eli/reg/2024/1183/oj)
- [Large Scale Pilots](https://ec.europa.eu/digital-building-blocks/sites/display/EUDIGITALIDENTITYWALLET/What+are+the+Large+Scale+Pilot+Projects)
