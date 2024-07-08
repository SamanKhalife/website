---
title: پردازش موازی خشن با استفاده از صف کار
content_type: task
weight: 20
---

<!-- overview -->

در این مثال، شما یک کار Job Kubernetes را با فرآیندهای کارگر موازی اجرا خواهید کرد.

در این مثال، هر زمان که یک Pod ایجاد می‌شود، یک واحد کار از یک صف وظایف را برمی‌دارد، آن را کامل می‌کند، آن را از صف حذف می‌کند و خاتمه می‌یابد.

اینجا یک مرور اجمالی از مراحل در این مثال آمده است:

1. **راه‌اندازی خدمت صف پیام.** در این مثال، از RabbitMQ استفاده می‌کنید، اما می‌توانید از سرویس دیگری هم استفاده کنید. در عمل، شما یک خدمت صف پیام را یک بار راه‌اندازی کرده و برای بسیاری از کارها استفاده می‌کنید.
2. **ایجاد یک صف و پر کردن آن با پیام‌ها.** هر پیام یک وظیفه را نمایان می‌کند که باید انجام شود. در این مثال، یک پیام یک عدد صحیح است که بر روی آن محاسبات طولانی انجام خواهیم داد.
3. **راه‌اندازی یک Job که بر روی وظایف از صف کار می‌کند.** Job چندین Pod را شروع می‌کند. هر پاد یک وظیفه را از صف پیام برمی‌دارد، آن را پردازش می‌کند، و خاتمه می‌یابد.

## {{% heading "prerequisites" %}}

شما باید از استفاده اولیه غیر موازی از [Job](/docs/concepts/workloads/controllers/job/) آشنا باشید.

{{< include "task-tutorial-prereqs.md" >}}

شما نیاز دارید به یک ثبت تصاویر کانتینر که می‌توانید تصاویر را برای اجرای در خوشه خود آپلود کنید.

این مثال وظیفه همچنین فرض می‌کند که Docker به صورت محلی نصب شده است.


<!-- steps -->

## راه‌اندازی خدمت صف پیام

در این مثال از RabbitMQ استفاده می‌شود، با این حال، شما می‌توانید مثال را به استفاده از سرویس دیگری از نوع AMQP تطبیق دهید.

در عمل، شما می‌توانید یک خدمت صف پیام را یک بار در یک خوشه راه‌اندازی کرده و برای بسیاری از کارها استفاده کنید، همچنین برای خدمات در حال اجرا به کار می‌برید.

RabbitMQ را به این شکل راه‌اندازی کنید:

```shell
# یک خدمت برای استفاده از StatefulSet بسازید
kubectl create -f https://kubernetes.io/examples/application/job/rabbitmq/rabbitmq-service.yaml
```
```
service "rabbitmq-service" created
```

```shell
kubectl create -f https://kubernetes.io/examples/application/job/rabbitmq/rabbitmq-statefulset.yaml
```
```
statefulset "rabbitmq" created
```

## تست خدمت صف پیام

حالا می‌توانیم با دسترسی به صف پیام آزمایش کنیم. یک Pod تعاملی موقت ایجاد می‌کنیم، ابزارهایی را در آن نصب می‌کنیم و با صف‌ها آزمایش می‌کنیم.

ابتدا یک Pod تعاملی موقت ایجاد کنید.

```shell
# یک کانتینر تعاملی موقت ایجاد کنید
kubectl run -i --tty temp --image ubuntu:22.04
```
```
Waiting for pod default/temp-loe07 to be running, status is Pending, pod ready: false
... [ خط قبلی چندین بار تکرار می‌شود ... که پایان خواهد یافت ] ...
```

توجه کنید که نام Pod و خط فرمان شما ممکن است متفاوت باشد.

بعد از آن `amqp-tools` را نصب کنید تا بتوانید با صف‌های پیام کار کنید.
دستورات بعدی نشان می‌دهند که باید در داخل پوشل تعاملی آن‌ها را اجرا کنید:

