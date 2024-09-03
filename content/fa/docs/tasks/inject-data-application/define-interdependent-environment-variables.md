---
title: "تعریف متغیرهای محیطی وابسته برای یک کانتینر"
content_type: task
weight: 20
---

<!-- overview -->

این صفحه نحوه تعریف متغیرهای محیطی وابسته برای یک کانتینر در یک Pod Kubernetes را نشان می‌دهد.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}}

<!-- steps -->

## تعریف یک متغیر محیطی وابسته برای یک کانتینر

زمانی که یک Pod ایجاد می‌کنید، می‌توانید متغیرهای محیطی وابسته را برای کانتینرهایی که در آن Pod اجرا می‌شوند، تنظیم کنید. برای تنظیم متغیرهای محیطی وابسته، می‌توانید از $(VAR_NAME) در `value` فیلد `env` در فایل پیکربندی استفاده کنید.

در این تمرین، شما یک Pod ایجاد می‌کنید که یک کانتینر اجرا می‌کند. فایل پیکربندی برای Pod یک متغیر محیطی وابسته با استفاده معمول تعریف شده را تعریف می‌کند. اینجاست که پیکربندی منظم برای Pod است:

{{% code_sample file="pods/inject/dependent-envars.yaml" %}}

1. بر اساس این پیکربندی، یک Pod ایجاد کنید:

   ```shell
   kubectl apply -f https://k8s.io/examples/pods/inject/dependent-envars.yaml
   ```

   ```
   pod/dependent-envars-demo created
   ```

2. لیست Podهای در حال اجرا را بگیرید:

   ```shell
   kubectl get pods dependent-envars-demo
   ```

   ```
   NAME                      READY     STATUS    RESTARTS   AGE
   dependent-envars-demo     1/1       Running   0          9s
   ```

3. بررسی لاگ‌ها برای کانتینری که در Pod شما اجرا می‌شود:

   ```shell
   kubectl logs pod/dependent-envars-demo
   ```

   ```
   UNCHANGED_REFERENCE=$(PROTOCOL)://172.17.0.1:80
   SERVICE_ADDRESS=https://172.17.0.1:80
   ESCAPED_REFERENCE=$(PROTOCOL)://172.17.0.1:80
   ```

   همانطور که در بالا نشان داده شده است، شما مرجع وابستگی صحیح `SERVICE_ADDRESS` و مرجع وابستگی بد `UNCHANGED_REFERENCE` و مرجع وابستگی رد شده `ESCAPED_REFERENCE` را تعریف کرده‌اید.

وقتی یک متغیر محیطی هنگام ارجاع داده شد از قبل تعریف شود، ارجاع می‌تواند به درستی حل شود، مانند مورد `SERVICE_ADDRESS`.

توجه داشته باشید که ترتیب در لیست `env` مهم است. یک متغیر محیطی در نظر گرفته نمی‌شود "تعریف شده" اگر در پایین‌ترین لیست مشخص شود. به همین دلیل، `UNCHANGED_REFERENCE` ناتوان در حل `$(PROTOCOL)` در مثال بالا است.

وقتی متغیر محیطی تعریف نشده باشد یا فقط برخی از متغیرها وجود داشته باشند، متغیر محیطی تعریف نشده به عنوان یک رشته معمولی در نظر گرفته می‌شود، مانند `UNCHANGED_REFERENCE`. توجه داشته باشید که متغیرهای محیطی بدرستی تجزیه نمی‌شوند، به طور کلی از آن می‌توان در مسیر جلوگیری از آغاز کانتینر استفاده کرد.

در این حالت، ترکیب `$(VAR_NAME)` می‌تواند با دو دولار `$`، به عنوان `ESCAPED_REFERENCE` فرار شود. ارجاع هرگز گسترش نمی‌یابد، که از اینکه متغیر مشخص شده باشد یا نه، خودش را مشاهده می‌کند.

## {{% heading "whatsnext" %}}

* در مورد [متغیرهای محیطی](/docs/tasks/inject-data-application/environment-variable-expose-pod-information/) بیشتر بدانید.
* [EnvVarSource](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#envvarsource-v1-core) را ببینید.
