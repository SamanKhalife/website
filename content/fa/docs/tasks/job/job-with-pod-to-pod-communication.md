---
title: Job با ارتباط پاد-به-پاد
content_type: task
min-kubernetes-server-version: v1.21
weight: 30
---

<!-- overview -->

در این مثال، شما یک Job را در [حالت تکمیل‌شده ایندکس شده](/blog/2021/04/19/introducing-indexed-jobs/) اجرا خواهید کرد که به گونه‌ای پیکربندی شده که پادهایی که توسط Job ایجاد می‌شوند می‌توانند با استفاده از نام‌های میزبان پاد به جای آدرس‌های IP پاد با یکدیگر ارتباط برقرار کنند.

پادهای درون یک Job ممکن است نیاز به ارتباط با یکدیگر داشته باشند. بارکاری که در هر پاد اجرا می‌شود می‌تواند به سرور API Kubernetes برای آگاهی از IPهای دیگر پادها پرس‌وجو کند، اما ساده‌تر این است که به سیستم DNS داخلی Kubernetes اعتماد کنیم.

Jobs در حالت تکمیل‌شده ایندکس شده به طور خودکار نام میزبان پادها را به فرمت `${jobName}-${completionIndex}` تنظیم می‌کنند. می‌توانید از این فرمت برای ساخت نام‌های میزبان پادها به صورت تعیین‌کننده استفاده کنید و ارتباط پاد-به-پاد را بدون نیاز به ایجاد یک اتصال کلاینت به صفحه کنترل Kubernetes برای به دست آوردن نام‌های میزبان/آدرس‌های IP پادها از طریق درخواست‌های API فعال کنید.

این پیکربندی برای موارد استفاده‌ای مفید است که ارتباط شبکه‌ای پادها مورد نیاز است اما نمی‌خواهید به یک اتصال شبکه‌ای با سرور API Kubernetes وابسته باشید.

## {{% heading "prerequisites" %}}

شما باید با استفاده پایه‌ای از [Job](/docs/concepts/workloads/controllers/job/) آشنا باشید.

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

{{<note>}}
اگر از MiniKube یا ابزار مشابهی استفاده می‌کنید، ممکن است نیاز به انجام
[مراحل اضافی](https://minikube.sigs.k8s.io/docs/handbook/addons/ingress-dns/)
برای اطمینان از داشتن DNS داشته باشید.
{{</note>}}

<!-- steps -->

## شروع یک Job با ارتباط پاد-به-پاد

برای فعال کردن ارتباط پاد-به-پاد با استفاده از نام‌های میزبان پادها در یک Job، باید اقدامات زیر را انجام دهید:

1. یک [سرویس بدون سرور](/docs/concepts/services-networking/service/#headless-services)
با یک انتخاب‌گر برچسب معتبر برای پادهایی که توسط Job شما ایجاد می‌شوند، تنظیم کنید. سرویس بدون سرور باید در همان namespace به عنوان Job باشد. یک روش ساده برای انجام این کار استفاده از انتخاب‌گر `job-name: <your-job-name>` است، زیرا برچسب `job-name` به طور خودکار توسط Kubernetes اضافه می‌شود. این پیکربندی سیستم DNS را به ایجاد رکوردهای نام‌های میزبان پادهای اجرای Job شما وادار می‌کند.

2. سرویس بدون سرور را به عنوان سرویس زیردامنه برای پادهای Job با قرار دادن مقدار زیر در مشخصات قالب Job پیکربندی کنید:

   ```yaml
   subdomain: <headless-svc-name>
   ```

### مثال 
در زیر یک مثال کاری از یک Job با ارتباط پاد-به-پاد از طریق نام‌های میزبان پاد فعال شده است.
Job تنها پس از موفقیت‌آمیز بودن پینگ هر پاد دیگر با استفاده از نام‌های میزبان تکمیل می‌شود.

{{<note>}}
در اسکریپت Bash که در هر پاد در مثال زیر اجرا می‌شود، نام‌های میزبان پادها می‌توانند در صورت نیاز از خارج از namespace با نام namespace پیشوند داشته باشند.
{{</note>}}

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-svc
spec:
  clusterIP: None # clusterIP باید None باشد تا یک سرویس بدون سرور ایجاد شود
  selector:
    job-name: example-job # باید با نام Job مطابقت داشته باشد
---
apiVersion: batch/v1
kind: Job
metadata:
  name: example-job
spec:
  completions: 3
  parallelism: 3
  completionMode: Indexed
  template:
    spec:
      subdomain: headless-svc # باید با نام سرویس مطابقت داشته باشد
      restartPolicy: Never
      containers:
      - name: example-workload
        image: bash:latest
        command:
        - bash
        - -c
        - |
          for i in 0 1 2
          do
            gotStatus="-1"
            wantStatus="0"
            while [ $gotStatus -ne $wantStatus ]
            do
              ping -c 1 example-job-${i}.headless-svc > /dev/null 2>&1
              gotStatus=$?
              if [ $gotStatus -ne $wantStatus ]; then
                echo "Failed to ping pod example-job-${i}.headless-svc, retrying in 1 second..."
                sleep 1
              fi
            done
            echo "Successfully pinged pod: example-job-${i}.headless-svc"
          done
```

پس از اعمال مثال بالا، هر یک از پادها می‌توانند با استفاده از `<pod-hostname>.<headless-service-name>` یکدیگر را از طریق شبکه دسترسی پیدا کنند. شما باید خروجی مشابه زیر را مشاهده کنید:

```shell
kubectl logs example-job-0-qws42
```

```
Failed to ping pod example-job-0.headless-svc, retrying in 1 second...
Successfully pinged pod: example-job-0.headless-svc
Successfully pinged pod: example-job-1.headless-svc
Successfully pinged pod: example-job-2.headless-svc
```

{{<note>}}
به خاطر داشته باشید که فرمت نام `<pod-hostname>.<headless-service-name>` که در این مثال استفاده شده است با سیاست DNS تنظیم شده به `None` یا `Default` کار نخواهد کرد.
می‌توانید در مورد سیاست‌های DNS پادها [اینجا](/docs/concepts/services-networking/dns-pod-service/#pod-s-dns-policy) بیشتر بیاموزید.
{{</note>}}
