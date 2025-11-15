# Summary: Section 15 - Container Orchestration using Kubernetes

## Обзор секции

В пятнадцатой секции курса изучалась оркестрация контейнеров с использованием Kubernetes. Были созданы Deployment, Service, ConfigMap и другие ресурсы Kubernetes для развертывания микросервисов.

## Что было изучено

### 1. Kubernetes основы
- Container orchestration platform
- Автоматическое управление контейнерами
- Масштабирование и самовосстановление
- Декларативное управление через YAML

### 2. Kubernetes объекты
- **Deployment** - управление репликами приложений
- **Service** - сетевой доступ к подам
- **ConfigMap** - хранение конфигураций
- **Pod** - минимальная единица развертывания

### 3. Deployment
- Определение образа контейнера
- Количество реплик
- Стратегия обновления
- Health checks (liveness, readiness probes)

### 4. Service
- Типы Service:
  - **ClusterIP** - внутренний доступ
  - **LoadBalancer** - внешний доступ
  - **NodePort** - доступ через порт узла
- Селекторы для связи с подами
- Балансировка нагрузки

### 5. ConfigMap
- Хранение конфигураций отдельно от кода
- Использование в подах через environment variables
- Обновление конфигураций без пересборки образов
- Централизованное управление

### 6. Environment Variables
- Передача конфигураций в контейнеры
- Использование ConfigMap для значений
- Переменные окружения для настройки приложений

### 7. Labels и Selectors
- Организация ресурсов через labels
- Селекторы для связи ресурсов
- Группировка и фильтрация

### 8. Команды kubectl
- `kubectl apply -f` - создание ресурсов
- `kubectl get` - просмотр ресурсов
- `kubectl describe` - детальная информация
- `kubectl delete` - удаление ресурсов
- `kubectl logs` - просмотр логов

## Что было реализовано

### Kubernetes манифесты
- **ConfigMap** (`eazybank-configmap`) - общие конфигурации
- **Deployments** для каждого сервиса:
  - configserver-deployment
  - eurekaserver-deployment
  - accounts-deployment
  - cards-deployment
  - loans-deployment
  - gatewayserver-deployment
- **Services** для каждого deployment
- **Keycloak** deployment и service

### ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: eazybank-configmap
data:
  SPRING_PROFILES_ACTIVE: "prod"
  SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"
```

### Deployment пример
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: accounts-deployment
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: accounts
        image: eazybytes/accounts:s12
        env:
        - name: SPRING_APPLICATION_NAME
          valueFrom:
            configMapKeyRef:
              name: eazybank-configmap
              key: ACCOUNTS_APPLICATION_NAME
```

### Service пример
```yaml
apiVersion: v1
kind: Service
metadata:
  name: accounts
spec:
  type: LoadBalancer
  ports:
  - port: 8080
    targetPort: 8080
```

## Технические детали

### Образы контейнеров
- Использование образов из предыдущих секций
- Теги версий (s12, s13, s14)
- Pull из registry

### Порты
- ConfigServer: 8071
- EurekaServer: 8070
- Accounts: 8080
- Cards: 9000
- Loans: 8090
- GatewayServer: 8072

### Конфигурация
- Использование ConfigMap для централизованных настроек
- Environment variables для специфичных значений
- Интеграция с Config Server

## Best Practices

1. **Ресурсы** - определение requests и limits
2. **Health Checks** - liveness и readiness probes
3. **Labels** - правильная организация ресурсов
4. **Secrets** - использование Secrets для чувствительных данных
5. **Namespaces** - разделение окружений
6. **Resource Quotas** - ограничение использования ресурсов

## Результаты секции

После завершения пятнадцатой секции студенты:
1. Понимают основы Kubernetes
2. Умеют создавать Deployment и Service
3. Знают, как использовать ConfigMap
4. Понимают сетевую модель Kubernetes
5. Умеют работать с kubectl
6. Знают best practices для Kubernetes

## Следующие шаги

В следующих секциях курса будут изучены:
- Helm для управления Kubernetes (Section 16)
- Kubernetes Service Discovery (Section 17)
- Развертывание в облаке (Section 18)
- Ingress и Service Mesh (Section 19)

