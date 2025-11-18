# Section 6: Configurations Management in Microservices

This section explores two approaches for managing configurations, highlighting the evolution from a basic to a more robust, centralized strategy.

### Approach 1: Spring Boot Profiles (in `v1-springboot`)

This version demonstrates a simple, file-based approach using Spring Boot's native profile feature.

*   **Why it's used:** For simple applications, this allows you to define different configurations for different environments (like development, QA, and production) within the application's own source code.
*   **How it's configured:**
    *   Multiple YAML files are created in the `src/main/resources` directory:
        *   `application.yml` (contains common properties)
        *   `application-qa.yml` (contains properties specific to the QA environment)
        *   `application-prod.yml` (contains properties specific to production)
    *   The active profile is chosen at runtime (e.g., by setting the `spring.profiles.active=prod` property), and Spring Boot automatically loads the correct set of properties.
*   **Limitations:** This approach is not ideal for a microservices architecture because every time a configuration value needs to change, the entire application must be rebuilt and redeployed.

---

### Approach 2: Centralized Management with Spring Cloud Config (in `v2-spring-cloud-config`)

This version introduces a powerful, centralized configuration management system using **Spring Cloud Config**.

*   **Why it's needed:** In a microservices architecture with dozens of services, managing configuration files inside each service is unmanageable. Spring Cloud Config provides a central place to store and manage configurations for all services. This allows for:
    1.  **Centralization:** All configuration is in one place (a Git repository), not scattered across many projects.
    2.  **Dynamic Updates:** Configuration changes can be pushed to services without requiring a restart.
    3.  **Auditability:** Using a Git repository for configuration provides a full history of all changes.

#### How it's configured:

The setup consists of two main parts: a server and its clients.

1.  **The `configserver` (The Source of Truth):**
    *   **Goal:** A new microservice whose only job is to provide configuration data to other services.
    *   **Dependencies:** It includes the `spring-cloud-config-server` dependency in its `pom.xml`.
    *   **Configuration:** Its `application.yml` file points to a Git repository where the actual configuration files are stored.
        *File: `configserver/src/main/resources/application.yml`*
        ```yaml
        spring:
          cloud:
            config:
              server:
                git:
                  uri: "https://github.com/eazybytes/eazybytes-config.git" # The backend Git repo
                  default-label: main
                  clone-on-start: true
        ```

2.  **The Clients (`accounts`, `loans`, `cards`):**
    *   **Goal:** To fetch their configuration from the `configserver` at startup instead of using their local `application.yml`.
    *   **Dependencies:** Each client includes the `spring-cloud-starter-config` dependency in its `pom.xml`.
    *   **Configuration:** A single line is added to each client's `application.yml` to tell it where to find the config server.
        *File: `accounts/src/main/resources/application.yml`*
        ```yaml
        spring:
          config:
            import: "optional:configserver:http://localhost:8071/"
        ```

#### How it works:

1.  When the `accounts` microservice starts up, the `spring-cloud-starter-config` library activates.
2.  It reads the `spring.config.import` property and sees that it needs to contact the `configserver` at `http://localhost:8071`.
3.  It sends a request to the `configserver`, identifying itself by its application name (`spring.application.name`, which is "accounts") and active profile (`spring.profiles.active`).
4.  The `configserver` receives the request. It clones the backend Git repository, finds the appropriate configuration file (e.g., `accounts-prod.yml`), and sends its contents back to the `accounts` service.
5.  The `accounts` service receives these properties from the server and uses them to configure itself, treating them with higher precedence than its own local `application.yml` file.

This setup creates a robust, centralized system for managing configurations across the entire microservices landscape.
