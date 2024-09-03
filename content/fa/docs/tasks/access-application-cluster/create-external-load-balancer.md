---
title: ایجاد یک بارگذار خارجی
content_type: task
weight: 80
---

<!-- overview -->

این صفحه نحوه ایجاد یک بارگذار خارجی را نشان می‌دهد.

هنگام ایجاد یک {{< glossary_tooltip text="Service" term_id="service" >}}، شما
گزینه‌ای برای ایجاد خودکار یک بارگذار ابری دارید. این به شما
یک آدرس IP قابل دسترسی از بیرون ارائه می‌دهد که ترافیک را به پورت مناسب روی نودهای
کلاستر شما ارسال می‌کند
_اگر کلاستر شما در یک محیط پشتیبانی شده اجرا شده و با
بسته ارائه‌دهنده بارگذار ابری مناسب پیکربندی شده باشد_.

همچنین می‌توانید از {{< glossary_tooltip term_id="ingress" >}} به جای Service استفاده کنید.
برای اطلاعات بیشتر، به [مستندات Ingress](/docs/concepts/services-networking/ingress/)
مراجعه کنید.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}}

کلاستر شما باید در یک محیط ابری یا دیگر محیط‌هایی که از پیکربندی بارگذار خارجی
پشتیبانی می‌کنند، اجرا شود.


<!-- steps -->

## ایجاد یک Service

### ایجاد یک Service از یک manifest

برای ایجاد یک بارگذار خارجی، خط زیر را به manifest
Service خود اضافه کنید:

```yaml
    type: LoadBalancer
```

manifest شما ممکن است شامل این دستورات باشد:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-service
spec:
  selector:
    app: example
  ports:
    - port: 8765
      targetPort: 9376
  type: LoadBalancer
```

### ایجاد یک Service با استفاده از kubectl

می‌توانید به جای اینکه manifest را به صورت مستقیم ایجاد کنید، با استفاده از دستور
`kubectl expose` و پرچم `--type=LoadBalancer`، Service را ایجاد کنید:

```bash
kubectl expose deployment example --port=8765 --target-port=9376 \
        --name=example-service --type=LoadBalancer
```

این دستور یک Service جدید با استفاده از همان selectors که در منبع اشاره شده است
منبع (در این مورد یک
{{< glossary_tooltip text="Deployment" term_id="deployment" >}} به نام `example`).

برای اطلاعات بیشتر، از جمله پرچم‌های اختیاری، به
[مرجع `kubectl expose`](/docs/reference/generated/kubectl/kubectl-commands/#expose)
مراجعه کنید.

## پیدا کردن آدرس IP شما

می‌توانید آدرس IP ایجاد شده برای Service خود را با گرفتن اطلاعات Service
از طریق `kubectl` پیدا کنید:

```bash
kubectl describe services example-service
```

که خروجی مشابه زیر را تولید می‌کند:

```
Name:                     example-service
Namespace:                default
Labels:                   app=example
Annotations:              <none>
Selector:                 app=example
Type:                     LoadBalancer
IP Families:              <none>
IP:                       10.3.22.96
IPs:                      10.3.22.96
LoadBalancer Ingress:     192.0.2.89
Port:                     <unset>  8765/TCP
TargetPort:               9376/TCP
NodePort:                 <unset>  30593/TCP
Endpoints:                172.17.0.3:9376
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

آدرس IP بارگذار خود را در کنار `LoadBalancer Ingress` لیست شده است.

{{< note >}}
اگر سرویس خود را در Minikube اجرا می‌کنید، می‌توانید آدرس IP و پورت اختصاصی شده را با استفاده از
```bash
minikube service example-service --url
```
پیدا کنید.
{{< /note >}}

## حفظ IP منبع مشتری

به طور پیش‌فرض، IP منبع دیده شده در کانتینر هدف *IP اصلی منبع*
مشتری نیست. برای فعال سازی حفظ IP مشتری، می‌توانید فیلدهای زیر را در
`.spec` سرویس پیکربندی کنید:

* `.spec.externalTrafficPolicy` - نشان می‌دهد که آیا این Service می‌خواهد ترافیک
  خارجی را به نقاط محلی نود یا کلاستر ارسال کند. دو گزینه موجود است: `Cluster` (پیش‌فرض)
  و `Local`. `Cluster` IP منبع مشتری را پنهان می‌کند و ممکن است باعث شود که یک وابستگی دوم
  به یک نود دیگر شود، اما باید پخش بار خوبی داشته باشد. `Local` IP منبع مشتری را حفظ می‌کند
  و از یک وابستگی دوم برای انواع LoadBalancer و NodePort جلوگیری می‌کند، اما خطرات تراز ممکن است
  پخش بار را خطرناک کند.
