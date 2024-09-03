---
reviewers:
- janetkuo
title: انجام Rollback بر روی یک DaemonSet
content_type: task
weight: 20
min-kubernetes-server-version: 1.7
---

<!-- overview -->

این صفحه نحوه انجام عملیات rollback بر روی یک {{< glossary_tooltip term_id="daemonset" >}} را نشان می دهد.


## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

شما باید از قبل بدانید که چگونه [بروزرسانی پیشروی را بر روی یک DaemonSet انجام دهید](/docs/tasks/manage-daemon/update-daemon-set/).

<!-- steps -->

## انجام Rollback بر روی یک DaemonSet

### مرحله 1: یافتن نسخه ای از DaemonSet که می خواهید به آن rollback دهید

می توانید این مرحله را انجام ندهید اگر فقط می خواهید به آخرین نسخه rollback دهید.

لیست تمام نسخه های یک DaemonSet را نمایش دهید:

```shell
kubectl rollout history daemonset <daemonset-name>
```

این لیستی از نسخه های DaemonSet را برمی گرداند:

```
daemonsets "<daemonset-name>"
REVISION        CHANGE-CAUSE
1               ...
2               ...
...
```

* Change cause از annotation `kubernetes.io/change-cause` در DaemonSet کپی شده است و در هنگام ایجاد به نسخه ها کپی می شود. شما می توانید `--record=true` را در `kubectl` مشخص کنید تا دستور اجرا شده در annotation change cause را ضبط کند.

برای دیدن جزئیات یک نسخه خاص:

```shell
kubectl rollout history daemonset <daemonset-name> --revision=1
```

این جزئیات آن نسخه را برمی گرداند:

```
daemonsets "<daemonset-name>" with revision #1
Pod Template:
Labels:       foo=bar
Containers:
app:
 Image:        ...
 Port:         ...
 Environment:  ...
 Mounts:       ...
Volumes:      ...
```

### مرحله 2: Rollback به یک نسخه خاص

```shell
# در --to-revision شماره نسخه را که از مرحله 1 دریافت کرده اید مشخص کنید
kubectl rollout undo daemonset <daemonset-name> --to-revision=<revision>
```

اگر موفق باشد، دستور به شما برمی گرداند:

```
daemonset "<daemonset-name>" rolled back
```

{{< note >}}
اگر پرچم `--to-revision` مشخص نشود، kubectl آخرین نسخه را انتخاب می کند.
{{< /note >}}

### مرحله 3: نظارت بر پیشرفت rollback DaemonSet

`kubectl rollout undo daemonset` به سرور می گوید که شروع به rollback DaemonSet کند. Rollback واقعی به صورت ناهمگام در داخل {{< glossary_tooltip term_id="control-plane" text="control plane" >}} انجام می شود.

برای نظارت بر پیشرفت rollback:

```shell
kubectl rollout status ds/<daemonset-name>
```

وقتی که rollback کامل شود، خروجی مشابه زیر خواهد بود:

```
daemonset "<daemonset-name>" successfully rolled out
```


<!-- discussion -->

## درک نسخه های DaemonSet

در مرحله قبلی `kubectl rollout history`، شما یک لیست از نسخه های DaemonSet را دریافت کردید. هر نسخه در یک منبع به نام ControllerRevision ذخیره می شود.

برای دیدن آنچه در هر نسخه ذخیره شده است، منابع raw ControllerRevision نسخه DaemonSet را پیدا کنید:

```shell
kubectl get controllerrevision -l <daemonset-selector-key>=<daemonset-selector-value>
```

این یک لیست از ControllerRevisions را برمی گرداند:

```
NAME                               CONTROLLER                     REVISION   AGE
<daemonset-name>-<revision-hash>   DaemonSet/<daemonset-name>     1          1h
<daemonset-name>-<revision-hash>   DaemonSet/<daemonset-name>     2          1h
```

هر ControllerRevision شامل annotations و template یک نسخه DaemonSet است.

`kubectl rollout undo` یک ControllerRevision خاص را می گیرد و template DaemonSet را با template موجود در ControllerRevision جایگزین می کند. `kubectl rollout undo` معادل به روزرسانی template DaemonSet به یک نسخه قبلی از طریق دستورات دیگر مانند `kubectl edit` یا `kubectl apply` است.

{{< note >}}
نسخه های DaemonSet فقط به صورت پیشرو پیش می روند. به عبارت دیگر، پس از اتمام rollback، شماره نسخه (فیلد `.revision`) ControllerRevision که به آن rollback شده است، پیشرفت می کند. به عنوان مثال، اگر شما در سیستم نسخه 1 و 2 را داشته باشید و از نسخه 2 به نسخه 1 rollback دهید، ControllerRevision با `.revision: 1` به `.revision: 3` پیش می رود.
{{< /note >}}

## رفع مشکلات

* به [رفع مشکلات بروزرسانی پیشروی DaemonSet](/docs/tasks/manage-daemon/update-daemon-set/#troubleshooting) مراجعه کنید.
