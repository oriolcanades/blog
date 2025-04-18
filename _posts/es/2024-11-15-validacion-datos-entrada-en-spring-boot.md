---
layout: post
title: "Validación de datos de entrada en Spring Boot"
date: 2024-11-15
categories: [Spring Boot, Seguridad]
tags: [JSR-303, Java, Spring Boot, Validaciones, Seguridad]
author: Oriol Canadés
version: 1.2
lang: es
---

## Introducción

En el desarrollo de aplicaciones Java, especialmente cuando trabajamos con **Spring Boot**, garantizar la integridad y la validez de los datos de entrada es fundamental. Aquí es donde entra en juego la especificación Java para la validación de datos, más conocida como **JSR-303**. Este estándar proporciona un marco robusto para validar datos de manera fácil y eficiente, lo que resulta ser un recurso invaluable para los desarrolladores, especialmente para aquellos que están comenzando su camino en el mundo de Spring Boot.

## ¿Qué es JSR-303?

**JSR-303**, o *Solicitud de Especificación de Java 303*, es una API de validación que permite a los desarrolladores aplicar fácilmente reglas de validación a los modelos de datos de sus aplicaciones. Es una opción popular para la validación de entrada en aplicaciones desarrolladas con Spring Boot, dada su capacidad para integrarse de manera fluida y su amplia gama de características.

## Configurando JSR-303 en tu proyecto Spring Boot
Para comenzar a utilizar JSR-303 en tu proyecto Spring Boot, el primer paso es incluir la dependencia necesaria en tu archivo `pom.xml` (Maven) o `build.gradle (Gradle). Esto habilitará la funcionalidad de validación en tu proyecto:

### Maven
```Maven
<dependency> 
  <groupId>org.springframework.boot</groupId> 
  <artifactId>spring-boot-starter-validation</artifactId> 
</dependency>
```

### Gradle
```Gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-validation'
}
```

Una vez agregada la dependencia, estás listo para implementar la validación en tus controladores.

# Implementación Básica

Para usar una validación en un controlador, simplemente lo pasas como argumento en un método del controlador anotado con `@Valid`. Esto activará la validación automática de Spring al recibir una solicitud, y si hay errores de validación, estos se pueden manejar o reportar adecuadamente.

```Java
@RestController
@RequestMapping("/api/v1/users")
public class AppUserController {

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void createUser(@Valid UserDTO userDTO, BindingResult result) {
        if (result.hasErrors()) {
            throw new InvalidAttributesException(result.message);
        }
        // Code here
    }
}
```

En este controlador, si hay errores de validación, el objeto `BindingResult` los captura, permitiéndote decidir cómo manejar estos errores, como enviarlos de vuelta al usuario.



## ¿Qué configuraciones debe tener el UserDTO?

Existen muchas configuraciones, pero vamos a ver las básicas:
    
```Java
import javax.validation.constraints.Email;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotBlank;

public record UserDTO(
    @NotBlank(message = "El nombre no puede estar vacío") String name,
    @Email(message = "Correo electrónico inválido") String email,
    @Min(value = 0, message = "La edad debe ser un número positivo") int age
) {}
```

En este ejemplo:
- `@NotBlank` se asegura de que el campo name no esté vacío.
- `@Email` valida que el campo email contenga una dirección de correo electrónico válida.
- `@Min` verifica que el campo age sea un número positivo.


# Conclusión

La especificación JSR-303 junto con Spring Boot proporciona una manera elegante y eficiente de validar los datos en tus aplicaciones. Siguiendo esta guía básica, puedes implementar validaciones robustas y mejorar la calidad y seguridad de tus proyectos.

¡Espero que este artículo te haya resultado útil!
