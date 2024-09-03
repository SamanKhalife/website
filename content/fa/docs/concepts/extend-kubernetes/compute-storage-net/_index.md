---
title: توسعه‌های محاسباتی، ذخیره‌سازی و شبکه
weight: 30
no_list: true
---

این بخش شامل توسعه‌هایی برای خوشه شماست که به عنوان بخشی از Kubernetes عرضه نمی‌شوند. شما می‌توانید از این توسعه‌ها برای بهبود نودهای موجود در خوشه خود یا برای فراهم کردن شبکه‌ای که Pods را به هم متصل می‌کند، استفاده کنید.

* پلاگین‌های ذخیره‌سازی [CSI](/docs/concepts/storage/volumes/#csi) و [FlexVolume](/docs/concepts/storage/volumes/#flexvolume)

  پلاگین‌های {{< glossary_tooltip text="رابط ذخیره‌سازی کانتینر" term_id="csi" >}} (CSI) روشی برای توسعه Kubernetes با پشتیبانی از نوع‌های جدید volumes فراهم می‌کنند. volumes می‌توانند توسط ذخیره‌سازی خارجی پایدار پشتیبانی شوند، یا ذخیره‌سازی موقتی فراهم کنند، یا ممکن است یک رابط فقط خواندنی به اطلاعات با استفاده از یک پارادایم فایل سیستم ارائه دهند.

  Kubernetes همچنین از پلاگین‌های [FlexVolume](/docs/concepts/storage/volumes/#flexvolume) پشتیبانی می‌کند، که از Kubernetes v1.23 (به نفع CSI) منسوخ شده‌اند.

  پلاگین‌های FlexVolume به کاربران اجازه می‌دهند تا نوع‌های volumeی را که به صورت بومی توسط Kubernetes پشتیبانی نمی‌شوند، mount کنند. وقتی شما یک Pod را اجرا می‌کنید که به ذخیره‌سازی FlexVolume متکی است، kubelet یک پلاگین باینری را برای mount کردن volume فراخوانی می‌کند. پیشنهاد طراحی [FlexVolume](https://git.k8s.io/design-proposals-archive/storage/flexvolume-deployment.md) بایگانی شده جزئیات بیشتری در این روش ارائه می‌دهد.

  [پرسش‌های متداول پلاگین Volume Kubernetes برای فروشندگان ذخیره‌سازی](https://github.com/kubernetes/community/blob/master/sig-storage/volume-plugin-faq.md#kubernetes-volume-plugin-faq-for-storage-vendors) شامل اطلاعات عمومی در مورد پلاگین‌های ذخیره‌سازی است.

* [پلاگین‌های دستگاه](/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/)

  پلاگین‌های دستگاه به یک گره اجازه می‌دهند تا منابع جدید Node را (علاوه بر موارد داخلی مانند `cpu` و `memory`) کشف کنند و این امکانات سفارشی node-local را به Podsی که آن‌ها را درخواست می‌کنند، ارائه دهند.

* [پلاگین‌های شبکه](/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)

  یک پلاگین شبکه به Kubernetes اجازه می‌دهد تا با توپولوژی‌ها و فناوری‌های شبکه مختلف کار کند. خوشه Kubernetes شما به یک _پلاگین شبکه_ نیاز دارد تا یک شبکه Pod کارآمد داشته باشد و از سایر جنبه‌های مدل شبکه Kubernetes پشتیبانی کند.

  Kubernetes {{< skew currentVersion >}} با پلاگین‌های شبکه {{< glossary_tooltip text="CNI" term_id="cni" >}} سازگار است.
