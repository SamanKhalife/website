---
title: مدیریت کلاستر
reviewers:
- davidopp
- lavalamp
weight: 100
content_type: concept
description: >
  جزئیات سطح پایین مرتبط با ایجاد یا مدیریت یک کلاستر Kubernetes.
no_list: true
card:
  name: setup
  weight: 60
  anchors:
  - anchor: "#securing-a-cluster"
    title: امنیت یک کلاستر
---

<!-- overview -->

مرور مدیریت کلاستر برای هر کسی است که در حال ایجاد یا مدیریت یک کلاستر Kubernetes است.
این پیش‌نیازهایی را فرض می‌کند که با [مفاهیم اصلی Kubernetes](/docs/concepts/) آشنا هستید.

<!-- body -->

## برنامه‌ریزی یک کلاستر

برای مشاهده راهنماها در [Setup](/docs/setup/)، برای مثال‌هایی از نحوه برنامه‌ریزی، راه‌اندازی و پیکربندی کلاسترهای Kubernetes. راه‌حل‌های معرفی شده در این مقاله به نام *دیستروها* نامیده می‌شوند.

{{< note  >}}
همه دیستروها به طور فعال نگهداری نمی‌شوند. دیستروهایی را که با یک نسخه جدید از Kubernetes تست شده‌اند، انتخاب کنید.
{{< /note >}}

قبل از انتخاب یک راهنما، در نظر داشته باشید:

- آیا می‌خواهید Kubernetes را روی کامپیوتر خود امتحان کنید، یا می‌خواهید یک کلاستر چند-نود با پایداری بالا بسازید؟ دیستروهایی را که برای نیازهای شما مناسب‌تر هستند، انتخاب کنید.
- آیا از **یک کلاستر Kubernetes میزبان شده** مانند
  [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) استفاده می‌کنید، یا **کلاستر خود را میزبان می‌کنید**؟
- آیا کلاستر شما **on-premises** است، یا **در cloud (IaaS)** قرار دارد؟ Kubernetes مستقیماً از کلاسترهای هیبریدی پشتیبانی نمی‌کند. به جای آن، می‌توانید چندین کلاستر راه‌اندازی کنید.
- **اگر شما Kubernetes را on-premises پیکربندی می‌کنید**، در نظر داشته باشید کدام
  [مدل شبکه](/docs/concepts/cluster-administration/networking/) بهترین است.
- آیا Kubernetes را بر روی سخت‌افزار **"bare metal"** یا روی **ماشین‌های مجازی (VMs)** اجرا می‌کنید؟
- آیا **می‌خواهید یک کلاستر را اجرا کنید**، یا انتظار دارید که **توسعه فعال کد پروژه Kubernetes** انجام دهید؟ اگر گزینه دوم را انتخاب می‌کنید، یک دیستروی فعالانه‌تر انتخاب کنید. برخی از دیستروها فقط از انتشارهای باینری استفاده می‌کنند، اما انتخاب‌های بیشتری ارائه می‌دهند.
- با [اجزاء](/docs/concepts/overview/components/) مورد نیاز برای اجرای یک کلاستر آشنا شوید.

## مدیریت یک کلاستر

* یاد بگیرید که چگونه [نودها را مدیریت کنید](/docs/concepts/architecture/nodes/).
  * درباره [اتواسکیلینگ کلاستر](/docs/concepts/cluster-administration/cluster-autoscaling/) بخوانید.

* یاد بگیرید که چگونه محدودیت منابع را برای کلاسترهای اشتراکی [راه‌اندازی و مدیریت کنید](/docs/concepts/policy/resource-quotas/).

## امنیت یک کلاستر

* [تولید گواهی‌نامه‌ها](/docs/tasks/administer-cluster/certificates/) مراحل تولید گواهی‌نامه با استفاده از زنجیره ابزارهای مختلف را شرح می‌دهد.

* [محیط کانتینر Kubernetes](/docs/concepts/containers/container-environment/) محیط کانتینرهای مدیریت شده توسط Kubelet را در یک Node Kubernetes شرح می‌دهد.

* [کنترل دسترسی به API Kubernetes](/docs/concepts/security/controlling-access) توضیح می‌دهد که Kubernetes چگونه کنترل دسترسی را برای API خود اجرا می‌کند.

* [احراز هویت](/docs/reference/access-authn-authz/authentication/) مفصل احراز هویت در Kubernetes را توضیح می‌دهد، از جمله گزینه‌های مختلف احراز هویت.

* [اجازه‌دهی](/docs/reference/access-authn-authz/authorization/) جدا از احراز هویت است، و کنترل می‌کند که چگونه تماس‌های HTTP را اداره می‌کند.

* [استفاده از Admission Controllers](/docs/reference/access-authn-authz/admission-controllers/)
  پلاگین‌هایی را توضیح می‌دهد که درخواست‌ها را به سمت سرور API Kubernetes پس از احراز هویت و اجازه‌دهی می‌گیرد.

* [استفاده از Sysctls در یک کلاستر Kubernetes](/docs/tasks/administer-cluster/sysctl-cluster/)
  به یک مدیر نشان می‌دهد که چگونه از ابزار خط فرمان `sysctl` برای تنظیم پارامترهای kernel استفاده کند.

* [آدیتینگ](/docs/tasks/debug/debug-cluster/audit/) توضیح می‌دهد که چگونه با لاگ‌های آدیت Kubernetes تعامل کنید.

### امنیت kubelet

* [ارتباط صفحه کنترل-نود](/docs/concepts/architecture/control-plane-node-communication/)
*

 [بوت استرپ TLS](/docs/reference/access-authn-authz/kubelet-tls-bootstrapping/)
* [احراز هویت/اجازه‌دهی kubelet](/docs/reference/access-authn-authz/kubelet-authn-authz/)

## خدمات اختیاری کلاستر

* [یکپارچگی DNS](/docs/concepts/services-networking/dns-pod-service/) توضیح می‌دهد که چگونه می‌توانید یک نام DNS را به طور مستقیم به یک سرویس Kubernetes حل کنید.

* [لاگ‌گذاری و نظارت بر فعالیت کلاستر](/docs/concepts/cluster-administration/logging/)
  توضیح می‌دهد که چگونه لاگ‌گذاری در Kubernetes کار می‌کند و چگونه آن را پیاده‌سازی کنید.
