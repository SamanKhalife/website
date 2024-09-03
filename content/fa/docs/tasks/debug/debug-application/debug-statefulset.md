---
reviewers:
- bprashanth
- enisoc
- erictune
- foxish
- janetkuo
- kow3ns
- smarterclayton
title: رفع اشکال یک StatefulSet
content_type: task
weight: 30
---

<!-- overview -->
این وظیفه به شما نشان می‌دهد که چگونه یک StatefulSet را رفع اشکال کنید.

## {{% heading "پیش‌نیازها" %}}

* شما باید یک خوشه Kubernetes داشته باشید و ابزار خط فرمان kubectl باید برای ارتباط با خوشه شما پیکربندی شده باشد.
* باید یک StatefulSet در حال اجرا داشته باشید که قصد بررسی آن را دارید.

<!-- steps -->

## رفع اشکال یک StatefulSet

برای لیست کردن تمام پادهایی که به یک StatefulSet تعلق دارند و دارای برچسب `app.kubernetes.io/name=MyApp` هستند، می‌توانید از دستور زیر استفاده کنید:

```shell
kubectl get pods -l app.kubernetes.io/name=MyApp
```

این دستور تمام پادهایی را که توسط StatefulSet مدیریت می‌شوند، لیست می‌کند.

اگر مشاهده کنید که هر پادی در وضعیت `Unknown` یا `Terminating` به مدت طولانی‌تری مانده است، به [وظیفه حذف پادهای StatefulSet](/docs/tasks/run-application/delete-stateful-set/) مراجعه کنید تا دستورالعمل‌های لازم برای برطرف کردن این مشکل را ببینید.
شما می‌توانید پادهای فردی را در یک StatefulSet با استفاده از راهنمای [رفع اشکال پادها](/docs/tasks/debug/debug-application/debug-pods/) بررسی کنید.

## {{% heading "وظیفه بعدی" %}}

برای یادگیری بیشتر در مورد [رفع اشکال یک init-container](/docs/tasks/debug/debug-application/debug-init-containers/)، ادامه دهید.

