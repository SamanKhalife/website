---
title: ارائه اطلاعات Pod به کانتینرها از طریق فایل‌ها
content_type: task
weight: 40
---

<!-- overview -->

این صفحه نشان می‌دهد که چگونه یک Pod می‌تواند از یک
[`downwardAPI` volume](/docs/concepts/storage/volumes/#downwardapi)
برای ارائه اطلاعات درباره خود به کانتینرهای در حال اجرا در Pod استفاده کند.
یک `downwardAPI` volume می‌تواند فیلدهای Pod و فیلدهای کانتینر را ارائه دهد.

در Kubernetes، دو روش برای ارائه فیلدهای Pod و کانتینر به یک کانتینر در حال اجرا وجود دارد:

* [متغیرهای محیطی](/docs/tasks/inject-data-application/environment-variable-expose-pod-information/)
* فایل‌های ولوم، همانطور که در این وظیفه توضیح داده شده است

به‌طور کلی، این دو روش برای ارائه فیلدهای Pod و کانتینر به Downward API خوانده می‌شود.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}}


<!-- steps -->

## ذخیره فیلدهای Pod

در این قسمت از تمرین، یک Pod ایجاد می‌کنید که یک کانتینر دارد، و شما
فیلدهای سطح Pod را به عنوان فایل‌ها به کانتینر در حال اجرا پروژه می‌دهید.
اینجا مانیفست Pod است:

{{% code_sample file="pods/inject/dapi-volume.yaml" %}}

در مانیفست، می‌بینید که Pod دارای یک Volume `downwardAPI` است،
و کانتینر این Volume را در `/etc/podinfo` mount می‌کند.

در آرایه `items` زیر `downwardAPI` نگاه کنید. هر عنصر از آرایه
یک `downwardAPI` volume را تعریف می‌کند.
عنصر اول مشخص می‌کند که مقدار فیلد `metadata.labels` Pod باید در فایل `labels` ذخیره شود.
عنصر دوم مشخص می‌کند که مقدار فیلد `annotations` Pod باید در فایل `annotations` ذخیره شود.

{{< note >}}
فیلدهای در این مثال فیلدهای Pod هستند و فیلدهای کانتینر در Pod نیستند.
{{< /note >}}

ایجاد Pod:

```shell
kubectl apply -f https://k8s.io/examples/pods/inject/dapi-volume.yaml
```

تأیید کنید که کانتینر در Pod در حال اجرا است:

```shell
kubectl get pods
```

لاگ‌های کانتینر را مشاهده کنید:

```shell
kubectl logs kubernetes-downwardapi-volume-example
```

خروجی محتوای فایل `labels` و `annotations` را نشان می‌دهد:

```
cluster="test-cluster1"
rack="rack-22"
zone="us-est-coast"

build="two"
builder="john-doe"
```

ورود به شل کانتینر در حال اجرا در Pod خود را بگیرید:

```shell
kubectl exec -it kubernetes-downwardapi-volume-example -- sh
```

در شل خود، محتوای فایل `labels` را مشاهده کنید:

```shell
/# cat /etc/podinfo/labels
```

خروجی نشان می‌دهد که تمام برچسب‌های Pod به فایل `labels` نوشته شده است:

```shell
cluster="test-cluster1"
rack="rack-22"
zone="us-est-coast"
```

به طور مشابه، محتوای فایل `annotations` را مشاهده کنید:

```shell
/# cat /etc/podinfo/annotations
```

فایل‌ها را در دایرکتوری `/etc/podinfo` مشاهده کنید:

```shell
/# ls -laR /etc/podinfo
```

در خروجی، مشاهده می‌کنید که فایل‌های `labels` و `annotations`
در یک زیردایرکتوری موقت هستند: در این مثال،
`..2982_06_02_21_47_53.299460680`. در دایرکتوری `/etc/podinfo`، `..data` لینک نمایان به زیردایرکتوری موقت است. همچنین در دایرکتوری `/etc/podinfo`،
`labels` و `annotations` نیز لینک نمایان هستند.

```
drwxr-xr-x  ... Feb 6 21:47 ..2982_06_02_21_47_53.299460680
lrwxrwxrwx  ... Feb 6 21:47 ..data -> ..2982_06_02_21_47_53.299460680
lrwxrwxrwx  ... Feb 6 21:47 annotations -> ..data/annotations
lrwxrwxrwx  ... Feb 6 21:47 labels -> ..data/labels

/etc/..2982_06_02_21_47_53.299460680:
total 8
-rw-r--r--  ... Feb  6 21:47 annotations
-rw-r--r--  ... Feb  6 21:47 labels
```

