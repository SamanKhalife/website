---
title: "توزیع اطلاعات امنیتی به صورت امن با استفاده از Secrets"
content_type: task
weight: 50
min-kubernetes-server-version: v1.6
---

<!-- overview -->

این صفحه نحوه تزریق اطلاعات حساس مانند رمز عبور و کلیدهای رمزنگاری به Pods را به صورت امن نشان می‌دهد.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}}

### تبدیل داده‌های رمزی به نمایش base-64

فرض کنید که می‌خواهید دو قطعه داده را به عنوان رمز ورود `my-app` و رمز عبور `39528$vdg7Jb` داشته باشید. ابتدا از یک ابزار رمزگذاری base64 برای تبدیل نام کاربری و رمز عبور خود به نمایش base64 استفاده کنید. در ادامه، یک مثال از برنامه base64 به طور معمول در دسترس است:

```shell
echo -n 'my-app' | base64
echo -n '39528$vdg7Jb' | base64
```

خروجی نشان می‌دهد که نمایش base-64 نام کاربری شما `bXktYXBw` و نمایش base-64 رمز عبور شما `Mzk1MjgkdmRnN0pi` است.

{{< caution >}}
از یک ابزار محلی و قابل اعتماد توسط سیستم عامل خود استفاده کنید تا ریسک‌های امنیتی ابزارهای خارجی را کاهش دهید.
{{< /caution >}}

<!-- steps -->

## ایجاد یک Secret

اینجا یک فایل پیکربندی است که می‌توانید برای ایجاد یک Secret استفاده کنید که نام کاربری و رمز عبور شما را نگهدارد:

{{% code_sample file="pods/inject/secret.yaml" %}}

1. ایجاد Secret

   ```shell
   kubectl apply -f https://k8s.io/examples/pods/inject/secret.yaml
   ```

2. مشاهده اطلاعات در مورد Secret:

   ```shell
   kubectl get secret test-secret
   ```

   خروجی:

   ```
   NAME          TYPE      DATA      AGE
   test-secret   Opaque    2         1m
   ```

3. مشاهده اطلاعات دقیق‌تر در مورد Secret:

   ```shell
   kubectl describe secret test-secret
   ```

   خروجی:

   ```
   Name:       test-secret
   Namespace:  default
   Labels:     <none>
   Annotations:    <none>

   Type:   Opaque

   Data
   ====
   password:   13 bytes
   username:   7 bytes
   ```

### ایجاد یک Secret مستقیماً با استفاده از kubectl

اگر می‌خواهید مرحله کدگذاری Base64 را بپردازید، می‌توانید از دستور `kubectl create secret` استفاده کنید. به عنوان مثال:

```shell
kubectl create secret generic test-secret --from-literal='username=my-app' --from-literal='password=39528$vdg7Jb'
```

این روش راحت‌تر است. رویکرد دقیقی که قبلاً نشان داده شد، هر گام را به صورت صریح اجرا می‌کند تا نشان دهد چه اتفاقی می‌افتد.

## ایجاد یک Pod که دسترسی به داده‌های Secret را از طریق یک Volume دارد

اینجا یک فایل پیکربندی است که می‌توانید برای ایجاد یک Pod استفاده کنید:

{{% code_sample file="pods/inject/secret-pod.yaml" %}}

1. ایجاد Pod:

   ```shell
   kubectl apply -f https://k8s.io/examples/pods/inject/secret-pod.yaml
   ```

2. تأیید کنید که Pod شما در حال اجرا است:

   ```shell
   kubectl get pod secret-test-pod
   ```

   خروجی:

   ```
   NAME              READY     STATUS    RESTARTS   AGE
   secret-test-pod   1/1       Running   0          42m
   ```

3. ورود به یک شل در کانتینری که در Pod شما اجرا می‌شود:

   ```shell
   kubectl exec -i -t secret-test-pod -- /bin/bash
   ```

