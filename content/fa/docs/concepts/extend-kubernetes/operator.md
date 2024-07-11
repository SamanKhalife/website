---
title: الگوی اپراتور
content_type: concept
weight: 30
---

<!-- overview -->

اپراتورها افزونه‌های نرم‌افزاری برای Kubernetes هستند که از
[منابع سفارشی](/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
برای مدیریت برنامه‌ها و اجزای آن‌ها استفاده می‌کنند. اپراتورها از اصول Kubernetes پیروی می‌کنند، به ویژه
[حلقه کنترل](/docs/concepts/architecture/controller).

<!-- body -->

## انگیزه

الگوی _اپراتور_ هدف اصلی یک اپراتور انسانی که مدیریت یک سرویس یا مجموعه‌ای از سرویس‌ها را بر عهده دارد، به تصویر می‌کشد. اپراتورهای انسانی که از برنامه‌ها و سرویس‌های خاصی مراقبت می‌کنند، دانش عمیقی از نحوه رفتار سیستم، نحوه راه‌اندازی آن، و نحوه واکنش به مشکلات دارند.

افرادی که بارهای کاری را بر روی Kubernetes اجرا می‌کنند، اغلب دوست دارند از اتوماسیون برای انجام وظایف تکراری استفاده کنند. الگوی اپراتور چگونگی نوشتن کد برای خودکارسازی وظیفه‌ای فراتر از آنچه که Kubernetes خود ارائه می‌دهد را به تصویر می‌کشد.

## اپراتورها در Kubernetes

Kubernetes برای اتوماسیون طراحی شده است. به صورت پیش‌فرض، بسیاری از اتوماسیون‌های داخلی را از هسته Kubernetes دریافت می‌کنید. می‌توانید از Kubernetes برای خودکارسازی راه‌اندازی و اجرای بارهای کاری استفاده کنید، *و* می‌توانید نحوه انجام این کار توسط Kubernetes را خودکار کنید.

مفهوم {{< glossary_tooltip text="الگوی اپراتور" term_id="operator-pattern" >}} در Kubernetes به شما امکان می‌دهد رفتار خوشه را بدون تغییر کد Kubernetes خود توسعه دهید، با لینک دادن {{< glossary_tooltip text="کنترل‌کننده‌ها" term_id="controller" >}} به یک یا چند منبع سفارشی. اپراتورها مشتریان API Kubernetes هستند که به عنوان کنترل‌کننده برای یک [منبع سفارشی](/docs/concepts/extend-kubernetes/api-extension/custom-resources/) عمل می‌کنند.

## یک مثال از اپراتور {#example}

برخی از مواردی که می‌توانید با استفاده از یک اپراتور خودکار کنید شامل:

* راه‌اندازی برنامه در صورت تقاضا
* گرفتن و بازگرداندن پشتیبان‌های وضعیت برنامه
* مدیریت به‌روزرسانی‌های کد برنامه همراه با تغییرات مرتبط مانند طرح‌های پایگاه داده یا تنظیمات اضافی
* انتشار یک سرویس به برنامه‌هایی که از APIهای Kubernetes برای کشف آن‌ها پشتیبانی نمی‌کنند
* شبیه‌سازی خرابی در همه یا بخشی از خوشه برای آزمایش مقاومت آن
* انتخاب یک رهبر برای یک برنامه توزیع شده بدون یک فرایند انتخاب داخلی اعضا

یک اپراتور در جزئیات ممکن است به چه شکلی باشد؟ در اینجا یک مثال است:

1. یک منبع سفارشی به نام SampleDB که می‌توانید در خوشه پیکربندی کنید.
2. یک استقرار که اطمینان می‌دهد یک Pod در حال اجرا است که شامل قسمت کنترل‌کننده اپراتور است.
3. یک تصویر کانتینری از کد اپراتور.
4. کد کنترل‌کننده که به کنترل‌پلین برای یافتن منابع SampleDB پیکربندی شده است.
5. هسته اپراتور کدی است که به سرور API می‌گوید چگونه واقعیت را با منابع پیکربندی شده تطبیق دهد.
   * اگر یک SampleDB جدید اضافه کنید، اپراتور PersistentVolumeClaims را برای ارائه ذخیره‌سازی پایدار دیتابیس، یک StatefulSet برای اجرای SampleDB و یک Job برای مدیریت پیکربندی اولیه ایجاد می‌کند.
   * اگر آن را حذف کنید، اپراتور یک اسنپ‌شات می‌گیرد و سپس اطمینان حاصل می‌کند که StatefulSet و Volumeها نیز حذف می‌شوند.
6. اپراتور همچنین مدیریت پشتیبان‌های منظم دیتابیس را انجام می‌دهد. برای هر منبع SampleDB، اپراتور تعیین می‌کند که چه زمانی یک Pod ایجاد شود که می‌تواند به دیتابیس متصل شود و پشتیبان بگیرد. این Pods به یک ConfigMap و / یا یک Secret که جزئیات اتصال به دیتابیس و مدارک هویتی را دارند، تکیه می‌کنند.
7. از آنجا که اپراتور به دنبال ارائه اتوماسیون قوی برای منبعی است که مدیریت می‌کند، کد پشتیبانی اضافی نیز وجود دارد. برای این مثال، کد بررسی می‌کند که آیا دیتابیس در حال اجرای نسخه قدیمی است و در صورت چنین، اشیای Job ایجاد می‌کند که آن را برای شما به‌روزرسانی کنند.

## راه‌اندازی اپراتورها

متداول‌ترین راه برای راه‌اندازی یک اپراتور، افزودن
تعریف منبع سفارشی و کنترل‌کننده متناظر آن به خوشه شما است.
کنترل‌کننده معمولاً خارج از
{{< glossary_tooltip text="کنترل‌پلین" term_id="control-plane" >}} اجرا می‌شود،
همانند اجرای هر برنامه کانتینری دیگر.
برای مثال، می‌توانید کنترل‌کننده را در خوشه خود به عنوان یک استقرار اجرا کنید.

## استفاده از یک اپراتور {#using-operators}

هنگامی که یک اپراتور راه‌اندازی شد، با افزودن، ویرایش یا حذف نوع منبعی که اپراتور استفاده می‌کند از آن استفاده می‌کنید. به دنبال مثال بالا، شما یک استقرار برای خود اپراتور تنظیم می‌کنید، و سپس:

```shell
kubectl get SampleDB                   # یافتن دیتابیس‌های پیکربندی شده

kubectl edit SampleDB/example-database # تغییر دستی برخی تنظیمات
```

و همین! اپراتور وظیفه اعمال تغییرات و همچنین حفظ سرویس موجود را بر عهده خواهد داشت.

## نوشتن اپراتور خودتان {#writing-operator}

اگر اپراتوری در اکوسیستم وجود ندارد که رفتار مورد نظر شما را پیاده‌سازی کند، می‌توانید خودتان آن را کد نویسی کنید. 

همچنین می‌توانید یک اپراتور (یعنی یک کنترل‌کننده) را با استفاده از هر زبان / زمان اجرایی که می‌تواند به عنوان [مشتری API Kubernetes](/docs/reference/using-api/client-libraries/) عمل کند، پیاده‌سازی کنید.

در ادامه چند کتابخانه و ابزار را معرفی می‌کنیم که می‌توانید برای نوشتن اپراتور بومی ابری خود از آن‌ها استفاده کنید.

{{% thirdparty-content %}}

* [Charmed Operator Framework](https://juju.is/)
* [Java Operator SDK](https://github.com/java-operator-sdk/java-operator-sdk)
* [Kopf](https://github.com/nolar/kopf) (فریمورک اپراتور Pythonic Kubernetes)
* [kube-rs](https://kube.rs/) (زبان Rust)
* [kubebuilder](https://book.kubebuilder.io/)
* [KubeOps](https://buehler.github.io/dotnet-operator-sdk/) (SDK اپراتور .NET)
* [Mast](https://docs.ansi.services/mast/user_guide/operator/)
* [Metacontroller](https://metacontroller.github.io/metacontroller/intro.html) همراه با WebHookهایی که خودتان پیاده‌سازی می‌کنید
* [Operator Framework](https://operatorframework.io)
* [shell-operator](https://github.com/flant/shell-operator)

## {{% heading "whatsnext" %}}


* مطالعه کنید [مقاله سفید اپراتور](https://github.com/cncf/tag-app-delivery/blob/163962c4b1cd70d085107fc579e3e04c2e14d59c/operator-wg/whitepaper/Operator-WhitePaper_v1-0.md) از {{< glossary_tooltip text="CNCF" term_id="cncf" >}}.
* اطلاعات بیشتر درباره [منابع سفارشی](/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
* اپراتورهای آماده را بر روی [OperatorHub.io](https://operatorhub.io/) بیابید که برای مورد استفاده شما مناسب باشد
* [انتشار](https://operatorhub.io/) اپراتور خود را برای دیگران
* مطالعه کنید [مقاله اصلی CoreOS](https://web.archive.org/web/20170129131616/https://coreos.com/blog/introducing-operators.html) که الگوی اپراتور را معرفی کرد (این نسخه آرشیوشده از مقاله اصلی است).
* مطالعه کنید [مقاله](https://cloud.google.com/blog/products/containers-kubernetes/best-practices-for-building-kubernetes-operators-and-stateful-apps) از Google Cloud درباره بهترین شیوه‌ها برای ساخت اپراتورها