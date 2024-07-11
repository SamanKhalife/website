---
title: Kubernetes Scheduler
content_type: concept
weight: 10
---

<!-- overview -->

در Kubernetes، _برنامه‌ریزی_ به معنای اطمینان از این است که {{< glossary_tooltip text="Pods" term_id="pod" >}}
به {{< glossary_tooltip text="Nodes" term_id="node" >}} متناسب اختصاص یابند تا
{{< glossary_tooltip term_id="kubelet" >}} بتواند آن‌ها را اجرا کند.

<!-- body -->

## مرور برنامه‌ریزی {#scheduling}

برنامه‌ریز پیش‌نهادهای ایجاد شده‌ی Podهایی را که هیچ گره‌ای به آن‌ها اختصاص نداده باشد، نظارت می‌کند. برای
هر Pod که برنامه‌ریز کشف می‌کند، برنامه‌ریز مسئول یافتن بهترین گره برای اجرای آن Pod است. برنامه‌ریز
این تصمیم را با در نظر گرفتن اصول برنامه‌ریزی که در زیر توضیح داده شده‌اند، اتخاذ می‌کند.

اگر می‌خواهید بفهمید چرا Podها بر روی یک گره خاص قرار می‌گیرند، یا اگر قصد دارید خودتان یک برنامه‌ریز
سفارشی پیاده‌سازی کنید، این صفحه به شما کمک می‌کند تا در مورد برنامه‌ریزی اطلاعات کسب کنید.

## kube-scheduler

[kube-scheduler](/docs/reference/command-line-tools-reference/kube-scheduler/)
برنامه‌ریز پیش‌فرض برای Kubernetes است و به عنوان بخشی از
{{< glossary_tooltip text="control plane" term_id="control-plane" >}} اجرا می‌شود.
kube-scheduler به گونه‌ای طراحی شده است که اگر بخواهید و نیاز دارید، می‌توانید
یک مولفه برنامه‌ریزی خود را بنویسید و از آن استفاده کنید.

Kube-scheduler یک گره بهینه را برای اجرای Podهای ایجاد شده تاکنون یا هنوز برنامه‌ریزی نشده
(unscheduled) انتخاب می‌کند. از آنجا که ظروف درون Podها - و خود Podها -
می‌توانند نیازهای مختلفی داشته باشند، برنامه‌ریز فیلترهایی را اجرا می‌کند تا هر گره‌ای
که نیازهای خاصی از جنبه‌های برنامه‌ریزی پذیرفتن پذیرفته نکند از بین ببرد. به صورت جایگزین،
API به شما اجازه می‌دهد که یک گره برای یک Pod مشخص کنید زمانی که آن را ایجاد می‌کنید، اما این
مورد نادر و تنها در موارد خاص انجام می‌شود.

در یک خوشه، گره‌هایی که نیازهای برنامه‌ریزی برای یک Pod را برآورده می‌کنند به گره‌های
_قابلیت_ می‌گویند. اگر هیچکدام از گره‌ها مناسب نباشند، Pod تا زمانی که برنامه‌ریز قادر به
قرار دادن آن نباشد، برنامه‌ریزی نمی‌شود.

برنامه‌ریز برای یک Pod گره‌های قابلیت را پیدا می‌کند و سپس یک مجموعه از عملکردها برای امتیاز
دهی به گره‌های قابلیت اجرا می‌کند و یک گره با بالاترین امتیاز را انتخاب می‌کند بین گره‌های
قابلیت‌ها برای اجرای Pod. برنامه‌ریز سپس API server را در مورد این تصمیم مطلع می‌کند که در
فرآیندی به نام _binding_ است.

عواملی که برای تصمیمات برنامه‌ریزی باید در نظر گرفته شوند شامل نیازهای منابع فردی و جمعی،
محدودیت‌های سخت‌افزاری / نرم‌افزاری / سیاست، مشخصات دوستی و ضد دوستی، محلیت داده، تداخل
بین بارکاری‌ها و غیره می‌باشد.

