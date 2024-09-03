---
title: مدیریت اشیاء Kubernetes با استفاده از فایل‌های پیکربندی به شیوه اظهاری
content_type: task
weight: 10
---

<!-- overview -->
اشیاء Kubernetes می‌توانند با ذخیره چندین فایل پیکربندی شیء در یک دایرکتوری و استفاده از `kubectl apply` برای ایجاد، به‌روزرسانی و حذف شوند. این روش امکان نگهداری تغییرات اعمال شده بر روی اشیاء زنده را بدون ادغام دوباره آن‌ها به فایل‌های پیکربندی ارائه می‌دهد. `kubectl diff` همچنین به شما پیش‌نمایشی از تغییراتی که `apply` انجام خواهد داد، ارائه می‌دهد.

## {{% heading "prerequisites" %}}

نصب [`kubectl`](/docs/tasks/tools/).

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!-- steps -->

## مقایسه مزایا و معایب

ابزار `kubectl` سه نوع مدیریت اشیاء را پشتیبانی می‌کند:

* دستورات امپراتیوی
* پیکربندی اشیاء به شیوه امپراتیو
* پیکربندی اشیاء به شیوه اظهاری

برای بحث در مورد مزایا و معایب هر یک از این انواع مدیریت اشیاء، به [مدیریت اشیاء Kubernetes](/docs/concepts/overview/working-with-objects/object-management/) مراجعه کنید.

## مرور

پیکربندی اشیاء به شیوه اظهاری نیازمند درک کاملی از تعریفات و پیکربندی اشیاء Kubernetes است. اگر تا کنون این کار را انجام نداده‌اید، مستندات زیر را مطالعه و کامل کنید:

* [مدیریت اشیاء Kubernetes با دستورات امپراتیوی](/docs/tasks/manage-kubernetes-objects/imperative-command/)
* [مدیریت امپراتیوی اشیاء Kubernetes با فایل‌های پیکربندی](/docs/tasks/manage-kubernetes-objects/imperative-config/)

در ادامه تعاریفی از اصطلاحات مورد استفاده در این مستند آمده است:

- *فایل پیکربندی شیء / فایل پیکربندی*: فایلی که پیکربندی یک شیء Kubernetes را تعریف می‌کند. این موضوع نشان می‌دهد که چگونه فایل‌های پیکربندی به `kubectl apply` منتقل می‌شوند. فایل‌های پیکربندی معمولاً در کنترل منابع نگهداری می‌شوند، مانند Git.
- *پیکربندی زنده شیء / پیکربندی زنده*: مقادیر پیکربندی زنده یک شیء، همانطور که توسط خوشه Kubernetes مشاهده می‌شود. این‌ها در ذخیره‌سازی خوشه Kubernetes نگهداری می‌شوند، معمولاً etcd.
- *نویسنده پیکربندی اظهاری / نویسنده اظهاری*: شخص یا مؤلف نرم‌افزار که به‌روزرسانی‌ها را به پیکربندی زنده اعمال می‌کند. نویسندگان زنده که در این موضوع یاد شده‌اند، تغییرات را در فایل‌های پیکربندی اشیاء اعمال می‌کنند و `kubectl apply` را برای نوشتن تغییرات اجرا می‌کنند.

## چگونگی ایجاد اشیاء

از `kubectl apply` برای ایجاد تمام اشیاء استفاده کنید، به جز آن اشیاء که از قبل وجود دارند، که توسط فایل‌های پیکربندی در یک دایرکتوری مشخص تعریف شده‌اند:

```shell
kubectl apply -f <directory>
```

این دستور برای هر شیء، شماره `kubectl.kubernetes.io/last-applied-configuration` را تنظیم می‌کند. این نشانگر حاوی محتویات فایل پیکربندی شیء است که برای ایجاد شیء استفاده شده است.

{{< note >}}
برای پردازش بازگشتی دایرکتوری‌ها، پرچم `-R` را اضافه کنید.
{{< /note >}}

در زیر مثالی از یک فایل پیکربندی شیء آمده است:

{{% code_sample file="application/simple_deployment.yaml" %}}

از `kubectl diff` برای چاپ شیءی که ایجاد می‌شود، استفاده کنید:

```shell
kubectl diff -f https://k8s.io/examples/application/simple_deployment.yaml
```

