```markdown
reviewers:
- Random-Liu
- feiskyer
- mrunalp
title: Debugging Kubernetes nodes with crictl
content_type: task
weight: 30
---

<!-- overview -->

{{< feature-state for_k8s_version="v1.11" state="stable" >}}

`crictl` یک رابط خط فرمانی برای رانتایم‌های قابل‌استفاده با CRI است. شما می‌توانید از آن برای بازرسی و اشکال‌زدایی رانتایم‌های کانتینری و برنامه‌ها بر روی یک نود Kubernetes استفاده کنید. `crictl` و منبع آن در مخزن [cri-tools](https://github.com/kubernetes-sigs/cri-tools) میزبانی می‌شوند.

## {{% heading "prerequisites" %}}

`crictl` نیاز به یک سیستم عامل Linux با یک رانتایم CRI دارد.

<!-- steps -->

## نصب crictl

می‌توانید یک آرشیو فشرده `crictl` را از صفحه [release](https://github.com/kubernetes-sigs/cri-tools/releases) cri-tools برای چندین معماری مختلف دانلود کنید. نسخه‌ای که با نسخه Kubernetes شما سازگار است را دانلود کنید. آن را استخراج کرده و به یک مکان در مسیر سیستم خود، مانند `/usr/local/bin/`، منتقل کنید.

## استفاده عمومی

دستور `crictl` دارای چند زیردسته و پرچم زمان اجرا است. برای کسب اطلاعات بیشتر، از `crictl help` یا `crictl <subcommand> help` استفاده کنید.

می‌توانید پایانه برای `crictl` را با انجام یکی از اقدامات زیر تنظیم کنید:

* پرچم‌های `--runtime-endpoint` و `--image-endpoint` را تنظیم کنید.
* متغیرهای محیطی `CONTAINER_RUNTIME_ENDPOINT` و `IMAGE_SERVICE_ENDPOINT` را تنظیم کنید.
* پایانه را در پرونده پیکربندی `/etc/crictl.yaml` تنظیم کنید. برای مشخص کردن یک پرونده دیگر، زمانی که `crictl` را اجرا می‌کنید، از پرچم `--config=PATH_TO_FILE` استفاده کنید.

{{<note>}}
اگر پایانه‌ای را تنظیم نکنید، `crictl` سعی می‌کند به لیستی از پایانه‌های شناخته‌شده وصل شود که ممکن است منجر به افت عملکرد شود.
{{</note>}}

همچنین می‌توانید مقادیر timeout را هنگام اتصال به سرور مشخص کنید و اشکال‌زدایی را فعال یا غیرفعال کنید، با مشخص کردن مقادیر `timeout` یا `debug` در پرونده پیکربندی یا استفاده از پرچم‌های خط فرمان `--timeout` و `--debug`.

برای مشاهده یا ویرایش پیکربندی فعلی، محتویات `/etc/crictl.yaml` را مشاهده یا ویرایش کنید. به عنوان مثال، پیکربندی هنگام استفاده از رانتایم containerd به این شکل خواهد بود:

```
runtime-endpoint: unix:///var/run/containerd/containerd.sock
image-endpoint: unix:///var/run/containerd/containerd.sock
timeout: 10
debug: true
```

برای کسب اطلاعات بیشتر در مورد `crictl`، به مستندات [`crictl`](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md) مراجعه کنید.

## مثال‌های دستورات crictl

مثال‌های زیر نمایش می‌دهند چگونه برخی از دستورات `crictl` و خروجی‌های مثال را اجرا کنید.

{{< warning >}}
اگر از `crictl` برای ایجاد pod sandbox یا container در یک cluster Kubernetes در حال اجرا استفاده کنید، Kubelet در نهایت آنها را حذف خواهد کرد. `crictl` یک ابزار کاربردی برای اشکال‌زدایی است و نه یک ابزار جهت کارهای عمومی.
{{< /warning >}}

### لیست کردن pods

لیست تمام pods‌ها:

```shell
crictl pods
```

خروجی مشابه این است:

```
POD ID              CREATED              STATE               NAME                         NAMESPACE           ATTEMPT
926f1b5a1d33a       About a minute ago   Ready               sh-84d7dcf559-4r2gq          default             0
4dccb216c4adb       About a minute ago   Ready               nginx-65899c769f-wv2gp       default             0
a86316e96fa89       17 hours ago         Ready               kube-proxy-gblk4             kube-system         0
919630b8f81f1       17 hours ago         Ready               nvidia-device-plugin-zgbbv   kube-system         0
```

لیست pods بر اساس نام:

```shell
crictl pods --name nginx-65899c769f-wv2gp
```

خروجی مشابه این است:

```
POD ID              CREATED             STATE               NAME                     NAMESPACE           ATTEMPT
4dccb216c4adb       2 minutes ago       Ready               nginx-65899c769f-wv2gp   default             0
```

لیست pods بر اساس label:

```shell
crictl pods --label run=nginx
```

خروجی مشابه این است:

```
POD ID              CREATED             STATE               NAME                     NAMESPACE           ATTEMPT
4dccb216c4adb       2 minutes ago       Ready               nginx-65899c769f-wv2gp   default             0
```
```markdown
### لیست تصاویر

