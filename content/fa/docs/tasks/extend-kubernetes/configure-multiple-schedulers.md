```markdown
---
reviewers:
- davidopp
- madhusudancs
title: تنظیم چندین Scheduler
content_type: task
weight: 20
---

<!-- overview -->

Kubernetes با یک scheduler پیش‌فرض ارائه می‌شود که [اینجا](/docs/reference/command-line-tools-reference/kube-scheduler/) توصیف شده است.
اگر scheduler پیش‌فرض نیازهای شما را برآورده نمی‌کند، می‌توانید scheduler خود را پیاده‌سازی کنید.
علاوه بر این، می‌توانید چندین scheduler را به طور همزمان در کنار scheduler پیش‌فرض اجرا کرده و به Kubernetes بگویید که از کدام scheduler برای هر یک از پادهای شما استفاده کند. بیایید با یک مثال یاد بگیریم که چگونه چندین scheduler را در Kubernetes اجرا کنیم.

توضیح دقیق چگونگی پیاده‌سازی یک scheduler خارج از محدوده این سند است. لطفاً به پیاده‌سازی kube-scheduler در
[pkg/scheduler](https://github.com/kubernetes/kubernetes/tree/master/pkg/scheduler)
در دایرکتوری منبع Kubernetes برای یک مثال مرجع مراجعه کنید.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!-- steps -->

## بسته‌بندی scheduler

باینری scheduler خود را در یک تصویر کانتینر بسته‌بندی کنید. برای مقاصد این مثال، می‌توانید از scheduler پیش‌فرض (kube-scheduler) به عنوان scheduler دوم خود استفاده کنید.
کد منبع [Kubernetes را از GitHub کلون کنید](https://github.com/kubernetes/kubernetes)
و کد منبع را بسازید.

```shell
git clone https://github.com/kubernetes/kubernetes.git
cd kubernetes
make
```

یک تصویر کانتینر حاوی باینری kube-scheduler ایجاد کنید. اینجا یک `Dockerfile` برای ساخت تصویر است:

```docker
FROM busybox
ADD ./_output/local/bin/linux/amd64/kube-scheduler /usr/local/bin/kube-scheduler
```

فایل را به عنوان `Dockerfile` ذخیره کنید، تصویر را بسازید و آن را به یک رجیستری ارسال کنید. این مثال
تصویر را به
[Google Container Registry (GCR)](https://cloud.google.com/container-registry/)
ارسال می‌کند. برای جزئیات بیشتر، لطفاً مستندات GCR
[documentation](https://cloud.google.com/container-registry/docs/) را بخوانید. به طور متناوب،
می‌توانید از [docker hub](https://hub.docker.com/search?q=) نیز استفاده کنید. برای جزئیات بیشتر به مستندات docker hub
[documentation](https://docs.docker.com/docker-hub/repos/create/#create-a-repository) مراجعه کنید.

```shell
docker build -t gcr.io/my-gcp-project/my-kube-scheduler:1.0 .     # نام تصویر و مخزن
gcloud docker -- push gcr.io/my-gcp-project/my-kube-scheduler:1.0 # استفاده شده در اینجا فقط یک مثال است
```

## تعریف یک Deployment Kubernetes برای scheduler

اکنون که scheduler خود را در یک تصویر کانتینر دارید، یک پیکربندی پاد برای آن ایجاد کنید و آن را در خوشه Kubernetes خود اجرا کنید. اما به جای ایجاد مستقیم یک پاد در خوشه، می‌توانید از یک [Deployment](/docs/concepts/workloads/controllers/deployment/)
برای این مثال استفاده کنید. یک [Deployment](/docs/concepts/workloads/controllers/deployment/) یک
[Replica Set](/docs/concepts/workloads/controllers/replicaset/) را مدیریت می‌کند که به نوبه خود پادها را مدیریت می‌کند،
و بنابراین scheduler را در برابر خرابی‌ها مقاوم می‌کند. در اینجا پیکربندی deployment
است. آن را به عنوان `my-scheduler.yaml` ذخیره کنید:

{{% code_sample file="admin/sched/my-scheduler.yaml" %}}

در مانفیست فوق، از یک [KubeSchedulerConfiguration](/docs/reference/scheduling/config/)
برای سفارشی‌سازی رفتار پیاده‌سازی scheduler خود استفاده می‌کنید. این پیکربندی در هنگام راه‌اندازی به `kube-scheduler` با گزینه `--config` ارائه شده است. ConfigMap `my-scheduler-config` فایل پیکربندی را ذخیره می‌کند. پاد Deployment `my-scheduler` ConfigMap `my-scheduler-config` را به عنوان یک حجم نصب می‌کند.

در پیکربندی Scheduler مذکور، پیاده‌سازی scheduler شما از طریق یک [KubeSchedulerProfile](/docs/reference/config-api/kube-scheduler-config.v1/#kubescheduler-config-k8s-io-v1-KubeSchedulerProfile) نمایش داده شده است.
{{< note >}}
برای تعیین اینکه آیا یک scheduler مسئول برنامه‌ریزی یک پاد خاص است، فیلد `spec.schedulerName` در یک PodTemplate یا مانفیست پاد باید با فیلد `schedulerName` در `KubeSchedulerProfile` مطابقت داشته باشد.
همه schedulers که در خوشه اجرا می‌شوند باید نام‌های منحصر به فردی داشته باشند.
{{< /note >}}

همچنین توجه کنید که یک حساب سرویس اختصاصی `my-scheduler` ایجاد می‌کنید و ClusterRole `system:kube-scheduler` را به آن اختصاص می‌دهید تا بتواند همان امتیازات `kube-scheduler` را بدست آورد.

لطفاً به
[مستندات kube-scheduler](/docs/reference/command-line-tools-reference/kube-scheduler/) برای
توضیحات دقیق دیگر آرگومان‌های خط فرمان و
[مرجع پیکربندی Scheduler](/docs/reference/config-api/kube-scheduler-config.v1/) برای
توضیحات دقیق دیگر پیکربندی‌های قابل سفارشی‌سازی `kube-scheduler` مراجعه کنید.

## اجرای دومین scheduler در خوشه

برای اجرای scheduler خود در یک خوشه Kubernetes، پیکربندی مشخص شده در بالا را در یک خوشه Kubernetes ایجاد کنید:

```shell
kubectl create -f my-scheduler.yaml
```

تأیید کنید که پاد scheduler در حال اجرا است:

```shell
kubectl get pods --namespace=kube-system
```

```
NAME                                           READY     STATUS    RESTARTS   AGE
....
my-scheduler-lnf4s-4744f                       1/1       Running   0          2m
...
```

باید یک پاد my-scheduler در وضعیت "Running" را در این لیست در کنار پاد kube-scheduler پیش‌فرض ببینید.

### فعال‌سازی انتخاب رهبر

برای اجرای چندین scheduler با انتخاب رهبر فعال، باید مراحل زیر را انجام دهید:

فیلدهای زیر را برای KubeSchedulerConfiguration در ConfigMap `my-scheduler-config` در فایل YAML خود به روز رسانی کنید:

* `leaderElection.leaderElect` را به `true`
* `leaderElection.resourceNamespace` را به `<lock-object-namespace>` تنظیم کنید
* `leaderElection.resourceName` را به `<lock-object-name>` تنظیم کنید

{{< note >}}
سیستم کنترل قفل‌ها را برای شما ایجاد می‌کند، اما فضای نام باید قبلاً وجود داشته باشد. می‌توانید از فضای نام `kube-system` استفاده کنید.
{{< /note >}}

اگر RBAC در خوشه شما فعال باشد، باید ClusterRole `system:kube-scheduler` را به روز رسانی کنید. نام scheduler خود را به resourceNames قوانین اعمال شده برای منابع `endpoints` و `leases` اضافه کنید، مانند مثال زیر:

```shell
kubectl edit clusterrole system:kube-scheduler
```

{{% code_sample file="admin/sched/clusterrole.yaml" %}}

## مشخص کردن schedulers برای pods

حالا که scheduler دوم شما در حال اجرا است، برخی از pods را ایجاد کنید و آن‌ها را هدایت کنید تا توسط scheduler پیش‌فرض یا آنی که شما ارسال کرده‌اید برنامه‌ریزی شوند. برای برنامه‌ریزی یک pod خاص با استفاده از یک scheduler مشخص، نام scheduler را در مشخصات آن pod مشخص کنید. بیایید به سه مثال نگاهی بیندازیم.

- مشخصات Pod بدون نام scheduler

  {{% code_sample file="admin/sched/pod1.yaml" %}}

  وقتی نام scheduler مشخص نشود، pod به طور خودکار با استفاده از پیش‌فرض برنامه‌ریزی می‌شود.

  این فایل را به عنوان `pod1.yaml` ذخیره کنید و آن را به خوشه Kubernetes ارسال کنید.

  ```shell
  kubectl create -f pod1.yaml
  ```

- مشخصات Pod با `default-scheduler`

  {{% code_sample file="admin/sched/pod2.yaml" %}}

  یک scheduler با تعیین نام scheduler به عنوان یک مقدار برای `spec.schedulerName` مشخص می‌شود. در این مورد، ما نام scheduler پیش‌فرض را که `default-scheduler` است مشخص کرده‌ایم.

  این فایل را به عنوان `pod2.yaml` ذخیره کنید و آن را به خوشه Kubernetes ارسال کنید.

  ```shell
  kubectl create -f pod2.yaml
  ```

- مشخصات Pod با `my-scheduler`

  {{% code_sample file="admin/sched/pod3.yaml" %}}

  در این مورد، ما مشخص می‌کنیم که این pod باید با استفاده از schedulerی که ما ارسال کرده‌ایم - `my-scheduler` - برنامه‌ریزی شود. توجه کنید که مقدار `spec.schedulerName` باید با نام ارسال شده برای scheduler در فیلد `schedulerName` مطابقت داشته باشد.

  این فایل را به عنوان `pod3.yaml` ذخیره کنید و آن را به خوشه Kubernetes ارسال کنید.

  ```shell
  kubectl create -f pod3.yaml
  ```

  تأیید کنید که هر سه pod در حال اجرا هستند.

  ```shell
  kubectl get pods
  ```

<!-- discussion -->

### تأیید اینکه pods با schedulers مورد نظر برنامه‌ریزی شده‌اند

برای آسان‌تر کار کردن با این مثال‌ها، ما تأیید نکردیم که podها واقعاً با schedulers مورد نظر برنامه‌ریزی شده‌اند. می‌توانیم این موضوع را با تغییر ترتیب ارسال پیکربندی pod و deployment بالا تأیید کنیم. اگر تمام تنظیمات pod را به یک خوشه Kubernetes ارسال کنیم قبل از ارسال پیکربندی deployment scheduler، مشاهده می‌شود که pod `annotation-second-scheduler` به حالت "Pending" برای همیشه باقی می‌ماند در حالی که دو pod دیگر برنامه‌ریزی می‌شوند. بعد از ارسال پیکربندی deployment scheduler و اجرای scheduler جدید ما، pod `annotation-second-scheduler` نیز برنامه‌ریزی می‌شود.

همچنین می‌توانید به ورودی‌های "Scheduled" در لاگ‌های رویداد نگاهی بیندازید تا تأیید کنید که podها توسط schedulers مورد نظر برنامه‌ریزی شده‌اند.

```shell
kubectl get events
```
همچنین می‌توانید از [پیکربندی scheduler سفارشی](/docs/reference/scheduling/config/#multiple-profiles)
یا تصویر کانتینر سفارشی برای pod استاتیک مانیفست خود scheduler اصلی خوشه را با مشخصات مربوط به نودهای صفحه کنترل کننده اصلی تغییر دهید.
