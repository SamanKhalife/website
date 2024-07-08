---
title: اشکال‌زدایی نود‌های Kubernetes با استفاده از Kubectl
content_type: task
min-kubernetes-server-version: 1.20
---

<!-- overview -->
این صفحه نحوه اشکال‌زدایی یک [نود](/docs/concepts/architecture/nodes/)
در حال اجرا در خوشه Kubernetes با استفاده از دستور `kubectl debug` را نمایش می‌دهد.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

برای انجام این کار، باید دسترسی به ایجاد پادها و اختصاص آن‌ها به نودهای دلخواه را داشته باشید.
همچنین باید مجوز داشته باشید تا پادهایی که به سیستم‌های فایل میزبان دسترسی دارند را ایجاد کنید.

<!-- steps -->

## اشکال‌زدایی یک نود با استفاده از `kubectl debug node`

از دستور `kubectl debug node` برای استقرار یک پاد به یک نود که می‌خواهید مشکل آن را اشکال‌زدایی کنید، استفاده کنید.
این دستور در حالت‌هایی کاربرد دارد که نمی‌توانید به Node خود از طریق اتصال SSH دسترسی پیدا کنید.
وقتی که پاد ایجاد می‌شود، پاد یک شل تعاملی روی نود باز می‌کند.
برای ایجاد یک شل تعاملی روی یک نود با نام "mynode"، دستور زیر را اجرا کنید:

```shell
kubectl debug node/mynode -it --image=ubuntu
```

```console
Creating debugging pod node-debugger-mynode-pdx84 with container debugger on node mynode.
If you don't see a command prompt, try pressing enter.
root@mynode:/#
```

دستور اشکال‌زدایی به جمع‌آوری اطلاعات و رفع مشکلات کمک می‌کند. دستوراتی که ممکن است استفاده کنید شامل `ip`، `ifconfig`، `nc`، `ping` و `ps` و غیره می‌شود. همچنین می‌توانید ابزارهای دیگری مانند `mtr`، `tcpdump` و `curl` را از مدیر بسته مربوطه نصب کنید.

{{< note >}}

دستورات اشکال‌زدایی ممکن است بسته به تصویری که پاد اشکال‌زدایی از آن استفاده می‌کند، متفاوت باشد و ممکن است نیاز به نصب برخی دستورات داشته باشد.

{{< /note >}}

پاد اشکال‌زدایی به فایل سیستم ریشه‌ی Node در `/host` درون پاد دسترسی دارد.
اگر kubelet خود را در یک فضای نام فایل سیستم اجرا کنید، پاد اشکال‌زدایی فایل ریشه برای آن فضای نام را می‌بیند نه برای کل Node. برای یک نود لینوکس معمولی، می‌توانید به مسیرهای زیر برای پیدا کردن لاگ‌های مربوطه نگاه کنید:

`/host/var/log/kubelet.log`
: لاگ‌های kubelet که مسئول اجرای کانتینرها بر روی نود هستند.

`/host/var/log/kube-proxy.log`
: لاگ‌های kube-proxy که مسئول هدایت ترافیک به نقاط پایانی سرویس هستند.

`/host/var/log/containerd.log`
: لاگ‌های پردازه containerd که در نود اجرا می‌شود.

`/host/var/log/syslog`
: نمایش پیام‌ها و اطلاعات عمومی مربوط به سیستم.

`/host/var/log/kern.log`
: نمایش لاگ‌های کرنل.

در زمان ایجاد یک جلسه اشکال‌زدایی بر روی یک Node، به یاد داشته باشید که:

* `kubectl debug` به طور خودکار نام جدید پاد را بر اساس نام نود تولید می‌کند.
* فایل سیستم ریشه Node به `/host` مانت می‌شود.
* هرچند که کانتینر در فضای IPC، شبکه و PID میزبان اجرا می‌شود، اما پاد مجوز خصوصی ندارد. این به این معنی است که ممکن است برخی از اطلاعات فرآیند را نتوان خواند چون دسترسی به آن اطلاعات تنها برای کاربران ادمین محدود شده است. به عنوان مثال، دستور `chroot /host` شکست خواهد خورد.
اگر نیاز به پاد با مجوز دارید، آن را به صورت دستی ایجاد کنید.

## {{% heading "cleanup" %}}

وقتی از پاد اشکال‌زدایی استفاده کردید و کار تمام شد، آن را حذف کنید:

```shell
kubectl get pods
```

```none
NAME                          READY   STATUS       RESTARTS   AGE
node-debugger-mynode-pdx84    0/1     Completed    0          8m1s
```

```shell
# نام پاد را به تعویض کنید
kubectl delete pod node-debugger-mynode-pdx84 --now
```

```none
pod "node-debugger-mynode-pdx84" deleted
```

{{< note >}}

دستور `kubectl debug node` کار نمی‌کند اگر Node خاموش باشد (قطع اتصال از شبکه، یا kubelet متوقف شود و دوباره راه‌اندازی نشود و غیره).
در این صورت [اشکال‌زدایی یک Node خاموش/ناقص را](/docs/tasks/debug/debug-cluster/#example-debugging-a-down-unreachable-node)
بررسی کنید.

{{< /note >}}
