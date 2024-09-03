reviewers:
- brendandburns
- erictune
- mikedanese
title: شروع کار با Kubernetes
main_menu: true
weight: 20
content_type: concept
no_list: true
card:
  name: setup
  weight: 20
  anchors:
  - anchor: "#learning-environment"
    title: محیط یادگیری
  - anchor: "#production-environment"
    title: محیط تولید
  
<!-- مقدمه -->

این بخش روش‌های مختلف برای نصب و اجرای Kubernetes را لیست می‌کند. هنگام نصب Kubernetes، نوع نصب را بر اساس: آسانی نگهداری، امنیت، کنترل، منابع موجود، و تخصص مورد نیاز برای مدیریت و اجرای یک خوشه انتخاب کنید.

می‌توانید Kubernetes را [دانلود کنید](/releases/download/) تا یک خوشه Kubernetes را بر روی یک ماشین محلی، در ابر، یا برای مرکز داده خود ایجاد کنید.

چندین [مؤلفه Kubernetes](/docs/concepts/overview/components/) مانند {{< glossary_tooltip text="kube-apiserver" term_id="kube-apiserver" >}} یا {{< glossary_tooltip text="kube-proxy" term_id="kube-proxy" >}} همچنین می‌توانند به صورت [تصاویر ظرف‌های کانتینر](/releases/download/#container-images) درون خوشه نصب شوند.

توصیه می‌شود که مؤلفه‌های Kubernetes را همواره به عنوان تصاویر ظرف‌های کانتینر اجرا کنید، و Kubernetes را برای مدیریت این مؤلفه‌ها استفاده کنید.
مؤلفه‌هایی که کانتینرها را اجرا می‌کنند - به خصوص kubelet - نمی‌توانند در این دسته قرار گیرند.

اگر نمی‌خواهید خود یک خوشه Kubernetes را مدیریت کنید، می‌توانید یک سرویس مدیریت شده را انتخاب کنید، از جمله [پلتفرم‌های تاییدشده](/docs/setup/production-environment/turnkey-solutions/) و یا راه‌حل‌های استاندارد و سفارشی در ابرها و محیط‌های فلزی استفاده کنید.

<!-- متن اصلی -->

## محیط یادگیری

اگر در حال یادگیری Kubernetes هستید، از ابزارهایی که توسط جامعه Kubernetes پشتیبانی می‌شوند یا ابزارهای اکوسیستم برای نصب یک خوشه Kubernetes بر روی یک ماشین محلی استفاده کنید.
برای مشاهده ابزارهای [نصب](/docs/tasks/tools/) را ببینید.

## محیط تولید

هنگام ارزیابی یک راه‌حل برای [محیط تولید](/docs/setup/production-environment/)، در نظر داشته باشید که کدام جنبه‌های مدیریت یک خوشه Kubernetes (یا _ابسترکشن‌ها_) را می‌خواهید خودتان مدیریت کنید و کدام را به ارائه‌دهنده منتقل کنید.

برای یک خوشه که خودتان مدیریت می‌کنید، ابزار رسمی برای نصب Kubernetes [kubeadm](/docs/setup/production-environment/tools/kubeadm/) است.

## {{% heading "whatsnext" %}}

- [دانلود Kubernetes](/releases/download/)
- دانلود و [نصب ابزارها](/docs/tasks/tools/) از جمله `kubectl`
- انتخاب [موتور اجرای کانتینر](/docs/setup/production-environment/container-runtimes/) برای خوشه جدید شما
- آموزش [شیوه‌های بهتر](/docs/setup/best-practices/) برای راه‌اندازی خوشه

Kubernetes برای اجرای [صفحه کنترل](/docs/concepts/windows/) خود بر روی Linux طراحی شده است. درون خوشه خود می‌توانید برنامه‌ها را در Linux یا سیستم‌عامل‌های دیگر از جمله Windows اجرا کنید.
