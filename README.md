
---
title: Как настроить KEDA {{ k8s }} в {{ managed-k8s-full-name }}
description: Следуя данной инструкции, вы сможете настроить автомасштабирование.
---
KEDA (Kubernetes Event-Driven Autoscaling) — это оператор для Kubernetes, который позволяет автоматически масштабировать приложения на основе событий и метрик, таких как использование CPU, сообщений в очередях, базы данных и других внешних источников данных. KEDA предоставляет гибкость для масштабирования контейнеров как вверх, так и вниз в зависимости от текущей нагрузки, что особенно полезно для динамических и событийно-ориентированных приложений.
# Настройка KEDA


## Перед началом работы {#before-you-begin}

1. [Создайте кластер {{ managed-k8s-name }}](kubernetes-cluster/kubernetes-cluster-create.md) любой подходящей конфигурации.

1. {% include [Install and configure kubectl](../../_includes/managed-kubernetes/kubectl-install.md) %}

## Установка через Helm

{% note warning %}

Для работы KEDA требуется кластер Kubernetes версии 1.27 и выше.

{% endnote %}


- CLI {#cli}

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  1. Добавить репозиторий Helm

     ```bash
     helm repo add kedacore https://kedacore.github.io/charts
     ```

  1. Обновить резозиторий Helm:

     ```bash
     helm repo update
     ```
  1. Обновить резозиторий Helm:

     ```bash
     helm install keda kedacore/keda --namespace keda --create-namespace
     ```



## Запуск автомастабирования тестового приложения с помощью KEDA 

{% list tabs group=instructions %}

- CLI {#cli}

  1. Запустите тестовое приложение на apache: 

     ```bash
     kubectl apply -f https://k8s.io/examples/application/php-apache.yaml
     ```

  1. Проверьте состояние сервиса:

     ```bash
     kubectl get po     ```
  1. Создайте  ScaledObject для автомаштабирования

     ```yaml
     apiVersion: keda.sh/v1alpha1
     kind: ScaledObject
     metadata:
	     name: php-apache-scaledobject
	     namespace: default
	 spec:
		 scaleTargetRef:
			 apiVersion: apps/v1
			 kind: Deployment
			 name: php-apache
		 minReplicaCount: 1
		 maxReplicaCount: 10
		 triggers:
		 - type: cpu
		    metadata:
			    type: Utilization
			    value: "50"          
     ```
     
  1. Запустите для вашего приложения:

     ```bash
     kubectl apply -f scaledobject.yaml
     ```
   1. Проверьте состояние  ScaledObject:

     ```bash
     kubectl get scaledobject
     ```

## Создать нагрузку на веб-сервер запущенного контейнера


- CLI {#cli}

  1. Установите {{ k8s-vpa }} из [репозитория](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler):

     ```bash
     kubectl run -i \
      --tty load-generator \
      --rm --image=busybox \
      --restart=Never \
      -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"


  1. Проверьте состояние подов:

     ```bash
     kubectl get hpa
     ```

