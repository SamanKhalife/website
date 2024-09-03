---
reviewers:
- hasheddan
- pjbgf
- saschagrunert
title: محدود کردن Syscall های یک کانتینر با seccomp
content_type: آموزش
weight: 40
min-kubernetes-server-version: v1.22
---

<!-- مرور -->

{{< feature-state for_k8s_version="v1.19" state="stable" >}}

Seccomp مخفف Secure Computing Mode است و از ویژگی‌های هسته‌ی لینوکس است که از نسخه ۲.۶.۱۲ به بعد در دسترس است. این ویژگی به شما اجازه‌ی ایجاد یک محیط شنونده برای دسترسی‌های یک فرآیند به هسته‌ی لینوکس را می‌دهد، با محدود کردن تماس‌هایی که فرآیند می‌تواند از فضای کاربری به هسته انجام دهد. Kubernetes به شما امکان می‌دهد تا پروفایل‌های seccomp را که بر روی یک گره نصب شده‌اند را به پادها و کانتینرهای شما اعمال کنید.

### اهداف

* آموزش نحوه‌ی بارگذاری پروفایل‌های seccomp بر روی یک گره
* آموزش نحوه‌ی اعمال یک پروفایل seccomp به یک کانتینر
* مشاهده آدیتینگ تماس‌های سیستمی توسط یک فرایند کانتینر
* مشاهده رفتار هنگامی که یک پروفایل مفقود مشخص می‌شود
* زیرنظر قرارگیری یک پروفایل seccomp
* آموزش نحوه‌ی ایجاد پروفایل‌های seccomp دقیق
* آموزش نحوه‌ی اعمال پروفایل seccomp پیش‌فرض رانتایم کانتینر

## پیش‌نیازها

