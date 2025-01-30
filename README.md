# Проект по автоматизации развертывания приложения и системы мониторинга с использованием Ansible, Minikube, Nginx, Prometheus и Grafana

## Описание проекта

Данный проект представляет собой инфраструктурное решение для автоматизированного развёртывания приложения и системы мониторинга в локальном кластере Kubernetes с использованием Minikube. Инструменты мониторинга включают Prometheus и Grafana, настроенные для сбора метрик с приложений и узлов кластера. Все шаги выполнены с помощью сценариев Ansible, которые автоматизируют установку, настройку и деплой необходимых сервисов.

Доступные сервисы:

- **Sytem info app**: [http://service.scelle.ru]
- **Prometheus**: [http://prometheus.scelle.ru]
- **Grafana**: [http://grafana.scelle.ru] (login: admin password: etc6OT00p4tkpZPEUzrxkgltS0BdMJbWYns4xbtd)



## Состав проекта

- **Ansible**: Используется для автоматизации всех этапов установки и настройки.
- **Minikube**: Локальный кластер Kubernetes, позволяющий разворачивать контейнеризированные приложения.
- **Nginx**: Используется в качестве обратного прокси для внешнего доступа к сервисам через домены.
- **Prometheus**: Система мониторинга и алертинга, настроенная на сбор метрик с узлов кластера и приложений.
- **Grafana**: Платформа визуализации данных, интегрированная с Prometheus для отображения метрик.

## Структура проекта

- **roles/**: Каталог с Ansible-ролями для установки и настройки компонентов.
  - **deploy_app**: Роль для деплоя приложения в Minikube.
  - **expose_ingress_controller**: Роль для настройки и публикации Ingress-контроллера и настройки Nginx для проксирования запросов к сервисам.
  - **install_tools**: Роль для установки необходимых компонентов для деплоя приложения в кластер.
  - **setup_monitoring**: Роль для установки и настройки Node Exporter, Prometheus и Grafana.
- **playbook.yml**: Основной playbook Ansible для запуска всех ролей.

## Требования:

- **Ansible**.

## Установка и запуск

1. **Установите Ansible**:
    https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#

2. **Настройка Ansible-инвентаря**:
    Добавьте IP-адреса или имена хостов для управления конфигурацией в файл инвентаря (если это требуется для распределенного окружения).

3. **Запуск playbook Ansible**:
    Запустите основной playbook для установки и настройки всех компонентов:

    ```ansible-playbook playbook.yml -i path/to/inventory/filename -u <username>```

4. **Проверка развертывания**:
    - Убедитесь, что доступ к сервисам настроен через указанные домены.
    - App: `http://your-subdomain-for-app-service.your-domain.ru`.
    - Prometheus: `http://your-subdomain-for-prometheus.your-domain.ru`.
    - Grafana: `http://your-subdomain-for-grafana.your-domain.ru`.



### Если падает проброс портов для Ingress Controller можно настроить systemd-службу:

1. **Создайте директорию для конфигурационного файла `kubectl`:**

    ```sudo mkdir -p ~/.kube```

2. **Сохраните текущую конфигурацию `kubectl` от Minikube:** ### НЕ НУЖНО

    ```sudo minikube kubectl -- config view --flatten | sudo tee ~/.kube/config > /dev/null```


3. **Проверьте содержимое файла конфигурации `kubectl`:**

    ```sudo kubectl config view```

4. **Установите переменную окружения `KUBECONFIG`:**

    ```export KUBECONFIG=~/.kube/config```

5. **Проверьте подключение к нодам Kubernetes:**

    ```kubectl get nodes```

6. **Настройте и перезапустите службу `systemd` для проброса портов:**

    Откройте файл службы `k8s-port-forward.service` и убедитесь, что он настроен правильно:

      ```sudo nano /etc/systemd/system/k8s-port-forward.service```

    Пример содержимого службы:

      ```
      [Unit]
      Description=Kubernetes Port Forward for Ingress Controller
      After=network.target
      
      [Service]
      Environment="KUBECONFIG=/home/<username>/.kube/config"
      ExecStart=/usr/local/bin/kubectl port-forward service/ingress-nginx-controller -n ingress-nginx 5000:80
      Restart=always
      
      [Install]
      WantedBy=multi-user.target
      ```

    Убедитесь, что `<username>` заменён на ваше имя пользователя.

7. **Перезагрузите конфигурацию `systemd`, включите и запустите службу:**

    ```
    sudo systemctl daemon-reload
    sudo systemctl enable k8s-port-forward.service
    sudo systemctl start k8s-port-forward.service
    ```

8. **Проверьте статус службы:**

    ```sudo systemctl status k8s-port-forward.service```
