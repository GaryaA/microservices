# Summary: Section 13 - Event Driven Microservices using RabbitMQ

## Обзор секции

В тринадцатой секции курса изучалась event-driven архитектура микросервисов с использованием RabbitMQ и Spring Cloud Stream. Была реализована асинхронная коммуникация между микросервисами через сообщения.

## Что было изучено

### 1. Event-Driven Architecture
- Асинхронная коммуникация между микросервисами
- Слабая связанность сервисов
- Масштабируемость и отказоустойчивость
- Обработка событий вместо синхронных вызовов

### 2. RabbitMQ
- Message broker для асинхронной коммуникации
- Поддержка различных протоколов (AMQP)
- Гарантии доставки сообщений
- Очереди и exchanges для маршрутизации

### 3. Spring Cloud Stream
- Абстракция над message brokers
- Упрощение работы с messaging
- Поддержка различных binders (RabbitMQ, Kafka)
- Функциональный подход к обработке сообщений

### 4. Spring Cloud Functions
- Функциональный подход к обработке сообщений
- Определение функций через `@Bean` методы
- Автоматическая привязка к каналам
- Упрощение кода обработки

### 5. Message Bindings
- Input bindings - прием сообщений
- Output bindings - отправка сообщений
- Конфигурация через `spring.cloud.stream.bindings`
- Группировка потребителей через `group`

### 6. Message Destinations
- Topics/Exchanges для маршрутизации
- Именование destinations
- Подписка на события
- Публикация событий

### 7. Обработка сообщений
- Consumer функции для обработки входящих сообщений
- Producer функции для отправки сообщений
- Обработка ошибок
- Retry механизмы

### 8. Интеграция с существующими сервисами
- Отправка событий из Accounts сервиса
- Обработка событий в Message сервисе
- Асинхронная обработка коммуникаций (email, SMS)

## Что было реализовано

### RabbitMQ
- Message broker для всех микросервисов
- Exchanges и queues для маршрутизации
- Гарантии доставки

### Message Service
- **Порт**: 9010
- **Функциональность**:
  - Обработка событий отправки коммуникаций
  - Отправка email и SMS
  - Публикация событий об успешной отправке

### Accounts Service Integration
- Публикация событий при создании/обновлении счетов
- Destination: `send-communication`
- Асинхронная отправка уведомлений

### Spring Cloud Functions
```java
@Bean
public Function<MessageDto, MessageDto> email() {
    return messageDto -> {
        // Отправка email
        return messageDto;
    };
}

@Bean
public Function<MessageDto, MessageDto> sms() {
    return messageDto -> {
        // Отправка SMS
        return messageDto;
    };
}
```

### Конфигурация Stream Bindings
```yaml
spring:
  cloud:
    function:
      definition: email|sms
    stream:
      bindings:
        emailsms-in-0:
          destination: send-communication
          group: ${spring.application.name}
        emailsms-out-0:
          destination: communication-sent
```

## Технические детали

### Зависимости
- `spring-cloud-stream` - основной framework
- `spring-cloud-stream-binder-rabbit` - RabbitMQ binder
- `spring-cloud-function-core` - функциональная поддержка

### RabbitMQ Configuration
```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    connection-timeout: 10s
```

### Message Flow
1. Accounts service публикует событие в `send-communication`
2. Message service получает событие
3. Message service обрабатывает (email/sms)
4. Message service публикует событие в `communication-sent`
5. Accounts service получает подтверждение

## Best Practices

1. **Idempotency** - обработка дублирующихся сообщений
2. **Error Handling** - обработка ошибок при обработке сообщений
3. **Dead Letter Queues** - для сообщений, которые не удалось обработать
4. **Message Versioning** - поддержка разных версий сообщений
5. **Monitoring** - отслеживание обработки сообщений
6. **Testing** - тестирование event-driven логики

## Результаты секции

После завершения тринадцатой секции студенты:
1. Понимают event-driven архитектуру
2. Знают, как использовать RabbitMQ
3. Умеют работать с Spring Cloud Stream
4. Знают Spring Cloud Functions
5. Понимают, как интегрировать messaging в микросервисы
6. Знают best practices для event-driven архитектуры

## Следующие шаги

В следующих секциях курса будут изучены:
- Event-driven архитектура с Kafka (Section 14)
- Kubernetes оркестрация (Section 15)
- Helm для управления Kubernetes (Section 16)
- Kubernetes Service Discovery (Section 17)

