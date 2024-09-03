---
reviewers:
- derekwaynecarr
- mikedanese
- thockin
title: فضاهای نام
api_metadata:
- apiVersion: "v1"
  kind: "Namespace"
content_type: مفهوم
weight: 45
---

<!-- overview -->

در Kubernetes، _فضاهای نام_ یک مکانیزم برای جداسازی گروه‌هایی از منابع در یک خوشه تنها فراهم می‌کنند. نام‌های منابع باید در یک فضاهای نام منحصر به فرد باشند، اما در فضاهای نام مختلف باید یکتا نباشند. محدودیت مبتنی بر فضاهای نام فقط برای اشیاء مربوط به فضاهای نام (_مانند Deployments، Services و غیره_) قابل استفاده است و برای اشیاء سراسر خوشه (_مانند StorageClass، Nodes، PersistentVolumes و غیره_) منطبق نیست.

<!-- body -->

## زمان استفاده از چندین فضای نام

فضاهای نام برای استفاده در محیط‌هایی با تعداد زیادی کاربر منتشر شده در چندین تیم یا پروژه مناسب است. برای خوشه‌هایی با تعداد کاربران از چند تا ده نفر، شما به طور کلی نیازی به ایجاد یا مدیریت فضاهای نام ندارید. استفاده از فضاهای نام را آغاز کنید هنگامی که ویژگی‌هایی که فراهم می‌کنند را نیاز دارید.

فضاهای نام یک دامنه برای نام‌ها فراهم می‌کنند. نام‌های منابع باید در یک فضاهای نام منحصر به فرد باشند، اما در فضاهای نام مختلف باید یکتا نباشند. فضاهای نام نمی‌توانند داخل یکدیگر تو در تو باشند و هر منبع Kubernetes تنها می‌تواند در یک فضاهای نام باشد.

فضاهای نام یک روش برای تقسیم منابع خوشه بین چندین کاربر (از طریق [کوتاه‌نام منابع](/docs/concepts/policy/resource-quotas/)) هستند.

{{< note >}}
برای یک خوشه تولیدی، در نظر بگیرید _نه_ از فضاهای نام `default` استفاده کنید. به جای آن، فضاهای نام دیگر ایجاد کنید و از آن‌ها استفاده کنید.
{{< /note >}}

## فضاهای نام اولیه

Kubernetes با چهار فضاهای نام اولیه شروع می‌کند:

`default`
: Kubernetes این فضاهای نام را شامل می‌شود تا بتوانید از خوشه جدید خود استفاده کنید بدون اینکه ابتدا یک فضاهای نام ایجاد کنید.

