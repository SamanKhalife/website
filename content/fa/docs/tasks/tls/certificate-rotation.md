---
reviewers:
- jcbsmpsn
- mikedanese
title: "پیکربندی چرخه‌ی گواهی برای Kubelet"
content_type: task
---

<!-- overview -->
این صفحه نحوه‌ی فعال‌سازی و پیکربندی چرخه‌ی گواهی برای kubelet را نشان می‌دهد.


{{< feature-state for_k8s_version="v1.19" state="stable" >}}

## {{% heading "prerequisites" %}}

* نسخه‌ی Kubernetes 1.8.0 یا جدیدتر مورد نیاز است.


<!-- steps -->

## مرور کلی

kubelet برای اعتبارسنجی به API Kubernetes از گواهی‌ها استفاده می‌کند. به‌طور پیش‌فرض، این گواهی‌ها با انقضای یک ساله صادر می‌شوند تا نیاز به تجدید نشوند.

در Kubernetes [چرخه‌ی گواهی kubelet](/docs/reference/access-authn-authz/kubelet-tls-bootstrapping/) وجود دارد که به‌طور خودکار یک کلید جدید تولید و یک درخواست گواهی جدید از API Kubernetes در نزدیک شدن به انقضای گواهی فعلی ارسال می‌کند. پس از در دسترس بودن گواهی جدید، برای اعتبارسنجی اتصال‌ها به API Kubernetes استفاده خواهد شد.

## فعال‌سازی چرخه‌ی گواهی مشتری

فرآیند `kubelet` دارای یک آرگومان `--rotate-certificates` است که کنترل می‌کند که آیا kubelet به‌طور خودکار یک گواهی جدید را به عنوان انقضای گواهی در حال استفاده، درخواست می‌کند یا خیر.


فرآیند `kube-controller-manager` دارای یک آرگومان `--cluster-signing-duration` (`--experimental-cluster-signing-duration` قبل از 1.19) است که کنترل می‌کند که چه مدت گواهی‌ها صادر خواهند شد.

## درک پیکربندی چرخه‌ی گواهی

زمانی که یک kubelet راه‌اندازی می‌شود، اگر برای bootstrap کردن تنظیم شده باشد (استفاده از پرچم `--bootstrap-kubeconfig`)، از گواهی اولیه‌اش برای اتصال به API Kubernetes و ارسال یک درخواست امضای گواهی استفاده خواهد کرد. می‌توانید وضعیت درخواست‌های امضای گواهی را با استفاده از دستور زیر بررسی کنید:

```sh
kubectl get csr
```

اولیه، یک درخواست امضای گواهی از kubelet در یک نود وضعیت `Pending` دارد. اگر درخواست امضای گواهی به معیارهای خاصی برسد، توسط مدیر کنترل به‌طور خودکار تأیید می‌شود و سپس یک گواهی امضا شده برای مدت زمان مشخص‌شده توسط پارامتر `--cluster-signing-duration` صادر می‌شود و گواهی امضا شده به درخواست امضای گواهی پیوست می‌شود.

kubelet گواهی امضا شده را از API Kubernetes دریافت می‌کند و آن را در مسیر مشخص شده توسط `--cert-dir` ذخیره می‌کند. سپس kubelet از گواهی جدید برای اتصال به API Kubernetes استفاده خواهد کرد.

در نزدیک شدن به انقضای گواهی امضا شده، kubelet به‌طور خودکار یک درخواست امضای گواهی جدید را از API Kubernetes ارسال خواهد کرد. این ممکن است در هر نقطه‌ای بین 30% تا 10% از زمان باقی‌مانده بر روی گواهی انجام شود. دوباره، مدیر کنترل به‌طور خودکار درخواست گواهی را تأیید خواهد کرد و یک گواهی امضا شده به درخواست امضای گواهی پیوست خواهد شد. سپس kubelet گواهی امضا شده جدید را از API Kubernetes دریافت و آن را در مسیر مشخص شده ذخیره خواهد کرد. سپس اتصالات خود به API Kubernetes را بروزرسانی و مجدداً با استفاده از گواهی جدید به آن‌ها وصل خواهد شد.
```