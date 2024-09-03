---
title: "رفع مشکلات kubectl"
content_type: task
weight: 10
---

<!-- overview -->

این مستند درباره بررسی و تشخیص مشکلات مربوط به {{<glossary_tooltip text="kubectl" term_id="kubectl">}} است. اگر با مشکلات دسترسی به `kubectl` یا اتصال به خوشه‌ی خود مواجه شده‌اید، این مستند به تشریح سناریوها و راه‌حل‌های مختلف برای شناسایی و حل مشکل احتمالی کمک می‌کند.

<!-- body -->

## {{% heading "پیش‌نیازها" %}}

* نیاز به داشتن یک خوشه Kubernetes دارید.
* همچنین باید `kubectl` را نصب کرده باشید - جهت اطلاعات بیشتر [ابزارهای نصب](/docs/tasks/tools/#kubectl) را ببینید.

## بررسی تنظیمات kubectl

مطمئن شوید که `kubectl` را به درستی در ماشین محلی خود نصب و پیکربندی کرده‌اید. ورژن `kubectl` را برای اطمینان از به‌روز بودن و سازگاری آن با خوشه خود بررسی کنید.

بررسی ورژن kubectl:

```shell
kubectl version
```

نمونه‌ای از خروجی:

```console
Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.4",GitCommit:"fa3d7990104d7c1f16943a67f11b154b71f6a132", GitTreeState:"clean",BuildDate:"2023-07-19T12:20:54Z", GoVersion:"go1.20.6", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v5.0.1
Server Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.3",GitCommit:"25b4e43193bcda6c7328a6d147b1fb73a33f1598", GitTreeState:"clean",BuildDate:"2023-06-14T09:47:40Z", GoVersion:"go1.20.5", Compiler:"gc", Platform:"linux/amd64"}

```

اگر پیام `Unable to connect to the server: dial tcp <server-ip>:8443: i/o timeout` را ببینید، بجای `Server Version`، نیاز به رفع مشکلات اتصال kubectl با خوشه خود دارید.

مطمئن شوید که `kubectl` را به دنبال [مستندات رسمی نصب kubectl](/docs/tasks/tools/#kubectl) نصب کرده‌اید و متغیر محیطی `$PATH` را به درستی پیکربندی کرده‌اید.

## بررسی kubeconfig

برای اتصال به یک خوشه Kubernetes، `kubectl` نیاز به یک فایل `kubeconfig` دارد. این فایل معمولاً در دایرکتوری `~/.kube/config` قرار دارد. مطمئن شوید که یک فایل `kubeconfig` معتبر دارید. اگر فایل `kubeconfig` ندارید، می‌توانید آن را از مدیر Kubernetes خود دریافت کنید یا از دایرکتوری `/etc/kubernetes/admin.conf` کنترل پلن Kubernetes خود کپی کنید. اگر خوشه Kubernetes خود را روی یک پلتفرم ابری اجرا کرده‌اید و فایل `kubeconfig` خود را از دست داده‌اید، می‌توانید آن را با استفاده از ابزارهای ارائه‌دهنده ابر خود مجدداً تولید کنید. برای بازسازی فایل `kubeconfig`، به مستندات ارائه‌دهنده ابر مراجعه کنید.

بررسی کنید که متغیر محیطی `$KUBECONFIG` به درستی پیکربندی شده باشد. می‌توانید متغیر محیطی `$KUBECONFIG` را تنظیم کنید یا با استفاده از پارامتر `--kubeconfig` با kubectl، مسیر دایرکتوری فایل `kubeconfig` را مشخص کنید.

## بررسی اتصال VPN

اگر برای دسترسی به خوشه Kubernetes خود از شبکه خصوصی مجازی (VPN) استفاده می‌کنید، مطمئن شوید که اتصال VPN شما فعال و پایدار است. گاهی اوقات، قطع شدن اتصال VPN می‌تواند به مشکلات اتصال با خوشه منجر شود. مجدداً به VPN متصل شده و دوباره تلاش کنید تا به خوشه دسترسی پیدا کنید.

## احراز هویت و اجازه‌دهی

اگر از احراز هویت مبتنی بر توکن استفاده می‌کنید و `kubectl` خطاهایی مربوط به توکن احراز هویت یا آدرس سرور احراز هویت باز می‌گرداند، اطمینان حاصل کنید که توکن احراز هویت Kubernetes و آدرس سرور احراز هویت به درستی پیکربندی شده‌اند.

اگر `kubectl` خطاهایی مربوط به اجازه‌دهی باز می‌گرداند، مطمئن شوید که از مدارک ورودی کاربر معتبر استفاده می‌کنید و دسترسی لازم برای دسترسی به منبع مورد نیاز را دارید.

## بررسی کانتکست‌ها

Kubernetes از [چندین خوشه و کانتکست](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/) پشتیبانی می‌کند. اطمینان حاصل کنید که از کانتکست مناسب برای تعامل با خوشه خود استفاده می‌کنید.

لیست کانتکست‌های موجود:

```shell
kubectl config get-contexts
```

انتقال به کانتکست مناسب:

```shell
kubectl config use-context <نام-کانتکست>
```

## سرور API و توزیع‌کننده بار

سرور {{<glossary_tooltip text="kube-apiserver" term_id="kube-apiserver">}} مؤلفه مرکزی یک خوشه Kubernetes است. اگر سرور API یا توزیع‌کننده باری که در پیش روی سرورهای API شما اجرا می‌ش

ود قابل دسترسی یا پاسخگو نباشد، شما قادر به تعامل با خوشه نخواهید بود.

با استفاده از دستور `ping`، از قابل دسترسی بودن میزبان سرور API اطمینان حاصل کنید. اتصال شبکه و فایروال خوشه خود را بررسی کنید. اگر از یک ارائه‌دهنده ابر برای اجرای خوشه استفاده می‌کنید، وضعیت بررسی سلامتی ارائه‌دهنده ابر خود را برای سرور API خوشه چک کنید.

بررسی کنید که وضعیت توزیع‌کننده بار (اگر استفاده می‌کنید) برای اطمینان از سلامت و ارسال ترافیک به سرور API مناسب است.

## مشکلات TLS

سرور API Kubernetes فقط درخواست‌های HTTPS را به صورت پیش‌فرض خدمت می‌دهد. در این حالت ممکن است مشکلات TLS به دلیل دلایل مختلفی مانند انقضای گواهینامه یا اعتبار زنجیره رخ دهد.

می‌توانید گواهینامه TLS را در فایل kubeconfig که در دایرکتوری `~/.kube/config` قرار دارد، پیدا کنید. ویژگی `certificate-authority` شامل گواهینامه CA و ویژگی `client-certificate` شامل گواهینامه کاربر است.

اعتبار این گواهینامه‌ها را بررسی کنید:

```shell
openssl x509 -noout -dates -in $(kubectl config view --minify --output 'jsonpath={.clusters[0].cluster.certificate-authority}')
```

خروجی:

```console
notBefore=Sep  2 08:34:12 2023 GMT
notAfter=Aug 31 08:34:12 2033 GMT
```

```shell
openssl x509 -noout -dates -in $(kubectl config view --minify --output 'jsonpath={.users[0].user.client-certificate}')
```

خروجی:

```console
notBefore=Sep  2 08:34:12 2023 GMT
notAfter=Sep  2 08:34:12 2026 GMT
```

## بررسی ابزارهای کمکی kubectl

برخی از ابزارهای کمکی احراز هویت kubectl دسترسی آسان به خوشه‌های Kubernetes را فراهم می‌کنند. اگر از چنین ابزارهایی استفاده کرده‌اید و با مشکلات اتصال روبرو هستید، اطمینان حاصل کنید که پیکربندی‌های لازم هنوز هم در دسترس هستند.

بررسی پیکربندی kubectl برای جزئیات احراز هویت:

```shell
kubectl config view
```

اگر قبلاً از ابزاری مانند `kubectl-oidc-login` استفاده کرده‌اید، اطمینان حاصل کنید که هنوز هم نصب و پیکربندی شده است.
