---
title: استفاده از یک سرویس برای دسترسی به برنامه در یک خوشه
content_type: آموزش
weight: 60
---

<!-- overview -->

این صفحه نحوه ایجاد یک شی Kubernetes Service را نشان می‌دهد که مشتریان خارجی می‌توانند از آن استفاده کنند تا به یک برنامه در حال اجرا در یک خوشه دسترسی پیدا کنند. این سرویس برای یک برنامه که دو نمونه از آن در حال اجرا هستند، توازن بار فراهم می‌کند.

## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}}

## {{% heading "اهداف" %}}

- اجرای دو نمونه از برنامه Hello World.
- ایجاد یک شی Service که یک پورت node را ارائه می‌دهد.
- استفاده از شی Service برای دسترسی به برنامه در حال اجرا.

<!-- lessoncontent -->

## ایجاد یک سرویس برای برنامه در حال اجرا در دو Pod

اینجا فایل پیکربندی برای Deployment برنامه است:

{{% code_sample file="service/access/hello-application.yaml" %}}

1. اجرای برنامه Hello World در خوشه‌ی خود:
   برنامه Deployment را با استفاده از فایل فوق ایجاد کنید:

   ```shell
   kubectl apply -f https://k8s.io/examples/service/access/hello-application.yaml
   ```

   دستور فوق یک
   {{< glossary_tooltip text="Deployment" term_id="deployment" >}}
   و یک
   {{< glossary_tooltip term_id="replica-set" text="ReplicaSet" >}}
   مرتبط را ایجاد می‌کند. ReplicaSet دارای دو
   {{< glossary_tooltip text="Pod" term_id="pod" >}}
   است که هرکدام از آن‌ها برنامه Hello World را اجرا می‌کنند.

1. نمایش اطلاعات درباره‌ی Deployment:

   ```shell
   kubectl get deployments hello-world
   kubectl describe deployments hello-world
   ```

1. نمایش اطلاعات درباره‌ی اشیاء ReplicaSet شما:

   ```shell
   kubectl get replicasets
   kubectl describe replicasets
   ```

1. ایجاد یک شی Service که Deployment را ارائه می‌دهد:

   ```shell
   kubectl expose deployment hello-world --type=NodePort --name=example-service
   ```

1. نمایش اطلاعات درباره‌ی Service:

   ```shell
   kubectl describe services example-service
   ```

   خروجی مشابه زیر است:

   ```none
   Name:                   example-service
   Namespace:              default
   Labels:                 run=load-balancer-example
   Annotations:            <none>
   Selector:               run=load-balancer-example
   Type:                   NodePort
   IP:                     10.32.0.16
   Port:                   <unset> 8080/TCP
   TargetPort:             8080/TCP
   NodePort:               <unset> 31496/TCP
   Endpoints:              10.200.1.4:8080,10.200.2.5:8080
   Session Affinity:       None
   Events:                 <none>
   ```

   NodePort مقداری که برای Service استفاده می‌شود را یادداشت کنید. به عنوان مثال، در خروجی فوق، مقدار NodePort 31496 است.

1. لیست کردن پادهایی که برنامه Hello World را اجرا می‌کنند:

   ```shell
   kubectl get pods --selector="run=load-balancer-example" --output=wide
   ```

   خروجی مشابه زیر است:

   ```none
   NAME                           READY   STATUS    ...  IP           NODE
   hello-world-2895499144-bsbk5   1/1     Running   ...  10.200.1.4   worker1
   hello-world-2895499144-m1pwt   1/1     Running   ...  10.200.2.5   worker2
   ```

1. آدرس IP عمومی یکی از نودهای خود را که یک پاد Hello World را اجرا می‌کند بگیرید. راه گرفتن این آدرس بستگی به نحوه‌ی راه‌اندازی خوشه شما دارد. به عنوان مثال، اگر از Minikube استفاده می‌کنید، می‌توانید آدرس نود را با اجرای دستور `kubectl cluster-info` ببینید. اگر از نمونه‌های Google Compute Engine استفاده می‌کنید، می‌توانید از دستور `gcloud compute instances list` برای دیدن آدرس‌های عمومی نودهای خود استفاده کنید.

1. بر روی نود انتخاب شده، قانون دیوار آتش را ایجاد کنید که امکان ترافیک TCP را بر روی پورت node خود اجازه می‌دهد. به عنوان مثال، اگر Service شما دارای مقدار NodePort 31568 است، یک قانون دیوار آتش ایجاد کنید که امکان ترافیک TCP را بر روی پورت 31568 فراهم می‌کند. ارائه‌دهندگان مختلف ابر روش‌های مختلفی برای پیکربندی قوانین دیوار آتش ارائه می‌دهند.

1. از آدرس IP نود و پورت node برای دسترسی به برنامه Hello World استفاده کنید:

   ```shell
   curl http://<آدرس-عمومی-نود>:<پورت-node>
   ```

   که `<آدرس-عمومی-نود>` آدرس IP عمومی نود شما است و `<پورت-node>` مقدار NodePort برای سرویس شما است. پاسخ به یک درخواست موفق، پیام hello را نمایش می‌دهد:

   ```none
   Hello, world!
   Version: 2.0.0
   Hostname: hello-world-cdd4458f4-m47c8
   ```

## استفاده از یک فایل پیکربندی سرویس

به عنوان جایگزین استفاده از `kubectl expose`، می‌توانید از یک [فایل پیکربندی سرویس](/docs/concepts/services-networking/service/) برای ایجاد یک سرویس استفاده کنید.

## {{% heading "پاکسازی" %}}

برای حذف Service، این دستور را وارد کنید:

    kubectl delete services

 example-service

برای حذف Deployment، ReplicaSet و Pods که برنامه Hello World را اجرا می‌کنند، این دستور را وارد کنید:

    kubectl delete deployment hello-world

## {{% heading "مراحل بعدی" %}}

مراحل راه‌اندازی اتصال برنامه‌ها با سرویس‌ها را در
[راهنمای اتصال برنامه‌ها با سرویس‌ها](/docs/tutorials/services/connect-applications-service/)
دنبال کنید.