استفاده از لینک‌های نمایان از به روزرسانی اتمیک پویا اطلاعات فراداده‌ها تحتاج به نوعی به عوض‌کردن [rename(2)](http://man7.org/linux/man-pages/man2/rename.2.html).

{{< note >}}
کانتینری که از Downward API به عنوان یک
[subPath](/docs/concepts/storage/volumes/#using-subpath) بر روی مونتگذاری Downward API نمی‌گیرد.
{{< /note >}}

خروج از شل:

```shell
/# exit
```

## ذخیره فیلدهای کانتینر

در تمرین قبلی، فیلدهای سطح Pod را با استفاده از
Downward API
دسترسی‌پذیر کردید.
در این تمرین بعدی، قصد دارید فیلدهایی که بخشی از تعریف Pod هستند، اما از کانتینر خاص گرفته شده‌اند را انتقال دهید. در اینجا یک مانیفست برای یک Pod که دوباره فقط یک کانتینر دارد، آمده است:

{{% code_sample file="pods/inject/dapi-volume-resources.yaml" %}}

در مانیفست، می‌بینید که Pod دارای یک
`downwardAPI` volume است،
و کانتینر تک در آن Pod این volume را در `/etc/podinfo` mount می‌کند.

به آرایه `items` زیر `downwardAPI` نگاه کنید. هر عنصر از این آرایه
یک فایل در volume downward API را تعریف می‌کند.

عنصر اول مشخص می‌کند که در کانتینری با نام `client-container`،
مقدار فیلد `limits.cpu` با فرمت مشخص شده توسط `1m` باید به عنوان فایل `cpu_limit` منتشر شود. فیلد `divisor` اختیاری است و مقدار پیش‌فرض آن `1` است. یک مقسوم‌کننده `1` برای منابع `cpu` یا بایت برای منابع `memory` است.

ایجاد Pod:

```shell
kubectl apply -f https://k8s.io/examples/pods/inject/dapi-volume-resources.yaml
```

ورود به شل کانتینری که در Pod شما در حال اجرا است:

```shell
kubectl exec -it kubernetes-downwardapi-volume-example-2 -- sh
```

در شل خود، محتوای فایل `cpu_limit` را مشاهده کنید:

```shell
# این دستور را در یک شل داخل کانتینر اجرا کنید
cat /etc/podinfo/cpu_limit
```

می‌توانید از دستورات مشابه برای مشاهده فایل‌های `cpu_request`، `mem_limit` و `mem_request` استفاده کنید.

<!-- discussion -->

## انتقال کلیدها به مسیرها و مجوزهای خاص فایل

می‌توانید کلیدها را به مسیرها و مجوزهای خاص فایل در اساس فایل تنظیم کنید. برای اطلاعات بیشتر، به
[Secrets](/docs/concepts/configuration/secret/)
مراجعه کنید.

## {{% heading "whatsnext" %}}

* مشخصات
[`spec`](/docs/reference/kubernetes-api/workload-resources/pod-v1/#PodSpec)
API برای Pod را بخوانید. این شامل تعریف Container (بخشی از Pod) است.
* لیستی از [فیلدهای موجود](/docs/concepts/workloads/pods/downward-api/#available-fields) را که می‌توانید از طریق Downward API ارائه دهید بخوانید.

درباره ولوم‌ها در مرجع API قدیمی بخوانید:
* API تعریف [`Volume`](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#volume-v1-core) که یک ولوم عمومی را در یک Pod برای دسترسی به کانتینرها تعریف می‌کند.
* API تعریف [`DownwardAPIVolumeSource`](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#downwardapivolumesource-v1-core) که یک ولوم را که حاوی اطلاعات Downward API است، تعریف می‌کند.
* API تعریف [`DownwardAPIVolumeFile`](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#downwardapivolumefile-v1-core) که حاوی مراجع به شیء یا منبع برای پر کردن یک فایل در volume Downward API است.
* API تعریف [`ResourceFieldSelector`](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#resourcefieldselector-v1-core) که منابع کانتینر و فرمت خروجی آنها را مشخص می‌کند.
