---
reviewers:
- dchen1107
- egernst
- tallclair
title: Pod Overhead
content_type: concept
weight: 30
---

<!-- مرور کلی -->

{{< feature-state for_k8s_version="v1.24" state="stable" >}}

زمانی که یک Pod را در یک Node اجرا می‌کنید، Pod خود مقداری منابع سیستم را به خود می‌گیرد. این منابع اضافی نسبت به منابع مورد نیاز برای اجرای کانتینر(ها) داخل Pod هستند. در Kubernetes، _Pod Overhead_ یک روش است برای در نظر گرفتن منابع مصرفی توسط زیرساخت Pod به غیر از درخواست‌ها و محدودیت‌های کانتینرها.

<!-- بدنه -->

در Kubernetes، overhead Pod در زمان
[ورود](/docs/reference/access-authn-authz/extensible-admission-controllers/#what-are-admission-webhooks)
به وقت انجام می‌شود بر اساس overhead مرتبط با RuntimeClass Pod.

Overhead یک Pod به عنوان مجموع درخواست‌های منبع کانتینر در هنگام برنامه‌ریزی یک Pod در نظر گرفته می‌شود. به همین ترتیب، kubelet در هنگام اندازه‌گیری cgroup Pod نیز overhead Pod را در نظر می‌گیرد و هنگام انجام امتیازدهی تخلیه Pod.

## تنظیم overhead Pod {#set-up}

شما باید مطمئن شوید که از `RuntimeClass` استفاده می‌کنید که فیلد `overhead` را تعریف کند.

## مثال استفاده

برای کار با overhead Pod، شما نیاز به یک `RuntimeClass` دارید که فیلد `overhead` را تعریف کند. به عنوان مثال، شما می‌توانید از تعریف RuntimeClass زیر با یک زمان اجرای کانتینر مجازی (در این مثال، Kata Containers به همراه نظارت ماشین مجازی Firecracker) که حدود 120MiB برای هر Pod برای ماشین مجازی و سیستم عامل مهمان استفاده کنید:

```yaml
# شما باید این مثال را برای همخوانی با نام واقعی runtime و overhead منبع در کلاستر خود تغییر دهید.
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: kata-fc
handler: kata-fc
overhead:
  podFixed:
    memory: "120Mi"
    cpu: "250m"
```

بارهای کاری که با استفاده از handler `kata-fc` از RuntimeClass ایجاد می‌شوند، برای محاسبات کوتاه منبع، برنامه‌ریزی Node و همچنین اندازه‌گیری cgroup Pod، مصرف حافظه و cpu را در نظر می‌گیرند.

در نظر بگیرید که در اجرای مثال داده شده، test-pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  runtimeClassName: kata-fc
  containers:
  - name: busybox-ctr
    image: busybox:1.28
    stdin: true
    tty: true
    resources:
      limits:
        cpu: 500m
        memory: 100Mi
  - name: nginx-ctr
    image: nginx
    resources:
      limits:
        cpu: 1500m
        memory: 100Mi
```

{{< note >}}
اگر فقط `limits` در تعریف pod مشخص شده باشد، kubelet `requests` را از این محدودیت‌ها استنتاج خواهد کرد و آن‌ها را به عنوان مقدار `limits` تنظیم خواهد کرد.
{{< /note >}}

در زمان ورود، RuntimeClass [کنترل‌کننده ورود](/docs/reference/access-authn-authz/admission-controllers/)
ویژگی‌های PodSpec بارگذاری‌شده را برای شامل شدن overhead مانند RuntimeClass اصلاح می‌کند. اگر در PodSpec این فیلد از قبل تعریف شده باشد، Pod رد می‌شود. در مثال داده شده، از آنجا که فقط نام RuntimeClass مشخص شده است، کنترل‌کننده ورود Pod را به دلیل وجود overhead اصلاح می‌کند.

پس از اینکه کنترل‌کننده ورود RuntimeClass تغییراتی ایجاد کرده است، می‌توانید مقدار overhead Pod به روز را بررسی کنید:

```bash
kubectl get pod test-pod -o jsonpath='{.spec.overhead}'
```

خروجی به شرح زیر است:

```
map[cpu:250m memory:120Mi]
```

اگر [ResourceQuota](/docs/concepts/policy/resource-quotas/) تعریف شده باشد، مجموع درخواست‌های کانتینر و فیلد `overhead` شمرده می‌شود.

زمانی که kube-scheduler تصمیم می‌گیرد که یک Pod جدید باید بر روی کدام Node اجرا شود، این scheduler overhead آن Pod را همچنین مجموع درخواست‌های کانتینر برای آن Pod در نظر می‌گیرد. برای این مثال، scheduler درخواست‌ها و overhead را جمع می‌کند، سپس به دنبال یک Node می‌گردد که دارای 2.25 CPU و 320 MiB حافظه آزاد است.

هنگامی که یک Pod به یک Node اختصاص پیدا کرده است، kubelet بر روی آن Node یک {{< glossary_tooltip
text="cgroup" term_id="cgroup" >}} جدید برای آن Pod ایجاد می‌کند. در این cgroup، runtime کانتینر زیرین کانتینر را ایجاد خواهد کرد.

اگر برای هر کانتینر محدودیتی (QoS مضمون یا QoS قابل حمل با محدودیت‌های تعریف شده) تعریف شده باشد، kubelet حداکثر بالای cgroup Pod مرتبط با آن منبع (cpu.cfs_quota_us برای CPU و memory.limit_in_bytes برای حافظه) را تنظیم خواهد کرد. این حداکثر بر اساس جمع محدودیت‌های کانتینر به علاوه `overhead` تعریف شده در PodSpec است.

برای CPU، اگر Pod دارای QoS تضمین شده یا QoS قابل حمل باشد، kubelet `cpu.shares` را بر اساس جمع درخواست‌های کانتینر به علاوه `overhead` تعریف شده در PodSpec تنظیم خواهد کرد.

نگاهی به مثال ما بیندازید، درخواست‌های کانتینر برای بارکاری را تأیید کنید:

```bash
kubectl get pod test-pod -o jsonpath='{.spec.containers[*].resources.limits}'
```

جمع درخواست‌های کلی کانتینرها 2000m CPU و 200MiB حافظه است:

```
map[cpu: 500m memory:100Mi] map[cpu:1500m memory:100Mi]
```

این را با آنچه توسط Node مشاهده شده است مقایسه کنید:

```bash
kubectl describe node | grep test-pod -B2
```

خروجی نمایش می‌دهد درخواست‌ها برای 2250m CPU و 320MiB حافظه است. درخواست‌ها شامل Pod overhead است:

```
  Namespace    Name       CPU Requests  CPU Limits   Memory Requests  Memory Limits  AGE
  ---------    ----       ------------  ----------   ---------------  -------------  ---
  default      test-pod   2250m (56%)   2250m (56%)  320Mi (1%)       320Mi (1%)     36m
```

## تأیید حدود cgroup Pod

حدود cgroup حافظه Pod را در Node که بارکاری در آن اجرا می‌شود، بررسی کنید. در مثال زیر، از [`crictl`](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md)
بر روی Node استفاده می‌شود که یک CLI برای runtime کانتینرهای سازگار با CRI فراهم می‌کند. این یک مثال پیشرفته است برای نمایش رفتار Pod overhead و منتظر نباشید که کاربران باید به طور مستقیم cgroup را بر روی Node بررسی کنند.

اول، بر روی Node مشخص، شناسه Pod را تعیین کنید:

```bash
# این را بر روی Node اجرا کنید که Pod در آن زمانبندی شده است.
POD_ID="$(sudo crictl pods --name test-pod -q)"
```

از این تبعیت می‌کند که مسیر cgroup برای Pod را تعیین کنید:

```bash
# این را بر روی Node اجرا کنید که Pod در آن زمانبندی شده است.
sudo crictl inspectp -o=json $POD_ID | grep cgroupsPath
```

مسیر cgroup نتیجه‌گیری شده شامل `pause` container Pod است. cgroup سطح Pod یک فهرست بالاتر است.

```
  "cgroupsPath": "/kubepods/podd7f4b509-cf94-4951-9417-d1087c92a5b2/7ccf55aee35dd16aca4189c952d83487297f3cd760f1bbf09620e206e7d0c27a"
```

در این مورد خاص، مسیر cgroup Pod `kubepods/podd7f4b509-cf94-4951-9417-d1087c92a5b2` است.
بررسی تنظیم سطح cgroup Pod برای حافظه:

```bash
# این را بر روی Node اجرا کنید که Pod در آن زمانبندی شده است.
# همچنین، نام cgroup را تغییر دهید تا با cgroup تخصیص داده‌شده برای pod شما مطابقت داشته باشد.
 cat /sys/fs/cgroup/memory/kubepods/podd7f4b509-cf94-4951-9417-d1087c92a5b2/memory.limit_in_bytes
```

این 320 MiB است، همانطور که انتظار می‌رود:

```
335544320
```

### مشاهده پذیری

بعضی از متریک‌های `kube_pod_overhead_*` در [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
موجود است تا به شناسایی زمانی که Pod overhead استفاده می‌شود و به مشاهده پایداری بارکاری‌های در حال اجرا با overhead تعریف شده کمک کند.

## {{% heading "whatsnext" %}}

* بیشتر در مورد [RuntimeClass](/docs/concepts/containers/runtime-class/) بیاموزید
* [طراحی PodOverhead](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/688-pod-overhead)
  پیشنهاد ارتقا برای سیاق اضافی را بخوانید