```shell
apt-get update && apt-get install -y curl ca-certificates amqp-tools python3 dnsutils
```

بعداً، بررسی کنید که می‌توانید خدمت RabbitMQ را کشف کنید:

```
# دستورات زیر را در داخل Pod اجرا کنید
# توجه داشته باشید که خدمت rabbitmq-service یک نام DNS دارد که توسط Kubernetes فراهم می‌شود:
nslookup rabbitmq-service
```
```
Server:        10.0.0.10
Address:    10.0.0.10#53

Name:    rabbitmq-service.default.svc.cluster.local
Address: 10.0.147.152
```
(آدرس‌های IP ممکن است متفاوت باشند)

اگر افزونه kube-dns به درستی تنظیم نشده باشد، مرحله قبلی برای شما کار نخواهد کرد.
همچنین می‌توانید آدرس IP برای آن خدمت را در یک متغیر محیطی پیدا کنید:

```shell
# این بررسی را در داخل Pod اجرا کنید
env | grep RABBITMQ_SERVICE | grep HOST
```
```
RABBITMQ_SERVICE_SERVICE_HOST=10.0.147.152
```
(آدرس IP ممکن است متفاوت باشد)

بعداً بررسی کنید که می‌توانید یک صف ایجاد کنید، پیام‌ها را منتشر و مصرف کنید.

```shell
# دستورات زیر را در داخل Pod اجرا کنید
# در خط بعدی، rabbitmq-service نام میزبان است که خدم

ت rabbitmq-service
# برای دسترسی به آن می‌تواند بر روی آن دسترسی یافت.

/usr/bin/amqp-declare-queue --url=$BROKER_URL -q foo -d
```
```
foo
```

یک پیام را به صف منتشر کنید:
```shell
/usr/bin/amqp-publish --url=$BROKER_URL -r foo -p -b Hello

# و آن را به دست آورید.

/usr/bin/amqp-consume --url=$BROKER_URL -q foo -c 1 cat && echo 1>&2
```
```
Hello
```

در دستور آخر، ابزار `amqp-consume` یک پیام (`-c 1`) را از صف برمی‌دارد و آن پیام را به ورودی استاندارد یک دستور تعلیمی انتخابی می‌دهد.
در این مورد، برنامه `cat` کاراکترهایی را که از ورودی استاندارد خوانده شده‌اند را چاپ می‌کند و
پس از آن اکوی یک علامت تعجب برای مثال قابل خواندن است.

## پر کردن صف با وظایف

حالا صف را با چندین وظیفه شبیه‌سازی شده پر کنید. در این مثال، وظایف رشته‌هایی هستند که قرار است چاپ شوند.

در عمل، محتوای پیام‌ها ممکن است شامل:

- نام‌های فایل‌هایی که باید پردازش شوند
- پرچم‌های اضافی برای برنامه
- دامنه‌های کلیدها در یک جدول پایگاه داده
- پارامترهای پیکربندی برای یک شبیه‌سازی
- شماره فریم‌های یک صحنه برای رندر کردن

اگر داده‌های بزرگی وجود داشته باشد که به صورت فقط خواندنی توسط تمامی پادهای Job نیاز است، معمولاً آن را در یک سیستم فایل مشترک مانند NFS قرار می‌دهید و آن را به صورت readonly بر روی تمام پادها mount می‌کنید، یا برنامه را درون پاد نوشته و از یک سیستم فایل خوشه‌ای مانند HDFS به طور طبیعی داده را خوانده می‌کنید.

برای این مثال، شما صف را ایجاد کرده و با استفاده از ابزارهای خط فرمان AMQP آن را پر خواهید کرد. در عمل، ممکن است یک برنامه بنویسید تا با استفاده از کتابخانه مشتری AMQP، صف را پر کنید.

```shell
# این را بروی رایانه خود اجرا کنید، نه در Pod
/usr/bin/amqp-declare-queue --url=$BROKER_URL -q job1  -d
```
```
job1
```