4. داده‌های Secret از طریق یک Volume که زیر `/etc/secret-volume` مونت شده است به کانتینر نمایش داده می‌شود.

   در شل خود، فایل‌های موجود در دایرکتوری `/etc/secret-volume` را لیست کنید:

   ```shell
   # این را در شل داخل کانتینر اجرا کنید
   ls /etc/secret-volume
   ```

   خروجی نمایش داده می‌شود دو فایل، یکی برای هر قطعه اطلاعات محرمانه:

   ```
   password username
   ```

5. در شل خود، محتویات فایل‌های `username` و `password` را نمایش دهید:

   ```shell
   # این را در شل داخل کانتینر اجرا کنید
   echo "$( cat /etc/secret-volume/username )"
   echo "$( cat /etc/secret-volume/password )"
   ```

   خروجی نام کاربری و رمز عبور شما است:

   ```
   my-app
   39528$vdg7Jb
   ```

تغییر تصویر یا خط فرمان خود را طوری انجام دهید که برنامه در دایرکتوری `mountPath` بگردد. هر کلید در نقشه Secret `data` به عنوان یک نام فایل در این دایرکتوری تبدیل می‌شود.

### تخصیص کلیدهای Secret به مسیرهای فایل خاص

همچنین می‌توانید مسیرهای داخل حجم را که کلیدهای Secret در آن‌ها پروژه‌اید، کنترل کنید. برای تغییر مسیر مقصد هر کلید، از فیلد `.spec.volumes[].secret.items` استفاده کنید:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
```

زمانی که این Pod را استقرار می‌دهید، اتفاقات زیر رخ می‌دهد:

- کلید `username` از `mysecret` به دسترس کانتینر در مسیر `/etc/foo/my-group/my-username` است به جای `/etc/foo/username`.
- کلید `password` از آن جسم Secret پروژه‌نامه نیست.

اگر کلیدها را به طور صریح با استفاده از `.spec.volumes[].secret.items` لیست کنید، به موارد زیر توجه داشته باشید:

- فقط کلیدهای مشخص شده در `items` پروژه‌نامه نمایش داده می‌شوند.
- برای مصرف همه کلیدها از Secret، همه آنها باید در فیلد `items` لیست شوند.
- همه کلیدهای مشخص شده باید در Secret مربوطه وجود داشته باشند. در غیر این صورت، حجم ایجاد نمی‌شود.

### تنظیم مجوزهای POSIX برای کلیدهای Secret

می‌توانید بیت‌های دسترسی پرونده POSIX برای یک کلید مخفی تنظیم کنید. اگر هیچ مجوزی مشخص نکنید، به طور پیش‌فرض از `0644` استفاده می‌شود. همچنین می‌توانید حالت فایل پیش‌فرض POSIX برای کل حجم Secret تعیین کنید و در صورت لزوم به صورت اختصاصی نیز اجازه دهید.

به عنوان مثال، می‌توانید یک حالت پیش‌فرض مانند این تعیین کنید:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      defaultMode: 0400
```

Secret در `/etc/foo` مونت می‌شود؛ تمام فایل‌های ایجاد شده توسط مونت حجم Secret دسترسی `0400` را دارند.

{{< note >}}
اگر شما یک Pod یا یک قالب Pod را با استفاده از JSON تعریف می‌کنید، توجه کنید که مشخصات JSON از اعداد اوکتال برای اعداد استفاده نمی‌کند زیرا JSON مقدار `0400` را به عنوان مقدار دهدهی دهدهی به اعداد 400 می‌پذیرد. در JSON، به جای مقادیر اوکتال برای `defaultMode` از مقادیر دهدهی استفاده کنید. اگر شما YAML می‌نویسید، می‌توانید `defaultMode` را به صورت اوکتال بنویسید.
{{< /note >}}

## تعریف متغیرهای محیطی کانتینر با استفاده از داده‌های Secret

شما می‌توانید داده‌های Secret را به عنوان متغیرهای محیطی در کانتینرهای خود مصرف کنید.

