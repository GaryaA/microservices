# Summary: Section 14 - Event Driven Microservices using Kafka

## Обзор секции

В четырнадцатой секции курса изучалась event-driven архитектура микросервисов с использованием Apache Kafka и Spring Cloud Stream. Была реализована высокопроизводительная асинхронная коммуникация через Kafka topics.

## Что было изучено

### 1. Apache Kafka
- Высокопроизводительный distributed streaming platform
- Хранение сообщений в topics
- Партиционирование для масштабирования
- Гарантии доставки и упорядочивания

### 2. Kafka vs RabbitMQ
- Kafka: высокая пропускная способность, долгосрочное хранение
- RabbitMQ: традиционные очереди, гарантии доставки
- Выбор зависит от use case

### 3. Kafka Architecture
- **Topics** - категории сообщений
- **Partitions** - разделение topics для масштабирования
- **Producers** - отправка сообщений
- **Consumers** - получение сообщений
- **Consumer Groups** - распределение нагрузки

### 4. Spring Cloud Stream с Kafka
- Kafka binder для Spring Cloud Stream
- Конфигурация brokers
- Автоматическое создание topics
- Управление consumer groups

### 5. Kafka Binder Configuration
- Настройка brokers
- Конфигурация consumer groups
- Настройки партиционирования
- Offsets management

### 6. High Throughput
- Партиционирование topics
- Параллельная обработка
- Масштабирование consumers
- Batch processing

### 7. Message Ordering
- Гарантии упорядочивания в пределах партиции
- Key-based партиционирование
- Важность выбора partition key

### 8. Consumer Groups
- Распределение сообщений между consumers
- Масштабирование обработки
- Rebalancing при добавлении/удалении consumers

## Что было реализовано

### Kafka Broker
- **Порт**: 9092
- **Конфигурация**: KRaft mode (без Zookeeper)
- **Функциональность**:
  - Хранение сообщений в topics
  - Партиционирование
  - Гарантии доставки

### Message Service с Kafka
- Обработка событий через Kafka topics
- Consumer group для распределения нагрузки
- Публикация подтверждений

### Accounts Service Integration
- Публикация событий в Kafka topics
- Асинхронная обработка
- Интеграция с существующей логикой

### Spring Cloud Stream Kafka Configuration
```yaml
spring:
  cloud:
    stream:
      bindings:
        emailsms-in-0:
          destination: send-communication
          group: ${spring.application.name}
      kafka:
        binder:
          brokers:
            - localhost:9092
```

## Технические детали

### Зависимости
- `spring-cloud-stream` - основной framework
- `spring-cloud-stream-binder-kafka` - Kafka binder
- `spring-cloud-function-core` - функциональная поддержка

### Kafka Configuration
- KRaft mode для упрощения
- Настройка listeners
- Advertised listeners для доступа

### Message Processing
- Consumer groups для масштабирования
- Партиционирование для параллелизма
- Offsets для отслеживания прогресса

## Best Practices

1. **Партиционирование** - правильный выбор partition key
2. **Consumer Groups** - использование для масштабирования
3. **Idempotency** - обработка дублирующихся сообщений
4. **Monitoring** - отслеживание lag и throughput
5. **Retention** - настройка хранения сообщений
6. **Schema Registry** - управление схемами сообщений (Avro, JSON Schema)

## Результаты секции

После завершения четырнадцатой секции студенты:
1. Понимают Apache Kafka архитектуру
2. Знают различия между Kafka и RabbitMQ
3. Умеют использовать Spring Cloud Stream с Kafka
4. Понимают партиционирование и consumer groups
5. Знают, как масштабировать обработку сообщений
6. Знают best practices для Kafka в микросервисах

## Следующие шаги

В следующих секциях курса будут изучены:
- Kubernetes оркестрация (Section 15)
- Helm для управления Kubernetes (Section 16)
- Kubernetes Service Discovery (Section 17)
- Развертывание в облаке (Section 18)

