---
layout: post
title: "Estructura del proyecto Spring Boot: Una guía práctica"
date: 2025-01-14
categories: [Spring Boot, Design Patterns, Software Architecture]
tags: [Java, Spring Boot, Patrones de Diseño]
author: Oriol Canadés
version: 1.1
---

## Introducción

Cuando comienzas un proyecto en Spring Boot, una de las primeras decisiones críticas es cómo estructurarlo. 
Una organización adecuada del código puede ser la diferencia entre un proyecto fácil de mantener y uno que se convierta en un caos.

En este artículo, quiero compartir una estructura que he utilizado en varios proyectos con éxito. 
Aunque no existe una "estructura universal", esta guía te ofrecerá un buen punto de partida que puedes adaptar según tus necesidades.

## Estructura de proyecto

Recomiendo dividir el código en tres capas principales:
```text
  - application
  - domain
  - infrastructure
```

### Capa de aplicación

Esta capa maneja los flujos de trabajo (_workflows_) de la aplicación. Utiliza el patrón **Facade** para encapsular la lógica de negocio y proporcionar una interfaz sencilla al resto de las capas.

Principales responsabilidades:
- **Orquestar la lógica de negocio**: La capa de aplicación interactúa con los servicios de dominio.
- **Transformación de datos**: Convierte DTOs en entidades y viceversa utilizando mappers dedicados.
- **Gestión de excepciones**: Maneja errores de manera centralizada y genera respuestas significativas para el cliente.

Ejemplo de estructura:
```text
- application
    - exceptions
    - dto
    - mappers
    - workflows
        - impl
```

Ejemplo de código:
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

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class CreateOrderWorkflowImpl implements CreateOrderWorkflow {

    private final OrderMapper orderMapper;
    private final OrderService orderService;
    private final NotificationService notificationService;
    private final AuditService auditService;

    @Override
    public void execute(CreateOrderRequest request) {
        try {
            Order order = orderMapper.toEntity(request);
            orderService.create(order);
            notificationService.send(new Notification("Order created"));
            auditService.registry(INFO, "Order with id: " + order.getId() + " was created successfully");
        } catch (OrderAlreadyExistsException e) {
            log.warn("Order creation failed: {}", e.getMessage());
            notificationService.send(new Notification("Error creating order"));
            auditService.registry(ERROR, "Error creating order");
            throw new OrderCreationException("Failed to create order", e);
        }
    }
}
```

### Capa de dominio

La capa de dominio contiene la lógica de negocio central y debe mantenerse independiente de otras capas. Se organiza en:
```text
  - domain
    - entities
    - exceptions
    - repositories
    - services
```

Ejemplo de código:

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String description;
    private BigDecimal price;

    // Getters y setters
}
```

```java
public interface OrderRepository extends JpaRepository<Order, Long> {
}
```

```java
@Service
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {

    private final OrderRepository orderRepository;

    @Override
    public Order create(Order order) {
        if (orderRepository.existsById(order.getId())) {
            throw new OrderAlreadyExistsException("Order already exists with ID: " + order.getId());
        }
        return orderRepository.save(order);
    }
}
```

### Capa de infraestructura

La capa de infraestructura gestiona la interacción con el "mundo exterior": bases de datos, APIs y la capa de presentación. Contiene controladores, repositorios y configuraciones.

Estructura sugerida:
```text
  - infrastructure
    - adapters
    - controllers
    - configurations
```

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

#### Manejo global de excepciones

Centralizar el manejo de errores asegura respuestas consistentes:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(OrderCreationException.class)
    public ResponseEntity<ErrorResponse> handleOrderCreationException(OrderCreationException ex) {
        log.error("Error ID: {} - {}", ex.getErrorId(), ex.getMessage());
        ErrorResponse errorResponse = ErrorResponse.builder()
                .type("OrderCreationException")
                .status(HttpStatus.BAD_REQUEST.value())
                .message(ex.getMessage())
                .build();
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(errorResponse);
    }
}
```

## Últimos consejos

En las clases de excepciones, utiliza un identificador único para cada error. 
Esto facilita la búsqueda y el seguimiento de errores en los registros.

```java
public class OrderAlreadyExistsException extends RuntimeException {
    private final String errorId;
    
    public OrderAlreadyExistsException(String message) {
        super(message);
        this.errorId = UUID.randomUUID().toString();
    }
    
    public String getErrorId() {
        return errorId;
    }
}
```

## Conclusión

Esta estructura proporciona una base sólida para desarrollar aplicaciones Spring Boot escalables y mantenibles. Recuerda que cada proyecto es único, y esta guía debe adaptarse a tus necesidades específicas.

¿Y tú? ¿Cómo estructuras tus proyectos Spring Boot? ¡Comparte tus experiencias y opiniones en los comentarios!

¡Gracias por leer! 🚀

### Referencias
- [Spring Boot Reference Guide](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
- [Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.oreilly.com/library/view/domain-driven-design-tackling/0321125215/)
- [Clean Architecture: A Craftsman's Guide to Software Structure and Design](https://www.oreilly.com/library/view/clean-architecture-a/9780134494272/)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
- [Facade Pattern](https://refactoring.guru/design-patterns/facade)