`kube-node-lease`
: این فضاهای نام شامل اشیاء [Lease](/docs/concepts/architecture/leases/) مرتبط با هر گره است. Lease های گره به kubelet اجازه می‌دهد تا [heartbeats](/docs/concepts/architecture/nodes/#node-heartbeats) ارسال کند تا کنترل‌پلن بتواند شکست گره را تشخیص دهد.

`kube-public`
: این فضاهای نام توسط *همه* مشتریان قابل خواندن است (شامل آن‌هایی که تأیید هویت ندارند). این فضاهای نام اصولاً برای استفاده در خوشه‌ها تعیین شده است، در صورتی که برخی از منابع باید در سراسر خوشه قابل مشاهده و قابل خواندن باشند. جنبه عمومی این فضاهای نام فقط یک اصلاح است، نه یک الزام.

`kube-system`
: فضاهای نام برای اشیاء ایجاد شده توسط سیستم Kubernetes.

## کار با فضاهای نام

ایجاد و حذف فضاهای نام در [مستندات راهنمای مدیریت برای فضاهای نام](/docs/tasks/administer-cluster/namespaces) شرح داده شده است.

{{< note >}}
    اجتناب از ایجاد فضاهای نام با پیشوند `kube-`، زیرا برای فضاهای نام سیستم Kubernetes رزرو شده است.
{{< /note >}}

### مشاهده فضاهای نام

می‌توانید فضاهای نام فعلی را در یک خوشه با استفاده از دستور زیر لیست کنید:

```shell
kubectl get namespace
```
```
NAME              STATUS   AGE
default           Active   1d
kube-node-lease   Active   1d
kube-public       Active   1d
kube-system       Active   1d
```


### تنظیم فضای نام برای یک درخواست

برای تنظیم فضای نام برای یک درخواست فعلی، از پرچم `--namespace` استفاده کنید.

به عنوان مثال:

```shell
kubectl run nginx --image=nginx --namespace=<insert-namespace-name-here>
kubectl get pods --namespace=<insert-namespace-name-here>
```

### تنظیم اولویت فضای نام

می‌توانید فضای نام را برای تمام دستورات بعدی kubectl به طور دائمی ذخیره کنید در همان زمینه.

```shell
kubectl config set-context --current --namespace=<insert-namespace-name-here>
# اعتبارسنجی آن
kubectl config view --minify | grep namespace:
```

## فضاهای نام و DNS

وقتی یک [Service](/docs/concepts/services-networking/service/) را ایجاد می‌کنید، یک [ورودی DNS متناظر](/docs/concepts/services-networking/dns-pod-service/) ایجاد می‌شود. این ورودی به شکل `<service-name>.<namespace-name>.svc.cluster.local` است، به این معنی که اگر یک کانتینر فقط از `<service-name>` استفاده کند، به سرویسی که محلی برای فضای نام است، راه‌حل خواهد داد. این ب

رای استفاده از تنظیمات یکسان در انواع مختلف مانند توسعه، مرحله‌بندی و تولید مفید است. اگر می‌خواهید از طریق فضاهای نام از نام‌های کامل دامنه دسترسی پیدا کنید.

بنابراین، همهٔ نام‌های فضاهای نام باید [برچسب‌های DNS استاندارد RFC 1123](/docs/concepts/overview/working-with-objects/names/#dns-label-names) باشند.

{{< warning >}}
با ایجاد فضاهای نام با همان نام با [دامنه‌های بالاتر عمومی](https://data.iana.org/TLD/tlds-alpha-by-domain.txt)، خدمات در این فضاهای نام می‌توانند نام‌های DNS کوتاهی داشته باشند که با رکوردهای DNS عمومی همپوشانی دارند. بارگذاری از هر فضاهای نام که به دنبال جستجوی DNS بدون نقطه پایانی هستند، به این خدمات هدایت خواهد شد و از DNS عمومی سبقت خواهد گرفت.

برای کاهش این خطر، مجوزهای ایجاد فضاهای نام را به کاربران اعتماد شده محدود کنید. در صورت لزوم، می‌توانید کنترل‌های امنیتی شخص ثالث، مانند [کنترل‌کننده‌های ورودی گسترده وب](/docs/reference/access-authn-authz/extensible-admission-controllers/) را پیکربندی کنید تا ایجاد هر فضاهای نام با نام [TLD عمومی](https://data.iana.org/TLD/tlds-alpha-by-domain.txt) را مسدود کند.
{{< /warning >}}

## نه همه اشیاء در یک فضاهای نام هستند

بیشتر منابع Kubernetes (مانند پادها، خدمات، کنترل‌کننده‌های تکثیر و دیگر منابع) در برخی از فضاهای نام‌ها هستند. با این حال، منابع فضاهای نام خودشان در یک فضاهای نام نیستند. و منابع سطح پایین، مانند [گره‌ها](/docs/concepts/architecture/nodes/) و [حجم‌های ماندگار](/docs/concepts/storage/persistent-volumes/)، در هیچ فضاهای نامی نیستند.

برای دیدن اینکه منابع Kubernetes کدام‌ها در یک فضاهای نام هستند و کدام‌ها نیستند:

```shell
# در یک فضاهای نام
kubectl api-resources --namespaced=true

# نیست در یک فضاهای نام
kubectl api-resources --namespaced=false
```

## برچسب‌گذاری خودکار

{{< feature-state for_k8s_version="1.22" state="stable" >}}

صفحه کنترل Kubernetes یک برچسب ثابت `kubernetes.io/metadata.name` روی همه فضاهای نام قرار می‌دهد. ارزش برچسب نام فضاهای نام است.

## {{% heading "whatsnext" %}}

* بیشتر درباره [ایجاد یک فضاهای نام جدید](/docs/tasks/administer-cluster/namespaces/#creating-a-new-namespace) بیاموزید.
* بیشتر درباره [حذف یک فضاهای نام](/docs/tasks/administer-cluster/namespaces/#deleting-a-namespace) بیاموزید.
