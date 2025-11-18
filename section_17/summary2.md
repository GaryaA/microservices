# Section 17: Server-side Service Discovery with Kubernetes

This section refactors the application to use Kubernetes's native, **server-side service discovery**. This is a fundamental pattern for communication between services running inside a Kubernetes cluster.

### Why it's needed:

In a dynamic environment like Kubernetes, pods can be created, destroyed, and rescheduled at any time, meaning their IP addresses are constantly changing. It is not feasible to hardcode IP addresses. Kubernetes solves this with a built-in service discovery mechanism that provides a stable endpoint for a set of pods.

### How Kubernetes Service Discovery Works (The Core Mechanism):

1.  **Labels and Selectors**:
    *   Every pod created by a `Deployment` is given a **label**, like `app: accounts`.
    *   A Kubernetes **`Service`** object is created with a **selector** that matches this label, for example, `selector: { app: accounts }`.
    *   This `Service` continuously monitors the cluster for pods with that label.

2.  **A Stable Endpoint**:
    *   When the `Service` is created, Kubernetes gives it:
        *   A stable, internal IP address (a "Cluster IP").
        *   A stable **DNS name** that matches the service name (e.g., a `Service` named `loans` can be reached at the DNS address `loans`).
    *   This `Service` acts as a virtual load balancer. It maintains a list of the current, healthy IP addresses of the pods that match its selector.

3.  **DNS Resolution**:
    *   When one pod (e.g., `accounts`) wants to communicate with another (e.g., `loans`), it simply makes a request to the stable DNS name: `http://loans:8090`.
    *   Kubernetes's internal DNS server (CoreDNS) resolves this name to the stable Cluster IP of the `loans` `Service`.
    *   The `Service` then automatically forwards the request to one of the healthy `loans` pods.

This entire process is managed by Kubernetes on the server-side. The client pod only needs to know the DNS name of the service it wants to call.

### How Spring Cloud Integrates with Kubernetes Discovery:

This section adapts the Spring Boot application to leverage this native capability.

1.  **Kubernetes Discovery Client Dependency**:
    *   **Code Change**: The `spring-cloud-starter-kubernetes-discoveryclient` dependency is added to the `pom.xml` of each microservice.
        *File: `accounts/pom.xml`*
        ```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-kubernetes-discoveryclient</artifactId>
        </dependency>
        ```

2.  **Enabling the Discovery Client**:
    *   **Configuration Change**: The `application.yml` is updated to enable and configure the Kubernetes discovery client.
        *File: `accounts/src/main/resources/application.yml`*
        ```yaml
        spring:
          cloud:
            kubernetes:
              discovery:
                all-namespaces: true # Look for services in all Kubernetes namespaces
        ```

### How it all works together:

The application code using **Feign remains unchanged**. The Spring Cloud abstraction layer handles the new discovery mechanism seamlessly.

1.  The `accounts` service needs to call the `loans` service via its Feign client, which is defined with `@FeignClient("loans")`.
2.  The `DiscoveryClient` implementation provided by Spring Cloud Kubernetes is now active.
3.  Instead of talking to Eureka, this `DiscoveryClient` makes a request to the **Kubernetes API Server**.
4.  It asks the API server for the details of the `Service` named `loans`.
5.  The API server responds with the endpoints (the IP addresses of the running `loans` pods) that the `loans` `Service` is currently managing.
6.  Spring Cloud's load balancer then chooses one of these IP addresses, and the Feign client sends the request.

This approach simplifies the application architecture by removing the need for a separate service registry and relies directly on the robust, built-in features of the Kubernetes platform.
