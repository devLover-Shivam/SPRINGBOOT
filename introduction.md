# Introduction to Spring Boot

> **Learning goal:** Follow a web request from a browser to Java code and back, then see how Spring Boot organizes the pieces. Spring Boot removes much setup work, but understanding the layers underneath helps you build and troubleshoot reliable applications.

## 1. From a client to a server

A web application follows a client–server model. A **client** (such as a browser or mobile app) asks for something; a **server** receives the request, performs work, and returns a response. The client and server communicate using **HTTP** (Hypertext Transfer Protocol).

For example, a browser might request `GET /products/42`. The server looks up product 42 and responds with a status code, headers, and often a body containing HTML or JSON:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"id":42,"name":"Notebook"}
```

HTTP defines methods that describe the kind of operation:

| Method | Typical purpose |
| --- | --- |
| `GET` | Read a resource |
| `POST` | Create or submit data |
| `PUT` | Replace a resource |
| `PATCH` | Partially update a resource |
| `DELETE` | Remove a resource |

**Headers** carry extra information, such as the content type or an authorization token. **Status codes** summarize the result: `200` means success, `201` means created, `400` signals an invalid request, and `404` means the requested resource was not found. HTTPS is HTTP carried over TLS, which encrypts traffic and helps authenticate the server. In practice, public applications should use HTTPS to protect credentials and other data in transit.

## 2. Why not handle every connection ourselves?

Java includes networking APIs in packages such as `java.net`. They are useful for low-level networking, but building a web application directly on sockets means taking care of many details: accepting connections, parsing HTTP request lines and headers, reading bodies, managing concurrent clients, writing valid responses, and closing resources correctly.

That work is repetitive and easy to get wrong. Most applications should focus on business behavior—such as checking inventory—rather than implementing the HTTP protocol.

## 3. Servlets and Tomcat

The Java **Servlet API** provides a standard way to handle HTTP requests. A servlet is a Java component that receives a request and produces a response. A **servlet container** manages the network connection, maps requests to servlet code, handles lifecycle, and coordinates concurrent requests. **Tomcat** is a commonly used servlet container.

Historically, developers configured the container and servlet mappings separately. Spring MVC builds on the Servlet API: its central `DispatcherServlet` receives web requests and routes them to the appropriate controller. Spring Boot can package an embedded Tomcat server with an application, so you can run it as a regular Java program without installing a separate server.

## 4. Spring Framework: managing objects and responsibilities

A typical application needs objects that work together: controllers, services, and repositories. The Spring Framework's **IoC container** creates and manages these objects, called *beans*. **Inversion of Control (IoC)** means the application delegates object creation and wiring to the framework. **Dependency Injection (DI)** is how the framework supplies an object with the collaborators it needs.

```java
@Service
class GreetingService {
    String greeting() {
        return "Hello from Spring";
    }
}

@RestController
class GreetingController {
    private final GreetingService greetings;

    GreetingController(GreetingService greetings) {
        this.greetings = greetings; // Spring injects the service
    }

    @GetMapping("/greeting")
    String greeting() {
        return greetings.greeting();
    }
}
```

The controller does not construct its own service. Spring supplies the service through the constructor, making the relationship clear and easier to test. Constructor injection is generally a good default.

Spring is a family of modules, each focused on a common concern:

- **Spring Core** supplies the IoC container and dependency injection.
- **Spring MVC** supports web applications and REST APIs, including request mapping and data binding.
- **Spring Data** reduces repetitive data-access code and integrates with technologies such as JPA.
- **Spring Security** supports authentication, authorization, and other security controls.

These modules work together, but you choose what your application needs.

## 5. Spring Boot: practical defaults and less setup

Spring Boot builds on the Spring Framework. It provides an **opinionated** starting point: sensible defaults and conventions for common application needs, with room to customize them. Its key conveniences include:

- **Starter dependencies:** curated dependency groups, such as `spring-boot-starter-web`, that bring in compatible libraries for a common task.
- **Auto-configuration:** configuration based on the libraries on the classpath and the settings you provide. For instance, web dependencies typically lead Boot to configure Spring MVC and an embedded server.
- **Embedded server:** run the application directly, commonly with `java -jar`.
- **Externalized configuration:** adjust settings such as a server port or database URL through properties, YAML, environment variables, or command-line arguments.

A minimal REST endpoint can look like this:

```java
@SpringBootApplication
public class CatalogApplication {
    public static void main(String[] args) {
        SpringApplication.run(CatalogApplication.class, args);
    }
}

@RestController
@RequestMapping("/api/products")
class ProductController {
    @GetMapping("/{id}")
    Product find(@PathVariable long id) {
        return new Product(id, "Notebook");
    }
}

record Product(long id, String name) {}
```

With the web starter included, Boot wires the web stack and starts the embedded server. The endpoint can then return a Java object that Spring serializes as JSON. Boot's defaults are a starting point, not magic: you can inspect and override configuration when your application's needs differ.

## 6. Connecting to a database: JDBC, JPA, and Hibernate

Applications often need durable data. **JDBC** is Java's low-level standard API for connecting to relational databases and executing SQL. It gives direct control but requires routine work such as opening connections, binding parameters, reading result sets, and translating errors.

**JPA** (Jakarta Persistence API) is a specification for mapping Java objects to relational data and defining persistence operations. **Hibernate** is a popular implementation of JPA. Spring Data JPA builds on JPA to offer repositories and reduce common boilerplate.

These are related, but not interchangeable:

- JDBC is an API for working with SQL and database connections.
- JPA defines an object-persistence standard.
- Hibernate implements JPA and can generate SQL for mapped objects.
- Spring Data JPA adds convenient repository abstractions.

For example, a repository interface can express common operations without manually writing each query:

```java
interface ProductRepository extends JpaRepository<ProductEntity, Long> {
    List<ProductEntity> findByNameContainingIgnoreCase(String text);
}
```

For complex queries, performance tuning, or database-specific features, understanding SQL and the generated queries still matters.

## 7. Putting the application together

A common Spring Boot request path looks like this:

```text
Browser or mobile app
        │  HTTP/HTTPS request
        ▼
Embedded Tomcat (servlet container)
        ▼
Spring MVC DispatcherServlet → Controller
                                  ▼
                              Service
                                  ▼
                       Repository / Spring Data
                                  ▼
                         JPA provider (e.g. Hibernate)
                                  ▼
                         Database via JDBC / driver
        ▲
        └── HTTP response (status, headers, JSON or other body)
```

The **controller** handles the web boundary: it receives input and shapes the response. The **service** holds application or business rules. The **repository** handles persistence. Keeping these responsibilities distinct makes the code easier to understand and change. Spring manages the components and connects them through dependency injection; Boot supplies convenient configuration and runtime defaults.

This structure can be packaged as one **monolithic** application, which is often a straightforward way to begin. A system can also be split into **microservices**, where independently deployable services communicate over a network. The same fundamentals—HTTP, dependency injection, web handling, and data access—remain useful in either architecture.

## Key idea

Spring Boot helps you start and configure a Spring application quickly. Beneath it are familiar layers: HTTP, a servlet container, Spring MVC, managed Java components, and optional persistence and security modules. Learn what each layer does, and Boot's automation becomes easier to understand, customize, and troubleshoot.

