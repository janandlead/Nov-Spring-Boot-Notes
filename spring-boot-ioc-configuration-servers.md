# Spring Boot -- IoC Container, Configuration and Embedded Servers

Examples use `employee-management-service`, package `com.durgasoft`, Java 17 and Spring Boot 3.x/4.x.

## 1. Spring IoC Container

Inversion of Control (IoC) transfers responsibility for object creation and dependency management to Spring. Objects managed by Spring are called beans. Dependency Injection (DI) supplies dependencies to these objects.

Responsibilities include bean creation, configuration, dependency injection, scopes and lifecycle management.

``` mermaid
flowchart TD
 A[Spring IoC Container] --> B[BeanFactory]
 A --> C[ApplicationContext]
 B --> D[Basic bean management]
 C --> E[Bean management, events, resources and environment]
```

## 2. BeanFactory

`org.springframework.beans.factory.BeanFactory` is the fundamental container interface. It provides bean lookup, dependency injection and scope management. A basic bean factory typically creates beans on demand.

### Example

``` java
public class Employee {
    public void display() {
        System.out.println("Welcome Employee");
    }
}
```

``` java
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.beans.factory.support.RootBeanDefinition;

public class Main {
    public static void main(String[] args) {
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        factory.registerBeanDefinition("employee", new RootBeanDefinition(Employee.class));
        BeanFactory beanFactory = factory;
        Employee employee = beanFactory.getBean("employee", Employee.class);
        employee.display();
        factory.destroySingletons();
    }
}
```

Expected output: `Welcome Employee`. The bean is created on the first lookup in this example.

## 3. ApplicationContext

`org.springframework.context.ApplicationContext` extends the capabilities of BeanFactory. It is the standard container for Spring Boot applications.

Features: bean management, eager initialization of non-lazy singleton beans by default, application events, internationalization, resource loading, environment management and automatic detection of bean post-processors.

### Example

``` java
package com.durgasoft;
import org.springframework.stereotype.Component;

@Component
public class Employee {
    public void display() {
        System.out.println("Welcome Employee");
    }
}
```

``` java
package com.durgasoft;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;

@SpringBootApplication
public class EmployeeManagementServiceApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(
            EmployeeManagementServiceApplication.class, args);
        Employee employee = context.getBean(Employee.class);
        employee.display();
    }
}
```

### Execution flow (simplified)

``` mermaid
flowchart TD
 A[main method] --> B[SpringApplication.run]
 B --> C[Create ApplicationContext]
 C --> D[Component scanning and auto-configuration]
 D --> E[Create beans and inject dependencies]
 E --> F[ApplicationContext ready]
 F --> G[getBean]
```

### BeanFactory vs ApplicationContext

  -----------------------------------------------------------------------------------------------------
  Feature                       BeanFactory                   ApplicationContext
  ----------------------------- ----------------------------- -----------------------------------------
  Container                     Basic                         Advanced

  Initialization                Typically on demand           Non-lazy singletons eagerly initialized

  Dependency injection          Yes                           Yes

  Bean lifecycle                Yes                           Yes

  Events                        Not built in                  Yes

  Internationalization          Not built in                  Yes

  Environment management        Not built in                  Yes

  Post-processor registration   Manual                        Automatic detection

  Resource loading              Not part of basic interface   Yes

  Usage                         Low-level infrastructure      Standard Spring Boot applications
  -----------------------------------------------------------------------------------------------------

ApplicationContext also supports lazy initialization using `@Lazy` or `spring.main.lazy-initialization=true`.

### Important methods

  Method                       Purpose
  ---------------------------- -----------------------------------------
  `getBean()`                  Retrieve bean
  `containsBean()`             Check bean existence
  `getBeanDefinitionNames()`   Registered definition names
  `getBeanDefinitionCount()`   Number of definitions
  `getBeansOfType()`           Beans matching a type
  `isSingleton()`              Check singleton scope
  `isPrototype()`              Check prototype scope
  `getEnvironment()`           Access configuration environment
  `getApplicationName()`       Context application name (may be empty)
  `getStartupDate()`           Context startup timestamp
  `publishEvent()`             Publish application event
  `getResource()`              Load resource

``` java
System.out.println(context.getBeanDefinitionCount());
System.out.println(context.containsBean("employee"));
System.out.println(context.isSingleton("employee"));
for (String name : context.getBeanDefinitionNames()) {
    System.out.println(name);
}
```