افزودن موارد به صف:
```shell
for f in apple banana cherry date fig grape lemon melon
do
  /usr/bin/amqp-publish --url=$BROKER_URL -r job1 -p -b $f
done
```

شما 8 پیام به صف اضافه کردید.

## ایجاد تصویر کانتینر

حالا آماده‌اید تا یک تصویر ایجاد کنید که به عنوان یک Job اجرا خواهید کرد.

این Job از ابزار `amqp-consume` برای خواندن پیام از صف و اجرای کار واقعی استفاده خواهد کرد. اینجا یک برنامه ساده‌ای است که نمونه:

{{% code_sample language="python" file="application/job/rabbitmq/worker.py" %}}

به اسکریپت اجازه اجرا را بدهید:

```shell
chmod +x worker.py
```

حالا یک تصویر بسازید. یک پوشه موقت بسازید، به آن تغییر دهید، فایل [Dockerfile](/examples/application/job/rabbitmq/Dockerfile) را دانلود کنید، و همچنین [worker.py](/examples/application/job/rabbitmq/worker.py). سپس تصویر را با این دستور بسازید:

```shell
docker build -t job-wq-1 .
```

برای [Docker Hub](https://hub.docker.com/)، تصویر برنامه خود را با نام کاربری‌تان برچسب بزنید و به Hub بفرستید. دستورات زیر را وارد کنید. `username` را با نام کاربری Hub خود جایگزین کنید.

```shell
docker tag job-wq-1 <username>/job-wq-1
docker push <username>/job-wq-1
```

اگر از یک ثبت تصاویر کانتینر جایگزین استفاده می‌کنید، تصویر را برچسب گذاری کنید و آن را در آن‌جا پوش کنید.

## تعریف یک Job

در اینجا یک بیانیه برای یک Job است. شما باید یک کپی از منظر Job را بسازید (آن را `./job.yaml` بنامید)، و نام تصویر کانتینر را تغییر دهید تا با نامی که استفاده کرده‌اید همخوانی داشته باشد.

{{% code_sample file="application/job/rabbitmq/job.yaml" %}}

در این مثال، هر پاد بر روی یک مورد از صف کار می‌کند و سپس خاتمه می‌یابد.
بنابراین، تعداد کامل شدن Job با تعداد آیتم‌های کار مطابقت دارد. به همین دلیل، منظور مثال، `.spec.completions` را به `8` تنظیم کرده‌ایم.

## اجرای Job

حالا Job را اجرا کنید:

```shell
# این فرض را می‌کند که شما منظور را دانلود و سپس منظر را ویرایش کرده‌اید
kubectl apply -f ./job.yaml
```

می‌توانید منتظر موفقیت Job با یک زمان محدود باشید:

```shell
# بررسی شرایط نام حساس به حروف نوشته شده
kubectl wait --for=condition=complete --timeout=300s job/job-wq-1
```

بعداً بر روی Job بررسی کنید:

```shell
kubectl describe jobs/job-wq-1
```
```
Name:             job-wq-1
Namespace:        default
Selector:         controller-uid=41d75705-92df-11e7-b85e-fa163ee3c11f
Labels:           controller-uid=41d75705-92df-11e7-b85e-fa163ee3c11f
                  job-name=job-wq-1
Annotations:      <none>
Parallelism:      2
Completions:      8
Start Time:       Wed, 06 Sep 2022 16:42:02 +0000
Pods Statuses:    0 Running / 8 Succeeded / 0 Failed
Pod Template:
  Labels:       controller-uid=41d75705-92df-11e7-b85e-fa163ee3c11f
                job-name=job-wq-1
  Containers:
   c:
    Image:      container-registry.example/causal-jigsaw-637/job-wq-1
    Port:
    Environment:
      BROKER_URL:       amqp://guest:guest@rabbitmq-service:5672
      QUEUE:            job1
    Mounts:             <none>
  Volumes:              <none>
Events:
  FirstSeen  LastSeen   Count    From    SubobjectPath    Type      Reason              Message
  ─────────  ────────   ─

────    ────    ─────────────    ──────    ──────              ───────
  27s        27s        1        {job }                   Normal    SuccessfulCreate    Created pod: job-wq-1-hcobb
  27s        27s        1        {job }                   Normal    SuccessfulCreate    Created pod: job-wq-1-weytj
  27s        27s        1        {job }                   Normal    SuccessfulCreate    Created pod: job-wq-1-qaam5
  27s        27s        1        {job }                   Normal    SuccessfulCreate    Created pod: job-wq-1-b67sr
  26s        26s        1        {job }                   Normal    SuccessfulCreate    Created pod: job-wq-1-xe5hj
  15s        15s        1        {job }                   Normal    SuccessfulCreate    Created pod: job-wq-1-w2zqe
  14s        14s        1        {job }                   Normal    SuccessfulCreate    Created pod: job-wq-1-d6ppa
  14s        14s        1        {job }                   Normal    SuccessfulCreate    Created pod: job-wq-1-p17e0
```

تمام پادهای آن Job موفق بودند! شما تمام شده‌اید.

<!-- بحث -->

## جایگزین‌ها

این روش دارای این مزیت است که نیازی به اصلاح برنامه خود "کارگر" برای آگاهی از وجود یک صف کار ندارید. شما می‌توانید برنامه کارگر را بدون اصلاح در تصویر کانتینر خود اضافه کنید.

استفاده از این روش نیازمند اجرای یک سرویس صف پیام است. اگر اجرای یک سرویس صف نامناسب باشد، ممکن است به جای الگوهای [کار اصلی](/docs/concepts/workloads/controllers/job/#job-patterns) را در نظر بگیرید.

این روش یک پاد برای هر آیتم کار ایجاد می‌کند. اگر آیتم‌های کار شما فقط چند ثانیه طول بکشند، ایجاد یک پاد برای هر آیتم کار ممکن است هزینه زیادی را اضافه کند. طراحی دیگری را در نظر بگیرید، مانند در [مثال کار صف کار موازی دقیق](/docs/tasks/job/fine-parallel-processing-work-queue/)، که چندین آیتم کار را در هر پاد اجرا می‌کند.

در این مثال، از ابزار `amqp-consume` برای خواندن پیام از صف و اجرای برنامه واقعی استفاده کردید. این دارای این مزیت است که نیازی به اصلاح برنامه خود برای آگاهی از صف ندارید. [مثال کار صف کار موازی دقیق](/docs/tasks/job/fine-parallel-processing-work-queue/) نشان می‌دهد که چگونه با استفاده از یک کتابخانه مشتری با صف کار ارتباط برقرار کنید.

## هشدارها

اگر تعداد کامل شدن‌ها کمتر از تعداد موارد در صف تنظیم شود، آنگاه همه موارد پردازش نخواهند شد.

اگر تعداد کامل شدن‌ها بیشتر از تعداد موارد در صف تنظیم شود، آنگاه Job به نظر می‌رسد که کامل شده است، هرچند که همه موارد در صف پردازش شده‌اند. پادهای اضافی راه اندازی خواهند کرد که در انتظار یک پیام بلوکه می‌مانند.
شما باید مکانیزم خود را ایجاد کنید تا وقتی کار برای انجام دادن وجود دارد و اندازه صف را اندازه‌گیری کنید، تنظیمات تکمیل را برای هماهنگی با موارد مطابقت بدهید.

این الگو با احتمال کمی مسابقه روبرو می‌شود. اگر کانتینر بین زمانی که پیام توسط دستور `amqp-consume` تأیید شده است و زمانی که کانتینر با موفقیت خاتمه می‌یابد یا اگر گره پیش از آنکه kubelet توانایی پست موفقیت پاد را به API سرور داشته باشد، کشته شود، Job به نظر نخواهد رسید که کامل شده است، هرچند که همه موارد در صف پردازش شده‌اند.
