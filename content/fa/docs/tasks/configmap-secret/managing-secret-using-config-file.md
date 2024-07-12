---
title: مدیریت اسرار با استفاده از فایل پیکربندی
content_type: task
weight: 20
description: ایجاد اشیاء Secret با استفاده از فایل پیکربندی منابع.
---

<!-- overview -->

## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}}

<!-- steps -->

## ایجاد Secret {#create-the-config-file}

شما می‌توانید شیء `Secret` را ابتدا در یک مانیفست، به صورت JSON یا YAML تعریف کرده و سپس آن شیء را ایجاد کنید.
منبع [Secret](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#secret-v1-core) شامل دو نقشه است: `data` و `stringData`.
فیلد `data` برای ذخیره داده‌های دلخواه، کدگذاری شده با استفاده از base64 استفاده می‌شود.
فیلد `stringData` برای سهولت فراهم شده است و به شما اجازه می‌دهد تا همان داده‌ها را به صورت رشته‌های کدگذاری نشده ارائه دهید.
کلیدهای `data` و `stringData` باید شامل کاراکترهای الفبایی عددی، `-`، `_` یا `.` باشند.

مثال زیر دو رشته را با استفاده از فیلد `data` در یک Secret ذخیره می‌کند.

1. رشته‌ها را به base64 تبدیل کنید:

   ```shell
   echo -n 'admin' | base64
   echo -n '1f2d1e2e67df' | base64
   ```

   {{< note >}}
   مقادیر JSON و YAML سریال شده داده‌های Secret به عنوان رشته‌های base64 کدگذاری می‌شوند. خطوط جدید در این رشته‌ها معتبر نیستند و باید حذف شوند. هنگامی که از ابزار `base64` در Darwin/macOS استفاده می‌کنید، کاربران باید از استفاده از گزینه `-b` برای تقسیم خطوط طولانی خودداری کنند. برعکس، کاربران لینوکس باید گزینه `-w 0` را به دستورات `base64` اضافه کنند یا از خط لوله `base64 | tr -d '\n'` استفاده کنند اگر گزینه `-w` در دسترس نباشد.
   {{< /note >}}

   خروجی مشابه زیر خواهد بود:

   ```
   YWRtaW4=
   MWYyZDFlMmU2N2Rm
   ```

1. مانیفست را ایجاد کنید:

   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: mysecret
   type: Opaque
   data:
     username: YWRtaW4=
     password: MWYyZDFlMmU2N2Rm
   ```

   توجه داشته باشید که نام یک شیء Secret باید یک نام دامنه زیرمجموعه DNS معتبر باشد.

1. Secret را با استفاده از [`kubectl apply`](/docs/reference/generated/kubectl/kubectl-commands#apply) ایجاد کنید:

   ```shell
   kubectl apply -f ./secret.yaml
   ```

   خروجی مشابه زیر خواهد بود:

   ```
   secret/mysecret created
   ```

برای تأیید اینکه Secret ایجاد شده است و برای دیکد کردن داده‌های Secret، به [مدیریت اسرار با استفاده از kubectl](/docs/tasks/configmap-secret/managing-secret-using-kubectl/#verify-the-secret) مراجعه کنید.

### مشخص کردن داده‌های کدگذاری نشده هنگام ایجاد یک Secret

در برخی سناریوها، ممکن است بخواهید از فیلد `stringData` استفاده کنید. این فیلد به شما اجازه می‌دهد تا یک رشته کدگذاری نشده را مستقیماً در Secret قرار دهید و رشته در هنگام ایجاد یا به‌روزرسانی Secret برای شما کدگذاری خواهد شد.

یک مثال عملی از این می‌تواند جایی باشد که شما در حال استقرار یک برنامه هستید که از Secret برای ذخیره یک فایل پیکربندی استفاده می‌کند و می‌خواهید بخش‌هایی از آن فایل پیکربندی را در طول فرآیند استقرار خود پر کنید.

به عنوان مثال، اگر برنامه شما از فایل پیکربندی زیر استفاده می‌کند:

```yaml
apiUrl: "https://my.api.com/api/v1"
username: "<user>"
password: "<password>"
```

می‌توانید این را در یک Secret با استفاده از تعریف زیر ذخیره کنید:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
stringData:
  config.yaml: |
    apiUrl: "https://my.api.com/api/v1"
    username: <user>
    password: <password>
```

{{< note >}}
فیلد `stringData` برای یک Secret با استفاده از اعمال سمت سرور به خوبی کار نمی‌کند.
{{< /note >}}

هنگامی که داده‌های Secret را بازیابی می‌کنید، فرمان مقادیر کدگذاری شده را بازمی‌گرداند، نه مقادیر متن ساده‌ای که در `stringData` ارائه داده‌اید.

برای مثال، اگر فرمان زیر را اجرا کنید:

```shell
kubectl get secret mysecret -o yaml
```

خروجی مشابه زیر خواهد بود:

```yaml
apiVersion: v1
data:
  config.yaml: YXBpVXJsOiAiaHR0cHM6Ly9teS5hcGkuY29tL2FwaS92MSIKdXNlcm5hbWU6IHt7dXNlcm5hbWV9fQpwYXNzd29yZDoge3twYXNzd29yZH19
kind: Secret
metadata:
  creationTimestamp: 2018-11-15T20:40:59Z
  name: mysecret
  namespace: default
  resourceVersion: "7225"
  uid: c280ad2e-e916-11e8-98f2-025000000001
type: Opaque
```

```markdown
### مشخص کردن همزمان `data` و `stringData`

اگر یک فیلد را هم در `data` و هم در `stringData` مشخص کنید، مقدار از `stringData` استفاده می‌شود.

به عنوان مثال، اگر شما Secret زیر را تعریف کنید:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
stringData:
  username: administrator
```

{{< note >}}
فیلد `stringData` برای یک Secret با استفاده از اعمال سمت سرور به خوبی کار نمی‌کند.
{{< /note >}}

شیء `Secret` به صورت زیر ایجاد می‌شود:

```yaml
apiVersion: v1
data:
  username: YWRtaW5pc3RyYXRvcg==
kind: Secret
metadata:
  creationTimestamp: 2018-11-15T20:46:46Z
  name: mysecret
  namespace: default
  resourceVersion: "7579"
  uid: 91460ecb-e917-11e8-98f2-025000000001
type: Opaque
```

`YWRtaW5pc3RyYXRvcg==` به `administrator` دیکد می‌شود.

## ویرایش یک Secret {#edit-secret}

برای ویرایش داده‌ها در Secretی که با استفاده از یک مانیفست ایجاد کرده‌اید، فیلد `data` یا `stringData` را در مانیفست خود تغییر دهید و فایل را به خوشه‌ی خود اعمال کنید. می‌توانید یک شیء `Secret` موجود را ویرایش کنید مگر اینکه [immutable](/docs/concepts/configuration/secret/#secret-immutable) باشد.

به عنوان مثال، اگر می‌خواهید رمز عبور را از مثال قبل به `birdsarentreal` تغییر دهید، مراحل زیر را انجام دهید:

1. رشته جدید رمز عبور را کدگذاری کنید:

   ```shell
   echo -n 'birdsarentreal' | base64
   ```

   خروجی مشابه زیر خواهد بود:

   ```
   YmlyZHNhcmVudHJlYWw=
   ```

1. فیلد `data` را با رشته جدید رمز عبور خود به روز کنید:

   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: mysecret
   type: Opaque
   data:
     username: YWRtaW4=
     password: YmlyZHNhcmVudHJlYWw=
   ```

1. مانیفست را به خوشه‌ی خود اعمال کنید:

   ```shell
   kubectl apply -f ./secret.yaml
   ```

   خروجی مشابه زیر خواهد بود:

   ```
   secret/mysecret configured
   ```

Kubernetes شیء `Secret` موجود را به روز می‌کند. به طور دقیق، ابزار `kubectl` متوجه می‌شود که یک شیء `Secret` موجود با همان نام وجود دارد. `kubectl` شیء موجود را بازیابی می‌کند، تغییرات را برنامه‌ریزی می‌کند، و شیء `Secret` تغییر یافته را به صفحه‌کنترل خوشه‌ی شما ارسال می‌کند.

اگر `kubectl apply --server-side` را مشخص کرده باشید، `kubectl` به جای این از [Server Side Apply](/docs/reference/using-api/server-side-apply/) استفاده می‌کند.

## پاک کردن

برای حذف Secretی که ایجاد کرده‌اید:

```shell
kubectl delete secret mysecret
```

## {{% heading "بعدی" %}}

- بیشتر در مورد [مفهوم Secret](/docs/concepts/configuration/secret/) بخوانید
- یاد بگیرید که چگونه از [kubectl برای مدیریت Secrets استفاده کنید](/docs/tasks/configmap-secret/managing-secret-using-kubectl/)
- یاد بگیرید که چگونه از [kustomize برای مدیریت Secrets استفاده کنید](/docs/tasks/configmap-secret/managing-secret-using-kustomize/)
