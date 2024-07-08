---
title: اجرای وظایف خودکار با CronJob
min-kubernetes-server-version: v1.21
reviewers:
- chenopis
content_type: task
weight: 10
---

<!-- مرور -->

این صفحه نحوه اجرای وظایف خودکار با استفاده از شی {{< glossary_tooltip text="CronJob" term_id="cronjob" >}} در Kubernetes را نشان می‌دهد.

## {{% heading "پیش‌نیازها" %}}

* {{< include "task-tutorial-prereqs.md" >}}

<!-- مراحل -->

## ایجاد یک CronJob {#creating-a-cron-job}

کار‌های Cron نیازمند یک فایل پیکربندی هستند.
اینجا یک منیفست برای یک CronJob که هر دقیقه یک وظیفه نمایشی ساده اجرا می‌کند، آمده است:

{{% code_sample file="application/job/cronjob.yaml" %}}

برای اجرای مثال CronJob از این دستور استفاده کنید:

```shell
kubectl create -f https://k8s.io/examples/application/job/cronjob.yaml
```

خروجی مشابه زیر خواهد بود:

```
cronjob.batch/hello created
```

پس از ایجاد cron job، وضعیت آن را با استفاده از این دستور بدست آورید:

```shell
kubectl get cronjob hello
```

خروجی مشابه زیر خواهد بود:

```
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        <none>          10s
```

همانطور که از نتایج دستور می‌بینید، cron job هنوز هیچ کاری را برنامه‌ریزی یا اجرا نکرده است.
مشاهده کنید که چگونه از یک دقیقه برنامه‌ریزی شده:

```shell
kubectl get jobs --watch
```

خروجی مشابه زیر خواهد بود:

```
NAME               COMPLETIONS   DURATION   AGE
hello-4111706356   0/1                      0s
hello-4111706356   0/1           0s         0s
hello-4111706356   1/1           5s         5s
```

حالا شما یک کار در حال اجرا برنامه‌ریزی شده توسط cron job "hello" را دیده‌اید.
شما می‌توانید دیدن کار را متوقف کرده و دوباره cron job را برای دیدن اینکه کار را برنامه‌ریزی کرده‌است، مشاهده کنید:

```shell
kubectl get cronjob hello
```

خروجی مشابه زیر خواهد بود:

```
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        50s             75s
```

باید ببینید که cron job `hello` با موفقیت یک کار را در زمان مشخص شده در `LAST SCHEDULE` برنامه‌ریزی کرده است. در حال حاضر، هیچ کار فعالی وجود ندارد، به این معنی که کار تکمیل یا شکست خورده است.

حالا، پادهایی که آخرین کار برنامه‌ریزی شده را ایجاد کرده‌اند را پیدا کرده و خروجی استاندارد یکی از پادها را مشاهده کنید.

{{< note >}}
نام کار با نام پاد متفاوت است.
{{< /note >}}

```shell
# عوض کردن "hello-4111706356" با نام کار در سیستم شما
pods=$(kubectl get pods --selector=job-name=hello-4111706356 --output=jsonpath={.items[*].metadata.name})
```

نمایش لاگ پاد:

```shell
kubectl logs $pods
```

خروجی مشابه زیر خواهد بود:

```
Fri Feb 22 11:02:09 UTC 2019
Hello from the Kubernetes cluster
```

## حذف یک CronJob {#deleting-a-cron-job}

وقتی دیگر نیازی به یک cron job ندارید، آن را با استفاده از `kubectl delete cronjob <نام cronjob>` حذف کنید:

```shell
kubectl delete cronjob hello
```

حذف cron job تمام کارها و پادهایی را که ایجاد کرده‌است و متوقف می‌کند از ایجاد کارهای اضافی.
می‌توانید بیشتر درباره حذف کارها در [جمع آوری زباله](/docs/concepts/architecture/garbage-collection/) بخوانید.
