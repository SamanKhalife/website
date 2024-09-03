---
title: Job ایندکس شده برای پردازش موازی با تخصیص کار استاتیک
content_type: task
min-kubernetes-server-version: v1.21
weight: 30
---

{{< feature-state for_k8s_version="v1.24" state="stable" >}}

<!-- overview -->

در این مثال، شما یک Job در Kubernetes اجرا خواهید کرد که از چندین فرآیند کارگر موازی استفاده می‌کند.
هر کارگر یک کانتینر متفاوت است که در پاد خود اجرا می‌شود. پادها یک _شماره ایندکس_ دارند که به طور خودکار توسط کنترل پلین تنظیم می‌شود، که به هر پاد اجازه می‌دهد تا بخش خود از کار کلی را شناسایی کند.

ایندکس پاد در
{{< glossary_tooltip text="annotation" term_id="annotation" >}}
`batch.kubernetes.io/job-completion-index` به صورت یک رشته که مقدار دهدهی آن را نشان می‌دهد در دسترس است. برای اینکه فرآیند وظیفه کانتینری شده بتواند این ایندکس را به دست آورد، می‌توانید مقدار annotation را با استفاده از مکانیزم [downward API](/docs/concepts/workloads/pods/downward-api/) منتشر کنید.
برای راحتی، کنترل پلین به طور خودکار downward API را تنظیم می‌کند تا ایندکس را در متغیر محیطی `JOB_COMPLETION_INDEX` قرار دهد.

در اینجا یک مرور کلی از مراحل در این مثال آمده است:

1. **تعریف یک manifest Job با استفاده از تکمیل ایندکس شده**.
   downward API به شما اجازه می‌دهد تا annotation ایندکس پاد را به عنوان یک
   متغیر محیطی یا فایل به کانتینر منتقل کنید.
2. **شروع یک Job `Indexed` بر اساس آن manifest**.

## {{% heading "prerequisites" %}}

شما باید با استفاده پایه‌ای، غیر موازی، از [Job](/docs/concepts/workloads/controllers/job/) آشنا باشید.

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!-- steps -->

## انتخاب یک روش

برای دسترسی به مورد کاری از برنامه کارگر، چند گزینه دارید:

1. خواندن متغیر محیطی `JOB_COMPLETION_INDEX`.
   {{< glossary_tooltip text="controller" term_id="controller" >}}
   به طور خودکار این متغیر را به annotation حاوی ایندکس تکمیل پیوند می‌دهد.
2. خواندن یک فایل که حاوی ایندکس تکمیل است.
3. فرض کنید که نمی‌توانید برنامه را تغییر دهید، می‌توانید آن را با یک اسکریپت که ایندکس را با استفاده از هر یک از روش‌های فوق می‌خواند و آن را به چیزی که برنامه می‌تواند به عنوان ورودی استفاده کند، تبدیل کنید.

