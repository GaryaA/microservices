# Section 13: Event-Driven Microservices with RabbitMQ & Spring Cloud Stream

This section refactors the architecture from synchronous request-response communication to an **asynchronous, event-driven model** using **RabbitMQ** as a message broker and **Spring Cloud Stream** as the abstraction layer.

### Why it's needed:

In a synchronous model, if the `accounts` service needs to notify another service (e.g., a notification service), it has to make a direct API call and wait for a response. This creates tight coupling and a single point of failure. If the notification service is down, the account creation process might fail.

In an event-driven model, the `accounts` service simply publishes an event (a message) to a message broker (RabbitMQ) and immediately continues its work. Other services can then subscribe to these events and react to them independently and asynchronously. This decouples the services and makes the system more resilient and scalable.

### How it's configured and used:

1.  **Message Broker (RabbitMQ)**:
    *   **Configuration**: The `docker-compose.yml` is updated to run a **RabbitMQ** container. RabbitMQ will handle the storage and delivery of messages between services.

2.  **The Producer (`accounts` service)**:
    *   **Goal**: To send a "new account created" message.
    *   **Dependencies**: The `pom.xml` is updated with `spring-cloud-stream` and `spring-cloud-stream-binder-rabbit`.
    *   **Code**: The `accounts` service uses the `StreamBridge` utility provided by Spring Cloud Stream to send a message.
        *File: `accounts/src/main/java/com/eazybytes/accounts/service/impl/AccountsServiceImpl.java`*
        ```java
        @AllArgsConstructor
        public class AccountsServiceImpl implements IAccountsService {
            // ...
            private final StreamBridge streamBridge;

            private void sendCommunication(Accounts account, Customer customer) {
                var accountsMsgDto = new AccountsMsgDto(account.getAccountNumber(), customer.getName(), ...);
                log.info("Sending Communication request for the details: {}", accountsMsgDto);
                // Send the DTO to a binding named "sendCommunication-out-0"
                var result = streamBridge.send("sendCommunication-out-0", accountsMsgDto);
                log.info("Is the Communication request successfully triggered ? : {}", result);
            }
        }
        ```
    *   **Binding Configuration**: The `accounts` service's `application.yml` maps the `sendCommunication-out-0` binding to a RabbitMQ destination (a topic exchange).

3.  **The Consumer (`message` service)**:
    *   **Goal**: To receive the "new account created" message and "send" an email and SMS.
    *   **Dependencies**: The new `message` service also includes the Spring Cloud Stream and RabbitMQ dependencies.
    *   **Code**: This service uses **Spring Cloud Function**. It defines `Bean`s of type `java.util.function.Function`. Spring Cloud Stream automatically treats these as message handlers.
        *File: `message/src/main/java/com/eazybytes/message/functions/MessageFunctions.java`*
        ```java
        @Configuration
        public class MessageFunctions {
            @Bean
            public Function<AccountsMsgDto, AccountsMsgDto> email() {
                return accountsMsgDto -> {
                    log.info("Sending email with the details : " + accountsMsgDto.toString());
                    return accountsMsgDto;
                };
            }

            @Bean
            public Function<AccountsMsgDto, Long> sms() {
                return accountsMsgDto -> {
                    log.info("Sending sms with the details : " + accountsMsgDto.toString());
                    return accountsMsgDto.accountNumber();
                };
            }
        }
        ```
    *   **Binding Configuration**: The `message` service's `application.yml` defines how to route incoming messages from RabbitMQ to these functions.
        *File: `message/src/main/resources/application.yml`*
        ```yaml
        spring:
          cloud:
            function:
              definition: email|sms # Chains the functions: the output of email() goes to sms()
            stream:
              bindings:
                emailsms-in-0: # The input binding for the function chain
                  destination: send-communication # Subscribes to the 'send-communication' topic
                  group: ${spring.application.name} # Consumer group
        ```

### How it all works together:

1.  When a new account is created, `AccountsServiceImpl` calls `sendCommunication()`.
2.  `StreamBridge` sends the `AccountsMsgDto` message to the `sendCommunication-out-0` binding.
3.  Spring Cloud Stream, via the RabbitMQ binder, publishes this message to a RabbitMQ topic named `send-communication`.
4.  The `message` service, which is subscribed to this topic, receives the message on its `emailsms-in-0` binding.
5.  Spring Cloud Stream invokes the `email` function with the message payload.
6.  The output of the `email` function is then passed as input to the `sms` function.
7.  This creates a loosely coupled, resilient system where the `accounts` service doesn't need to know or care about how notifications are sent.