اگر یک کانتینر قبلاً یک Secret را در یک متغیر محیطی مصرف می‌کند، به روزرسانی Secret توسط کانتینر مشاهده نمی‌شود مگر اینکه کانتینر راه‌اندازی مجدد شود. برای این منظور راه‌حل‌های شخص ثالث برای انجام دادن این کار وجود دارد.

### تعریف یک متغیر محیطی کانتینر با داده از یک Secret تکی

- تعریف یک متغیر محیطی به عنوان یک جفت کلید-مقدار در یک Secret:

  ```shell
  kubectl create secret generic backend-user --from-literal=backend-username='backend-admin'
  ```

- تخصیص مقدار `backend-username` که در Secret تعریف شده است به متغیر محیطی `SECRET_USERNAME` در مشخصات Pod.

  {{% code_sample file="pods/inject/pod-single-secret-env-variable.yaml" %}}

- ایجاد Pod:

  ```shell
  kubectl create -f https://k8s.io/examples/pods/inject/pod-single-secret-env-variable.yaml
  ```

- در شل خود، محتویات متغیر محیطی `SECRET_USERNAME` کانتینر را نمایش دهید.

  ```shell
  kubectl exec -i -t env-single-secret -- /bin/sh -c 'echo $SECRET_USERNAME'
  ```

  خروجی مشابه زیر است:

  ```
  backend-admin
  ```

### تعریف متغیرهای محیطی کانتینر با داده از چندین Secret

- همانند مثال قبلی، ابتدا Secrets را ایجاد کنید.

  ```shell
  kubectl create secret generic backend-user --from-literal=backend-username='backend-admin'
  kubectl create secret generic db-user --from-literal=db-username='db-admin'
  ```

- تعریف متغیرهای محیطی در مشخصات Pod.

  {{% code_sample file="pods/inject/pod-multiple-secret-env-variable.yaml" %}}

- ایجاد Pod:

  ```shell
  kubectl create -f https://k8s.io/examples/pods/inject/pod-multiple-secret-env-variable.yaml
  ```

- در شل خود، متغیرهای محیطی کانتینر را نمایش دهید.

  ```shell
  kubectl exec -i -t envvars-multiple-secrets -- /bin/sh -c 'env | grep _USERNAME'
  ```

  خروجی مشابه زیر است:

  ```
  DB_USERNAME=db-admin
  BACKEND_USERNAME=backend-admin
  ```

## پیکربندی همه جفت کلید-مقدار‌های Secret به عنوان متغیرهای محیطی کانتینر

{{< note >}}
این قابلیت در Kubernetes نسخه ۱.۶ و بالاتر موجود است.
{{< /note >}}

- ایجاد یک Secret شامل چندین جفت کلید-مقدار:

  ```shell
  kubectl create secret generic test-secret --from-literal=username='my-app' --from-literal=password='39528$vdg7Jb'
  ```

- استفاده از envFrom برای تعریف تمام داده‌های Secret به عنوان متغیرهای محیطی کانتینر. کلید از Secret نام متغیر محیطی در Pod می‌شود.

  {{% code_sample file="pods/inject/pod-secret-envFrom.yaml" %}}

- ایجاد Pod:

  ```shell
  kubectl create -f https://k8s.io/examples/pods/inject/pod-secret-envFrom.yaml
  ```

- در شل خود، نمایش متغیرهای محیطی `username` و `password` کانتینر.

  ```shell
  kubectl exec -i -t envfrom-secret -- /bin/sh -c 'echo "username: $username\npassword: $password\n"'
  ```

  خروجی مشابه زیر است:

  ```
  username: my-app
  password: 39528$vdg7Jb
  ```

## مثال: ارائه اعتبارهای prod/test به Pods با استفاده از Secrets {#provide-prod-test-creds}