لیست تمام تصاویر:

```shell
crictl images
```

خروجی مشابه این است:

```
IMAGE                                     TAG                 IMAGE ID            SIZE
busybox                                   latest              8c811b4aec35f       1.15MB
k8s-gcrio.azureedge.net/hyperkube-amd64   v1.10.3             e179bbfe5d238       665MB
k8s-gcrio.azureedge.net/pause-amd64       3.1                 da86e6ba6ca19       742kB
nginx                                     latest              cd5239a0906a6       109MB
```

لیست تصاویر بر اساس مخزن:

```shell
crictl images nginx
```

خروجی مشابه این است:

```
IMAGE               TAG                 IMAGE ID            SIZE
nginx               latest              cd5239a0906a6       109MB
```

تنها نمایش شناسه‌های تصاویر:

```shell
crictl images -q
```

خروجی مشابه این است:

```
sha256:8c811b4aec35f259572d0f79207bc0678df4c736eeec50bc9fec37ed936a472a
sha256:e179bbfe5d238de6069f3b03fccbecc3fb4f2019af741bfff1233c4d7b2970c5
sha256:da86e6ba6ca197bf6bc5e9d900febd906b133eaa4750e6bed647b0fbe50ed43e
sha256:cd5239a0906a6ccf0562354852fae04bc5b52d72a2aff9a871ddb6bd57553569
```

### لیست کانتینرها

لیست تمام کانتینرها:

```shell
crictl ps -a
```

خروجی مشابه این است:

```
CONTAINER ID        IMAGE                                                                                                             CREATED             STATE               NAME                       ATTEMPT
1f73f2d81bf98       busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47                                   7 minutes ago       Running             sh                         1
9c5951df22c78       busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47                                   8 minutes ago       Exited              sh                         0
87d3992f84f74       nginx@sha256:d0a8828cccb73397acb0073bf34f4d7d8aa315263f1e7806bf8c55d8ac139d5f                                     8 minutes ago       Running             nginx                      0
1941fb4da154f       k8s-gcrio.azureedge.net/hyperkube-amd64@sha256:00d814b1f7763f4ab5be80c58e98140dfc69df107f253d7fdd714b30a714260a   18 hours ago        Running             kube-proxy                 0
```

لیست کانتینرهای در حال اجرا:

```shell
crictl ps
```

خروجی مشابه این است:

```
CONTAINER ID        IMAGE                                                                                                             CREATED             STATE               NAME                       ATTEMPT
1f73f2d81bf98       busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47                                   6 minutes ago       Running             sh                         1
87d3992f84f74       nginx@sha256:d0a8828cccb73397acb0073bf34f4d7d8aa315263f1e7806bf8c55d8ac139d5f                                     7 minutes ago       Running             nginx                      0
1941fb4da154f       k8s-gcrio.azureedge.net/hyperkube-amd64@sha256:00d814b1f7763f4ab5be80c58e98140dfc69df107f253d7fdd714b30a714260a   17 hours ago        Running             kube-proxy                 0
```

### اجرای دستور در یک کانتینر در حال اجرا

```shell
crictl exec -i -t 1f73f2d81bf98 ls
```

خروجی مشابه این است:

```
bin   dev   etc   home  proc  root  sys   tmp   usr   var
```

### دریافت لاگ‌های یک کانتینر

دریافت تمام لاگ‌های کانتینر:

```shell
crictl logs 87d3992f84f74
```

خروجی مشابه این است:

