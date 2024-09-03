---
reviewers:
- janetkuo
title: انجام بروزرسانی پیشروی بر روی یک DaemonSet
content_type: task
weight: 10
---

<!-- overview -->
این صفحه نحوه انجام عملیات بروزرسانی پیشروی بر روی یک {{< glossary_tooltip term_id="daemonset" >}} را نشان می دهد.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}}

<!-- steps -->

## استراتژی بروزرسانی DaemonSet

DaemonSet دارای دو نوع استراتژی بروزرسانی است:

* `OnDelete`: با استراتژی بروزرسانی `OnDelete`، بعد از بروزرسانی قالب DaemonSet، پادهای جدید DaemonSet فقط زمانی ایجاد می شوند که شما پادهای قدیمی DaemonSet را به صورت دستی حذف کنید. این رفتار مشابه DaemonSet در نسخه‌های Kubernetes 1.5 یا قبلی است.
* `RollingUpdate`: این استراتژی بروزرسانی پیش‌فرض است. با استراتژی بروزرسانی `RollingUpdate`، بعد از بروزرسانی قالب یک DaemonSet، پادهای قدیمی DaemonSet به صورت کنترل شده از بین می روند و پادهای جدید DaemonSet به صورت خودکار ایجاد می شوند. حداکثر یک پاد از DaemonSet در هر گره در طول فرآیند کل بروزرسانی در حال اجرا خواهد بود.

## انجام بروزرسانی پیشروی

برای فعال کردن ویژگی بروزرسانی پیشروی یک DaemonSet، باید `.spec.updateStrategy.type` آن را به `RollingUpdate` تنظیم کنید.

