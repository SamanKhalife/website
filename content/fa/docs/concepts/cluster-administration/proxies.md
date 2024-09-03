---
title: Proxies در Kubernetes
content_type: concept
weight: 100
---

<!-- overview -->
این صفحه توضیح می‌دهد که چگونه Proxies در Kubernetes استفاده می‌شوند.


<!-- body -->

## Proxies

در استفاده از Kubernetes، شما ممکن است با چندین نوع Proxy مختلف مواجه شوید:

1. [kubectl proxy](/docs/tasks/access-application-cluster/access-cluster/#directly-accessing-the-rest-api):

   - در دسکتاپ کاربر یا در یک Pod اجرا می‌شود
   - از آدرس localhost به apiserver Kubernetes پروکسی می‌کند
   - از HTTP برای ارتباط کلاینت به پروکسی استفاده می‌کند
   - از HTTPS برای ارتباط پروکسی به apiserver استفاده می‌کند
   - مکان apiserver را مشخص می‌کند
   - هدرهای احراز هویت را اضافه می‌کند

1. [apiserver proxy](/docs/tasks/access-application-cluster/access-cluster-services/#discovering-builtin-services):

   - یک بستنی درونی است که به apiserver اضافه شده است
   - کاربر را خارج از خوشه به آدرس‌های IP خوشه که به طور دیگر قابل دسترسی نیستند متصل می‌کند
   - در پردازش‌های apiserver اجرا می‌شود
   - از HTTPS برای ارتباط کلاینت به پروکسی استفاده می‌کند (یا HTTP اگر apiserver به این شکل پیکربندی شده باشد)
   - پروکسی به هدف ممکن است از HTTP یا HTTPS استفاده کند که توسط پروکسی با استفاده از اطلاعات موجود انتخاب شده است
   - می‌توان از آن برای دسترسی به یک Node، Pod یا Service استفاده کرد
   - در صورت استفاده برای دسترسی به یک Service، بارگذاری توازن انجام می‌دهد

1. [kube proxy](/docs/concepts/services-networking/service/#ips-and-vips):

   - بر روی هر Node اجرا می‌شود
   - UDP، TCP و SCTP را پروکسی می‌کند
   - HTTP را نمی‌فهمد
   - بارگذاری توازن فراهم می‌کند
   - فقط برای دسترسی به Services استفاده می‌شود

1. یک Proxy/Load-balancer در جلوی apiserver(s):

   - وجود و اجرا بسته به خوشه به خوشه متفاوت است (مانند nginx)
   - بین تمام مشتریان و یک یا چند apiserver قرار دارد
   - در صورت وجود چند apiserver، عملیات توازن بار را انجام می‌دهد.

1. Load Balancers ابری بر روی سرویس‌های خارجی:

   - توسط برخی ارائه‌دهندگان ابر ارائه شده است (مانند AWS ELB، Google Cloud Load Balancer)
   - به طور خودکار ایجاد می‌شود زمانی که سرویس Kubernetes نوع `LoadBalancer` دارد
   - معمولاً فقط پروتکل‌های UDP/TCP را پشتیبانی می‌کند
   - پشتیبانی SCTP به عهده پیاده‌سازی Load Balancer ارائه‌دهنده ابر است
   - پیاده‌سازی بسته به ارائه‌دهنده ابر متفاوت است.

کاربران Kubernetes عمدتاً نیازی به نگرانی درباره چیزهای غیر از دو نوع اولیه ندارند. معمولاً مدیر خوشه اطمینان می‌یابد که انواع دیگر به درستی تنظیم شده باشند.

## درخواست‌های Redirect

Proxies قابلیت‌های Redirect را جایگزین کرده‌اند. Redirectها منسوخ شده‌اند.

