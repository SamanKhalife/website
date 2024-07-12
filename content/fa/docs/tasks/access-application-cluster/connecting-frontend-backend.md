---
title: اتصال یک فرانت‌اند به یک بک‌اند با استفاده از سرویس‌ها
content_type: tutorial
weight: 70
---

<!-- overview -->
این وظیفه نحوه ایجاد یک میکروسرویس فرانت‌اند و یک میکروسرویس بک‌اند را نشان می‌دهد. میکروسرویس بک‌اند یک خداحافظ ساده است. فرانت‌اند با استفاده از nginx و یک شیء Kubernetes {{< glossary_tooltip term_id="service" >}} به بک‌اند دسترسی پیدا می‌کند.

## {{% heading "objectives" %}}

* ایجاد و اجرای یک میکروسرویس بک‌اند نمونه به نام `hello` با استفاده از یک {{< glossary_tooltip term_id="deployment" >}} شیء.
* استفاده از یک شیء سرویس برای ارسال ترافیک به چندین نمونه از میکروسرویس بک‌اند.
* ایجاد و اجرای یک میکروسرویس فرانت‌اند به نام `nginx` با استفاده از یک Deployment شیء.
* پیکربندی میکروسرویس فرانت‌اند برای ارسال ترافیک به میکروسرویس بک‌اند.
* استفاده از یک شیء سرویس با نوع `LoadBalancer` برای ارائه میکروسرویس فرانت‌اند به بیرون از کلاستر.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

