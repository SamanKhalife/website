---
reviewers:
- dchen1107
- liggitt
title: ارتباط بین نودها و صفحه کنترل
content_type: concept
weight: 20
aliases:
- master-node-communication
---

<!-- overview -->

این سند مسیرهای ارتباطی بین {{< glossary_tooltip term_id="kube-apiserver" text="سرور API" >}} و {{< glossary_tooltip text="کلاستر" term_id="cluster" length="all" >}} Kubernetes را فهرست می‌کند. هدف این است که به کاربران اجازه دهد نصب خود را سفارشی کنند تا پیکربندی شبکه را بهبود بخشند به‌گونه‌ای که کلاستر بتواند روی یک شبکه غیرقابل اعتماد (یا روی IPهای عمومی کامل در یک ارائه‌دهنده ابری) اجرا شود.

<!-- body -->

## نود به صفحه کنترل

Kubernetes دارای یک الگوی API "مرکز و شعاع" است. تمام استفاده‌های API از نودها (یا پادهایی که اجرا می‌کنند) در سرور API خاتمه می‌یابد. هیچ‌یک از سایر اجزای صفحه کنترل به‌گونه‌ای طراحی نشده‌اند که سرویس‌های از راه دور را ارائه دهند. سرور API به‌گونه‌ای تنظیم شده است که به اتصالات از راه دور در یک پورت HTTPS امن (معمولاً ۴۴۳) با یک یا چند شکل از [احراز هویت](/docs/reference/access-authn-authz/authentication/) کلاینت گوش دهد. یک یا چند شکل از [مجوزدهی](/docs/reference/access-authn-authz/authorization/) باید فعال باشد، به ویژه اگر [درخواست‌های ناشناس](/docs/reference/access-authn-authz/authentication/#anonymous-requests) یا [توکن‌های حساب سرویس](/docs/reference/access-authn-authz/authentication/#service-account-tokens) مجاز باشند.

نودها باید با {{< glossary_tooltip text="گواهی" term_id="certificate" >}} ریشه عمومی برای کلاستر مجهز شوند تا بتوانند به‌طور ایمن به سرور API متصل شوند به همراه اعتبارنامه‌های معتبر کلاینت. یک رویکرد خوب این است که اعتبارنامه‌های کلاینت ارائه شده به kubelet به صورت یک گواهی کلاینت باشند. برای تهیه خودکار گواهی‌های کلاینت kubelet، به [بوت‌استرپینگ TLS kubelet](/docs/reference/access-authn-authz/kubelet-tls-bootstrapping/) مراجعه کنید.

{{< glossary_tooltip text="پادها" term_id="pod" >}} که می‌خواهند به سرور API متصل شوند، می‌توانند با استفاده از حساب سرویس به‌طور ایمن این کار را انجام دهند به‌طوری که Kubernetes به‌طور خودکار گواهی ریشه عمومی و یک توکن حامل معتبر را هنگام ایجاد پاد در آن تزریق کند.
سرویس `kubernetes` (در فضای نام `default`) با یک آدرس IP مجازی تنظیم شده است که به (از طریق `{{< glossary_tooltip text="kube-proxy" term_id="kube-proxy" >}}`) به نقطه پایانی HTTPS روی سرور API هدایت می‌شود.

اجزای صفحه کنترل نیز از طریق پورت امن با سرور API ارتباط برقرار می‌کنند.

در نتیجه، حالت عملیاتی پیش‌فرض برای اتصالات از نودها و پادهای در حال اجرا روی نودها به صفحه کنترل به‌طور پیش‌فرض امن است و می‌تواند روی شبکه‌های غیرقابل اعتماد و/یا عمومی اجرا شود.
## صفحه کنترل به نود

دو مسیر ارتباطی اصلی از صفحه کنترل (سرور API) به نودها وجود دارد.
اولین مسیر از سرور API به فرآیند {{< glossary_tooltip text="kubelet" term_id="kubelet" >}} است که بر روی هر نود در کلاستر اجرا می‌شود.
دومین مسیر از سرور API به هر نود، پاد یا سرویس از طریق قابلیت _پراکسی_ سرور API است.

### سرور API به kubelet

اتصالات از سرور API به kubelet برای موارد زیر استفاده می‌شوند:

* واکشی لاگ‌ها برای پادها.
* اتصال (معمولاً از طریق `kubectl`) به پادهای در حال اجرا.
* فراهم کردن قابلیت پورت فورواردینگ kubelet.

این اتصالات در نقطه پایانی HTTPS kubelet خاتمه می‌یابند. به طور پیش‌فرض، سرور API گواهی سرویس‌دهنده kubelet را تأیید نمی‌کند که این باعث می‌شود ارتباط در معرض حملات مرد میانی قرار گیرد و اجرای آن بر روی شبکه‌های غیرقابل اعتماد و/یا عمومی **ناامن** باشد.

برای تأیید این ارتباط، از فلگ `--kubelet-certificate-authority` استفاده کنید تا یک باندل گواهی ریشه برای تأیید گواهی سرویس‌دهنده kubelet به سرور API ارائه دهید.

اگر این امکان وجود ندارد، در صورت نیاز از [تونل‌سازی SSH](#ssh-tunnels) بین سرور API و kubelet استفاده کنید تا از اتصال بر روی یک شبکه غیرقابل اعتماد یا عمومی جلوگیری کنید.

در نهایت، [احراز هویت و/یا مجوزدهی kubelet](/docs/reference/access-authn-authz/kubelet-authn-authz/) باید فعال باشد تا API kubelet ایمن شود.

### سرور API به نودها، پادها و سرویس‌ها

اتصالات از سرور API به نود، پاد یا سرویس به طور پیش‌فرض اتصالات HTTP ساده هستند و بنابراین نه احراز هویت شده‌اند و نه رمزگذاری شده‌اند. این اتصالات می‌توانند با افزودن `https:` به نام نود، پاد یا سرویس در URL API بر روی یک اتصال HTTPS ایمن اجرا شوند، اما گواهی ارائه شده توسط نقطه پایانی HTTPS را تأیید نمی‌کنند و اعتبارنامه‌های کلاینت را ارائه نمی‌دهند. بنابراین، در حالی که اتصال رمزگذاری شده خواهد بود، هیچ تضمینی برای تمامیت آن ارائه نمی‌دهد. این اتصالات **در حال حاضر امن نیستند** تا بر روی شبکه‌های غیرقابل اعتماد یا عمومی اجرا شوند.

### تونل‌های SSH

Kubernetes از [تونل‌های SSH](https://www.ssh.com/academy/ssh/tunneling) برای حفاظت از مسیرهای ارتباطی بین صفحه کنترل و نودها پشتیبانی می‌کند. در این پیکربندی، سرور API یک تونل SSH به هر نود در کلاستر ایجاد می‌کند (با اتصال به سرور SSH که در پورت ۲۲ گوش می‌دهد) و تمام ترافیکی که برای kubelet، نود، پاد یا سرویس تعیین شده است را از طریق تونل ارسال می‌کند.
این تونل اطمینان می‌دهد که ترافیک خارج از شبکه‌ای که نودها در آن اجرا می‌شوند، در معرض قرار نمی‌گیرد.

{{< note >}}
تونل‌های SSH در حال حاضر از رده خارج شده‌اند، بنابراین نباید از آنها استفاده کنید مگر اینکه دقیقاً بدانید که چه می‌کنید. [سرویس Konnectivity](#konnectivity-service) جایگزین این کانال ارتباطی است.
{{< /note >}}

### سرویس Konnectivity

{{< feature-state for_k8s_version="v1.18" state="beta" >}}

به عنوان جایگزینی برای تونل‌های SSH، سرویس Konnectivity پروکسی سطح TCP را برای ارتباط بین صفحه کنترل و کلاستر فراهم می‌کند. سرویس Konnectivity از دو بخش تشکیل شده است: سرور Konnectivity در شبکه صفحه کنترل و عوامل Konnectivity در شبکه نودها.
عوامل Konnectivity اتصالات را به سرور Konnectivity آغاز کرده و اتصالات شبکه را حفظ می‌کنند.
پس از فعال کردن سرویس Konnectivity، تمام ترافیک از صفحه کنترل به نودها از طریق این اتصالات انجام می‌شود.

[وظیفه سرویس Konnectivity](/docs/tasks/extend-kubernetes/setup-konnectivity/) را دنبال کنید تا سرویس Konnectivity را در کلاستر خود راه‌اندازی کنید.

## {{% heading "whatsnext" %}}

* درباره [اجزای صفحه کنترل Kubernetes](/docs/concepts/overview/components/#control-plane-components) بخوانید.
* بیشتر درباره [مدل هاب و شعاع](https://book.kubebuilder.io/multiversion-tutorial/conversion-concepts.html#hubs-spokes-and-other-wheel-metaphors) بیاموزید.
* یاد بگیرید چگونه [یک کلاستر را ایمن کنید](/docs/tasks/administer-cluster/securing-a-cluster/).
* بیشتر درباره [API Kubernetes](/docs/concepts/overview/kubernetes-api/) بیاموزید.
* [سرویس Konnectivity را راه‌اندازی کنید](/docs/tasks/extend-kubernetes/setup-konnectivity/).
* [از پورت فورواردینگ برای دسترسی به برنامه‌ها در کلاستر استفاده کنید](/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).
* یاد بگیرید چگونه [لاگ‌ها را برای پادها واکشی کنید](/docs/tasks/debug/debug-application/debug-running-pod/#examine-pod-logs)، [از kubectl port-forward استفاده کنید](/docs/tasks/access-application-cluster/port-forward-access-application-cluster/#forward-a-local-port-to-a-port-on-the-pod).