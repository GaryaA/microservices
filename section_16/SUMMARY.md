# Summary: Section 16 - Deep Dive on Helm

## Обзор секции

В шестнадцатой секции курса изучался Helm - package manager для Kubernetes. Были созданы Helm charts для управления развертыванием микросервисов, что упрощает управление сложными Kubernetes приложениями.

## Что было изучено

### 1. Helm основы
- Package manager для Kubernetes
- Упрощение управления Kubernetes приложениями
- Шаблонизация манифестов
- Версионирование развертываний

### 2. Helm Charts
- Структура chart:
  - `Chart.yaml` - метаданные chart
  - `values.yaml` - значения по умолчанию
  - `templates/` - шаблоны Kubernetes манифестов
  - `templates/_helpers.tpl` - вспомогательные функции

### 3. Chart Dependencies
- Зависимости между charts
- Переиспользование общих компонентов
- Library charts для общих шаблонов
- Управление зависимостями через `Chart.yaml`

### 4. Templates и Values
- Go template синтаксис
- Переменные через `values.yaml`
- Условная логика в шаблонах
- Функции и pipelines

### 5. Common Templates
- Создание переиспользуемых шаблонов
- `eazybank-common` library chart
- Общие deployment и service шаблоны
- Параметризация через values

### 6. Environment-specific Charts
- Отдельные charts для разных окружений
- `dev-env`, `qa-env`, `prod-env`
- Разные values для каждого окружения
- Управление версиями

### 7. Helm Commands
- `helm create` - создание нового chart
- `helm install` - установка chart
- `helm upgrade` - обновление release
- `helm uninstall` - удаление release
- `helm list` - список releases
- `helm template` - рендеринг шаблонов

### 8. Helm Values
- Переопределение значений при установке
- Использование `-f` для файлов values
- `--set` для отдельных значений
- Приоритет значений

### 9. Helm Releases
- Версионирование развертываний
- История изменений через `helm history`
- Rollback через `helm rollback`
- Управление жизненным циклом

## Что было реализовано

### Library Chart: eazybank-common
- Общие шаблоны для всех сервисов
- Deployment template с параметризацией
- Service template
- ConfigMap integration
- Environment variables handling

### Service Charts
- Отдельные charts для каждого сервиса:
  - configserver
  - eurekaserver
  - accounts
  - cards
  - loans
  - gatewayserver
  - message

### Environment Charts
- **dev-env** - development окружение
- **qa-env** - QA окружение
- **prod-env** - production окружение
- Зависимости от service charts

### Third-party Charts
- Использование Bitnami charts:
  - Keycloak
  - Kafka
  - Grafana
  - Loki
  - Tempo
  - Prometheus

### Common Template Example
```yaml
{{- define "common.deployment" -}}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.deploymentName }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
      - name: {{ .Values.appLabel }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
{{- end }}
```

## Технические детали

### Chart Structure
```
chart-name/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── _helpers.tpl
└── charts/ (dependencies)
```

### Values Example
```yaml
image:
  repository: eazybytes/accounts
  tag: s12
replicaCount: 1
deploymentName: accounts-deployment
appLabel: accounts
```

### Dependencies
```yaml
dependencies:
  - name: eazybank-common
    version: 0.1.0
    repository: file://../../eazybank-common
  - name: accounts
    version: 0.1.0
    repository: file://../../eazybank-services/accounts
```

## Best Practices

1. **Версионирование** - семантическое версионирование charts
2. **Переиспользование** - использование library charts
3. **Values организация** - логическая структура values
4. **Документация** - README для каждого chart
5. **Тестирование** - `helm template` для проверки
6. **Безопасность** - использование Secrets для чувствительных данных

## Результаты секции

После завершения шестнадцатой секции студенты:
1. Понимают концепцию Helm
2. Умеют создавать Helm charts
3. Знают, как использовать templates и values
4. Понимают зависимости между charts
5. Умеют управлять releases
6. Знают best practices для Helm

## Следующие шаги

В следующих секциях курса будут изучены:
- Kubernetes Service Discovery (Section 17)
- Развертывание в облаке (Section 18)
- Ingress и Service Mesh (Section 19)
- Заключение курса (Section 20)

