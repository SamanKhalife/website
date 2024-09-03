### مدیریت Secrets با استفاده از Kustomize

`kubectl` از ابزار مدیریت اشیاء [Kustomize](/docs/tasks/manage-kubernetes-objects/kustomization/) پشتیبانی می‌کند تا Secrets و ConfigMaps را مدیریت کنید. شما می‌توانید با استفاده از Kustomize یک *مولد منابع* ایجاد کنید که یک Secret را تولید کرده و با استفاده از `kubectl` به سرور API اعمال کنید.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}}

<!-- steps -->

## ایجاد یک Secret

شما می‌توانید با تعریف `secretGenerator` در فایل `kustomization.yaml` به منابع دیگری مانند فایل‌های `.env` یا مقادیر حرفه‌ای ارجاع دهید. به عنوان مثال، دستورات زیر یک فایل Kustomization برای نام کاربری `admin` و رمز عبور `1f2d1e2e67df` ایجاد می‌کنند.

{{< note >}}
فیلد `stringData` برای یک Secret با استفاده از کاربر سمت سرور خوب کار نمی‌کند.
{{< /note >}}

### ایجاد فایل Kustomization

{{< tabs name="داده‌های Secret" >}}
{{< tab name="حروف اصلی" codelang="yaml" >}}
secretGenerator:
- name: database-creds
  literals:
  - username=admin
  - password=1f2d1e2e67df
{{< /tab >}}
{{% tab name="فایل‌ها" %}}
1.  اطلاعات احراز هویت را در فایل‌ها ذخیره کنید. نام فایل‌ها کلیدهای رمز را مشخص می‌کنند:

    ```shell
    echo -n 'admin' > ./username.txt
    echo -n '1f2d1e2e67df' > ./password.txt
    ```
    پرچم `-n` مطمئن می‌شود که فایل‌های تولید شده دارای کاراکتر جدید خط در پایان متن نیستند.

1.  فایل `kustomization.yaml` را ایجاد کنید:

    ```yaml
    secretGenerator:
    - name: database-creds
      files:
      - username.txt
      - password.txt
    ```
{{% /tab %}}}
{{% tab name="فایل‌های .env" %}}
همچنین می‌توانید `secretGenerator` را در فایل `kustomization.yaml` با ارائه فایل‌های `.env` تعریف کنید. به عنوان مثال، فایل `kustomization.yaml` زیر داده‌ها را از فایل `.env.secret` به دست می‌آورد:

```yaml
secretGenerator:
- name: db-user-pass
  envs:
  - .env.secret
```
{{% /tab %}}}
{{< /tabs >}}

در همه حالات، شما نیازی به رمزگذاری base64 اطلاعات ندارید. نام فایل YAML **باید** `kustomization.yaml` یا `kustomization.yml` باشد.

### اعمال فایل kustomization

برای ایجاد Secret، پوشه‌ای را که حاوی فایل kustomization است را اعمال کنید:

```shell
kubectl apply -k <directory-path>
```

خروجی مشابه زیر خواهد بود:

```
secret/database-creds-5hdh7hhgfk created
```

زمانی که یک Secret ایجاد می‌شود، نام Secret با استفاده از هش داده Secret تولید می‌شود و مقدار هش به نام اضافه می‌شود. این اطمینان می‌دهد که هر بار که داده تغییر می‌کند، یک Secret جدید تولید می‌شود.

برای اطمینان از اینکه Secret ایجاد شده است و اطلاعات Secret را رمزگشایی کنید،

```shell
kubectl get -k <directory-path> -o jsonpath='{.data}' 
```

خروجی مشابه زیر خواهد بود:

```
{ "password": "UyFCXCpkJHpEc2I9", "username": "YWRtaW4=" }
```

```shell
echo 'UyFCXCpkJHpEc2I9' | base64 --decode
```

خروجی مشابه زیر خواهد بود:

```
S!B*d$zDsb=
```

برای اطلاعات بیشتر، به مراجعه کنید:
[مدیریت Secrets با استفاده از kubectl](/docs/tasks/configmap-secret/managing-secret-using-kubectl/#verify-the-secret) و
[مدیریت اعلامی اشیاء Kubernetes با استفاده از Kustomize](/docs/tasks/manage-kubernetes-objects/kustomization/).

## ویرایش یک Secret {#edit-secret}

1.  در فایل `kustomization.yaml` خود، داده‌ها را مانند `password` ویرایش کنید.
1.  پوشه‌ای را که حاوی فایل kustomization است را اعمال کنید:

    ```shell
    kubectl apply -k <directory-path>
    ```

    خروجی مشابه زیر خواهد بود:

    ```
    secret/db-user-pass-6f24b56cc8 created
    ```

Secret ویرایش شده به عنوان یک شیء `Secret` جدید ایجاد می‌شود، به جای به‌روزرسانی شیء `Secret` موجود. شما ممکن است نیاز داشته باشید تا مراجع به Secret را در Pod‌های خود به‌روز کنید.

## پاک کردن

برای حذف یک Secret، از `kubectl` استفاده کنید:

```shell
kubectl delete secret db-user-pass
```

## {{% heading "whatsnext" %}}

- بیشتر در مورد [مفهوم Secret](/docs/concepts/configuration/secret/) بخوانید
- یاد بگیرید که چگونه [Secrets را با استفاده از kubectl مدیریت کنید](/docs/tasks/configmap-secret/managing-secret-using-kubectl/)
- یاد بگیرید که چگونه [Secrets را با استفاده از فایل پیکربندی مدیریت کنید](/docs/tasks/configmap-secret/managing-secret-using-config-file/)
