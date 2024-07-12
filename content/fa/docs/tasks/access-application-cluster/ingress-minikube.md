---
title: راه‌اندازی Ingress بر روی Minikube با کنترل‌کننده NGINX Ingress
content_type: task
weight: 110
min-kubernetes-server-version: 1.19
---

<!-- overview -->

[Ingress](/docs/concepts/services-networking/ingress/) یک شی API است که قوانینی را تعریف می‌کند که به دسترسی خارجی به خدمات در یک کلاستر اجازه می‌دهد. یک [کنترل‌کننده Ingress](/docs/concepts/services-networking/ingress-controllers/) قوانین تعیین شده در Ingress را اجرا می‌کند.

این صفحه به شما نحوه راه‌اندازی یک Ingress ساده را نشان می‌دهد که درخواست‌ها را به سرویس‌های 'web' یا 'web2' بسته به URI HTTP هدایت می‌کند.

## {{% heading "prerequisites" %}}

برای این آموزش فرض می‌شود که از `minikube` برای اجرای یک کلاستر Kubernetes محلی استفاده می‌کنید.
برای آموزش نحوه نصب `minikube` به [نصب ابزارها](/docs/tasks/tools/#minikube) مراجعه کنید.

{{< note >}}
این آموزش از یک کانتینر استفاده می‌کند که نیازمند معماری AMD64 است. اگر از `minikube` بر روی یک کامپیوتر با معماری CPU متفاوت استفاده می‌کنید، می‌توانید از یک درایور استفاده کنید که بتواند AMD64 را شبیه‌سازی کند. به عنوان مثال، درایور Docker Desktop این کار را انجام می‌دهد.
{{< /note >}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}
اگر از یک نسخه قدیمی‌تر از Kubernetes استفاده می‌کنید، به مستندات آن نسخه مراجعه کنید.

### ایجاد یک کلاستر minikube

اگر هنوز یک کلاستر را به صورت محلی راه‌اندازی نکرده‌اید، دستور `minikube start` را اجرا کنید تا یک کلاستر ایجاد شود.

<!-- steps -->

## فعال‌سازی کنترل‌کننده Ingress

1. برای فعال‌سازی کنترل‌کننده NGINX Ingress، دستور زیر را اجرا کنید:

   ```shell
   minikube addons enable ingress
   ```

1. تأیید کنید که کنترل‌کننده NGINX Ingress در حال اجرا است

   ```shell
   kubectl get pods -n ingress-nginx
   ```

   {{< note >}}
   ممکن است تا یک دقیقه زمان ببرد تا شما این پادها را به حالت اجرا ببینید.
   {{< /note >}}

   خروجی مشابه زیر خواهد بود:

   ```none
   NAME                                        READY   STATUS      RESTARTS    AGE
   ingress-nginx-admission-create-g9g49        0/1     Completed   0          11m
   ingress-nginx-admission-patch-rqp78         0/1     Completed   1          11m
   ingress-nginx-controller-59b45fb494-26npt   1/1     Running     0          11m
   ```

## استقرار برنامه hello, world

1. با استفاده از دستور زیر، یک Deployment ایجاد کنید:

   ```shell
   kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
   ```

   خروجی باید این باشد:

   ```none
   deployment.apps/web created
   ```

   تأیید کنید که Deployment در حالت آماده (Ready) است:

   ```shell
   kubectl get deployment web
   ```

   خروجی مشابه این خواهد بود:

   ```none
   NAME   READY   UP-TO-DATE   AVAILABLE   AGE
   web    1/1     1            1           53s
   ```



1. سرویس را ارائه دهید:

   ```shell
   kubectl expose deployment web --type=NodePort --port=8080
   ```

   خروجی باید این باشد:

   ```none
   service/web exposed
   ```

1. تأیید کنید که سرویس ایجاد شده است و بر روی یک پورت نود در دسترس است:

   ```shell
   kubectl get service web
   ```

   خروجی مشابه زیر خواهد بود:

   ```none
   NAME      TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
   web       NodePort   10.104.133.249   <none>        8080:31637/TCP   12m
   ```

1. با استفاده از `minikube service`، سرویس را از طریق NodePort بازدید کنید. دستورات مربوط به پلتفرم خود را دنبال کنید:

   {{< tabs name="minikube_service" >}}
   {{% tab name="Linux" %}}

   ```shell
   minikube service web --url
   ```
   خروجی مشابه این خواهد بود:
   ```none
   http://172.17.0.15:31637
   ```
   URL به دست آمده را در دستور قبلی اجرا کنید:
   ```shell
   curl http://172.17.0.15:31637
   ```
   {{% /tab %}}
   {{% tab name="MacOS" %}}
   ```shell
   # دستور باید در یک ترمینال جداگانه اجرا شود.
   minikube service web --url
   ```
   خروجی مشابه این خواهد بود:
   ```none
   http://127.0.0.1:62445
   ! از آنجا که شما از درایور Docker در darwin استفاده می کنید، ترمینال باید باز بماند تا اجرا شود.
   ```
   از یک ترمینال دیگر، URL به دست آمده را در دستور قبلی اجرا کنید:
   ```shell
   curl http://127.0.0.1:62445
   ```
   {{% /tab %}}
   {{< /tabs >}}
   <br>
   خروجی مشابه زیر خواهد بود:

   ```none
   Hello, world!
   Version: 1.0.0
   Hostname: web-55b8c6998d-8k564
   ```

   اکنون می‌توانید به برنامه نمونه از طریق آدرس IP Minikube و NodePort دسترسی پیدا کنید. مرحله بعد به شما این امکان ر

ا می‌دهد تا به برنامه با استفاده از منبع Ingress دسترسی پیدا کنید.

## ایجاد یک Ingress

مانیفست زیر یک Ingress را تعریف می‌کند که ترافیک را از طریق `hello-world.example` به سرویس شما می‌فرستد.

1. فایل `example-ingress.yaml` را از فایل زیر ایجاد کنید:

   {{% code_sample file="service/networking/example-ingress.yaml" %}}

1. با اجرای دستور زیر، شی Ingress را ایجاد کنید:

   ```shell
   kubectl apply -f https://k8s.io/examples/service/networking/example-ingress.yaml
   ```

   خروجی باید مشابه زیر باشد:

   ```none
   ingress.networking.k8s.io/example-ingress created
   ```

1. تأیید کنید که آدرس IP تنظیم شده است:

   ```shell
   kubectl get ingress
   ```

   {{< note >}}
   این ممکن است چند دقیقه زمان ببرد.
   {{< /note >}}

   باید یک آدرس IPv4 در ستون `ADDRESS` را ببینید؛ به عنوان مثال:

   ```none
   NAME              CLASS   HOSTS                 ADDRESS        PORTS   AGE
   example-ingress   nginx   hello-world.example   172.17.0.15    80      38s
   ```

1. تأیید کنید که کنترل‌کننده Ingress ترافیک را هدایت می‌کند، با دنبال کردن دستورات مربوط به پلتفرم شما:

   {{< note >}}
   شبکه محدود است اگر از درایور Docker بر روی MacOS (Darwin) استفاده می‌کنید و IP نود به طور مستقیم قابل دسترس نیست. برای کار کردن Ingress، باید یک ترمینال جدید باز کرده و `minikube tunnel` را اجرا کنید.
   برای این کار نیاز به دسترسی `sudo` دارید، پس رمز عبور را وارد کنید و کلید آن را بزنید.
   {{< /note >}}

   {{< tabs name="ingress" >}}
   {{% tab name="Linux" %}}

   ```shell
   curl --resolve "hello-world.example:80:$( minikube ip )" -i http://hello-world.example
   ```
   {{% /tab %}}
   {{% tab name="MacOS" %}}

   ```shell
   minikube tunnel
   ```
   خروجی مشابه این خواهد بود:

   ```none
   Tunnel successfully started

   NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

   The service/ingress example-ingress requires privileged ports to be exposed: [80 443]
   sudo permission will be asked for it.
   Starting tunnel for service example-ingress.
   ```

   از یک ترمینال دیگر، دستور زیر را اجرا کنید:
   ```shell
   curl --resolve "hello-world.example:80:127.0.0.1" -i http://hello-world.example
   ```

   {{% /tab %}}
   {{< /tabs >}}
   <br>
   باید مشاهده کنید:

   ```none
   Hello, world!
   Version: 1.0.0
   Hostname: web-55b8c6998d-8k564
   ```

1. اختیاری، شما می‌توانید همچنین از مرورگر خود به `hello-world.example` مراجعه کنید.

   یک خط را به پایان `/etc/hosts` روی
     کامپیوتر خود اضافه کنید (شما نیاز به دسترسی مدیریتی خواهید داشت):

     {{< tabs name="hosts" >}}
     {{% tab name="Linux" %}}
   آدرس IP خارجی را که توسط minikube گزارش شده را بررسی کنید
   ```none
     minikube ip
   ```
   <br>

   ```none
     172.17.0.15 hello-world.example
   ```

   {{< note >}}
   آدرس IP را برای مطابقت با خروجی `minikube ip` تغییر دهید.
   {{< /note >}}
   {{% /tab %}}
     {{% tab name="MacOS" %}}
   ```none
   127.0.0.1 hello-world.example
   ```
     {{% /tab %}}
     {{< /tabs >}}

     <br>

     پس از انجام این تغییر، مرورگر وب شما درخواست‌ها را به `hello-world.example` به Minikube می‌فرستد.

## ایجاد یک Deployment دوم

1. با استفاده از دستور زیر، یک Deployment دیگر ایجاد کنید:

   ```shell
   kubectl create deployment web2 --image=gcr.io/google-samples/hello-app:2.0
   ```

   خروجی باید این باشد:

   ```none
   deployment.apps/web2 created
   ```
   تأیید کنید که Deployment در حالت آماده (Ready) است:

   ```shell
   kubectl get deployment web2
   ```

   خروجی مشابه این خواهد بود:

   ```none
   NAME   READY   UP-TO-DATE   AVAILABLE   AGE
   web2   1/1     1            1           16s
   ```

1. سرویس دوم را ارائه دهید:

   ```shell
   kubectl expose deployment web2 --port=8080 --type=NodePort
   ```

   خروجی باید این باشد:

   ```none
   service/web2 exposed
   ```

## ویرایش Ingress موجود {#edit-ingress}

1. مانیفست `example-ingress.yaml` موجود را ویرایش کرده و خطوط زیر را به انتهای آن اضافه کنید:

    ```yaml
    - path: /v2
      pathType: Prefix
      backend:
        service:
          name: web2
          port:
            number: 8080
    ```

1. تغییرات را اعمال کنید:

   ```shell
   kubectl apply -f example-ingress.yaml
   ```

   باید مشاهده کنید:

   ```none
   ingress.networking/example-ingress configured
   ```

## تست Ingress شما

1. دسترسی به نسخه اول برنامه Hello World را بررسی کنید.

   {{< tabs name="ingress2-v1" >}}
   {{% tab name="Linux" %}}

   ```shell
   curl --resolve "hello-world.example:80:$( minikube ip )" -i http://hello-world.example
   ```
   {{% /tab %}}
   {{% tab name="MacOS" %}}

   ```shell
   minikube tunnel
   ```
   خروجی مشابه این خواهد بود:

   ```none
   Tunnel successfully started

   NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

   The service/ingress example-ingress requires privileged ports to be exposed: [80 443]
   sudo permission will be asked for it.
   Starting tunnel for service example-ingress.
   ```

   از یک ترمینال جدید، دستور زیر را اجرا کنید:
   ```shell
   curl --resolve "hello-world.example:80:127.0.0.1" -i http://hello-world.example
   ```

   {{% /tab %}}
   {{< /tabs >}}
   <br>

   خروجی مشابه این خواهد بود:

   ```none
   Hello, world!
   Version: 1.0.0
   Hostname: web-55b8c6998d-8k564
   ```

1. دسترسی به نسخه دوم برنامه Hello World را بررسی کنید.

   {{< tabs name="ingress2-v2" >}}
   {{% tab name="Linux" %}}

   ```shell
   curl --resolve "hello-world.example:80:$( minikube ip )" -i http://hello-world.example/v2
   ```
   {{% /tab %}}
   {{% tab name="MacOS" %}}

   ```shell
   minikube tunnel
   ```
   خروجی مشابه این خواهد بود:

   ```none
   Tunnel successfully started

   NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

   The service/ingress example-ingress requires privileged ports to be exposed: [80 443]
   sudo permission will be asked for it.
   Starting tunnel for service example-ingress.
   ```

   از یک ترمینال جدید، دستور زیر را اجرا کنید:
   ```shell
   curl --resolve "hello-world.example:80:127.0.0.1" -i http://hello-world.example/v2
   ```

   {{% /tab %}}
   {{< /tabs >}}

   خروجی مشابه این خواهد بود:

   ```none
   Hello, world!
   Version: 2.0.0
   Hostname: web2-75cd47646f-t8cjk
   ```

   {{< note >}}
   اگر مرحله اختیاری را برای به‌روزرسانی `/etc/hosts` انجام داده‌اید، می‌توانید همچنین از مرورگر خود به `hello-world.example` و `hello-world.example/v2` مراجعه کنید.
   {{< /note >}}

## {{% heading "whatsnext" %}}

* بیشتر بخوانید درباره [Ingress](/docs/concepts/services-networking/ingress/)
* بیشتر بخوانید درباره [Ingress Controllers](/docs/concepts/services-networking/ingress-controllers/)
* بیشتر بخوانید درباره [Services](/docs/concepts/services-networking/service/)
