---
title: اعمال استانداردهای امنیتی Pod در سطح فضاهای نام
content_type: آموزشی
weight: 20
---

{{% alert title="توجه" %}}
این آموزش فقط برای خوشه‌های جدید اعمال می‌شود.
{{% /alert %}}

Pod Security Admission یک کنترل‌گر پذیرش است که زمانی که پادها ایجاد می‌شوند،
[استانداردهای امنیتی Pod](/docs/concepts/security/pod-security-standards/)
را اعمال می‌کند. این یک ویژگی GA است در نسخه v1.25.
در این آموزش، شما استاندارد امنیتی `baseline` را یک فضای نام در هر زمان اعمال می‌کنید.

همچنین می‌توانید استانداردهای امنیتی Pod را به چندین فضای نام در یک بار در سطح خوشه اعمال کنید.
برای دستورالعمل‌ها، به
[اعمال استانداردهای امنیتی Pod در سطح خوشه](/docs/tutorials/security/cluster-level-pss/)
مراجعه کنید.

## {{% heading "پیش‌نیازها" %}}

بر روی کارگاه کاری خود نصب کنید:

- [kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- [kubectl](/docs/tasks/tools/)

## ایجاد خوشه

1. یک خوشه `kind` به این صورت ایجاد کنید:

   ```shell
   kind create cluster --name psa-ns-level
   ```

   خروجی مشابه زیر است:

   ```
   Creating cluster "psa-ns-level" ...
    ✓ Ensuring node image (kindest/node:v{{< skew currentPatchVersion >}}) 🖼
    ✓ Preparing nodes 📦
    ✓ Writing configuration 📜
    ✓ Starting control-plane 🕹️
    ✓ Installing CNI 🔌
    ✓ Installing StorageClass 💾
   Set kubectl context to "kind-psa-ns-level"
   You can now use your cluster with:

   kubectl cluster-info --context kind-psa-ns-level

   Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
   ```

1. کانتکست kubectl را به خوشه جدید تنظیم کنید

   ```shell
   kubectl cluster-info --context kind-psa-ns-level
   ```
   خروجی مشابه زیر است:

   ```
   Kubernetes control plane is running at https://127.0.0.1:50996
   CoreDNS is running at https://127.0.0.1:50996/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

   To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
   ```

## ایجاد یک namespace

یک فضای نام جدید به نام `example` ایجاد کنید:

```shell
kubectl create ns example
```

خروجی مشابه زیر است:

```
namespace/example created
```

## فعال‌سازی بررسی استانداردهای امنیتی Pod برای آن فضای نام


1. استانداردهای امنیتی Pod را بر روی این فضای نام با استفاده از برچسب‌های پشتیبانی‌شده توسط
کنترل‌گر پذیرش استاندارد Pod داخلی فعال کنید. در این مرحله، شما یک بررسی را پیکربندی
می‌کنید برای هشدار دادن به پادهایی که با آخرین نسخه استاندارد امنیتی baseline همخوانی ندارند.

   ```shell
   kubectl label --overwrite ns example \
      pod-security.kubernetes.io/warn=baseline \
      pod-security.kubernetes.io/warn-version=latest
   ```

2. می‌توانید چندین بررسی استاندارد امنیتی Pod را در هر فضای نام با استفاده از برچسب‌ها پیکربندی کنید.
دستور زیر استاندارد امنیتی baseline را enforce کرده، اما بررسی‌های warn و audit
را برای استانداردهای امنیتی restricted به عنوان آخرین نسخه (مقدار پیش‌فرض) اعمال می‌کند:

   ```shell
   kubectl label --overwrite ns example \
     pod-security.kubernetes.io/enforce=baseline \
     pod-security.kubernetes.io/enforce-version=latest \
     pod-security.kubernetes.io/warn=restricted \
     pod-security.kubernetes.io/warn-version=latest \
     pod-security.kubernetes.io/audit=restricted \
     pod-security.kubernetes.io/audit-version=latest
   ```

## تأیید اجرای استاندارد امنیتی Pod

1. یک پاد baseline در فضای نام `example` ایجاد کنید:

   ```shell
   kubectl apply -n example -f https://k8s.io/examples/security/example-baseline-pod.yaml
   ```
   پاد به درستی شروع می‌شود؛ خروجی شامل هشدار است. به عنوان مثال:

   ```
   Warning: would violate PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "nginx" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "nginx" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "nginx" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "nginx" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
   pod/nginx created
   ```

1. یک پاد baseline در فضای نام `default` ایجاد کنید:

   ```shell
   kubectl apply -n default -f https://k8s.io/examples/security/example-baseline-pod.yaml
   ```
   خروجی مشابه زیر است:

   ```
   pod/nginx created
   ```

تنظیمات اعمال استانداردهای امنیتی Pod و تنظیمات هشدار فقط برای فضای نام `example`
اعمال شد. می‌توانید همان پاد را در فضای نام `default` بدون هشدار ایجاد کنید.
## پاکسازی

حالا با اجرای دستور زیر خوشه را که بالا ایجاد کرده‌اید، حذف کنید:

```shell
kind delete cluster --name psa-ns-level
```

## {{% heading "مراحل بعدی" %}}

- یک
  [اسکریپت شل](/examples/security/kind-with-namespace-level-baseline-pod-security.sh)
  را برای انجام تمام مراحل فوق به یکدیگر اجرا کنید.

  1. خوشه kind را ایجاد کنید
  2. فضای نام جدید ایجاد کنید
  3. استاندارد امنیتی Pod `baseline` را در حالت `enforce` اعمال کنید در حالی که
     استاندارد امنیتی `restricted` را هم در حالت `warn` و `audit` اعمال کنید.
  4. یک پاد جدید با استانداردهای امنیتی Pod زیر اعمال کنید

- [Pod Security Admission](/docs/concepts/security/pod-security-admission/)
- [Pod Security Standards](/docs/concepts/security/pod-security-standards/)
- [اعمال استانداردهای امنیتی Pod در سطح خوشه](/docs/tutorials/security/cluster-level-pss/)
```

این ترجمه بر اساس فرمتی است که در داکیومنت‌های فنی استفاده می‌شود و تلاش شده است که همه جزئیات و اطلاعات مفید در هر قسمت حفظ شود.