---
title: "مثال: استقرار کاساندرا با استفاده از StatefulSet"
reviewers:
- ahmetb
content_type: tutorial
weight: 30
---
<!-- دید کلی -->
این آموزش به شما نحوه اجرای [Apache Cassandra](https://cassandra.apache.org/) را در Kubernetes آموزش می‌دهد. کاساندرا به عنوان یک پایگاه داده، نیازمند ذخیره‌سازی مستمر است تا اطمینان از دوام داده‌ها (وضعیت برنامه) داشته باشد. در این مثال، یک تامین کننده بذر سفارشی کاساندرا به پایگاه داده اجازه می‌دهد تا نمونه‌های جدید کاساندرا را کشف کند وقتی که به خوشه کاساندرا می‌پیوندند.

*StatefulSets* این امکان را فراهم می‌کند تا برنامه‌های دارای حالت (stateful) را به راحتی در خوشه Kubernetes خود استقرار دهید. برای اطلاعات بیشتر در مورد ویژگی‌های استفاده شده در این آموزش، به [StatefulSet](/docs/concepts/workloads/controllers/statefulset/) مراجعه کنید.

{{< note >}}
کاساندرا و Kubernetes هر دو از اصطلاح _node_ برای اعضای یک خوشه استفاده می‌کنند. در این آموزش، Pods‌هایی که به StatefulSet تعلق دارند، به عنوان نودهای کاساندرا و اعضای خوشه کاساندرا (که به آن _ring_ گفته می‌شود) شناخته می‌شوند. وقتی این Pods‌ها در خوشه Kubernetes شما اجرا می‌شوند، سیستم کنترل Kubernetes این Pods‌ها را بر روی کوبرنتیز برنامه‌ریزی می‌کند.
{{< glossary_tooltip text="Nodes" term_id="node" >}}

وقتی یک نود Cassandra شروع به کار می‌کند، از یک لیست بذر (_seed list_) برای راه‌اندازی کشف نود‌های دیگر در حلقه استفاده می‌کند. این آموزش یک تامین کننده بذر سفارشی کاساندرا ارائه می‌دهد که به پایگاه داده اجازه می‌دهد تا نمونه‌های جدید کاساندرا را کشف کند وقتی که در خوشه Kubernetes شما ظاهر می‌شوند.
{{< /note >}}


## {{% heading "اهداف" %}}

* ایجاد و اعتبارسنجی یک سرویس headless {{< glossary_tooltip text="Service" term_id="service" >}} برای Cassandra.
* استفاده از {{< glossary_tooltip term_id="StatefulSet" >}} برای ایجاد یک حلقه Cassandra.
* اعتبارسنجی StatefulSet.
* اصلاح StatefulSet.
* حذف StatefulSet و Pods مربوطه {{< glossary_tooltip text="Pods" term_id="pod" >}}.


## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}}

برای تکمیل این آموزش، باید با مفاهیم پایه‌ای مانند {{< glossary_tooltip text="Pods" term_id="pod" >}}، {{< glossary_tooltip text="Services" term_id="service" >}}، و {{< glossary_tooltip text="StatefulSets" term_id="StatefulSet" >}} آشنایی داشته باشید.

### دستورالعمل‌های تنظیم اضافی Minikube

{{< caution >}}
[Minikube](https://minikube.sigs.k8s.io/docs/) با پیش‌فرض 2048MB حافظه و 2 CPU اجرا می‌شود. این تنظیمات می‌تواند به اشکالات منابع ناکافی منجر شود. برای جلوگیری از این مشکلات، Minikube را با تنظیمات زیر راه‌اندازی کنید:

```shell
minikube start --memory 5120 --cpus=4
```
{{< /caution >}}


<!-- محتوای درس -->
## ایجاد یک سرویس headless برای Cassandra {#creating-a-cassandra-headless-service}

در Kubernetes، {{< glossary_tooltip text="Service" term_id="service" >}} مجموعه‌ای از {{< glossary_tooltip text="Pods" term_id="pod" >}} هستند که همان کار را انجام می‌دهند.

سرویس زیر برای جستجوهای DNS بین کاساندرا Pods و مشتریان داخل خوشه شما استفاده می‌شود:

{{% code_sample file="application/cassandra/cassandra-service.yaml" %}}

برای ردیابی تمام اعضای Cassandra StatefulSet از فایل `cassandra-service.yaml`، یک سرویس ایجاد کنید:

```shell
kubectl apply -f https://k8s.io/examples/application/cassandra/cassandra-service.yaml
```


### اعتبارسنجی (اختیاری) {#validating}

دریافت سرویس Cassandra:

```shell
kubectl get svc cassandra
```

پاسخ:

```
NAME        TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
cassandra   ClusterIP   None         <none>        9042/TCP   45s
```

اگر سرویسی با نام `cassandra` نبینید، این بدان معنی است که ایجاد سرویس ناموفق بوده است. برای رفع مشکلات متداول، به [Debug Services](/docs/tasks/debug/debug-application/debug-service/) مراجعه کنید.

## استفاده از StatefulSet برای ایجاد یک حلقه Cassandra

مانیفست StatefulSet زیر یک حلقه Cassandra را که از سه Pods تشکیل شده است، ایجاد می‌کند:

{{% code_sample file="application/cassandra/cassandra-statefulset.yaml" %}}

ایجاد Cassandra StatefulSet از فایل `cassandra-statefulset.yaml`:

```shell
# اگر قادر به اعمال cassandra-statefulset.yaml بدون اصلاح باشید، از این دستور استفاده کنید
kubectl apply -f https://k8s.io/examples/application/cassandra/cassandra-statefulset.yaml
```

اگر نیاز به اصلاح `cassandra-statefulset.yaml` برای سازگاری با خوشه خود دارید، فایل `cassandra-statefulset.yaml` را دانلود کنید و سپس مانیفست را اعمال کنید:

```shell
# اگر به اصلاح محلی cassandra-statefulset.yaml نیاز داشته باشید، از این دستور استفاده کن

ید
kubectl apply -f cassandra-statefulset.yaml
```

## اعتبارسنجی StatefulSet Cassandra

1. دریافت StatefulSet Cassandra:

    ```shell
    kubectl get statefulset cassandra
    ```

    پاسخ باید مشابه زیر باشد:

    ```
    NAME        DESIRED   CURRENT   AGE
    cassandra   3         0         13s
    ```

    منبع `StatefulSet`، Pods‌ها را به صورت متوالی استقرار می‌دهد.

2. دریافت Pods برای مشاهده وضعیت ایجاد مرتب:

    ```shell
    kubectl get pods -l="app=cassandra"
    ```

    پاسخ باید مشابه زیر باشد:

    ```shell
    NAME          READY     STATUS              RESTARTS   AGE
    cassandra-0   1/1       Running             0          1m
    cassandra-1   0/1       ContainerCreating   0          8s
    ```

    ممکن است چند دقیقه طول بکشد تا تمامی سه Pods ایجاد شوند. پس از ایجاد آن‌ها، همان دستور خروجی مشابه زیر را نمایش می‌دهد:

    ```
    NAME          READY     STATUS    RESTARTS   AGE
    cassandra-0   1/1       Running   0          10m
    cassandra-1   1/1       Running   0          9m
    cassandra-2   1/1       Running   0          8m
    ```

3. اجرای دستور [nodetool](https://cwiki.apache.org/confluence/display/CASSANDRA2/NodeTool) کاساندرا داخل اولین Pod برای نمایش وضعیت حلقه:

    ```shell
    kubectl exec -it cassandra-0 -- nodetool status
    ```

    پاسخ باید مشابه زیر باشد:

    ```
    Datacenter: DC1-K8Demo
    ======================
    Status=Up/Down
    |/ State=Normal/Leaving/Joining/Moving
    --  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
    UN  172.17.0.5  83.57 KiB  32           74.0%             e2dd09e6-d9d3-477e-96c5-45094c08db0f  Rack1-K8Demo
    UN  172.17.0.4  101.04 KiB  32           58.8%             f89d6835-3a42-4419-92b3-0e62cae1479c  Rack1-K8Demo
    UN  172.17.0.6  84.74 KiB  32           67.1%             a6a1e8c2-3dc5-4417-b1a0-26507af2aaad  Rack1-K8Demo
    ```

## اصلاح StatefulSet Cassandra

استفاده از `kubectl edit` برای اصلاح اندازه یک StatefulSet Cassandra.

1. اجرای دستور زیر:

    ```shell
    kubectl edit statefulset cassandra
    ```

    این دستور یک ویرایشگر در ترمینال شما باز می‌کند. خطی که باید تغییر دهید، فیلد `replicas` است. نمونه زیر از فایل StatefulSet است:

    ```yaml
    # لطفاً شی زیر را ویرایش کنید. خطوطی که با '#' شروع می‌شوند نادیده گرفته می‌شوند،
    # و فایل خالی موجب لغو ویرایش می‌شود. اگر خطا اتفاق بیافتد وقتی که این فایل را ذخیره می‌کنید،
    # این فایل با خطاهای مربوطه دوباره باز خواهد شد.
    #
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      creationTimestamp: 2016-08-13T18:40:58Z
      generation: 1
      labels:
      app: cassandra
      name: cassandra
      namespace: default
      resourceVersion: "323"
      uid: 7a219483-6185-11e6-a910-42010a8a0fc0
    spec:
      replicas: 3
    ```

1. تغییر تعداد replicas به 4 و سپس ذخیره کنید.

    حالا StatefulSet به سهولت با 4 Pods اجرا می‌شود.

1. برای تأیید تغییرات خود، StatefulSet Cassandra را دریافت کنید:

    ```shell
    kubectl get statefulset cassandra
    ```

    پاسخ باید مشابه زیر باشد:

    ```
    NAME        DESIRED   CURRENT   AGE
    cassandra   4         4         36m
    ```

## {{% heading "پاکسازی" %}}

حذف یا تنظیم اندازه کم StatefulSet، منجر به حذف حجم‌های دائمی مرتبط با StatefulSet نمی‌شود.
این تنظیمات برای امنیت شماست چرا که داده‌های شما ارزشمندتر از حذف خودکار منابع مربوط به StatefulSet است.

{{< warning >}}
بسته به کلاس ذخیره‌سازی و سیاست بازیابی، حذف *PersistentVolumeClaims* ممکن است منجر به حذف حجم‌های مرتبط نیز شود.
هرگز فرض نکنید که می‌توانید به داده‌ها دسترسی داشته باشید اگر درخواست‌های حجم‌های آن‌ها حذف شوند.
{{< /warning >}}

1. اجرای دستورات زیر (به هم پیوسته به صورت یک دستور واحد) برای حذف همه چیز در StatefulSet Cassandra:

    ```shell
    grace=$(kubectl get pod cassandra-0 -o=jsonpath='{.spec.terminationGracePeriodSeconds}') \
      && kubectl delete statefulset -l app=cassandra \
      && echo "Sleeping ${grace} seconds" 1>&2 \
      && sleep $grace \
      && kubectl delete persistentvolumeclaim -l app=cassandra
    ```

1. اجرای دستور زیر برای حذف سرویسی که برای Cassandra تنظیم کرده‌اید:

    ```shell
    kubectl delete service -l app=cassandra
    ```

## متغیرهای محیطی کانتینر Cassandra

Pods در این آموزش از تصویر [`gcr.io/google-samples/cassandra:v13`](https://github.com/kubernetes/examples/blob/master/cassandra/image/Dockerfile)
از مخازن [container registry](https://cloud.google.com/container-registry/docs/) Google استفاده می‌کنند.
این تصویر Docker بر اساس [debian-base](https://github.com/kubernetes/release/tree/master/images/build/debian-base)
و شامل OpenJDK 8 می‌باشد.

این تصویر شامل نصب استاندارد کاساندرا از مخازن Debian Apache است.
با استفاده از متغیرهای محیطی، می‌توانید مقادیری که در `cassandra.yaml` وارد می‌شوند، تغییر دهید.

| متغیر محیطی            | مقدار پیش‌فرض    |
| ------------------------ |:---------------: |
| `CASSANDRA_CLUSTER_NAME` | `'Test Cluster'` |
| `CASSANDRA_NUM_TOKENS`   | `32`             |
| `CASSANDRA_RPC_ADDRESS`  | `0.0.0.0`        |


## {{% heading "چه_بعدا" %}}

* آموزش [مقیاس کردن StatefulSet](/docs/tasks/run-application/scale-stateful-set/) را بیاموزید.
* بیشتر در مورد [*KubernetesSeedProvider*](https://github.com/kubernetes/examples/blob/master/cassandra/java/src/main/java/io/k8s/cassandra/KubernetesSeedProvider.java) بدانید.
* تنظیمات دیگر [Seed Provider Configurations](https://git.k8s.io/examples/cassandra/java/README.md) را ببینید.
