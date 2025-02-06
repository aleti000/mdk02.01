# Лекция 25: Настройка Kubernetes

## Цели и задачи
- Изучить основные принципы работы Kubernetes (K8s) как системы оркестрации контейнеров.
- Разобраться с архитектурой Kubernetes и её компонентами (Master, Node, Pod, Service).
- Научиться устанавливать и настраивать Kubernetes для управления контейнеризированными приложениями.
- Понять процесс развертывания, масштабирования и управления приложениями в Kubernetes.
- Изучить диагностику и устранение неисправностей в работе Kubernetes.

---

## Основной материал

### 1. Введение в Kubernetes

#### Что такое Kubernetes?
Kubernetes (K8s) — это платформа с открытым исходным кодом для автоматизации развертывания, масштабирования и управления контейнеризированными приложениями. Она предоставляет:
- Оркестрацию контейнеров (например, Docker).
- Балансировку нагрузки и маршрутизацию трафика.
- Автоматическое восстановление и масштабирование.

#### Зачем нужен Kubernetes?
- Упрощение управления контейнеризированными приложениями.
- Высокая отказоустойчивость за счет репликации.
- Масштабируемость: возможность добавлять новые узлы и контейнеры без простоя.

#### Основные компоненты:
1. Master: Управляет кластером (API Server, Scheduler, Controller Manager).
2. Node: Рабочие машины, на которых запускаются контейнеры.
3. Pod: Минимальная единица развертывания в Kubernetes.
4. Service: Обеспечивает доступ к группе подов.
5. Deployment: Управляет состоянием приложений.

---

### 2. Установка Kubernetes

#### Используемые инструменты
Для установки Kubernetes используется kubeadm. Например, в Ubuntu:

1. Обновите список пакетов:
   apt update
2. Установите необходимые зависимости:
   apt install -y docker.io kubelet kubeadm kubectl
3. Включите и запустите Docker:
   systemctl enable docker
   systemctl start docker
4. Отключите swap:
   swapoff -a

#### Инициализация Master-узла
Инициализируйте кластер на Master-узле:
kubeadm init --pod-network-cidr=10.244.0.0/16

Настройте конфигурацию для пользователя:
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

#### Установка сетевого плагина
Установите сетевой плагин (например, Flannel):
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

#### Присоединение Worker-узлов
Присоедините Worker-узлы к кластеру:
kubeadm join <master-ip>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>

---

### 3. Развертывание приложений

#### Создание Deployment
Создайте Deployment для приложения:
kubectl create deployment myapp --image=nginx

#### Просмотр статуса
Проверьте статус Deployment:
kubectl get deployments

#### Масштабирование
Масштабируйте Deployment:
kubectl scale deployment myapp --replicas=3

---

### 4. Настройка Service

#### Создание Service
Создайте Service для доступа к приложению:
kubectl expose deployment myapp --type=NodePort --port=80

#### Просмотр Service
Проверьте статус Service:
kubectl get services

#### Доступ к приложению
Получите IP и порт для доступа:
kubectl get nodes -o wide
curl http://<node-ip>:<node-port>

---

### 5. Управление конфигурацией

#### Создание ConfigMap
Создайте ConfigMap для хранения конфигурации:
kubectl create configmap myconfig --from-literal=key=value

#### Использование ConfigMap
Добавьте ConfigMap в Deployment:
env:
- name: MY_KEY
  valueFrom:
    configMapKeyRef:
      name: myconfig
      key: key

---

### 6. Диагностика и устранение неисправностей

#### Просмотр логов
Просмотрите логи Pod:
kubectl logs <pod-name>

#### Вход в Pod
Выполните команду внутри Pod:
kubectl exec -it <pod-name> -- /bin/bash

#### Распространенные проблемы:
- Недостаточное количество ресурсов на узлах.
- Ошибки сетевой конфигурации.
- Проблемы с образами контейнеров.

---

### 7. Безопасность Kubernetes

#### Настройка RBAC
Настройте роли и привязки для ограничения доступа:
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

#### Регулярное обновление
Регулярно обновляйте Kubernetes для защиты от уязвимостей.

---

### 8. Пример настройки Kubernetes

#### Шаги:
1. Установите Kubernetes:
   apt install -y docker.io kubelet kubeadm kubectl
2. Инициализируйте Master-узел:
   kubeadm init --pod-network-cidr=10.244.0.0/16
3. Установите сетевой плагин:
   kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
4. Присоедините Worker-узлы:
   kubeadm join <master-ip>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
5. Разверните приложение:
   kubectl create deployment myapp --image=nginx
6. Создайте Service:
   kubectl expose deployment myapp --type=NodePort --port=80

---

## Домашнее задание
1. Установите и настройте Kubernetes на трех виртуальных машинах.
2. Разверните приложение (например, Nginx) и проверьте его работу.
3. Масштабируйте приложение до трех реплик.
4. Настройте Service для доступа к приложению.
5. Изучите логи и найдите записи о работе Kubernetes.

---

## Вопросы для самопроверки
1. Что такое Kubernetes и для чего он используется?
2. Как установить и настроить Kubernetes?
3. Как развернуть и масштабировать приложение в Kubernetes?
4. Как настроить Service для доступа к приложению?
5. Какие меры безопасности можно применить для защиты Kubernetes?