{{< note >}}
`diff` از [dry-run سمت سرور](/docs/reference/using-api/api-concepts/#dry-run) استفاده می‌کند که باید در `kube-apiserver` فعال شود.

از آنجا که `diff` یک درخواست اعمال کردن سمت سرور را در حالت dry-run انجام می‌دهد، نیاز به دسترسی‌های `PATCH`، `CREATE` و `UPDATE` دارد. جهت کسب اطلاعات بیشتر به [مجوزهای Dry-Run](/docs/reference/using-api/api-concepts#dry-run-authorization) مراجعه کنید.
{{< /note >}}

شیء را با استفاده از `kubectl apply` ایجاد کنید:

```shell
kubectl apply -f https://k8s.io/examples/application/simple_deployment.yaml
```

پیکربندی زنده را با استفاده از `kubectl get` چاپ کنید:

```shell
kubectl get -f https://k8s.io/examples/application/simple_deployment.yaml -o yaml
```

خروجی نشان می‌دهد که توضیحات `kubectl.kubernetes.io/last-applied-configuration` به پیکربندی زنده نوشته شده است و با فایل پیکربندی همخوانی دارد:

```yaml
kind: Deployment


metadata:
  annotations:
    # ...
    # This is the json representation of simple_deployment.yaml
    # It was written by kubectl apply when the object was created
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"minReadySeconds":5,"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.14.2","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
  # ...
spec:
  # ...
  minReadySeconds: 5
  selector:
    matchLabels:
      # ...
      app: nginx
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.14.2
        # ...
        name: nginx
        ports:
        - containerPort: 80
        # ...
      # ...
    # ...
  # ...
```

## چگونگی به‌روزرسانی اشیاء

شما همچنین می‌توانید از `kubectl apply` برای به‌روزرسانی تمام اشیاء تعریف شده در یک دایرکتوری استفاده کنید، حتی اگر این اشیاء از قبل وجود داشته باشند. این رویکرد به انجام موارد زیر می‌پردازد:

1. تنظیم فیلدهایی که در فایل پیکربندی ظاهر می‌شوند در پیکربندی زنده.
2. پاک کردن فیلدهای حذف شده از فایل پیکربندی در پیکربندی زنده.

```shell
kubectl diff -f <directory>
kubectl apply -f <directory>
```

{{< note >}}
برای پردازش بازگشتی دایرکتوری‌ها، پرچم `-R` را اضافه کنید.
{{< /note >}}

در ادامه یک نمونه از فایل پیکربندی آمده است:

{{% code_sample file="application/simple_deployment.yaml" %}}

از `kubectl apply` برای ایجاد شیء استفاده کنید:

```shell
kubectl apply -f https://k8s.io/examples/application/simple_deployment.yaml
```

{{< note >}}
به‌عنوان نمایش، دستور فوق به یک فایل پیکربندی تکی ارجاع می‌دهد به‌جای یک دایرکتوری.
{{< /note >}}

پیکربندی زنده را با استفاده از `kubectl get` چاپ کنید:

```shell
kubectl get -f https://k8s.io/examples/application/simple_deployment.yaml -o yaml
```

خروجی نشان می‌دهد که نشانگر `kubectl.kubernetes.io/last-applied-configuration` به پیکربندی زنده نوشته شده است و با فایل پیکربندی همخوانی دارد:

```yaml
kind: Deployment
metadata:
  annotations:
    # ...
    # این نمایندهٔ نمایشی از simple_deployment.yaml در json است
    # که توسط kubectl apply در زمان ایجاد شیء نوشته شده است.
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"minReadySeconds":5,"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.14.2","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
  # ...
spec:
  # ...
  minReadySeconds: 5
  selector:
    matchLabels:
      # ...
      app: nginx
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.14.2
        # ...
        name: nginx
        ports:
        - containerPort: 80
      # ...
    # ...
  # ...
```

با استفاده از `kubectl scale` فقط فیلد `replicas` را در پیکربندی زنده به‌روز کنید. این کار با استفاده از `kubectl apply` انجام نمی‌شود:

```shell
kubectl scale deployment/nginx-deployment --replicas=2
```

پیکربندی زنده را با استفاده از `kubectl get` چاپ کنید:

```shell
kubectl get deployment nginx-deployment -o yaml
```

خروجی نشان می‌دهد که فیلد `replicas` به مقدار 2 تنظیم شده است و اطلاعات `last-applied-configuration` شامل فیلد `replicas` نمی‌شود:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    # ...
    # توجه داشته باشید که این اطلاعات شامل replicas نمی‌شود
    # زیرا این اطلاعات از طریق apply به‌روز نشده‌اند.
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"minReadySeconds":5,"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.14.2","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
  # ...
spec:
  replicas: 2 # توسط `kubectl scale` تنظیم شده است. توسط `kubectl apply` نادیده گرفته می‌شود.
  # ...
  minReadySeconds: 5
  selector:
    matchLabels:
      # ...
      app: nginx
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.14.2
        # ...
        name: nginx
        ports:
        - containerPort: 80
      # ...
    # ...
  # ...
```

پیکربندی فایل `update_deployment.yaml` را برای تغییر تصویر از `nginx:1.14.2` به `nginx:1.16.1` به‌روز کنید و فیلد `minReadySeconds` را حذف کنید:

{{% code_sample file="application/update_deployment.yaml" %}}

تغییرات اعمال شده به فایل پیکربندی را با `kubectl diff` اعمال کنید:

```shell
kubectl diff -f https://k8s.io/examples/application/update_deployment.yaml
kubectl apply -f https://k8s.io/examples/application/update_deployment.yaml
```

پیکربندی زنده را با استفاده از `kubectl get` چاپ کنید:

```shell
kubectl get -f https://k8s.io/examples/application/update_deployment.yaml -o yaml
```

خروجی نشان می‌دهد که تغییرات زیر در پیکربندی زنده اعمال شده است:

* فیلد `replicas` مقدار 2 توسط `kubectl scale` حفظ شده است. این امکان به خاطر این است که از فایل پیکربندی حذف شده است.
* فیلد `image` به `nginx:1.16.1` از `nginx:1.14.2` به‌روزرسانی شده است.
* نشانگر `last-applied-configuration` با تصویر جدید به‌روزرسانی شده است.
* فیلد `minReadySeconds` حذف شده است.
* نشانگر `last-applied-configuration` دیگر شامل فیلد `minReadySeconds` نمی‌باشد.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    # ...
    # نشانگر از تصویر به nginx 1.16.1 به‌روزرسانی شده است
    # اما شامل تصویر به nginx 1.14.2 نمی‌شود.
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"selector":{"matchLabels":{"

app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.16.1","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
    # ...
spec:
  replicas: 2 # توسط `kubectl scale` تنظیم شده است. توسط `kubectl apply` نادیده گرفته می‌شود.
  # minReadySeconds cleared by `kubectl apply`
  # ...
  selector:
    matchLabels:
      # ...
      app: nginx
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.16.1 # Set by `kubectl apply`
        # ...
        name: nginx
        ports:
        - containerPort: 80
      # ...
    # ...
  # ...
```
{{< warning >}}
ترکیب `kubectl apply` با دستورات شیء امریکی مانند `create` و `replace` پشتیبانی نمی‌شود. این به این دلیل است که `create` و `replace` دارای `kubectl.kubernetes.io/last-applied-configuration` نیستند که `kubectl apply` از آن برای محاسبه به‌روزرسانی‌ها استفاده می‌کند.
{{< /warning >}}

## چگونگی حذف اشیاء

برای حذف اشیاء مدیریت شده توسط `kubectl apply` دو روش وجود دارد.

### پیشنهاد شده: `kubectl delete -f <filename>`

حذف دستی اشیاء با استفاده از دستور امریکی پیشنهاد می‌شود، زیرا واضح‌تر در مورد چه چیزی حذف می‌شود و احتمال حذف ناخواسته‌ای کاربر را کاهش می‌دهد:

```shell
kubectl delete -f <filename>
```

### جایگزین: `kubectl apply -f <directory> --prune`

به عنوان جایگزین برای `kubectl delete`، می‌توانید از `kubectl apply` برای شناسایی اشیاءی که باید پس از حذف فایل‌های پیکربندی آنها از یک دایرکتوری در فایل‌سیستم محلی حذف شوند، استفاده کنید.

در Kubernetes {{< skew currentVersion >}}، دو حالت pruning در `kubectl apply` موجود است:

- حالت مبتنی بر لیست اجازه‌دهنده: این حالت از زمان kubectl v1.5 وجود دارد اما هنوز به دلیل مشکلات طراحی خود از نظر استفاده، صحت و عملکرد در حالت آلفا است. حالت مبتنی بر ApplySet برای جایگزینی آن طراحی شده است.
- حالت مبتنی بر ApplySet: یک مجموعه اجرایی یک شی‌سرور (به طور پیش‌فرض، یک Secret) است که kubectl می‌تواند برای پیگیری دقیق و کارآمد عضویت مجموعه در عملیات‌های `apply` استفاده کند. این حالت در ورژن kubectl v1.27 به عنوان جایگزین حالت مبتنی بر لیست اجازه‌دهنده به طور آلفا معرفی شد.

{{< tabs name="kubectl_apply_prune" >}}
{{% tab name="لیست اجازه‌دهنده" %}}

{{< feature-state for_k8s_version="v1.5" state="alpha" >}}

{{< warning >}}
در استفاده از `--prune` با `kubectl apply` در حالت لیست اجازه‌دهنده، مراقب باشید. کدام اشیاء در این حالت پران شده وابسته به مقادیر پران کننده`--prune-allowlist`،`--selector`و`--namespace` و بررسی در دینامیک شما از اشیاء در برنامه ریزی تغییرات می‌باشد. مخصوصا اگر ارزشهای بین فراخوانی تغییر کند، این می‌تواند به حذف یا حفظ غیرمنتظره شیئی ادامه دهد.
{{< /warning >}}

برای استفاده از حالت pruning مبتنی بر لیست اجازه‌دهنده، پرچم‌های زیر را به فراخوانی `kubectl apply` خود اضافه کنید:

- `--prune`: حذف اشیاء قبلی که در مجموعه گذشته به فراخوانی کنونی منتقل نشده‌اند.
- `--prune-allowlist`: فهرستی از نوع‌های گروه-نسخه-موارد (GVKs) را برای حذف در نظر بگیرید. این پرچم اختیاری است اما بسیار تشویق می‌شود، زیرا ارزش پیش‌فرض آن یک فهرست جزئی از نوع‌های نشانه‌دار و دامنه خوشه‌ای است، که می‌تواند به نتایج غیرمنتظره منجر شود.
- `--selector/-l`: از انتخاب گر برچسب برای محدود کردن مجموعه اشیاء انتخاب شده برای حذف استفاده کنید. این پرچم اختیاری است اما بسیار تشویق می‌شود.
- `--all`: بجای `--selector/-l` برای به صراحت انتخاب همه اشیاء اعمال شده پیشنهاد می‌شود.

حذف مبتنی بر لیست اجازه‌دهنده به API server برای تمام اشیاء از GVKs اجازه‌دهنده که با برچسب‌های داده شده مطابقت داشته باشد، و سعی می‌کند با پروندن تنظیمات شیء مانندی که آن در مجلد بی‌مانده به آن مطابقت دارد که اگر یک شیء مطابقت داشت، و اگر شما مجموعه‌ای از `kubectl.kubernetes.io/last-applied-configuration` می‌شود.

```shell
kubectl apply -f <directory> --prune -l <labels> --prune-allowlist=<gvk-list>
```

{{< warning >}}
باید از `kubectl apply` با prune کنار هم تنها در برابر دایرکتوری اصلی شیئ‌ها اجرا شود. اجرا در برابر زیر‌دایرکتوری‌ها می‌تواند باعث حذف ناخواسته شیئی شود اگر از پیش اعمال شده باشد، برچسب‌های داده شده (اگر هستند) و در زیر‌دایرکتوری‌ها ظاهر نشوند.
{{< /warning >}}

{{% /tab %}}

{{% tab name="Apply set" %}}

{{< feature-state for_k8s_version="v1.27" state="alpha" >}}

{{< caution >}}
`kubectl apply --prune --applyset` آلفا است و تغییرات سازگاری‌پذیر ممکن است در انتشارهای بعدی معرفی شود

.
{{< /caution >}}

برای استفاده از pruning مبتنی بر ApplySet، متغیر محیطی `KUBECTL_APPLYSET=true` را تنظیم کرده و پرچم‌های زیر را به فراخوانی `kubectl apply` خود اضافه کنید:
- `--prune`: حذف اشیاء قبلی که در مجموعه گذشته به فراخوانی کنونی منتقل نشده‌اند.
- `--applyset`: نام یک شیء که kubectl می‌تواند برای پیگیری دقیق و کارآمد عضویت مجموعه در عملیات‌های `apply` استفاده کند.

```shell
KUBECTL_APPLYSET=true kubectl apply -f <directory> --prune --applyset=<name>
```

به طور پیش‌فرض، نوع شیء ApplySet استفاده‌شده یک Secret است. با این حال، ConfigMaps نیز می‌توانند در فرمت `--applyset=configmaps/<name>` استفاده شوند. هنگام استفاده از یک Secret یا ConfigMap، kubectl شیء را ایجاد می‌کند اگر این شیء از قبل وجود نداشته باشد.

استفاده از منابع سفارشی به عنوان شیء پدر ApplySet همچنین ممکن است. برای فعال کردن این، Custom Resource Definition (CRD) را که منبع را تعریف می‌کند با `applyset.kubernetes.io/is-parent-type: true` برچسب بزنید. سپس، شیء را ایجاد کنید که می‌خواهید به عنوان شیء پدر ApplySet استفاده کنید (kubectl این کار را به طور خودکار برای منابع سفارشی انجام نمی‌دهد). در نهایت، به آن شیء در پرچم applyset به صورت زیر ارجاع دهید:
`--applyset=<resource>.<group>/<name>` (به عنوان مثال، `widgets.custom.example.com/widget-name`).

با pruning مبتنی بر ApplySet، kubectl برچسب `applyset.kubernetes.io/part-of=<parentID>` را به هر شیء در مجموعه اضافه می‌کند قبل از اینکه آنها به سرور ارسال شوند. به دلایل عملکردی، همچنین لیست انواع منابع و فضاهایی که مجموعه شامل می‌شود را جمع‌آوری می‌کند و این‌ها را در حالت برچسب‌گذاری بر روی شیء والدین زنده اضافه می‌کند. در نهایت، در پایان عملیات apply، این شیء از API server برای اشیاء این انواع در آن فضا (یا در حالت مورد استفاده) که به مجموعه تعلق دارند، به عنوان مشخص شده است توسط applyset.kubernetes.io/part-of=<parentID>` برچسب را پرس و جو می‌کند.

نکات احتیاطی و محدودیت‌ها:

- هر شیء ممکن است عضوی از حداکثر یک مجموعه باشد.
- پرچم `--namespace` برای استفاده از هر شیء نامزد، شامل Secret پیش‌فرض مورد نیاز است. این به معنای آن است که ApplySets در چندین فضا نیازمند استفاده از یک منبع سفارشی متمرکز به عنوان شیء والدین هستند.
- برای استفاده ایمن از pruning مبتنی بر ApplySet با چندین دایرکتوری، برای هرکدام از آنها یک نام یکتا ApplySet استفاده کنید.

{{% /tab %}}

## چگونگی مشاهده یک شیء

می‌توانید از `kubectl get` با `-o yaml` برای مشاهده تنظیمات یک شیء زنده استفاده کنید:

```shell
kubectl get -f <filename|url> -o yaml
```

## چگونگی محاسبه تفاوت‌ها و ادغام تغییرات تطبیق

{{< caution >}}
یک *patch* عملیات به روز رسانی است که به موارد خاص یک شیء محدود می‌شود به جای شیء کامل. این امکان را فراهم می‌کند تنها یک مجموعه خاص از موارد یک شیء را بدون خواندن آن بدست آورد.
{{< /caution >}}

زمانی که `kubectl apply` پیکربندی زنده یک شیء را به روزرسانی می‌کند، این کار را با ارسال یک درخواست patch به سرور API انجام می‌دهد. این patch به روزرسانی‌های محدود به فیلدهای خاص پیکربندی زنده را تعریف می‌کند. دستور `kubectl apply` این درخواست patch را با استفاده از فایل پیکربندی، پیکربندی زنده، و نشانه `last-applied-configuration` که در پیکربندی زنده ذخیره شده، محاسبه می‌کند.

### محاسبه ادغام patch

دستور `kubectl apply` محتوای فایل پیکربندی را به زیر نشانه `kubectl.kubernetes.io/last-applied-configuration` می‌نویسد. این برای شناسایی فیلدهایی که از پیکربندی فایل حذف شده‌اند و باید از پیکربندی زنده پاک شوند، استفاده می‌شود. این مراحل برای محاسبه که کدام فیلدها باید حذف یا تنظیم شوند، استفاده می‌شود:

1. محاسبه فیلدهایی که باید حذف شوند. این فیلدها در `last-applied-configuration` وجود دارند و در فایل پیکربندی نیستند.
2. محاسبه فیلدهایی که باید اضافه یا تنظیم شوند. ا

ین فیلدها در فایل پیکربندی وجود دارند و ارزش آنها با پیکربندی زنده مطابقت ندارد.

در اینجا محاسبات ادغامی که توسط `kubectl apply` انجام می‌دهد:

1. با خواندن ارزش‌ها از `last-applied-configuration` و مقایسه آنها با ارزش‌های موجود در فایل پیکربندی، محاسبه فیلدهایی که باید حذف شوند.
   پاک کردن فیلدهای به وضوح در فایل پیکربندی شیء محلی مانند کنسول موجود باشد.

این پیکربندی زنده است که نتیجه ادغام می‌باشد:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    # ...
    # این حاوی تصویر به روز شده به nginx 1.16.1 است، اما حاوی تصویر به nginx 1.16.1 و حذف تنظیمات به 2 نمی‌باشد.
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.16.1","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
    # ...
spec:
  selector:
    matchLabels:
      # ...
      app: nginx
  replicas: 2 # تنظیم شده توسط `kubectl scale`. توسط `kubectl apply` نادیده گرفته شده است.
  # minReadySeconds cleared by `kubectl apply`
  # ...
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.16.1 # تنظیم شده توسط `kubectl apply`
        # ...
        name: nginx
        ports:
        - containerPort: 80
        # ...
      # ...
    # ...
  # ...
```

### چگونگی ادغام انواع مختلف فیلدها

نحوه ادغام یک فیلد خاص در یک فایل پیکربندی با پیکربندی زنده به نوع فیلد وابسته است. چند نوع فیلد وجود دارد:

- *primitive*: یک فیلد از نوع رشته، عدد صحیح، یا بولی. به عنوان مثال، `image` و `replicas` فیلدهای ابتدایی هستند. **عملیات:** جایگزینی.

- *map*، همچنین به عنوان *object* شناخته می‌شود: یک فیلد از نوع map یا یک نوع پیچیده که حاوی زیرفیلدها است. به عنوان مثال، `labels`،
  `annotations`، `spec` و `metadata` همه map هستند. **عملیات:** ادغام عناصر یا زیرفیلدها.

- *list*: یک فیلد حاوی یک لیست از موارد که می‌تواند شامل انواع ابتدایی یا map باشد. به عنوان مثال، `containers`، `ports` و `args` لیست‌ها هستند. **عملیات:** متغیر.

زمانی که `kubectl apply` یک فیلد map یا list را به‌روز می‌کند، معمولاً کل فیلد را جایگزین نمی‌کند، بلکه زیرفیلدها یا عناصر فردی را به‌روز می‌کند. به عنوان مثال، هنگام ادغام `spec` در یک Deployment، کل `spec` جایگزین نمی‌شود. به جای آن، زیرفیلدهای `spec`، مانند `replicas`، مقایسه و ادغام می‌شوند.

### ادغام تغییرات در فیلدهای primitive

فیلدهای ابتدایی جایگزین یا پاک می‌شوند.

{{< note >}}
`-` برای "قابل اعمال نیست" استفاده می‌شود زیرا مقدار استفاده نمی‌شود.
{{< /note >}}

| فیلد در فایل پیکربندی شیء  | فیلد در فایل پیکربندی زنده  | فیلد در last-applied-configuration | عملیات |
|---------------------------|--------------------------|---------------------------------|--------|
| بله                       | بله                      | -                               | تنظیم زنده به ارزش فایل پیکربندی. |
| بله                       | خیر                      | -                               | تنظیم زنده به پیکربندی محلی.   |
| خیر                       | -                        | بله                             | از پیکربندی زنده پاک کنید.     |
| خیر                       | -                        | خیر                             | هیچ کاری انجام نمی‌دهد. ارزش زنده را نگه می‌دارد. |

### ادغام تغییرات در فیلدهای map

فیلدهایی که نقش نقشه‌ها را به خود می‌گیرند، با مقایسه هر یک از زیرفیلدها یا عناصر نقشه ادغام می‌شوند:

{{< note >}}
`-` برای "قابل اعمال نیست" استفاده می‌شود زیرا مقدار استفاده نمی‌شود.
{{< /note >}}

| کلید در فایل پیکربندی شیء   | کلید در فایل پیکربندی زنده   | فیلد در last-applied-configuration | عملیات |
|---------------------------------|--------------------------------|-----------------------------------|--------|
| بله                             | بله                            | -                                 | مقایسه مقادیر زیرفیلدها.     |
| بله                             | خیر                            | -                                 | تنظیم زنده به پیکربندی محلی.   |
| خیر                             | -                              | بله                               | از پیکربندی زنده حذف کنید.    |
| خیر                             | -                              | خیر                               | هیچ کاری انجام نمی‌دهد. ارزش زنده را نگه می‌دارد. |

### ادغام تغییرات برای فیلدهای از نوع لیست

ادغام تغییرات در یک لیست از سه استراتژی استفاده می‌کند:

* جایگزینی لیست اگر تمام عناصر آن از انواع ابتدایی باشند.
* ادغام عناصر فردی در یک لیست از عناصر پیچیده.
* ادغام یک لیست از عناصر ابتدایی.

انتخاب استراتژی برای هر فیلد به صورت فردی انجام می‌شود.

#### جایگزینی لیست اگر تمام عناصر آن از انواع ابتدایی باشند

لیست را مانند یک فیلد ابتدایی در نظر بگیرید. کل لیست را جایگزین یا حذف کنید.
این حفظ ترتیب را حفظ می‌کند.

**مثال:** از `kubectl apply` برای به‌روزرسانی فیلد `args` یک Container در یک Pod استفاده کنید. این مقدار `args` در پیکربندی زنده را به مقدار در فایل پیکربندی تنظیم می‌کند.
هر `args` عنصری که پیش‌تر به پیکربندی زنده اضافه شده بود از دست می‌رود.
ترتیب عناصر `args` تعریف شده در فایل پیکربندی در پیکربندی زنده حفظ می‌شود.

```yaml
# مقدار last-applied-configuration
    args: ["a", "b"]

# مقدار فایل پیکربندی
    args: ["a", "c"]

# پیکربندی زنده
    args: ["a", "b", "d"]

# نتیجه پس از ادغام
    args: ["a", "c"]
```

**توضیحات:** ادغام مقدار فایل پیکربندی به عنوان مقدار جدید لیست استفاده شد.

#### ادغام عناصر فردی از یک لیست از عناصر پیچیده:

لیست را مانند یک نقشه در نظر بگیرید، و یک فیلد خاص از هر عنصر را به عنوان یک کلید در نظر بگیرید.
عناصر فردی را اضافه، حذف یا به‌روزرسانی کنید.
این حفظ ترتیب را حفظ نمی‌کند.

این استراتژی ادغام از یک برچسب خاص برای هر فیلد استفاده می‌کند که به نام `patchMergeKey` معروف است. `patchMergeKey`
برای هر فیلد در کد منبع Kubernetes تعریف شده است:
[types.go](https://github.com/kubernetes/api/blob/d04500c8c3dda9c980b668c57abc2ca61efcf5c4/core/v1/types.go#L2747)
زمانی که یک لیست از نقشه‌ها را ادغام می‌کنید، فیلد مشخص شده به عنوان `patchMergeKey` برای عنصر مشخص شده
برای آن عنصر استفاده می‌شود مانند یک کلید نقشه.

**مثال:** از `kubectl apply` برای به‌روزرسانی فیلد `containers` در PodSpec استفاده کنید.
این مخلوط لیست را به عنوان یک نقشه در نظر می‌گیرد که هر عنصر به عنوان یک کلید با نام `name` عمل می‌کند.

```yaml
# مقدار last-applied-configuration
    containers:
    - name: nginx
      image: nginx:1.16
    - name: nginx-helper-a # کلید: nginx-helper-a; در نتیجه حذف خواهد شد
      image: helper:1.3
    - name: nginx-helper-b # کلید: nginx-helper-b; خواهد ماند
      image: helper:1.3

# مقدار فایل پیکربندی
    containers:
    - name: nginx
      image: nginx:1.16
    - name: nginx-helper-b
      image: helper:1.3
    - name: nginx-helper-c # کلید: nginx-helper-c; در نتیجه اضافه خواهد شد
      image: helper:1.3

# پیکربندی زنده
    containers:
    - name: nginx
      image: nginx:1.16
    - name: nginx-helper-a
      image: helper:1.3
    - name: nginx-helper-b
      image: helper:1.3
      args: ["run"] # فیلد حفظ شده است
    - name: nginx-helper-d # کلید: nginx-helper-d; خواهد ماند
      image: helper:1.3

# نتیجه پس از ادغام
    containers:
    - name: nginx
      image: nginx:1.16
      # عنصر nginx-helper-a حذف شد
    - name: nginx-helper-b
      image: helper:1.3
      args: ["run

"] # فیلد حفظ شد
    - name: nginx-helper-c # عنصر اضافه شد
      image: helper:1.3
    - name: nginx-helper-d # عنصر نادیده گرفته شد
      image: helper:1.3
```

**توضیحات:**

- کانتینر با نام "nginx-helper-a" حذف شد زیرا هیچ کانتینری با آن نام در فایل پیکربندی وجود نداشت.
- کانتینر با نام "nginx-helper-b" تغییرات به `args` در پیکربندی زنده را حفظ کرد. `kubectl apply` قادر بود تشخیص دهد
  که "nginx-helper-b" در پیکربندی زنده همان "nginx-helper-b" در فایل پیکربندی است، حتی با وجود اینکه فیلدهای آنها مقادیر متفاوتی داشتند (بدون `args` در فایل پیکربندی). این به این دلیل بود که مقدار فیلد `patchMergeKey` (نام) در هر دو یکسان بود.
- کانتینر با نام "nginx-helper-c" اضافه شد زیرا هیچ کانتینری با آن نام در پیکربندی زنده وجود نداشت، اما یکی با آن نام در فایل پیکربندی وجود داشت.
- کانتینر با نام "nginx-helper-d" حفظ شد زیرا هیچ عنصری با آن نام در last-applied-configuration وجود نداشت.
### ادغام یک لیست از عناصر ابتدایی

از نسخه 1.5 Kubernetes، ادغام لیست عناصر ابتدایی پشتیبانی نمی‌شود.

{{< note >}}
استراتژی‌هایی که برای یک فیلد خاص انتخاب می‌شوند توسط برچسب `patchStrategy` در [types.go](https://github.com/kubernetes/api/blob/d04500c8c3dda9c980b668c57abc2ca61efcf5c4/core/v1/types.go#L2748) کنترل می‌شود.
اگر هیچ `patchStrategy` برای فیلدی از نوع لیست مشخص نشده باشد، لیست جایگزین می‌شود.
{{< /note >}}

{{< comment >}}
TODO(pwittrock): Uncomment this for 1.6

- با لیست به عنوان مجموعه‌ای از عناصر ابتدایی رفتار کنید. عناصر فردی را جایگزین یا حذف کنید. ترتیب و تکرار را حفظ نمی‌کند.

**مثال:** استفاده از `apply` برای به‌روزرسانی فیلد `finalizers` از ObjectMeta عناصر افزوده شده به پیکربندی زنده را حفظ می‌کند. ترتیب `finalizers` از بین می‌رود.
{{< /comment >}}

## مقادیر پیش‌فرض فیلدها

سرور API برخی فیلدها را به مقادیر پیش‌فرض در پیکربندی زنده تنظیم می‌کند اگر آنها هنگام ایجاد شیء مشخص نشده باشند.

در اینجا یک فایل پیکربندی برای یک Deployment است. این فایل `strategy` را مشخص نمی‌کند:

{{% code_sample file="application/simple_deployment.yaml" %}}

شیء را با استفاده از `kubectl apply` ایجاد کنید:

```shell
kubectl apply -f https://k8s.io/examples/application/simple_deployment.yaml
```

پیکربندی زنده را با استفاده از `kubectl get` چاپ کنید:

```shell
kubectl get -f https://k8s.io/examples/application/simple_deployment.yaml -o yaml
```

خروجی نشان می‌دهد که سرور API چندین فیلد را به مقادیر پیش‌فرض در پیکربندی زنده تنظیم کرده است. این فیلدها در فایل پیکربندی مشخص نشده بودند.

```yaml
apiVersion: apps/v1
kind: Deployment
# ...
spec:
  selector:
    matchLabels:
      app: nginx
  minReadySeconds: 5
  replicas: 1 # به طور پیش‌فرض توسط سرور API تنظیم شده
  strategy:
    rollingUpdate: # به طور پیش‌فرض توسط سرور API - مشتق شده از strategy.type
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate # به طور پیش‌فرض توسط سرور API
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.14.2
        imagePullPolicy: IfNotPresent # به طور پیش‌فرض توسط سرور API تنظیم شده
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP # به طور پیش‌فرض توسط سرور API تنظیم شده
        resources: {} # به طور پیش‌فرض توسط سرور API تنظیم شده
        terminationMessagePath: /dev/termination-log # به طور پیش‌فرض توسط سرور API تنظیم شده
      dnsPolicy: ClusterFirst # به طور پیش‌فرض توسط سرور API تنظیم شده
      restartPolicy: Always # به طور پیش‌فرض توسط سرور API تنظیم شده
      securityContext: {} # به طور پیش‌فرض توسط سرور API تنظیم شده
      terminationGracePeriodSeconds: 30 # به طور پیش‌فرض توسط سرور API تنظیم شده
# ...
```

در یک درخواست patch، فیلدهای پیش‌فرض مجدداً پیش‌فرض نمی‌شوند مگر اینکه به طور صریح به عنوان بخشی از درخواست patch پاک شوند. این می‌تواند رفتار غیرمنتظره‌ای برای فیلدهایی که بر اساس مقادیر سایر فیلدها پیش‌فرض می‌شوند، ایجاد کند. وقتی سایر فیلدها بعداً تغییر می‌کنند، مقادیر پیش‌فرض شده از آنها به‌روزرسانی نمی‌شوند مگر اینکه به طور صریح پاک شوند.

برای این دلیل، توصیه می‌شود که فیلدهای خاصی که توسط سرور پیش‌فرض شده‌اند به طور صریح در فایل پیکربندی تعریف شوند، حتی اگر مقادیر مورد نظر با پیش‌فرض‌های سرور هم‌خوانی داشته باشند. این باعث می‌شود که تشخیص مقادیر متناقض که توسط سرور مجدداً پیش‌فرض نمی‌شوند آسان‌تر شود.

**مثال:**

```yaml
# پیکربندی آخرین اعمال شده
spec:
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

# فایل پیکربندی
spec:
  strategy:
    type: Recreate # مقدار به‌روزرسانی شده
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

# پیکربندی زنده
spec:
  strategy:
    type: RollingUpdate # مقدار پیش‌فرض
    rollingUpdate: # مقدار پیش‌فرض مشتق شده از type
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

# نتیجه پس از ادغام - خطا!
spec:
  strategy:
    type: Recreate # مقدار به‌روزرسانی شده: ناسازگار با rollingUpdate
    rollingUpdate: # مقدار پیش‌فرض: ناسازگار با "type: Recreate"
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

در این مثال، مقدار پیش‌فرض `rollingUpdate` که توسط سرور API تنظیم شده بود با مقدار `type` به‌روزرسانی شده `Recreate` در پیکربندی زنده ناسازگار است، که باعث خطا در ادغام می‌شود.
**توضیح:**

1. کاربر یک Deployment ایجاد می‌کند بدون تعریف `strategy.type`.
2. سرور `strategy.type` را به `RollingUpdate` پیش‌فرض می‌کند و مقادیر `strategy.rollingUpdate` را پیش‌فرض می‌کند.
3. کاربر `strategy.type` را به `Recreate` تغییر می‌دهد. مقادیر `strategy.rollingUpdate` در مقادیر پیش‌فرض باقی می‌مانند، هرچند که سرور انتظار دارد آنها پاک شوند. اگر مقادیر `strategy.rollingUpdate` ابتدا در فایل پیکربندی تعریف شده بودند، واضح‌تر بود که باید حذف شوند.
4. اعمال شکست می‌خورد زیرا `strategy.rollingUpdate` پاک نشده است. فیلد `strategy.rollingupdate` نمی‌تواند با یک `strategy.type` از نوع `Recreate` تعریف شود.

توصیه: این فیلدها باید به طور صریح در فایل پیکربندی شیء تعریف شوند:

- انتخاب‌کننده‌ها و برچسب‌های PodTemplate بر روی بارهای کاری، مانند Deployment، StatefulSet، Job، DaemonSet، ReplicaSet، و ReplicationController
- استراتژی انتشار Deployment

### چگونه فیلدهای پیش‌فرض شده توسط سرور یا فیلدهای تنظیم شده توسط نویسندگان دیگر را پاک کنیم

فیلدهایی که در فایل پیکربندی ظاهر نمی‌شوند را می‌توان با تنظیم مقادیر آنها به `null` و سپس اعمال فایل پیکربندی پاک کرد. برای فیلدهایی که توسط سرور پیش‌فرض شده‌اند، این کار باعث پیش‌فرض‌سازی مجدد مقادیر می‌شود.

## چگونه مالکیت یک فیلد را بین فایل پیکربندی و نویسندگان دستوری مستقیم تغییر دهیم

این‌ها تنها روش‌هایی هستند که باید برای تغییر یک فیلد شیء فردی استفاده کنید:

- از `kubectl apply` استفاده کنید.
- به طور مستقیم به پیکربندی زنده بنویسید بدون تغییر فایل پیکربندی: به عنوان مثال، از `kubectl scale` استفاده کنید.

### تغییر مالکیت از یک نویسنده دستوری مستقیم به یک فایل پیکربندی

فیلد را به فایل پیکربندی اضافه کنید. برای فیلد، به‌روزرسانی‌های مستقیم به پیکربندی زنده را که از طریق `kubectl apply` انجام نمی‌شود، متوقف کنید.

### تغییر مالکیت از یک فایل پیکربندی به یک نویسنده دستوری مستقیم

از Kubernetes 1.5، تغییر مالکیت یک فیلد از یک فایل پیکربندی به یک نویسنده دستوری نیاز به مراحل دستی دارد:

- فیلد را از فایل پیکربندی حذف کنید.
- فیلد را از حاشیه‌نویسی `kubectl.kubernetes.io/last-applied-configuration` روی شیء زنده حذف کنید.

## تغییر روش‌های مدیریت

اشیاء Kubernetes باید فقط با استفاده از یک روش در یک زمان مدیریت شوند. تغییر از یک روش به روش دیگر ممکن است، اما یک فرآیند دستی است.

{{< note >}}
استفاده از حذف دستوری با مدیریت اعلامی مشکلی ندارد.
{{< /note >}}

{{< comment >}}
TODO(pwittrock): نیاز داریم که استفاده از دستورات دستوری با
پیکربندی شیء اعلامی به گونه‌ای کار کند که فیلدها را به حاشیه‌نویسی ننوشت، و در عوض. سپس این نقطه گلوله‌ای را اضافه کنیم.

- استفاده از دستورات دستوری با پیکربندی اعلامی برای مدیریت جایی که هر کدام فیلدهای مختلف را مدیریت می‌کنند.
{{< /comment >}}

### مهاجرت از مدیریت دستوری فرمانی به پیکربندی شیء اعلامی

مهاجرت از مدیریت دستوری فرمانی به پیکربندی شیء اعلامی شامل چندین مرحله دستی است:

1. شیء زنده را به یک فایل پیکربندی محلی صادر کنید:

   ```shell
   kubectl get <kind>/<name> -o yaml > <kind>_<name>.yaml
   ```

1. به صورت دستی فیلد `status` را از فایل پیکربندی حذف کنید.

   {{< note >}}
   این مرحله اختیاری است، زیرا `kubectl apply` فیلد وضعیت را به‌روزرسانی نمی‌کند
   حتی اگر در فایل پیکربندی وجود داشته باشد.
   {{< /note >}}

1. حاشیه‌نویسی `kubectl.kubernetes.io/last-applied-configuration` را بر روی شیء تنظیم کنید:

   ```shell
   kubectl replace --save-config -f <kind>_<name>.yaml
   ```

1. فرآیندها را تغییر دهید تا به صورت انحصاری از `kubectl apply` برای مدیریت شیء استفاده کنند.

{{< comment >}}
TODO(pwittrock): چرا صادرات فیلد وضعیت را حذف نمی‌کند؟ به نظر می‌رسد که باید حذف کند.
{{< /comment >}}

### مهاجرت از پیکربندی شیء دستوری به پیکربندی شیء اعلامی

1. حاشیه‌نویسی `kubectl.kubernetes.io/last-applied-configuration` را بر روی شیء تنظیم کنید:

   ```shell
   kubectl replace --save-config -f <kind>_<name>.yaml
   ```

1. فرآیندها را تغییر دهید تا به صورت انحصاری از `kubectl apply` برای مدیریت شیء استفاده کنند.

## تعریف انتخاب‌کننده‌های کنترلر و برچسب‌های PodTemplate

{{< warning >}}
به‌روزرسانی انتخاب‌کننده‌ها بر روی کنترلرها به شدت توصیه نمی‌شود.
{{< /warning >}}

رویکرد توصیه شده این است که یک برچسب PodTemplate تنها و غیرقابل تغییر تعریف کنید
که فقط توسط انتخاب‌کننده کنترلر استفاده می‌شود و هیچ معنای دیگری ندارد.

**مثال:**

```yaml
selector:
  matchLabels:
      controller-selector: "apps/v1/deployment/nginx"
template:
  metadata:
    labels:
      controller-selector: "apps/v1/deployment/nginx"
```

## {{% heading "whatsnext" %}}

* [مدیریت اشیاء Kubernetes با استفاده از دستورات دستوری](/docs/tasks/manage-kubernetes-objects/imperative-command/)
* [مدیریت دستوری اشیاء Kubernetes با استفاده از فایل‌های پیکربندی](/docs/tasks/manage-kubernetes-objects/imperative-config/)
* [مرجع دستورات Kubectl](/docs/reference/generated/kubectl/kubectl-commands/)
* [مرجع API Kubernetes](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)