### انتخاب گره در kube-scheduler {#kube-scheduler-implementation}

kube-scheduler در یک عملیات دو مرحله‌ای یک گره را برای Pod انتخاب می‌کند:

1. فیلترینگ
1. امتیازدهی

مرحله _فیلترینگ_ مجموعه‌ای از گره‌ها را پیدا می‌کند که قابلیت برنامه‌ریزی Pod را دارند.
به عنوان مثال، فیلتر PodFitsResources بررسی می‌کند که آیا یک گره کاندیدا دارای منابع
کافی برای برآورده کردن درخواست‌های منابع خاص Pod است یا خیر. پس از این مرحله، لیست گره‌ها
شامل هر گره مناسب است؛ اغلب، بیشتر از یکی خواهد بود. اگر لیست خالی باشد، آن Pod (تا آن
لحظه) برنامه‌ریزی نشده است.

در مرحله _امتیازدهی_، برنامه‌ریز به گره‌های باقی‌مانده امتیاز می‌دهد تا انتخاب مناسب‌ترین
مکان برای اجرای Pod را انتخاب کند. برنامه‌ریز به هر گره باقی‌مانده براساس قوانین فعال
امتیازدهی امتیاز می‌دهد.

سرانجام، kube-scheduler Pod را به گره با بالاترین رتبه اختصاص می‌دهد. اگر بیش از یک گره
با امتیازهای مساوی وجود داشته باشد، kube-scheduler یکی از این‌ها را به صورت تصادفی انتخاب
می‌کند.

دو روش پشتیبانی شده برای پیکربندی رفتارهای فیلترینگ و امتیازدهی برنامه‌ریز عبارتند از:

1. [سیاست‌های برنامه‌ریزی](/docs/reference/scheduling/policies) که به شما امکان پیکربندی _شرایط_ برای فیلترینگ و _اولویت‌ها_ برای امتیازدهی می‌دهد.
1. [پروفایل‌های برنامه‌ریزی](/docs/reference/scheduling/config/#profiles) که به شما امکان می‌دهد پلاگین‌هایی را که مراحل مختلف برنامه‌ریزی را پیاده‌سازی می‌کنند، شامل `QueueSort`، `Filter`، `Score`، `Bind`، `Reserve`، `Permit` و دیگرها را پیکربندی کنید. همچنین می‌توانید kube-scheduler را به اجرای پروفایل‌های مختلف تنظیم کنید.

## {{% heading "چه بعدی است" %}}

* در مورد [تنظیم بهینه‌سازی عملکرد برنامه‌ریز](/docs/concepts/scheduling-eviction/scheduler-perf-tuning/) بیشتر بخوانید
* در مورد [محدودیت‌های پخش توپولوژی Pod](/docs/concepts/scheduling-eviction/topology-spread-constraints/) بیشتر بخوانید
* در مورد [مستندات مرجع](/docs/reference/command-line-tools-reference/kube-scheduler/) برای kube-scheduler بیشتر بخوانید
* در مورد [پیکربندی kube-scheduler (v1)](/docs/reference/config-api/kube-scheduler-config.v1/) بیشتر بخوانید
* در مورد [پیکربندی چندین برنامه‌ریز](/docs/tasks/extend-kubernetes/configure-multiple-schedulers/) بیشتر بخوانید
* در مورد [سیاست‌های مدیریت توپولوژی](/docs/tasks/administer-cluster/topology-manager/) بیشتر بخوانید
* در مورد [هزینه‌های Overhead Pod](/docs/concepts/scheduling-eviction/pod-overhead/) بیشتر بخوانید
* در مورد برنامه‌ریزی Podهایی که از حجم‌ها استفاده می‌کنند در:
  * [پشتیبانی از Topology Volume](/docs/concepts/storage/storage-classes/#volume-binding-mode)
  * [پیگیری ظرفیت ذخیره سازی](/docs/concepts/storage/storage-capacity/)
  * [محدودیت‌های حجم خاص گره](/docs/concepts/storage/storage-limits/)