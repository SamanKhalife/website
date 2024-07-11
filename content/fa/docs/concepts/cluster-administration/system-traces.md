---
title: ردیابی‌ها برای مؤلفه‌های سیستم Kubernetes
reviewers:
- logicalhan
- lilic
content_type: concept
weight: 90
---

<!-- مرور -->

{{< feature-state for_k8s_version="v1.27" state="beta" >}}

ردیابی‌های مؤلفه‌های سیستم، زمان‌تأخیر و رابطه بین عملیات‌ها در خوشه را ثبت می‌کنند.

مؤلفه‌های Kubernetes از
[پروتکل OpenTelemetry](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/protocol/otlp.md#opentelemetry-protocol-specification)
با صادرکننده gRPC استفاده می‌کنند و می‌توانند جمع‌آوری و هدایت ردیابی‌ها را به پشتیبان‌های ردیابی با استفاده از
[مجموعه OpenTelemetry](https://github.com/open-telemetry/opentelemetry-collector#-opentelemetry-collector)
انجام دهند.

<!-- متن -->

## جمع‌آوری ردیابی

مؤلفه‌های Kubernetes صادرکننده‌های gRPC داخلی برای OTLP برای صادرکردن ردیابی‌ها دارند، با یک OpenTelemetry Collector یا بدون OpenTelemetry Collector.

برای راهنمای کامل در جمع‌آوری ردیابی‌ها و استفاده از مجموعه‌کننده، به
[شروع با مجموعه‌کننده OpenTelemetry](https://opentelemetry.io/docs/collector/getting-started/)
مراجعه کنید. با این حال، چند نکته وجود دارد که ویژه مؤلفه‌های Kubernetes هستند.

به طور پیش‌فرض، مؤلفه‌های Kubernetes ردیابی‌ها را با استفاده از صادرکننده grpc برای OTLP روی
[پورت IANA OpenTelemetry](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=opentelemetry)، 4317 صادر می‌کنند.
به عنوان مثال، اگر مجموعه‌کننده به عنوان یک پرچم‌کننده به یک مؤلفه Kubernetes نوعی وابسته باشد، پیکربندی اینگونه از ردیابی‌ها را جمع‌آوری کند و آن‌ها را به خروجی استاندارد بفرستد:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
exporters:
  # با جایگزین کردن این صادرکننده با صادرکننده مؤلفه خود
  logging:
    logLevel: debug
service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [logging]
```

برای صرفه‌جویی در منابع و حذف نیاز به یک مجموعه‌کننده، می‌توانید ردیابی‌ها را مستقیماً به یک سرور پشتیبانی شده بفرستید، 
با مشخص کردن فیلد endpoint در فایل پیکربندی ردیابی Kubernetes با آدرس مورد نظر سرور پشتیبانی ردیابی. این روش نیاز به مجموعه‌کننده را نادیده می‌گیرد و ساختار کلی را ساده‌تر می‌کند.

برای پیکربندی هدر پشتیبانی ردیابی، از جزئیات احراز هویت، متغیرهای محیطی با `OTEL_EXPORTER_OTLP_HEADERS` استفاده می‌شود،
جهت اطلاعات بیشتر مشاهده کنید: [پیکربندی صادرکننده OTLP](https://opentelemetry.io/docs/languages/sdk-configuration/otlp-exporter/).

همچنین، برای پیکربندی ویژگی‌های منبع ردیابی مانند نام خوشه Kubernetes، فضای نام، نام Pod، و غیره، همچنین می‌توانید از متغیرهای محیطی با `OTEL_RESOURCE_ATTRIBUTES` استفاده کنید،
جهت مشاهده: [منبع Kubernetes OTLP](https://opentelemetry.io/docs/specs/semconv/resource/k8s/).

## ردیابی‌های مؤلفه‌ها

### ردیابی‌های kube-apiserver

kube-apiserver برای درخواست‌های HTTP ورودی، و برای درخواست‌های خروجی به webhook ها، etcd، و درخواست‌های re-entrant اسپن‌ها تولید می‌کند. این که درخواست‌ها ورودی 
[W3C Trace Context](https://www.w3.org/TR/trace-context/) را با درخواست‌های خروجی ادغام می‌کند، ولی از متن ردیابی متصل به درخواست‌های ورودی استفاده نمی‌کند، 
زیرا kube-apiserver اغلب یک نقطه‌ی انتهای عمومی است.

#### فعال‌سازی ردیابی در kube-apiserver

برای فعال‌سازی ردیابی، یک فایل پیکربندی ردیابی به kube-apiserver ارائه دهید با `--tracing-config-file=<path-to-config>`. این یک نمونه پیکربندی است که اسپن‌ها را برای 1 در 10000 درخواست ضبط می‌کند و از آدرس پیش‌فرض OpenTelemetry استفاده می‌کند:

```yaml
apiVersion: apiserver.config.k8s.io/v1beta1
kind: TracingConfiguration
# default value
#endpoint: localhost:4317
samplingRatePerMillion: 100
```

برای اطلاعات بیشتر درباره struct `TracingConfiguration`، مشاهده کنید:
[API server config API (v1beta1)](/docs/reference/config-api/apiserver-config.v1beta1/#apiserver-k8s-io-v1beta1-TracingConfiguration).

### ردیابی‌های kubelet

{{< feature-state feature_gate_name="KubeletTracing" >}}

رابط CRI kubelet و سرورهای http مجوزدار برای تولید اسپن ردیابی شده‌اند. همانطور که با apiserver اتفاق می‌افتد، ادغام متن ردیابی تنظیم شده‌اند. 
تصمیم نمونه ردیابی از یک اسپن والدین همیشه احترام می‌گذارد.
اگر نرخ نمون

ه‌گیری پیکربندی ردیابی ارائه شود، آن نمونه‌گیری را به اسپن‌ها بدون والدین اعمال می‌کند. فعال بدون یک آدرس گیرنده مجموعه‌کننده، آدرس پیش‌فرض گیرنده مجموعه‌کننده OpenTelemetry با "localhost:4317" تنظیم شده‌است.

#### فعال‌سازی ردیابی در kubelet

برای فعال‌سازی ردیابی، پیکربندی ردیابی را اعمال کنید
[پیکربندی ردیابی](https://github.com/kubernetes/component-base/blob/release-1.27/tracing/api/v1/types.go).
این یک نمونه از تکه‌کد پیکربندی kubelet است که اسپن‌ها را برای 1 در 10000 درخواست ضبط می‌کند و از آدرس پیش‌فرض OpenTelemetry استفاده می‌کند:

```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
featureGates:
  KubeletTracing: true
tracing:
  # default value
  #endpoint: localhost:4317
  samplingRatePerMillion: 100
```

اگر `samplingRatePerMillion` به یک میلیون (`1000000`) تنظیم شود، هر اسپن به صادرکننده ارسال می‌شود.

kubelet در Kubernetes v{{< skew currentVersion >}} اسپن‌ها را از زباله‌برداری، روتین همگام‌سازی پاد، و هر روش gRPC جمع‌آوری می‌کند. kubelet متن ردیابی را با درخواست‌های gRPC ادغام می‌کند تا
توانایی هایی که حاصله از اندازه گیری‌هایی می‌باشد debugging.

لطفا توجه داشته باشید که صادرات اسپن‌ها همواره با یک overhead محدودی برای شبکه و CPU همراه است، وابسته به تنظیمات کلی سیستم. اگر هر مشکلی در خوشه باشد که با فعال‌سازی ردیابی روبرو شود، این مشکل را با کاهش نرخ نمونه‌گیری یا حذف کامل ردیابی با حذف کنفیگ فعال کنید.

## پایداری

ابزار موجودیت‌های زیر فعال همیشه، و شاید می‌تواند تغییر کند در راه‌های مختلف. تا این ویژگی به ثبات کند، هیچ گونه تضمین از مقایسه‌پذیری برای ابزار فعالیت‌های ردیابی وجود ندارد.

## {{% heading "whatsnext" %}}

* [شروع با مجموعه‌کننده OpenTelemetry](https://opentelemetry.io/docs/collector/getting-started/)
* [پیکربندی صادرکننده OTLP](https://opentelemetry.io/docs/languages/sdk-configuration/otlp-exporter/)
