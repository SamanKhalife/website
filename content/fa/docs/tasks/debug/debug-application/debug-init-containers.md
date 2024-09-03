---
reviewers:
- bprashanth
- enisoc
- erictune
- foxish
- janetkuo
- kow3ns
- smarterclayton
title: رفع مشکلات کانتینرهای Init
content_type: task
weight: 40
---

<!-- overview -->

این صفحه نحوه بررسی مشکلات مرتبط با اجرای کانتینرهای Init را نشان می‌دهد. دستورات مثال زیر به پاد به عنوان `<pod-name>` و کانتینرهای Init به عنوان `<init-container-1>` و `<init-container-2>` اشاره دارند.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

* باید با مفاهیم پایه‌ای [کانتینرهای Init](/docs/concepts/workloads/pods/init-containers/) آشنا باشید.
* باید [یک کانتینر Init پیکربندی کرده باشید](/docs/tasks/configure-pod-container/configure-pod-initialization/#create-a-pod-that-has-an-init-container).

<!-- steps -->

## بررسی وضعیت کانتینرهای Init

وضعیت پاد خود را نمایش دهید:

```shell
kubectl get pod <pod-name>
```

برای مثال، وضعیت `Init:1/2` نشان می‌دهد که یکی از دو کانتینر Init با موفقیت اجرا شده است:

```
NAME         READY     STATUS     RESTARTS   AGE
<pod-name>   0/1       Init:1/2   0          7s
```

برای مشاهده مثال‌های بیشتری از مقادیر وضعیت و معانی آن‌ها، به [فهم وضعیت پاد](#understanding-pod-status) مراجعه کنید.

## دریافت جزئیات درباره کانتینرهای Init

اطلاعات دقیق‌تری درباره اجرای کانتینرهای Init مشاهده کنید:

```shell
kubectl describe pod <pod-name>
```

برای مثال، یک پاد با دو کانتینر Init ممکن است به صورت زیر نمایش داده شود:

```
Init Containers:
  <init-container-1>:
    Container ID:    ...
    ...
    State:           Terminated
      Reason:        Completed
      Exit Code:     0
      Started:       ...
      Finished:      ...
    Ready:           True
    Restart Count:   0
    ...
  <init-container-2>:
    Container ID:    ...
    ...
    State:           Waiting
      Reason:        CrashLoopBackOff
    Last State:      Terminated
      Reason:        Error
      Exit Code:     1
      Started:       ...
      Finished:      ...
    Ready:           False
    Restart Count:   3
    ...
```

همچنین می‌توانید به صورت برنامه‌نویسی به وضعیت‌های کانتینرهای Init دسترسی پیدا کنید با خواندن فیلد `status.initContainerStatuses` در مشخصه پاد:

```shell
kubectl get pod nginx --template '{{.status.initContainerStatuses}}'
```

این دستور به صورت رویه‌ای همان اطلاعات بالا را به صورت JSON خام برمی‌گرداند.

## دسترسی به لاگ‌های کانتینرهای Init

نام کانتینر Init را به همراه نام پاد برای دسترسی به لاگ‌های آن ارسال کنید.

```shell
kubectl logs <pod-name> -c <init-container-2>
```

کانتینرهای Init که اسکریپت shell اجرا می‌کنند، دستورات را هنگام اجرا چاپ می‌کنند. به عنوان مثال، می‌توانید این کار را در Bash با اجرای `set -x` در ابتدای اسکریپت انجام دهید.

<!-- discussion -->

## فهمیدن وضعیت پاد

وضعیت یک پاد با `Init:` شروع می‌شود و وضعیت اجرای کانتینرهای Init را خلاصه می‌کند. جدول زیر برخی از مقادیر وضعیت مثالی را که ممکن است در اشکال‌زدایی کانتینرهای Init مشاهده کنید، توضیح می‌دهد.

وضعیت | معنی
------ | -------
`Init:N/M` | پاد دارای `M` کانتینر Init است و تا کنون `N` کانتینر اجرا شده‌اند.
`Init:Error` | یک کانتینر Init نتوانست اجرا شود.
`Init:CrashLoopBackOff` | یک کانتینر Init به طور مکرر نتوانست اجرا شود.
`Pending` | پاد هنوز با اجرای کانتینرهای Init شروع نکرده است.
`PodInitializing` یا `Running` | پاد همه کانتینرهای Init خود را اجرا کرده است یا در حال انجام آن است.
