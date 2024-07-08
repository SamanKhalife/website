---
title: "اجرای Pods فقط بر روی برخی از نودها"
content_type: task
weight: 30
---
<!-- overview -->

این صفحه نحوه اجرای Pods را فقط بر روی برخی از نودها به عنوان بخشی از یک DaemonSet نشان می دهد.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}}

## اجرای Pods فقط بر روی برخی از نودها

تصور کنید که می خواهید یک DaemonSet را اجرا کنید، اما فقط نیاز دارید تا این پادهای daemon را بر روی نودهایی که دارای حافظه SSD محلی هستند، اجرا کنید. به عنوان مثال، Pod ممکن است خدمات کش را به نود ارائه دهد و این کش فقط زمانی مفید است که ذخیره سازی محلی با تاخیر کم در دسترس باشد.

### مرحله 1: اضافه کردن برچسب به نودهای شما

برچسب `ssd=true` را به نودهایی اضافه کنید که دارای SSD هستند.

```shell
kubectl label nodes example-node-1 example-node-2 ssd=true
```

### مرحله 2: ایجاد manifest

بیایید یک DaemonSet ایجاد کنیم که پادهای daemon را فقط بر روی نودهای دارای برچسب SSD اجرا می کند.

بعد، از `nodeSelector` استفاده کنید تا اطمینان حاصل کنید که DaemonSet فقط Pods را بر روی نودهایی که برچسب `ssd` آنها به `"true"` تنظیم شده است، اجرا می کند.

{{% code_sample file="controllers/daemonset-label-selector.yaml" %}}

### مرحله 3: ایجاد DaemonSet

از manifest برای ایجاد DaemonSet با استفاده از `kubectl create` یا `kubectl apply` استفاده کنید.

بیایید یک نود دیگر را با `ssd=true` برچسب بزنیم.

```shell
kubectl label nodes example-node-3 ssd=true
```

برچسب گذاری نود به طور خودکار کنترل پلن را (به طور خاص، کنترل کننده DaemonSet) متحرک می کند تا پاد daemon جدید را در آن نود اجرا کند.

```shell
kubectl get pods -o wide
```

خروجی مشابه زیر خواهد بود:

```
NAME                              READY     STATUS    RESTARTS   AGE    IP      NODE
<daemonset-name><some-hash-01>    1/1       Running   0          13s    .....   example-node-1
<daemonset-name><some-hash-02>    1/1       Running   0          13s    .....   example-node-2
<daemonset-name><some-hash-03>    1/1       Running   0          5s     .....   example-node-3
```
