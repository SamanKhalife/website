---
title: درباره cgroup v2
content_type: مفهوم
weight: 50
---

<!-- مرور -->

در لینوکس، {{< glossary_tooltip text="control groups" term_id="cgroup" >}} منابعی را که به فرآیندها اختصاص داده می‌شود، محدود می‌کنند.

{{< glossary_tooltip text="kubelet" term_id="kubelet" >}} و runtime کانتینری زیرین نیاز به ارتباط با cgroup دارند تا مدیریت منابع برای پادها و کانتینرها را اعمال کنند که شامل درخواست‌های cpu/memory و محدودیت‌ها برای بارکارهای کانتینری شده است.

در لینوکس دو نسخه از cgroup وجود دارد: cgroup v1 و cgroup v2. cgroup v2 نسل جدید API `cgroup` است.

<!-- محتوا -->


## چیست cgroup v2؟ {#cgroup-v2}
{{< feature-state for_k8s_version="v1.25" state="stable" >}}

cgroup v2 نسخه بعدی از API `cgroup` لینوکس است. cgroup v2 سیستم کنترل یکپارچه‌ای با قابلیت‌های بهبود یافته مدیریت منابع فراهم می‌کند.

cgroup v2 بهبود‌های متعددی نسبت به cgroup v1 دارد، از جمله:

