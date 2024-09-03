---
title: دسترسی به شل در یک کانتینر در حال اجرا
content_type: task
---

<!-- overview -->

این صفحه نحوه استفاده از `kubectl exec` برای دسترسی به یک شل در یک کانتینر در حال اجرا را نشان می‌دهد.


## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}}


<!-- steps -->

## دسترسی به شل در یک کانتینر

در این تمرین، یک پاد ایجاد می‌کنید که یک کانتینر دارد. این کانتینر تصویر nginx را اجرا می‌کند. فایل پیکربندی برای این پاد به شرح زیر است:

{{% code_sample file="application/shell-demo.yaml" %}}

پاد را ایجاد کنید:

```shell
kubectl apply -f https://k8s.io/examples/application/shell-demo.yaml
```

اطمینان حاصل کنید که کانتینر در حال اجرا است:

```shell
kubectl get pod shell-demo
```

دسترسی به شل کانتینر در حال اجرا:

```shell
kubectl exec --stdin --tty shell-demo -- /bin/bash
```

{{< note >}}
دو خط تیره (`--`) جدا کننده‌ای هستند که آرگومان‌هایی که می‌خواهید به دستور منتقل کنید را از آرگومان‌های kubectl جدا می‌کنند.
{{< /note >}}

در شل خود، لیست دایرکتوری روت را نمایش دهید:

```shell
# این دستور را در داخل کانتینر اجرا کنید
ls /
```

در شل خود، با دیگر دستورات آزمایش کنید. نمونه‌هایی عبارتند از:

```shell
# می‌توانید این دستورات نمونه را در داخل کانتینر اجرا کنید
ls /
cat /proc/mounts
cat /proc/1/maps
apt-get update
apt-get install -y tcpdump
tcpdump
apt-get install -y lsof
lsof
apt-get install -y procps
ps aux
ps aux | grep nginx
```

## نوشتن صفحه ریشه برای nginx

دوباره به فایل پیکربندی پاد خود نگاه کنید. این پاد دارای یک حجم `emptyDir` است و کانتینر این حجم را در `/usr/share/nginx/html` می‌مونت می‌کند.

در شل خود، فایل `index.html` را در دایرکتوری `/usr/share/nginx/html` ایجاد کنید:

```shell
# این دستور را در داخل کانتینر اجرا کنید
echo 'Hello shell demo' > /usr/share/nginx/html/index.html
```

در شل خود، یک درخواست GET به سرور nginx ارسال کنید:

```shell
# این را در شل داخل کانتینر خود اجرا کنید
apt-get update
apt-get install curl
curl http://localhost/
```

خروجی نمایش داده می‌شود که متنی است که شما به فایل `index.html` نوشته‌اید:

```
Hello shell demo
```

وقتی که با شل خود کار تمام شد، `exit` را وارد کنید.

```shell
exit # برای خروج از شل در کانتینر
```

## اجرای دستورات فردی در یک کانتینر

در یک پنجره دستور عادی، نه شل خود، متغیرهای محیطی را در کانتینر در حال اجرا لیست کنید:

```shell
kubectl exec shell-demo -- env
```

با اجرای دستورات دیگر آزمایش کنید. نمونه‌هایی عبارتند از:

```shell
kubectl exec shell-demo -- ps aux
kubectl exec shell-demo -- ls /
kubectl exec shell-demo -- cat /proc/1/mounts
```


<!-- discussion -->

## باز کردن یک شل هنگامی که یک پاد دارای بیش از یک کانتینر است

اگر یک پاد بیش از یک کانتینر داشته باشد، از `--container` یا `-c` برای مشخص کردن کانتینر در دستور `kubectl exec` استفاده کنید. به عنوان مثال، فرض کنید یک پاد به نام my-pod داشته باشید و پاد دارای دو کانتینر به نام _main-app_ و _helper-app_ باشد. دستور زیر یک شل به کانتینر _main-app_ باز می‌کند.

```shell
kubectl exec -i -t my-pod --container main-app -- /bin/bash
```

{{< note >}}
گزینه‌های کوتاه `-i` و `-t` معادل گزینه‌های بلند `--stdin` و `--tty` هستند.
{{< /note >}}


## {{% heading "وظیفه بعدی" %}}


* درباره [kubectl exec](/docs/reference/generated/kubectl/kubectl-commands/#exec) بیشتر بخوانید.