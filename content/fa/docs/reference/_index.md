---
title: مرجع
approvers:
- chenopis
linkTitle: "مرجع"
main_menu: true
weight: 70
content_type: concept
no_list: true
---

<!-- مرور -->

این بخش از مستندات Kubernetes شامل مراجع است.

<!-- محتوا -->

## مرجع API

* [لغت‌نامه](/docs/reference/glossary/) - لیست جامع و استاندارد اصطلاحات Kubernetes

* [مرجع API Kubernetes](/docs/reference/kubernetes-api/)
* [مرجع API یک صفحه‌ای برای Kubernetes {{< param "version" >}}](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)
* [استفاده از API Kubernetes](/docs/reference/using-api/) - مروری بر API برای Kubernetes.
* [کنترل دسترسی API](/docs/reference/access-authn-authz/) - جزئیات درباره نحوه کنترل دسترسی API Kubernetes
* [برچسب‌ها، آنوتیشن‌ها و Taints شناخته‌شده](/docs/reference/labels-annotations-taints/)

## کتابخانه‌های مشتری پشتیبانی شده رسمی

برای فراخوانی API Kubernetes از زبان برنامه‌نویسی، می‌توانید از
[کتابخانه‌های مشتری](/docs/reference/using-api/client-libraries/) استفاده کنید. کتابخانه‌های مشتری پشتیبانی شده رسمی عبارتند از:

