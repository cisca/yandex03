# Проблема монолитного решения

1. Отказоусточивость. Если монолитное приложение откажет или посыпется база данных, то все управление системой остановится.
2. Автоматизация и масштабирование. При необходимости добавить новых пользователей или новую функциональность организация может столкенуться с трудностью или невозможностью добавления новой фунциональности.
3. В условиях, когда конкретные физические устройства (датчики) могут быть сняты с производства или изменены производителем, нам требуется, чтобы система поддерживала версионность API и при этом в БД данные попадали унифицированными. В условиях монолита наращивание количества типа устройст и версионности протоколов взаимодействий приведет к сильному разрастанию кодовой базы и сложности её поддержания.
4. Требования к возможности подключать средства пользователями самостоятельно предполагает необходимость усиленной поддержки пользователей и наличия средств диагностики. Иначе говоря, нам нужны системы логирования данных, которые техническая поддержка может просматривать в режиме онлайн, а также преобразование этих данных в человеко-понятный формат, чтобы пользователь мог сам диагностировать ошибки при исталляции новых устройств. Таким образом, нам нужен сервис, который будет работать с логами системы и из преобразованием.
5. Системы обработки потоковых данных (видеонаблюдение) меняет требования к профилю нагрузки, а это в свою очередь приводит нас к необходимости выведения этих данных в отдельный канал, чтобы передаваемые данные не "положили" нам сервер.

Context.pulm - диаграмма контекста

Container.pulm - диаграмма контейнеров

Component.pulm - диаграмма компонентов

Code.pulm - диаграмма кода

ER.pulm - диаграмма ER




# Базовая настройка

## Запуск minikube

[Инструкция по установке](https://minikube.sigs.k8s.io/docs/start/)

```bash
minikube start
```

## Добавление токена авторизации GitHub

[Получение токена](https://github.com/settings/tokens/new)

```bash
kubectl create secret docker-registry ghcr --docker-server=https://ghcr.io --docker-username=<github_username> --docker-password=<github_token> -n default
```

## Установка API GW kusk

[Install Kusk CLI](https://docs.kusk.io/getting-started/install-kusk-cli)

```bash
kusk cluster install
```

## Смена адреса образа в helm chart

После того как вы сделали форк репозитория и у вас в репозитории отработал GitHub Action. Вам нужно получить адрес образа <https://github.com/><github_username>/architecture-sprint-3/pkgs/container/architecture-sprint-3

Он выглядит таким образом
```ghcr.io/<github_username>/architecture-sprint-3:latest```

Замените адрес образа в файле `helm/smart-home-monolith/values.yaml` на полученный файл:

```yaml
image:
  repository: ghcr.io/<github_username>/architecture-sprint-3
  tag: latest
```

## Настройка terraform

[Установите Terraform](https://yandex.cloud/ru/docs/tutorials/infrastructure-management/terraform-quickstart#install-terraform)

Создайте файл ~/.terraformrc

```hcl
provider_installation {
  network_mirror {
    url = "https://terraform-mirror.yandexcloud.net/"
    include = ["registry.terraform.io/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
```

## Применяем terraform конфигурацию

```bash
cd terraform
terraform init
terraform apply
```

## Настройка API GW

```bash
kusk deploy -i api.yaml
```

## Проверяем работоспособность

```bash
kubectl port-forward svc/kusk-gateway-envoy-fleet -n kusk-system 8080:80
curl localhost:8080/hello
```

## Delete minikube

```bash
minikube delete
```