* `.spec.healthCheckNodePort` - پورت بررسی وضعیت node را مشخص می‌کند
  (شماره پورت عددی) برای سرویس. اگر مشخص نکنید
  `healthCheckNodePort`، کنترل‌کننده سرویس یک پورت از محدوده NodePort خوشه شما را اختصاص می‌دهد.
  می‌توانید آن محدوده را با تنظیم یک گزینه خط فرمان سرور API
  `--service-node-port-range`. سرویس از مقدار `healthCheckNodePort` مشخص شده توسط کاربر استفاده خواهد کرد
  اگر شما آن را مشخص کنید، تا زمانی که
  Type Service به LoadBalancer و `externalTrafficPolicy` تنظیم شود
  به `Local`.

پیکربندی `externalTrafficPolicy` به Local در manifest Service
این ویژگی را فعال می‌کند. برای مثال:

```yaml


apiVersion: v1
kind: Service
metadata:
  name: example-service
spec:
  selector:
    app: example
  ports:
    - port: 8765
      targetPort: 9376
  externalTrafficPolicy: Local
  type: LoadBalancer
```

### هشدارها و محدودیت‌ها در حفظ IP منابع

خدمات توازن بار از برخی ارائه‌دهندگان ابر اجازه می‌دهند که وزن‌های مختلف را برای هر هدف پیکربندی نکنند.

هر هدف به طور مساوی در ارسال ترافیک به نود‌ها، ترازمندی بیرونی نیست که بر روی پادهای مختلف موجود است.
بارگذار خارجی از تعداد پادهای موجود در هر نود که به عنوان هدف استفاده می‌شود، بی‌خبر است.

جایی که `NumServicePods <<  NumNodes` یا `NumServicePods >> NumNodes`، توزیع نزدیک به برابر
دیده می‌شود، حتی بدون وزن‌ها.

ترافیک پاد به پاد داخلی باید مانند خدمات ClusterIP عمل کند، با احتمال برابر در تمام پادها.

## جمع آوری بارگذارهای خارجی

{{< feature-state for_k8s_version="v1.17" state="stable" >}}

در حالت عادی، منابع بارگذارهای متناظر در ارائه‌دهنده ابر باید
به سرعت پس از حذف یک نوع Service LoadBalancer پاک شود. اما معلوم است
که در گوشه‌های مختلف موارد مختلفی وجود دارد که پس از
حذف خدمات مربوط به بارگذارها در ابر، منابع متروک می‌مانند. محافظت از تمام‌کننده برای خدمات LoadBalancers
برای جلوگیری از وقوع این موارد معرفی شد. با استفاده از تمام‌کننده‌ها، یک منبع سرویس
هرگز پس از پاک شدن منابع بارگذار به روز نمی‌شود.

به طور خاص، اگر یک Service نوع LoadBalancer داشته باشد، کنترل‌کننده خدمات
تمام‌کننده‌ای به نام `service.kubernetes.io/load-balancer-cleanup` را ضمیمه می‌کند.
تمام‌کننده تنها بعد از پاک شدن منبع بارگذار از بین می‌رود.
این اقدام مانع از ماندگاری منابع بارگذار در گوشه‌های مختلفی مانند
خدمات کنترل‌کننده می‌شود.

## ارائه دهندگان بارگذار خارجی

مهم است که داده‌گذاری برای این عملکرد از طریق یک بارگذار خارجی ارائه شود که در مقابل کلاستر Kubernetes خارجی است.

زمانی که نوع Service به LoadBalancer تنظیم شود، Kubernetes عملکرد معادل Type برابر با ClusterIP را به پادهای
Kubernetes داخل کلاستر ارائه می‌دهد و با ایجاد خودکار بارگذار خارجی
IP‌های لازم را برای نودهای میزبان پادهای Kubernetes مربوطه برنامه‌ریزی می‌کند. پیچیدگی‌های چک‌های اصلاحات (اگر لازم باشد) و قوانین فیلترینگ (اگر لازم باشد) را کنترل می‌کند. یک بار ارائه‌دهنده ابر یک آدرس IP برای بارگذار خارجی اختصاص می‌دهد، برنامه کنترلی آن آدرس IP خارجی را جستجو می‌کند و آن را در شیء Service پر می‌کند.

## {{% heading "whatsnext" %}}

* دنبال کردن [آموزش اتصال برنامه‌ها با خدمات](/docs/tutorials/services/connect-applications-service/)
* خواندن درباره [Service](/docs/concepts/services-networking/service/)
* خواندن درباره [Ingress](/docs/concepts/services-networking/ingress/)