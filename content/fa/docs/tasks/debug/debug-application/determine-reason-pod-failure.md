---
title: تشخیص علت شکست پاد
content_type: task
weight: 30
---

<!-- overview -->

این صفحه نحوه نوشتن و خواندن پیام‌های پایانی کانتینر را نشان می‌دهد.

پیام‌های پایانی به کانتینرها این امکان را می‌دهند که اطلاعاتی درباره رویدادهای فاتال را به یک مکان بنویسند که می‌توان به راحتی آن‌ها را با ابزارهایی مانند داشبوردها و نرم‌افزارهای نظارتی بازیابی و نمایش دهید. در اغلب موارد، اطلاعاتی که شما در یک پیام پایانی قرار می‌دهید، باید همچنین در
[لاگ‌های Kubernetes](/docs/concepts/cluster-administration/logging/)
نیز نوشته شود.

## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}}

<!-- steps -->

## نوشتن و خواندن یک پیام پایانی

در این تمرین، یک پاد ایجاد می‌کنید که یک کانتینر اجرا می‌کند. فایل مانیفست برای آن پاد دستوری را مشخص می‌کند که هنگام شروع کانتینر اجرا می‌شود:

{{% code_sample file="debug/termination.yaml" %}}

1. یک پاد بر اساس فایل پیکربندی YAML ایجاد کنید:

    ```shell
    kubectl apply -f https://k8s.io/examples/debug/termination.yaml
    ```

    در فایل YAML، در فیلدهای `command` و `args`، می‌توانید ببینید که کانتینر برای ۱۰ ثانیه خوابیده و سپس پیام "Sleep expired" را به فایل `/dev/termination-log` می‌نویسد. پس از نوشتن پیام "Sleep expired"، کانتینر خاتمه می‌یابد.

1. اطلاعات مربوط به پاد را نمایش دهید:

    ```shell
    kubectl get pod termination-demo
    ```

    دستور فوق را تکرار کنید تا پاد دیگر در حال اجرا نباشد.

1. اطلاعات دقیق درباره پاد را نمایش دهید:

    ```shell
    kubectl get pod termination-demo --output=yaml
    ```

    خروجی شامل پیام "Sleep expired" است:

    ```yaml
    apiVersion: v1
    kind: Pod
    ...
        lastState:
          terminated:
            containerID: ...
            exitCode: 0
            finishedAt: ...
            message: |
              Sleep expired
            ...
    ```

1. از یک الگوی Go برای فیلتر کردن خروجی استفاده کنید تا شامل فقط پیام پایانی شود:

    ```shell
    kubectl get pod termination-demo -o go-template="{{range .status.containerStatuses}}{{.lastState.terminated.message}}{{end}}"
    ```

اگر شما یک پاد چند کانتینری دارید، می‌توانید از یک الگوی Go استفاده کنید تا نام کانتینر را نیز در بر بگیرد. این کار به شما کمک می‌کند تا کانتینری که در حال خرابی است را پیدا کنید:

```shell
kubectl get pod multi-container-pod -o go-template='{{range .status.containerStatuses}}{{printf "%s:\n%s\n\n" .name .lastState.terminated.message}}{{end}}'
```

## سفارشی‌سازی پیام پایانی

Kubernetes پیام‌های پایانی را از فایل پیام پایانی با مسیر مشخص شده در فیلد `terminationMessagePath` یک کانتینر بازیابی می‌کند که مقدار پیش‌فرض آن `/dev/termination-log` است. با سفارشی‌سازی این فیلد، می‌توانید به Kubernetes بگویید که از یک فایل دیگر استفاده کند. Kubernetes از محتویات فایل مشخص شده برای پر کردن پیام وضعیت کانتینر در هر دو حالت موفقیت آمیز و شکست استفاده می‌کند.

پیام پایانی باید یک پیام نهایی خلاصه‌کننده باشد، مانند یک پیام شکست گزارش خرابی.
kubelet پیام‌هایی را که بیش از ۴۰۹۶ بایت طولانی‌تر هستند، کوتاه‌تر می‌کند.

طول کلی پیام در تمام کانتینرها محدود به ۱۲KiB است که به طور مساوی به هر کانتینر تقسیم می‌شود.
برای مثال، اگر ۱۲ کانتینر (`initContainers` یا `containers`) وجود داشته باشد، هر کدام دارای ۱۰۲۴ بایت فضای پیام پایانی موجود هستند.

مسیر پیش‌فرض پیام پایانی `/dev/termination-log` است.
شما نمی‌توانید مسیر پیام پایانی را پس از راه‌اندازی یک پاد تنظیم کنید.

در مثال زیر، کانتینر پیام‌های پایانی را به `/tmp/my-log` می‌نویسد تا Kubernetes آن را بازیابی کند:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: msg-path-demo
spec:
  containers:
  - name: msg-path-demo-container
    image: debian
    terminationMessagePath: "/tmp/my-log"
```

علاوه بر این، کاربران می‌توانند فیلد `terminationMessagePolicy` یک کانتینر را برای سفارشی‌سازی بیشتر تنظیم کنند. این فیلد به طور پیش‌فرض به "`File`" تنظیم شده است که به این معنی است که پیام‌های پایانی فقط از فایل پیام پایانی بازیابی می‌شوند. با تنظیم `terminationMessagePolicy` به "`FallbackToLogsOnError`"، می‌توانید به Kubernetes بگویید که از آخرین بخش خروجی لاگ کانتینر استفاده کند اگر فایل پیام پایانی خالی باشد و کانتینر با خطا خاتمه یابد. خروجی لاگ به ۲۰۴۸ بایت یا

۸۰ خط محدود شده است، هر کدام کوچکتر است.

## {{% heading "وظیفه بعدی" %}}

* به فیلد `terminationMessagePath` در
  [کانتینر](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#container-v1-core)
  مراجعه کنید.
* به [ImagePullBackOff](/docs/concepts/containers/images/#imagepullbackoff) در [Images](/docs/concepts/containers/images/) مراجعه کنید.
* در مورد [بازیابی لاگ‌ها](/docs/concepts/cluster-administration/logging/) یاد بگیرید.
* درباره [الگوهای Go](https://pkg.go.dev/text/template) یاد بگیرید.
* درباره [وضعیت پاد](/docs/tasks/debug/debug-application/debug-init-containers/#understanding-pod-status) و [فاز پاد](/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase) یاد بگیرید.
* درباره [وضعیت‌های کانتینر](/docs/concepts/workloads/pods/pod-lifecycle/#container-states) یاد بگیرید.

