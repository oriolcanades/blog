---
layout: post
title: "Estructura del proyecto Spring Boot: Una guía práctica"
date: 2025-01-10
categories: [Spring Boot, Design Patterns, Software Architecture]
tags: [Java, Spring Boot, Patrones de Diseño]
author: Oriol Canadés
---

## Introducción

Al iniciar un nuevo proyecto Spring Boot, una de las primeras preguntas que surgen es: ¿Cómo lo estructuro? La organización adecuada del código puede marcar la diferencia entre un proyecto fácil de escalar y mantener, y uno que se convierta en un dolor de cabeza.

En este artículo, quiero compartir una estructura de proyecto que he utilizado recientemente en varios proyectos Spring Boot, obteniendo buenos resultados. Aunque no existe una "estructura perfecta", esta guía puede servirte como punto de partida y adaptarse a las necesidades específicas de tu proyecto.

## Estructura de proyecto

Recomiendo separar las capas de la aplicación en tres partes principales:
```text
  - application
  - domain
  - infrastructure
```

### Capa de aplicación

Esta capa contiene los flujos de trabajo (_workflows_) de la aplicación. Para ello, se utiliza el patrón de diseño **Facade**, que encapsula la lógica de negocio y expone una interfaz sencilla al resto de las capas.

Cada flujo de trabajo inyecta los servicios necesarios para realizar operaciones, siguiendo principios de diseño como los de SOLID (Responsabilidad única, Open/Closed e Inversión de dependencias). Las implementaciones de estos workflows se ubican en el paquete impl, mientras que los datos que fluyen entre capas se definen en el paquete dto. Esto asegura que la capa de aplicación sea independiente de la estructura de las entidades de la base de datos.

La transformación de DTOs (Data Transfer Objects) a entidades también se realiza en esta capa. Utilizar mappers dedicados permite mantener la separación de responsabilidades y evita que el dominio dependa de estructuras externas. Los DTOs encapsulan los datos que viajan entre las capas de infraestructura y dominio, manteniendo el dominio independiente de las estructuras externas y facilitando la adaptación a cambios.

Ejemplo de estructura:
```text
- application
    - workflows
        - impl
    - dto
```

Ejemplo de código:
```java
public interface CreateOrderWorkflow {
    void execute(CreateOrderRequest request);
}
```

```java
@Service
@RequiredArgsConstructor
public class CreateOrderWorkflowImpl implements CreateOrderWorkflow {

    private final OrderMapper orderMapper;
    private final OrderService orderService;
    private final NotificationService notificationService;
    private final LoggingService loggingService;
    
    @Override
    public void execute(CreateOrderRequest request) {
        try {
            Order order = orderMapper.toEntity(request);
            orderService.create(order);
            notificationService.send(new Notification("Order created"));
            loggingService.logInfo("Order with id: " + order.getId() + " was created successfully");
        } catch (Exception e) {
            notificationService.send(new Notification("Error creating order"));
            loggingService.logError("Error creating order");
            throw new OrderCreationException("Error creating order", e);
        }
    }
}
```

En este ejemplo, el flujo `CreateOrderWorkflow` orquesta la creación de una orden, delegando responsabilidades a los servicios correspondientes. Esto garantiza la reutilización y modularidad de los servicios.

### Capa de dominio

La capa de dominio encapsula las entidades de negocio, los adaptadores (_mappers_) y los servicios. Aquí reside la lógica de negocio, que se expone mediante interfaces para que la capa de aplicación pueda interactuar con ella.

Ejemplo de estructura:
```text
  - domain
    - entities
    - exceptions
    - mappers
    - services
```

#### Excepciones en el dominio

Dado que la lógica de negocio pertenece al dominio, las excepciones relacionadas con esta lógica también deben definirse y lanzarse desde esta capa. Esto asegura que los problemas relacionados con las reglas de negocio se manejen en el contexto adecuado.

Ejemplo de excepción en el dominio:
```java
public class OrderNotFoundException extends RuntimeException {
    private final String errorId;
    
    public OrderNotFoundException(String message) {
        super(message);
        this.errorId = UUID.randomUUID().toString();
    }
    
    public String getErrorId() {
        return errorId;
    }
}
```

