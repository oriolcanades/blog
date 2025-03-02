---
layout: post
title: "Buenas Prácticas para la Inyección de Configuraciones en Spring Boot"
date: 2025-03-02
categories: [Spring Boot, Best Practices]
tags: [Spring Boot, Configuraciones, Inyección de Dependencias, Buenas Prácticas]
author: Oriol Canadés
version: 1.0
---

Hoy quiero revisar las buenas prácticas acerca de la inyección de configuraciones definidas en el application.yaml (o application.properties) en el código de nuestro proyecto Spring. Con ello me propongo poder garantizar la mantenibilidad, seguridad y flexibilidad. Aquí te dejo las mejores estrategias:

## Uso de @Value para valores simples

Lo uso sobre todo para PoC o cuando solo necesito inyectar una propiedad puntual o propiedades no-completas.

Ejemplo en application.yaml:
```yaml
app:
  name: spring-demo
  version: 1.0.0
```

Clase en Java:
```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Getter
@Component
public class ConfigService {

    @Value("${app.name}")
    private String appName;

    @Value("${app.version}")
    private String appVersion;
    
}
```

## Uso de @ConfigurationProperties para agrupar configuraciones

Cuando necesito manejar múltiples valores o estructuras algo más complejas, la mejor estrategia es usar @ConfigurationProperties.

Ejemplo en application.yaml:
```yaml
server:
  port: 8080
  timeout: 5000

app:
  security:
    enabled: true
    roles:
      - ADMIN
      - USER
```

Clase de Configuración:
```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;
import java.util.List;

@Component
@ConfigurationProperties(prefix = "app.security")
public class SecurityProperties {

    private boolean enabled;
    private List<String> roles;

    public boolean isEnabled() {
        return enabled;
    }

    public void setEnabled(boolean enabled) {
        this.enabled = enabled;
    }

    public List<String> getRoles() {
        return roles;
    }

    public void setRoles(List<String> roles) {
        this.roles = roles;
    }
}
```

Uso en el código:
```java
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class SecurityService {

    private final SecurityProperties securityProperties;

    public void printRoles() {
        System.out.println("Available roles: " + securityProperties.getRoles());
    }
    
}
```

## Uso de Environment para leer configuraciones dinámicamente

Para acceder a propiedades de forma dinámica sin inyectarlas en una clase, puedes usar Environment. Aunque no lo he utilizado en mis proyectos, es una opción válida y útil en ciertos escenarios.

```java
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;

@Component
public class ConfigService {

    private final Environment environment;

    public ConfigService(Environment environment) {
        this.environment = environment;
    }

    public String getProperty(String key) {
        return environment.getProperty(key);
    }
}
```

## Uso de perfiles (@Profile) para cambiar configuraciones

Otra estrategia es definir valores diferentes según el perfil (dev, prod, etc.). Para proyectos que usan perfiles de Spring Boot, es una buena opción, pero desde que trabajo con Kubernetes, prefiero usar configuraciones externas (lo veremos en el último punto).

Ejemplo en application.yaml:
```yaml
# application-dev.yaml
server:
 port: 8081

# application-prod.yaml
server:
 port: 8082
```
En el código, puedes condicionar una configuración según el perfil activo:
```java
import org.springframework.context.annotation.Profile;
import org.springframework.stereotype.Service;

@Service
@Profile("dev")
public class DevService {
    public DevService() {
        System.out.println("Servicio en modo desarrollo");
    }
}
```

## Uso de @Configuration con @Bean para configuración avanzada

Reconozco que, inicialmente, no estaba familiarizado con esta práctica, pero me parece interesante. Siempre he usado la lógica en una clase con @ConfigurationProperties, pero esta estrategia permite configurar valores personalizados de forma más avanzada.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean
    public String applicationName(@Value("${app.name}") String appName) {
        return "Nombre de la aplicación: " + appName;
    }
}
```

## Uso de variables de entorno y configuración externa

Para evitar exponer información sensible en application.yaml, la mejor opción es usar variables de entorno. Este es el enfoque que prefiero, ya que me permite mantener las credenciales y configuraciones sensibles fuera del código. Además, al darles un nombre descriptivo, es más fácil de entender y mantener. Aunque Spring mapea las variables de entorno a las propiedades, es más claro para el desarrollador ver "DB_URL" que "SPRING_DATASOURCE_URL".

Ejemplo en application.yaml:
```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USER}
    password: ${DB_PASSWORD}
```

## Cuadro resumen de Buenas Prácticas

| Estrategia                                                          | Uso recomendado                                       | Ventajas                                                        | Desventajas                                         |
|---------------------------------------------------------------------|-------------------------------------------------------|-----------------------------------------------------------------|-----------------------------------------------------|
| `@Value`                                                            | Para valores simples.                                 | Fácil de usar.                                                  | No soporta estructuras complejas (listas, objetos). |
| Dificultad para inyectar en clases que no son manejadas por Spring. |                                                       |                                                                 |                                                     |
| `@ConfigurationProperties`                                          | Para agrupar configuraciones y estructuras complejas. | Permite agrupar configuraciones relacionadas en una sola clase. |                                                     |

Soporta estructuras complejas (listas, mapas, objetos anidados).  
Facilita la validación de propiedades. | Necesita que `@EnableConfigurationProperties(SecurityProperties.class)` esté habilitado si no se usa `@Component`.  |
| `Environment`                   | Para acceder dinámicamente a propiedades.             | Útil cuando necesitas leer valores de manera dinámica.                                                                                                      | Menos tipado, puedes cometer errores con los nombres de las propiedades.  |
| `@Profile`                      | Para gestionar configuraciones por entornos.          | Facilita la configuración por entornos.  
Evita código condicional en la aplicación.  | Si no se configura correctamente, puede causar problemas al no cargar el perfil adecuado.  |
| `@Configuration` con `@Bean`     | Para configurar valores personalizados.               | Permite modificar valores antes de inyectarlos.  
Separa lógica de configuración del código de negocio.  | Puede ser innecesario para configuraciones simples.  |
| Variables de entorno (`${VAR}`)  | Para credenciales y configuraciones externas.         | Seguridad y flexibilidad.  
Evita exponer credenciales en el código.  | Puede ser complicado depurar si las variables no están configuradas correctamente.  |
