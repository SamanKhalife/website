---
reviewers:
- derekwaynecarr
- klueska
title: مدیران منابع گره
content_type: concept
weight: 50
---

<!-- overview -->

برای پشتیبانی از بارهای کاری حساس به تاخیر و با توانمندی بالا، Kubernetes مجموعه‌ای از مدیران منابع ارائه می‌دهد. این مدیران به هماهنگی و بهینه‌سازی منابع گره برای Pods‌ها با نیازهای خاص برای منابع CPU، دستگاه‌ها و حافظه (hugepages) می‌پردازند.

<!-- body -->

مدیر اصلی، مدیریت توپولوژی، یک جزء Kubelet است که به‌وسیله‌ی سیاست‌های خود [مدیریت منابع کلی](/docs/tasks/administer-cluster/topology-manager/) را هماهنگ می‌کند.

پیکربندی مدیران انفرادی در اسناد اختصاص‌یافته شده توضیح داده شده است:

- [سیاست‌های مدیریت CPU](/docs/tasks/administer-cluster/cpu-management-policies/)
- [مدیر دستگاه](/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/#device-plugin-integration-with-the-topology-manager)
- [سیاست‌های مدیریت حافظه](/docs/tasks/administer-cluster/memory-manager/)