Esta excepción podría lanzarse, por ejemplo, cuando un servicio de dominio no puede encontrar una orden específica:
```java
@Service
public class OrderServiceImpl implements OrderService {
    @Override
    public Order findOrderById(Long id) {
        return orderRepository.findById(id)
                .orElseThrow(() -> new OrderNotFoundException("Order not found with ID: " + id));
    }
}
```

#### Adaptadores en el dominio

Los adaptadores (_mappers_), que transforman entre entidades y DTOs, deben ubicarse en esta capa. Al centralizar los adaptadores aquí:

- Se asegura la consistencia en las transformaciones entre DTOs y entidades.
- Se facilita la reutilización de lógica de mapeo en todo el sistema.
- Se mantiene el dominio como propietario de su representación interna y las transformaciones necesarias.

Ejemplo de mapper en la capa de dominio:
```java
@Component
public class OrderMapper {

    public Order toEntity(CreateOrderRequest request) {
        return new Order(request.getId(), request.getDescription(), request.getPrice());
    }

    public CreateOrderResponse toDto(Order order) {
        return new CreateOrderResponse(order.getId(), order.getDescription(), order.getPrice());
    }
}
```

Los servicios en esta capa no deben interactuar directamente entre sí. En su lugar, la capa de aplicación se encarga de orquestar la comunicación. Esto evita dependencias cíclicas y mantiene un contexto claro y definido.

### Capa de infraestructura

La capa de infraestructura interactúa con el "mundo exterior": bases de datos, APIs externas y la capa de presentación. Contiene controladores, repositorios y servicios de configuración.

Ejemplo de estructura:
```text
  - infrastructure
    - adapters
    - controllers
    - repositories
    - configurations
```

#### Controladores

Los controladores deben limitarse a recibir las peticiones y delegar la lógica de negocio en la capa de aplicación. Esto asegura una separación clara de responsabilidades y evita la duplicación de código.

Ejemplo de código:
```java
@RestController
@RequestMapping("/orders")
@RequiredArgsConstructor
public class OrderController {

    private final CreateOrderWorkflow createOrderWorkflow;

    @PostMapping
    public ResponseEntity<Void> createOrder(@Valid @RequestBody CreateOrderRequest request) {
        createOrderWorkflow.execute(request);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
    
}
```

Para centralizar la gestión de excepciones y proporcionar respuestas consistentes a los consumidores de la API, se puede utilizar un `@ControllerAdvice`. Este gestor global captura excepciones lanzadas desde cualquier controlador y devuelve una respuesta adecuada en forma de un DTO.

Ejemplo de código:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(OrderNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleOrderNotFoundException(OrderNotFoundException ex) {
        RestApiErrorResponse errorResponse = buildResponse(ex, HttpStatus.NOT_FOUND);
        log.error("Error ID: {} - {}", ex.getErrorId(), ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(errorResponse);
    }
    
    private RestApiErrorResponse buildResponse(Exception ex, HttpStatus status) {
        return ErrorResponse.builder()
                .type(ex.getClass().getSimpleName())
                .title(status.getReasonPhrase())
                .status(status.value())
                .detail(ex.getMessage())
                .instance(ex.getErrorId())
                .build();
    }
    
}
```

> Los logs para consola o fichero deben generarse en los puntos de entrada principales de la aplicación (controladores o GlobalExceptionHandler) para capturar los eventos globales. Es preferible usar herramientas como SLF4J y configurarlas adecuadamente con logback o log4j.

## Conclusión

La estructura de proyecto que he presentado en este artículo es solo una de las muchas formas posibles de organizar un proyecto Spring Boot. Aunque ha demostrado ser efectiva en mi experiencia, es crucial adaptarla a las necesidades específicas de cada proyecto.

¿Y tú? ¿Cómo estructuras tus proyectos Spring Boot? ¡Comparte tus experiencias y opiniones en los comentarios!

¡Gracias por leer! 🚀

### Referencias
- [Spring Boot Reference Guide](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
- [Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.oreilly.com/library/view/domain-driven-design-tackling/0321125215/)
- [Clean Architecture: A Craftsman's Guide to Software Structure and Design](https://www.oreilly.com/library/view/clean-architecture-a/9780134494272/)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
- [Facade Pattern](https://refactoring.guru/design-patterns/facade)
```
