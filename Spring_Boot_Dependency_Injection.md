# Spring Boot -- Dependency Injection (DI)

## 1. Introduction

Dependency Injection (DI) is a design pattern in which an object receives its dependencies externally rather than creating them itself. Spring's IoC container creates, configures and manages beans and injects their dependencies. IoC is the broader principle; DI is a technique for implementing it.

### Without DI

``` java
public class EmployeeController {
    private EmployeeService employeeService = new EmployeeService();
}
```

The controller creates its own dependency, leading to tight coupling and difficulty replacing or mocking the service.

### With DI

``` java
@RestController
public class EmployeeController {
    private final EmployeeService employeeService;
    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }
}
```

Spring supplies the service. The controller focuses on using it.

## 2. Internal working mechanism

``` mermaid
flowchart TD
    A[Spring Boot startup] --> B[Component scanning and bean registration]
    B --> C[Resolve dependencies]
    C --> D[Create EmployeeService bean]
    D --> E[Inject into EmployeeController]
    E --> F[Application ready]
```

Spring discovers stereotype-annotated classes and other bean definitions, resolves dependencies, creates beans and performs initialization. Constructor dependencies are supplied during construction; field and setter dependencies are injected afterward, before initialization callbacks.

## 3. Advantages

-   Loose coupling, particularly when using interfaces.
-   Easier unit testing with mocks.
-   Improved maintainability and separation of concerns.
-   Centralized bean lifecycle management.
-   Flexible implementation selection and reuse.

## 4. Types of Dependency Injection

``` mermaid
flowchart TD
    DI[Dependency Injection] --> C[Constructor Injection]
    DI --> S[Setter Injection]
    DI --> F[Field Injection]
```

### Common service class

``` java
package com.durgasoft.service;

import org.springframework.stereotype.Service;

@Service
public class EmployeeService {
    public String getEmployees() {
        return "Employee details fetched successfully";
    }
}
```

The following controller examples are alternatives: use only one at a time. Assume the application class is in `com.durgasoft` and Spring Web is included.

## 5. Constructor Injection

Spring supplies dependencies through the constructor. This is the recommended approach for mandatory dependencies.

``` java
package com.durgasoft.controller;

import com.durgasoft.service.EmployeeService;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/employees")
public class EmployeeController {
    private final EmployeeService employeeService;

    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }

    @GetMapping
    public String getEmployees() {
        return employeeService.getEmployees();
    }
}
```

``` mermaid
flowchart TD
    A[IoC container] --> B[Resolve EmployeeService]
    B --> C[Invoke EmployeeController constructor]
    C --> D[Assign final dependency reference]
```

### Is @Autowired mandatory?

Since Spring Framework 4.3, a single constructor does not require `@Autowired`. With multiple constructors, explicitly annotating the intended injection constructor can avoid ambiguity.

``` java
@Autowired
public EmployeeController(EmployeeService employeeService) {
    this.employeeService = employeeService;
}
```

### Why use final?

`private final EmployeeService employeeService;` ensures the reference is assigned once and cannot be reassigned. It makes required dependencies explicit and supports reliable initialization. It does not make the referenced object immutable or guarantee a non-null value when manually constructing the class.

### Multiple dependencies

``` java
public EmployeeController(EmployeeService employeeService,
                          DepartmentService departmentService) {
    this.employeeService = employeeService;
    this.departmentService = departmentService;
}
```

Declare both fields as `private final` and register both services as beans.

### Lombok alternative

``` java
@RestController
@RequiredArgsConstructor
public class EmployeeController {
    private final EmployeeService employeeService;
    private final DepartmentService departmentService;
}
```

Lombok generates a constructor for uninitialized final fields and required `@NonNull` fields. Lombok dependency and annotation processing are required.

**Advantages:** explicit dependencies, easy testing, final fields, complete initialization.

**Disadvantages:** lengthy constructors with many dependencies; constructor-based circular dependencies cannot normally be resolved. Too many dependencies may indicate excessive responsibilities.

## 6. Setter Injection

Spring invokes a setter after constructing the bean.

