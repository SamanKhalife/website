---
title: "تعریف متغیرهای محیطی برای یک کانتینر"
content_type: task
weight: 20
---

<!-- overview -->

این صفحه نحوه تعریف متغیرهای محیطی برای یک کانتینر در یک Pod Kubernetes را نشان می‌دهد.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}}

<!-- steps -->

## تعریف یک متغیر محیطی برای یک کانتینر

زمانی که یک Pod ایجاد می‌کنید، می‌توانید متغیرهای محیطی را برای کانتینرهایی که در Pod اجرا می‌شوند، تنظیم کنید. برای تنظیم متغیرهای محیطی، فیلد `env` یا `envFrom` را در فایل پیکربندی قرار دهید.

فیلدهای `env` و `envFrom` اثرات مختلفی دارند.

`env`
: به شما امکان می‌دهد متغیرهای محیطی را برای یک کانتینر تنظیم کنید، با مشخص کردن مستقیم مقدار برای هر متغیر که نام آن را می‌دهید.

`envFrom`
: به شما امکان می‌دهد متغیرهای محیطی را برای یک کانتینر با ارجاع به ConfigMap یا Secret تنظیم کنید. هنگامی که از `envFrom` استفاده می‌کنید، تمام جفت‌های کلید-مقدار در ConfigMap یا Secret مرجع شده به عنوان متغیرهای محیطی برای کانتینر تنظیم می‌شوند. همچنین می‌توانید یک پیشوند مشترک رشته را مشخص کنید.

می‌توانید بیشتر در مورد [ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables) و [Secret](/docs/tasks/inject-data-application/distribute-credentials-secure/#configure-all-key-value-pairs-in-a-secret-as-container-environment-variables) بخوانید.

این صفحه نحوه استفاده از `env` را توضیح می‌دهد.

در این تمرین، شما یک Pod ایجاد می‌کنید که یک کانتینر اجرا می‌کند. فایل پیکربندی برای Pod یک متغیر محیطی با نام `DEMO_GREETING` و مقدار `"Hello from the environment"` را تعریف می‌کند. اینجاست که پیکربندی منظم برای Pod است:

{{% code_sample file="pods/inject/envars.yaml" %}}

1. بر اساس این پیکربندی، یک Pod ایجاد کنید:

   ```shell
   kubectl apply -f https://k8s.io/examples/pods/inject/envars.yaml
   ```

1. لیست Podهای در حال اجرا را بگیرید:

   ```shell
   kubectl get pods -l purpose=demonstrate-envars
   ```

   خروجی مشابه زیر است:

   ```
   NAME            READY     STATUS    RESTARTS   AGE
   envar-demo      1/1       Running   0          9s
   ```

1. لیست متغیرهای محیطی کانتینر Pod:

   ```shell
   kubectl exec envar-demo -- printenv
   ```

   خروجی مشابه این است:

   ```
   NODE_VERSION=4.4.2
   EXAMPLE_SERVICE_PORT_8080_TCP_ADDR=10.3.245.237
   HOSTNAME=envar-demo
   ...
   DEMO_GREETING=Hello from the environment
   DEMO_FAREWELL=Such a sweet sorrow
   ```

{{< note >}}
متغیرهای محیطی تنظیم شده با استفاده از فیلد `env` یا `envFrom`، هر متغیر محیطی مشخص شده در تصویر کانتینر را بازنویسی می‌کنند.
{{< /note >}}

{{< note >}}
متغیرهای محیطی ممکن است به یکدیگر ارجاع دهند، اما ترتیب این موارد مهم است.
متغیرهایی که از دیگران تعریف شده‌اند در همان محیط، باید بعد از آن‌ها تعریف شوند. به طور مشابه، از ارجاع‌های مدور اجتناب کنید.
{{< /note >}}

## استفاده از متغیرهای محیطی در داخل تنظیمات شما

متغیرهای محیطی که در تنظیمات یک Pod زیر `.spec.containers[*].env[*]` تعریف می‌کنید، می‌توانند در جاهای دیگری از تنظیمات مورد استفاده قرار گیرند، به عنوان مثال در دستورات و آرگومان‌هایی که برای کانتینرهای Pod تعیین می‌کنید. در پیکربندی زیر به عنوان مثال، متغیرهای محیطی `GREETING`، `HONORIFIC` و `NAME` به ترتیب به مقادیر `Warm greetings to`، `The Most Honorable` و `Kubernetes` تنظیم می‌شوند. متغیر محیطی `MESSAGE` مجموعه این متغیرهای محیطی را ترکیب می‌کند و سپس آن را به عنوان آرگومان CLI به کانتینر `env-print-demo` منتقل می‌کند.

نام متغیرهای محیطی شامل حروف، اعداد، زیرخط، نقطه یا خط تیره است، اما اولین حرف نمی‌تواند یک عدد باشد.
اگر گیت فیچر `RelaxedEnvironmentVariableValidation` فعال باشد، می‌توان از تمام [کاراکترهای ASCII قابل چاپ](https://www.ascii-code.com/characters/printable-characters) به جز "=" برای نام متغیرهای محیطی استفاده کرد.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: print-greeting
spec:
  containers:
  - name: env-print-demo
    image: bash
    env:
    - name: GREETING
      value: "Warm greetings to"
    - name: HONORIFIC
      value: "The Most Honorable"
    - name: NAME
      value: "Kubernetes"
    - name: MESSAGE
      value: "$(GREETING) $(HONORIFIC) $(NAME)"
    command: ["echo"]
    args: ["$(MESSAGE

)"]
```

پس از ایجاد، دستور `echo Warm greetings to The Most Honorable Kubernetes` بر روی کانتینر اجرا می‌شود.

## {{% heading "whatsnext" %}}

* بیشتر در مورد [متغیرهای محیطی](/docs/tasks/inject-data-application/environment-variable-expose-pod-information/) بخوانید.
* در مورد [استفاده از Secrets به عنوان متغیرهای محیطی](/docs/concepts/configuration/secret/#using-secrets-as-environment-variables) بیشتر بدانید.
* [EnvVarSource](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#envvarsource-v1-core) را ببینید.
