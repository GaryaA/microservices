# Summary: Section 11 - Observability and Monitoring of Microservices

## Обзор секции

В одиннадцатой секции курса изучались инструменты и практики для обеспечения observability (наблюдаемости) и мониторинга микросервисов. Были интегрированы Prometheus, Grafana, Loki, Tempo и OpenTelemetry для полного понимания работы системы.

## Что было изучено

### 1. Три столпа Observability
- **Metrics** - количественные измерения производительности
- **Logs** - структурированные записи событий
- **Traces** - отслеживание запросов через систему

### 2. Micrometer
- Библиотека для сбора метрик в Java приложениях
- Интеграция с Spring Boot Actuator
- Поддержка различных реестров метрик
- Автоматический сбор метрик JVM, HTTP, базы данных

### 3. Prometheus
- Система мониторинга и сбора метрик
- Pull модель сбора данных
- PromQL для запросов метрик
- Хранение временных рядов

### 4. Grafana
- Визуализация метрик и логов
- Создание дашбордов
- Интеграция с Prometheus, Loki, Tempo
- Алерты на основе метрик

### 5. Loki
- Система агрегации логов
- Интеграция с Grafana
- Эффективное хранение логов
- LogQL для запросов логов

### 6. Tempo
- Распределенная трассировка
- Хранение trace данных
- Интеграция с Grafana
- Поддержка различных форматов трассировки

### 7. OpenTelemetry
- Стандарт для инструментирования приложений
- Автоматическая инструментация через Java Agent
- Сбор метрик, логов и трасс
- Экспорт в различные системы

### 8. Структурированное логирование
- Использование trace ID и span ID
- Паттерн логирования: `%5p [${spring.application.name},%X{trace_id},%X{span_id}]`
- Корреляция логов с трассами
- Контекстная информация в логах

### 9. Distributed Tracing
- Отслеживание запросов через несколько микросервисов
- Correlation ID для связывания логов
- Визуализация потока запросов
- Выявление узких мест

### 10. Health Checks и Readiness/Liveness Probes
- Actuator health endpoints
- Kubernetes probes
- Мониторинг состояния приложений
- Автоматическое восстановление

## Что было реализовано

### Prometheus
- Сбор метрик из всех микросервисов
- Конфигурация через `prometheus.yml`
- Scraping endpoints: `/actuator/prometheus`
- Хранение метрик для анализа

### Grafana
- Дашборды для визуализации метрик
- Интеграция с Prometheus, Loki, Tempo
- Настройка datasources
- Создание панелей для мониторинга

### Loki
- Агрегация логов из всех микросервисов
- Хранение в MinIO (S3-совместимое хранилище)
- Интеграция с Grafana для просмотра логов
- Поиск по логам через LogQL

### Tempo
- Хранение distributed traces
- Прием трасс через OTLP
- Интеграция с Grafana
- Визуализация трасс запросов

### OpenTelemetry
- Автоматическая инструментация через Java Agent
- Сбор метрик, логов и трасс
- Экспорт в Tempo
- Конфигурация через переменные окружения

### Alloy
- Агент для сбора и экспорта данных
- Интеграция с различными системами
- Конфигурация через YAML

## Технические детали

### Зависимости
- `micrometer-registry-prometheus` - для экспорта метрик
- `opentelemetry-javaagent` - для автоматической инструментации
- Spring Boot Actuator - для endpoints

### Конфигурация OpenTelemetry
```yaml
environment:
  JAVA_TOOL_OPTIONS: "-javaagent:/app/libs/opentelemetry-javaagent-2.11.0.jar"
  OTEL_EXPORTER_OTLP_ENDPOINT: http://tempo:4318
  OTEL_METRICS_EXPORTER: none
  OTEL_LOGS_EXPORTER: none
```

### Структура логов
```
%5p [${spring.application.name},%X{trace_id},%X{span_id}]
```

### Prometheus Scraping
```yaml
scrape_configs:
  - job_name: 'accounts'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['accounts:8080']
```

## Best Practices

1. **Структурированное логирование** - использование JSON или структурированных форматов
2. **Trace ID в логах** - корреляция логов с трассами
3. **Метрики бизнес-логики** - не только технические метрики
4. **Алерты** - настройка уведомлений о проблемах
5. **Retention политики** - управление хранением данных
6. **Производительность** - минимизация влияния на приложение

## Результаты секции

После завершения одиннадцатой секции студенты:
1. Понимают три столпа observability
2. Умеют настраивать Prometheus для сбора метрик
3. Знают, как использовать Grafana для визуализации
4. Умеют настраивать Loki для агрегации логов
5. Понимают distributed tracing с Tempo
6. Знают, как использовать OpenTelemetry для инструментации
7. Знают best practices для observability

## Следующие шаги

В следующих секциях курса будут изучены:
- Безопасность микросервисов (Section 12)
- Event-driven архитектура с RabbitMQ (Section 13)
- Event-driven архитектура с Kafka (Section 14)
- Kubernetes оркестрация (Section 15)

