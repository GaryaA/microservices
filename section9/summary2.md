# Section 9: Gateway, Routing & Cross-cutting Concerns in Microservices

This section implements the API Gateway pattern using **Spring Cloud Gateway**. The gateway serves as a single entry point for all client requests. Unlike previous sections where configuration was primarily in YAML files, this section defines the routing logic programmatically within the application.

### Key Components and Configuration:

1.  **Gateway Server (`gatewayserver`)**:
    *   A new `gatewayserver` microservice is added, built with the `spring-cloud-starter-gateway` dependency.
    *   It registers with Eureka to discover downstream services.

2.  **Programmatic Route Configuration**:
    *   The routing rules are defined inside the `GatewayserverApplication.java` file by creating a `RouteLocator` bean. This approach provides a type-safe, programmatic way to configure routes.
    *   The configuration is done using a `RouteLocatorBuilder`:

    ```java
    @Bean
    public RouteLocator eazyBankRouteConfig(RouteLocatorBuilder routeLocatorBuilder) {
        return routeLocatorBuilder.routes()
                .route(p -> p
                        .path("/eazybank/accounts/**") // Matches requests for the accounts service
                        .filters( f -> f.rewritePath("/eazybank/accounts/(?<segment>.*)","/${segment}")
                                        .addResponseHeader("X-Response-Time", LocalDateTime.now().toString()))
                        .uri("lb://ACCOUNTS")) // Forwards to the 'ACCOUNTS' service discovered via Eureka
                .route(p -> p
                        .path("/eazybank/loans/**")
                        // ... similar filters
                        .uri("lb://LOANS"))
                .route(p -> p
                        .path("/eazybank/cards/**")
                        // ... similar filters
                        .uri("lb://CARDS"))
                .build();
    }
    ```

### How It Works:

*   **Routing**: The `.path()` method defines a predicate that matches incoming request URLs.
*   **Load Balancing**: The `.uri("lb://...")` method uses Spring Cloud's load balancer (integrated with Eureka) to find a healthy instance of the specified service (e.g., `ACCOUNTS`).
*   **Cross-Cutting Concerns**: The `.filters()` method is used to modify requests and responses. In this section, it demonstrates two key functions:
    1.  `rewritePath`: Modifies the URL to be compatible with the downstream service's API.
    2.  `addResponseHeader`: Adds a custom header to the response, illustrating how the gateway can handle cross-cutting concerns like adding metadata or logging.

This programmatic approach provides a powerful and flexible way to manage API routing and centralized logic in a microservices architecture.
