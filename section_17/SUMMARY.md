# Summary: Section 17 - Server-side Service Discovery and Load Balancing using Kubernetes

## Обзор секции

В семнадцатой секции курса изучалась server-side service discovery и load balancing в Kubernetes. Была рассмотрена замена Eureka на встроенные возможности Kubernetes для обнаружения сервисов и балансировки нагрузки.

## Что было изучено

### 1. Kubernetes Native Service Discovery
- Встроенные возможности Kubernetes для service discovery
- DNS-based discovery через Service объекты
- Отказ от внешних систем (Eureka) в пользу нативных механизмов
- Упрощение архитектуры

### 2. Spring Cloud Kubernetes
- Интеграция Spring Cloud с Kubernetes
- Использование Kubernetes API для service discovery
- Конфигурация через ConfigMap и Secrets
- Health checks через Kubernetes probes

### 3. Kubernetes Services
- Service объекты для сетевого доступа
- DNS имена для сервисов
- Автоматическая балансировка нагрузки
- Service types: ClusterIP, LoadBalancer, NodePort

### 4. Server-side Load Balancing
- Балансировка на уровне Kubernetes
- Не требует client-side библиотек
- Автоматическое распределение трафика
- Health checks для исключения нездоровых подов

### 5. Spring Cloud Kubernetes Discovery
- Автоматическое обнаружение сервисов через Kubernetes API
- Интеграция с Spring Cloud LoadBalancer
- Использование Kubernetes Service names
- Отказ от Eureka Client

### 6. ConfigMap и Secrets
- Использование Kubernetes ConfigMap для конфигураций
- Secrets для чувствительных данных
- Динамическое обновление конфигураций
- Интеграция с Spring Cloud Config

### 7. Health Checks
- Liveness probes - проверка работоспособности
- Readiness probes - проверка готовности принимать трафик
- Интеграция с Spring Boot Actuator
- Автоматическое восстановление

### 8. Service Mesh (опционально)
- Введение в концепцию Service Mesh
- Подготовка к использованию Istio
- Дополнительные возможности для service discovery

## Что было реализовано

### Замена Eureka на Kubernetes Services
- Удаление Eureka Server из архитектуры
- Использование Kubernetes Service объектов
- DNS-based discovery через имена сервисов

### Spring Cloud Kubernetes Integration
- Зависимость `spring-cloud-starter-kubernetes-client`
- Автоматическое обнаружение сервисов
- Использование Kubernetes API

### Service Configuration
```yaml
apiVersion: v1
kind: Service
metadata:
  name: accounts
spec:
  selector:
    app: accounts
  ports:
  - port: 8080
    targetPort: 8080
```

### Service Discovery через DNS
- Использование имен сервисов: `accounts`, `cards`, `loans`
- Автоматическое разрешение через Kubernetes DNS
- Load balancing через Service

## Технические детали

### Зависимости
- `spring-cloud-starter-kubernetes-client` - Kubernetes интеграция
- `spring-cloud-starter-kubernetes-client-config` - ConfigMap поддержка
- Удаление Eureka зависимостей

### Kubernetes API
- Доступ к Kubernetes API для service discovery
- RBAC для доступа к API
- ServiceAccount для приложений

### DNS Resolution
- Формат: `{service-name}.{namespace}.svc.cluster.local`
- Упрощенный формат: `{service-name}` в том же namespace
- Автоматическое разрешение

## Best Practices

1. **Использование нативных возможностей** - предпочтение Kubernetes механизмов
2. **Service naming** - понятные имена сервисов
3. **Health checks** - правильная настройка probes
4. **RBAC** - минимальные необходимые права
5. **Namespaces** - организация сервисов по namespace
6. **Monitoring** - отслеживание service discovery

## Результаты секции

После завершения семнадцатой секции студенты:
1. Понимают Kubernetes native service discovery
2. Знают, как использовать Spring Cloud Kubernetes
3. Умеют заменять Eureka на Kubernetes Services
4. Понимают server-side load balancing
5. Знают, как использовать ConfigMap и Secrets
6. Знают best practices для Kubernetes service discovery

## Следующие шаги

В следующих секциях курса будут изучены:
- Развертывание в облаке (Section 18)
- Ingress и Service Mesh (Section 19)
- Заключение курса (Section 20)

