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

> :file_folder: .github/pipelines
> :file_folder: .mvn/gradle/grapper
> :file_folder: config
> :file_folder: docs
> :file_folder: src
>   :file_folder: main
>       :file_folder: java
>           :file_folder: config
>           :file_folder: controller
>           :file_folder: exception
>           :file_folder: facade
>           :file_folder: model
>           :file_folder: repository
>           :file_folder: service
>           :file_folder: util
>       :file_folder: resources
>           :file_folder: db/migrations
>           :page_facing_up: application.yml
>   :file_folder: test
> :page_facing_up: .gitignore
> :page_facing_up: Dockerfile
> :page_facing_up: CHANGELOG
> :page_facing_up: README.md
> :page_facing_up: pom.xml/build.gradle


