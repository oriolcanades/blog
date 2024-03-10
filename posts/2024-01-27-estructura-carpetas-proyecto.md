---
layout: post
title:  "Estructura de carpetas en un proyecto Java/Spring"
date:   2024-01-27
categories: [java, spring-boot, tools]
---

# Estructura de carpetas en un proyecto Java/Spring
_27/01/2024 - v1.0.0 - Oriol Canadés_

## Introducción

Todo proyecto necesita una estructura básica para organizarse de manera efectiva. En esta entrada explicaremos como podemos estructurar un proyecto Java/Spring. Un buen abordaje sería apostar por una organización por capas, aunque también podría usarse una organización por características/features.

Este podría ser un ejemplo base de la estructura de un proyecto:

```plaintext
|- .github / pipelines
    |- workflows
    |- CODEOWNERS
|- .mvn / gradle/grapper
|- config
    |- checkstyle
    |- docker
    |- monitoring
|- docs
    |- diagrams
    |- img
|- src
    |- main
        |- java
            |- config
            |- controller
            |- exception
            |- facade
            |- model
            |- repository
            |- service
            |- util
        |- resources
            |- db/migrations
            |- application.yml
    |- test
|- .gitignore
|- Dockerfile
|- CHANGELOG
|- README.md
|- pom.xml / build.gradle
```
