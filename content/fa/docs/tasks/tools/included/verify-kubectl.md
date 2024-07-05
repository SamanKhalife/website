---
title: "تأیید نصب kubectl"
description: "چگونگی تأیید نصب kubectl."
headless: true
_build:
  list: never
  render: never
  publishResources: false
---

برای اینکه kubectl بتواند به یک خوشه Kubernetes دسترسی پیدا کند و با آن ارتباط برقرار کند، نیاز به یک
[kubeconfig فایل](/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
دارد که به طور خودکار ایجاد می‌شود هنگامی که با استفاده از
[kube-up.sh](https://github.com/kubernetes/kubernetes/blob/master/cluster/kube-up.sh)
یا موفقیت آمیز دپلوی کردن خوشه Minikube خلق می‌شود.
پیش‌فرضاً تنظیمات kubectl در مسیر `~/.kube/config` قرار دارد.

برای بررسی صحیح بودن تنظیمات kubectl، وضعیت خوشه را دریافت کنید:

```shell
kubectl cluster-info
```

اگر پاسخی URL مشاهده کردید، kubectl به درستی پیکربندی شده است تا به خوشه شما دسترسی داشته باشد.

اگر پیامی مشابه زیر مشاهده کردید، به این معنی است که kubectl به درستی پیکربندی نشده است یا قادر به اتصال به یک خوشه Kubernetes نیست:

```
The connection to the server <server-name:port> was refused - did you specify the right host or port?
```

برای مثال، اگر قصد دارید یک خوشه Kubernetes را روی لپ‌تاپ خود اجرا کنید (محلی)، ابتدا نیاز به یک ابزار مانند [Minikube](https://minikube.sigs.k8s.io/docs/start/) دارید و سپس دستورات فوق را دوباره اجرا کنید.

اگر `kubectl cluster-info` پاسخ url داد اما به خوشه خود دسترسی ندارید، برای بررسی صحیح بودن تنظیمات، از دستور زیر استفاده کنید:

```shell
kubectl cluster-info dump
```

### رفع مشکل پیام خطای 'No Auth Provider Found' {#no-auth-provider-found}

در Kubernetes 1.26، kubectl احراز هویت داخلی را برای ارائه‌دهندگان مدیریت شده Kubernetes ابری زیر حذف کرد. این ارائه‌دهندگان افزونه‌های kubectl را منتشر کرده‌اند
برای اطلاعات بیشتر، به مستندات ارائه‌دهنده زیر مراجعه کنید:

* Azure AKS: [افزونه kubelogin](https://azure.github.io/kubelogin/)
* Google Kubernetes Engine: [افزونه gke-gcloud-auth](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl#install_plugin)

(ممکن است دلایل دیگری نیز وجود داشته باشد که به دلیل تغییراتی ناخواسته با این پیام خطا روبرو شوید.)
