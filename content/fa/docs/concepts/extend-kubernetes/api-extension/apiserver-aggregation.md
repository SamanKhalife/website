---
title: لایه تجمیع API Kubernetes
reviewers:
- lavalamp
- cheftako
- chenopis
content_type: concept
weight: 20
---

<!-- overview -->

لایه تجمیع به Kubernetes اجازه می‌دهد که با API‌های اضافی، خارج از آنچه که توسط API‌های اصلی Kubernetes ارائه می‌شود، گسترش یابد.
API‌های اضافی می‌توانند شامل راه‌حل‌های آماده مانند
[متریک سرور](https://github.com/kubernetes-sigs/metrics-server) باشند یا API‌هایی که خودتان توسعه می‌دهید.

لایه تجمیع از
[منابع سفارشی](/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
که یک روش برای این است که
{{< glossary_tooltip term_id="kube-apiserver" text="kube-apiserver" >}}
قادر به شناسایی انواع جدید شیء باشد، متفاوت است.

<!-- body -->

## لایه تجمیع

لایه تجمیع در پردازه با kube-apiserver اجرا می‌شود. تا زمانی که یک منبع توسعه ثبت نشود، لایه تجمیع هیچ کاری انجام نمی‌دهد. برای ثبت یک API، شما یک شیء _APIService_ را اضافه می‌کنید که مسیر URL را در API Kubernetes "در اختیار می‌گیرد". در آن نقطه، لایه تجمیع هر چیزی را که به آن مسیر API ارسال شود (مانند `/apis/myextension.mycompany.io/v1/…`) به APIService ثبت شده پراکسی می‌کند.

روش معمول برای پیاده‌سازی APIService، اجرای یک *سرور API توسعه* در Pod(s) است که در کلاستر شما اجرا می‌شود. اگر از سرور API توسعه برای مدیریت منابع در کلاسترتان استفاده می‌کنید، سرور API توسعه (همچنین نوشته می‌شود به عنوان "extension-apiserver") معمولاً با یک یا چند {{< glossary_tooltip text="کنترل‌کننده" term_id="controller" >}} هماهنگ می‌شود. کتابخانه apiserver-builder یک چارچوب ابتدایی برای هر دو سرور API توسعه و کنترل‌کننده(ها) مرتبط فراهم می‌کند.

### تاخیر پاسخ

سرورهای API توسعه باید شبکه با تاخیر پایینی به و از kube-apiserver داشته باشند. درخواست‌های کشفی نیاز به دوره بازگشتی حداکثر پنج ثانیه از kube-apiserver دارند.

اگر سرور API توسعه شما نتواند این نیاز به تاخیر را تأمین کند، در نظر داشته باشید تغییراتی ایجاد کنید که به شما اجازه دهد آن را تأمین کنید.

## {{% heading "whatsnext" %}}

* برای راه‌اندازی تجمیع‌گر در محیط خود، [لایه تجمیع را پیکربندی کنید](/docs/tasks/extend-kubernetes/configure-aggregation-layer/).
* سپس، [راه‌اندازی یک سرور API توسعه](/docs/tasks/extend-kubernetes/setup-extension-api-server/) را برای کار با لایه تجمیع انجام دهید.
* در مورد [APIService](/docs/reference/kubernetes-api/cluster-resources/api-service-v1/) در مرجع API بیشتر بخوانید.

یا به صورت جایگزین: یاد بگیرید که چگونه با استفاده از [تعریف‌های منبع سفارشی](/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) API Kubernetes را گسترش دهید.
