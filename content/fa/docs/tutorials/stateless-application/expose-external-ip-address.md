---
title: "نمایش یک آدرس IP خارجی برای دسترسی به یک برنامه در یک خوشه"
content_type: tutorial
weight: 10
---

<!-- مرور -->

این صفحه نحوه‌ی ایجاد یک شیء خدمت Kubernetes که یک آدرس IP خارجی را ارائه می‌دهد را نمایش می‌دهد.

## {{% heading "پیش‌نیازها" %}}

* نصب [kubectl](/docs/tasks/tools/).
* استفاده از یک ارائه‌دهنده ابری مانند Google Kubernetes Engine یا Amazon Web Services برای ایجاد یک خوشه Kubernetes. این آموزش یک [بارگذار بار خارجی](/docs/tasks/access-application-cluster/create-external-load-balancer/) ایجاد می‌کند که نیازمند ارائه‌دهنده ابری است.
* پیکربندی `kubectl` برای ارتباط با سرور API Kubernetes خود. برای دستورالعمل‌ها، به مستندات ارائه‌دهنده ابری خود مراجعه کنید.

## {{% heading "اهداف" %}}

* اجرای پنج نمونه از یک برنامه سلام دنیا.
* ایجاد یک شیء خدمت که یک آدرس IP خارجی را ارائه می‌دهد.
* استفاده از شیء خدمت برای دسترسی به برنامه در حال اجرا.

<!-- محتوای درس -->

## ایجاد یک خدمت برای برنامه در حال اجرا در پنج پاد

1. اجرای یک برنامه سلام دنیا در خوشه‌ی خود:

   {{% code_sample file="service/load-balancer-example.yaml" %}}

   ```shell
   kubectl apply -f https://k8s.io/examples/service/load-balancer-example.yaml
   ```
   دستور فوق یک
   {{< glossary_tooltip text="Deployment" term_id="deployment" >}}
   و یک
   {{< glossary_tooltip term_id="replica-set" text="ReplicaSet" >}}
   مرتبط را ایجاد می‌کند. ReplicaSet دارای پنج
   {{< glossary_tooltip text="Pods" term_id="pod" >}}
   است که هر کدام از آن‌ها برنامه سلام دنیا را اجرا می‌کنند.

1. نمایش اطلاعات درباره Deployment:

   ```shell
   kubectl get deployments hello-world
   kubectl describe deployments hello-world
   ```

1. نمایش اطلاعات درباره اشیاء ReplicaSet شما:

   ```shell
   kubectl get replicasets
   kubectl describe replicasets
   ```

1. ایجاد یک شیء خدمت که Deployment را ارائه می‌دهد:

   ```shell
   kubectl expose deployment hello-world --type=LoadBalancer --name=my-service
   ```

1. نمایش اطلاعات درباره خدمت:

   ```shell
   kubectl get services my-service
   ```

   خروجی مشابه این خواهد بود:

   ```console
   NAME         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)    AGE
   my-service   LoadBalancer   10.3.245.137   104.198.205.71   8080/TCP   54s
   ```

   {{< note >}}

   خدمت با `type=LoadBalancer` توسط ارائه‌دهندگان ابری خارجی پشتیبانی می‌شود که در این مثال تحت پوشش قرار نمی‌گیرد. لطفاً به [این صفحه](/docs/concepts/services-networking/service/#loadbalancer) برای جزئیات بیشتر مراجعه کنید.

   {{< /note >}}

   {{< note >}}

   اگر آدرس IP خارجی به عنوان `<pending>` نشان داده شود، لطفاً یک دقیقه صبر کنید و دوباره همان دستور را وارد کنید.

   {{< /note >}}

1. نمایش اطلاعات تفصیلی درباره خدمت:

   ```shell
   kubectl describe services my-service
   ```

   خروجی مشابه این خواهد بود:

   ```console
   Name:           my-service
   Namespace:      default
   Labels:         app.kubernetes.io/name=load-balancer-example
   Annotations:    <none>
   Selector:       app.kubernetes.io/name=load-balancer-example
   Type:           LoadBalancer
   IP:             10.3.245.137
   LoadBalancer Ingress:   104.198.205.71
   Port:           <unset> 8080/TCP
   NodePort:       <unset> 32377/TCP
   Endpoints:      10.0.0.6:8080,10.0.1.6:8080,10.0.1.7:8080 + 2 more...
   Session Affinity:   None
   Events:         <none>
   ```

   لطفاً آدرس IP خارجی (`LoadBalancer Ingress`) که توسط خدمت شما ارائه شده را یادداشت کنید. در این مثال، آدرس IP خارجی 104.198.205.71 است. همچنین مقدار `Port` و `NodePort` را نیز یادداشت کنید. در این مثال، `Port` 8080 و `NodePort` 32377 است.

1. در خروجی قبلی، مشاهده می‌کنید که خدمت دارای چندین نقطه پایانی (endpoint) است: 10.0.0.6:8080, 10.0.1.6:8080, 10.0.1.7:8080 + 2 بیشتر. این آدرس‌های داخلی پادهایی هستند که برنامه سلام دنیا را اجرا می‌کنند. برای تأیید اینکه این آدرس‌ها پادها هستند، این دستور را وارد کنید:

   ```shell
   kubectl get pods --output=wide
   ```

   خروجی مشابه این خواهد بود:

   ```console
   NAME                         ...  IP         NODE
   hello-world-2895499144-1jaz9 ...  10.0.1.6   gke-cluster-1-default-pool-e0b8d269-1afc
   hello-world-2895499144-2e5uh ...  10.0.1.8   gke-cluster-1-default-pool-e0

b8d269-1afc
   hello-world-2895499144-9m4h1 ...  10.0.0.6   gke-cluster-1-default-pool-e0b8d269-5v7a
   hello-world-2895499144-o4z13 ...  10.0.1.7   gke-cluster-1-default-pool-e0b8d269-1afc
   hello-world-2895499144-segjf ...  10.0.2.5   gke-cluster-1-default-pool-e0b8d269-cpuc
   ```

1. استفاده از آدرس IP خارجی (`LoadBalancer Ingress`) برای دسترسی به برنامه سلام دنیا:

   ```shell
   curl http://<external-ip>:<port>
   ```

   جایی که `<external-ip>` آدرس IP خارجی (`LoadBalancer Ingress`) خدمت شما است و `<port>` مقدار `Port` در توصیف خدمت شما است. اگر از minikube استفاده می‌کنید، تایپ کردن `minikube service my-service` به طور خودکار برنامه سلام دنیا را در مرورگر باز می‌کند.

   پاسخ به درخواست موفق یک پیام سلام خواهد بود:

   ```shell
   Hello, world!
   Version: 2.0.0
   Hostname: 0bd46b45f32f
   ```

## {{% heading "پاکسازی" %}}

برای حذف خدمت، این دستور را وارد کنید:

```shell
kubectl delete services my-service
```

برای حذف Deployment، ReplicaSet و Pods که برنامه سلام دنیا را اجرا می‌کنند، این دستور را وارد کنید:

```shell
kubectl delete deployment hello-world
```

## {{% heading "مرحله‌بعد" %}}

بیشتر در مورد
[اتصال برنامه‌ها با خدمات](/docs/tutorials/services/connect-applications-service/)
بیاموزید.
```