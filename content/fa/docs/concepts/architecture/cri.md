---
title: رابط اجرای کانتینر (CRI)
content_type: concept
weight: 60
---

<!-- overview -->

CRI یک رابط افزونه‌ای است که به kubelet امکان استفاده از انواع مختلف
زمان اجرای کانتینر را می‌دهد، بدون اینکه نیازی به بازکامپایل کردن اجزای
کلاستر باشد.

شما به یک
{{<glossary_tooltip text="زمان اجرای کانتینر" term_id="container-runtime">}} کاری بر روی
هر نود در کلاستر خود نیاز دارید، تا
{{< glossary_tooltip text="kubelet" term_id="kubelet" >}} بتواند
{{< glossary_tooltip text="پادها" term_id="pod" >}} و کانتینرهای آن‌ها را راه‌اندازی کند.

{{< glossary_definition prepend="رابط اجرای کانتینر (CRI) است" term_id="container-runtime-interface" length="all" >}}

<!-- body -->

## API {#api}

{{< feature-state for_k8s_version="v1.23" state="stable" >}}

kubelet به عنوان یک کلاینت هنگام اتصال به زمان اجرای کانتینر از طریق gRPC عمل می‌کند.
نقاط پایانی سرویس زمان اجرا و سرویس تصویر باید در زمان اجرای کانتینر موجود باشند،
که می‌توانند به صورت جداگانه در داخل kubelet با استفاده از
فلگ‌های [خط فرمان](/docs/reference/command-line-tools-reference/kubelet)
`--image-service-endpoint` تنظیم شوند.

برای Kubernetes v{{< skew currentVersion >}}، kubelet ترجیح می‌دهد از CRI `v1` استفاده کند.
اگر یک زمان اجرای کانتینر از `v1` CRI پشتیبانی نکند، kubelet سعی می‌کند
هر نسخه پشتیبانی شده قدیمی‌تری را مذاکره کند.
kubelet v{{< skew currentVersion >}} همچنین می‌تواند CRI `v1alpha2` را مذاکره کند، اما
این نسخه به عنوان منسوخ‌شده در نظر گرفته می‌شود.
اگر kubelet نتواند یک نسخه پشتیبانی‌شده CRI را مذاکره کند، kubelet تسلیم شده
و به عنوان یک نود ثبت نمی‌شود.

## ارتقا

هنگام ارتقای Kubernetes، kubelet سعی می‌کند به طور خودکار
آخرین نسخه CRI را در هنگام راه‌اندازی مجدد کامپوننت انتخاب کند. اگر این کار شکست بخورد،
سپس بازگشت به حالت ذکر شده در بالا صورت می‌گیرد. اگر یک بازاتصال gRPC نیاز باشد به دلیل
اینکه زمان اجرای کانتینر ارتقا یافته است، سپس زمان اجرای کانتینر نیز باید
از نسخه اولیه انتخاب‌شده پشتیبانی کند یا انتظار می‌رود که بازاتصال شکست بخورد. این
نیاز به راه‌اندازی مجدد kubelet دارد.

## {{% heading "whatsnext" %}}

- بیشتر درباره تعریف پروتکل CRI بیاموزید [تعریف پروتکل](https://github.com/kubernetes/cri-api/blob/c75ef5b/pkg/apis/runtime/v1/api.proto)