Prefer ApplicationContext for normal Spring Boot development.

## 4. application.properties vs application.yaml

External configuration allows changes to application behavior without modifying Java code. Both formats support server settings, application names, database settings, logging, profiles and custom properties.

Default project location:

``` text
employee-management-service/
  src/main/java/com/durgasoft/
  src/main/resources/application.properties
  pom.xml
```

Alternatively use `application.yaml` or `application.yml`.

### Properties format

``` properties
spring.application.name=employee-management-service
server.port=9090
spring.datasource.url=jdbc:postgresql://localhost:5432/employeedb
spring.datasource.username=postgres
spring.datasource.password=${DB_PASSWORD}
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
logging.level.root=INFO
logging.level.com.durgasoft=DEBUG
```

### YAML format

``` yaml
spring:
  application:
    name: employee-management-service
  datasource:
    url: jdbc:postgresql://localhost:5432/employeedb
    username: postgres
    password: ${DB_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
server:
  port: 9090
logging:
  level:
    root: INFO
    com.durgasoft: DEBUG
```

Database settings are illustrative and require the appropriate dependencies and database. They are not required for the REST demo. Use schema migrations rather than `ddl-auto=update` for controlled production deployments.

  Feature              Properties                   YAML
  -------------------- ---------------------------- --------------------
  Extension            .properties                  .yaml / .yml
  Structure            Flat                         Hierarchical
  Syntax               key=value                    key: value
  Indentation          Not structural               Required
  Repeated prefixes    Common                       Reduced
  Lists                Indexed or comma-separated   Native list syntax
  Multiple documents   Supported by Spring Boot     Supported
  Comments             \# or !                      \#

### Lists

``` properties
app.languages[0]=Java
app.languages[1]=Python
app.languages[2]=JavaScript
```

``` yaml
app:
  languages:
    - Java
    - Python
    - JavaScript
```

Use spaces, not tabs, for YAML indentation. Two spaces is a common convention. Incorrect indentation can change the meaning of a property.

If both formats exist in the same location, properties takes precedence for conflicting keys. Prefer one format to avoid confusion.

## 5. Changing the server port

The default embedded web server port is 8080. Changing ports helps avoid conflicts and run multiple services locally.

### Properties

``` properties
server.port=9090
```

### YAML

``` yaml
server:
  port: 9090
```

### Command line

``` bash
java -jar employee-management-service.jar --server.port=9091
mvn spring-boot:run -Dspring-boot.run.arguments="--server.port=9091"
```

### Environment variable

PowerShell:

``` powershell
$env:SERVER_PORT="9092"
java -jar employee-management-service.jar
```

Linux/macOS:

``` bash
export SERVER_PORT=9092
java -jar employee-management-service.jar
```

### Random port

``` properties
server.port=0
```

The operating system assigns an available port. Useful for integration tests.

### REST controller demo

``` java
package com.durgasoft.controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class EmployeeController {
    @GetMapping("/welcome")
    public String welcome() {
        return "Welcome to Spring Boot";
    }
}
```

Restart the application and test `GET http://localhost:9090/welcome`.

Expected response: `200 OK` with body `Welcome to Spring Boot`.

``` mermaid
flowchart LR
 A[Client] --> B[Embedded server port 9090]
 B --> C[DispatcherServlet]
 C --> D[EmployeeController]
```

### Multiple microservices

  Service          Port
  -------------- ------
  Employee         8081
  Department       8082
  Order            8083
  Payment          8084
  Notification     8085

### Port already in use

If the application fails because port 8080 is already occupied, stop the conflicting process or set `server.port=8081` and restart.

## 6. Embedded web servers

An embedded server is packaged with the application and starts along with it. Unlike traditional external WAR deployment, a standalone Spring Boot executable JAR does not require a separately installed servlet container.

Benefits: simplified deployment, independent configuration, easy containerization and suitability for microservices.

  Server     Spring Boot 3.x      Spring Boot 4.x
  ---------- -------------------- ------------------------------
  Tomcat     Default MVC server   Default MVC server
  Jetty      Supported            Supported
  Undertow   Supported            Not supported out of the box

Spring WebFlux uses Reactor Netty by default, a different web stack.

## 7. Exclude Tomcat and add Jetty

Jetty is an open-source Java HTTP server and Servlet container. Tomcat is normally included transitively by the MVC web starter.

