---
layout: post
title: "Implementando eIDAS2: lecciones reales desde dentro"
date: 2025-04-21
categories: [Identidad Digital]
author: Oriol Canadés
version: 1.1
lang: es
---

# Introducción

Durante los últimos cuatro años he estado trabajando diseñando y construyendo soluciones técnicas relacionadas con la identidad digital. Hemos implementado servicios que actúan como emisores de credenciales verificables (Credential Issuers), herramientas que validan esas credenciales (Verifiers), y wallets donde los usuarios las almacenan y presentan. Todo esto sucedía antes de que eIDAS2 fuera aprobado oficialmente por la Unión Europea, y a raíz del bagaje heredado del equipo previo a mi entrada en este sector, que fue el encargado de desarrollar uno de los wallets utilizados durante la pandemia.

Hace poco me senté a releer el reglamento eIDAS2, aprobado en abril de 2024, para revisar las implicaciones que tenía sobre las soluciones que habíamos diseñado y construido. Por eso, hoy quiero compartir algunas lecciones aprendidas y reflexiones sobre lo que significa implementar eIDAS2 desde dentro. Este artículo no es un análisis legal ni una guía técnica detallada. Es un relato honesto sobre lo que funciona, lo que aún está verde y lo que nadie te cuenta hasta que te toca implementarlo de verdad.

Vamos por el principio.

## ¿Qué es eIDAS y qué cambia con eIDAS2?

La regulación **eIDAS** 910/2014 fue el primer intento europeo de establecer un marco común para la identificación electrónica y los servicios de confianza. Permitía firmar documentos electrónicamente con validez legal y establecía las bases para identificar a ciudadanos entre estados miembros. Pero tenía limitaciones importantes: su adopción era voluntaria, estaba muy centrado en el sector público y su implementación fue muy desigual. Prueba de ello es que, a día de hoy y después de haber trabajado en algunos proyectos europeos, hay países que aún no disponen de un sistema de identificación electrónica para entidades legales o jurídicas que cumpla con eIDAS.

La regulación **eIDAS2** 2024/1183 es una evolución ambiciosa. Con ella, se introduce la Cartera Europea de Identidad Digital (EUDI Wallet), obligando a los Estados miembros de la UE a proporcionar al menos una, y extiende su uso al sector privado. Además, permite almacenar y compartir atributos como el título universitario, el permiso de conducir o una representación legal, bajo el control exclusivo del usuario.

En otras palabras, mientras que eIDAS abría la puerta a la firma digital, eIDAS2 propone una infraestructura completa para la identidad digital europea.

## Ser "compatible con eIDAS2" no es lo que crees.

Uno de los errores más comunes que veo es pensar que usar JWT o implementar un login con credenciales verificables (VC) ya te hace compatible con eIDAS2. Nada más lejos de la realidad.

eIDAS2 exige:
- Que las identidades y atributos estén emitidos por **prestadores cualificados de servicios de confianza**, sujetos a certificación.
- Que los **atributos estén firmados electrónicamente**, en particular mediante **firmas electrónicas cualificadas** que permitan su validación independiente.
- Que los usuarios tengan control total sobre qué comparten y con quién, mediante mecanismos como la **divulgación selectiva** (*Selective Disclosure*).
- Que los datos estén protegidos con **separación lógica de funciones**, e idealmente también física, en los componentes críticos como la wallet.

Y eso es solo el principio. A nivel técnico, muchos flujos cambian radicalmente. Por ejemplo, ya no hablamos solo de tokens de acceso, sino de **presentaciones verificables (VP)** que encapsulan credenciales firmadas (**VC**) en formatos como *jwt_vc_json*. Además, la identidad ya no se limita al individuo: también abarca a **personas jurídicas**, **dispositivos**, **organizaciones**, e incluso **delegaciones de poder** entre ellos.

## ¿Está todo resuelto en eIDAS2?

No. Aún hay muchas áreas verdes:
- Los **esquemas de atributos** no están estandarizados y quedan a la espera de actos de ejecución.
- Las herramientas de verificación con UI son aún escasas o muy fragmentadas.
- No existe aún un patrón claro para **wallets empresariales** o entornos B2B.
- La coordinación entre reguladores, técnicos y usuarios va lenta.
- Las **especificaciones internacionales** aún están en fase de borrador o cambian con frecuencia.
- La interoperabilidad entre distintos formatos de credenciales y wallets requiere esfuerzos importantes de adaptación.
- Los marcos de confianza y auditoría técnica aún son vagos y dependen de implementación nacional. Sin entrar en aquellos que sean de naturaleza privada y que requieran de interoperabilidad entre distintos proveedores.

Pero estamos en un punto histórico. Por primera vez, Europa ha definido un marco legal, técnico y ético que puede transformar la forma en que gestionamos la identidad digital. Ahora toca a los arquitectos, desarrolladores y líderes de producto convertir esa visión en software real. Y ya hay varios proyectos en marcha, casi todos en la línea de servicios para el ciudadano. Los cuatro grandes proyectos piloto (Large Scale Pilots) son:

- **POTENTIAL**: impulsa innovación y colaboración en sectores clave como servicios gubernamentales, banca, telecomunicaciones, licencias de conducir móviles, firmas electrónicas y salud.
- **EU Digital Wallet Consortium (EWC)**: enfocado en credenciales de viaje digitales (DTC), construyendo sobre la aplicación de referencia del wallet europeo.
- **NOBID**: conjunto de países nórdicos y bálticos, junto con Italia y Alemania, que exploran el uso del wallet para autorización de pagos por parte del usuario.
- **DC4EU**: liderado por la Comisión Europea, busca ofrecer una infraestructura interoperable para servicios públicos y privados transfronterizos.

## Conclusión

Implementar eIDAS2 no es simplemente actualizar tu sistema de login. Es repensar cómo representas a tus usuarios, a tus servicios, a tus máquinas. Es entender que las identidades ya no son solo "cosas que verificas", sino "elementos vivos que interactúan, firman, delegan y prueban cosas por sí mismas".

Este viaje no lo hicimos solos. Lo hicimos preguntando, probando, equivocándonos y volviendo a intentarlo. Y todo lo aprendido está hoy ayudando a definir nuevas formas de autenticación, autorización y confianza en el mundo digital europeo.

Si estás empezando con eIDAS2, mi único consejo es este: **no empieces por la Wallet**. Empieza por entender los **flujos**, los **poderes delegables**, las **credenciales cualificadas**, los **roles técnicos** que impone el reglamento, y luego deja que la tecnología se adapte a ti, no al revés.

## Referencias

- [Reglamento eIDAS 910/2014](https://eur-lex.europa.eu/legal-content/ES/ALL/?uri=celex:32014R0910)
- [Reglamento eIDAS 2 2024/1183](https://eur-lex.europa.eu/eli/reg/2024/1183/oj)
- [Large Scale Pilots](https://ec.europa.eu/digital-building-blocks/sites/display/EUDIGITALIDENTITYWALLET/What+are+the+Large+Scale+Pilot+Projects)