در این مثال، یک Pod نشان می‌دهد که یک Secret حاوی اعتبارهای محیط تولید و یک Pod دیگر نشان می‌دهد که یک Secret حاوی اعتبارهای محیط آزمایشی را مصرف می‌کند.

1. ایجاد Secret برای اعتبارهای محیط prod:

   ```shell
   kubectl create secret generic prod-db-secret --from-literal=username=produser --from-literal=password=Y4nys7f11
   ```

   خروجی مشابه زیر است:

   ```
   secret "prod-db-secret" created
   ```

1. ایجاد Secret برای اعتبارهای محیط آزمایشی:

   ```shell
   kubectl create secret generic test-db-secret --from-literal=username=testuser --from-literal=password=iluvtests
   ```

   خروجی مشابه زیر است:

   ```
   secret "test-db-secret" created
   ```

   {{< note >}}
   کاراکترهای ویژه مانند `$`, `\`, `*`, `=`, و `!` توسط شل شما تفسیر می‌شوند و نیاز به escaping دارند.

   در بیشتر شل‌ها، راه ساده‌تر برای escaping پسورد این است که آن را با نقل قول تکی (`'`) در احاطه کنید.
   به عنوان مثال، اگر پسورد واقعی شما `S!B\*d$zDsb=` است، دستور را به این صورت اجرا کنید:

   ```shell
   kubectl create secret generic dev-db-secret --from-literal=username=devuser --from-literal=password='S!B\*d$zDsb='
   ```

   شما نیازی به escaping کاراکترهای ویژه در پسوردها از طریق فایل‌ها (`--from-file`) ندارید.
   {{< /note >}}

1. ایجاد مانیفست‌های Pod:

   ```shell
   cat <<EOF > pod.yaml
   apiVersion: v1
   kind: List
   items:
   - kind: Pod
     apiVersion: v1
     metadata:
       name: prod-db-client-pod
       labels:
         name: prod-db-client
     spec:
       volumes:
       - name: secret-volume
         secret:
           secretName: prod-db-secret
       containers:
       - name: db-client-container
         image: myClientImage
         volumeMounts:
         - name: secret-volume
           readOnly: true
           mountPath: "/etc/secret-volume"
   - kind: Pod
     apiVersion: v1
     metadata:
       name: test-db-client-pod
       labels:
         name: test-db-client
     spec:
       volumes:
       - name: secret-volume
         secret:
           secretName: test-db-secret
       containers:
       - name: db-client-container
         image: myClientImage
         volumeMounts:
         - name: secret-volume
           readOnly: true
           mountPath: "/etc/secret-volume"
   EOF
   ```

   {{< note >}}
   چگونگی مشخصات برای دو Pods کارایی‌های مختلف فقط در یک فیلد متفاوت‌تر است؛ این موضوع برای ایجاد Pods از یک الگوی مشترک موجود است.
   {{< /note >}}

1. اعمال تمام این اشیاء بر روی سرور API با اجرا:

   ```shell
   kubectl create -f pod.yaml
   ```

هر دو کانتینر دارای فایل‌های زیر بر روی فایل‌سیستم خود با مقادیر برای هر محیط کانتینر خواهند بود:

```
/etc/secret-volume/username
/etc/secret-volume/password
```

می‌توانید مشخصات پایه Pod را با استفاده از دو حساب سرویس:

1. `prod-user` با `prod-db-secret`
1. `test-user` با `test-db-secret`

مشخصات Pod به این شکل کوتاه می‌شود:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: prod-db-client-pod
  labels:
    name: prod-db-client
spec:
  serviceAccount: prod-db-client
  containers:
  - name: db-client-container
    image: myClientImage
```

### مراجع

- [Secret](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#secret-v1-core)
- [Volume](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#volume-v1-core)
- [Pod](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#pod-v1-core)

## {{% heading "whatsnext" %}}

- یادگیری بیشتر درباره [Secrets](/docs/concepts/configuration/secret/).
- یادگیری درباره [Volumes](/docs/concepts/storage/volumes/).
