---
reviewers:
- enisoc
- erictune
- foxish
- janetkuo
- kow3ns
- smarterclayton
title: اجرای یک برنامه Stateful تکرارشونده
content_type: tutorial
weight: 30
---

<!-- overview -->

این صفحه نشان می‌دهد که چگونه یک برنامه stateful تکرارشونده را با استفاده از
{{< glossary_tooltip term_id="statefulset" >}} اجرا کنید.
این برنامه یک پایگاه داده MySQL تکرارشونده است. توپولوژی مثال شامل
یک سرور اولیه و چندین نسخه تکراری است که از همگام‌سازی مبتنی بر ردیف استفاده می‌کند.

{{< note >}}
**این یک پیکربندی تولیدی نیست**. تنظیمات MySQL بر روی پیش‌فرض‌های ناامن باقی مانده‌اند تا تمرکز بر روی الگوهای کلی اجرای برنامه‌های stateful در Kubernetes باشد.
{{< /note >}}

## {{% heading "prerequisites" %}}

- {{< include "task-tutorial-prereqs.md" >}}
- {{< include "default-storage-class-prereqs.md" >}}
- این آموزش فرض می‌کند که با
  [PersistentVolumes](/docs/concepts/storage/persistent-volumes/)
  و [StatefulSets](/docs/concepts/workloads/controllers/statefulset/)،
  و همچنین مفاهیم اصلی دیگر مانند [Pods](/docs/concepts/workloads/pods/)،
  [Services](/docs/concepts/services-networking/service/)، و
  [ConfigMaps](/docs/tasks/configure-pod-container/configure-pod-configmap/) آشنا هستید.
- آشنایی با MySQL مفید است، اما این آموزش قصد دارد الگوهای عمومی را ارائه دهد که باید برای سیستم‌های دیگر نیز مفید باشد.
- شما از namespace پیش‌فرض یا namespace دیگری که شامل اشیاء متناقض نیست استفاده می‌کنید.

## {{% heading "objectives" %}}

- استقرار یک توپولوژی MySQL تکراری با یک StatefulSet.
- ارسال ترافیک مشتری MySQL.
- مشاهده مقاومت در برابر خرابی.
- افزایش و کاهش مقیاس StatefulSet.

<!-- lessoncontent -->

## استقرار MySQL

استقرار MySQL مثال شامل یک ConfigMap، دو Service،
و یک StatefulSet است.

### ایجاد یک ConfigMap {#configmap}

ConfigMap را از فایل پیکربندی YAML زیر ایجاد کنید:

{{% code_sample file="application/mysql/mysql-configmap.yaml" %}}

```shell
kubectl apply -f https://k8s.io/examples/application/mysql/mysql-configmap.yaml
```

این ConfigMap تنظیمات `my.cnf` را فراهم می‌کند که به شما امکان کنترل مستقل
پیکربندی بر روی سرور اصلی MySQL و نسخه‌های آن را می‌دهد.
در این حالت، شما می‌خواهید سرور اصلی بتواند لاگ‌های همگام‌سازی را به نسخه‌ها سرویس دهد
و می‌خواهید نسخه‌ها هرگونه نوشتنی که از طریق همگام‌سازی نمی‌آید را رد کنند.

هیچ چیز خاصی در مورد خود ConfigMap نیست که باعث شود قسمت‌های مختلف
به Podهای مختلف اعمال شوند.
هر Pod تصمیم می‌گیرد کدام قسمت را در زمان شروع کارش بررسی کند،
بر اساس اطلاعاتی که توسط کنترل‌کننده StatefulSet ارائه شده است.

### ایجاد Service‌ها {#services}

Service‌ها را از فایل پیکربندی YAML زیر ایجاد کنید:

{{% code_sample file="application/mysql/mysql-services.yaml" %}}

```shell
kubectl apply -f https://k8s.io/examples/application/mysql/mysql-services.yaml
```

Service بدون سرور یک خانه برای ورودی‌های DNS فراهم می‌کند که StatefulSet
{{< glossary_tooltip text="controllers" term_id="controller" >}} برای هر
Pod که بخشی از مجموعه است ایجاد می‌کند.
از آنجایی که Service بدون سرور به نام `mysql` است، Podها با
حل کردن `<pod-name>.mysql` از درون هر Pod دیگری در همان Kubernetes
خوشه و namespace قابل دسترسی هستند.