``` java
package com.durgasoft.controller;

import com.durgasoft.service.EmployeeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/employees")
public class EmployeeController {
    private EmployeeService employeeService;

    @Autowired
    public void setEmployeeService(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }

    @GetMapping
    public String getEmployees() {
        return employeeService.getEmployees();
    }
}
```

``` mermaid
flowchart TD
    A[Create controller] --> B[Resolve service]
    B --> C[Invoke setter]
    C --> D[Dependency assigned]
```

### Optional dependency

``` java
@Autowired(required = false)
public void setNotificationService(NotificationService notificationService) {
    this.notificationService = notificationService;
}
```

If the bean is missing, Spring skips this setter. The application must handle the missing dependency safely. Setter injection is useful for genuinely optional or replaceable dependencies.

**Advantages:** flexible configuration, easy manual mock injection.

**Disadvantages:** mutable dependency references, additional setter methods, object can exist before dependency assignment.

## 7. Field Injection

Spring injects directly into an annotated instance field using reflection.

``` java
package com.durgasoft.controller;

import com.durgasoft.service.EmployeeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/employees")
public class EmployeeController {
    @Autowired
    private EmployeeService employeeService;

    @GetMapping
    public String getEmployees() {
        return employeeService.getEmployees();
    }
}
```

``` mermaid
flowchart TD
    A[Create controller] --> B[Identify Autowired field]
    B --> C[Resolve service]
    C --> D[Inject using reflection]
```

**Advantages:** concise code, convenient for demonstrations.

**Disadvantages:** hidden dependencies, harder plain unit testing, unsuitable for final dependency fields.

Manually calling `new EmployeeController()` does not trigger Spring injection. Calling its method may therefore throw `NullPointerException`. Constructor injection avoids this problem by making the dependency explicit.

## 8. Comparison

  --------------------------------------------------------------------------------------------------------
  Feature                  Constructor                           Setter               Field
  ------------------------ ------------------------------------- -------------------- --------------------
  Injection point          Constructor                           Setter               Field

  @Autowired               Not required for single constructor   Required\*           Required\*

  Final fields             Yes                                   No                   Avoid

  Mandatory dependencies   Recommended                           Supported            Supported

  Optional dependencies    Supported                             Useful               Supported

  Plain unit testing       Easy                                  Easy                 Difficult

  Dependency visibility    High                                  Medium               Low

  Timing                   During construction                   After construction   After construction
  --------------------------------------------------------------------------------------------------------

\*For ordinary annotation-based injection using @Autowired; other annotations and configuration mechanisms exist.

## 9. Multiple implementations

Create each public type in a separate Java file under `com.durgasoft.service`. Imports and package declarations are omitted in some snippets for brevity.

``` java
public interface PaymentService {
    String makePayment();
}
```

``` java
@Service
public class UpiPaymentService implements PaymentService {
    public String makePayment() {
        return "Payment completed using UPI";
    }
}
```

``` java
@Service
public class CreditCardPaymentService implements PaymentService {
    public String makePayment() {
        return "Payment completed using Credit Card";
    }
}
```

``` java
@Service
public class DebitCardPaymentService implements PaymentService {
    public String makePayment() {
        return "Payment completed using Debit Card";
    }
}
```

``` java
@RestController
@RequestMapping("/payments")
public class PaymentController {
    private final PaymentService paymentService;

    public PaymentController(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    @GetMapping
    public String makePayment() {
        return paymentService.makePayment();
    }
}
```

Three beans match the dependency type. Without a disambiguating rule, startup fails with `NoUniqueBeanDefinitionException`, typically wrapped in `UnsatisfiedDependencyException`.

``` mermaid
flowchart TD
    U[UPI bean] --> A{Which PaymentService?}
    C[Credit card bean] --> A
    D[Debit card bean] --> A
    A --> E[Ambiguous dependency]
```

## 10. @Primary

`@Primary` gives a bean preference among eligible candidates for single-valued injection.

``` java
import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Service;

@Service
@Primary
public class UpiPaymentService implements PaymentService {
    public String makePayment() {
        return "Payment completed using UPI";
    }
}
```

Keep the other implementations and controller unchanged.

