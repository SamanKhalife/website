---
title: اجرای برنامه بی‌حالت با استفاده از Deployment
min-kubernetes-server-version: v1.9
content_type: آموزش
weight: 10
---

<!-- مرور -->

در این صفحه نحوه اجرای یک برنامه با استفاده از اشیاء Deployment در Kubernetes نمایش داده شده است.

## {{% heading "اهداف" %}}

- ایجاد یک deployment برای nginx.
- استفاده از دستور kubectl برای نمایش اطلاعات درباره deployment.
- به‌روزرسانی deployment.

## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!-- محتوای درس -->

## ایجاد و بررسی یک deployment برای nginx

می‌توانید یک برنامه را با ایجاد یک شیء Deployment در Kubernetes اجرا کنید، و می‌توانید یک Deployment را در یک فایل YAML توصیف کنید. به عنوان مثال، این فایل YAML یک Deployment را که تصویر Docker nginx:1.14.2 را اجرا می‌کند، توصیف می‌کند:

{{% code_sample file="application/deployment.yaml" %}}

1. ایجاد یک Deployment بر اساس فایل YAML:

   ```shell
   kubectl apply -f https://k8s.io/examples/application/deployment.yaml
   ```

1. نمایش اطلاعات درباره Deployment:

   ```shell
   kubectl describe deployment nginx-deployment
   ```

   خروجی مشابه این است:

   ```
   Name:     nginx-deployment
   Namespace:    default
   CreationTimestamp:  Tue, 30 Aug 2016 18:11:37 -0700
   Labels:     app=nginx
   Annotations:    deployment.kubernetes.io/revision=1
   Selector:   app=nginx
   Replicas:   2 desired | 2 updated | 2 total | 2 available | 0 unavailable
   StrategyType:   RollingUpdate
   MinReadySeconds:  0
   RollingUpdateStrategy:  1 max unavailable, 1 max surge
   Pod Template:
     Labels:       app=nginx
     Containers:
       nginx:
       Image:              nginx:1.14.2
       Port:               80/TCP
       Environment:        <none>
       Mounts:             <none>
     Volumes:              <none>
   Conditions:
     Type          Status  Reason
     ----          ------  ------
     Available     True    MinimumReplicasAvailable
     Progressing   True    NewReplicaSetAvailable
   OldReplicaSets:   <none>
   NewReplicaSet:    nginx-deployment-1771418926 (2/2 replicas created)
   No events.
   ```

1. لیست کردن پادهای ایجاد شده توسط deployment:

   ```shell
   kubectl get pods -l app=nginx
   ```

   خروجی مشابه این است:

   ```
   NAME                                READY     STATUS    RESTARTS   AGE
   nginx-deployment-1771418926-7o5ns   1/1       Running   0          16h
   nginx-deployment-1771418926-r18az   1/1       Running   0          16h
   ```

1. نمایش اطلاعات درباره یک Pod:

   ```shell
   kubectl describe pod <نام-پاد>
   ```

   که `<نام-پاد>` نام یکی از پادهای شما است.

## به‌روزرسانی deployment

می‌توانید deployment را با اعمال یک فایل YAML جدید به‌روزرسانی کنید. این فایل YAML مشخص می‌کند که deployment باید به nginx نسخه 1.16.1 به‌روز شود.

{{% code_sample file="application/deployment-update.yaml" %}}

1. اعمال فایل YAML جدید:

   ```shell
   kubectl apply -f https://k8s.io/examples/application/deployment-update.yaml
   ```

1. مشاهده deployment که پادهای جدیدی را ایجاد کرده و پادهای قدیمی را حذف می‌کند:

   ```shell
   kubectl get pods -l app=nginx
   ```

## افزایش تعداد نمونه‌ها با افزایش تعداد پیش‌نمونه

می‌توانید تعداد پادها را در Deployment خود با اعمال یک فایل YAML جدید افزایش دهید. این فایل YAML مقدار `replicas` را به 4 تنظیم می‌کند که نشان می‌دهد deployment باید چهار پاد داشته باشد:

{{% code_sample file="application/deployment-scale.yaml" %}}

1. اعمال فایل YAML جدید:

   ```shell
   kubectl apply -f https://k8s.io/examples/application/deployment-scale.yaml
   ```

1. تأیید کنید که Deployment چهار پاد دارد:

   ```shell
   kubectl get pods -l app=nginx
   ```

   خروجی مشابه این است:

   ```
   NAME                               READY     STATUS    RESTARTS   AGE
   nginx-deployment-148880595-4zdqq   1/1       Running   0          25s
   nginx-deployment-148880595-6zgi1   1/1       Running   0          25s
   nginx-deployment-148880595-fxcez   1/1       Running   0          2m
   nginx-deployment-148880595-rwovn   1/1       Running   0          2m
   ```

## حذف یک deployment

با نام deployment را حذف کنید:

```shell
kubectl delete deployment nginx-deployment
```

## ReplicationControllers -- روش قدیمی

روش ترجیحی برای ایجاد یک برنامه تکثیر شده استفاده از یک Deployment است، که به نوبه خود از ReplicaSet استفاده می‌کند. قبل از اینکه Deployment و ReplicaSet به Kubernetes اضافه شوند، برنامه‌های تکثیر شده با استفاده از [ReplicationController](/docs/concepts/workloads/controllers/replicationcontroller/) پیکربندی می‌شدند.

## {{% heading "چی بعد" %}}

- بیشتر در مورد [اشیاء Deployment](/docs/concepts/workloads/controllers/deployment/) بیاموزید.