Service مشتری، به نام `mysql-read`، یک Service عادی با IP خوشه‌ای خود است
که اتصالات را بین تمام Podهای MySQL که گزارش Ready بودن می‌دهند توزیع می‌کند. مجموعه نقطه‌های پایانی شامل سرور اصلی MySQL و همه نسخه‌ها می‌شود.

توجه داشته باشید که تنها کوئری‌های خواندن می‌توانند از Service مشتری متعادل استفاده کنند.
از آنجایی که تنها یک سرور اصلی MySQL وجود دارد، مشتریان باید برای اجرای
نوشتن‌ها به طور مستقیم به Pod اصلی MySQL (از طریق ورودی DNS آن در Service بدون سرور) متصل شوند.

### ایجاد StatefulSet {#statefulset}

در نهایت، StatefulSet را از فایل پیکربندی YAML زیر ایجاد کنید:

{{% code_sample file="application/mysql/mysql-statefulset.yaml" %}}

```shell
kubectl apply -f https://k8s.io/examples/application/mysql/mysql-statefulset.yaml
```

می‌توانید با اجرای دستور زیر پیشرفت راه‌اندازی را مشاهده کنید:

```shell
kubectl get pods -l app=mysql --watch
```

بعد از مدتی، باید همه ۳ Pod به حالت `Running` برسند:

```
NAME      READY     STATUS    RESTARTS   AGE
mysql-0   2/2       Running   0          2m
mysql-1   2/2       Running   0          1m
mysql-2   2/2       Running   0          1m
```

برای لغو مشاهده **Ctrl+C** را فشار دهید.

