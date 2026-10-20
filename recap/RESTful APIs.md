---
title: RESTful APIs
---

# 1. Index

<!--toc:start-->

- [1. Index](#1-index)
- [2. RESTful APIs](#2-restful-apis)
  - [2.1. HTTP Methods](#21-http-methods)
  - [2.2. REST Request](#22-rest-request)
  - [2.3. REST Response](#23-rest-response)
    - [2.3.1. Pagination](#231-pagination)
  - [2.4. Versioning](#24-versioning)
- [3. Spring Boot](#3-spring-boot)
  - [3.1. Controller](#31-controller)
  - [3.2. Model](#32-model)
  - [3.3. Repository](#33-repository)
  - [3.4. Service](#34-service)
  - [3.5. Configuration](#35-configuration)
- [4. OpenAPI and Swagger](#4-openapi-and-swagger)
  - [4.1. SpringDoc OpenAPI](#41-springdoc-openapi)
  - [4.2. Swagger](#42-swagger)
  <!--toc:end-->

# 2. RESTful APIs

REST stands for **Representational State Transfer**. It is an architectural
style for designing networked applications. RESTful APIs are web services that
adhere to the principles of REST, allowing clients to interact with resources
using standard HTTP methods (`GET`, `POST`, `PUT`, `DELETE`) to operate on
resources.

REST is not a specification or protocol, but rather a loose set of rules
that has been the _de facto_ standard for building web APIs since the early 2000s.

An API that follows REST principles is called a RESTful API.
The key characteristics of RESTful APIs include:

- **Statelessness**: Each request from a client to the server must contain all
  the information needed to understand and process the request. The server does
  not store any client context between requests.
- **Client-Server Architecture**: The client and server are separate entities that
  communicate over a network. The client is responsible for the user interface
  and user experience, while the server handles data storage and business logic.
- **Uniform Interface**: RESTful APIs provide a consistent and standardized way
  to interact with resources. This includes using standard HTTP methods, resource
  URIs, and media types for representation.

Some URI example are:

```url
https://api.example.com/api/v3/products
https://api.example.com/api/v3/users
```

Resources should be grouped by nouns and not verbs.

We should avoid using `getAllProducts` or `getProducts` as the endpoint name,
as it is a verb that repeats the HTTP method `GET`.

## 2.1. HTTP Methods

The fundamental methods of RESTful APIs are:

- `POST` (**Create**): is used to create a new resource on the server. The
  client sends data in the request body, and the server responds with the
  created resource's URI or representation. It is the only **non idempotent**
  operation.
- `GET` (**Read**): is used to retrieve a resource or a collection of
  resources from the server. The client sends a request to the server, and the
  server responds with the requested data.
- `PUT` (**Update**): is used to update an existing resource on the server.
- `DELETE` (**Delete**): is used to remove a resource from the server.

Some additional methods are:

- `PATCH` (**Partial Update**): is used to update a resource partially. The client
  sends only the fields that need to be updated, and the server applies the
  changes to the existing resource.
- `OPTIONS` (**Describe**): is used to describe the communication options for a
  resource. The client sends a request to the server, and the server responds
  with the allowed HTTP methods and other relevant information.

We define an **idempotent** operation as one that, when called multiple times,
always produces the same result. In other words, making the same request
multiple times has the same effect as making it once.

## 2.2. REST Request

The request line contains an **HTTP verb** from the ones we saw before
followed by the **URI of the resource** we'd like to access.

A request may include an optional **HTTP request body** containing a custom
payload of data, usually encoded in `JSON` format.
This type of request is typically used for `POST` and `PUT` methods, and they
are flagged as `application/json` in the `Content-Type` header.

## 2.3. REST Response

After receiving a request to manipulate data, the server process it, and usually
produces a response.

The response is an HTTP message containing:

- **Status Code**: A three-digit code indicating the result of the request.
  They are classified into five categories:
  - **Informational** (`1xx`): The request was received, and the server is
    continuing to process it.
  - **Successful** (`2xx`): The request was successfully received, understood,
    and accepted.
  - **Redirection** (`3xx`): Further action is needed to complete the request.
  - **Client Error** (`4xx`): The request contains bad syntax or cannot be fulfilled.
  - **Server Error** (`5xx`): The server failed to fulfill a valid request.
- **Body**: The response body may contain the requested data, an error message,
  or other relevant information. Is typically in `JSON` format.

### 2.3.1. Pagination

Sometimes the server response returns a **huge amount of data**.
Instead of trying to show all the results together, we can use **pagination** to
split the results into smaller chunks.

A common approach is to include in the request URI a query specifying `limit` and
`offset` (defaulting to `0`), which allows the client to request a specific
subset of the data.

```url
GET /evets?limit=10&offset=20
```

## 2.4. Versioning

**Versioning** is a slim and clever way to manage changes in an API over time.

It allows developers to introduce new features, fix bugs, or make improvements
without breaking existing clients that rely on the previous version of the API,
providing **backward compatibility**.

There are many way to version an API, with the most straightforward being to
include the version number in the URI path, like:

```url
/api/v1/events
/api/v2/events
/api/v3/events
```

# 3. Spring Boot

Spring Boot is a popular framework for building RESTful APIs in Java.

It reduced the need for boilerplate code and configuration, allowing developers
to focus on writing business logic.

It provides **built-in support for HTTP methods**, **auto-configuration** and
embedded servers like `Tomcat`.

To set it up we can create a **Spring Boot Project** using the **Spring Initializr**
at [https://start.spring.io/](https://start.spring.io/).

Using it allows us to build REST APIs quickly and efficiently.

A standard project structure for a Spring Boot application is:

```example
your-project/
└── src
    └── main
        ├── java
        │    │
        │    └── com/example/yourproject
        │        │
        │        ├── controller
        │        │
        │        ├── model
        │        │
        │        ├── repository
        │        │
        │        ├── service
        │        │
        │        ├── config
        │        │
        │        └── YourProjectApplication.java
        └── resources
            ├── application.properties (or .yml)
            ├── static (optional)
            ├── templates (optional)
            └── schema.sql and data.sql (optional)
```

The flow is the following:

1. **Controller** maps the request to handler methods. It does not contain
   business logic, but it is responsible for handling HTTP requests and responses.
2. **Model**: entities that represent the data structure of the application.
   They are typically annotated with `@Entity` and mapped to database tables.
3. **Service** contains the business logic of the application. It is
   responsible for processing data, performing calculations, and coordinating interactions
   between the controller and repository.
4. **Repository**: interfaces that define the data access layer. They are
   typically annotated with `@Repository` and extend Spring Data JPA
   interfaces like `JpaRepository` or `CrudRepository`, providing methods for
   CRUD operations on the database.
5. **Config**: contains configuration classes that define beans, security
   settings, and other application
6. **Resources**: contains static files, templates, and configuration files
   like `application.properties` or `application.yml`.

The **main class** of our `Spring Boot` application is annotated with
`@SpringBootApplication`:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class EventDescriptionEnhancerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EventDescriptionEnhancerApplication.class, args);
    }
}
```

This starts the application and initializes the Spring context, starting the
`Tomcat` server at the `8080` port.

## 3.1. Controller

The `controller/` package contains **the REST controllers** that handle incoming
HTTP requests and return responses.

The class is annotated with:

- `@RestController`: indicates the class handles REST requests and returns
  data as `JSON` or `XML`
- `@RequestMapping`: specifies the base URL path for all the endpoints in the controller.

Each method in the controller is then annotated with the appropriate HTTP method
annotation (`@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`) and
the specific endpoint path.

The methods response is a `ResponseEntity<>` object, which allows us to customize
the HTTP status code and the response body.

The arguments of the methods are annotated with:

- `@PathVariable`: binds a method parameter to a URI template variable. usually
  used in `GET`, `PUT`, and `DELETE` requests to identify a specific resource.
- `@RequestParam`: binds a method parameter to a query parameter in the request
  URL. It is typically used for filtering, sorting, or pagination.

An example of a controller class is:

```java
@RestController
@RequestMapping("/api/v1/events")
@RequiredArgsConstructor
public class EventController {
  private final EventService eventService;

  // This maps to `/api/v1/events?offset=0&limit=10`
  @GetMapping
  public List<Event> getAllEvents(@RequestParam(defaultValue = "0") int offset,
                                  @RequestParam(defaultValue = "10") int limit){
    return eventService.getAllEvents(offset, limit);
  }

  @GetMapping("/{id}")
  public ResponseEntity<Event> getEventById(@PathVariable Long id) {
    return eventService.getEventById(id);
  }

  @PostMapping
  public ResponseEntity<Event> createEvent(@RequestBody Event event) {
    return eventService.createEvent(event);     // 201 Created
  }

  @PutMapping("/{id}")
  public ResponseEntity<Event> updateEvent(@PathVariable Long id,
                                           @RequestBody Event updatedEvent) {
    Event event = eventService.updateEvent(id, updatedEvent);
    return ResponseEntity.ok(event);            // 200 OK
  }

  @DeleteMapping("/{id}")
  public ResponseEntity<Void> deleteEvent(@PathVariable Long id) {
    eventService.deleteEvent(id);
    return ResponseEntity.noContent().build();    // 204 No Content
  }
}
```

## 3.2. Model

The `model/` package holds the **domain models** or **entity classes** representing
the business objects.

With a database, this is where we define **JPA entities**.

```java
// Lombok annotation to generate getters, setters, and other utility methods
import lombok.Data;
// JPA annotations for defining entity classes and mapping them to database tables
import jakarta.persistence.*;
import java.time.LocalDateTime;

@Data
@Entity
public class Event {
    // Auto-incremented primary key
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT")
    private String description;

    private LocalDate date;
}
```

## 3.3. Repository

The `repository/` package holds the **repository interfaces**.
They are typically based on **Spring Data JPA** and provide data access logic (e.g.,
CRUD operations) for the entity classes.

```java
public interface EventRepository extends JpaRepository<Event, Long> {
    /**
    *
    * JPA provides built-in methods for CRUD operations, such as getters,
    * savers, and deleters, so we don't need to implement them manually.
    *
    * Further custom query methods can be defined here if needed
    *
    */
}
```

## 3.4. Service

The `service/` package contains the **service classes** that encapsulate the business
logic that mediates between controllers and repositories.

Services classes are annotated with `@Service`:

```java
@Service
@RequiredArgsConstructor
puvlic class EventService {
    private final EventRepository eventRepository;

    public static final String API_URL =
      "https://api.openai.com/v1/chat/completions";

    // Never hard-code sensitive information like API keys in your source code.
    // Save them as environment variables or in a secure configuration file.
    //  and retrieve them at runtime using `System.getenv()` or a configuration
    //  library.
    public static fial String API_KEY = System.getenv("OPENAI_API_KEY");

    public List<Event> getAllEvents(int offset, int limit) {
        Pageable pageable = PageRequest.of(offset / limit, limit);
        return eventRepository.findAll(pageable).getContent();
    }

    public ResponseEntity<Event> getEventById(Long id) {
        Optional<Event> event = eventRepository.findById(id);
        return event.map(ResponseEntity::ok)
                    .orElseGet(() -> ResponseEntity.notFound().build());
    }

    public ResponseEntity<Event> createEvent(Event event) {
        Event savedEvent = eventRepository.save(event);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedEvent);
    }

    public Event updateEvent(Long id, Event updatedEvent) {
        return eventRepository.findById(id)
                .map(event -> {
                    event.setTitle(updatedEvent.getTitle());
                    event.setDescription(updatedEvent.getDescription());
                    event.setDate(updatedEvent.getDate());
                    return eventRepository.save(event);
                })
                .orElseThrow(() -> new ResourceNotFoundException(
                      "Event not found with id " + id));
    }

    public void deleteEvent(Long id) {
        if (!eventRepository.existsById(id)) {
            throw new ResourceNotFoundException("Event not found with id " + id);
        }
        eventRepository.deleteById(id);
    }
}
```

## 3.5. Configuration

The `config/` package is where we put **configuration classes**.

Here we can define _beans_, _swagger configuration_, _security settings_, and other
application-specific configurations, using the `@Configuration` annotation.

```java
@Configuration
public class SwaggerConfig {
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("Event API")
                        .version("1.0")
                        .description("API for managing events"));
    }
}
```

# 4. OpenAPI and Swagger

The **OpenAPI Specification** (formerly known as Swagger)
is a standard for defining RESTful APIs in a machine-readable format,
typically using `JSON` or `YAML`.
It allows developers to generate **automatically** API documentation, client
SDKs and server stubs.

## 4.1. SpringDoc OpenAPI

Its a library that integrates the OpenAPI specification into Spring Boot applications,
allowing developers to automatically generate OpenAPI compliant documentation for
their RESTful APIs.

The documentation is generated based on the annotations in our code, such as
`@RestController`, `@RequestMapping`, and `@Operation`.

It also provides an **interactive Swagger UI** for testing the API endpoints directly
from the browser.

To use SpringDoc OpenAPI, we need to add the following dependency to our `pom.xml`:

```xml
<!-- Swagger for API documentation -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.8.9</version>
    <!-- The version must match the Spring Boot version -->
    <!-- 2.8.x works with Spring Boot 3.5 -->
</dependency>
```

## 4.2. Swagger

Is a set of **open-source tools** built around the OpenAPI Specification
that help design, build, document and consume RESTful APIs.

It generates **interactive documentation** and makes it easy for developers
to explore, understand and test the API endpoints.

We are able to see a [Swagger UI Demo](https://petstore.swagger.io/) for the
OpenAPI specification.

We can notice that is groups the endpoints by **tags**, using different colors
for each HTTP method, and allows us to test the endpoints directly from the browser.

When using Swagger in Spring Boot, we can access the Swagger UI by default
at the following URL:
[http://localhost:8080/swagger-ui/index.html](http://localhost:8080/swagger-ui/index.html)
