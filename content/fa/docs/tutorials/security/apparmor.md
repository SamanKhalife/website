---
reviewers:
- stclair
title: "محدود کردن دسترسی یک کانتینر به منابع با AppArmor"
content_type: آموزش
weight: 30
---

<!-- overview -->

{{< feature-state feature_gate_name="AppArmor" >}}

این صفحه به شما نحوه بارگذاری پروفایل‌های AppArmor در نودهای خود نشان می‌دهد و نیز به شما کمک می‌کند که این پروفایل‌ها را در Podها اعمال کنید. برای اطلاعات بیشتر درباره اینکه Kubernetes چگونه می‌تواند Podها را با استفاده از AppArmor محدود کند، به [محدودیت‌های امنیتی هسته لینوکس برای Podها و کانتینرها](/docs/concepts/security/linux-kernel-security-constraints/#apparmor) مراجعه کنید.

## {{% heading "اهداف" %}}

* مشاهده یک مثال از بارگذاری یک پروفایل بر روی یک Node
* یادگیری نحوه اعمال پروفایل بر روی یک Pod
* یادگیری نحوه بررسی اینکه پروفایل بارگذاری شده است یا خیر
* مشاهده اتفاقاتی که پس از نقض پروفایل اتفاق می‌افتد
* مشاهده اتفاقاتی که پس از عدم بارگذاری پروفایل اتفاق می‌افتد

## {{% heading "پیش‌نیازها" %}}

AppArmor یک ماژول اختیاری هسته و ویژگی Kubernetes است، بنابراین اطمینان حاصل کنید که قبل از ادامه، این ویژگی بر روی نودهای شما پشتیبانی می‌شود:

1. ماژول هسته AppArmor فعال باشد - برای اجرای یک پروفایل AppArmor، ماژول هسته AppArmor باید نصب و فعال شده باشد. برخی توزیع‌ها، مانند Ubuntu و SUSE، این ماژول را به طور پیش‌فرض فعال می‌کنند و بسیاری از توزیع‌های دیگر پشتیبانی اختیاری ارائه می‌دهند. برای بررسی اینکه آیا ماژول فعال است یا خیر، می‌توانید فایل `/sys/module/apparmor/parameters/enabled` را بررسی کنید:

   ```shell
   cat /sys/module/apparmor/parameters/enabled
   Y
   ```

   kubelet قبل از اجازه دادن به یک Pod که به طور صریح با AppArmor پیکربندی شده است، اطمینان حاصل می‌کند که AppArmor بر روی میزبان فعال است.

2. رانتایم کانتینر AppArmor را پشتیبانی می‌کند - تمام رانتایم‌های معمول کانتینری که توسط Kubernetes پشتیبانی می‌شوند، باید AppArmor را پشتیبانی کنند، از جمله {{< glossary_tooltip term_id="containerd" >}} و {{< glossary_tooltip term_id="cri-o" >}}. لطفا به مستندات رانتایم مربوطه مراجعه کنید و اطمینان حاصل کنید که خوشه از نیازها برای استفاده از AppArmor پشتیبانی می‌کند.

3. پروفایل بارگذاری شده است - AppArmor با تعیین یک پروفایل AppArmor که هر کانتینر باید با آن اجرا شود، به یک Pod اعمال می‌شود. اگر هر یک از پروفایل‌های مشخص شده در هسته بارگذاری نشده باشد، kubelet این Pod را رد می‌کند. می‌توانید پروفایل‌های بارگذاری شده را در یک Node با بررسی فایل `/sys/kernel/security/apparmor/profiles` بررسی کنید. به عنوان مثال:

   ```shell
   ssh gke-test-default-pool-239f5d02-gyn2 "sudo cat /sys/kernel/security/apparmor/profiles | sort"
   ```
   ```
   apparmor-test-deny-write (enforce)
   apparmor-test-audit-write (enforce)
   docker-default (enforce)
   k8s-nginx (enforce)
   ```

   برای جزئیات بیشتر درباره بارگذاری پروفایل‌ها در نودها، به [راه‌اندازی نودها با پروفایل‌ها](#setting-up-nodes-with-profiles) مراجعه کنید.

<!-- lessoncontent -->

## امنیت یک Pod

{{< note >}}
پیش از Kubernetes v1.30، AppArmor از طریق annotations مشخص می‌شد. برای مشاهده مستندات با این API منسوخ، از انتخاب‌گر نسخه مستندات استفاده کنید.
{{< /note >}}

پروفایل‌های AppArmor می‌توانند در سطح Pod یا سطح کانتینر مشخص شوند. پروفایل AppArmor کانتینر اولویت بالاتری نسبت به پروفایل Pod دارد.

```yaml
securityContext:
  appArmorProfile:
    type: <نوع_پروفایل>
```

که `<نوع_پروفایل>` یکی از این‌ها است:

* `RuntimeDefault` برای استفاده از پروفایل پیش‌فرض رانتایم
* `Localhost` برای استفاده از یک پروفایل بارگذاری شده در میزبان (راهنمایی زیر)
* `Unconfined` برای اجرا بدون AppArmor

برای جزئیات کامل درباره API پروفایل AppArmor، به [مرجع API](#api-reference) مراجعه کنید.

برای بررسی اینکه پروفایل اعمال شده است، می‌توانید بررسی کنید که فرآیند root کانتینر با پروفایل درست در حال اجرا است با بررسی proc attr آن:

```shell
kubectl exec <نام_پاد> -- cat /proc/1/attr/current
```

خروجی باید به شکل زیر باشد:

```
cri-containerd.apparmor.d (enforce)
```

## مثال

*این مثال فرض می‌کند که

 شما پیش‌فرض یک خوشه را با پشتیبانی AppArmor تنظیم کرده‌اید.*

ابتدا، پروفایلی که می‌خواهید بر روی نودهای خود بارگذاری کنید را بارگذاری کنید. این پروفایل تمام عملیات نوشتن فایل را مسدود می‌کند:

```
#include <tunables/global>

profile k8s-apparmor-example-deny-write flags=(attach_disconnected) {
  #include <abstractions/base>

  file,

  # مسدود کردن همه عملیات نوشتن فایل.
  deny /** w,
}
```

این پروفایل باید بر روی تمام نودها بارگذاری شود، زیرا شما نمی‌دانید Pod کجا زمانبندی می‌شود. برای این مثال می‌توانید از SSH برای نصب پروفایل‌ها استفاده کنید، اما رویکردهای دیگری در [راه‌اندازی نودها با پروفایل‌ها](#setting-up-nodes-with-profiles) بحث شده‌اند.

```shell
# این مثال فرض می‌کند که نام نودها با نام‌های میزبان همخوانی دارد و قابل دسترسی از طریق SSH هستند.
NODES=($(kubectl get nodes -o name))

for NODE in ${NODES[*]}; do ssh $NODE 'sudo apparmor_parser -q <<EOF
#include <tunables/global>

profile k8s-apparmor-example-deny-write flags=(attach_disconnected) {
  #include <abstractions/base>

  file,

  # مسدود کردن همه عملیات نوشتن فایل.
  deny /** w,
}
EOF'
done
```

سپس، یک Pod ساده با نام "Hello AppArmor" با پروفایل deny-write اجرا کنید:

{{% code_sample file="pods/security/hello-apparmor.yaml" %}}

```shell
kubectl create -f hello-apparmor.yaml
```

می‌توانید بررسی کنید که کانتینر واقعاً با آن پروفایل در حال اجرا است با بررسی `/proc/1/attr/current`:

```shell
kubectl exec hello-apparmor -- cat /proc/1/attr/current
```

خروجی باید به شکل زیر باشد:
```
k8s-apparmor-example-deny-write (enforce)
```

سرانجام، می‌توانید ببینید چه اتفاقی می‌افتد اگر پروفایل را نقض کرده و به یک فایل نوشته شود:

```shell
kubectl exec hello-apparmor -- touch /tmp/test
```
```
touch: /tmp/test: Permission denied
error: error executing remote command: command terminated with non-zero exit code: Error executing in Docker Container: 1
```

برای پایان دادن به این موضوع، ببینید چه اتفاقی می‌افتد اگر سعی کنید پروفایلی را مشخص کنید که بارگذاری نشده است:

```shell
kubectl create -f /dev/stdin <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: hello-apparmor-2
spec:
  securityContext:
    appArmorProfile:
      type: Localhost
      localhostProfile: k8s-apparmor-example-allow-write
  containers:
  - name: hello
    image: busybox:1.28
    command: [ "sh", "-c", "echo 'Hello AppArmor!' && sleep 1h" ]
EOF
```
```
pod/hello-apparmor-2 created
```

اگرچه Pod با موفقیت ایجاد شد، اما بررسی‌های بیشتر نشان می‌دهد که در حالت انتظار است:

```shell
kubectl describe pod hello-apparmor-2
```
```
Name:          hello-apparmor-2
Namespace:     default
Node:          gke-test-default-pool-239f5d02-x1kf/10.128.0.27
Start Time:    Tue, 30 Aug 2016 17:58:56 -0700
Labels:        <none>
Annotations:   container.apparmor.security.beta.kubernetes.io/hello=localhost/k8s-apparmor-example-allow-write
Status:        Pending
...
Events:
  Type     Reason     Age              From               Message
  ----     ------     ----             ----               -------
  Normal   Scheduled  10s              default-scheduler  Successfully assigned default/hello-apparmor to gke-test-default-pool-239f5d02-x1kf
  Normal   Pulled     8s               kubelet            Successfully pulled image "busybox:1.28" in 370.157088ms (370.172701ms including waiting)
  Normal   Pulling    7s (x2 over 9s)  kubelet            Pulling image "busybox:1.28"
  Warning  Failed     7s (x2 over 8s)  kubelet            Error: failed to get container spec opts: failed to generate apparmor spec opts: apparmor profile not found k8s-apparmor-example-allow-write
  Normal   Pulled     7s               kubelet            Successfully pulled image "busybox:1.28" in 90.980331ms (91.005869ms including waiting)
```

یک رویداد پیام خطا را با دلیل ارائه می‌دهد، واژه‌های خاص به وابستگی می‌تواند تغییر کند:
```
  Warning  Failed     7s (x2 over 8s)  kubelet            Error: failed to get container spec opts: failed to generate apparmor spec opts: apparmor profile not found
```

## مدیریت

### راه‌اندازی نودها با پروفایل‌ها

Kubernetes {{< skew currentVersion >}} هیچ مکانیزم داخلی برای بارگذاری پروفایل‌های AppArmor بر روی نودها ارائه نمی‌دهد. پروفایل‌ها می‌توانند از طریق زیرساخت سفارشی یا ابزارهایی مانند [اپراتور پروفایل‌های امنیتی Kubernetes](https://github.com/kubernetes-sigs/security-profiles-operator) بارگذاری شوند.

برنامه‌ریز نمی‌داند که پروفایل‌هایی که بر روی کدام نود بارگذاری شده‌اند، بنابراین مجموعه کامل پروفایل‌ها باید بر روی هر نود بارگذاری شود. یک رویکرد جایگزین این است که برای هر پروفایل (یا کلاس پروفایل‌ها) برچسب Node اضافه کنید و از یک [انتخاب‌گر Node](/docs/concepts/scheduling-eviction/assign-pod-node/) برای اطمینان از اینکه Pod بر روی یک Node با پروفایل مورد نیاز اجرا می‌شود، استفاده کنید.

## نویسندگان پروفایل‌ها

مشخص کردن پروفایل‌های AppArmor به درستی می‌تواند کاری پیچیده باشد. خوشبختانه برخی اب

زارها برای کمک به این کار وجود دارند:

* `aa-genprof` و `aa-logprof` به وسیلهٔ نظارت بر فعالیت برنامه و لاگ‌گیری، قوانین پروفایل تولید می‌کنند و اقداماتی که انجام می‌دهد را تأیید می‌کنند. دستورالعمل‌های بیشتر توسط
  [مستندات AppArmor](https://gitlab.com/apparmor/apparmor/wikis/Profiling_with_tools) ارائه شده‌اند.
* [bane](https://github.com/jfrazelle/bane) یک مولد پروفایل AppArmor برای Docker است که از یک زبان پروفایل ساده استفاده می‌کند.

برای رفع اشکال مشکلات با AppArmor، می‌توانید لاگ‌های سیستم را بررسی کنید تا ببینید که چه چیزی به صورت دقیقی رد شده است. AppArmor پیام‌های متنی را به dmesg وارد می‌کند و خطاها معمولاً در لاگ‌های سیستم یا از طریق journalctl قابل پیدا شدن هستند. اطلاعات بیشتر در
[AppArmor failures](https://gitlab.com/apparmor/apparmor/wikis/AppArmor_Failures) ارائه شده‌اند.

## مشخصات محدود کردن AppArmor

{{< caution >}}
قبل از Kubernetes v1.30، AppArmor از طریق حاشیه‌نویسی‌ها مشخص شده بود. از انتخاب نسخه مستندات برای مشاهده مستندات با این API منسوخ شده استفاده کنید.
{{< /caution >}}

### AppArmor profile در context امنیتی {#appArmorProfile}

می‌توانید `appArmorProfile` را در `securityContext` یا `securityContext` Pod مشخص کنید. اگر پروفایل در سطح پاد (مانند زنجیره‌ای، sidecar، و container‌های ephemeral) تنظیم شود، به عنوان پروفایل پیش‌فرض برای تمام کانتینرها در پاد استفاده می‌شود. اگر پروفایل‌های pod و container تعیین شوند، پروفایل container استفاده خواهد شد.

یک پروفایل AppArmor دارای 2 فیلد است:

`type` _(الزامی)_ - نشان می‌دهد که کدام نوع پروفایل AppArmor اعمال می‌شود. گزینه‌های معتبر شامل موارد زیر هستند:

`Localhost`
: یک پروفایل قبلی بر روی نود (تعیین شده توسط `localhostProfile`).

`RuntimeDefault`
: پروفایل پیش‌فرض اجرایی کانتینر.

`Unconfined`
: عدم اجرای AppArmor.

`localhostProfile` - نام یک پروفایل بارگذاری شده روی نود که باید استفاده شود.
این گزینه باید فقط ارائه شود اگر و تنها اگر نوع `Localhost` است.

## {{% heading "whatsnext" %}}


منابع اضافی:

* [راهنمای سریع به زبان پروفایل AppArmor](https://gitlab.com/apparmor/apparmor/wikis/QuickProfileLanguage)
* [مرجع سیاست اصلی AppArmor](https://gitlab.com/apparmor/apparmor/wikis/Policy_Layout)