ممکن است بخواهید مقادیر زیر را نیز تنظیم کنید:
[`.spec.updateStrategy.rollingUpdate.maxUnavailable`](/docs/reference/kubernetes-api/workload-resources/daemon-set-v1/#DaemonSetSpec) 
(پیش‌فرض 1)،
[`.spec.minReadySeconds`](/docs/reference/kubernetes-api/workload-resources/daemon-set-v1/#DaemonSetSpec)
(پیش‌فرض 0) و
[`.spec.updateStrategy.rollingUpdate.maxSurge`](/docs/reference/kubernetes-api/workload-resources/daemon-set-v1/#DaemonSetSpec)
(پیش‌فرض 0).

### ایجاد یک DaemonSet با استراتژی بروزرسانی `RollingUpdate`

این فایل YAML یک DaemonSet را با استراتژی بروزرسانی به عنوان 'RollingUpdate' مشخص می کند.

{{% code_sample file="controllers/fluentd-daemonset.yaml" %}}

پس از تأیید استراتژی بروزرسانی قالب DaemonSet، DaemonSet را ایجاد کنید:

```shell
kubectl create -f https://k8s.io/examples/controllers/fluentd-daemonset.yaml
```

همچنین، اگر قصد دارید DaemonSet را با `kubectl apply` به روز کنید، از دستور زیر استفاده کنید:

```shell
kubectl apply -f https://k8s.io/examples/controllers/fluentd-daemonset.yaml
```

### بررسی استراتژی بروزرسانی `RollingUpdate` در DaemonSet

بررسی کنید که استراتژی بروزرسانی DaemonSet شما تنظیم شده است به `RollingUpdate`:

```shell
kubectl get ds/fluentd-elasticsearch -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' -n kube-system
```

اگر شما هنوز DaemonSet را در سیستم ایجاد نکرده‌اید، به جای این دستور، با دستور زیر، مانیفست DaemonSet خود را بررسی کنید:

```shell
kubectl apply -f https://k8s.io/examples/controllers/fluentd-daemonset.yaml --dry-run=client -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}'
```

خروجی از هر دو دستور باید به صورت زیر باشد:

```
RollingUpdate
```

اگر خروجی `RollingUpdate` نبود، به عقب برگردید و موردی که در شیء یا مانیفست DaemonSet تغییر داده‌اید را مطابقت دهید.

### به روزرسانی قالب یک DaemonSet

هرگونه به روزرسانی به `.spec.template` یک DaemonSet با استراتژی `RollingUpdate`، باعث شروع یک بروزرسانی پیشروی می‌شود. به روزرسانی DaemonSet را با اعمال یک فایل YAML جدید انجام دهید. این می‌تواند با چندین دستور `kubectl` مختلف انجام شود.

{{% code_sample file="controllers/fluentd-daemonset-update.yaml" %}}

#### دستورات اظهاری

اگر شما می‌خواهید DaemonSets را با استفاده از
[فایل‌های پیکربندی](/docs/tasks/manage-kubernetes-objects/declarative-config/)
به‌روزرسانی کنید، از `kubectl apply` استفاده کنید:

```shell
kubectl apply -f https://k8s.io/examples/controllers/fluentd-daemonset-update.yaml
```

#### دستورات امپراتیوی

اگر شما می‌خواهید DaemonSets را با استفاده از
[دستورات امپراتیوی](/docs/tasks/manage-kubernetes-objects/imperative-command/)
به‌روزرسانی کنید، از `kubectl edit` استفاده کنید:

```shell
kubectl edit ds/fluentd-elasticsearch -n kube-system
```

##### فقط به‌روزرسانی تصویر ظاهری

اگر شما فقط نیاز به به‌روزرسانی تصویر کانتینر در قالب DaemonSet دارید، یعنی
`.spec.template.spec.containers[*].image`، از `kubectl set image` استفاده کنید:

```shell
kubectl set image ds/fluentd-elasticsearch fluentd-elasticsearch=quay.io/fluentd_elasticsearch/fluentd:v2.6.0 -n kube-system
```

### نظارت بر وضعیت بروزرسانی پیشروی

سرانجام، وضعیت انتشار آخرین بروزرسانی پیشروی DaemonSet را نظارت کنید:

```shell
kubectl rollout status ds/fluentd-elasticsearch -n kube-system
```

وقتی که بروزرسانی کامل شود، خروجی مشابه زیر خواهد بود:

```shell
daemonset "fluentd-elasticsearch" با موفقیت انتشار یافت
```

## رفع مشکلات

### عقب ماندن بروزرسانی پیشروی DaemonSet

گاهی اوقات، یک بروزرسانی پیش

روی DaemonSet ممکن است عقب بماند. چندین علت احتمالی وجود دارد:

#### برخی از گره‌ها از منابع خالی شده‌اند

با گره‌هایی که پادهای DaemonSet را ندارند، از طریق مقایسه خروجی `kubectl get nodes` و خروجی:

```shell
kubectl get pods -l name=fluentd-elasticsearch -o wide -n kube-system
```

پیدا کنید. بعد از یافتن این گره‌ها، برخی از پادهای غیر-DaemonSet را از گره حذف کنید تا جایگاه پادهای جدید DaemonSet را فراهم کنید.

{{< note >}}
این باعث قطع خدمات می‌شود که پادهای حذف شده توسط کنترل‌کننده‌های کنترل نمی‌شوند یا پادها متمایل نیستند.
{{< /note >}}

#### انتشار شکسته

اگر بروزرسانی قالب جدید DaemonSet شکسته باشد، به عنوان مثال، کانتینر به سرعت از بین می رود، یا تصویر کانتینر وجود ندارد (معمولاً به دلیل تایپو)، بروزرسانی DaemonSet پیش نخواهد رفت.

برای رفع این مشکل، قالب DaemonSet را دوباره به‌روزرسانی کنید. بروزرسانی جدید توسط بروزرسانی‌های ناکامی قبلی مسدود نمی‌شود.

#### اختلاف ساعت

اگر `.spec.minReadySeconds` در DaemonSet مشخص شده باشد، اختلاف ساعت بین مستر و گره‌ها باعث می‌شود که DaemonSet قادر به شناسایی پیشرفت مناسب بروزرسانی نباشد.

## پاکسازی

حذف DaemonSet از یک فضای نام:

```shell
kubectl delete ds fluentd-elasticsearch -n kube-system
```

## {{% heading "whatsnext" %}}

* به [انجام rollback بر روی یک DaemonSet](/docs/tasks/manage-daemon/rollback-daemon-set/) مراجعه کنید
* به [ایجاد یک DaemonSet برای پذیرش پادهای موجود DaemonSet](/docs/concepts/workloads/controllers/daemonset/) مراجعه کنید
