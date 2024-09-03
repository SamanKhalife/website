---
reviewers:
- mikedanese
content_type: concept
title: Tools for Monitoring Resources
weight: 15
---

مرور اجمالی:

برای مقیاس‌پذیر کردن یک برنامه و ارائه خدمات قابل اعتماد، نیاز به درک رفتار برنامه هنگام استقرار دارید. شما می‌توانید با بررسی عملکرد برنامه در یک خوشه Kubernetes، اطلاعات کاملی از مصرف منابع برنامه را در سطوح مختلف مانند کانتینرها، [پادها](/docs/concepts/workloads/pods/)، [سرویس‌ها](/docs/concepts/services-networking/service/) و ویژگی‌های کل خوشه بدست آورید. این اطلاعات به شما امکان می‌دهد تا عملکرد برنامه خود را ارزیابی کرده و نقاط مشکل‌زایی را که می‌تواند بهبود عملکرد کلی را ایجاد کند، شناسایی کنید.

بدن:

در Kubernetes، مانیتورینگ برنامه به یک راه‌حل نظارتی منحصر به فرد بستگی ندارد. در خوشه‌های جدید، می‌توانید از لوله‌کشی‌های [متریک‌های منابع](#resource-metrics-pipeline) یا [متریک‌های کامل](#full-metrics-pipeline) برای جمع‌آوری آمارهای نظارتی استفاده کنید.

### لوله‌کشی متریک‌های منابع

لوله‌کشی متریک‌های منابع مجموعه‌ای محدود از متریک‌ها مرتبط با اجزای خوشه از قبیل کنترل‌کننده [Horizontal Pod Autoscaler](/docs/tasks/run-application/horizontal-pod-autoscale/) و ابزار `kubectl top` را فراهم می‌کند. این متریک‌ها توسط [metrics-server](https://github.com/kubernetes-sigs/metrics-server)، یک سرویس سبک و موقت در حافظه، جمع‌آوری می‌شوند و از طریق API `metrics.k8s.io` ارائه می‌شوند.

metrics-server همهٔ نودهای خوشه را کشف کرده و از هر نود برای استفاده از CPU و حافظه، پرس‌وجو می‌کند. kubelet به عنوان پلی بین مستر Kubernetes و نودها عمل می‌کند و پادها و کانتینرهای در حال اجرا را در یک ماشین مدیریت می‌کند. kubelet هر پاد را به کانتینرهای تشکیل‌دهنده‌اش تبدیل می‌کند و آمار استفاده از هر کانتینر را از رابط اجرای کانتینر به دست می‌آورد. اگر از یک رابط اجرای کانتینر استفاده کنید که از cgroups و فضاهای نام‌ها برای اجرای کانتینرها استفاده می‌کند و رابط اجرای کانتینر آمارهای استفاده را منتشر نمی‌کند، آنگاه kubelet می‌تواند به طور مستقیم آمارهای آن را جستجو کند (استفاده از کد از [cAdvisor](https://github.com/google/cadvisor)). پس از ورود آمارهای این گونه، kubelet سپس آمارهای تجمیع شده مصرف منابع پاد را از طریق API متریک‌های منابع metrics-server ارائه می‌دهد. این API در `/metrics/resource/v1beta1` در پورت‌های احراز هویت و فقط خواندنی kubelet سرویس می‌شود.

### لوله‌کشی متریک‌های کامل

لوله‌کشی متریک‌های کامل به شما دسترسی به متریک‌های پرسش‌شده می‌دهد. Kubernetes می‌تواند با واکنش به این متریک‌ها به طور خودکار اندازه پادها را یا تطبیق دهد براساس وضعیت فعلی خود، با استفاده از مکانیسم‌هایی مانند Horizontal Pod Autoscaler. لوله‌کشی نظارتی متریک‌ها را از kubelet جمع‌آوری کرده و سپس آن‌ها را به Kubernetes از طریق یک آداپتور ارائه می‌دهد که API `custom.metrics.k8s.io` یا `external.metrics.k8s.io` را پیاده‌سازی می‌کند.

Kubernetes برای کار با [OpenMetrics](https://openmetrics.io/) طراحی شده است که یکی از
[پروژه‌های نظارت و تجزیه و تحلیل CNCF](https://landscape.cncf.io/?group=projects-and-products&view-mode=card#observability-and-analysis--monitoring)
است که بر روی فرمت افشای Prometheus
[Prometheus exposition format](https://prometheus.io/docs/instrumenting/exposition_formats/)
به صورتی تقریباً 100٪ متناسب بازگشتی ساخته شده است.

اگر به [منظری CNCF](https://landscape.cncf.io/?group=projects-and-products&view-mode=card#observability-and-analysis--monitoring) نگاهی انداخته باشید، می‌توانید یک تعدادی از پروژه‌های نظارتی را که با Kubernetes کار می‌کنند را ببینید، با اسکراب کردن اطلاعات متریک و استفاده از آن برای کمک به مشاهده خوشه خود. انتخاب ابزار مناسب بر عهده شماست. منظره CNCF برای مشاهده و تجزیه و تحلیل شامل یک مخلوطی از نرم‌افزارهای متن باز، نرم‌افزار سرویس به عنوان خدمات و سایر محصولات تجاری است.

وقتی که یک لوله‌کشی متریک‌های کامل را طراحی و اجرا می‌کنید، می‌توانید این داده‌های نظارتی را به Kubernetes برگردانید. به عنوان مثال، یک HorizontalPodAutoscaler می‌توان

د از متریک‌های پردازش شده برای محاسبه اینکه چند پاد برای یک جزء از بار کاری خود اجرا شود، استفاده کند.

ادغام یک لوله‌کشی متریک‌های کامل در اجرای Kubernetes شما به علت دامنه بسیار گسترده احتمالی راه‌حل‌ها، خارج از حوزه مستندات Kubernetes است.

انتخاب پلتفرم نظارت بسیار بستگی به نیازها، بودجه و منابع فنی شما دارد. Kubernetes هیچ لوله‌کشی خاص متریک را توصیه نمی‌کند؛ [گزینه‌های زیادی](https://landscape.cncf.io/?group=projects-and-products&view-mode=card#observability-and-analysis--monitoring) موجود است. سیستم نظارتی شما باید قادر باشد به استاندارد متریک‌های [OpenMetrics](https://openmetrics.io/) پاسخ دهد و باید بر اساس طراحی و پیاده‌سازی کلی شما انتخاب شود.

## {{% heading "whatsnext" %}}

درباره ابزارهای اضافی اشکال‌زدایی، شامل:

* [لاگ‌نویسی](/docs/concepts/cluster-administration/logging/)
* [نظارت](/docs/tasks/debug/debug-cluster/resource-usage-monitoring/)
* [ورود به کانتینرها از طریق `exec`](/docs/tasks/debug/debug-application/get-shell-running-container/)
* [اتصال به کانتینرها از طریق پروکسی‌ها](/docs/tasks/extend-kubernetes/http-proxy-access-api/)
* [اتصال به کانتینرها از طریق انتقال پورت](/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)
* [بررسی نود Kubernetes با crictl](/docs/tasks/debug/debug-cluster/crictl/)
