---
title: اتوماسیون مقیاس خوشه
linkTitle: اتوماسیون مقیاس خوشه
description: >
  مدیریت خودکار گره‌های خوشه برای سازگاری با تقاضا.
content_type: concept
weight: 120
---

<!-- overview -->

Kubernetes نیاز به {{< glossary_tooltip text="گره" term_id="node" >}} در خوشه شما برای اجرای {{< glossary_tooltip text="پاد" term_id="pod" >}} دارد. این به این معنی است که باید ظرفیت کافی برای پادهای بارکار و برای خود Kubernetes فراهم شود.

می‌توانید مقدار منابع موجود در خوشه خود را به طور خودکار تنظیم کنید: اتوماسیون مقیاس گره. می‌توانید تعداد گره‌ها را تغییر دهید یا ظرفیتی که گره‌ها ارائه می‌دهند را تغییر دهید. رویکرد اول به عنوان مقیاس افقی شناخته می‌شود، در حالی که رویکرد دوم به عنوان مقیاس عمودی شناخته می‌شود.

Kubernetes حتی می‌تواند مقیاس‌پذیری اتوماتیک چند بعدی برای گره‌ها فراهم کند.

<!-- body -->

## مدیریت دستی گره‌ها

می‌توانید ظرفیت سطح گره را به صورت دستی مدیریت کنید، جایی که شما یک مقدار ثابت از گره‌ها را پیکربندی می‌کنید؛ می‌توانید از این روش استفاده کنید حتی اگر فراهم‌سازی (فرآیند تنظیم، مدیریت و خارج کردن) برای این گره‌ها به صورت خودکار باشد.

این صفحه درباره گام بعدی است و مدیریت اتوماتیک مقدار ظرفیت گره (CPU، حافظه و دیگر منابع گره) موجود در خوشه شما است.

## اتوماسیون افقی خودکار {#autoscaling-horizontal}

### اتوماسیون مقیاس خوشه

شما می‌توانید از [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler) برای مدیریت مقیاس گره‌های خود به صورت خودکار استفاده کنید.
Cluster Autoscaler می‌تواند با ارائه دهنده ابر، یا با [API خوشه](https://github.com/kubernetes/autoscaler/blob/c6b754c359a8563050933a590f9a5dece823c836/cluster-autoscaler/cloudprovider/clusterapi/README.md) Kubernetes ادغام شود تا مدیریت واقعی گره‌ها که نیاز است، انجام دهد.

Cluster Autoscaler گره‌ها را زمانی اضافه می‌کند که پادهای قابل برنامه‌ریزی نباشند، و
گره‌ها را زمانی حذف می‌کند که این گره‌ها خالی باشند.

#### ادغام‌های ارائه دهنده ابر {#cluster-autoscaler-providers}

[README](https://github.com/kubernetes/autoscaler/tree/c6b754c359a8563050933a590f9a5dece823c836/cluster-autoscaler#readme)
برای Cluster Autoscaler فهرستی از ادغام‌های ارائه دهنده ابر را فهرست می‌کند که در دسترس هستند.

## اتوماسیون مقیاس چند بعدی حساس به هزینه {#autoscaling-multi-dimension}

### Karpenter {#autoscaler-karpenter}

[Karpenter](https://karpenter.sh/) از طریق افزونه‌هایی که با ارائه دهنده‌های ابر خاص ادغام می‌شود، مدیریت مستقیم گره‌ها را پشتیبانی می‌کند و می‌تواند بهینه‌سازی برای هزینه کلی انجام دهد.

> Karpenter به طور خودکار منابع محاسباتی مناسب را برای برنامه‌های خوشه شما راه‌اندازی می‌کند. این طراحی شده است تا به شما اجازه دهد از ابر با فرآیند سریع و ساده ارائه محاسبات برای خوشه‌های Kubernetes بهره ببرید.

ابزار Karpenter برای ادغام با ارائه دهنده ابر طراحی شده است که اطلاعات قیمت برای سرورهای موجود از طریق یک API وب نیز در دسترس باشد.

به عنوان مثال، اگر چندین پاد دیگر در خوشه خود راه‌اندازی کنید، ابزار Karpenter ممکن است یک گره جدیدی را خریداری کند که بزرگتر از یکی از گره‌هایی باشد که در حال استفاده است، و سپس یک گره موجود را خاموش کند.

#### ادغام‌های ارائه دهنده ابر {#karpenter-providers}

{{% thirdparty-content vendor="true" %}}

ادغام‌هایی بین هسته Karpenter و ارائه دهندگان ابر زیر موجود هستند:

- [Amazon Web Services](https://github.com/aws/karpenter-provider-aws)
- [Azure](https://github.com/Azure/karpenter-provider-azure)


## اجزاء مرتبط

### Descheduler

[Descheduler](https://github.com/kubernetes-sigs/descheduler) می‌تواند به شما کمک کند
پادها را بر روی تعداد کمتری از گره‌ها یکپارچه کنید، تا در مقیاس خودکار کاهش پیدا کند
زمانی که خوشه ظرفیت فضای خود را دارد.

### اندازه‌گیری بار کاری بر اساس اندازه خوشه

#### اتوماسیون مقیاس متناسب خوشه

برای بارهای کاری که باید بر اساس اند

ازه خوشه مقیاس داده شوند (به عنوان مثال
`cluster-dns` یا سایر اجزاء سیستمی)، می‌توانید از
[_Cluster Proportional Autoscaler_](https://github.com/kubernetes-sigs/cluster-proportional-autoscaler)
استفاده کنید.<br />

Cluster Proportional Autoscaler تعداد گره‌های برنامه‌ریزی شده و هسته‌ها را نظارت می‌کند
و تعداد رپلیک‌های بارکار هدف را مطابق با آن مقیاس می‌دهد.

#### اتوماسیون مقیاس عمودی متناسب خوشه

اگر تعداد رپلیک‌ها باید ثابت بماند، می‌توانید بارهای کاری خود را بر اساس اندازه خوشه با استفاده از
[_Cluster Proportional Vertical Autoscaler_](https://github.com/kubernetes-sigs/cluster-proportional-vertical-autoscaler)
مقیاس دهید. این پروژه در **بتا** است و می‌توان آن را در GitHub پیدا کرد.

در حالی که Cluster Proportional Autoscaler تعداد رپلیک‌های بارکار را مقیاس می‌دهد، Cluster Proportional Vertical Autoscaler
درخواست منابع برای بارکاری (مانند یک Deployment یا DaemonSet) را بر اساس تعداد گره‌ها و/یا هسته‌ها
در خوشه تنظیم می‌کند.

## {{% heading "whatsnext" %}}

- درباره [اتوماسیون مقیاس بارکاری](/docs/concepts/workloads/autoscaling/) بخوانید
