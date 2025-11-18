# Section 14: Event-Driven Architecture with Kafka and Streaming

This section demonstrates the power of using abstractions like **Spring Cloud Stream** by replacing the message broker from **RabbitMQ** to **Apache Kafka**. It also introduces the concept of **event streaming**, which is a core strength of Kafka.

### What is Kafka and Why Use It?

While RabbitMQ is a versatile message broker, **Kafka** is a **distributed event streaming platform**. This means it's designed from the ground up to handle a continuous, high-volume flow of events in real-time.

Key differences and advantages of Kafka:
*   **Durability & Immutability:** Events in Kafka are written to a persistent, append-only log called a **topic**. Events are not deleted after being consumed; they can be "re-read" multiple times by different consumers. This is powerful for analytics, auditing, or recovering from failures.
*   **Scalability:** Kafka is designed to be distributed across multiple servers (a cluster), allowing it to handle massive throughput.
*   **Streaming Capabilities:** Kafka is not just for simple messaging. It's the foundation for complex **stream processing**, where you can transform, aggregate, and analyze data in real-time as it flows through the system.

### How it's Implemented in the Project:

The most important takeaway from this section is that **the application code does not change**. The `StreamBridge` in the `accounts` service and the `Function` beans in the `message` service remain identical. The migration is purely a configuration change.

1.  **Dependency Change:**
    *   In the `pom.xml` of the `accounts` and `message` services, the RabbitMQ binder is swapped for the Kafka binder.
        ```xml
        <!-- REMOVED: spring-cloud-stream-binder-rabbit -->
        <!-- ADDED: -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-binder-kafka</artifactId>
        </dependency>
        ```

2.  **Configuration Change:**
    *   The `docker-compose.yml` is updated to run a `kafka` service instead of `rabbitmq`.
    *   The environment variables in `docker-compose.yml` are changed to point to the Kafka broker.
        ```yaml
        accounts:
          environment:
            SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS: "kafka:9092"
        ```

### What are Streams and How are they Used Here?

In the context of Kafka and Spring Cloud Stream, a "stream" is a continuous flow of data from a source to a destination. The `message` service in this project demonstrates a simple stream processing pipeline.

*   **Code:**
    *File: `message/.../functions/MessageFunctions.java`*
    ```java
    @Configuration
    public class MessageFunctions {
        @Bean
        public Function<AccountsMsgDto, AccountsMsgDto> email() { ... }

        @Bean
        public Function<AccountsMsgDto, Long> sms() { ... }
    }
    ```
*   **Configuration:**
    *File: `message/src/main/resources/application.yml`*
    ```yaml
    spring:
      cloud:
        function:
          definition: email|sms
        stream:
          bindings:
            emailsms-in-0:
              destination: send-communication
    ```

*   **Explanation of the Stream:**
    1.  **Input Stream:** The `emailsms-in-0` binding defines the start of the stream. It reads a continuous flow of `AccountsMsgDto` events from the `send-communication` Kafka topic.
    2.  **Processing Step 1 (`email`):** Each event from the input stream is passed to the `email` function. This function logs the event and then returns it.
    3.  **Processing Step 2 (`sms`):** The `|` in the function definition (`email|sms`) creates a **pipeline**. The output of the `email` function becomes the input for the `sms` function. The `sms` function receives the `AccountsMsgDto`, logs it, and its own output is sent to another topic (if configured).

This is a simple example, but it illustrates the core idea of stream processing: creating a pipeline of functions that process data as it arrives, in real-time. You could easily add more functions to this chain (e.g., `...|analytics|fraudDetection|...`) to build a complex data processing pipeline without changing the original producer. This is a powerful capability that Kafka is uniquely suited for.