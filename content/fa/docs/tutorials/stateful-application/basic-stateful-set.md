---
reviewers:
- enisoc
- erictune
- foxish
- janetkuo
- kow3ns
- smarterclayton
title: مقدمه‌ای در مدیریت StatefulSets
content_type: آموزش
weight: 10
---

<!-- overview -->
این آموزش به معرفی مدیریت برنامه‌ها با استفاده از
{{< glossary_tooltip text="StatefulSets" term_id="statefulset" >}}
می‌پردازد. این آموزش نحوه ایجاد، حذف، مقیاس‌پذیری و به‌روزرسانی پادهای StatefulSets را نشان می‌دهد.


## {{% heading "پیش‌نیازها" %}}

قبل از شروع به این آموزش، باید با مفاهیم زیر در Kubernetes آشنا شوید:

* [Pods](/docs/concepts/workloads/pods/)
* [Cluster DNS](/docs/concepts/services-networking/dns-pod-service/)
* [Headless Services](/docs/concepts/services-networking/service/#headless-services)
* [PersistentVolumes](/docs/concepts/storage/persistent-volumes/)
* [PersistentVolume Provisioning](https://github.com/kubernetes/examples/tree/master/staging/persistent-volume-provisioning/)
* ابزار خط فرمان [kubectl](/docs/reference/kubectl/kubectl/)

{{% include "task-tutorial-prereqs.md" %}}
باید `kubectl` را به گونه‌ای پیکربندی کنید که از فضای نام `default` استفاده کند.
اگر از یک خوشه موجود استفاده می‌کنید، مطمئن شوید که استفاده از فضای نام پیش‌فرض آن خوشه برای تمرین مناسب است. ایده‌آل است که در یک خوشه تمرینی که هیچ بارگذاری واقعی اجرا نمی‌کند تمرین کنید.

همچنین مفید است که صفحه مفهوم در مورد [StatefulSets](/docs/concepts/workloads/controllers/statefulset/) را بخوانید.

{{< note >}}
این آموزش فرض می‌کند که خوشه شما به طور پویا به پیوسته‌سازی PersistentVolumes پیکربندی شده است. همچنین نیاز دارید که [StorageClass پیش‌فرض](/docs/concepts/storage/storage-classes/#default-storageclass) را داشته باشید.
اگر خوشه شما به پیوسته‌سازی ذخیره سازی پیوسته‌سازی نشده باشد، شما باید قبل از شروع این آموزش دو حجم ۱ گیگابایتی را به صورت دستی ایجاد کنید و خوشه خود را به گونه‌ای پیکربندی کنید که این PersistentVolumes به الگوهای PersistentVolumeClaim تعریف شده در StatefulSet متناظر باشند.
{{< /note >}}

## {{% heading "اهداف" %}}

StatefulSets برای استفاده با برنامه‌های با حالت و سیستم‌های توزیع‌شده طراحی شده‌اند. با این حال، مدیریت برنامه‌های با حالت و سیستم‌های توزیع‌شده در Kubernetes یک موضوع گسترده و پیچیده است. به منظور نمایش ویژگی‌های ابتدایی یک StatefulSet و جلوگیری از اشتباه در مفهوم اولیه، یک برنامه وب ساده با استفاده از یک StatefulSet را مستقر خواهید کرد.

پس از این آموزش، با موارد زیر آشنا خواهید شد.

* نحوه ایجاد یک StatefulSet
* نحوه مدیریت پادهای یک StatefulSet
* نحوه حذف یک StatefulSet
* نحوه مقیاس‌پذیری یک StatefulSet
* نحوه به‌روزرسانی پادهای یک StatefulSet

<!-- lessoncontent -->

## ایجاد یک StatefulSet

شروع به ایجاد یک StatefulSet (و سرویسی که بر آن وابسته است) با استفاده از مثال زیر کنید. این مشابه مثالی است که در [StatefulSets](/docs/concepts/workloads/controllers/statefulset/) توضیح داده شده است. این یک [headless Service](/docs/concepts/services-networking/service/#headless-services) به نام `nginx` ایجاد می‌کند تا آدرس‌های IP پادهای StatefulSet به نام `web` را منتشر کند.

{{% code_sample file="application/web/web.yaml" %}}

شما نیاز به حداقل دو پنجره ترمینال دارید. در ترمینال اول، از [`kubectl get`](/docs/reference/generated/kubectl/kubectl-commands/#get) برای تماشای ایجاد پادهای StatefulSet استفاده کنید.

```shell
# از این ترمینال برای اجرای دستوراتی که با --watch مشخص می‌شود استفاده کنید
# این watch را پایان دهید زمانی که از شما خواسته شود که watch جدیدی را شروع کنید
kubectl get pods --watch -l app=nginx
```

در ترمینال دوم، از [`kubectl apply`](/docs/reference/generated/kubectl/kubectl-commands/#apply) برای ایجاد headless Service و StatefulSet استفاده کنید:

```shell
kubectl apply -f https://k8s.io/examples/application/web/web.yaml
```
```
service/nginx created
statefulset.apps/web created
```
دستور بالا دو پاد ایجاد می‌کند که هر کدام یک وب‌سرور [NGINX](https://www.nginx.com) را اجرا می‌کنند. سرویس `nginx` را دریافت کنید...
```shell
kubectl get service nginx
```
```
NAME      TYPE         CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx     ClusterIP    None         <none>        80/TCP    12s
```
...سپس StatefulSet `web` را دریافت کنید تا تأیید کنید که هر دو با موفقیت ایجاد شده‌اند:
```shell
kubectl get statefulset web
```
```
NAME   READY   AGE
web    2/2     37s
```

### ایجاد ترتیبی پادها

به صورت پیش‌فرض، یک StatefulSet پادهای خود را به ترتیبی دقیق ایجاد می‌کند.

برای یک StatefulSet با _n_ کپی، زمانی که پادها در حال مستقر شدن هستند، به ترتیب و به صورت متوالی از _{0..n-1}_ ایجاد می‌شوند. خروجی دستور `kubectl get` را در ترمینال اول بررسی کنید. در نهایت، خروجی به شکل مثال زیر خواهد بود.

```shell
# یک watch جدید را شروع نکنید؛
# این باید در حال حاضر اجرا باشد
kubectl get pods --watch -l app=nginx
```
```
NAME      READY     STATUS    RESTARTS   AGE
web-0     0/1       Pending   0          0s
web-0     0/1       Pending   0         0s
web-0     0/1       ContainerCreating   0         0s
web-0     1/1       Running   0         19s
web-1     0/1       Pending   0         0s
web-1     0/1       Pending   0         0s
web-1     0/1       ContainerCreating   0         0s
web-1     1/1       Running   0         18s
```

توجه کنید که پاد `web-1` تا زمانی که پاد `web-0` به حالت _Running_ نرسیده است (نگاه کنید به [فاز پاد](/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase)) و _Ready_ نشده است (نگاه کنید به `type` در [شرایط پاد](/docs/concepts/workloads/pods/pod-lifecycle/#pod-conditions))، شروع به کار نمی‌کند.

بعداً در این آموزش شما [استارتاپ موازی](#parallel-pod-management) را تمرین خواهید کرد.

{{< note >}}
برای تنظیم شماره ترتیبی که به هر پاد در یک StatefulSet اختصاص داده می‌شود، نگاه کنید به [ترتیب شروع](/docs/concepts/workloads/controllers/statefulset/#start-ordinal).
{{< /note >}}

## پادها در یک StatefulSet

پادها در یک StatefulSet دارای یک شاخص ترتیبی یکتا و هویت شبکه‌ای پایدار هستند.

### بررسی شاخص ترتیبی پادها

پادهای StatefulSet را دریافت کنید:

```shell
kubectl get pods -l app=nginx
```
```
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          1m
web-1     1/1       Running   0          1m
```

همانطور که در مفهوم [StatefulSets](/docs/concepts/workloads/controllers/statefulset/) ذکر شد، پادهای یک StatefulSet دارای هویتی چسبنده و یکتا هستند. این هویت بر اساس یک شاخص ترتیبی یکتا است که توسط StatefulSet {{< glossary_tooltip term_id="controller" text="کنترلر">}} به هر پاد اختصاص داده می‌شود.  
نام‌های پادها به شکل `<نام statefulset>-<شاخص ترتیبی>` هستند.
از آنجایی که StatefulSet `web` دارای دو کپی است، دو پاد `web-0` و `web-1` را ایجاد می‌کند.

### استفاده از هویت‌های شبکه‌ای پایدار

هر پاد دارای یک hostname پایدار بر اساس شاخص ترتیبی خود است. از [`kubectl exec`](/docs/reference/generated/kubectl/kubectl-commands/#exec) برای اجرای دستور `hostname` در هر پاد استفاده کنید:

```shell
for i in 0 1; do kubectl exec "web-$i" -- sh -c 'hostname'; done
```
```
web-0
web-1
```
از [`kubectl run`](/docs/reference/generated/kubectl/kubectl-commands/#run) استفاده کنید تا یک کانتینر که دستور `nslookup` را از پکیج `dnsutils` فراهم می‌کند، اجرا شود.
با استفاده از `nslookup` روی hostnames پادها، می‌توانید آدرس‌های DNS داخل خوشه آنها را بررسی کنید:

```shell
kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
```
که یک شل جدید را شروع می‌کند. در آن شل جدید، اجرا کنید:
```shell
# این را در شل کانتینر dns-test اجرا کنید
nslookup web-0.nginx
```
خروجی مشابه زیر است:
```
Server:    10.0.0.10
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-0.nginx
Address 1: 10.244.1.6

nslookup web-1.nginx
Server:    10.0.0.10
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-1.nginx
Address 1: 10.244.2.6
```

(و اکنون از شل کانتینر خارج شوید: `exit`)

CNAME سرویس بدون سر به رکوردهای SRV اشاره می‌کند (یک رکورد برای هر پاد که در حالت Running و Ready است). رکوردهای SRV به رکوردهای A اشاره می‌کنند که آدرس‌های IP پادها را شامل می‌شوند.

در یک ترمینال، پادهای StatefulSet را مشاهده کنید:

```shell
# یک watch جدید شروع کنید
# این watch را پایان دهید زمانی که مشاهده کردید که حذف کامل شده است
kubectl get pod --watch -l app=nginx
```
در یک ترمینال دوم، از [`kubectl delete`](/docs/reference/generated/kubectl/kubectl-commands/#delete) برای حذف تمام پادهای StatefulSet استفاده کنید:

```shell
kubectl delete pod -l app=nginx
```
```
pod "web-0" deleted
pod "web-1" deleted
```

صبر کنید تا StatefulSet آنها را مجدداً راه‌اندازی کند و هر دو پاد به حالت Running و Ready منتقل شوند:

```shell
# این باید در حال حاضر در حال اجرا باشد
kubectl get pod --watch -l app=nginx
```
```
NAME      READY     STATUS              RESTARTS   AGE
web-0     0/1       ContainerCreating   0          0s
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          2s
web-1     0/1       Pending   0         0s
web-1     0/1       Pending   0         0s
web-1     0/1       ContainerCreating   0         0s
web-1     1/1       Running   0         34s
```

از `kubectl exec` و `kubectl run` برای مشاهده hostnames پادها و ورودی‌های DNS داخل خوشه استفاده کنید. ابتدا hostnames پادها را مشاهده کنید:

```shell
for i in 0 1; do kubectl exec web-$i -- sh -c 'hostname'; done
```
```
web-0
web-1
```
سپس، اجرا کنید:
```shell
kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
```
که یک شل جدید را شروع می‌کند.
در آن شل جدید، اجرا کنید:
```shell
# Run this in the dns-test container shell
nslookup web-0.nginx
```خروجی مشابه زیر است:
```
Server:    10.0.0.10
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-0.nginx
Address 1: 10.244.1.7

nslookup web-1.nginx
Server:    10.0.0.10
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-1.nginx
Address 1: 10.244.2.8
```

(و اکنون از شل کانتینر خارج شوید: `exit`)

شماره‌های ترتیبی پادها، hostnames، رکوردهای SRV و نام‌های رکورد A تغییر نکرده‌اند، اما آدرس‌های IP مربوط به پادها ممکن است تغییر کرده باشد. در خوشه‌ای که برای این آموزش استفاده شده است، تغییر کرده‌اند. به همین دلیل مهم است که برنامه‌های دیگر را برای اتصال به پادها در یک StatefulSet بر اساس آدرس IP یک پاد خاص پیکربندی نکنید (اتصال به پادها با حل hostname آنها مشکلی ندارد).

#### کشف پادهای خاص در یک StatefulSet

اگر نیاز دارید اعضای فعال یک StatefulSet را پیدا کرده و به آنها متصل شوید، باید CNAME سرویس بدون سر (`nginx.default.svc.cluster.local`) را استعلام کنید. رکوردهای SRV مربوط به CNAME فقط شامل پادهای StatefulSet هستند که در حالت Running و Ready هستند.

اگر برنامه شما از قبل منطق اتصال را پیاده‌سازی کرده است که تست‌های liveness و readiness را انجام می‌دهد، می‌توانید از رکوردهای SRV پادها (`web-0.nginx.default.svc.cluster.local`، `web-1.nginx.default.svc.cluster.local`) استفاده کنید، زیرا پایدار هستند و برنامه شما می‌تواند آدرس‌های پادها را زمانی که به حالت Running و Ready منتقل می‌شوند کشف کند.

اگر برنامه شما می‌خواهد هر پاد سالمی در یک StatefulSet را پیدا کند و بنابراین نیازی به پیگیری هر پاد خاص ندارد، می‌توانید به آدرس IP یک سرویس `type: ClusterIP` متصل شوید که توسط پادهای آن StatefulSet پشتیبانی می‌شود. می‌توانید از همان سرویسی که StatefulSet را ردیابی می‌کند (که در `serviceName` StatefulSet مشخص شده است) یا یک سرویس جداگانه که مجموعه پادهای مناسب را انتخاب می‌کند، استفاده کنید.

### نوشتن به ذخیره‌سازی پایدار

PersistentVolumeClaims برای `web-0` و `web-1` را دریافت کنید:

```shell
kubectl get pvc -l app=nginx
```
خروجی مشابه زیر است:
```
NAME        STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
www-web-0   Bound     pvc-15c268c7-b507-11e6-932f-42010a800002   1Gi        RWO           48s
www-web-1   Bound     pvc-15c79307-b507-11e6-932f-42010a800002   1Gi        RWO           48s
```

کنترلر StatefulSet دو {{< glossary_tooltip text="PersistentVolumeClaims" term_id="persistent-volume-claim" >}} ایجاد کرده است که به دو {{< glossary_tooltip text="PersistentVolumes" term_id="persistent-volume" >}} متصل شده‌اند.

از آنجا که خوشه‌ای که برای این آموزش استفاده شده است به صورت خودکار PersistentVolumes را فراهم می‌کند، PersistentVolumes به صورت خودکار ایجاد و متصل شده‌اند.

وب‌سرور NGINX به طور پیش‌فرض یک فایل ایندکس از مسیر `/usr/share/nginx/html/index.html` ارائه می‌دهد. فیلد `volumeMounts` در `spec` StatefulSet تضمین می‌کند که دایرکتوری `/usr/share/nginx/html` توسط یک PersistentVolume پشتیبانی می‌شود.

hostnames پادها را به فایل‌های `index.html` آنها بنویسید و تأیید کنید که وب‌سرورهای NGINX hostnames را ارائه می‌دهند:

```shell
for i in 0 1; do kubectl exec "web-$i" -- sh -c 'echo "$(hostname)" > /usr/share/nginx/html/index.html'; done

for i in 0 1; do kubectl exec -i -t "web-$i" -- curl http://localhost/; done
```
```
web-0
web-1
```

{{< note >}}
اگر به جای آن پاسخ **403 Forbidden** برای دستور curl بالا دیدید، باید مجوزهای دایرکتوری که توسط `volumeMounts` متصل شده است را اصلاح کنید (به دلیل [باگ در استفاده از حجم‌های hostPath](https://github.com/kubernetes/kubernetes/issues/2630))، با اجرای:

`for i in 0 1; do kubectl exec web-$i -- chmod 755 /usr/share/nginx/html; done`

قبل از تلاش مجدد برای اجرای دستور `curl` بالا.
{{< /note >}}

در یک ترمینال، پادهای StatefulSet را مشاهده کنید:

```shell
# این watch را پایان دهید زمانی که به انتهای بخش رسیدید.
# در ابتدای "Scaling a StatefulSet" یک watch جدید شروع خواهید کرد.
kubectl get pod --watch -l app=nginx
```

در یک ترمینال دوم، تمام پادهای StatefulSet را حذف کنید:

```shell
kubectl delete pod -l app=nginx
```
```
pod "web-0" deleted
pod "web-1" deleted
```
خروجی دستور `kubectl get` را در ترمینال اول بررسی کنید و صبر کنید تا همه پادها به حالت Running و Ready منتقل شوند.

```shell
# این باید در حال حاضر در حال اجرا باشد
kubectl get pod --watch -l app=nginx
```
```
NAME      READY     STATUS              RESTARTS   AGE
web-0     0/1       ContainerCreating   0          0s
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          2s
web-1     0/1       Pending   0         0s
web-1     0/1       Pending   0         0s
web-1     0/1       ContainerCreating   0         0s
web-1     1/1       Running   0         34s
```

تأیید کنید که وب‌سرورها همچنان hostnames خود را ارائه می‌دهند:

```
for i in 0 1; do kubectl exec -i -t "web-$i" -- curl http://localhost/; done
```
```
web-0
web-1
```

حتی اگر `web-0` و `web-1` مجدداً زمان‌بندی شده‌اند، آنها همچنان hostnames خود را ارائه می‌دهند زیرا PersistentVolumes مربوط به PersistentVolumeClaims آنها به volumeMounts آنها دوباره متصل می‌شوند. مهم نیست که `web-0` و `web-1` روی کدام نود زمان‌بندی شده‌اند، PersistentVolumes آنها به نقاط اتصال مناسب متصل خواهد شد.
## مقیاس‌دهی یک StatefulSet

مقیاس‌دهی یک StatefulSet به معنای افزایش یا کاهش تعداد نمونه‌ها (مقیاس‌دهی افقی) است. این کار با به‌روزرسانی فیلد `replicas` انجام می‌شود. می‌توانید از [`kubectl scale`](/docs/reference/generated/kubectl/kubectl-commands/#scale) یا [`kubectl patch`](/docs/reference/generated/kubectl/kubectl-commands/#patch) برای مقیاس‌دهی یک StatefulSet استفاده کنید.

### مقیاس‌دهی بالا

مقیاس‌دهی بالا به معنای اضافه کردن نمونه‌های بیشتر است. در صورتی که برنامه شما قادر به توزیع کار در سراسر StatefulSet باشد، مجموعه جدید و بزرگتر از پادها می‌توانند کار بیشتری انجام دهند.

در یک پنجره ترمینال، پادهای StatefulSet را مشاهده کنید:

```shell
# اگر قبلاً یک watch در حال اجرا دارید، می‌توانید از آن استفاده کنید.
# در غیر این صورت، یکی را شروع کنید.
# این watch را زمانی که 5 پاد سالم برای StatefulSet وجود داشته باشد پایان دهید
kubectl get pods --watch -l app=nginx
```

در یک پنجره ترمینال دیگر، از `kubectl scale` برای مقیاس‌دهی تعداد نمونه‌ها به 5 استفاده کنید:

```shell
kubectl scale sts web --replicas=5
```
```
statefulset.apps/web scaled
```

خروجی دستور `kubectl get` را در ترمینال اول بررسی کنید و صبر کنید تا سه پاد اضافی به حالت Running و Ready منتقل شوند.

```shell
# این باید در حال حاضر در حال اجرا باشد
kubectl get pod --watch -l app=nginx
```
```
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          2h
web-1     1/1       Running   0          2h
NAME      READY     STATUS    RESTARTS   AGE
web-2     0/1       Pending   0          0s
web-2     0/1       Pending   0         0s
web-2     0/1       ContainerCreating   0         0s
web-2     1/1       Running   0         19s
web-3     0/1       Pending   0         0s
web-3     0/1       Pending   0         0s
web-3     0/1       ContainerCreating   0         0s
web-3     1/1       Running   0         18s
web-4     0/1       Pending   0         0s
web-4     0/1       Pending   0         0s
web-4     0/1       ContainerCreating   0         0s
web-4     1/1       Running   0         19s
```

کنترلر StatefulSet تعداد نمونه‌ها را مقیاس‌دهی کرد. همانطور که در [ایجاد StatefulSet](#ordered-pod-creation) دیدید، کنترلر StatefulSet هر پاد را به صورت متوالی با توجه به شاخص ترتیبی آن ایجاد کرد و منتظر ماند تا هر پاد به حالت Running و Ready برسد و سپس پاد بعدی را راه‌اندازی کرد.

### مقیاس‌دهی پایین

مقیاس‌دهی پایین به معنای کاهش تعداد نمونه‌ها است. به عنوان مثال، ممکن است این کار را انجام دهید زیرا سطح ترافیک به یک سرویس کاهش یافته است و در مقیاس فعلی منابع بیکار وجود دارد.

در یک پنجره ترمینال، پادهای StatefulSet را مشاهده کنید:

```shell
# این watch را زمانی که تنها 3 پاد برای StatefulSet وجود داشته باشد پایان دهید
kubectl get pod --watch -l app=nginx
```

در یک پنجره ترمینال دیگر، از `kubectl patch` برای مقیاس‌دهی StatefulSet به سه نمونه استفاده کنید:

```shell
kubectl patch sts web -p '{"spec":{"replicas":3}}'
```
```
statefulset.apps/web patched
```

منتظر بمانید تا `web-4` و `web-3` به حالت Terminating منتقل شوند.

```shell
# این باید در حال حاضر در حال اجرا باشد
kubectl get pods --watch -l app=nginx
```
```
NAME      READY     STATUS              RESTARTS   AGE
web-0     1/1       Running             0          3h
web-1     1/1       Running             0          3h
web-2     1/1       Running             0          55s
web-3     1/1       Running             0          36s
web-4     0/1       ContainerCreating   0          18s
NAME      READY     STATUS    RESTARTS   AGE
web-4     1/1       Running   0          19s
web-4     1/1       Terminating   0         24s
web-4     1/1       Terminating   0         24s
web-3     1/1       Terminating   0         42s
web-3     1/1       Terminating   0         42s
```

### خاتمه‌یافته‌های ترتیب‌دار

صفحه کنترل یکی یکی پادها را حذف کرد، به ترتیب معکوس با توجه به شاخص ترتیبی آنها، و منتظر ماند تا هر پاد به طور کامل خاموش شود قبل از حذف پاد بعدی.

PersistentVolumeClaims مربوط به StatefulSet را دریافت کنید:

```shell
kubectl get pvc -l app=nginx
```
```
NAME        STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
www-web-0   Bound     pvc-15c268c7-b507-11e6-932f-42010a800002   1Gi        RWO           13h
www-web-1   Bound     pvc-15c79307-b507-11e6-932f-42010a800002   1Gi        RWO           13h
www-web-2   Bound     pvc-e1125b27-b508-11e6-932f-42010a800002   1Gi        RWO           13h
www-web-3   Bound     pvc-e1176df6-b508-11e6-932f-42010a800002   1Gi        RWO           13h
www-web-4   Bound     pvc-e11bb5f8-b508-11e6-932f-42010a800002   1Gi        RWO           13h
```

هنوز پنج PersistentVolumeClaims و پنج PersistentVolumes وجود دارد. هنگامی که ذخیره‌سازی پایدار یک پاد را بررسی کردید، دیدید که PersistentVolumes متصل به پادهای یک StatefulSet هنگامی که پادهای StatefulSet حذف می‌شوند، حذف نمی‌شوند. این همچنان درست است زمانی که حذف پادها به دلیل کاهش مقیاس StatefulSet انجام می‌شود.
## به‌روزرسانی StatefulSets

کنترلر StatefulSet از به‌روزرسانی‌های خودکار پشتیبانی می‌کند. استراتژی به‌کاررفته توسط فیلد `spec.updateStrategy` شیء API StatefulSet تعیین می‌شود. این ویژگی می‌تواند برای ارتقای تصاویر کانتینرها، درخواست‌ها و/یا محدودیت‌های منابع، برچسب‌ها و حاشیه‌نویسی‌های پادهای یک StatefulSet استفاده شود.

دو استراتژی به‌روزرسانی معتبر وجود دارد: `RollingUpdate` (پیش‌فرض) و `OnDelete`.

### RollingUpdate {#rolling-update}

استراتژی به‌روزرسانی `RollingUpdate` تمام پادهای یک StatefulSet را در ترتیب معکوس به‌روزرسانی می‌کند، در حالی که تضمین‌های StatefulSet را رعایت می‌کند.

شما می‌توانید به‌روزرسانی‌های یک StatefulSet که از استراتژی `RollingUpdate` استفاده می‌کند را به _پارتیشن‌ها_ تقسیم کنید، با مشخص کردن `.spec.updateStrategy.rollingUpdate.partition`. این کار را در ادامه این آموزش تمرین خواهید کرد.

ابتدا یک به‌روزرسانی ساده را امتحان کنید.

در یک پنجره ترمینال، StatefulSet `web` را پچ کنید تا تصویر کانتینر را دوباره تغییر دهید:

```shell
kubectl patch statefulset web --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"registry.k8s.io/nginx-slim:0.24"}]'
```
```
statefulset.apps/web patched
```

در یک ترمینال دیگر، پادهای StatefulSet را مشاهده کنید:

```shell
# این watch را زمانی که rollout کامل شد پایان دهید
#
# اگر مطمئن نیستید، یک دقیقه دیگر آن را در حال اجرا بگذارید
kubectl get pod -l app=nginx --watch
```

خروجی مشابه زیر است:
```
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          7m
web-1     1/1       Running   0          7m
web-2     1/1       Running   0          8m
web-2     1/1       Terminating   0         8m
web-2     1/1       Terminating   0         8m
web-2     0/1       Terminating   0         8m
web-2     0/1       Terminating   0         8m
web-2     0/1       Terminating   0         8m
web-2     0/1       Terminating   0         8m
web-2     0/1       Pending   0         0s
web-2     0/1       Pending   0         0s
web-2     0/1       ContainerCreating   0         0s
web-2     1/1       Running   0         19s
web-1     1/1       Terminating   0         8m
web-1     0/1       Terminating   0         8m
web-1     0/1       Terminating   0         8m
web-1     0/1       Terminating   0         8m
web-1     0/1       Pending   0         0s
web-1     0/1       Pending   0         0s
web-1     0/1       ContainerCreating   0         0s
web-1     1/1       Running   0         6s
web-0     1/1       Terminating   0         7m
web-0     1/1       Terminating   0         7m
web-0     0/1       Terminating   0         7m
web-0     0/1       Terminating   0         7m
web-0     0/1       Terminating   0         7m
web-0     0/1       Terminating   0         7m
web-0     0/1       Pending   0         0s
web-0     0/1       Pending   0         0s
web-0     0/1       ContainerCreating   0         0s
web-0     1/1       Running   0         10s
```

پادهای StatefulSet به ترتیب معکوس به‌روزرسانی می‌شوند. کنترلر StatefulSet هر پاد را خاتمه می‌دهد و منتظر می‌ماند تا به حالت Running و Ready منتقل شود، سپس پاد بعدی را به‌روزرسانی می‌کند. توجه داشته باشید که حتی اگر کنترلر StatefulSet به پاد بعدی نرود تا زمانی که جانشین ترتیبی آن Running و Ready شود، هر پادی که در طول به‌روزرسانی با خطا مواجه شود را به نسخه موجود آن پاد بازمی‌گرداند.

پادهایی که قبلاً به‌روزرسانی شده‌اند به نسخه به‌روزشده بازگردانده می‌شوند و پادهایی که هنوز به‌روزرسانی نشده‌اند به نسخه قبلی بازگردانده می‌شوند. به این ترتیب، کنترلر تلاش می‌کند تا برنامه را سالم نگه دارد و به‌روزرسانی را در حضور خطاهای متناوب سازگار نگه دارد.

برای مشاهده تصاویر کانتینر پادها، دستور زیر را اجرا کنید:

```shell
for p in 0 1 2; do kubectl get pod "web-$p" --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'; echo; done
```
```
registry.k8s.io/nginx-slim:0.24
registry.k8s.io/nginx-slim:0.24
registry.k8s.io/nginx-slim:0.24
```

تمام پادهای StatefulSet اکنون تصویر کانتینر قبلی را اجرا می‌کنند.

{{< note >}}
شما می‌توانید از `kubectl rollout status sts/<name>` نیز برای مشاهده وضعیت یک به‌روزرسانی نوردی StatefulSet استفاده کنید
{{< /note >}}

#### مرحله‌بندی یک به‌روزرسانی

شما می‌توانید به‌روزرسانی‌های یک StatefulSet که از استراتژی `RollingUpdate` استفاده می‌کند را به _پارتیشن‌ها_ تقسیم کنید، با مشخص کردن `.spec.updateStrategy.rollingUpdate.partition`.

برای اطلاعات بیشتر، می‌توانید [به‌روزرسانی‌های نوردی تقسیم‌شده](/docs/concepts/workloads/controllers/statefulset/#partitions) را در صفحه مفهوم StatefulSet مطالعه کنید.

شما می‌توانید یک به‌روزرسانی را با استفاده از فیلد `partition` درون `.spec.updateStrategy.rollingUpdate` مرحله‌بندی کنید.

ابتدا، StatefulSet `web` را پچ کنید تا یک پارتیشن به فیلد `updateStrategy` اضافه کنید:

```shell
# مقدار "partition" تعیین می‌کند که تغییرات به کدام ordinals اعمال می‌شود
# مطمئن شوید که از عددی بزرگتر از آخرین ordinal برای StatefulSet استفاده کنید
kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"partition":3}}}}'
```
```
statefulset.apps/web patched
```

دوباره StatefulSet را پچ کنید تا تصویر کانتینر مورد استفاده این StatefulSet را تغییر دهید:

```shell
kubectl patch statefulset web --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"registry.k8s.io/nginx-slim:0.21"}]'
```
```
statefulset.apps/web patched
```

یک پاد در StatefulSet را حذف کنید:

```shell
kubectl delete pod web-2
```
```
pod "web-2" deleted
```

منتظر بمانید تا پاد جایگزین `web-2` به حالت Running و Ready منتقل شود:

```shell
# زمانی که web-2 سالم شد، watch را پایان دهید
kubectl get pod -l app=nginx --watch
```
```
NAME      READY     STATUS              RESTARTS   AGE
web-0     1/1       Running             0          4m
web-1     1/1       Running             0          4m
web-2     0/1       ContainerCreating   0          11s
web-2     1/1       Running   0         18s
```

تصویر کانتینر پاد را دریافت کنید:

```shell
kubectl get pod web-2 --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'
```
```
registry.k8s.io/nginx-slim:0.24
```

توجه کنید که، حتی با وجود اینکه استراتژی به‌روزرسانی `RollingUpdate` است، StatefulSet پاد را با تصویر کان

تینر اصلی بازیابی کرد. این به این دلیل است که ordinal پاد کمتر از `partition` مشخص‌شده توسط `updateStrategy` است.
## انجام rollout canary

اکنون قصد داریم یک rollout canary را برای تغییر مرحله‌ای انجام دهیم. برای انجام این کار، `partition` را که قبلاً تنظیم کرده‌ایم کاهش می‌دهیم.

### کاهش partition برای canary rollout

ابتدا، StatefulSet را برای کاهش `partition` پچ کنید:

```shell
# مقدار "partition" باید با بالاترین ordinal موجود برای StatefulSet مطابقت داشته باشد
kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"partition":2}}}}'
```

خروجی:
```
statefulset.apps/web patched
```

اکنون، کنترل پلین جایگزینی برای `web-2` را تحریک می‌کند. منتظر بمانید تا پاد جدید `web-2` به حالت Running و Ready برسد.

### مشاهده پادهای جدید

در یک پنجره ترمینال دیگر، پادهای StatefulSet را مشاهده کنید:

```shell
kubectl get pod -l app=nginx --watch
```

خروجی مشابه زیر است:

```
NAME      READY     STATUS              RESTARTS   AGE
web-0     1/1       Running             0          4m
web-1     1/1       Running             0          4m
web-2     0/1       ContainerCreating   0          11s
web-2     1/1       Running   0         18s
```

تصویر کانتینر پاد `web-2` را دریافت کنید:

```shell
kubectl get pod web-2 --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'
```

خروجی:
```
registry.k8s.io/nginx-slim:0.21
```

وقتی `partition` را تغییر دادید، کنترلر StatefulSet به‌طور خودکار پاد `web-2` را به‌روزرسانی کرد زیرا ordinal پاد بزرگتر یا مساوی `partition` بود.

### حذف پاد `web-1`

پاد `web-1` را حذف کنید:

```shell
kubectl delete pod web-1
```

خروجی:
```
pod "web-1" deleted
```

منتظر بمانید تا پاد `web-1` به حالت Running و Ready برسد:

```shell
kubectl get pod -l app=nginx --watch
```

خروجی مشابه زیر است:

```
NAME      READY     STATUS        RESTARTS   AGE
web-0     1/1       Running       0          6m
web-1     0/1       Terminating   0          6m
web-2     1/1       Running       0          2m
web-1     0/1       Terminating   0         6m
web-1     0/1       Terminating   0         6m
web-1     0/1       Terminating   0         6m
web-1     0/1       Pending   0         0s
web-1     0/1       Pending   0         0s
web-1     0/1       ContainerCreating   0         0s
web-1     1/1       Running   0         18s
```

تصویر کانتینر پاد `web-1` را دریافت کنید:

```shell
kubectl get pod web-1 --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'
```

خروجی:
```
registry.k8s.io/nginx-slim:0.24
```

پاد `web-1` به پیکربندی اصلی خود بازگردانده شد زیرا ordinal پاد کمتر از partition بود.

### rollout phased

شما می‌توانید یک rollout phased را با استفاده از یک به‌روزرسانی نوردی partition شده انجام دهید.

ابتدا partition را به `0` تغییر دهید:

```shell
kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"partition":0}}}}'
```

خروجی:
```
statefulset.apps/web patched
```

منتظر بمانید تا تمام پادهای StatefulSet به حالت Running و Ready برسند:

```shell
kubectl get pod -l app=nginx --watch
```

خروجی مشابه زیر است:

```
NAME      READY     STATUS              RESTARTS   AGE
web-0     1/1       Running             0          3m
web-1     0/1       ContainerCreating   0          11s
web-2     1/1       Running             0          2m
web-1     1/1       Running   0         18s
web-0     1/1       Terminating   0         3m
web-0     1/1       Terminating   0         3m
web-0     0/1       Terminating   0         3m
web-0     0/1       Terminating   0         3m
web-0     0/1       Terminating   0         3m
web-0     0/1       Terminating   0         3m
web-0     0/1       Pending   0         0s
web-0     0/1       Pending   0         0s
web-0     0/1       ContainerCreating   0         0s
web-0     1/1       Running   0         3s
```

جزئیات تصویر کانتینر پادها را دریافت کنید:

```shell
for p in 0 1 2; do kubectl get pod "web-$p" --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'; echo; done
```

خروجی:
```
registry.k8s.io/nginx-slim:0.21
registry.k8s.io/nginx-slim:0.21
registry.k8s.io/nginx-slim:0.21
```

با تغییر `partition` به `0`، اجازه دادید که StatefulSet فرآیند به‌روزرسانی را ادامه دهد.

### استراتژی به‌روزرسانی OnDelete

شما می‌توانید این استراتژی به‌روزرسانی را برای یک StatefulSet با تنظیم `.spec.template.updateStrategy.type` به `OnDelete` انتخاب کنید.

پچ کردن StatefulSet `web` برای استفاده از استراتژی به‌روزرسانی `OnDelete`:

```shell
kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"OnDelete"}}}'
```

خروجی:
```
statefulset.apps/web patched
```

زمانی که این استراتژی به‌روزرسانی انتخاب شود، کنترلر StatefulSet به‌طور خودکار پادها را هنگامی که یک تغییر در فیلد `.spec.template` StatefulSet انجام شود به‌روزرسانی نمی‌کند. شما باید فرآیند rollout را خودتان مدیریت کنید - یا به‌صورت دستی، یا با استفاده از اتوماسیون جداگانه.
## حذف StatefulSets

StatefulSet از حذف غیرپایدار و حذف پایدار پشتیبانی می‌کند. در حذف غیرپایدار، پادهای StatefulSet در هنگام حذف StatefulSet حذف نمی‌شوند. در حذف پایدار، هم StatefulSet و هم پادهای آن حذف می‌شوند.

### حذف غیرپایدار

برای مشاهده پادهای StatefulSet:

```shell
# این دستور را تا زمانی که هیچ پادی برای StatefulSet وجود نداشته باشد، اجرا کنید
kubectl get pods --watch -l app=nginx
```

از دستور `kubectl delete` برای حذف StatefulSet استفاده کنید و پارامتر `--cascade=orphan` را به دستور اضافه کنید تا Kubernetes تنها StatefulSet را حذف کند و پادهای آن را حذف نکند:

```shell
kubectl delete statefulset web --cascade=orphan
```

خروجی:
```
statefulset.apps "web" deleted
```

برای بررسی وضعیت پادها:

```shell
kubectl get pods -l app=nginx
```

خروجی:
```
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          6m
web-1     1/1       Running   0          7m
web-2     1/1       Running   0          5m
```

حتی با اینکه `web` حذف شده است، همه پادها همچنان در حال اجرا و آماده هستند.

پاد `web-0` را حذف کنید:

```shell
kubectl delete pod web-0
```

خروجی:
```
pod "web-0" deleted
```

برای بررسی وضعیت پادهای StatefulSet:

```shell
kubectl get pods -l app=nginx
```

خروجی:
```
NAME      READY     STATUS    RESTARTS   AGE
web-1     1/1       Running   0          10m
web-2     1/1       Running   0          7m
```

از آنجا که StatefulSet `web` حذف شده است، `web-0` دوباره راه‌اندازی نشده است.

### بازیابی StatefulSet

برای مشاهده پادهای StatefulSet:

```shell
# این دستور را تا زمانی که دوباره پادها را راه‌اندازی کنید، اجرا کنید
kubectl get pods --watch -l app=nginx
```

در یک ترمینال دیگر، StatefulSet را دوباره ایجاد کنید. توجه داشته باشید که اگر سرویس `nginx` را حذف نکرده‌اید، خطایی مبنی بر وجود سرویس از قبل دریافت خواهید کرد.

```shell
kubectl apply -f https://k8s.io/examples/application/web/web.yaml
```

خروجی:
```
statefulset.apps/web created
service/nginx unchanged
```

این خطا را نادیده بگیرید. تنها نشان می‌دهد که تلاش شده است سرویس headless `nginx` ایجاد شود، در حالی که این سرویس از قبل وجود دارد.

برای بررسی وضعیت پادها:

```shell
kubectl get pods --watch -l app=nginx
```

خروجی مشابه زیر است:

```
NAME      READY     STATUS    RESTARTS   AGE
web-1     1/1       Running   0          16m
web-2     1/1       Running   0          2m
NAME      READY     STATUS    RESTARTS   AGE
web-0     0/1       Pending   0          0s
web-0     0/1       Pending   0         0s
web-0     0/1       ContainerCreating   0         0s
web-0     1/1       Running   0         18s
web-2     1/1       Terminating   0         3m
web-2     0/1       Terminating   0         3m
web-2     0/1       Terminating   0         3m
web-2     0/1       Terminating   0         3m
```

هنگامی که StatefulSet `web` دوباره ایجاد شد، ابتدا `web-0` را راه‌اندازی کرد. از آنجا که `web-1` از قبل در حال اجرا و آماده بود، هنگامی که `web-0` به حالت Running و Ready رسید، این پاد را پذیرفت. چون StatefulSet را با `replicas` برابر با 2 دوباره ایجاد کردید، پس از ایجاد `web-0` و پس از تأیید `web-1` که از قبل در حال اجرا و آماده بود، `web-2` متوقف شد.

برای بررسی محتوای فایل `index.html` که توسط وب سرورهای پادها سرو می‌شود:

```shell
for i in 0 1; do kubectl exec -i -t "web-$i" -- curl http://localhost/; done
```

خروجی:
```
web-0
web-1
```

حتی اگر هم StatefulSet و هم پاد `web-0` را حذف کرده باشید، این پاد همچنان hostname اصلی خود را در فایل `index.html` سرو می‌کند. این به این دلیل است که StatefulSet هیچ وقت PersistentVolumes مرتبط با یک پاد را حذف نمی‌کند. وقتی StatefulSet را دوباره ایجاد کردید و آن `web-0` را راه‌اندازی کرد، PersistentVolume اصلی آن دوباره mount شد.

### حذف پایدار

برای مشاهده پادهای StatefulSet:

```shell
# این دستور را تا بخش بعدی اجرا کنید
kubectl get pods --watch -l app=nginx
```

در یک ترمینال دیگر، StatefulSet را دوباره حذف کنید. این بار، پارامتر `--cascade=orphan` را حذف کنید.

```shell
kubectl delete statefulset web
```

خروجی:
```
statefulset.apps "web" deleted
```

برای بررسی وضعیت پادها:

```shell
kubectl get pods --watch -l app=nginx
```

خروجی مشابه زیر است:

```
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          11m
web-1     1/1       Running   0          27m
NAME      READY     STATUS        RESTARTS   AGE
web-0     1/1       Terminating   0          12m
web-1     1/1       Terminating   0         29m
web-0     0/1       Terminating   0         12m
web-0     0/1       Terminating   0         12m
web-0     0/1       Terminating   0         12m
web-1     0/1       Terminating   0         29m
web-1     0/1       Terminating   0         29m
web-1     0/1       Terminating   0         29m
```

همانطور که در بخش [Scaling Down](#scaling-down) مشاهده کردید، پادها یکی یکی و به ترتیب معکوس ordinal هایشان حذف می‌شوند. قبل از حذف یک پاد، کنترلر StatefulSet منتظر می‌ماند تا جانشین آن پاد کاملاً حذف شود.

توجه:
هرچند حذف پایدار یک StatefulSet را به همراه پادهای آن حذف می‌کند، اما cascade سرویس headless مرتبط با StatefulSet را حذف نمی‌کند. شما باید سرویس `nginx` را به صورت دستی حذف کنید.

```shell
kubectl delete service nginx
```

خروجی:
```
service "nginx" deleted
```

دوباره StatefulSet و سرویس headless را ایجاد کنید:

```shell
kubectl apply -f https://k8s.io/examples/application/web/web.yaml
```

خروجی:
```
service/nginx created
statefulset.apps/web created
```

وقتی تمام پادهای StatefulSet به حالت Running و Ready رسیدند، محتوای فایل‌های `index.html` آن‌ها را دریافت کنید:

```shell
for i in 0 1; do kubectl exec -i -t "web-$i" -- curl http://localhost/; done
```

خروجی:
```
web-0
web-1
```

حتی اگر StatefulSet و تمام پادهای آن را به طور کامل حذف کرده باشید، پادها با PersistentVolumes خود دوباره ایجاد می‌شوند و `web-0` و `web-1` همچنان hostnames خود را سرو می‌کنند.

در نهایت، سرویس `nginx` را حذف کنید:

```shell
kubectl delete service nginx
```

خروجی:
```
service "nginx" deleted
```

و StatefulSet `web` را حذف کنید:

```shell
kubectl delete statefulset web
```

خروجی:
```
statefulset "web" deleted
```
## سیاست مدیریت پادها

برای برخی از سیستم‌های توزیع‌شده، تضمین‌های ترتیب StatefulSet غیرضروری و/یا نامطلوب هستند. این سیستم‌ها فقط نیاز به یکتایی و هویت دارند.

شما می‌توانید یک [سیاست مدیریت پاد](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#pod-management-policies) را برای اجتناب از این ترتیب سختگیرانه مشخص کنید؛ یا `OrderedReady` (پیش‌فرض)، یا `Parallel`.

### مدیریت پاد OrderedReady

مدیریت پاد `OrderedReady` پیش‌فرض برای StatefulSet‌ها است. این تنظیم به کنترل‌کننده StatefulSet می‌گوید که به تضمین‌های ترتیب احترام بگذارد.

از این سیاست زمانی استفاده کنید که برنامه شما نیاز دارد یا انتظار دارد که تغییرات، مانند انتشار نسخه جدید برنامه، به ترتیب عدد ترتیبی (شماره پاد) که StatefulSet ارائه می‌دهد، انجام شود. به عبارت دیگر، اگر پادهای `app-0`، `app-1` و `app-2` را داشته باشید، Kubernetes ابتدا `app-0` را به‌روزرسانی می‌کند و آن را بررسی می‌کند. پس از انجام بررسی‌ها، Kubernetes `app-1` و در نهایت `app-2` را به‌روزرسانی می‌کند.

اگر دو پاد دیگر اضافه کنید، Kubernetes ابتدا `app-3` را تنظیم می‌کند و منتظر می‌ماند تا سالم شود و سپس `app-4` را مستقر می‌کند.

به دلیل اینکه این تنظیم پیش‌فرض است، شما قبلاً استفاده از آن را تمرین کرده‌اید.

### مدیریت پاد Parallel

گزینه دیگر، مدیریت پاد `Parallel` است که به کنترل‌کننده StatefulSet می‌گوید تا همه پادها را به‌صورت موازی راه‌اندازی یا خاتمه دهد و منتظر نشود که پادها به حالت `Running` و `Ready` درآیند یا به‌طور کامل خاتمه یابند قبل از اینکه پاد دیگری را راه‌اندازی یا خاتمه دهد.

گزینه مدیریت پاد `Parallel` فقط بر رفتار عملیات‌های مقیاس‌گذاری تأثیر می‌گذارد. به‌روزرسانی‌ها تحت تأثیر قرار نمی‌گیرند؛ Kubernetes همچنان تغییرات را به ترتیب انجام می‌دهد. برای این آموزش، برنامه بسیار ساده است: یک وب‌سرور که نام میزبان خود را می‌گوید (زیرا این یک StatefulSet است، نام میزبان هر پاد متفاوت و قابل پیش‌بینی است).

{{% code_sample file="application/web/web-parallel.yaml" %}}

این مانیفست با آنچه قبلاً دانلود کرده‌اید یکسان است، به جز اینکه `.spec.podManagementPolicy` از StatefulSet `web` به `Parallel` تنظیم شده است.

در یک ترمینال، پادهای StatefulSet را مشاهده کنید.

```shell
# این watch را تا پایان بخش باز نگه دارید
kubectl get pod -l app=nginx --watch
```

در یک ترمینال دیگر، StatefulSet را برای مدیریت پاد `Parallel` بازپیکربندی کنید:

```shell
kubectl apply -f https://k8s.io/examples/application/web/web-parallel.yaml
```
```
service/nginx updated
statefulset.apps/web updated
```

ترمینالی که watch را اجرا می‌کنید باز نگه دارید. در یک پنجره ترمینال دیگر، StatefulSet را مقیاس‌دهی کنید:

```shell
kubectl scale statefulset/web --replicas=5
```
```
statefulset.apps/web scaled
```

خروجی ترمینالی که فرمان `kubectl get` را اجرا می‌کند بررسی کنید. ممکن است شبیه به این باشد:

```
web-3     0/1       Pending   0         0s
web-3     0/1       Pending   0         0s
web-3     0/1       Pending   0         7s
web-3     0/1       ContainerCreating   0         7s
web-2     0/1       Pending   0         0s
web-4     0/1       Pending   0         0s
web-2     1/1       Running   0         8s
web-4     0/1       ContainerCreating   0         4s
web-3     1/1       Running   0         26s
web-4     1/1       Running   0         2s
```

StatefulSet سه پاد جدید را راه‌اندازی کرد و منتظر نشد تا اولین پاد به حالت Running و Ready درآید قبل از راه‌اندازی پاد دوم و سوم.

این رویکرد زمانی مفید است که بار کاری شما عنصری دارای وضعیت داشته باشد یا پادها باید بتوانند یکدیگر را با نام‌گذاری قابل پیش‌بینی شناسایی کنند، و به ویژه اگر گاهی نیاز دارید که ظرفیت بیشتری را سریعاً فراهم کنید. اگر این سرویس وب ساده برای آموزش به طور ناگهانی 1,000,000 درخواست اضافی در دقیقه دریافت کند، شما می‌خواهید چند پاد دیگر اجرا کنید - اما همچنین نمی‌خواهید منتظر بمانید تا هر پاد جدید راه‌اندازی شود. شروع پادهای اضافی به صورت موازی زمان بین درخواست ظرفیت اضافی و داشتن آن برای استفاده را کاهش می‌دهد.

## {{% heading "cleanup" %}}

شما باید دو ترمینال باز داشته باشید تا آماده باشید فرمان‌های `kubectl` را به عنوان بخشی از پاکسازی اجرا کنید.

```shell
kubectl delete sts web
# sts مخفف statefulset است
```

می‌توانید `kubectl get` را مشاهده کنید تا ببینید آن پادها حذف می‌شوند.
```shell
# هنگامی که آنچه را که نیاز دارید مشاهده کردید، watch را پایان دهید
kubectl get pod -l app=nginx --watch
```
```
web-3     1/1       Terminating   0         9m
web-2     1/1       Terminating   0         9m
web-3     1/1       Terminating   0         9m
web-2     1/1       Terminating   0         9m
web-1     1/1       Terminating   0         44m
web-0     1/1       Terminating   0         44m
web-0     0/1       Terminating   0         44m
web-3     0/1       Terminating   0         9m
web-2     0/1       Terminating   0         9m
web-1     0/1       Terminating   0         44m
web-0     0/1       Terminating   0         44m
web-2     0/1       Terminating   0         9m
web-2     0/1       Terminating   0         9m
web-2     0/1       Terminating   0         9m
web-1     0/1       Terminating   0         44m
web-1     0/1       Terminating   0         44m
web-1     0/1       Terminating   0         44m
web-0     0/1       Terminating   0         44m
web-0     0/1       Terminating   0         44m
web-0     0/1       Terminating   0         44m
web-3     0/1       Terminating   0         9m
web-3     0/1       Terminating   0         9m
web-3     0/1       Terminating   0         9m
```

در طی حذف، یک StatefulSet همه پادها را به طور همزمان حذف می‌کند؛ منتظر نمی‌ماند تا جانشین ترتیبی یک پاد حذف شود قبل از حذف آن پاد.

ترمینالی که فرمان `kubectl get` را اجرا می‌کند ببندید و سرویس `nginx` را حذف کنید:

```shell
kubectl delete svc nginx
```

رسانه‌های ذخیره‌سازی دائمی برای PersistentVolume‌هایی که در این آموزش استفاده شده‌اند حذف کنید.

```shell
kubectl get pvc
```
```
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
www-web-0   Bound    pvc-2bf00408-d366-4a12-bad0-1869c65d0bee   1Gi        RWO            standard       25m
www-web-1   Bound    pvc-ba3bfe9c-413e-4b95-a2c0-3ea8a54dbab4   1Gi        RWO            standard       24m
www-web-2   Bound    pvc-cba6cfa6-3a47-486b-a138-db5930207eaf   1Gi        RWO            standard       15m
www-web-3   Bound    pvc-0c04d7f0-787a-4977-8da3-d9d3a6d8d

752   1Gi        RWO            standard       15m
www-web-4   Bound    pvc-b2c73489-e70b-4a4e-9ec1-9eab439aa43e   1Gi        RWO            standard       14m
```

```shell
kubectl get pv
```
```
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
pvc-0c04d7f0-787a-4977-8da3-d9d3a6d8d752   1Gi        RWO            Delete           Bound    default/www-web-3   standard                15m
pvc-2bf00408-d366-4a12-bad0-1869c65d0bee   1Gi        RWO            Delete           Bound    default/www-web-0   standard                25m
pvc-b2c73489-e70b-4a4e-9ec1-9eab439aa43e   1Gi        RWO            Delete           Bound    default/www-web-4   standard                14m
pvc-ba3bfe9c-413e-4b95-a2c0-3ea8a54dbab4   1Gi        RWO            Delete           Bound    default/www-web-1   standard                24m
pvc-cba6cfa6-3a47-486b-a138-db5930207eaf   1Gi        RWO            Delete           Bound    default/www-web-2   standard                15m
```

```shell
kubectl delete pvc www-web-0 www-web-1 www-web-2 www-web-3 www-web-4
```

```
persistentvolumeclaim "www-web-0" deleted
persistentvolumeclaim "www-web-1" deleted
persistentvolumeclaim "www-web-2" deleted
persistentvolumeclaim "www-web-3" deleted
persistentvolumeclaim "www-web-4" deleted
```

```shell
kubectl get pvc
```

```
No resources found in default namespace.
```
{{< note >}}
شما همچنین باید رسانه‌های ذخیره‌سازی دائمی برای PersistentVolume‌هایی که در این آموزش استفاده شده‌اند حذف کنید.
مراحل لازم را دنبال کنید، بر اساس محیط، پیکربندی ذخیره‌سازی و روش تأمین، تا اطمینان حاصل کنید که همه ذخیره‌ها بازیابی می‌شوند.
{{< /note >}}