برای این مثال، فرض کنید که گزینه ۳ را انتخاب کرده‌اید و می‌خواهید ابزار [rev](https://man7.org/linux/man-pages/man1/rev.1.html) را اجرا کنید. این برنامه یک فایل را به عنوان آرگومان می‌پذیرد و محتوای آن را معکوس چاپ می‌کند.

```shell
rev data.txt
```

شما از ابزار `rev` از
تصویر کانتینری [`busybox`](https://hub.docker.com/_/busybox) استفاده خواهید کرد.

از آنجا که این تنها یک مثال است، هر پاد تنها یک کار کوچک انجام می‌دهد (معکوس کردن یک رشته کوتاه). در یک بار کاری واقعی، ممکن است به عنوان مثال، یک Job ایجاد کنید که نشان‌دهنده
وظیفه تولید 60 ثانیه ویدئو بر اساس داده‌های صحنه باشد.
هر مورد کاری در Job رندر ویدئو، رندر یک فریم خاص از آن کلیپ ویدئویی خواهد بود. تکمیل ایندکس شده به این معناست که هر پاد در Job می‌داند که کدام فریم را رندر و منتشر کند، با شمارش فریم‌ها از ابتدای کلیپ.

## تعریف یک Job ایندکس شده

در اینجا یک نمونه manifest Job که از حالت تکمیل `Indexed` استفاده می‌کند آورده شده است:

{{% code_sample language="yaml" file="application/job/indexed-job.yaml" %}}

در مثال بالا، شما از متغیر محیطی `JOB_COMPLETION_INDEX` داخلی که توسط کنترل‌کننده Job برای همه کانتینرها تنظیم شده است، استفاده می‌کنید. یک [کانتینر اولیه](/docs/concepts/workloads/pods/init-containers/)
ایندکس را به یک مقدار استاتیک نگاشت می‌کند و آن را به یک فایل می‌نویسد که از طریق یک [حجم emptyDir](/docs/concepts/storage/volumes/#emptydir) با کانتینر اجرایی کارگر به اشتراک گذاشته می‌شود.
به طور اختیاری، می‌توانید [متغیر محیطی خود را از طریق downward API تعریف کنید](/docs/tasks/inject-data-application/environment-variable-expose-pod-information/)
تا ایندکس را به کانتینرها منتشر کنید. همچنین می‌توانید یک لیست از مقادیر را از یک [ConfigMap به عنوان متغیر محیطی یا فایل](/docs/tasks/configure-pod-container/configure-pod-configmap/) بارگذاری کنید.

به طور متناوب، می‌توانید به طور مستقیم [از downward API برای ارسال مقدار annotation به عنوان یک فایل حجم](/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/#store-pod-fields) استفاده کنید،
مانند مثال زیر:

{{% code_sample language="yaml" file="application/job/indexed-job-vol.yaml" %}}

## اجرای Job

اکنون Job را اجرا کنید:

```shell
# این از روش اول استفاده می‌کند (وابسته به $JOB_COMPLETION_INDEX)
kubectl apply -f https://kubernetes.io/examples/application/job/indexed-job.yaml
```

وقتی این Job را ایجاد می‌کنید، کنترل پلین یک سری پاد ایجاد می‌کند، یکی برای هر ایندکسی که مشخص کرده‌اید. مقدار `.spec.parallelism` تعیین می‌کند که چند پاد می‌توانند همزمان اجرا شوند در حالی که `.spec.completions` تعیین می‌کند که چند پاد در کل Job ایجاد می‌شود.

از آنجا که `.spec.parallelism` کمتر از `.spec.completions` است، کنترل پلین منتظر تکمیل برخی از پادهای اول می‌ماند قبل از اینکه پادهای بیشتری را شروع کند.

می‌توانید منتظر بمانید تا Job موفق شود، با یک تایم‌اوت:
```shell
# بررسی شرط نام حساس به حروف بزرگ و کوچک نیست
kubectl wait --for=condition=complete --timeout=300s job/indexed-job
```

حالا، Job را توصیف کنید و بررسی کنید که موفق بوده است.


```shell
kubectl describe jobs/indexed-job
```

خروجی مشابه زیر است:

```
Name:              indexed-job
Namespace:         default
Selector:          controller-uid=bf865e04-0b67-483b-9a90-74cfc4c3e756
Labels:            controller-uid=bf865e04-0b67-483b-9a90-74cfc4c3e756
                   job-name=indexed-job
Annotations:       <none>
Parallelism:       3
Completions:       5
Start Time:        Thu, 11 Mar 2021 15:47:34 +0000
Pods Statuses:     2 Running / 3 Succeeded / 0 Failed
Completed Indexes: 0-2
Pod Template:
  Labels:  controller-uid=bf865e04-0b67-483b-9a90-74cfc4c3e756
           job-name=indexed-job
  Init Containers:
   input:
    Image:      docker.io/library/bash
    Port:       <none>
    Host Port:  <none>
    Command:
      bash
      -c
      items=(foo bar baz qux xyz)
      echo ${items[$JOB_COMPLETION_INDEX]} > /input/data.txt

    Environment:  <none>
    Mounts:
      /input from input (rw)
  Containers:
   worker:
    Image:      docker.io/library/busybox
    Port:       <none>
    Host Port:  <none>
    Command:
      rev
      /input/data.txt
    Environment:  <none>
    Mounts:
      /input from input (rw)
  Volumes:
   input:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  4s    job-controller  Created pod: indexed-job-njkjj
  Normal  SuccessfulCreate  4s    job-controller  Created pod: indexed-job-9kd4h
  Normal  SuccessfulCreate  4s    job-controller  Created pod: indexed-job-qjwsz
  Normal  SuccessfulCreate  1s    job-controller  Created pod: indexed-job-fdhq5
  Normal  SuccessfulCreate  1s    job-controller  Created pod: indexed-job-ncslj
```

در این مثال، شما Job را با مقادیر سفارشی برای هر ایندکس اجرا می‌کنید. شما می‌توانید خروجی یکی از پادها را بررسی کنید:

```

shell
kubectl logs indexed-job-fdhq5 # این را تغییر دهید تا با نام یک پاد از آن Job مطابقت داشته باشد
```

خروجی مشابه زیر است:

```
xuq
```
