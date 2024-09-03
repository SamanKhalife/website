---
title: ابزارهای دیگر
reviewers:
- janetkuo
content_type: concept
weight: 150
no_list: true
---

<!-- مرور -->

Kubernetes شامل چندین ابزار است که به شما کمک می‌کنند تا با سیستم Kubernetes کار کنید.

<!-- محتوا -->

## crictl

[`crictl`](https://github.com/kubernetes-sigs/cri-tools) یک رابط خط فرمانی است برای بازرسی و اشکال‌زدایی اجرایگرهای کانتینر سازگار با {{<glossary_tooltip term_id="cri" text="CRI">}}.

## Dashboard

[`Dashboard`](/docs/tasks/access-application-cluster/web-ui-dashboard/)، رابط کاربری مبتنی بر وب Kubernetes، به شما امکان می‌دهد تا برنامه‌های کانتینری‌شده را در یک خوشه Kubernetes ارائه دهید، مشکلات آن‌ها را رفع کنید، و خوشه و منابع آن را مدیریت کنید.

## Helm
{{% thirdparty-content single="true" %}}

[Helm](https://helm.sh/) یک ابزار برای مدیریت بسته‌های منابع پیش‌تنظیم شده Kubernetes است. این بسته‌ها به عنوان _نمودارهای Helm_ شناخته می‌شوند.

از Helm برای:

* پیدا کردن و استفاده از نرم‌افزارهای محبوب بسته‌بندی شده به عنوان نمودارهای Kubernetes
* به اشتراک گذاری برنامه‌های خود به عنوان نمودارهای Kubernetes
* ساختن ساخت‌های قابل تکرار از برنامه‌های Kubernetes شما
* مدیریت هوشمند فایل‌های توصیف Kubernetes شما
* مدیریت انتشارهای بسته‌های Helm استفاده کنید

## Kompose

[`Kompose`](https://github.com/kubernetes/kompose) یک ابزار است که به کاربران Docker Compose کمک می‌کند به Kubernetes منتقل شوند.

از Kompose برای:

* ترجمه یک فایل Docker Compose به اشیاء Kubernetes
* از توسعه Docker محلی به مدیریت برنامه شما از طریق Kubernetes بروید
* تبدیل فایل‌های yaml Docker Compose v1 یا v2 یا [Bundles برنامه‌های توزیع شده](https://docs.docker.com/compose/bundles/)

## Kui

[`Kui`](https://github.com/kubernetes-sigs/kui) یک ابزار GUI است که درخواست‌های معمولی خط فرمان `kubectl` شما را با گرافیک پاسخ می‌دهد.

Kui از درخواست‌های خط فرمان `kubectl` عادی استفاده می‌کند و به جای جداول ASCII، نمایش GUI با جداولی که می‌توانید آن‌ها را مرتب کنید را ارائه می‌دهد.

Kui به شما این امکان را می‌دهد:

* به طور مستقیم روی نام‌های منابع طولانی و خودکار کلیک کنید به جای کپی و paste کردن
* نوشتن دستورات `kubectl` و مشاهده اجرای آن‌ها، گاهی اوقات حتی سریع‌تر از `kubectl` خود
* یک `Job` را پرس و جو کنید و اجرای آن را به عنوان یک نمودار آبشاری نمایش دهید
* از طریق رابط کاربری تبدیل شده، منابع در خوشه خود را مشاهده کنید

## Minikube

[`minikube`](https://minikube.sigs.k8s.io/docs/) یک ابزار است که یک خوشه Kubernetes تک‌نودی را به صورت محلی بر روی کارگاه کاری شما اجرا می‌کند برای اهداف توسعه و آزمایش.

