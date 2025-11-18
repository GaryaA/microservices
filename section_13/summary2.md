# Section 13: Event-Driven Architecture with RabbitMQ

This section refactors the system to use an **Event-Driven Architecture (EDA)**. This is a fundamental shift from the previous request-response model to a more decoupled, resilient, and scalable design using **RabbitMQ** as a message broker.

### What is Event-Driven Architecture and Why is it Needed?

In a traditional **request-response** model, services are tightly coupled. When the `accounts` service creates an account, it might need to directly call a `notification` service and wait for a response. If the `notification` service is slow or down, the entire account creation process is blocked or fails.

**Event-Driven Architecture** decouples services by introducing a "message broker" (like RabbitMQ) that acts as an intermediary.
*   Instead of making a direct call, a service (the **Producer**) simply emits an **event**—a message describing something that happened (e.g., "AccountCreated").
*   It sends this event to the message broker and immediately moves on, without waiting for a response.
*   Other services (the **Consumers**) can subscribe to these events. When a new event appears, the broker delivers it to them, and they can react independently.

This makes the system:
*   **Resilient:** The `accounts` service can create accounts even if the `message` service is down.
*   **Scalable:** You can add more consumers to handle high loads without affecting the producer.
*   **Flexible:** You can easily add new services that react to existing events without changing any of the original services.

### How it's Implemented in the Project:

The project uses **Spring Cloud Stream** as a powerful abstraction layer over RabbitMQ.

1.  **The Producer (`accounts` service):**
    *   **Goal:** To publish an `AccountCreated` event.
    *   **Mechanism:** It uses the `StreamBridge` utility from Spring Cloud Stream. This is a simple way to send a message to a specific "output binding".
    *   **Code:**
        *File: `accounts/.../AccountsServiceImpl.java`*
        ```java
        private final StreamBridge streamBridge;
        // ...
        private void sendCommunication(Accounts account, Customer customer) {
            var accountsMsgDto = new AccountsMsgDto(account.getAccountNumber(), ...);
            // Send the event to the "sendCommunication-out-0" channel
            streamBridge.send("sendCommunication-out-0", accountsMsgDto);
        }
        ```
    *   Spring Cloud Stream takes this message and, using the RabbitMQ "binder", publishes it to a RabbitMQ exchange (topic).

2.  **The Consumer (`message` service):**
    *   **Goal:** To listen for `AccountCreated` events and process them (e.g., send an email/SMS).
    *   **Mechanism:** It uses **Spring Cloud Function**. By simply defining a `Bean` of type `java.util.function.Function`, it automatically becomes a message handler.
    *   **Code:**
        *File: `message/.../functions/MessageFunctions.java`*
        ```java
        @Configuration
        public class MessageFunctions {
            @Bean
            public Function<AccountsMsgDto, AccountsMsgDto> email() {
                return accountsMsgDto -> {
                    log.info("Sending email with the details: " + accountsMsgDto);
                    return accountsMsgDto;
                };
            }
        }
        ```
    *   **Binding Configuration:** The service's `application.yml` file "binds" the `email` function to the RabbitMQ queue.
        *File: `message/src/main/resources/application.yml`*
        ```yaml
        spring:
          cloud:
            stream:
              bindings:
                # The input binding for the 'email' function
                email-in-0:
                  destination: send-communication # Subscribes to the 'send-communication' topic
                  group: message # A consumer group name
        ```

This setup creates a truly decoupled system where the `accounts` service has no knowledge of the `message` service or how notifications are handled. It simply announces that an event has occurred.