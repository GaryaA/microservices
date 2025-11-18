# Section 8: Service Discovery & Service Registration in microservices

This section implements a service discovery and registration mechanism using Netflix Eureka, enabling dynamic communication between microservices.

### Key Components and Configuration:

1.  **Eureka Server (`eurekaserver`)**:
    *   A new `eurekaserver` microservice is introduced, acting as the central registry. It uses the `spring-cloud-starter-netflix-eureka-server` dependency.
    *   The server is configured to prevent it from registering itself as a client. This is typically done with the following properties in its configuration file:
        ```yaml
        eureka:
          client:
            registerWithEureka: false
            fetchRegistry: false
        ```

2.  **Eureka Clients (`accounts`, `loans`, `cards`)**:
    *   The existing microservices are configured as Eureka clients by adding the `spring-cloud-starter-netflix-eureka-client` dependency.
    *   Each client's `application.yml` is configured to connect to the Eureka server and register itself:
        ```yaml
        eureka:
          instance:
            preferIpAddress: true  # Use the service's IP address when registering
          client:
            fetchRegistry: true      # Fetch the registry of services from Eureka
            registerWithEureka: true # Register this service with Eureka
            serviceUrl:
              defaultZone: http://localhost:8070/eureka/ # The URL of the Eureka server
        ```

### How Services Discover Each Other:

This section also introduces **Feign**, a declarative REST client, to facilitate communication between services.

*   **Feign Client Interface**: Inside the `accounts` service, a `LoansFeignClient` interface is created:
    ```java
    @FeignClient("loans") // "loans" is the application name registered in Eureka
    public interface LoansFeignClient {
        @GetMapping(value = "/api/fetch", consumes = "application/json")
        public ResponseEntity<LoansDto> fetchLoanDetails(@RequestParam String mobileNumber);
    }
    ```
*   **Discovery Process**:
    1.  When the `accounts` service needs to call the `loans` service, it invokes a method on the `LoansFeignClient` interface.
    2.  Spring Cloud, integrated with Eureka, uses the service name `"loans"` to look up the physical address (IP and port) of an available `loans` service instance from its local copy of the Eureka registry.
    3.  Feign then constructs and sends an HTTP request to that address. This process is seamless and abstracts away the complexity of dealing with dynamic IP addresses and ports.

This setup creates a robust environment where services can locate and communicate with each other by name, without needing to know their exact network locations.
