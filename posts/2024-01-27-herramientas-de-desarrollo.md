---
layout: post
title:  "Herramientas de desarrollo"
date:   2024-01-27
categories: [java, spring-boot, tools]
---

# Herramientas de desarrollo
_27/01/2024 - v1.0.0 - Oriol Canadés_

## El IDE

IntelliJ IDEA es actualmente, el IDE más recomendable para proyectos Java y Spring, debido a su integración superior con estos entornos.

IntelliJ ofrece características destacadas como la integración nativa con frameworks de Spring, un potente motor de refactorización y una excelente gestión de dependencias Maven y Gradle. 

Aunque también apoyo el uso de Eclipse o SpringToolSuite según las preferencias personales, aliento la utilización de IntelliJ, especialmente por su capacidad para facilitar la colaboración. 

Si todos los miembros del equipo trabajan con un mismo tipo de IDE y una versión uniforme, compartir soluciones y resolver problemas en equipo se vuelve más eficiente.

IntelliJ IDEA está disponible en dos versiones: una gratuita llamada Community, que es suficiente para la mayoría de los proyectos, y otra de pago con funciones avanzadas.

[Descarga IntelliJ IDEA aquí](https://www.jetbrains.com/es-es/idea/download/)

### Plugins en IntelliJ

Para maximizar la eficiencia y calidad del desarrollo, recomendamos los siguientes plugins para IntelliJ:

**SonarLint**: Un imprescindible para análisis de código en tiempo real. Detecta bugs, errores y code smells. Al vincularlo con SonarCloud, se potencia significativamente su capacidad analítica. Este plugin es particularmente útil para mantener los altos estándares de calidad en proyectos Java/Spring.

**Lombok**: Reduce la verbosidad del código Java mediante anotaciones como @Getter, @Setter, @Builder. Aunque mejora la legibilidad, es importante entender su impacto y usarlo conscientemente para mantener la claridad del código para todos los miembros del equipo.

**PlantUML Integration**: Esencial para la creación de diagramas de secuencia y su mantenimiento dentro del repositorio de código. Este plugin ayuda a mantener una documentación visual actualizada y accesible.

**Diagrams.net Integration**: Permite la creación y edición de diagramas de flujo y otros tipos de diagramas. Es una herramienta útil para la documentación de procesos y flujos de trabajo.

## Otros programas

### Docker Desktop

Una herramienta vital para la creación de entornos de desarrollo virtualizados. Facilita el manejo de dependencias de software externas como bases de datos y herramientas de monitoreo y registro. En proyectos de microservicios, Docker asegura la consistencia entre los entornos de desarrollo y producción. Cada proyecto incluirá una carpeta /config/docker con el archivo compose.yml necesario.

[Descarga Docker Desktop aquí](https://www.docker.com/products/docker-desktop/)

### DBeaver

Una interfaz gráfica avanzada para la gestión de bases de datos. Permite agregar conexiones y facilita la interacción con tablas y esquemas, lo que es especialmente útil en entornos de bases de datos complejos.

[Descarga DBeaver aquí.](https://dbeaver.io/download/)

### Postman

Indispensable para la prueba y desarrollo de API. Aunque se recomienda adoptar estrategias de TDD o BDD, Postman ofrece una forma robusta y flexible de probar API. Las colecciones de pruebas pueden exportarse en formato JSON y añadirse a la carpeta /config/postman del proyecto, permitiendo a todos los desarrolladores importarlas y utilizarlas localmente.

[Descarga Postman aquí.](https://www.postman.com/downloads/)
