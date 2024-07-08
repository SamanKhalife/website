---
title: پردازش موازی با استفاده از گسترش‌ها
content_type: task
min-kubernetes-server-version: v1.8
weight: 50
---

<!-- overview -->

این وظیفه اجرای چندین {{< glossary_tooltip text="Jobs" term_id="job" >}} را بر اساس یک قالب مشترک نشان می‌دهد. شما می‌توانید از این روش برای پردازش دسته‌ای از کارها به صورت موازی استفاده کنید.

برای این مثال، تنها سه آیتم وجود دارد: _سیب_، _موز_ و _گیلاس_.
Jobهای نمونه هر آیتم را با چاپ یک رشته و سپس توقف پردازش می‌کنند.

برای آشنایی با نحوه استفاده از این الگو در موارد استفاده واقعی، به [استفاده از Jobs در بارهای کاری واقعی](#using-jobs-in-real-workloads) مراجعه کنید.

## {{% heading "prerequisites" %}}

باید با استفاده پایه‌ای و غیر موازی از [Job](/docs/concepts/workloads/controllers/job/) آشنا باشید.

{{< include "task-tutorial-prereqs.md" >}}

برای استفاده از قالب‌بندی پایه‌ای، به ابزار خط فرمان `sed` نیاز دارید.

برای دنبال کردن مثال پیشرفته قالب‌بندی، به یک نصب کاری از [Python](https://www.python.org/) و کتابخانه قالب‌بندی Jinja2 برای پایتون نیاز دارید.

پس از نصب پایتون، می‌توانید Jinja2 را با اجرای دستور زیر نصب کنید:

```shell
pip install --user jinja2
```

<!-- steps -->

## ایجاد Jobs بر اساس یک قالب

ابتدا، قالب زیر از یک Job را به یک فایل به نام `job-tmpl.yaml` دانلود کنید.
در اینجا چیزی که دانلود می‌کنید آورده شده است:

{{% code_sample file="application/job/job-tmpl.yaml" %}}

```shell
# استفاده از curl برای دانلود job-tmpl.yaml
curl -L -s -O https://k8s.io/examples/application/job/job-tmpl.yaml
```

فایلی که دانلود کرده‌اید هنوز یک {{< glossary_tooltip text="manifest" term_id="manifest" >}} معتبر برای Kubernetes نیست. در عوض، این قالب یک نمایش YAML از یک شی Job با برخی از جای‌گذاری‌هایی است که باید پر شوند تا قابل استفاده باشد. نحو `$ITEM` برای Kubernetes معنایی ندارد.

### ایجاد مانیفست‌ها از قالب

قطعه شل زیر از `sed` برای جایگزینی رشته `$ITEM` با متغیر حلقه استفاده می‌کند و در یک دایرکتوری موقتی به نام `jobs` می‌نویسد. اکنون این را اجرا کنید:

```shell
# گسترش قالب به چند فایل، یکی برای هر آیتم برای پردازش.
mkdir ./jobs
for i in apple banana cherry
do
  cat job-tmpl.yaml | sed "s/\$ITEM/$i/" > ./jobs/job-$i.yaml
done
```

بررسی کنید که آیا کار کرده است:

```shell
ls jobs/
```

خروجی مشابه زیر خواهد بود:

```
job-apple.yaml
job-banana.yaml
job-cherry.yaml
```

شما می‌توانید از هر نوع زبان قالب‌بندی (به عنوان مثال: Jinja2؛ ERB) استفاده کنید یا یک برنامه برای تولید مانیفست‌های Job بنویسید.

### ایجاد Jobs از مانیفست‌ها

در مرحله بعد، همه Jobs را با یک دستور kubectl ایجاد کنید:

```shell
kubectl create -f ./jobs
```

خروجی مشابه زیر خواهد بود:

```
job.batch/process-item-apple created
job.batch/process-item-banana created
job.batch/process-item-cherry created
```

حالا، Jobs را بررسی کنید:

```shell
kubectl get jobs -l jobgroup=jobexample
```

خروجی مشابه زیر خواهد بود:

```
NAME                  COMPLETIONS   DURATION   AGE
process-item-apple    1/1           14s        22s
process-item-banana   1/1           12s        21s
process-item-cherry   1/1           12s        20s
```

استفاده از گزینه `-l` در kubectl فقط Jobs‌هایی را انتخاب می‌کند که بخشی از این گروه هستند (ممکن است Jobs‌های دیگری در سیستم وجود داشته باشند که مرتبط نباشند).

همچنین می‌توانید با استفاده از همان {{< glossary_tooltip text="label selector" term_id="selector" >}} پادها را نیز بررسی کنید:

```shell
kubectl get pods -l jobgroup=jobexample
```

خروجی مشابه زیر خواهد بود:

```
NAME                        READY     STATUS      RESTARTS   AGE
process-item-apple-kixwv    0/1       Completed   0          4m
process-item-banana-wrsf7   0/1       Completed   0          4m
process-item-cherry-dnfu9   0/1       Completed   0          4m
```

می‌توانیم از این دستور واحد برای بررسی خروجی همه Jobs به یکباره استفاده کنیم:

```shell
kubectl logs -f -l jobgroup=jobexample
```

خروجی باید به این شکل باشد:

```
Processing item apple
Processing item banana
Processing item cherry
```

### پاکسازی {#cleanup-1}

```shell
# حذف Jobs‌هایی که ایجاد کرده‌اید
# کلاستر شما به صورت خودکار Pods آن‌ها را پاکسازی می‌کند
kubectl delete job -l jobgroup=jobexample
```

## استفاده از پارامترهای پیشرفته قالب

در [مثال اول](#create-jobs-based-on-a-template)، هر نمونه از قالب یک پارامتر داشت و آن پارامتر نیز در نام Job استفاده شد. با این حال، [نام‌ها](/docs/concepts/overview/working-with-objects/names/#names) به طور محدود تنها می‌توانند شامل کاراکترهای خاصی باشند.

این مثال کمی پیچیده‌تر از زبان قالب‌بندی [Jinja](https://palletsprojects.com/p/jinja/) برای تولید مانیفست‌ها و سپس اشیاء از آن مانیفست‌ها با چندین پارامتر برای هر Job استفاده می‌کند.

برای این قسمت از وظیفه، شما از یک اسکریپت پایتون تک‌خطی برای تبدیل قالب به مجموعه‌ای از مانیفست‌ها استفاده خواهید کرد.

ابتدا، قالب زیر از یک شی Job را به یک فایل به نام `job.yaml.jinja2` کپی و جای‌گذاری کنید:

```liquid
{% set params = [{ "name": "apple", "url": "http://dbpedia.org/resource/Apple", },
                  { "name": "banana", "url": "http://dbpedia.org/resource/Banana", },
                  { "name": "cherry", "url": "http://dbpedia.org/resource/Cherry" }]
%}
{% for p in params %}
{% set name = p["name"] %}
{% set url = p["url"] %}
---
apiVersion: batch/v1
kind: Job
metadata:
  name: jobexample-{{ name }}
  labels:
    jobgroup: jobexample
spec:
  template:
    metadata:
      name: jobexample
      labels:
        jobgroup: jobexample
    spec:
      containers:
      - name: c
        image: busybox:1.28
        command: ["sh", "-c", "echo Processing URL {{ url }} && sleep 5"]
      restartPolicy: Never
{% endfor %}
```

قالب بالا دو پارامتر برای هر شی Job با استفاده از لیستی از دیکشنری‌های پایتون تعریف می‌کند (خطوط 1-4). یک حلقه `for` یک مانیفست Job برای هر مجموعه از پارامترها تولید می‌کند (باقی خطوط).

این مثال به ویژگی‌ای از YAML تکیه دارد. یک فایل YAML می‌تواند شامل چندین سند (در اینجا، مانیفست‌های Kubernetes) باشد که با `---` در یک خط جداگانه جدا شده‌اند. شما می‌توانید خروجی را مستقیماً به `kubectl` ارسال کنید تا Jobs را ایجاد کنید.

در مرحله بعد، از این برنامه تک‌خطی پایتون برای گسترش قالب استفاده کنید:

```shell
alias render_template='python -c "from jinja2 import Template; import sys; print(Template(sys.stdin.read()).render());"'
```

از `render_template` برای تبدیل پارامترها و قالب به یک فایل YAML حاوی مانیفست‌های Kubernetes استفاده کنید:

```shell
# این نیاز به alias‌ای که قبلاً تعریف کرده‌اید دارد
cat job.yaml.jinja2 | render_template > jobs.yaml
```

می‌توانید `jobs.yaml` را مشاهده کنید تا مطمئن شوید که اسکریپت `render_template` به درستی کار کرده است.

زمانی که مطمئن شدید که `render_template` به درستی کار می‌کند، می‌توانید خروجی آن را به `kubectl` ارسال کنید:

```shell
cat job.yaml.jinja2 | render_template | kubectl apply -f -
```

Kubernetes Jobsهایی را که ایجاد کرده‌اید قبول و اجرا می‌کند.

### پاکسازی {#cleanup-2}

```shell
# حذف Jobs‌هایی که ایجاد کرده‌اید
# کلاستر شما به صورت خودکار Pods آن‌ها را پاکسازی می‌کند
kubectl delete job -l jobgroup=jobexample
```

<!-- discussion -->

## استفاده از Jobs در بارهای کاری واقعی

در یک مورد استفاده واقعی، هر Job محاسبات قابل توجهی انجام می‌دهد، مانند رندر کردن یک فریم از یک فیلم، یا پردازش یک محدوده از سطرهای یک پایگاه داده. اگر در حال رندر کردن یک فیلم بودید، می‌توانستید `$ITEM` را به شماره فریم تنظیم کنید. اگر در حال پردازش سطرهای یک جدول پایگاه داده بودید، می‌توانستید `$ITEM` را برای نمایش محدوده سطرهای پایگاه داده که باید پردازش شوند تنظیم کنید.

در وظیفه، شما یک دستور برای جمع‌آوری خروجی از Pods با گرفتن لاگ‌های آن‌ها اجرا کردید. در یک مورد استفاده واقعی، هر Pod برای یک Job خروجی خود را به حافظه دائمی می‌نویسد قبل از اینکه کامل شود. می‌توانید برای هر Job از یک PersistentVolume استفاده کنید یا از یک سرویس حافظه خارجی استفاده کنید. به عنوان مثال، اگر در حال رندر کردن فریم‌های یک فیلم هستید، از HTTP برای `PUT` داده‌های فریم رندر شده به یک URL استفاده کنید، با استفاده از یک URL متفاوت برای هر فریم.

## برچسب‌ها روی Jobs و Pods

بعد از ایجاد یک Job، Kubernetes به صورت خودکار برچسب‌های اضافی اضافه می‌کند که Pods یک Job را از Pods یک Job دیگر متمایز می‌کند.

در این مثال، هر Job و قالب Pod آن یک برچسب دارد: `jobgroup=jobexample`.

Kubernetes خودش به برچسب‌های نام‌گذاری شده `jobgroup` توجهی نمی‌کند. تنظیم یک برچسب برای همه Jobs‌هایی که از یک قالب ایجاد می‌کنید، انجام عملیات روی همه آن Jobs‌ها را به صورت یکجا آسان می‌کند. در [مثال اول](#create-jobs-based-on-a-template) از یک قالب برای ایجاد چندین Job استفاده کردید. قالب تضمین می‌کند که هر Pod نیز همان برچسب را دریافت می‌کند، بنابراین می‌توانید با یک دستور واحد، همه Pods برای این Jobs‌های قالب‌بندی شده را بررسی کنید.

{{< note >}}
کلید برچسب `jobgroup` ویژه یا رزرو شده نیست.
می‌توانید طرح برچسب‌گذاری خود را انتخاب کنید.
برچسب‌های [توصیه شده](/docs/concepts/overview/working-with-objects/common-labels/#labels) وجود دارد که اگر بخواهید می‌توانید از آن‌ها استفاده کنید.
{{< /note >}}

## جایگزین‌ها

اگر قصد دارید تعداد زیادی از اشیاء Job ایجاد کنید، ممکن است متوجه شوید که:

- حتی با استفاده از برچسب‌ها، مدیریت تعداد زیادی Jobs مشکل است.
- اگر بسیاری از Jobs را در یک دسته ایجاد کنید، ممکن است بار زیادی بر روی صفحه کنترل Kubernetes وارد کنید. به طور متناوب، سرور API Kubernetes می‌تواند شما را محدود کند و درخواست‌های شما را به طور موقت با وضعیت 429 رد کند.
- شما توسط یک {{< glossary_tooltip text="resource quota" term_id="resource-quota" >}} بر روی Jobs محدود شده‌اید: سرور API برخی از درخواست‌های شما را زمانی که مقدار زیادی کار در یک دسته ایجاد می‌کنید به طور دائمی رد می‌کند.

الگوهای [Job دیگری](/docs/concepts/workloads/controllers/job/#job-patterns) وجود دارد که می‌توانید برای پردازش حجم زیادی از کارها بدون ایجاد تعداد زیادی از اشیاء Job استفاده کنید.

همچنین می‌توانید نوشتن [کنترلر](/docs/concepts/architecture/controller/) خود را برای مدیریت خودکار اشیاء Job در نظر بگیرید.