``` mermaid
flowchart LR
    U[UPI - Primary] --> P[PaymentController]
    C[Credit card] --- I[IoC container]
    D[Debit card] --- I
```

Test: `GET http://localhost:8080/payments`

Expected response: `Payment completed using UPI`

-   Use when one implementation should be preferred by default.
-   Can be used on component classes and `@Bean` methods.
-   Other beans are still registered and can be injected.
-   Multiple eligible primary beans may cause ambiguity.
-   Does not exclude other beans from collection injection.

## 11. @Qualifier

`@Qualifier` narrows eligible beans for a specific dependency.

### Constructor injection

``` java
import org.springframework.beans.factory.annotation.Qualifier;

public PaymentController(
        @Qualifier("creditCardPaymentService") PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

Expected response: `Payment completed using Credit Card`, even if UPI is primary.

``` mermaid
flowchart TD
    A[PaymentService candidates] --> B[Qualifier: creditCardPaymentService]
    B --> C[CreditCardPaymentService]
    C --> D[PaymentController]
```

### Setter injection

``` java
@Autowired
public void setPaymentService(
        @Qualifier("debitCardPaymentService") PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

### Field injection

``` java
@Autowired
@Qualifier("upiPaymentService")
private PaymentService paymentService;
```

### Bean names

  Class                      Default bean name
  -------------------------- --------------------------
  EmployeeService            employeeService
  UpiPaymentService          upiPaymentService
  CreditCardPaymentService   creditCardPaymentService
  DebitCardPaymentService    debitCardPaymentService

Default stereotype bean names generally decapitalize the class name, except when the first two characters are uppercase.

### Custom bean name

``` java
@Service("creditCard")
public class CreditCardPaymentService implements PaymentService {
    public String makePayment() {
        return "Payment completed using Credit Card";
    }
}
```

Inject using `@Qualifier("creditCard") PaymentService paymentService`.

### Semantic qualifier

``` java
@Service
@Qualifier("onlinePayment")
public class UpiPaymentService implements PaymentService {
    public String makePayment() {
        return "Payment completed using UPI";
    }
}
```

Inject using `@Qualifier("onlinePayment") PaymentService paymentService`.

## 12. @Primary vs @Qualifier

  Feature      @Primary                    @Qualifier
  ------------ --------------------------- ------------------------------------
  Purpose      Preferred bean              Narrow eligible candidates
  Applied at   Bean definition             Injection point or bean definition
  Scope        General preference          Specific requirement
  Selection    Among eligible candidates   Filters candidates

A unique qualifier selects its matching bean even when another bean is primary. If multiple beans match a qualifier, a primary bean among those candidates can resolve the ambiguity.

## 13. Injecting all implementations

### List injection

``` java
@Service
public class PaymentProcessor {
    private final List<PaymentService> paymentServices;
    public PaymentProcessor(List<PaymentService> paymentServices) {
        this.paymentServices = paymentServices;
    }
    public void processPayments() {
        for (PaymentService service : paymentServices) {
            System.out.println(service.makePayment());
        }
    }
}
```

### Map injection

``` java
@Service
public class PaymentProcessor {
    private final Map<String, PaymentService> paymentServices;
    public PaymentProcessor(Map<String, PaymentService> paymentServices) {
        this.paymentServices = paymentServices;
    }
    public String processPayment(String paymentType) {
        PaymentService service = paymentServices.get(paymentType);
        if (service == null) {
            throw new IllegalArgumentException("Unsupported payment type");
        }
        return service.makePayment();
    }
}
```

Import `java.util.List` or `java.util.Map` as appropriate. The map keys are bean names and values are bean instances. This is useful for runtime implementation selection and the Strategy pattern. Use the two processor examples separately.

## 14. Unit testing with Mockito

``` java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.*;

class EmployeeControllerTest {
    @Test
    void testGetEmployees() {
        EmployeeService service = mock(EmployeeService.class);
        when(service.getEmployees()).thenReturn("Employee details");
        EmployeeController controller = new EmployeeController(service);
        assertEquals("Employee details", controller.getEmployees());
        verify(service).getEmployees();
    }
}
```

This test uses the constructor injection version and does not require starting Spring. Include Spring Boot's test starter and import the application classes.

## 15. Real-world layered architecture

``` mermaid
flowchart TD
    A[Client / Postman] -->|HTTP| B[EmployeeController]
    B -->|Injected service| C[EmployeeService]
    C -->|Injected repository| D[EmployeeRepository]
    D -->|SQL| E[(PostgreSQL)]
```

``` java
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
}
```

``` java
@Service
public class EmployeeService {
    private final EmployeeRepository employeeRepository;
    public EmployeeService(EmployeeRepository employeeRepository) {
        this.employeeRepository = employeeRepository;
    }
    public List<Employee> getEmployees() {
        return employeeRepository.findAll();
    }
}
```

``` java
@RestController
@RequestMapping("/employees")
public class EmployeeController {
    private final EmployeeService employeeService;
    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }
    @GetMapping
    public List<Employee> getEmployees() {
        return employeeService.getEmployees();
    }
}
```

Spring Data JPA creates the repository implementation. This separate example assumes an Employee entity, Spring Data JPA, PostgreSQL driver and datasource configuration.

## 16. Practical testing scenarios

  -----------------------------------------------------------------------------------
  Scenario                                        Expected result
  ----------------------------------------------- -----------------------------------
  Single implementation                           Successful injection

  Three implementations without disambiguation    NoUniqueBeanDefinitionException

  UPI primary                                     UPI selected

  Credit card qualifier                           Credit card selected

  Debit card qualifier                            Debit card selected

  UPI primary plus unique credit card qualifier   Credit card selected

  Two eligible primary beans                      Ambiguity

  Invalid qualifier                               No matching bean

  List or Map injection                           All matching beans

  Single constructor without Autowired            Successful injection

  Ordinary setter without Autowired               No automatic setter injection

  No registered implementation                    No matching bean
  -----------------------------------------------------------------------------------

## 17. Best practices

-   Prefer constructor injection for mandatory dependencies.
-   Use private final fields for constructor-injected dependencies.
-   Use setters for genuinely optional or replaceable dependencies.
-   Avoid field injection in production application classes.
-   Use interfaces when abstraction or multiple implementations are useful.
-   Use Primary for preference and Qualifier for specific requirements.
-   Avoid circular dependencies and excessive constructor parameters.
-   Do not manually instantiate dependencies that should be Spring-managed.

## 18. Interview questions

1.  **What is DI?** Supplying dependencies externally instead of creating them internally.
2.  **IoC vs DI?** IoC is a broader principle; DI is one implementation technique.
3.  **Types of DI?** Constructor, setter and field injection.
4.  **Recommended type?** Constructor injection for mandatory dependencies.
5.  **Is Autowired mandatory on a single constructor?** No.
6.  **Default DI type?** Spring Boot does not enforce a universal default injection type.
7.  **Why final?** Prevents reassignment of the dependency reference.
8.  **Multiple matching beans?** Spring applies selection rules; unresolved ambiguity causes NoUniqueBeanDefinitionException.
9.  **Primary vs Qualifier?** Preference versus candidate filtering.
10. **Can both be used?** Yes; a unique qualifier can select a non-primary bean.
11. **Can all implementations be injected?** Yes, using collections or maps.
12. **Does DI require Spring?** No, DI can also be performed manually or through other frameworks.

## 19. Summary

  Concept                 Remember
  ----------------------- -----------------------------------------
  IoC container           Creates and manages beans
  Constructor injection   Supply dependencies during construction
  Setter injection        Supply dependencies through a method
  Field injection         Assign dependencies directly to fields
  @Primary                Prefer this eligible bean
  @Qualifier              Narrow candidates to matching beans
  final                   Prevent reference reassignment

## References

-   [Spring Framework -- Dependency Injection](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
-   [Spring Framework -- Annotation-based container configuration](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config.html)

**Note:** These are instructional snippets, not a complete tested project. Mermaid diagrams require a compatible Markdown viewer. The interactive simulator from the chat is represented by the practical testing scenarios in this static file.
