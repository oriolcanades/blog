---
layout: post
title:  "Perfiles Spring"
date:   2024-01-30
categories: [java, spring-boot, profiles]
---

# Perfiles Spring
_30/01/2024 - v1.0.0 - Oriol Canadés_

## Perfiles

* Default → application.yml
* Dev → application-dev.yml
* Test → application-test.yml (ambiente de Preproducción o QA)
* Prod → application-prod.yml

## Buenas Prácticas

* **Separación de Configuraciones**: Mantén las configuraciones específicas del entorno separadas de la configuración por defecto. Esto mejora la legibilidad y el mantenimiento.


* **Uso de Variables de Entorno**: Para datos sensibles como contraseñas o tokens, usa variables de entorno en lugar de hardcoding en los archivos YAML.


* **Consistencia en Propiedades**: Asegúrate de que todas las configuraciones relevantes estén presentes en cada archivo de perfil, incluso si algunos valores son los mismos en diferentes entornos. Esto evita la confusión sobre qué propiedades están activas en cada perfil.


* **Perfiles por Funcionalidad**: Además de separar por entorno, considera usar perfiles para características específicas, como 'batch' o 'scheduling', lo que permite activar o desactivar ciertas funcionalidades fácilmente.


* **Control de Versiones**: Versiona tus archivos de configuración para rastrear cambios y mantener un historial de modificaciones.

## Anti-Patterns

* **Configuraciones Duplicadas**: Evita duplicar la misma configuración en múltiples archivos de perfil. Utiliza el archivo application.yml para propiedades comunes.


* **Datos Sensibles en el Código**: Nunca incluyas credenciales o datos sensibles directamente en los archivos de configuración.


* **Configuraciones Específicas del Entorno en el Código**: Evita definir configuraciones específicas del entorno dentro del código. En su lugar, utiliza los archivos de perfil y variables de entorno.


* **Sobrecarga de Perfiles**: No crees demasiados perfiles que pueden hacer que la gestión de configuraciones sea complicada y propensa a errores.


* **Falta de Documentación**: Documenta tus archivos de configuración y asegúrate de que el equipo comprenda el propósito de cada perfil.
