---
reviewers:
- derekwaynecarr
title: مدیریت صفحات بزرگ (HugePages)
content_type: task
description: پیکربندی و مدیریت صفحات بزرگ به عنوان یک منبع قابل برنامه‌ریزی در یک خوشه.
---

<!-- مرور -->
{{< feature-state feature_gate_name="HugePages" >}}

Kubernetes امکان تخصیص و مصرف صفحات بزرگ قبل از تخصیص شده توسط برنامه‌ها در یک Pod را پشتیبانی می‌کند. این صفحه توضیح می‌دهد که کاربران چگونه می‌توانند صفحات بزرگ را مصرف کنند.

## {{% heading "پیش‌نیازها" %}}

برای اینکه یک Node در Kubernetes توانایی گزارش دادن ظرفیت صفحات بزرگ خود را داشته باشد، باید
[huge pages](https://www.kernel.org/doc/html/latest/admin-guide/mm/hugetlbpage.html)
را از پیش تخصیص دهد.

یک Node می‌تواند برای اندازه‌های مختلف صفحه بزرگ (HugePages) را پیش از آن تخصیص دهد، به عنوان مثال، خط زیر در `/etc/default/grub` `2*1GiB` از صفحات 1 GiB و `512*2 MiB` از صفحات 2 MiB را تخصیص می‌دهد:

```
GRUB_CMDLINE_LINUX="hugepagesz=1G hugepages=2 hugepagesz=2M hugepages=512"
```

نود‌ها به‌طور خودکار تمام منابع صفحه بزرگ را کشف و گزارش می‌دهند.

وقتی شما Node را توصیف می‌کنید، باید موارد مشابه زیر را در بخش‌های `Capacity` و `Allocatable` ببینید:

```
Capacity:
  cpu:                ...
  ephemeral-storage:  ...
  hugepages-1Gi:      2Gi
  hugepages-2Mi:      1Gi
  memory:             ...
  pods:               ...
Allocatable:
  cpu:                ...
  ephemeral-storage:  ...
  hugepages-1Gi:      2Gi
  hugepages-2Mi:      1Gi
  memory:             ...
  pods:               ...
```

{{< note >}}
برای صفحات پویا (پس از بوت)، Kubelet باید برای اعلام‌های جدید بازنشسته شود.
{{< /note >}}

<!-- مراحل -->

## API

صفحات بزرگ می‌توانند با استفاده از نیازهای منابع سطح کانتینر با نام منبع `hugepages-<size>` مصرف شوند، جایی که `<size>` نمایش کوتاه‌ترین دستور یکنواخت با استفاده از مقادیر عددی پشتیبانی شده در یک Node خاص است. به عنوان مثال، اگر یک Node از اندازه‌های 2048KiB و 1048576KiB پشتیبانی کند، منابع قابل برنامه‌ریزی `hugepages-2Mi` و `hugepages-1Gi` را نمایش می‌دهد. بر خلاف CPU یا حافظه، صفحات بزرگ از تخصیص بیش از حد پشتیبانی نمی‌کنند. توجه کنید که هنگام درخواست منابع صفحه بزرگ، باید منابع حافظه یا CPU نیز درخواست شود.

یک Pod می‌تواند از اندازه‌های مختلف صفحه بزرگ در یک مشخصه Pod تقاضا کند. در این صورت باید از نمایش `medium: HugePages-<hugepagesize>` برای تمام اتصالات حجم استفاده کند.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: huge-pages-example
spec:
  containers:
  - name: example
    image: fedora:latest
    command:
    - sleep
    - inf
    volumeMounts:
    - mountPath: /hugepages-2Mi
      name: hugepage-2mi
    - mountPath: /hugepages-1Gi
      name: hugepage-1gi
    resources:
      limits:
        hugepages-2Mi: 100Mi
        hugepages-1Gi: 2Gi
        memory: 100Mi
      requests:
        memory: 100Mi
  volumes:
  - name: hugepage-2mi
    emptyDir:
      medium: HugePages-2Mi
  - name: hugepage-1gi
    emptyDir:
      medium: HugePages-1Gi
```

یک Pod می‌تواند فقط از `medium: HugePages` استفاده کند اگر فقط از یک اندازه صفحه بزرگ درخواست کند.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: huge-pages-example
spec:
  containers:
  - name: example
    image: fedora:latest
    command:
    - sleep
    - inf
    volumeMounts:
    - mountPath: /hugepages
      name: hugepage
    resources:
      limits:
        hugepages-2Mi: 100Mi
        memory: 100Mi
      requests:
        memory: 100Mi
  volumes:
  - name: hugepage
    emptyDir:
      medium: HugePages
```

- درخواست صفحات بزرگ باید با محدودیت‌ها برابر باشد. این پیش‌فرض است اگر محدودیت‌ها مشخص شده باشند، اما درخواست‌ها نشده باشند.
- صفحات بزرگ در دامنه کانتینر منعزل می‌شوند، بنابراین هر کانتینر محدودیت خود را در صندوق cgroup خود به عنوان درخواست در یک مشخصه کانتینر دارد.
- اسکیل‌های EmptyDir که توسط صفحات بزرگ پشتیبانی می‌شوند، نمی‌توانند بیشتر از حافظه صفحه بزرگ درخواست شده توسط Pod مصرف کنند.
- برنامه‌هایی که از طریق `shmget()` با `SHM_HUGETLB` صفحات بزرگ را مصرف می‌کنند، باید با گروه تکمیلی که با `proc/sys/vm/hugetlb_shm_group` همخوانی دارد، اجرا شوند.
- استفاده از صفحات بزرگ در یک فضای نام قابل کنترل با استفاده از ResourceQuota، مشابه با دیگر منابع محاسباتی مانند `cpu` یا `memory` با استف

اده از token `hugepages-<size>` قابل مدیریت است.
