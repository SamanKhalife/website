---
title: مدیریت Secrets با استفاده از ابزار خط فرمان kubectl
content_type: task
weight: 10
description: ایجاد اشیاء Secret با استفاده از دستور خط فرمان kubectl.
---

<!-- overview -->

این صفحه به شما نشان می‌دهد که چگونه از ابزار خط فرمان `kubectl` برای ایجاد، ویرایش، مدیریت و حذف Kubernetes {{<glossary_tooltip text="Secrets" term_id="secret">}} استفاده کنید.

## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}}

<!-- steps -->

## ایجاد یک Secret

یک شیء `Secret` اطلاعات حساسی مانند اعتبارهایی که Pods برای دسترسی به خدمات استفاده می‌کنند را ذخیره می‌کند. به عنوان مثال، ممکن است برای ذخیره نام کاربری و رمز عبور مورد نیاز برای دسترسی به پایگاه داده، نیاز به یک Secret داشته باشید.

می‌توانید Secret را با ارسال داده‌های خام در دستور، یا با ذخیره اعتبارها در فایل‌ها و ارسال آن‌ها در دستور ایجاد کنید. دستورات زیر یک Secret را ایجاد می‌کنند که نام کاربری `admin` و رمز عبور `S!B*d$zDsb=` را ذخیره می‌کند.

### استفاده از داده‌های خام

دستور زیر را اجرا کنید:

```shell
kubectl create secret generic db-user-pass \
    --from-literal=username=admin \
    --from-literal=password='S!B*d$zDsb='
```

شما باید از نقل قول تکی `''` برای از بین بردن کاراکترهای خاص مانند `$`، `\`، `*`، `=` و `!` در رشته‌های خود استفاده کنید. اگر این کار را نکنید، پوسته‌ی خود رشته‌های این کاراکترها را تفسیر می‌کند.

{{< note >}}
فیلد `stringData` برای یک Secret با استفاده از اعمال سمت سرور به خوبی کار نمی‌کند.
{{< /note >}}

### استفاده از فایل‌های منبع

1. اعتبارها را در فایل‌ها ذخیره کنید:

   ```shell
   echo -n 'admin' > ./username.txt
   echo -n 'S!B*d$zDsb=' > ./password.txt
   ```

   پرچم `-n` اطمینان حاصل می‌کند که فایل‌های تولید شده دارای کاراکتر اضافی newline در انتهای متن نباشند. این اهمیت دارد زیرا وقتی `kubectl` یک فایل را می‌خواند و محتوا را به رشته‌ای base64 تبدیل می‌کند، کاراکتر اضافی newline نیز کدگذاری می‌شود. شما نیازی به از بین بردن کاراکترهای خاص در رشته‌هایی که در فایل‌ها درج می‌کنید ندارید.

1. مسیرهای فایل را در دستور `kubectl` ارسال کنید:

   ```shell
   kubectl create secret generic db-user-pass \
       --from-file=./username.txt \
       --from-file=./password.txt
   ```

   نام کلید پیش‌فرض نام فایل است. شما می‌توانید به‌طور اختیاری نام کلید را با استفاده از `--from-file=[key=]source` تنظیم کنید. به عنوان مثال:

   ```shell
   kubectl create secret generic db-user-pass \
       --from-file=username=./username.txt \
       --from-file=password=./password.txt
   ```

با هر یک از این روش‌ها، خروجی مشابه زیر خواهد بود:

```
secret/db-user-pass created
```

### تایید Secret {#verify-the-secret}

بررسی کنید که Secret ایجاد شده است:

```shell
kubectl get secrets
```

خروجی مشابه زیر خواهد بود:

```
NAME              TYPE       DATA      AGE
db-user-pass      Opaque     2         51s
```

جزئیات Secret را مشاهده کنید:

```shell
kubectl describe secret db-user-pass
```

خروجی مشابه زیر خواهد بود:

```
Name:            db-user-pass
Namespace:       default
Labels:          <none>
Annotations:     <none>

Type:            Opaque

Data
====
password:    12 bytes
username:    5 bytes
```

دستورات `kubectl get` و `kubectl describe` به طور پیش‌فرض از نمایش محتویات یک `Secret` خودداری می‌کنند. این اقدام برای جلوگیری از اتفاقی شدن نمایش دادن `Secret` به صورت اتفاقی، یا ذخیره شدن در یک لاگ ترمینال است.

### رمزگشایی Secret {#decoding-secret}

1. محتوای Secret را که ایجاد کرده‌اید بررسی کنید:

   ```shell
   kubectl get secret db-user-pass -o jsonpath='{.data}'
   ```

   خروجی مشابه زیر خواهد بود:

   ```json
   { "password": "UyFCXCpkJHpEc2I9", "username": "YWRtaW4=" }
   ```

1. داده `password` را رمزگشایی کنید:

   ```shell
   echo 'UyFCXCpkJHpEc2I9' | base64 --decode
   ```

   خروجی مشابه زیر خواهد بود:

   ```
   S!B*d$zDsb=
   ```

   {{< caution >}}
   این یک مثال برای اهداف مستندسازی است. در عمل، این روش ممکن است باعث ذخیره دستور با داده‌های رمزگذاری شده در تاریخچه پوسته شما شود. هر کسی که دسترسی به کامپیوتر شما دارد می‌تواند دستور را پیدا کرده و رمز را رمزگشایی کند. روش بهتر این است که دستورهای بررسی و رمزگشایی را ترکیب کنید.
   {{< /caution >}}

   ```shell
   kubectl get secret db-user-pass -o jsonpath='{.data.password}' | base64 --decode
   ```

## ویرایش یک Secret {#edit-secret}

می‌توانید یک شیء `Secret` موجود را ویرایش کنید مگر اینکه [غیرقابل تغییر](/docs/concepts/configuration/secret/#secret-immutable) باشد. برای ویرایش یک Secret، دستور زیر را اجرا کنید:

```shell
kubectl edit secrets <نام-secret>
```

این دستور ویرایشگر پیش‌فرض شما را باز می‌کند و به شما امکان می‌دهد تا مقادیر رمزگذاری شده base64 در فیلد `data` را به‌روز کنید، مانند مثال زیر:

```yaml
# لطفاً اشیاء زیر را ویرایش کنید. خطوطی که با '#' شروع می‌شوند، نادیده گرفته خواهند شد،
# و یک فایل خالی ویرایش را متوقف خواهد کرد. اگر هنگام ذخیره این فایل خطا رخ دهد، آن با خطاهای مربوطه
# مجدداً باز خواهد شد.
#
apiVersion: v1
data:
  password: UyFCXCpkJHpEc2I9
  username: YWRtaW4=
kind: Secret
metadata:
  creationTimestamp: "2022-06-28T17:44:13Z"
  name: db-user-pass
  namespace: default
  resourceVersion: "12708504"
  uid: 91becd59-78fa-4c85-823f-6d44436242ac
type: Opaque
```

## پاک کردن

برای حذف یک Secret، دستور زیر را اجرا کنید:

```shell
kubectl delete secret db-user-pass
```

## {{% heading "whatsnext" %}}

- بیشتر در مورد [مفهوم Secret](/docs/concepts/configuration/secret/) بخوانید
- یاد بگیرید که چگونه [Secrets را با استفاده از فایل پیکربندی مدیریت کنید](/docs/tasks/configmap-secret/managing-secret-using-config-file/)
- یاد بگیرید که چگونه [Secrets را با استفاده از kustomize مدیریت کنید](/docs/tasks/configmap-secret/managing-secret-using-kustomize/)
