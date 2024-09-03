---
title: لیست کردن تمام تصاویر کانتینرهای در حال اجرا در یک کلاستر
content_type: task
weight: 100
---

<!-- overview -->

این صفحه نحوه استفاده از kubectl برای لیست کردن تمام تصاویر کانتینرهایی که در پادها در حال اجرا در یک کلاستر هستند را نشان می‌دهد.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!-- steps -->

در این تمرین شما از kubectl برای گرفتن تمام پادهای در حال اجرا در همه فضاها استفاده خواهید کرد و خروجی را به گونه‌ای فرمت می‌کنید که فقط لیست نام‌های تصاویر کانتینرها را دریافت کند.

## لیست کردن تمام تصاویر کانتینرها در همه فضاها

- گرفتن همه پادها در همه فضاها با استفاده از `kubectl get pods --all-namespaces`
- فرمت کردن خروجی برای شامل تنها لیست نام‌های تصاویر کانتینرها با استفاده از `-o jsonpath={.items[*].spec['initContainers', 'containers'][*].image}`. این دستور به صورت بازگشتی فیلد `image` را از json برمی‌دارد.
  - برای اطلاعات بیشتر درباره استفاده از jsonpath به [مرجع jsonpath](/docs/reference/kubectl/jsonpath/) مراجعه کنید.
- فرمت کردن خروجی با استفاده از ابزارهای استاندارد: `tr`، `sort`، `uniq`
  - استفاده از `tr` برای جایگزینی فضاها با خطوط جدید
  - استفاده از `sort` برای مرتب‌سازی نتایج
  - استفاده از `uniq` برای جمع‌آوری شمارش تصاویر

```shell
kubectl get pods --all-namespaces -o jsonpath="{.items[*].spec['initContainers', 'containers'][*].image}" |\
tr -s '[[:space:]]' '\n' |\
sort |\
uniq -c
```

jsonpath به شکل زیر تفسیر می‌شود:

- `.items[*]`: برای هر مقدار برگشتی
- `.spec`: مشخصات را دریافت کنید
- `['initContainers', 'containers'][*]`: برای هر کانتینر
- `.image`: تصویر را دریافت کنید

{{< note >}}
در دریافت یک پاد تکی بر اساس نام، به عنوان مثال `kubectl get pod nginx`، بخش `.items[*]` از مسیر باید حذف شود زیرا به جای لیست مقادیر، یک پاد تکی برگشت می‌دهد.
{{< /note >}}

## لیست کردن تصاویر کانتینرها بر اساس هر پاد

می‌توان با استفاده از عملیات `range`، فرمت‌دهی را بهبود داد و از عناصر به صورت جداگانه استفاده کرد.

```shell
kubectl get pods --all-namespaces -o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}' |\
sort
```

## لیست کردن تصاویر کانتینرها با فیلتر کردن بر اساس برچسب پاد

برای هدف گذاشتن فقط پادهایی که برچسب خاصی دارند، از پرچم -l استفاده کنید. این دستور تنها پادهایی را با برچسب‌هایی که با `app=nginx` مطابقت دارند، می‌سنجد.

```shell
kubectl get pods --all-namespaces -o jsonpath="{.items[*].spec.containers[*].image}" -l app=nginx
```

## لیست کردن تصاویر کانتینرها با فیلتر کردن بر اساس فضای‌نام پاد

برای هدف گذاشتن تنها پادها در یک فضای‌نام خاص، از پرچم فضای‌نام استفاده کنید. این دستور تنها پادهایی را در فضای‌نام `kube-system` می‌سنجد.

```shell
kubectl get pods --namespace kube-system -o jsonpath="{.items[*].spec.containers[*].image}"
```

## لیست کردن تصاویر کانتینرها با استفاده از قالب go-template به جای jsonpath

به عنوان جایگزین jsonpath، kubectl از [go-templates](https://pkg.go.dev/text/template) برای فرمت‌دهی به خروجی پشتیبانی می‌کند:

```shell
kubectl get pods --all-namespaces -o go-template --template="{{range .items}}{{range .spec.containers}}{{.image}} {{end}}{{end}}"
```

## {{% heading "whatsnext" %}}

### مراجعه

* راهنمای مرجع [Jsonpath](/docs/reference/kubectl/jsonpath/)
* راهنمای مرجع [Go template](https://pkg.go.dev/text/template)
