---
layout: post
title:  "Manejo de excepciones en Spring Boot"
date:   2024-03-09
categories: [java, spring-boot, error-handling]
---

# Manejo de excepciones en Spring Boot
_09/03/2024 - v1.0.0 - Oriol Canadés_

## Introducción

Vamos a hablar del manejo de excepciones en Spring Boot. En un proyecto Spring Boot, es importante manejar las excepciones de manera adecuada. En este post, vamos a ver cómo manejarlas de manera global y entender cómo funciona.

Para simplificar la gestión de errores tanto como sea posible, necesitamos construir ciertas estructuras. Una de ellas es el `GlobalExceptionHandler`, que se encarga de centralizar el manejo de las excepciones que puedan ocurrir durante una solicitud y devolver una respuesta adecuada.

## ¿Cómo funciona el GlobalExceptionHandler en SpringBoot?

Cuando ocurre una excepción, la aplicación de Spring Boot la redirigirá al GlobalExceptionHandler. Dependiendo del tipo específico de excepción, el GlobalExceptionHandler puede generar un mensaje de error adecuado y devolverlo al cliente. Esto ayuda a proporcionar mensajes de error más significativos a los clientes y también se puede utilizar para registrar información detallada sobre las excepciones.

El uso y aplicación del GlobalExceptionHandler tiene varios puntos que debemos tener en cuenta:

- **Controladores**: Los controladores de Spring Boot son responsables de manejar las solicitudes y devolver las respuestas adecuadas. Si ocurre una excepción, el controlador la redirigirá al GlobalExceptionHandler, por tanto el controlador ya no incluirá un try-catch para manejar las excepciones.
- **Excepciones**: Siempre trabajaremos con excepciones personalizadas (*custom*) que extiendan de `RuntimeException`. De esta manera, podemos tener un control más preciso sobre las excepciones que ocurren en la aplicación y además, cuando trabajemos con try-catch en nuestros métodos, capturaremos la excepción genérica y lanzaremos una excepción personalizada para que el GlobalExceptionHandler la maneje. Las excepciones pueden tener métodos para exponer un mensaje o un mensaje y el throwable original.
- **Logging**: Es importante registrar las excepciones que ocurren en la aplicación. El GlobalExceptionHandler puede registrar las excepciones que maneja, pero también es importante registrar las excepciones que ocurren en otros lugares de la aplicación. Por tanto, es importante tener un buen sistema de logging en la aplicación. Por ello, es recomendable usar el `log.error()` para registrar las excepciones en el GlobalExceptionHandler y el `log.warn()` para registrar las excepciones en otros lugares de la aplicación.

Vamos a ver un ejemplo de cómo implementar un GlobalExceptionHandler en Spring Boot.

### Controller

Como podemos ver, el controlador no incluye un try-catch para manejar las excepciones. Si ocurre una excepción, la redirigirá al GlobalExceptionHandler. El controlador se encarga del happy path y el GlobalExceptionHandler se encarga de manejar las excepciones.

```java
@RestController
@RequestMapping("/api")
@RequiredArgsConstructor
public class UserController {

    private UserService userService;

    @GetMapping("/users/{id}")
    @ResponseStatus(HttpStatus.OK)
    public UserResponse getUser(@PathVariable Long id) {
        return userService.getUser(id);
    }

    @PostMapping("/users")
    @ResponseStatus(HttpStatus.CREATED)
    public void createUser(@RequestBody UserRequest userRequest) {
        return userService.createUser(userRequest);
    }
}
```

### Exception

La excepción personalizada que extiende de `RuntimeException` y que se lanzará en el servicio. La excepción puede tener un mensaje o un mensaje y el throwable original como habíamos comentado anteriormente.

```java
public class UserNotFoundException extends RuntimeException {

    public UserNotFoundException(Long id) {
        super("User not found with id: " + id);
    }
    
    public UserNotFoundException(Long id, Throwable throwable) {
        super("User not found with id: " + id, throwable);
    }
    
}
```
### Service

En este servicio, si ocurre una excepción, lanzaremos una excepción personalizada que será manejada por el GlobalExceptionHandler.

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private UserRepository userRepository;
    private UserMapper userMapper;

    public UserResponse getUser(Long id) {
        User user = userRepository.findById(id)
                .map(Uer::new)
                .orElseThrow(() -> new UserNotFoundException(id));
        return userMapper.toUserResponse(user);
    }

    public void createUser(UserRequest userRequest) {
        userRepository.save(new User(userRequest));
    }
    
}
```

### GlobalExceptionHandler

El GlobalExceptionHandler manejará las excepciones que ocurran en la aplicación. En este caso, si ocurre una excepción de tipo `UserNotFoundException`, devolverá un mensaje de error adecuado y un código de estado 404.

```java
import java.lang.annotation.Repeatable;

@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public GlobalErrorResponse handleUserNotFoundException(UserNotFoundException ex) {
        log.error("User not found", ex);
        return new GlobalErrorResponse(ex.getMessage());
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public GlobalErrorResponse handleException(Exception ex) {
        log.error("Internal server error", ex);
        return new GlobalErrorResponse("Internal server error");
    }

}
```

### GlobalErrorResponse

Un punto importante es definir el modelo de respuesta de error que devolverá el GlobalExceptionHandler. En este caso, devolverá un mensaje de error.

```java
@Data
@AllArgsConstructor
public class GlobalErrorResponse {
    private String title;
    private String message;
    private String path;
}
```

Si el proyecto implementa Java 17, podemos definir la clase del modelo de respuesta de error como un record.

```java 
public record GlobalErrorMessage(String title, String message, String path) {
}
```

## Recursos
- [Error Handling for REST with Spring](https://www.baeldung.com/exception-handling-for-rest-with-spring)
- [Spring Boot Global Exception Handler](https://medium.com/@aedemirsen/spring-boot-global-exception-handler-842d7143cf2a)