```
10.240.0.96 - - [06/Jun/2018:02:45:49 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.47.0" "-"
10.240.0.96 - - [06/Jun/2018:02:45:50 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.47.0" "-"
10.240.0.96 - - [06/Jun/2018:02:45:51 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.47.0" "-"
```

دریافت تنها `N` خط آخر از لاگ‌ها:

```shell
crictl logs --tail=1 87d3992f84f74
```

خروجی مشابه این است:

```
10.240.0.96 - - [06/Jun/2018:02:45:51 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.47.0" "-"
```
### اجرای یک محیط پاد

استفاده از `crictl` برای اجرای یک محیط پاد مفید است برای اشکال‌زدایی ران‌تایم‌های کانتینرها. در یک خوشه Kubernetes در حال اجرا، محیط پاد در نهایت توسط Kubelet متوقف و حذف می‌شود.

1. یک فایل JSON مشابه زیر ایجاد کنید:

   ```json
   {
     "metadata": {
       "name": "nginx-sandbox",
       "namespace": "default",
       "attempt": 1,
       "uid": "hdishd83djaidwnduwk28bcsb"
     },
     "log_directory": "/tmp",
     "linux": {
     }
   }
   ```

2. از دستور `crictl runp` برای اعمال فایل JSON و اجرای محیط پاد استفاده کنید.

   ```shell
   crictl runp pod-config.json
   ```

   شناسه محیط پاد برگردانده می‌شود.

### ایجاد یک کانتینر

استفاده از `crictl` برای ایجاد یک کانتینر مفید است برای اشکال‌زدایی ران‌تایم‌های کانتینرها. در یک خوشه Kubernetes در حال اجرا، کانتینر در نهایت توسط Kubelet متوقف و حذف می‌شود.

1. تصویر busybox را بکشید

   ```shell
   crictl pull busybox
   ```
   ```none
   Image is up to date for busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
   ```

2. پیکربندی‌ها را برای پاد و کانتینر ایجاد کنید:

   **پاد کانفیگ**:

   ```json
   {
     "metadata": {
       "name": "busybox-sandbox",
       "namespace": "default",
       "attempt": 1,
       "uid": "aewi4aeThua7ooShohbo1phoj"
     },
     "log_directory": "/tmp",
     "linux": {
     }
   }
   ```

   **کانتینر کانفیگ**:

   ```json
   {
     "metadata": {
       "name": "busybox"
     },
     "image":{
       "image": "busybox"
     },
     "command": [
       "top"
     ],
     "log_path":"busybox.log",
     "linux": {
     }
   }
   ```

3. کانتینر را ایجاد کنید، با انتقال شناسه پیش‌فرض پاد، فایل پیکربندی کانتینر و فایل پیکربندی پاد.

   ```shell
   crictl create f84dd361f8dc51518ed291fbadd6db537b0496536c1d2d6c05ff943ce8c9a54f container-config.json pod-config.json
   ```

4. لیست تمام کانتینرها را بررسی کنید و اطمینان حاصل کنید که کانتینر تازه‌ای که ایجاد شده است وضعیت `Created` را دارد.

   ```shell
   crictl ps -a
   ```

   خروجی مشابه این است:

   ```
   CONTAINER ID        IMAGE               CREATED             STATE               NAME                ATTEMPT
   3e025dd50a72d       busybox             32 seconds ago      Created             busybox             0
   ```

### شروع یک کانتینر

برای شروع یک کانتینر، شناسه آن را به `crictl start` منتقل کنید:

```shell
crictl start 3e025dd50a72d956c4f14881fbb5b1080c9275674e95fb67f965f6478a957d60
```

خروجی مشابه این است:

```
3e025dd50a72d956c4f14881fbb5b1080c9275674e95fb67f965f6478a957d60
```

بررسی کنید که کانتینر وضعیت `Running` را دارد.

```shell
crictl ps
```

خروجی مشابه این است:

```
CONTAINER ID   IMAGE    CREATED              STATE    NAME     ATTEMPT
3e025dd50a72d  busybox  About a minute ago   Running  busybox  0
```

## {{% heading "whatsnext" %}}

* [مطالعه بیشتر در مورد `crictl`](https://github.com/kubernetes-sigs/cri-tools).
* [نقشه‌برداری دستورات CLI Docker به `crictl`](/docs/reference/tools/map-crictl-dockercli/).
