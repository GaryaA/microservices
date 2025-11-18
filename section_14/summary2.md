# Section 14: Event-Driven Microservices with Kafka

This section demonstrates the flexibility of the **Spring Cloud Stream** framework by swapping the underlying message broker from **RabbitMQ** to **Apache Kafka**. The goal is to show that with a well-designed abstraction, changing the messaging technology requires minimal code changes.

### Why it's needed:

Different message brokers have different strengths. RabbitMQ is a versatile and reliable traditional message broker, while Kafka is a high-throughput, distributed streaming platform often favored for big data and real-time analytics. An organization might choose one over the other based on specific project requirements for scalability, persistence, or ecosystem compatibility. This section shows how to make that switch.

### How it's configured:

The key insight of this section is that **no application code is changed**. The logic in `AccountsServiceImpl` (using `StreamBridge`) and `MessageFunctions` (the `Function` beans) remains identical to the previous section. The entire migration from RabbitMQ to Kafka is achieved through configuration changes.

1.  **Message Broker (Kafka)**:
    *   **Configuration**: The `docker-compose.yml` file is updated to remove the `rabbitmq` service and add a `kafka` service. This starts a Kafka broker, which will now handle message persistence and delivery.

2.  **Producer/Consumer Dependency Change**:
    *   **Configuration**: In the `pom.xml` of both the `accounts` and `message` services, the RabbitMQ binder dependency is replaced with the Kafka binder.
        *File: `accounts/pom.xml`*
        ```xml
        <!-- REMOVED -->
        <!-- <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
        </dependency> -->

        <!-- ADDED -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-binder-kafka</artifactId>
        </dependency>
        ```

3.  **Broker Connection Configuration**:
    *   **Configuration**: The environment variables in `docker-compose.yml` for the `accounts` and `message` services are updated to point to the Kafka broker instead of RabbitMQ.
        *File: `docker-compose/default/docker-compose.yml`*
        ```yaml
        accounts:
          # ...
          environment:
            # This line tells the Kafka binder where to find the broker
            SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS: "kafka:9092"
        ```

### How it works:

The application logic is completely decoupled from the messaging middleware.
*   When `StreamBridge.send()` is called in the `accounts` service, Spring Cloud Stream looks at the available binders on the classpath.
*   It finds the Kafka binder (`spring-cloud-stream-binder-kafka`).
*   The binder reads the connection properties (`SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS`) and knows how to connect to Kafka.
*   It then translates the abstract `send()` operation into a native Kafka "produce" operation, sending the message to a Kafka topic.
*   Similarly, on the consumer side (`message` service), the Kafka binder connects to the broker, subscribes to the appropriate topic, and delivers incoming messages to the `MessageFunctions` beans.

This demonstrates the power of the abstraction provided by Spring Cloud Stream, allowing developers to write portable, event-driven business logic that is not tied to a specific messaging technology.
