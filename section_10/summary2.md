# Section 10: Making Microservices Resilient

This section implements resilience patterns using **Spring Cloud Resilience4j** and **Spring Cloud Gateway** to protect the system from cascading failures.

### 1. Circuit Breaker with Feign Client Fallback

To prevent failures in one service from affecting others, a fallback mechanism is added to the Feign clients in the `accounts` service.

*   **Code**: The `@FeignClient` annotation is updated with a `fallback` class.
    *File: `accounts/src/main/java/com/eazybytes/accounts/service/client/LoansFeignClient.java`*
    ```java
    @FeignClient(name="loans",fallback = LoansFallback.class)
    public interface LoansFeignClient {
        // ...
    }
    ```
*   **Explanation**: If the `loans` service is unavailable, Resilience4j opens the circuit. Instead of failing, the call is redirected to an instance of `LoansFallback.class`, which provides a default response.

    *File: `accounts/src/main/java/com/eazybytes/accounts/service/client/LoansFallback.java`*
    ```java
    @Component
    public class LoansFallback implements LoansFeignClient {

        @Override
        public ResponseEntity<LoansDto> fetchLoanDetails(String correlationId, String mobileNumber) {
            return null; // Returns a default (null) response
        }
    }
    ```

### 2. Gateway-Level Resilience Patterns

Resilience patterns are also applied directly at the API Gateway, providing a centralized point of control.

*   **Code**: The `RouteLocator` bean in `GatewayserverApplication.java` is configured with filters for circuit breaking, retries, and rate limiting.

    *File: `gatewayserver/src/main/java/com/eazybytes/gatewayserver/GatewayserverApplication.java`*

*   **Circuit Breaker (for `accounts` service)**: If the `accounts` service fails, the gateway forwards the request to a fallback URI.
    ```java
    .route(p -> p
        .path("/eazybank/accounts/**")
        .filters( f -> f.circuitBreaker(config -> config.setName("accountsCircuitBreaker")
                .setFallbackUri("forward:/contactSupport")))
        .uri("lb://ACCOUNTS"))
    ```

*   **Retry (for `loans` service)**: The gateway automatically retries failed `GET` requests to the `loans` service up to 3 times.
    ```java
    .route(p -> p
        .path("/eazybank/loans/**")
        .filters( f -> f.retry(retryConfig -> retryConfig.setRetries(3)
                .setMethods(HttpMethod.GET)
                .setBackoff(Duration.ofMillis(100),Duration.ofMillis(1000),2,true)))
        .uri("lb://LOANS"))
    ```

*   **Rate Limiter (for `cards` service)**: This protects the `cards` service from being overloaded by limiting the request rate using a Redis-based limiter.
    ```java
    .route(p -> p
        .path("/eazybank/cards/**")
        .filters( f -> f.requestRateLimiter(config -> config.setRateLimiter(redisRateLimiter())
                .setKeyResolver(userKeyResolver())))
        .uri("lb://CARDS"))
    ```
This multi-layered approach to resilience, both within and in front of the microservices, creates a much more stable and fault-tolerant system.
