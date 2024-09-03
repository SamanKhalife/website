---
title: پردازش موازی دقیق با استفاده از صف کار
content_type: task
weight: 30
---

<!-- overview -->

در این مثال، شما یک Job در Kubernetes اجرا خواهید کرد که چندین وظیفه موازی را به عنوان فرآیندهای کارگر اجرا می‌کند، هر یک به عنوان یک پاد جداگانه اجرا می‌شود.

در این مثال، همانطور که هر پاد ایجاد می‌شود، یک واحد کار از یک صف وظیفه انتخاب می‌کند، آن را پردازش می‌کند و تا زمانی که صف به پایان برسد، تکرار می‌شود.

در اینجا یک مرور کلی از مراحل در این مثال آمده است:

1. **یک سرویس ذخیره‌سازی برای نگهداری صف کار راه‌اندازی کنید.**  در این مثال، شما از Redis برای ذخیره
   آیتم‌های کاری استفاده خواهید کرد. در [مثال قبلی](/docs/tasks/job/coarse-parallel-processing-work-queue)، از RabbitMQ استفاده کردید. در این مثال، از Redis و یک کتابخانه مشتری سفارشی صف کار استفاده خواهید کرد؛
   زیرا AMQP روش خوبی برای مشتریان فراهم نمی‌کند تا تشخیص دهند که یک صف کار با طول محدود خالی شده است. در عمل، شما یک ذخیره مانند Redis را یک بار راه‌اندازی می‌کنید و آن را برای صف‌های کار بسیاری از Jobها و چیزهای دیگر استفاده مجدد می‌کنید.
1. **یک صف ایجاد کرده و آن را با پیام‌ها پر کنید.**  هر پیام نمایانگر یک وظیفه‌ای است که باید انجام شود. در
   این مثال، یک پیام یک عدد صحیح است که ما یک محاسبه طولانی روی آن انجام خواهیم داد.
1. **یک Job راه‌اندازی کنید که بر روی وظایف از صف کار کند.**  این Job چندین پاد را راه‌اندازی می‌کند. هر پاد یک وظیفه از صف پیام‌ها می‌گیرد، آن را پردازش می‌کند و تا زمانی که صف به پایان برسد، تکرار می‌شود.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}}

