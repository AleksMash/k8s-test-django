## Описание проекта

Проект докеризированого Django-сайта с целью изучения работы с Kubernetes.

## Разворачиваем сайт в k8s

Для изучения разворачиваем локальный кластер k8s - устанавливаем и настраиваем инструменты локальной разработки **kubectl** и **minikube** ([инструкции по установке](https://kubernetes.io/docs/tasks/tools/)).

Клонируем настоящий репозитарий к себе, в терминале перейдие в корневой каталог проекта. Все  bash-команды, приведенные ниже даны относительно корневой папки проекта.

Создаем образ Django-приложения внутри кластера:

```bash
cd backend_main_django
minikube image build -t django_app 
```

Устанавливаем Helm ([руководство по установке](https://helm.sh/ru/docs/intro/install/) и регистрируем в нем репозиторий чартов.

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Устанавливаем в кластер базу данных postgres:

```
helm install database bitnami/postgresql --values .\kubernetes\db_params.yml
```

Создаем в кластере конфигурацию с переменными окружения для Django-приложения

```bash
kubectl apply -f .\kubernetes\configmap_db_in_k8s.yml
```

Активируем Ingress-контроллер в нашем кластере

```bash
minikube addons enable ingress
```

Деплоим наш Django - проект

```bash
kubectl apply -f .\kubernetes\deployment+service+ingress.yml
```

Узнайте IP адрес кластера

```bash
minikube ip
```

Для целей локального тестирования работы сайта пропишите в файле `hosts`  DNS имя `star-burger.test`  на IP полученный адрес кластера См. ([Файл hosts: где находится и как его изменить | Рег.ру](https://help.reg.ru/support/dns-servery-i-nastroyka-zony/rabota-s-dns-serverami/fayl-hosts-gde-nakhoditsya-i-kak-yego-izmenit)

Выполните миграции:

```bash
kubectl apply -f .\kubernetes\migrate_job.yml
```

Узнайте имя, присвоенное Pod Django-приложения (команда `kubectl get pods`), после чего  
зайдите в терминал контейнера Django-приложения...

```bash
kubectl exec -ti <имя пода Django-приложения> -- bash
```

...и создайте суперпользователя:

```bash
python manage.py createsuperuser
```

Создайте периодическую задачу для очистки устаревших сессий.

```bash
kubectl apply -f .\manifests\django-clearsessions.yml
```

Если все было настроено верно, то перейдя по адресу [http://star-burger.test](http://star-burger.test) вы окажетесь на странице входа в административную панель django. Убедитесь, что вход по имени и паролю созданного выше суперпользователя работает.