{{< note >}}
اگر پیشرفتی مشاهده نمی‌کنید، اطمینان حاصل کنید که یک فراهم‌کننده PersistentVolume دینامیک
فعال دارید، همانطور که در [پیش‌نیازها](#before-you-begin) ذکر شده است.
{{< /note >}}

این manifest از تکنیک‌های مختلفی برای مدیریت Podهای stateful به عنوان بخشی از
یک StatefulSet استفاده می‌کند. بخش بعدی برخی از این تکنیک‌ها را برجسته می‌کند تا توضیح دهد
چه اتفاقی می‌افتد زمانی که StatefulSet Podها را ایجاد می‌کند.

## درک اولیه‌سازی Podهای stateful

کنترل‌کننده StatefulSet Podها را یکی یکی، به ترتیب شاخص
ordinal آن‌ها راه‌اندازی می‌کند.
صبر می‌کند تا هر Pod گزارش Ready بودن بدهد قبل از اینکه Pod بعدی را شروع کند.

علاوه بر این، کنترل‌کننده به هر Pod یک نام یکتا و پایدار از فرم
`<statefulset-name>-<ordinal-index>` اختصاص می‌دهد که منجر به Podهایی به نام `mysql-0`,
`mysql-1`، و `mysql-2` می‌شود.

قالب Pod در manifest فوق از این
خواص برای اجرای منظم راه‌اندازی همگام‌سازی MySQL بهره می‌برد.
```markdown
### تولید پیکربندی

قبل از شروع هر یک از کانتینرها در spec پاد، ابتدا پاد هر 
[کانتینر ابتدایی](/docs/concepts/workloads/pods/init-containers/)
را به ترتیب تعریف شده اجرا می‌کند.

اولین کانتینر ابتدایی، به نام `init-mysql`، فایل‌های پیکربندی ویژه MySQL را بر اساس شاخص ترتیبی تولید می‌کند.

اسکریپت شاخص ترتیبی خود را با استخراج آن از انتهای نام پاد که توسط دستور `hostname` بازگردانده می‌شود، تعیین می‌کند.
سپس شاخص ترتیبی (با یک جابجایی عددی برای جلوگیری از مقادیر رزرو شده) را در فایلی به نام `server-id.cnf` در دایرکتوری `conf.d` MySQL ذخیره می‌کند.
این فرآیند شناسه یکتا و پایدار که توسط StatefulSet فراهم شده را به حوزه شناسه‌های سرور MySQL که نیاز به ویژگی‌های مشابه دارند ترجمه می‌کند.

اسکریپت در کانتینر `init-mysql` همچنین `primary.cnf` یا `replica.cnf` را از ConfigMap با کپی کردن محتوا به `conf.d` اعمال می‌کند.
از آنجا که توپولوژی مثال شامل یک سرور MySQL اصلی و تعداد نامحدودی از نسخه‌ها است، اسکریپت شاخص ترتیبی `0` را به سرور اصلی اختصاص می‌دهد و بقیه را به عنوان نسخه‌ها تعیین می‌کند.
ترکیب با کنترل‌کننده StatefulSet 
[ضمانت ترتیب استقرار](/docs/concepts/workloads/controllers/statefulset/#deployment-and-scaling-guarantees)،
این تضمین می‌کند که سرور MySQL اصلی قبل از ایجاد نسخه‌ها آماده است تا بتوانند شروع به همگام‌سازی کنند.

### کلون کردن داده‌های موجود

به طور کلی، زمانی که یک پاد جدید به عنوان یک نسخه به مجموعه می‌پیوندد، باید فرض کند که سرور MySQL اصلی ممکن است از قبل داده‌هایی روی آن داشته باشد. همچنین باید فرض کند که لاگ‌های همگام‌سازی ممکن است تا آغاز زمان باز نگردند.
این فرضیات محافظه‌کارانه کلیدی برای امکان افزایش و کاهش اندازه StatefulSet در طول زمان است، به جای اینکه اندازه آن به اندازه اولیه ثابت باشد.

دومین کانتینر ابتدایی، به نام `clone-mysql`، عملیات کلون را بر روی یک پاد نسخه‌ای در اولین باری که روی یک PersistentVolume خالی راه‌اندازی می‌شود انجام می‌دهد.
این به معنای کپی کردن تمام داده‌های موجود از یک پاد در حال اجرا دیگر است،
تا حالت محلی آن به اندازه کافی همگام باشد که بتواند همگام‌سازی را از سرور اصلی آغاز کند.

خود MySQL مکانیزمی برای انجام این کار ارائه نمی‌دهد، بنابراین مثال از یک ابزار محبوب منبع‌باز به نام Percona XtraBackup استفاده می‌کند.
در طول کلون، ممکن است سرور MySQL منبع کاهش عملکرد را تجربه کند.
برای به حداقل رساندن تأثیر بر روی سرور اصلی MySQL، اسکریپت به هر پاد دستور می‌دهد که از پادی که شاخص ترتیبی آن یک واحد کمتر است کلون کند.
این کار می‌کند زیرا کنترل‌کننده StatefulSet همیشه اطمینان می‌دهد که پاد `N` قبل از شروع پاد `N+1` آماده است.

### شروع همگام‌سازی

پس از اینکه کانتینرهای ابتدایی با موفقیت تکمیل شدند، کانتینرهای معمولی اجرا می‌شوند.
پادهای MySQL شامل یک کانتینر `mysql` که سرور `mysqld` واقعی را اجرا می‌کند و یک کانتینر `xtrabackup` که به عنوان
[sidecar](/blog/2015/06/the-distributed-system-toolkit-patterns)
عمل می‌کند.

کانتینر `xtrabackup` به فایل‌های داده کلون شده نگاه می‌کند و تعیین می‌کند که آیا نیاز است همگام‌سازی MySQL را بر روی نسخه آغاز کند.
در صورت نیاز، منتظر می‌ماند تا `mysqld` آماده شود و سپس دستورات
`CHANGE MASTER TO` و `START SLAVE` را با پارامترهای همگام‌سازی استخراج شده از فایل‌های کلون XtraBackup اجرا می‌کند.

هنگامی که یک نسخه شروع به همگام‌سازی می‌کند، سرور اصلی MySQL خود را به خاطر می‌آورد و در صورت راه‌اندازی مجدد سرور یا قطع ارتباط به طور خودکار دوباره متصل می‌شود.
همچنین، از آنجایی که نسخه‌ها به دنبال سرور اصلی با نام پایدار DNS آن
(`mysql-0.mysql`) هستند، به طور خودکار سرور اصلی را پیدا می‌کنند حتی اگر به دلیل برنامه‌ریزی مجدد یک IP پاد جدید دریافت کند.

در نهایت، پس از شروع همگام‌سازی، کانتینر `xtrabackup` به اتصالات پادهای دیگر که درخواست کلون داده دارند گوش می‌دهد.
این سرور به طور نامحدود فعال باقی می‌ماند تا در صورتی که StatefulSet افزایش یابد یا در صورتی که پاد بعدی PersistentVolumeClaim خود را از دست بدهد و نیاز به تکرار کلون داشته باشد.

## ارسال ترافیک مشتری

شما می‌توانید کوئری‌های آزمایشی را به سرور MySQL اصلی (نام میزبان `mysql-0.mysql`)
با اجرای یک کانتینر موقت با تصویر `mysql:5.7` و اجرای
باینری مشتری `mysql` ارسال کنید.

```shell
kubectl run mysql-client --image=mysql:5.7 -i --rm --restart=Never --\
  mysql -h mysql-0.mysql <<EOF
CREATE DATABASE test;
CREATE TABLE test.messages (message VARCHAR(250));
INSERT INTO test.messages VALUES ('hello');
EOF
```

از نام میزبان `mysql-read` برای ارسال کوئری‌های آزمایشی به هر سروری که گزارش Ready بودن می‌دهد استفاده کنید:

```shell
kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --\
  mysql -h mysql-read -e "SELECT * FROM test.messages"
```

باید خروجی مشابه زیر را دریافت کنید:

```
Waiting for pod default/mysql-client to be running, status is Pending, pod ready: false
+---------+
| message |
+---------+
| hello   |
+---------+
pod "mysql-client" deleted
```

برای نشان دادن اینکه سرویس `mysql-read` اتصالات را بین سرورها توزیع می‌کند، می‌توانید `SELECT @@server_id` را در یک حلقه اجرا کنید:

```shell
kubectl run mysql-client-loop --image=mysql:5.7 -i -t --rm --restart=Never --\
  bash -ic "while sleep 1; do mysql -h mysql-read -e 'SELECT @@server_id,NOW()'; done"
```

باید مشاهده کنید که `@@server_id` گزارش شده به صورت تصادفی تغییر می‌کند، زیرا یک نقطه پایانی متفاوت ممکن است در هر تلاش اتصال انتخاب شود:

```
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         100 | 2006-01-02 15:04:05 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         102 | 2006-01-02 15:04:06 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         101 | 2006-01-02 15:04:07 |
+-------------+---------------------+
```

می‌توانید با فشار دادن **Ctrl+C** زمانی که می‌خواهید حلقه را متوقف کنید، اما بهتر است آن را در یک پنجره دیگر فعال نگه دارید تا اثرات مراحل بعدی را مشاهده کنید.

## شبیه‌سازی خرابی پاد و نود {#simulate-pod-and-node-downtime}

برای نشان دادن افزایش دسترسی به خواندن از مجموعه نسخه‌ها به جای یک سرور واحد، حلقه `SELECT @@server_id` از بالا را در حال اجرا نگه دارید در حالی که یک پاد را از حالت Ready خارج می‌کنید.

### شکستن پروب آمادگی

[پروب آمادگی](/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-readiness-probes)
برای کانتینر `mysql` دستور `mysql -h 127.0.0.1 -e 'SELECT 1'` را اجرا می‌کند تا مطمئن شود که سرور فعال است و قادر به اجرای کوئری‌ها است.

یکی از راه‌های مجبور کردن این پروب آمادگی به شکست، شکستن آن دستور است:

```shell
kubectl exec mysql-2 -c mysql -- mv /usr/bin/mysql /usr/bin/mysql.off
```

این دستور به فایل‌سیستم واقعی کانتینر برای پاد `mysql-2` می‌رود و دستور `mysql` را تغییر نام می‌دهد تا پروب آمادگی نتواند آن را پیدا کند.
بعد از چند ثانیه، پاد باید گزارش دهد که یکی از کانتینرهای آن Ready نیست،
که می‌توانید با اجرای دستور زیر بررسی کنید:

```shell
kubectl get pod mysql-2
```

دنبال `1

/2` در ستون `READY` باشید:

```
NAME      READY     STATUS    RESTARTS   AGE
mysql-2   1/2       Running   0          3m
```

در این نقطه، باید مشاهده کنید که حلقه `SELECT @@server_id` شما به اجرا ادامه می‌دهد،
اگرچه دیگر هرگز `102` را گزارش نمی‌دهد.
به خاطر داشته باشید که اسکریپت `init-mysql` شناسه سرور را به صورت `100 + $ordinal` تعریف کرده است،
بنابراین شناسه سرور `102` به پاد `mysql-2` مربوط می‌شود.

اکنون پاد را تعمیر کنید و باید بعد از چند ثانیه دوباره در خروجی حلقه ظاهر شود:

```shell
kubectl exec mysql-2 -c mysql -- mv /usr/bin/mysql.off /usr/bin/mysql
```

### حذف پادها

StatefulSet همچنین پادها را بازسازی می‌کند اگر حذف شوند، مشابه کاری که یک ReplicaSet برای پادهای بی‌حالت انجام می‌دهد.

```shell
kubectl delete pod mysql-2
```

کنترل‌کننده StatefulSet متوجه می‌شود که دیگر پاد `mysql-2` وجود ندارد،
و یک پاد جدید با همان نام و مرتبط با همان PersistentVolumeClaim ایجاد می‌کند.
باید مشاهده کنید که شناسه سرور `102` برای مدتی از خروجی حلقه ناپدید می‌شود و سپس به طور خودکار باز می‌گردد.
```

### خارج کردن یک نود

اگر کلاستر Kubernetes شما دارای چندین نود است، می‌توانید با اجرای دستور
[drain](/docs/reference/generated/kubectl/kubectl-commands/#drain) خرابی نود را شبیه‌سازی کنید (مانند زمانی که نودها ارتقاء می‌یابند).

ابتدا مشخص کنید که یکی از پادهای MySQL روی کدام نود قرار دارد:

```shell
kubectl get pod mysql-2 -o wide
```

نام نود باید در ستون آخر نمایش داده شود:

```
NAME      READY     STATUS    RESTARTS   AGE       IP            NODE
mysql-2   2/2       Running   0          15m       10.244.5.27   kubernetes-node-9l2t
```

سپس، با اجرای دستور زیر، نود را خارج کنید که این نود را محصور می‌کند تا پادهای جدید نتوانند روی آن زمان‌بندی شوند و سپس هر پاد موجودی را خارج می‌کند.
`<node-name>` را با نام نود که در مرحله قبل پیدا کردید جایگزین کنید.

{{< caution >}}
خارج کردن یک نود می‌تواند بر سایر بارهای کاری و برنامه‌هایی که روی همان نود در حال اجرا هستند تاثیر بگذارد. فقط در کلاستر آزمایشی مرحله زیر را انجام دهید.
{{< /caution >}}

```shell
# به توصیه‌های بالا درباره تأثیر بر سایر بارهای کاری توجه کنید
kubectl drain <node-name> --force --delete-emptydir-data --ignore-daemonsets
```

حالا می‌توانید مشاهده کنید که پاد روی نود دیگری زمان‌بندی می‌شود:

```shell
kubectl get pod mysql-2 -o wide --watch
```

باید چیزی شبیه به این باشد:

```
NAME      READY   STATUS          RESTARTS   AGE       IP            NODE
mysql-2   2/2     Terminating     0          15m       10.244.1.56   kubernetes-node-9l2t
[...]
mysql-2   0/2     Pending         0          0s        <none>        kubernetes-node-fjlm
mysql-2   0/2     Init:0/2        0          0s        <none>        kubernetes-node-fjlm
mysql-2   0/2     Init:1/2        0          20s       10.244.5.32   kubernetes-node-fjlm
mysql-2   0/2     PodInitializing 0          21s       10.244.5.32   kubernetes-node-fjlm
mysql-2   1/2     Running         0          22s       10.244.5.32   kubernetes-node-fjlm
mysql-2   2/2     Running         0          30s       10.244.5.32   kubernetes-node-fjlm
```

و باز هم باید مشاهده کنید که شناسه سرور `102` برای مدتی از خروجی حلقه 
`SELECT @@server_id` ناپدید می‌شود و سپس باز می‌گردد.

اکنون نود را به حالت عادی بازگردانید:

```shell
kubectl uncordon <node-name>
```

## مقیاس‌بندی تعداد نسخه‌ها

وقتی از تکرار MySQL استفاده می‌کنید، می‌توانید ظرفیت کوئری‌های خواندن خود را با اضافه کردن نسخه‌ها افزایش دهید.
برای یک StatefulSet، می‌توانید این کار را با یک دستور ساده انجام دهید:

```shell
kubectl scale statefulset mysql  --replicas=5
```

با اجرای دستور زیر مشاهده کنید که پادهای جدید بالا می‌آیند:

```shell
kubectl get pods -l app=mysql --watch
```

هنگامی که بالا آمدند، باید مشاهده کنید که شناسه‌های سرور `103` و `104` در خروجی حلقه 
`SELECT @@server_id` ظاهر می‌شوند.

همچنین می‌توانید اطمینان حاصل کنید که این سرورهای جدید داده‌هایی که قبل از وجودشان اضافه کرده‌اید را دارند:

```shell
kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --\
  mysql -h mysql-3.mysql -e "SELECT * FROM test.messages"
```

```
Waiting for pod default/mysql-client to be running, status is Pending, pod ready: false
+---------+
| message |
+---------+
| hello   |
+---------+
pod "mysql-client" deleted
```

کاهش مقیاس نیز به سادگی انجام می‌شود:

```shell
kubectl scale statefulset mysql --replicas=3
```

{{< note >}}
اگرچه افزایش مقیاس به صورت خودکار PersistentVolumeClaim های جدید ایجاد می‌کند، کاهش مقیاس به صورت خودکار این PVC ها را حذف نمی‌کند.

این امکان را به شما می‌دهد که آن PVC های اولیه را نگه دارید تا افزایش مقیاس دوباره سریع‌تر انجام شود یا قبل از حذف آنها داده‌ها را استخراج کنید.
{{< /note >}}

می‌توانید این را با اجرای دستور زیر مشاهده کنید:

```shell
kubectl get pvc -l app=mysql
```

که نشان می‌دهد همه ۵ PVC همچنان وجود دارند، حتی با وجود اینکه StatefulSet را به ۳ کاهش داده‌اید:

```
NAME           STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
data-mysql-0   Bound     pvc-8acbf5dc-b103-11e6-93fa-42010a800002   10Gi       RWO           20m
data-mysql-1   Bound     pvc-8ad39820-b103-11e6-93fa-42010a800002   10Gi       RWO           20m
data-mysql-2   Bound     pvc-8ad69a6d-b103-11e6-93fa-42010a800002   10Gi       RWO           20m
data-mysql-3   Bound     pvc-50043c45-b1c5-11e6-93fa-42010a800002   10Gi       RWO           2m
data-mysql-4   Bound     pvc-500a9957-b1c5-11e6-93fa-42010a800002   10Gi       RWO           2m
```

اگر قصد ندارید از PVC های اضافی استفاده کنید، می‌توانید آنها را حذف کنید:

```shell
kubectl delete pvc data-mysql-3
kubectl delete pvc data-mysql-4
```

## {{% heading "cleanup" %}}

1. حلقه `SELECT @@server_id` را با فشار دادن **Ctrl+C** در ترمینال آن لغو کنید، یا از ترمینال دیگری دستور زیر را اجرا کنید:

   ```shell
   kubectl delete pod mysql-client-loop --now
   ```

2. StatefulSet را حذف کنید. این عمل همچنین پادها را به مرحله پایان می‌برد.

   ```shell
   kubectl delete statefulset mysql
   ```

3. اطمینان حاصل کنید که پادها ناپدید می‌شوند.
   ممکن است مدتی طول بکشد تا عملیات پایان کامل شود.

   ```shell
   kubectl get pods -l app=mysql
   ```

   شما متوجه می‌شوید که پادها پایان یافته‌اند وقتی که دستور بالا برگرداند:

   ```
   No resources found.
   ```

4. ConfigMap، سرویس‌ها و PersistentVolumeClaim ها را حذف کنید.

   ```shell
   kubectl delete configmap,service,pvc -l app=mysql
   ```

5. اگر PersistentVolume ها را به صورت دستی ایجاد کرده‌اید، باید آنها را به صورت دستی حذف کنید و منابع زیرین را آزاد کنید.
   اگر از یک پروویژنر پویا استفاده کرده‌اید، آن به طور خودکار PersistentVolume ها را زمانی که ببیند شما PersistentVolumeClaim ها را حذف کرده‌اید، حذف می‌کند.
   برخی از پروویژنرهای پویا (مانند آنهایی که برای EBS و PD هستند) همچنین منابع زیرین را پس از حذف PersistentVolume ها آزاد می‌کنند.

## {{% heading "whatsnext" %}}

- درباره [مقیاس‌بندی یک StatefulSet](/docs/tasks/run-application/scale-stateful-set/) بیشتر بیاموزید.
- درباره [عیب‌یابی یک StatefulSet](/docs/tasks/debug/debug-application/debug-statefulset/) بیشتر بیاموزید.
- درباره [حذف یک StatefulSet](/docs/tasks/run-application/delete-stateful-set/) بیشتر بیاموزید.
- درباره [حذف اجباری پادهای StatefulSet](/docs/tasks/run-application/force-delete-stateful-set-pod/) بیشتر بیاموزید.
- در مخزن [Helm Charts](https://artifacthub.io/) به دنبال مثال‌های دیگر برنامه‌های stateful بگردید.