شما نیاز به یک رجیستری تصویر کانتینر دارید که بتوانید تصاویر را برای اجرا در کلاستر خود آپلود کنید.
این مثال از [Docker Hub](https://hub.docker.com/) استفاده می‌کند، اما می‌توانید آن را برای یک رجیستری تصویر کانتینر متفاوت تطبیق دهید.

این مثال وظیفه همچنین فرض می‌کند که شما Docker را به صورت محلی نصب کرده‌اید. از Docker برای
ساخت تصاویر کانتینر استفاده می‌کنید.

<!-- steps -->

با استفاده پایه‌ای، غیر موازی، از [Job](/docs/concepts/workloads/controllers/job/) آشنا باشید.

<!-- steps -->

## راه‌اندازی Redis

برای این مثال، به خاطر سادگی، شما یک نمونه از Redis را راه‌اندازی خواهید کرد.
به [مثال Redis](https://github.com/kubernetes/examples/tree/master/guestbook) برای مثالی از استقرار Redis به صورت مقیاس‌پذیر و تکراری مراجعه کنید.

شما می‌توانید فایل‌های زیر را مستقیماً دانلود کنید:

- [`redis-pod.yaml`](/examples/application/job/redis/redis-pod.yaml)
- [`redis-service.yaml`](/examples/application/job/redis/redis-service.yaml)
- [`Dockerfile`](/examples/application/job/redis/Dockerfile)
- [`job.yaml`](/examples/application/job/redis/job.yaml)
- [`rediswq.py`](/examples/application/job/redis/rediswq.py)
- [`worker.py`](/examples/application/job/redis/worker.py)

برای راه‌اندازی یک نمونه از Redis، نیاز دارید که پاد redis و سرویس redis را ایجاد کنید:

```shell
kubectl apply -f https://k8s.io/examples/application/job/redis/redis-pod.yaml
kubectl apply -f https://k8s.io/examples/application/job/redis/redis-service.yaml
```

## پر کردن صف با وظایف

حال بیایید صف را با برخی "وظایف" پر کنیم. در این مثال، وظایف رشته‌هایی هستند که باید چاپ شوند.

یک پاد تعاملی موقت برای اجرای Redis CLI راه‌اندازی کنید.

```shell
kubectl run -i --tty temp --image redis --command "/bin/sh"
```
```
Waiting for pod default/redis2-c7h78 to be running, status is Pending, pod ready: false
Hit enter for command prompt
```

اکنون Enter را بزنید، Redis CLI را شروع کنید و یک لیست با برخی آیتم‌های کاری ایجاد کنید.

```shell
redis-cli -h redis
```
```console
redis:6379> rpush job2 "apple"
(integer) 1
redis:6379> rpush job2 "banana"
(integer) 2
redis:6379> rpush job2 "cherry"
(integer) 3
redis:6379> rpush job2 "date"
(integer) 4
redis:6379> rpush job2 "fig"
(integer) 5
redis:6379> rpush job2 "grape"
(integer) 6
redis:6379> rpush job2 "lemon"
(integer) 7
redis:6379> rpush job2 "melon"
(integer) 8
redis:6379> rpush job2 "orange"
(integer) 9
redis:6379> lrange job2 0 -1
1) "apple"
2) "banana"
3) "cherry"
4) "date"
5) "fig"
6) "grape"
7) "lemon"
8) "melon"
9) "orange"
```

بنابراین، لیست با کلید `job2` صف کار خواهد بود.

توجه: اگر Kube DNS به درستی تنظیم نشده باشد، ممکن است نیاز داشته باشید که
اولین مرحله از بلوک بالا را به `redis-cli -h $REDIS_SERVICE_HOST` تغییر دهید.

## ایجاد یک تصویر کانتینر {#create-an-image}

اکنون شما آماده هستید تا یک تصویر ایجاد کنید که کارها را در آن صف پردازش کند.

شما از یک برنامه کارگر پایتون با یک مشتری Redis برای خواندن
پیام‌ها از صف پیام استفاده خواهید کرد.

یک کتابخانه ساده مشتری صف کار Redis فراهم شده است،
به نام `rediswq.py` ([دانلود](/examples/application/job/redis/rediswq.py)).

برنامه "کارگر" در هر پاد از Job از کتابخانه مشتری صف کار برای دریافت کار استفاده می‌کند. در اینجا آن را مشاهده می‌کنید:

{{% code_sample language="python" file="application/job/redis/worker.py" %}}

همچنین می‌توانید فایل‌های [`worker.py`](/examples/application/job/redis/worker.py)،
[`rediswq.py`](/examples/application/job/redis/rediswq.py)، و
[`Dockerfile`](/examples/application/job/redis/Dockerfile) را دانلود کنید و سپس
تصویر کانتینر را بسازید. در اینجا مثالی از استفاده از Docker برای ساخت تصویر آمده است:

```shell
docker build -t job-wq-2 .
```

### آپلود تصویر

برای [Docker Hub](https://hub.docker.com/)، تصویر برنامه خود را با
نام کاربری خود برچسب‌گذاری کنید و با دستورات زیر به Hub ارسال کنید. نام کاربری `<username>` را با نام کاربری Hub خود جایگزین کنید.

```shell
docker tag job-wq-2 <username>/job-wq-2
docker push <username>/job-wq-2
```

شما باید به یک مخزن عمومی ارسال کنید یا [کلاستر خود را پیکربندی کنید تا بتواند به
مخزن خصوصی شما دسترسی پیدا کند](/docs/concepts/containers/images/).

## تعریف یک Job

در اینجا یک مانفیست برای Job که ایجاد خواهید کرد وجود دارد:

{{% code_sample file="application/job/redis/job.yaml" %}}

{{< note >}}
مطمئن شوید که مانفیست را ویرایش کنید تا
`gcr.io/myproject` را به مسیر خودتان تغییر دهید.
{{< /note >}}

در این مثال، هر پاد بر روی چندین آیتم از صف کار می‌کند و سپس زمانی که هیچ آیتمی باقی نمانده است، خاتمه می‌یابد.
از آنجایی که خود کارگران تشخیص می‌دهند که صف کار خالی شده است، و کنترل‌کننده Job از صف کار خبر ندارد،
به کارگران تکیه می‌کند تا زمانی که کارشان تمام شد، سیگنال دهند.
کارگران با خاتمه دادن با موفقیت سیگنال می‌دهند که صف خالی است. بنابراین، به محض اینکه **هر** کارگری
با موفقیت خاتمه دهد، کنترل‌کننده می‌داند که کار تمام شده است و پادها به زودی خاتمه خواهند یافت.
بنابراین، باید شمارش تکمیل Job را تنظیم نکنید. کنترل‌کننده Job منتظر خواهد ماند تا
پادهای دیگر نیز تکمیل شوند.

## اجرای Job

بنابراین، حالا Job را اجرا کنید:

```shell
# فرض بر این است که مانفیست را دانلود کرده و سپس ویرایش کرده‌اید
kubectl apply -f ./job.yaml
```

اکنون کمی صبر کنید و سپس وضعیت Job را بررسی کنید:

```shell
kubectl describe jobs/job-wq-2
```
```
Name:             job-wq-2
Namespace:        default
Selector:         controller-uid=b1c7e4e3-92e1-11e7-b85e-fa163ee3c11f
Labels:           controller-uid=b1c7e4e3-92e1-11e7-b85e-fa163ee3c11f
                  job-name=job-wq-2
Annotations:      <none>
Parallelism:      2
Completions:      <unset>
Start Time:       Mon, 11 Jan 2022 17:07:59 +0000
Pods Statuses:    1 Running / 0 Succeeded / 0 Failed
Pod Template:
  Labels:       controller-uid=b1c7e4e3-92e1-11e7-b85e-fa163ee3c11f
                job-name=job-wq-2
  Containers:
   c:
    Image:              container-registry.example/exampleproject/job-wq-2
    Port:
    Environment:        <none>
    Mounts:             <none>
  Volumes:              <none>
Events:
  FirstSeen    LastSeen    Count    From            SubobjectPath    Type        Reason            Message
  ---------    --------    -----    ----            -------------    --------    ------            -------
  33s          33s         1        {job-controller }                Normal      SuccessfulCreate  Created pod: job-wq-2-lglf8
```

می‌توانید منتظر بمانید تا Job با موفقیت به پایان برسد، با یک محدودیت زمانی:
```shell
# بررسی وضعیت شرط نام به حروف حساس نیست
kubectl wait --for=condition=complete --timeout=300s job/job-wq-2
```

```shell
kubectl logs pods/job-wq-2-7r7b2
```
```
Worker with sessionID: bbd72d0a-9e5c-4dd6-abf6-416cc267991f
Initial queue state: empty=False
Working on banana
Working on date
Working on lemon
```

همانطور که می‌بینید، یکی از پادهای این Job بر روی چندین واحد کاری کار کرده است.

<!-- discussion -->

## جایگزین‌ها

اگر اجرای یک سرویس صف یا تغییر دادن کانتینرهای خود برای استفاده از صف کار برای شما ناخوشایند است، می‌توانید
یکی از الگوهای دیگر [job patterns](/docs/concepts/workloads/controllers/job/#job-patterns) را در نظر بگیرید.

اگر یک جریان پیوسته از کارهای پردازش پس‌زمینه‌ای دارید که باید اجرا شوند، در این صورت
اجرای کارگران پس‌زمینه‌ای خود با یک ReplicaSet و در نظر گرفتن اجرای یک کتابخانه پردازش پس‌زمینه‌ای مانند
[https://github.com/resque/resque](https://github.com/resque/resque) را در نظر بگیرید.
