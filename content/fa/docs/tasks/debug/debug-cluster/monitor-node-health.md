---
title: Monitor Node Health
content_type: task
reviewers:
- Random-Liu
- dchen1107
weight: 20
---

<!-- overview -->

*Node Problem Detector* یک دیمون برای نظارت و گزارش‌دهی دربارهٔ سلامت یک نود است. می‌توانید Node Problem Detector را به عنوان یک `DaemonSet` یا به عنوان یک دیمون مستقل اجرا کنید. Node Problem Detector اطلاعات دربارهٔ مشکلات نود را از دیمون‌های مختلف جمع‌آوری کرده و این شرایط را به سرور API به عنوان [Conditions](/docs/concepts/architecture/nodes/#condition) نود یا به عنوان [Events](/docs/reference/kubernetes-api/cluster-resources/event-v1) گزارش می‌دهد.

برای آموزش نصب و استفاده از Node Problem Detector، به [مستندات پروژه Node Problem Detector](https://github.com/kubernetes/node-problem-detector) مراجعه کنید.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}}

<!-- steps -->

## محدودیت‌ها

* Node Problem Detector برای گزارش مشکلات کرنل از فرمت log کرنل استفاده می‌کند. برای یادگیری نحوهٔ گسترش فرمت log کرنل، به [افزودن پشتیبانی برای یک فرمت log دیگر](#support-other-log-format) مراجعه کنید.

## فعال‌سازی Node Problem Detector

بعضی ارائه‌دهندگان ابر Node Problem Detector را به عنوان {{< glossary_tooltip text="Addon" term_id="addons" >}} فعال می‌کنند. همچنین می‌توانید Node Problem Detector را با استفاده از `kubectl` یا با ایجاد یک Addon DaemonSet فعال کنید.

### استفاده از kubectl برای فعال‌سازی Node Problem Detector {#using-kubectl}

`kubectl` انعطاف‌پذیرترین مدیریت Node Problem Detector را فراهم می‌کند. می‌توانید پیکربندی پیش‌فرض را بازنویسی کنید تا به محیط خودتان بپیوندد یا برای شناسایی مشکلات نود سفارشی شده را شناسایی کنید. به عنوان مثال:

1. یک پیکربندی Node Problem Detector مشابه `node-problem-detector.yaml` ایجاد کنید:

   {{% code_sample file="debug/node-problem-detector.yaml" %}}

   {{< note >}}
   باید مطمئن شوید که دایرکتوری log سیستمی مناسب برای توزیع سیستم عامل شما است.
   {{< /note >}}

1. Node Problem Detector را با استفاده از `kubectl` شروع کنید:

   ```shell
   kubectl apply -f https://k8s.io/examples/debug/node-problem-detector.yaml
   ```

### استفاده از یک Addon pod برای فعال‌سازی Node Problem Detector {#using-addon-pod}

اگر از یک راه‌حل راه‌اندازی خوشه سفارشی استفاده می‌کنید و نیازی به بازنویسی پیکربندی پیش‌فرض ندارید، می‌توانید از Addon pod برای بهینه‌سازی بیشتر استفاده کنید.

پیکربندی `node-problem-detector.yaml` را ایجاد کرده و پیکربندی را در دایرکتوری `/etc/kubernetes/addons/node-problem-detector` روی یک نود کنترل پلن ذخیره کنید.

## بازنویسی پیکربندی

پیکربندی پیش‌فرض هنگام ساخت تصویر Docker Node Problem Detector تعبیه می‌شود.

اما می‌توانید از [`ConfigMap`](/docs/tasks/configure-pod-container/configure-pod-configmap/)
برای بازنویسی پیکربندی استفاده کنید:

1. فایل‌های پیکربندی را در `config/` تغییر دهید.
1. `ConfigMap` `node-problem-detector-config` را ایجاد کنید:

   ```shell
   kubectl create configmap node-problem-detector-config --from-file=config/
   ```

1. فایل `node-problem-detector.yaml` را برای استفاده از `ConfigMap` تغییر دهید:

   {{% code_sample file="debug/node-problem-detector-configmap.yaml" %}}

1. Node Problem Detector را با فایل پیکربندی جدید مجدداً ایجاد کنید:

   ```shell
   # اگر Node Problem Detector در حال اجرا است، قبل از بازسازی حذف کنید
   kubectl delete -f https://k8s.io/examples/debug/node-problem-detector.yaml
   kubectl apply -f https://k8s.io/examples/debug/node-problem-detector-configmap.yaml
   ```

{{< note >}}
این رویکرد فقط برای Node Problem Detectorی که با `kubectl` شروع شده است، قابل اجرا است.
{{< /note >}}

بازنویسی پیکربندی پشتیبانی نمی‌شود اگر Node Problem Detector به عنوان یک افزونه خوشه اجرا شود. مدیر Addon پشتیبانی نمی‌کند از `ConfigMap`.

## دیمون‌های مشکل

یک دیمون مشکل، یک زیردیمون از Node Problem Detector است. این دیمون‌ها انواع مختلفی از مشکلات نود را نظارت می‌کنند و آن‌ها را به Node Problem Detector گزارش می‌دهند.
چندین نوع دیمون مشکل پشتیبانی می‌شود.

- نوع `SystemLogMonitor` از دیمون، لاگ‌های سیستم را نظارت می‌کند و مشکلات و متریک‌ها را براساس قوانین پیش‌فرض گزارش می‌دهد. می‌توانید پیکربندی‌های خود را برای منابع لاگ مختلفی مانند [filelog](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/config/kernel-monitor-filelog.json)،
  [kmsg](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/config/kernel-monitor.json)،
  [kernel](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/config/kernel-monitor-counter.json)،
  [abrt](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/config/abrt-adaptor.json)،
  و [systemd](https://github.com/kubernetes

/node-problem-detector/blob/v0.8.12/config/systemd-monitor-counter.json)
  سفارشی کنید.

- نوع `SystemStatsMonitor` از دیمون، انواع مختلفی از آمارهای سیستمی مربوط به سلامت را به عنوان متریک جمع‌آوری می‌کند. می‌توانید با به روزرسانی فایل
  [پیکربندی](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/config/system-stats-monitor.json)
  رفتار آن را سفارشی کنید.

- نوع `CustomPluginMonitor` از دیمون، با اجرای اسکریپت‌های تعریف شده توسط کاربر، مشکلات مختلف نود را فراخوانی و بررسی می‌کند. می‌توانید از نوع‌های مختلفی از مانیتورهای پلاگین سفارشی برای نظارت بر مشکلات مختلف استفاده کنید و با به روزرسانی
  [پیکربندی](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/config/custom-plugin-monitor.json)
  رفتار دیمون را سفارشی کنید.

- نوع `HealthChecker` از دیمون، سلامتی kubelet و runtime container را در یک نود بررسی می‌کند.

### افزودن پشتیبانی برای فرمت log دیگر {#support-other-log-format}

مانیتور لاگ سیستم در حال حاضر از لاگ‌های مبتنی بر فایل، journald و kmsg پشتیبانی می‌کند. منابع اضافی می‌توانند با اجرای یک نظارت جدید
[watcher log](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/pkg/systemlogmonitor/logwatchers/types/log_watcher.go)
پیوسته شوند.

### اضافه کردن مانیتورهای پلاگین سفارشی

می‌توانید Node Problem Detector را با توسعه یک پلاگین سفارشی برای اجرای هر اسکریپت نظارتی که به هر زبانی نوشته شده باشد، گسترش دهید. اسکریپت‌های نظارتی باید با پروتکل پلاگین در خروجی کد و خروجی استاندارد سازگار باشند. برای کسب اطلاعات بیشتر، لطفاً به
[پیشنهاد رابط پلاگین](https://docs.google.com/document/d/1jK_5YloSYtboj-DtfjmYKxfNnUxCAvohLnsH5aGCAYQ/edit#)
مراجعه کنید.

## Exporter

یک ارسال‌کننده، مشکلات نود و/یا متریک‌ها را به برخی پشتیبانها گزارش می‌دهد.
ارسال‌کننده‌های زیر پشتیبانی می‌شوند:

- **Kubernetes exporter**: این ارسال‌کننده مشکلات نود را به سرور API Kubernetes گزارش می‌دهد. مشکلات موقت به عنوان رویدادها و مشکلات دائمی به عنوان شرایط نود گزارش می‌شوند.

- **Prometheus exporter**: این ارسال‌کننده مشکلات نود و متریک‌ها را به صورت محلی به عنوان متریک‌های Prometheus
  (یا OpenMetrics) گزارش می‌دهد. می‌توانید آدرس IP و پورت برای ارسال‌کننده را با استفاده از آرگومان‌های خط فرمان مشخص کنید.

- **Stackdriver exporter**: این ارسال‌کننده مشکلات نود و متریک‌ها را به API مانیتورینگ Stackdriver
  گزارش می‌دهد. رفتار ارسال را می‌توان با استفاده از یک
  [پیکربندی فایل](https://github.com/kubernetes/node-problem-detector/blob/v0.8.12/config/exporter/stackdriver-exporter.json)
  سفارشی کرد.

<!-- discussion -->

## پیشنهادات و محدودیت‌ها

توصیه می‌شود Node Problem Detector را در خوشه‌ی خود اجرا کنید تا سلامت نود را نظارت کنید. هنگام اجرای Node Problem Detector، انتظار است که هزینهٔ منابع اضافی برای هر نود وجود داشته باشد.
معمولاً این کار درست است، زیرا:

* log کرنل به نسبت آرام رشد می‌کند.
* محدودیت منابع برای Node Problem Detector تنظیم شده است.
* حتی در بار بالا، مصرف منابع قابل قبول است. برای کسب اطلاعات بیشتر، به
  [نتیجهٔ مقایسه Node Problem Detector](https://github.com/kubernetes/node-problem-detector/issues/2#issuecomment-220255629)
  مراجعه کنید.
