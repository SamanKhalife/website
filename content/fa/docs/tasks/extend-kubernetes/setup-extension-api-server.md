---
title: راه‌اندازی یک سرور API Extension
reviewers:
- lavalamp
- cheftako
- chenopis
content_type: task
weight: 15
---

<!-- overview -->

راه‌اندازی یک سرور API Extension برای کار با لایه تجمیع به Kubernetes اجازه می‌دهد تا apiserver Kubernetes با APIs اضافی که قسمت اصلی APIs Kubernetes نیستند، گسترش یابد.


## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

* باید [لایه تجمیع](/docs/tasks/extend-kubernetes/configure-aggregation-layer/) را پیکربندی کنید و پرچم‌های apiserver را فعال کنید.


<!-- steps -->

## راه‌اندازی یک سرور API Extension برای کار با لایه تجمیع

مراحل زیر توضیح می‌دهد که چگونه یک extension-apiserver را به صورت سطح بالا راه‌اندازی کنید. این مراحل به طور مشترک برای استفاده از پیکربندی‌های YAML یا استفاده از APIs اعمال می‌شود. برای مثال کنکرت این مراحل با استفاده از پیکربندی‌های YAML، می‌توانید به [sample-apiserver](https://github.com/kubernetes/sample-apiserver/blob/master/README.md) در مخزن Kubernetes مراجعه کنید.

همچنین، می‌توانید از یک راه‌حل شخص ثالث مانند [apiserver-builder](https://github.com/kubernetes-sigs/apiserver-builder-alpha/blob/master/README.md) استفاده کنید که برای شما یک قابلیت اولیه ایجاد می‌کند و همه مراحل زیر را به صورت خودکار برای شما انجام می‌دهد.

1. اطمینان حاصل کنید که API APIService فعال است (با استفاده از `--runtime-config` بررسی کنید). این باید به طور پیش‌فرض فعال باشد مگر اینکه به صورت متعمد در خوشه شما غیرفعال شده باشد.
1. ممکن است نیاز به ایجاد یک قانون RBAC برای اضافه کردن اشیاء APIService داشته باشید یا مدیر خوشه خود را برای انجام این کار درخواست دهید. (از آنجایی که گسترش APIs تأثیر گسترده‌ای بر کل خوشه دارد، توصیه نمی‌شود که امتحان، توسعه یا اشکال‌زدایی یک گسترش API را در یک خوشه زنده انجام دهید.)
1. فضای نام Kubernetes را که می‌خواهید extension api-service خود را در آن اجرا کنید، ایجاد کنید/به آن دسترسی پیدا کنید.
1. یک گواهی CA برای استفاده در امضای گواهی سروری که extension api-server برای HTTPS استفاده می‌کند، ایجاد/دریافت کنید.
1. یک گواهی سروری/کلید برای استفاده از api-server برای HTTPS ایجاد کنید. این گواهی باید توسط CA بالا امضا شود. همچنین باید یک CN از نام DNS Kube داشته باشد. این از طریق خدمت Kubernetes ایجاد شده است و به فرم `<نام سرویس>.<فضای نام سرویس>.svc` است.
1. یک secret Kubernetes با گواهی/کلید سرور در فضای نام خود ایجاد کنید.
1. یک deployment Kubernetes برای extension api-server ایجاد کنید و اطمینان حاصل کنید که شما دارای بارگذاری secret به عنوان یک واحد هستید. این باید حاوی یک مرجع به تصویر کار extension api-server شما باشد. همچنین deployment باید در فضای نام شما باشد.
1. اطمینان حاصل کنید که extension-apiserver شما این گواهی‌ها را از آن حجم بارگذاری می‌کند و از آنها در handshake HTTPS استفاده می‌کند.
1. یک حساب سرویس Kubernetes در فضای نام خود ایجاد کنید.
1. یک cluster role Kubernetes برای عملیاتی که می‌خواهید روی منابع خود اجازه دهید، ایجاد کنید.
1. یک cluster role binding Kubernetes از حساب سرویس در فضای نام خود به cluster role ایجاد شده توسط شما.
1. یک cluster role binding Kubernetes از حساب سرویس در فضای نام خود به cluster role `system:auth-delegator` ایجاد کنید تا تصمیمات احراز هویت را به سرور هسته API Kubernetes منتقل کنید.
1. یک role binding Kubernetes از حساب سرویس در فضای نام خود به نقش `extension-apiserver-authentication-reader` ایجاد کنید. این به extension api-server شما اجازه می‌دهد که به configmap `extension-apiserver-authentication` دسترسی داشته باشد.
1. یک apiservice Kubernetes ایجاد کنید. گواهی CA بالا باید به صورت Base64 کد شده، بدون خطوط جدید استفاده شود و در spec.caBundle در apiservice استفاده شود. این نباید دارای فضای نام باشد. اگر از [kube-aggregator API](https://github.com/kubernetes/kube-aggregator/) استفاده می‌کنید، فقط گواهی CA پیکربندی PEM را بدهید زیرا کدگذاری Base64 برای شما انجام می‌شود.
1. از kubectl برای دریافت منابع خود استفاده کنید. در صورت اجرا، kubectl باید پیام "No resources found." را بازگرداند. این پیام نشان دهنده آن است که همه چیز کار کرده است اما در حال حاضر شما هیچ اشیاء از آن نوع منابع ایجاد نشده است.


## {{% heading "مراحل بعدی" %}}

* مرور مراحل برای [پیکربندی لایه تجمیع API](/docs/tasks/extend-kubernetes/configure-aggregation-layer/) و فعال کردن پرچم‌های apiserver.
* برای مشاهده موجز، [گسترش API Kubernetes با لایه تجمیع](/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/) را بررسی کنید.
* یاد بگیرید که چگونه با استفاده از Custom Resource Definitions [API Kubernetes را گسترش دهید](/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/).