برای انجام تمام مراحل این آموزش، شما باید ابزار [kind](/docs/tasks/tools/#kind) و [kubectl](/docs/tasks/tools/#kubectl) را نصب کرده باشید.

دستورات استفاده‌شده در آموزش فرض می‌کند که از [Docker](https://www.docker.com/) به عنوان رانتایم کانتینر خود استفاده می‌کنید. (کلاستر ایجاد شده توسط `kind` ممکن است از یک رانتایم کانتینر متفاوت داخلی استفاده کند). همچنین می‌توانید از [Podman](https://podman.io/) استفاده کنید، اما در این صورت باید دستورالعمل‌های خاصی را برای موفقیت آمیز بودن مراحل دنبال کنید.

این آموزش نمونه‌هایی را نشان می‌دهد که هنوز بتا هستند (از ورژن v1.25 به بعد) و دیگری که فقط از قابلیت‌های seccomp عمومی در دسترس استفاده می‌کنند. باید مطمئن شوید که کلاستر شما برای ورژنی که استفاده می‌کنید [به درستی پیکربندی شده است](https://kind.sigs.k8s.io/docs/user/quick-start/#setting-kubernetes-version).

همچنین این آموزش از ابزار `curl` برای دانلود نمونه‌ها به کامپیوتر شما استفاده می‌کند. می‌توانید گام‌ها را تطبیق دهید تا از ابزار دیگری استفاده کنید اگر تمایل دارید.

{{< note >}}
امکان اعمال یک پروفایل seccomp بر روی یک کانتینری که با `privileged: true` تنظیم شده است در context امن وجود ندارد. کانتینرهای privileged همیشه به عنوان `Unconfined` اجرا می‌شوند.
{{< /note >}}

<!-- مراحل -->

## دانلود نمونه‌های پروفایل seccomp {#download-profiles}

محتویات این پروفایل‌ها بعداً بررسی می‌شود، اما برای الان آن‌ها را دانلود کنید و در یک دایرکتوری به نام `profiles/` ذخیره کنید تا بتوانید آن‌ها را به کلاستر بارگذاری کنید.

{{< tabs name="tab_with_code" >}}
{{< tab name="audit.json" >}}
{{% code_sample file="pods/security/seccomp/profiles/audit.json" %}}
{{< /tab >}}
{{< tab name="violation.json" >}}
{{% code_sample file="pods/security/seccomp/profiles/violation.json" %}}
{{< /tab >}}
{{< tab name="fine-grained.json" >}}
{{% code_sample file="pods/security/seccomp/profiles/fine-grained.json" %}}
{{< /tab >}}
{{< /tabs >}}

اجرای دستورات زیر:

```shell
mkdir ./profiles
curl -L -o profiles/audit.json https://k8s.io/examples/pods/security/seccomp/profiles/audit.json
curl -L -o profiles/violation.json https://k8s.io/examples/pods/security/seccomp/profiles/violation.json
curl -L -o profiles/fine-grained.json https://k8s.io/examples/pods/security/seccomp/profiles/fine-grained.json
ls profiles
```

باید در انتهای مرحله‌ی آخر سه پروفایل لیست شده باشد:
```
audit.json  fine-grained.json  violation.json
```

## ایجاد یک کلاستر کوبرنیتیس محلی با استفاده از kind

برای سادگی، می‌توانید از [kind](https://kind.sigs.k8s.io/) برای ایجاد یک کلاستر تک نود با پروفایل‌های seccomp بارگذاری شده استفاده کنید. Kind کوبرنیتیس را در Docker اجرا می‌کند، بنابراین هر گره از کلاستر یک کانتینر است. این امکان را فراهم می‌کند که فایل‌ها را در سیستم فایل هر کانتینر مشابه بارگذاری کردن فایل‌ها بر روی یک گره داشته باشید.

{{%

 code_sample file="pods/security/seccomp/kind.yaml" %}}

فایل پیکربندی kind را دانلود کنید و آن را در یک فایل با نام `kind.yaml` ذخیره کنید:

```shell
curl -L -O https://k8s.io/examples/pods/security/seccomp/kind.yaml
```

می‌توانید نسخه‌ی خاصی از کوبرنیتیس را با تنظیم تصویر کانتینر گره تنظیم کنید. برای اطلاعات بیشتر، به [Nodes](https://kind.sigs.k8s.io/docs/user/configuration/#nodes) در مستندات kind بپردازید.

این آموزش فرض می‌کند که از ورژن Kubernetes {{< param "version" >}} استفاده می‌کنید.

به عنوان یک ویژگی بتا، می‌توانید کوبرنیتیس را پیکربندی کنید تا از پروفایلی که رانتایم کانتینر [ترجیح می‌دهد](#enable-the-use-of-runtimedefault-as-the-default-seccomp-profile-for-all-workloads) به عنوان پروفایل seccomp پیش‌فرض برای همه‌ی بارها استفاده کند، به جای اینکه به `Unconfined` بپردازد. اگر می‌خواهید این کار را امتحان کنید، قبل از ادامه مراحل، به آن‌ها مراجعه کنید.

پس از اینکه پیکربندی kind را قرار دادید، کلاستر kind را با آن پیکربندی ایجاد کنید:

```shell
kind create cluster --config=kind.yaml
```

پس از آماده‌سازی کلاستر جدید Kubernetes، کانتینر Docker را که به عنوان یک گره تک نود کلاستر اجرا می‌شود شناسایی کنید:

```shell
docker ps
```

باید خروجی مشخصات یک کانتینر که با نام `kind-control-plane` در حال اجرا است را ببینید. خروجی مشابه این خواهد بود:

```
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                       NAMES
6a96207fed4b        kindest/node:v1.18.2   "/usr/local/bin/entr…"   27 seconds ago      Up 24 seconds       127.0.0.1:42223->6443/tcp   kind-control-plane
```

اگر دسترسی به سیستم فایل این کانتینر را مشاهده کنید، باید ببینید که دایرکتوری `profiles/` با موفقیت به مسیر seccomp پیش‌فرض kubelet بارگذاری شده است. برای اجرای یک دستور در پاد از `docker exec` استفاده کنید:

```shell
# شناسه کانتینر 6a96207fed4b را با آن چیزی که از "docker ps" دیدید، تغییر دهید
docker exec -it 6a96207fed4b ls /var/lib/kubelet/seccomp/profiles
```

```
audit.json  fine-grained.json  violation.json
```

شما تأیید می‌کنید که این پروفایل‌های seccomp برای kubelet در حال اجرا در kind در دسترس هستند.

## ایجاد یک پاد که از پروفایل seccomp پیش‌فرض رانتایم کانتینر استفاده می‌کند

بیشتر رانتایم‌های کانتینر یک مجموعه منطقی از تماس‌های سیستمی را که مجاز یا نامجاز هستند، فراهم می‌کنند. می‌توانید این تنظیمات پیش‌فرض را برای بارهای کاری خود به ارث ببرید با تنظیم نوع seccomp در context امنیتی یک پاد یا کانتینر به `RuntimeDefault`.

{{< note >}}
اگر پیکربندی `seccompDefault` [را فعال کرده‌اید](/docs/reference/config-api/kubelet-config.v1beta1/)، آنگاه هنگامی که پروفایل seccomp دیگری مشخص نمی‌شود، پادها از پروفایل seccomp `RuntimeDefault` استفاده می‌کنند. در غیر این صورت، پیش‌فرض `Unconfined` است.
{{< /note >}}

در اینجا یک مانیفست برای یک پاد است که برای همه‌ی کانتینرهای خود از پروفایل seccomp `RuntimeDefault` استفاده می‌کند:

{{% code_sample file="pods/security/seccomp/ga/default-pod.yaml" %}}

ایجاد این پاد را انجام دهید:
```shell
kubectl apply -f https://k8s.io/examples/pods/security/seccomp/ga/default-pod.yaml
```

```shell
kubectl get pod default-pod
```

باید پاد به عنوان شروع موفقیت‌آمیز نمایش داده شود:
```
NAME        READY   STATUS    RESTARTS   AGE
default-pod 1/1     Running   0          20s
```

پیش از ادامه به بخش بعدی، پاد را حذف کنید:

```shell
kubectl delete pod default-pod --wait --now
```


```markdown
## ایجاد یک پاد با پروفایل seccomp برای حسابرسی تماس‌های سیستمی

برای شروع، پروفایل `audit.json` که تمام تماس‌های سیستمی فرآیند را ثبت می‌کند، را به یک پاد جدید اعمال کنید.

اینجا یک مانیفست برای این پاد است:

{{% code_sample file="pods/security/seccomp/ga/audit-pod.yaml" %}}

{{< note >}}
نسخه‌های قدیمی‌تر Kubernetes به شما امکان می‌دادند تا با استفاده از آنوتیشن‌ها، رفتار seccomp را پیکربندی کنید. Kubernetes {{< skew currentVersion >}} فقط از فیلدهای داخل `.spec.securityContext` برای پیکربندی seccomp پشتیبانی می‌کند و این آموزش این رویکرد را توضیح می‌دهد.
{{< /note >}}

پاد را در کلاستر ایجاد کنید:

```shell
kubectl apply -f https://k8s.io/examples/pods/security/seccomp/ga/audit-pod.yaml
```

این پروفایل هیچ محدودیتی روی تماس‌های سیستمی اعمال نمی‌کند، بنابراین پاد باید با موفقیت شروع شود.

```shell
kubectl get pod audit-pod
```

```
NAME        READY   STATUS    RESTARTS   AGE
audit-pod   1/1     Running   0          30s
```

برای تعامل با این نقطه پایانی ارائه شده توسط این کانتینر، یک سرویس NodePort ایجاد کنید که به شما امکان دسترسی به نقطه پایانی را از داخل کانتینر control plane kind فراهم می‌کند.

```shell
kubectl expose pod audit-pod --type NodePort --port 5678
```

بررسی کنید که سرویس به چه درگاهی روی گره اختصاص داده شده است.

```shell
kubectl get service audit-pod
```

خروجی مشابه زیر خواهد بود:

```
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
audit-pod   NodePort   10.111.36.142   <none>        5678:32373/TCP   72s
```

اکنون می‌توانید از `curl` استفاده کنید تا به این نقطه پایانی از داخل کانتینر control plane kind دسترسی پیدا کنید، در پورتی که این سرویس آن را ارائه می‌دهد.

```shell
# شناسه کانتینر control plane 6a96207fed4b و شماره پورت 32373 را با آنچه که از "docker ps" دیدید، تغییر دهید
docker exec -it 6a96207fed4b curl localhost:32373
```

```
just made some syscalls!
```

می‌بینید که فرآیند در حال اجرا است، اما در واقع چه تماس‌های سیستمی انجام داد؟ چون این پاد در یک کلاستر محلی اجرا می‌شود، شما باید این تماس‌ها را در `/var/log/syslog` در سیستم محلی خود ببینید. یک پنجره ترمینال جدید باز کنید و خروجی تماس‌های `http-echo` را با `tail` برای دنبال‌کردن از نظر اطلاعاتی ببینید:

```shell
# مسیر لاگ بر روی کامپیوتر شما ممکن است با "/var/log/syslog" متفاوت باشد
tail -f /var/log/syslog | grep 'http-echo'
```

به عنوان مثال:
```
Jul  6 15:37:40 my-machine kernel: [369128.669452] audit: type=1326 audit(1594067860.484:14536): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=51 compat=0 ip=0x46fe1f code=0x7ffc0000
Jul  6 15:37:40 my-machine kernel: [369128.669453] audit: type=1326 audit(1594067860.484:14537): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=54 compat=0 ip=0x46fdba code=0x7ffc0000
Jul  6 15:37:40 my-machine kernel: [369128.669455] audit: type=1326 audit(1594067860.484:14538): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=202 compat=0 ip=0x455e53 code=0x7ffc0000
Jul  6 15:37:40 my-machine kernel: [369128.669456] audit: type=1326 audit(1594067860.484:14539): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=288 compat=0 ip=0x46fdba code=0x7ffc0000
Jul  6 15:37:40 my-machine kernel: [369128.669517] audit: type=1326 audit(1594067860.484:14540): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=0 compat=0 ip=0x46fd44 code=0x7ffc0000
Jul  6 15:37:40 my-machine kernel: [369128.669519] audit: type=1326 audit(1594067860.484:14541): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=270 compat=0 ip=0x4559b1 code=0x7ffc0000
Jul  6 15:38:40 my-machine kernel: [369188.671648] audit: type=1326 audit(1594067920.488:14559): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=270 compat=0 ip=0x4559b1 code=0x7ffc0000
Jul  6 15:38:40 my-machine kernel: [369188.671726] audit: type=1326 audit(1594067920.488:14560): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=202 compat=0 ip=0x455e53 code=0x7ffc0000
```

می‌توانید با نگاه کردن به ورودی `syscall=` در هر خط، شما می‌توانید تماس‌های سیستمی مورد نیاز توسط فرآیند `http-echo` را فهمیده و با توجه به آن‌ها، می‌توانید یک پروفایل seccomp برای این کانتینر طراحی کنید.

قبل از ادامه به بخش بعدی، سرویس و پاد را حذف کنید:

```shell
kubectl delete service audit-pod --wait
kubectl delete pod audit-pod --wait --now
```

## ایجاد یک پاد با پروفایل seccomp که باعث نقض می‌شود

برای نمایش، یک پروفایل را به پاد اعمال کنید که اجازه هیچ تماس سیستمی را نمی‌دهد.

مانیفست برای این نمایش به این شرح است:

{{% code_sample file="pods/security/seccomp/ga/violation-pod.yaml" %}}

سعی کنید پاد را در کلاستر ایجاد کنید:

```shell
kubectl apply -f https://k8s.io/examples/pods/security/seccomp/ga/violation-pod.yaml
```

پاد ایجاد می‌شود، اما مشکلی وجود دارد.
اگر وضعیت پاد را بررسی کنید، باید ببینید که ناتوان در شروع بوده است.

```shell
kubectl get pod violation-pod
```

```
NAME            READY   STATUS             RESTARTS   AGE
violation-pod   0/1     CrashLoopBackOff   1          6s
```

همانطور که در مثال قبل مشاهده کردید، فرآیند `http-echo` نیاز به تعداد زیادی تماس سیستمی دارد. اینجا seccomp به این شکل تنظیم شده است که برای هر تماس سیستمی با `"defaultAction": "SCMP_ACT_ERRNO"` خطا دهد. این بسیار امن است، اما توانایی انجام کارهای معنی‌دار را از بین می‌برد. آنچه که واقعاً می‌خواهید این است که فقط امتیازهای لازم را به بار بیاورید.

پیش از ادامه به بخش بعدی، پاد را حذف کنید:

```shell
kubectl delete pod violation-pod --wait --now
```

## ایجاد یک پاد با پروفایل seccomp که فقط اجازه تماس‌های سیستمی لازم را می‌دهد

اگر به پروفایل `fine-grained.json` نگاه کنید، تعدادی از تماس‌های سیستمی که در syslog مشاهده می‌شوند را متوجه خواهید شد. در این حالت، پروفایل با تنظیم `"defaultAction": "SCMP_ACT_LOG"` تنظیم شده است. حالا پروفایل با `"defaultAction": "SCMP_ACT_ERRNO"` تنظیم شده است، اما به صورت صریح برخی از تماس‌های سیستمی را در بلوک `"action": "SCMP_ACT_ALLOW"` مجاز می‌کند. بهتر است که کانتینر با موفقیت اجرا شود و شما پیامی به syslog نفرستید.

مانیفست برای این مثال به این شرح است:

{{% code_sample file="pods/security/seccomp/ga/fine-pod.yaml" %}}

پاد را در کلاستر خود ایجاد کنید:

```shell
kubectl apply -f https://k8s.io/examples/pods/security/seccomp/ga/fine-pod.yaml
```

```shell
kubectl get pod fine-pod
```

باید ببینید که پاد با موفقیت شروع شده است:
```
NAME        READY   STATUS    RESTARTS   AGE
fine-pod   1/1     Running   0          30s
```

یک پنجره ترمینال جدید باز کنید و از `tail` برای مانیتور کردن ورودی‌های لاگ که تماس‌های `http-echo` را ذکر می‌کنند استفاده کنید:

```shell
# مسیر لاگ بر روی کامپیوتر شما ممکن است با "/var/log/syslog" متفاوت باشد
tail -f /var/log/syslog | grep 'http-echo'
```

بعد، پاد را با استفاده از یک سرویس NodePort به بیرون برگردانید:

```shell
kubectl expose pod fine-pod --type NodePort --port 5678
```

بررسی کنید که سرویس به چه درگاهی روی گره اختصاص داده شده است:

```shell
kubectl get service fine-pod
```

خروجی مشابه زیر خواهد بود:

```
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
fine-pod    NodePort   10.111.36.142   <none>        5678:32373/TCP   72s
```

از `curl` برای دسترسی به این نقطه پایانی از داخل کانتینر control plane kind استفاده کنید:

```shell
# تغییر شناسه کانتینر control plane 6a96207fed4b به شماره پورت 32373 که از "docker ps" دیده شد، انجام دهید
docker exec -it 6a96207fed4b curl localhost:32373
```

```
just made some syscalls!
```

در syslog هیچ خروجی نباید ببینید. این به این دلیل است که پروفایل تمام تماس‌های سیستمی لازم را مجاز کرده است و مشخص کرده است که در صورت فراخوانی یکی از آن‌ها خارج از لیست، خطایی ایجاد شود. این وضعیت ایده‌آل از منظر امنیتی است، اما نیاز به تحلیل دقیق‌تر برنامه دارد. خوب است اگر راهی ساده‌تر برای نزدیک‌تر شدن به این امنیت وجود داشت.

پیش از ادامه به بخش بعدی، سرویس و پاد را حذف کنید:

```shell
kubectl delete service fine-pod --wait
kubectl delete pod fine-pod --wait --now
```

```markdown
## فعالسازی استفاده از `RuntimeDefault` به عنوان پروفایل seccomp پیش‌فرض برای همه بارگذاری‌ها

{{< feature-state state="stable" for_k8s_version="v1.27" >}}

برای استفاده از پروفایل seccomp به صورت پیش‌فرض، شما باید kubelet را با فلگ خط فرمان `--seccomp-default` برای هر نودی که می‌خواهید از آن استفاده کنید، اجرا کنید.

اگر این ویژگی فعال شود، kubelet به صورت پیش‌فرض از پروفایل seccomp `RuntimeDefault` استفاده خواهد کرد که توسط runtime کانتینر تعریف شده است، به جای استفاده از حالت `Unconfined` (غیرفعال سازی seccomp). پروفایل‌های پیش‌فرض با هدف ارائه مجموعه قوانین امنیتی قوی هستند در حالی که کارایی بارگذاری را حفظ می‌کنند. احتمال دارد که پروفایل‌های پیش‌فرض بین runtime های کانتینر و نسخه‌های آن‌ها متفاوت باشند، برای مثال در مقایسه بین CRI-O و containerd.

{{< note >}}
فعال کردن این ویژگی نه API field `securityContext.seccompProfile` Kubernetes را تغییر می‌دهد و نه آنوتیشن‌های قدیمی که برای بارگذاری‌ها استفاده می‌شده است. این امکان را به کاربران می‌دهد تا هر زمان که لازم باشد، بدون تغییر واقعی پیکربندی بارگذاری را برگردانند. ابزارهایی مانند
[`crictl inspect`](https://github.com/kubernetes-sigs/cri-tools)
می‌تواند برای بررسی استفاده شده از پروفایل seccomp توسط یک کانتینر استفاده شود.
{{< /note >}}

برخی بارگذاری‌ها ممکن است نیاز به محدودیت کمتری از تماس‌های سیستمی داشته باشند نسبت به دیگران. این به این معنی است که آن‌ها ممکن است در زمان اجرا با پروفایل `RuntimeDefault` شکست بخورند. برای پیشگیری از چنین شکستی، می‌توانید:

- بارگذاری را به صورت صریح به عنوان `Unconfined` اجرا کنید.
- ویژگی `SeccompDefault` را برای نودها غیرفعال کنید. همچنین اطمینان حاصل کنید که بارگذاری‌ها بر روی نودهایی که این ویژگی غیرفعال است، زمان‌بندی شوند.
- برای بارگذاری یک پروفایل seccomp سفارشی ایجاد کنید.

اگر این ویژگی را به یک خوشه شبیه به تولید فعال می‌کردید، پروژه Kubernetes
پیشنهاد می‌دهد که این دروازه ویژگی را بر روی یک زیرمجموعه از نودهای خود فعال کنید و سپس اجرای بارگذاری را قبل از ارائه تغییر به سراسر خوشه تست کنید.

شما می‌توانید اطلاعات بیشتری درباره استراتژی ارتقاء و پایین‌روی احتمالی را در پیشنهاد ارتقاء Kubernetes مرتبط پیدا کنید:
[Enable seccomp by default](https://github.com/kubernetes/enhancements/tree/9a124fd29d1f9ddf2ff455c49a630e3181992c25/keps/sig-node/2413-seccomp-by-default#upgrade--downgrade-strategy).

Kubernetes {{< skew currentVersion >}} به شما امکان می‌دهد که پروفایل seccomp را که هنگام مشخص کردن یک پاد از آن به صورت خاص تعریف نمی‌کند، پیکربندی کنید. با این حال، همچنان نیاز به فعالسازی این پیش‌فرض برای هر نودی که می‌خواهید از آن استفاده کنید، دارید.

اگر یک خوشه Kubernetes {{< skew currentVersion >}} دارید و می‌خواهید این ویژگی را فعال کنید، به یکی از دو روش زیر عمل کنید: kubelet را با فلگ خط فرمان `--seccomp-default` اجرا کنید یا آن را از طریق [فایل پیکربندی kubelet](/docs/tasks/administer-cluster/kubelet-config-file/) فعال کنید. برای فعال کردن این دروازه ویژگی در [kind](https://kind.sigs.k8s.io)، اطمینان حاصل کنید که `kind` نسخه حداقل مورد نیاز Kubernetes را فراهم می‌کند و ویژگی `SeccompDefault` را [در تنظیمات kind](https://kind.sigs.k8s.io/docs/user/quick-start/#enable-feature-gates-in-your-cluster) فعال کنید:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    image: kindest/node:v1.28.0@sha256:9f3ff58f19dcf1a0611d11e8ac989fdb30a28f40f236f59f0bea31fb956ccf5c
    kubeadmConfigPatches:
      - |
        kind: JoinConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            seccomp-default: "true"
  - role: worker
    image: kindest/node:v1.28.0@sha256:9f3ff58f19dcf1a0611d11e8ac989fdb30a28f40f236f59f0bea31fb956ccf5c
    kubeadmConfigPatches:
      - |
        kind: JoinConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            seccomp-default: "true"
```

اگر خوشه آماده است، پس از اجرای یک پاد:

```shell
kubectl run --rm -it --restart=Never --image=alpine alpine -- sh
```

باید پروفایل seccomp پیش‌فرض را داشته باشد. این

 می‌تواند با استفاده از `docker exec` برای اجرای `crictl inspect` برای کانتینر در کارگر kind اطمینان حاصل کند:

```shell
docker exec -it kind-worker bash -c \
    'crictl inspect $(crictl ps --name=alpine -q) | jq .info.runtimeSpec.linux.seccomp'
```

```json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "architectures": ["SCMP_ARCH_X86_64", "SCMP_ARCH_X86", "SCMP_ARCH_X32"],
  "syscalls": [
    {
      "names": ["..."]
    }
  ]
}
```

## {{% heading "whatsnext" %}}

شما می‌توانید در مورد seccomp Linux بیشتر بدانید:

* [مروری بر seccomp](https://lwn.net/Articles/656307/)
* [پروفایل‌های امنیتی seccomp برای Docker](https://docs.docker.com/engine/security/seccomp/)