- طراحی سلسله مراتب یکپارچه در API
- اجازه ایمن تر اختصاص زیردرخت به کانتینرها
- ویژگی‌های جدید مانند [Pressure Stall Information](https://www.kernel.org/doc/html/latest/accounting/psi.html)
- مدیریت بهبود یافته تخصیص منابع و جدایی بر روی منابع متعدد
  - حسابداری یکپارچه برای انواع مختلف اختصاصات حافظه (حافظه شبکه، حافظه هسته و غیره)
  - حسابداری برای تغییرات منابع غیر فوری مانند بازنویسی پنجره حافظه

بعضی از ویژگی‌های Kubernetes انحصاراً از cgroup v2 برای مدیریت و جداسازی بهتر منابع استفاده می‌کنند. به عنوان مثال، ویژگی [MemoryQoS](/docs/concepts/workloads/pods/pod-qos/#memory-qos-with-cgroup-v2) کیفیت خدمات حافظه را بهبود می‌بخشد و بر اساس اجزای cgroup v2 اعتماد می‌کند.


## استفاده از cgroup v2 {#using-cgroupv2}

روش توصیه شده برای استفاده از cgroup v2، استفاده از یک توزیع لینوکس است که به طور پیش فرض cgroup v2 را فعال کند و استفاده کند.

برای بررسی اینکه توزیع شما از cgroup v2 استفاده می‌کند، به [شناسایی نسخه cgroup در گره‌های لینوکس](#check-cgroup-version) مراجعه کنید.

### الزامات

cgroup v2 شرایط زیر را دارد:

* توزیع سیستم عامل cgroup v2 را فعال کند
* نسخه هسته لینوکس 5.8 یا جدیدتر باشد
* runtime کانتینر cgroup v2 را پشتیبانی کند. به عنوان مثال:
  * [containerd](https://containerd.io/) نسخه 1.4 و بالاتر
  * [cri-o](https://cri-o.io/) نسخه 1.20 و بالاتر
* kubelet و runtime کانتینر به تنظیم درایور cgroup [systemd](/docs/setup/production-environment/container-runtimes#systemd-cgroup-driver) نیاز دارند

### پشتیبانی از توزیع‌های لینوکس cgroup v2

برای فهرست توزیع‌های لینوکس که از cgroup v2 استفاده می‌کنند، به [مستندات cgroup v2](https://github.com/opencontainers/runc/blob/main/docs/cgroup-v2.md) مراجعه کنید.

<!-- این لیست باید همیشه همگام با https://github.com/opencontainers/runc/blob/main/docs/cgroup-v2.md باشد -->
* Container Optimized OS (از M97 به بعد)
* Ubuntu (از نسخه 21.10، توصیه می‌شود 22.04+)
* Debian GNU/Linux (از Debian 11 bullseye به بعد)
* Fedora (از نسخه 31 به بعد)
* Arch Linux (از آوریل 2021 به بعد)
* توزیع‌های RHEL و مشابه RHEL (از نسخه 9 به بعد)

برای بررسی اینکه توزیع شما از cgroup v2 استفاده می‌کند، به مستندات توزیع خود مراجعه کنید یا دستورات در [شناسایی نسخه cgroup در گره‌های لینوکس](#check-cgroup-version) را دنبال کنید.

همچنین می‌توانید به صورت دستی cgroup v2 را در توزیع لینوکس خود فعال کنید با تغییر آرگومان‌های بوت kernel cmdline. اگر توزیع شما از GRUB استفاده می‌کند، `systemd.unified_cgroup_hierarchy=1` باید در `GRUB_CMDLINE_LINUX` زیر `/etc/default/grub` اضافه شود، و سپس با `sudo update-grub` بروزرسانی شود. با این حال، روش توصیه شده استفاده از یک توزیع است که به طور پیش فرض cgroup v2 را فعال کند.

### مهاجرت به cgroup v2 {#migrating-cgroupv2}

برای مهاجرت به cgroup v2، اطمینان حاصل کنید که شرایط [الزامات](#requirements) را دارید، سپس به یک نسخه هسته که cgroup v2 را به طور پیش فرض فعال می‌کند، ارتقاء دهید.

kubelet به طور خودکار شناسایی می‌کند که سیستم عامل در حال اجرای cgroup v2 است و به موجب آن عملک

رد می‌کند بدون نیاز به پیکربندی اضافی.

تفاوت قابل توجهی در تجربه کاربری وجود نخواهد داشت هنگامی که به cgroup v2 جابجا می‌شود، مگر اینکه کاربران به طور مستقیم به فایل سیستم cgroup دسترسی داشته باشند، یا از داخل کانتینرها.

cgroup v2 از API متفاوتی نسبت به cgroup v1 استفاده می‌کند، بنابراین اگر برنامه‌هایی وجود داشته باشند که به طور مستقیم به فایل سیستم cgroup دسترسی دارند، باید آن‌ها را به نسخه‌های جدیدتری که cgroup v2 را پشتیبانی می‌کنند، به روز رسانی کنید. به عنوان مثال:

* برخی از نهادهای نظارت و امنیت شخص ثالث ممکن است وابستگی به فایل سیستم cgroup داشته باشند. این نهادها را به نسخه‌هایی که cgroup v2 را پشتیبانی می‌کنند به روز رسانی کنید.
* اگر [cAdvisor](https://github.com/google/cadvisor) را به عنوان یک DaemonSet مستقل برای نظارت بر پادها و کانتینرها اجرا می‌کنید، آن را به نسخه v0.43.0 یا بالاتر به روز کنید.
* اگر برنامه‌های جاوا اجرا می‌کنید، از نسخه‌های کاملاً پشتیبانی شده از cgroup v2 استفاده کنید:
    * [OpenJDK / HotSpot](https://bugs.openjdk.org/browse/JDK-8230305): jdk8u372، 11.0.16، 15 و بالاتر
    * [IBM Semeru Runtimes](https://www.ibm.com/support/pages/apar/IJ46681): 8.0.382.0، 11.0.20.0، 17.0.8.0 و بالاتر
    * [IBM Java](https://www.ibm.com/support/pages/apar/IJ46681): 8.0.8.6 و بالاتر
* اگر از بسته [uber-go/automaxprocs](https://github.com/uber-go/automaxprocs) استفاده می‌کنید، اطمینان حاصل کنید که نسخه‌ای که استفاده می‌کنید حداقل v1.5.1 باشد.

## شناسایی نسخه cgroup در گره‌های لینوکس {#check-cgroup-version}

نسخه cgroup به توزیع لینوکس استفاده شده و نسخه cgroup پیش فرض را مشخص می‌کند. برای بررسی اینکه توزیع شما از کدام نسخه cgroup استفاده می‌کند، دستور `stat -fc %T /sys/fs/cgroup/` را در گره اجرا کنید:

```shell
stat -fc %T /sys/fs/cgroup/
```

برای cgroup v2، خروجی `cgroup2fs` است.

برای cgroup v1، خروجی `tmpfs` است.

## {{% heading "whatsnext" %}}

- درباره [cgroups](https://man7.org/linux/man-pages/man7/cgroups.7.html) بیشتر بدانید
- درباره [runtime کانتینر](/docs/concepts/architecture/cri) بیشتر بدانید
- درباره [درایورهای cgroup](/docs/setup/production-environment/container-runtimes#cgroup-drivers) بیشتر بدانید