این وظیفه از [Services با بارگذاری خارجی](/docs/tasks/access-application-cluster/create-external-load-balancer/) استفاده می‌کند که نیاز به یک محیط پشتیبانی شده دارد. اگر محیط شما این را پشتیبانی نمی‌کند، می‌توانید از یک Service از نوع [NodePort](/docs/concepts/services-networking/service/#type-nodeport) استفاده کنید.

<!-- lessoncontent -->

## ایجاد بک‌اند با استفاده از یک Deployment

بک‌اند یک میکروسرویس ساده خداحافظ است. اینجا فایل پیکربندی برای Deployment بک‌اند است:

{{% code_sample file="service/access/backend-deployment.yaml" %}}

ایجاد Deployment بک‌اند:

```shell
kubectl apply -f https://k8s.io/examples/service/access/backend-deployment.yaml
```

مشاهده اطلاعات درباره Deployment بک‌اند:

```shell
kubectl describe deployment backend
```

خروجی شبیه به این است:

```
Name:                           backend
Namespace:                      default
CreationTimestamp:              Mon, 24 Oct 2016 14:21:02 -0700
Labels:                         app=hello
                                tier=backend
                                track=stable
Annotations:                    deployment.kubernetes.io/revision=1
Selector:                       app=hello,tier=backend,track=stable
Replicas:                       3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:                   RollingUpdate
MinReadySeconds:                0
RollingUpdateStrategy:          1 max unavailable, 1 max surge
Pod Template:
  Labels:       app=hello
                tier=backend
                track=stable
  Containers:
   hello:
    Image:              "gcr.io/google-samples/hello-go-gke:1.0"
    Port:               80/TCP
    Environment:        <none>
    Mounts:             <none>
  Volumes:              <none>
Conditions:
  Type          Status  Reason
  ----          ------  ------
  Available     True    MinimumReplicasAvailable
  Progressing   True    NewReplicaSetAvailable
OldReplicaSets:                 <none>
NewReplicaSet:                  hello-3621623197 (3/3 replicas created)
Events:
...
```

## ایجاد شیء سرویس `hello`

کلید برای ارسال درخواست‌ها از فرانت‌اند به بک‌اند، سرویس بک‌اند است. یک سرویس یک آدرس IP مداوم و نام DNS ماندگاری ایجاد می‌کند تا همیشه بتوان به میکروسرویس بک‌اند دسترسی پیدا کرد. سرویس از {{< glossary_tooltip text="selectors" term_id="selector" >}} برای پیدا کردن پاد‌ها استفاده می‌کند که ترافیک را به آن‌ها هدایت می‌کند.

ابتدا فایل پیکربندی سرویس را بررسی کنید:

{{% code_sample file="service/access/backend-service.yaml" %}}

در فایل پیکربندی، می‌بینید که سرویس با نام `hello` ترافیک را به پاد‌هایی هدایت می‌کند که دارای برچسب‌های `app: hello` و `tier: backend` هستند.

ایجاد سرویس بک‌اند:

```shell
kubectl apply -f https://k8s.io/examples/service/access/backend-service.yaml
```

در این مرحله، یک Deployment `backend` با اجرای سه نسخه از برنامه `hello` شما وجود دارد، و یک سرویس وجود دارد که می‌تواند ترافیک را به آن‌ها هدایت کند. با این حال، این سرویس هیچ‌گاه خارج از کلاستر قابل دسترسی یا قابل حل نیست.

## ایجاد فرانت‌اند

حال که بک‌اند خود را در حال اجرا دارید، می‌توانید یک فرانت‌اند ایجاد کنید که از خارج از کلاستر قابل دسترسی باشد و به بک‌اند با استفاده از ارسال درخواست‌ها به آن متصل شود.

فرانت‌اند درخواست‌ها را به پاد‌های کارگر بک‌اند ارسال می‌کند با استفاده از نام DNS که به سرویس بک‌اند داده شده است. نام DNS `hello` است که مقدار فیلد `name` در فایل پیکربندی `examples/service/access/backend-service.yaml` است.

پاد‌ها در Deployment فرانت‌اند تصویر nginx را اجرا می‌کنند که پیکربندی شده است برای ارسال درخواست‌ها به سرویس بک‌اند `hello`. اینجا فایل پیکربندی nginx است:

{{% code_sample file="service/access/frontend-nginx.conf" %}}

مشابه بک‌اند، فرانت‌اند یک Deployment و یک سرویس دارد. یک تفاوت مهم بین سرویس‌های بک‌اند و

 فرانت‌اند این است که پیکربندی سرویس فرانت‌اند دارای `type: LoadBalancer` است که به معنای استفاده از یک بارگذار خودکار تأمین شده توسط ارائه‌دهنده ابر شما است و از خارج از کلاستر قابل دسترسی خواهد بود.

{{% code_sample file="service/access/frontend-service.yaml" %}}

{{% code_sample file="service/access/frontend-deployment.yaml" %}}

ایجاد Deployment و سرویس فرانت‌اند:

```shell
kubectl apply -f https://k8s.io/examples/service/access/frontend-deployment.yaml
kubectl apply -f https://k8s.io/examples/service/access/frontend-service.yaml
```

خروجی تأیید می‌کند که هر دو منبع ایجاد شده‌اند:

```
deployment.apps/frontend created
service/frontend created
```

{{< note >}}
پیکربندی nginx در
[تصویر کانتینر](/examples/service/access/Dockerfile) نسخه گرفته شده است. روش بهتری برای انجام این کار استفاده از یک
[ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/)
است تا بتوانید پیکربندی را به راحتی تغییر دهید.
{{< /note >}}

## تعامل با سرویس فرانت‌اند

با ایجاد یک سرویس از نوع LoadBalancer، می‌توانید از این دستور استفاده کنید تا IP خارجی را پیدا کنید:

```shell
kubectl get service frontend --watch
```

این نمایش پیکربندی برای سرویس `frontend` را نمایش می‌دهد و تغییرات را پایش می‌کند. در ابتدا، IP خارجی به عنوان `<pending>` لیست می‌شود:

```
NAME       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)  AGE
frontend   LoadBalancer   10.51.252.116   <pending>     80/TCP   10s
```

همانطور که یک IP خارجی تأمین شده است، اما، پیکربندی بروزرسانی می‌شود تا شامل آی‌پی جدید زیر عنوان `EXTERNAL-IP` شود:

```
NAME       TYPE           CLUSTER-IP      EXTERNAL-IP        PORT(S)  AGE
frontend   LoadBalancer   10.51.252.116   XXX.XXX.XXX.XXX    80/TCP   1m
```

حالا می‌توانید از آن IP برای تعامل با سرویس `frontend` از خارج از کلاستر استفاده کنید.

## ارسال ترافیک از طریق فرانت‌اند

فرانت‌اند و بک‌اند در حال حاضر متصل شده‌اند. می‌توانید به نقطه پایانی آن ضربه بزنید با استفاده از دستور curl بر روی آی‌پی خارجی سرویس فرانت‌اند خود.

```shell
curl http://${EXTERNAL_IP} # جایگزین کردن این بخش با EXTERNAL-IP که پیش‌تر دیده‌اید
```

خروجی پیام تولید شده توسط بک‌اند را نمایش می‌دهد:

```json
{"message":"Hello"}
```

## {{% heading "cleanup" %}}

برای حذف سرویس‌ها، این دستور را وارد کنید:

```shell
kubectl delete services frontend backend
```

برای حذف Deployments، ReplicaSets و Pods که برنامه‌های بک‌اند و فرانت‌اند را اجرا می‌کنند، این دستور را وارد کنید:

```shell
kubectl delete deployment frontend backend
```

## {{% heading "whatsnext" %}}

* بیشتر در مورد [Services](/docs/concepts/services-networking/service/) بیاموزید
* بیشتر در مورد [ConfigMaps](/docs/tasks/configure-pod-container/configure-pod-configmap/) بیاموزید
* بیشتر در مورد [DNS برای سرویس و پادها](/docs/concepts/services-networking/dns-pod-service/) بیاموزید