### Spring Boot 3.x -- Maven

``` xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jetty</artifactId>
    </dependency>
</dependencies>
```

### Spring Boot 4.x -- Maven

``` xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webmvc</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jetty</artifactId>
    </dependency>
</dependencies>
```

Spring Boot dependency management supplies compatible versions.

### Gradle (Spring Boot 4.x)

``` groovy
dependencies {
    implementation('org.springframework.boot:spring-boot-starter-webmvc') {
        exclude group: 'org.springframework.boot', module: 'spring-boot-starter-tomcat'
    }
    implementation 'org.springframework.boot:spring-boot-starter-jetty'
}
```

For Boot 3.x use `spring-boot-starter-web` instead of `spring-boot-starter-webmvc`.

### Architecture before and after

``` mermaid
flowchart TD
 subgraph Before
 A[Client] --> B[Tomcat] --> C[DispatcherServlet] --> D[REST Controller]
 end
 subgraph After
 E[Client] --> F[Jetty] --> G[DispatcherServlet] --> H[REST Controller]
 end
```

Spring Boot auto-configuration detects the available server classes and configures the appropriate server factory. With Jetty available and Tomcat removed, it configures `JettyServletWebServerFactory`. No controller changes are required.

### Verification

1.  Reload Maven dependencies in the IDE.
2.  Run `mvn dependency:tree` and verify Jetty is present and embedded Tomcat is absent. If another dependency brings Tomcat back, exclude it there too.
3.  Run `mvn spring-boot:run`.
4.  Check startup logs for Jetty and the configured port. Exact log wording varies by version.
5.  Test `GET http://localhost:9090/welcome` with `server.port=9090` configured.

Expected response: `Welcome to Spring Boot`.

## 8. Tomcat vs Jetty

  Feature                     Tomcat                       Jetty
  --------------------------- ---------------------------- --------------------
  Organization                Apache Software Foundation   Eclipse Foundation
  Default Spring MVC server   Yes                          No
  Embedded support            Yes                          Yes
  Servlet API                 Yes                          Yes
  WebSocket support           Yes                          Yes
  Asynchronous requests       Yes                          Yes
  Performance                 Workload dependent           Workload dependent

Tomcat is a sensible default. Choose Jetty for organizational standards, specific features or demonstrated workload benefits. Neither is universally faster.

## 9. Practical assignments

1.  Create an Employee component and retrieve it using ApplicationContext.
2.  Print bean names and bean definition count.
3.  Convert properties to YAML.
4.  Change the port to 9090 and test the welcome endpoint.
5.  Run two applications on different ports.
6.  Configure a random port.
7.  Replace Tomcat with Jetty and verify the dependency tree and startup logs.
8.  Test the same REST endpoint after switching servers.

## 10. Interview questions

-   **BeanFactory vs ApplicationContext?** Basic bean management vs additional enterprise container services.
-   **Which container does Spring Boot use?** ApplicationContext.
-   **Return type of SpringApplication.run()?** ConfigurableApplicationContext.
-   **Default port?** 8080.
-   **How to change port?** Configure server.port, SERVER_PORT or command-line arguments.
-   **What does server.port=0 do?** Requests an available port from the OS.
-   **Properties vs YAML?** Flat vs hierarchical configuration.
-   **Default MVC server?** Tomcat.
-   **How to replace Tomcat?** Exclude its starter and add the Jetty starter.
-   **Controller changes required?** No.
-   **Can Spring Boot run without a web server?** Yes, set `spring.main.web-application-type=none`.
-   **Can two independent servers use the same IP and port?** Normally no.

## 11. Quick revision

  ---------------------------------------------------------------------------------------------
  Topic                               Key takeaway
  ----------------------------------- ---------------------------------------------------------
  IoC                                 Spring manages objects and dependencies

  BeanFactory                         Basic container interface

  ApplicationContext                  Standard advanced container

  Properties                          Flat key-value format

  YAML                                Hierarchical format

  server.port                         Configures HTTP server port

  Tomcat                              Default MVC server

  Jetty                               Alternative MVC server

  Exclusions                          Remove transitive dependencies

  Auto-configuration                  Configures components based on classpath and conditions
  ---------------------------------------------------------------------------------------------

**Note:** Examples are instructional and have not been executed. Mermaid diagrams require a compatible Markdown viewer.