- [کتابخانه مشتری Go Kubernetes](https://github.com/kubernetes/client-go/)
- [کتابخانه مشتری Python Kubernetes](https://github.com/kubernetes-client/python)
- [کتابخانه مشتری Java Kubernetes](https://github.com/kubernetes-client/java)
- [کتابخانه مشتری JavaScript Kubernetes](https://github.com/kubernetes-client/javascript)
- [کتابخانه مشتری C# Kubernetes](https://github.com/kubernetes-client/csharp)
- [کتابخانه مشتری Haskell Kubernetes](https://github.com/kubernetes-client/haskell)

## CLI

* [kubectl](/docs/reference/kubectl/) - ابزار CLI اصلی برای اجرای دستورات و مدیریت خوشه‌های Kubernetes.
  * [JSONPath](/docs/reference/kubectl/jsonpath/) - راهنمایی نحوه استفاده از عبارات [JSONPath](https://goessner.net/articles/JsonPath/) با kubectl.
* [kubeadm](/docs/reference/setup-tools/kubeadm/) - ابزار CLI برای آسان کردن تامین یک خوشه امن Kubernetes.

## اجزا

* [kubelet](/docs/reference/command-line-tools-reference/kubelet/) - عامل اصلی که در هر نود اجرا می‌شود. kubelet مجموعه‌ای از PodSpecs را می‌گیرد
  و اطمینان می‌یابد که کانتینرهای توصیف شده در حال اجرا و سالم هستند.
* [kube-apiserver](/docs/reference/command-line-tools-reference/kube-apiserver/) -
  REST API که داده‌ها را برای اشیاء API مانند پادها، خدمات، کنترل‌کننده‌های تکثیر اعتبارسنجی و پیکربندی می‌کند.
* [kube-controller-manager](/docs/reference/command-line-tools-reference/kube-controller-manager/) -
  دیمونی که حلقه‌های کنترل اصلی که با Kubernetes ارسال شده‌اند را درون خود جای داده است.
* [kube-proxy](/docs/reference/command-line-tools-reference/kube-proxy/) - می‌تواند هدایت ساده جریان TCP/UDP یا هدایت TCP/UDP گرداننده از مجموعه ای از پشتیبانان را انجام دهد.
* [kube-scheduler](/docs/reference/command-line-tools-reference/kube-scheduler/) -
  برنامه‌ریزی که مدیریت دسترسی، عملکرد و ظرفیت را انجام می‌دهد.

  * [سیاست‌های برنامه‌ریزی](/docs/reference/scheduling/policies)
  * [پروفایل‌های برنامه‌ریزی](/docs/reference/scheduling/config#profiles)

* لیست [پورت‌ها و پروتکل‌ها](/docs/reference/networking/ports-and-protocols/) که باید در صفحه کنترل و نود‌های کارگر باز باشد

## API‌های پیکربندی

این بخش مستندات API‌های "بیرون از انتشار" را که برای پیکربندی اجزا یا ابزارهای Kubernetes استفاده می‌شود، میزبانی می‌کند. بیشتر این API‌ها به طور رسمی توسط API سرور به صورت RESTful ارائه نمی‌شود، اما برای یک کاربر یا اپراتور اساسی هستند که می‌خواهد خوشه را مدیریت کند یا استفاده کند.

* [kubeconfig (v1)](/docs/reference/config-api/kubeconfig.v1/)
* [ادمیسنور پیش‌فرض kube-apiserver (v1)](/docs/reference/config-api/apiserver-admission.v1/)
* [پیکربندی kube-apiserver (v1alpha1)](/docs/reference/config-api/apiserver-config.v1alpha1/) و
* [پیکربندی kube-apiserver (v1beta1)](/docs/reference/config-api/apiserver-config.v1beta1/) و
  [پیکربندی kube-apiserver (v1)](/docs/reference/config-api/apiserver-config.v1/)
* [محدودیت نرخ رویداد kube-apiserver (v1alpha1)](/docs/reference/config-api/apiserver-eventratelimit.v1alpha1/)
* [پیکربندی kubelet (v1alpha1)](/docs/reference/config-api/kubelet-config.v1alpha1/) و
  [پیکربندی kubelet (v1beta1)](/docs/reference/config-api/kubelet-config.v1beta1/)
  [پیکربندی kubelet (v1)](/docs/reference/config-api/kubelet-config.v1/)
* [تهیه کنندگان اعتبار kubelet (v1)](/docs/reference/config-api/kubelet-credentialprovider.v1/)
* [پیکربندی kube-scheduler (v1beta3)](/docs/reference/config-api/kube-scheduler-config.v1beta3/) و
  [پیکربندی kube-scheduler (v1)](/docs/reference/config-api/kube-scheduler-config.v1/)
* [پیکربندی kube-controller-manager (v1alpha1)](/docs/reference/config-api/kube-controller-manager-config.v1alpha1/)
* [پیکربندی kube-proxy (v1alpha1)](/docs/reference/config-api/kube-proxy-config.v1alpha1/)
* [API audit.k8s.io/v1](/docs/reference/config-api/apiserver-audit.v1/)
* [API احراز هویت مشتری (v1beta1)](/docs/reference/config-api/client-authentication.v1beta1/) و
  [API احراز ه

ویت مشتری (v1)](/docs/reference/config-api/client-authentication.v1/)
* [پیکربندی WebhookAdmission (v1)](/docs/reference/config-api/apiserver-webhookadmission.v1/)
* [API ImagePolicy (v1alpha1)](/docs/reference/config-api/imagepolicy.v1alpha1/)

## API پیکربندی برای kubeadm

* [v1beta3](/docs/reference/config-api/kubeadm-config.v1beta3/)
* [v1beta4](/docs/reference/config-api/kubeadm-config.v1beta4/)

## API‌های خارجی

این‌ها API‌هایی هستند که توسط پروژه Kubernetes تعریف شده‌اند، اما توسط پروژه اصلی پیاده‌سازی نمی‌شوند:

* [API متریکس (v1beta1)](/docs/reference/external-api/metrics.v1beta1/)
* [API متریکس سفارشی (v1beta2)](/docs/reference/external-api/custom-metrics.v1beta2)
* [API متریکس خارجی (v1beta1)](/docs/reference/external-api/external-metrics.v1beta1)

## اسناد طراحی

آرشیوی از اسناد طراحی برای قابلیت‌های کاربردی Kubernetes. نقطه شروع خوب‌ای است
[معماری Kubernetes](https://git.k8s.io/design-proposals-archive/architecture/architecture.md) و
[بررسی طراحی Kubernetes](https://git.k8s.io/design-proposals-archive).

