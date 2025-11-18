# Section 12: Microservices Security - A Detailed Walkthrough

This section implements a robust, centralized security model. Here is a step-by-step explanation of how a user is authenticated and authorized to access a protected resource.

### The Goal:
A user wants to access their account details by making a request to `/eazybank/accounts/api/fetch`. This endpoint is protected and requires the `ACCOUNTS` role.

---

### Step 1: Authentication - Getting the JWT Token from Keycloak

*   **Why?** Before the user can access any protected API, they must first prove their identity. This process is handled entirely by **Keycloak**, our identity provider.
*   **How it works:**
    1.  The user's application (e.g., a web front-end) redirects the user to the Keycloak login page.
    2.  The user enters their username and password.
    3.  Keycloak verifies these credentials against its user database.
    4.  If successful, Keycloak generates a **JSON Web Token (JWT)** and sends it back to the user's application.
*   **What is the JWT?** It's a digitally signed piece of JSON data. It contains information about the user (like their username) and, crucially, their permissions. For example:
    ```json
    {
      "sub": "user-id-123",
      "name": "John Doe",
      "realm_access": {
        "roles": [
          "ACCOUNTS",
          "USER"
        ]
      },
      "exp": 1678886400
    }
    ```
    This token is proof of identity and permissions.

---

### Step 2: The API Request - Presenting the Token to the Gateway

*   **Why?** Now that the user has the JWT, they can use it to make authenticated requests to our application.
*   **How it works:**
    1.  The user's application creates a request to the API Gateway, for example, `GET /eazybank/accounts/api/fetch`.
    2.  It adds an `Authorization` header to this request, containing the JWT.
        ```
        Authorization: Bearer <the_long_jwt_token_string>
        ```
    3.  The request is sent to the **API Gateway**.

---

### Step 3: The Gateway - Verification and Authorization

The Gateway is the central security checkpoint. When it receives the request, it performs two critical actions defined in its `SecurityConfig`.

*   **Action A: Token Validation (Authentication)**
    1.  Spring Security on the gateway sees the `Authorization: Bearer ...` header.
    2.  It knows it must validate this token. To do this, it needs Keycloak's public key.
    3.  It uses the configured URL to fetch the public keys from Keycloak.
        *File: `docker-compose/default/docker-compose.yml`*
        ```yaml
        environment:
          SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_JWK-SET-URI: "http://keycloak:8080/..."
        ```
    4.  It verifies the JWT's signature with the public key. This proves the token is authentic and wasn't tampered with. It also checks if the token has expired.
    5.  **If validation fails, the process stops, and a `401 Unauthorized` error is returned.**

*   **Action B: Role Checking (Authorization)**
    1.  If the token is valid, the gateway proceeds to check if the user has the right permissions.
    2.  It consults its security rules. The path `/eazybank/accounts/**` requires the `ACCOUNTS` role.
        *File: `gatewayserver/src/main/java/com/eazybytes/gatewayserver/config/SecurityConfig.java`*
        ```java
        .pathMatchers("/eazybank/accounts/**").hasRole("ACCOUNTS")
        ```
    3.  It uses the `KeycloakRoleConverter` to look inside the JWT's `realm_access.roles` claim and extracts the list of roles.
    4.  It checks if `ACCOUNTS` is in this list.
    5.  **If the role is not found, the process stops, and a `403 Forbidden` error is returned.**

---

### Step 4: Access Granted - Forwarding to the Microservice

*   **Why?** The user has been successfully authenticated and authorized. The gateway can now forward the request to the internal `accounts` microservice.
*   **How it works:**
    1.  The gateway forwards the original request to the `accounts` service's internal address (discovered via Eureka).
    2.  **Crucially, in this setup, the gateway does *not* pass the JWT token along.** The `accounts` service has no knowledge of the user or their roles.

---

### Step 5: The Microservice - Processing the Request

*   **Why?** The `accounts` service's only job is to perform its business logic.
*   **How it works:**
    1.  The `accounts` service receives a simple, unauthenticated request from the gateway.
    2.  It has no security configuration of its own. It operates on the principle that any request it receives has already been fully vetted by the gateway.
    3.  It processes the request, fetches the data, and returns a response to the gateway.
    4.  The gateway then relays this response back to the original user.

This flow centralizes security logic in the gateway, simplifying the downstream microservices and ensuring that no unauthenticated or unauthorized requests can reach them.
