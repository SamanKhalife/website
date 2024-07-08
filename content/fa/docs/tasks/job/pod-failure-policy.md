---
title: مدیریت شکست‌های تکرارپذیر و غیرتکرارپذیر پاد با استفاده از سیاست شکست پاد
content_type: task
min-kubernetes-server-version: v1.25
weight: 60
---

{{< feature-state feature_gate_name="JobPodFailurePolicy" >}}

<!-- overview -->

این سند نحوه استفاده از
[سیاست شکست پاد](/docs/concepts/workloads/controllers/job#pod-failure-policy)،
همراه با سیاست شکست بازگشت پیش‌فرض
[سیاست شکست بازگشت پاد](/docs/concepts/workloads/controllers/job#pod-backoff-failure-policy)،
را برای بهبود کنترل در برخورد با شکست‌های سطح پاد یا کانتینر در یک {{<glossary_tooltip text="Job" term_id="job">}} نشان می‌دهد.

تعریف سیاست شکست پاد ممکن است به شما کمک کند:
* استفاده بهتر از منابع محاسباتی با جلوگیری از تلاش‌های تکراری غیرضروری پاد.
* جلوگیری از شکست‌های Job به دلیل قطعی‌های پاد (مانند {{<glossary_tooltip text="پیش‌دستی" term_id="preemption" >}}،
{{<glossary_tooltip text="تخلیه توسط API" term_id="api-eviction" >}}
یا تخلیه مبتنی بر {{<glossary_tooltip text="taint" term_id="taint" >}}).

## {{% heading "prerequisites" %}}

شما باید با استفاده اولیه از [Job](/docs/concepts/workloads/controllers/job/) آشنا باشید.

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

اطمینان حاصل کنید که [دروازه‌های ویژگی](/docs/reference/command-line-tools-reference/feature-gates/)
`PodDisruptionConditions` و `JobPodFailurePolicy` در خوشه شما فعال هستند.

## استفاده از سیاست شکست پاد برای جلوگیری از تلاش‌های تکراری غیرضروری پاد

با مثال زیر، می‌توانید یاد بگیرید که چگونه از سیاست شکست پاد برای
جلوگیری از تلاش‌های تکراری غیرضروری پاد استفاده کنید زمانی که شکست پاد نشان‌دهنده یک خطای نرم‌افزاری غیرتکرارپذیر است.

ابتدا، یک Job بر اساس پیکربندی زیر ایجاد کنید:

{{% code_sample file="/controllers/job-pod-failure-policy-failjob.yaml" %}}

با اجرای دستور:

```sh
kubectl create -f job-pod-failure-policy-failjob.yaml
```

پس از حدود ۳۰ ثانیه کل Job باید خاتمه یابد. وضعیت Job را با اجرای دستور زیر بررسی کنید:

```sh
kubectl get jobs -l job-name=job-pod-failure-policy-failjob -o yaml
```

در وضعیت Job، یک شرط `Failed` با فیلد `reason` برابر `PodFailurePolicy` مشاهده کنید. علاوه بر این، فیلد `message` شامل اطلاعات بیشتری درباره خاتمه Job است، مانند:
`Container main for pod default/job-pod-failure-policy-failjob-8ckj8 failed with exit code 42 matching FailJob rule at index 0`.

برای مقایسه، اگر سیاست شکست پاد غیرفعال باشد، ۶ تلاش تکراری پاد انجام خواهد شد که حداقل ۲ دقیقه زمان می‌برد.

### پاکسازی

Job ایجاد شده را حذف کنید:

```sh
kubectl delete jobs/job-pod-failure-policy-failjob
```

خوشه به‌طور خودکار پادها را پاکسازی می‌کند.

## استفاده از سیاست شکست پاد برای نادیده گرفتن قطعی‌های پاد

با مثال زیر، می‌توانید یاد بگیرید که چگونه از سیاست شکست پاد برای نادیده گرفتن قطعی‌های پاد و جلوگیری از افزایش شمارنده تلاش مجدد پاد به سمت محدودیت `.spec.backoffLimit` استفاده کنید.

{{< caution >}}
زمان‌بندی برای این مثال مهم است، بنابراین ممکن است بخواهید مراحل را قبل از اجرا بخوانید. برای ایجاد قطعی پاد، مهم است که گره را در حالی که پاد روی آن در حال اجراست (در کمتر از ۹۰ ثانیه از زمان زمان‌بندی پاد) تخلیه کنید.
{{< /caution >}}

1. یک Job بر اساس پیکربندی زیر ایجاد کنید:

   {{% code_sample file="/controllers/job-pod-failure-policy-ignore.yaml" %}}

   با اجرای دستور:

   ```sh
   kubectl create -f job-pod-failure-policy-ignore.yaml
   ```

2. این دستور را برای بررسی `nodeName` که پاد روی آن زمان‌بندی شده است اجرا کنید:

   ```sh
   nodeName=$(kubectl get pods -l job-name=job-pod-failure-policy-ignore -o jsonpath='{.items[0].spec.nodeName}')
   ```

3. گره را قبل از تکمیل پاد تخلیه کنید (در کمتر از ۹۰ ثانیه):

   ```sh
   kubectl drain nodes/$nodeName --ignore-daemonsets --grace-period=0
   ```

4. `.status.failed` را بررسی کنید تا اطمینان حاصل کنید که شمارنده Job افزایش نیافته است:

   ```sh
   kubectl get jobs -l job-name=job-pod-failure-policy-ignore -o yaml
   ```

5. گره را از حالت تخلیه خارج کنید:

   ```sh
   kubectl uncordon nodes/$nodeName
   ```

Job از سر گرفته شده و موفق می‌شود.

برای مقایسه، اگر سیاست شکست پاد غیرفعال باشد، قطعی پاد منجر به خاتمه کل Job می‌شود (زیرا `.spec.backoffLimit` برابر با ۰ است).

### پاکسازی

Job ایجاد شده را حذف کنید:

```sh
kubectl delete jobs/job-pod-failure-policy-ignore
```

خوشه به‌طور خودکار پادها را پاکسازی می‌کند.

## استفاده از سیاست شکست پاد برای جلوگیری از تلاش‌های تکراری غیرضروری پاد بر اساس شرایط سفارشی پاد

با مثال زیر، می‌توانید یاد بگیرید که چگونه از سیاست شکست پاد برای جلوگیری از تلاش‌های تکراری غیرضروری پاد بر اساس شرایط سفارشی پاد استفاده کنید.

{{< note >}}
مثال زیر از نسخه 1.27 کار می‌کند زیرا به انتقال پادهای حذف شده، در فاز `Pending`، به یک فاز نهایی متکی است
(نگاه کنید به: [فاز پاد](/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase)).
{{< /note >}}

1. ابتدا، یک Job بر اساس پیکربندی زیر ایجاد کنید:

   {{% code_sample file="/controllers/job-pod-failure-policy-config-issue.yaml" %}}

   با اجرای دستور:

   ```sh
   kubectl create -f job-pod-failure-policy-config-issue.yaml
   ```

   توجه کنید که، تصویر نادرست پیکربندی شده است، زیرا وجود ندارد.

2. وضعیت پادهای Job را با اجرای دستور زیر بررسی کنید:

   ```sh
   kubectl get pods -l job-name=job-pod-failure-policy-config-issue -o yaml
   ```

   شما خروجی مشابه زیر مشاهده خواهید کرد:
   ```yaml
   containerStatuses:
   - image: non-existing-repo/non-existing-image:example
      ...
      state:
      waiting:
         message: Back-off pulling image "non-existing-repo/non-existing-image:example"
         reason: ImagePullBackOff
         ...
   phase: Pending
   ```

   توجه کنید که پاد در فاز `Pending` باقی می‌ماند زیرا نمی‌تواند تصویر نادرست پیکربندی شده را بارگیری کند. این ممکن است به طور اصولی یک مشکل گذرا باشد و تصویر بتواند بارگیری شود. اما در این مورد، تصویر وجود ندارد، بنابراین این واقعیت را با یک شرط سفارشی نشان می‌دهیم.

3. شرط سفارشی را اضافه کنید. ابتدا پچ را با اجرای دستور زیر آماده کنید:

   ```sh
   cat <<EOF > patch.yaml
   status:
     conditions:
     - type: ConfigIssue
       status: "True"
       reason: "NonExistingImage"
       lastTransitionTime: "$(date -u +"%Y-%m-%dT%H:%M:%SZ")"
   EOF
   ```
   دوم، یکی از پادهای ایجاد شده توسط Job را با اجرای دستور زیر انتخاب کنید:
   ```
   podName=$(kubectl get pods -l job-name=job-pod-failure-policy-config-issue -o jsonpath='{.items[0].metadata.name}')
   ```

   سپس، پچ را بر روی یکی از پادها با اجرای دستور زیر اعمال کنید:

   ```sh
   kubectl patch pod $podName --subresource=status --patch-file=patch.yaml
   ```

   اگر به درستی اعمال شود، اطلاعیه‌ای مشابه زیر دریافت خواهید کرد:

   ```sh
   pod/job-pod-failure-policy-config-issue-k6pvp patched
   ```

4. پاد را برای انتقال آن به فاز `Failed` حذف کنید، با اجرای دستور زیر:

   ```sh
   kubectl delete pods/$podName
   ```

5. وضعیت Job را با اجرای دستور زیر بررسی کنید:

   ```sh
   kubectl get jobs -l job-name=job-pod-failure-policy-config-issue -o yaml
   ```

   در وضعیت Job، یک شرط `Failed` با فیلد `reason` برابر `PodFailurePolicy` مشاهده کنید. علاوه بر این، فیلد `message` شامل اطلاعات بیشتری درباره خاتمه Job است، مانند:
   `Pod default/job-pod-failure-policy-config-issue-k6pvp has condition ConfigIssue matching FailJob rule at index 0`.

{{< note >}}
در یک محیط تولیدی، مراحل 3 و 4 باید توسط یک کنترلر ارائه شده توسط کاربر خودکار شوند.
{{< /note >}}

### پاکسازی

Job ایجاد شده را حذف کنید:

```sh
kubectl delete jobs/job-pod-failure-policy-config-issue
```

خوشه به‌طور خودکار پادها را پاکسازی می‌کند.

## جایگزین‌ها

می‌توانید به‌طور کامل به
[سیاست شکست بازگشت پاد](/docs/concepts/workloads/controllers/job#pod-backoff-failure-policy) اتکا کنید،
با مشخص کردن فیلد `.spec.backoffLimit` برای Job. اما، در بسیاری از مواقع
یافتن تعادلی بین تنظیم مقدار کم برای `.spec.backoffLimit` برای جلوگیری از تلاش‌های تکراری غیرضروری پاد، و در عین حال بالا بودن مقدار برای اطمینان از اینکه Job به دلیل قطعی‌های پاد خاتمه نمی‌یابد، مشکل است.
