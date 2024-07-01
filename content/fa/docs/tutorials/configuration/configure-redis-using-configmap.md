---
reviewers:
- eparis
- pmorie
title: پیکربندی Redis با استفاده از ConfigMap
content_type: tutorial
weight: 30
---

<!-- overview -->

این صفحه یک مثال واقعی از نحوه پیکربندی Redis با استفاده از ConfigMap را ارائه می‌دهد و بر اساس وظیفه [پیکربندی یک پاد برای استفاده از ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/) بنا شده است.



## {{% heading "objectives" %}}


* ایجاد یک ConfigMap با مقادیر پیکربندی Redis
* ایجاد یک پاد Redis که ConfigMap ایجاد شده را مونت و استفاده کند
* اطمینان از اینکه پیکربندی به درستی اعمال شده است.



## {{% heading "prerequisites" %}}


{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

* مثال نشان داده شده در این صفحه با `kubectl` 1.14 و بالاتر کار می‌کند.
* آشنایی با [پیکربندی یک پاد برای استفاده از ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/).



<!-- lessoncontent -->


## مثال واقعی: پیکربندی Redis با استفاده از ConfigMap

مراحل زیر را دنبال کنید تا یک حافظه کش Redis با استفاده از داده‌های ذخیره شده در یک ConfigMap پیکربندی کنید.

ابتدا یک ConfigMap با یک بلوک پیکربندی خالی ایجاد کنید:

```shell
cat <<EOF >./example-redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: ""
EOF
```

ConfigMap ایجاد شده در بالا را همراه با یک مانفیست پاد Redis اعمال کنید:

```shell
kubectl apply -f example-redis-config.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
```

محتوای مانفیست پاد Redis را بررسی کنید و به نکات زیر توجه کنید:

* یک حجم به نام `config` توسط `spec.volumes[1]` ایجاد شده است.
* `کلید` و `مسیر` تحت `spec.volumes[1].configMap.items[0]` کلید `redis-config` از ConfigMap `example-redis-config` را به عنوان یک فایل به نام `redis.conf` در حجم `config` افشا می‌کند.
* حجم `config` سپس در `/redis-master` توسط `spec.containers[0].volumeMounts[1]` مونت می‌شود.

این کار اثر خالص افشای داده‌های `data.redis-config` از ConfigMap `example-redis-config` به عنوان `/redis-master/redis.conf` در داخل پاد را دارد.

{{% code_sample file="pods/config/redis-pod.yaml" %}}

اشیاء ایجاد شده را بررسی کنید:

```shell
kubectl get pod/redis configmap/example-redis-config
```

باید خروجی زیر را مشاهده کنید:

```
NAME        READY   STATUS    RESTARTS   AGE
pod/redis   1/1     Running   0          8s

NAME                             DATA   AGE
configmap/example-redis-config   1      14s
```

به یاد داشته باشید که کلید `redis-config` در ConfigMap `example-redis-config` خالی باقی ماند:

```shell
kubectl describe configmap/example-redis-config
```

باید یک کلید `redis-config` خالی را مشاهده کنید:

```shell
Name:         example-redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
redis-config:
```

از `kubectl exec` برای ورود به پاد و اجرای ابزار `redis-cli` برای بررسی پیکربندی فعلی استفاده کنید:

```shell
kubectl exec -it redis -- redis-cli
```

`maxmemory` را بررسی کنید:

```shell
127.0.0.1:6379> CONFIG GET maxmemory
```

باید مقدار پیش‌فرض 0 را نشان دهد:

```shell
1) "maxmemory"
2) "0"
```

به همین ترتیب، `maxmemory-policy` را بررسی کنید:

```shell
127.0.0.1:6379> CONFIG GET maxmemory-policy
```

که باید مقدار پیش‌فرض `noeviction` را نشان دهد:

```shell
1) "maxmemory-policy"
2) "noeviction"
```

اکنون بیایید مقداری مقادیر پیکربندی را به ConfigMap `example-redis-config` اضافه کنیم:

{{% code_sample file="pods/config/example-redis-config.yaml" %}}

ConfigMap به‌روز شده را اعمال کنید:

```shell
kubectl apply -f example-redis-config.yaml
```

تأیید کنید که ConfigMap به‌روز شده است:

```shell
kubectl describe configmap/example-redis-config
```

باید مقادیر پیکربندی که به تازگی اضافه کرده‌ایم را مشاهده کنید:

```shell
Name:         example-redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
redis-config:
----
maxmemory 2mb
maxmemory-policy allkeys-lru
```

دوباره پاد Redis را با استفاده از `redis-cli` از طریق `kubectl exec` بررسی کنید تا ببینید آیا پیکربندی اعمال شده است:

```shell
kubectl exec -it redis -- redis-cli
```

`maxmemory` را بررسی کنید:

```shell
127.0.0.1:6379> CONFIG GET maxmemory
```

هنوز مقدار پیش‌فرض 0 را نشان می‌دهد:

```shell
1) "maxmemory"
2) "0"
```

به همین ترتیب، `maxmemory-policy` هنوز به تنظیمات پیش‌فرض `noeviction` باقی مانده است:

```shell
127.0.0.1:6379> CONFIG GET maxmemory-policy
```

بازده:

```shell
1) "maxmemory-policy"
2) "noeviction"
```

مقادیر پیکربندی تغییر نکرده‌اند زیرا پاد نیاز به بازنشانی دارد تا مقادیر به‌روز شده از ConfigMap های مرتبط را بگیرد. بیایید پاد را حذف کرده و دوباره ایجاد کنیم:

```shell
kubectl delete pod redis
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
```

اکنون یک بار دیگر مقادیر پیکربندی را بررسی کنید:

```shell
kubectl exec -it redis -- redis-cli
```

`maxmemory` را بررسی کنید:

```shell
127.0.0.1:6379> CONFIG GET maxmemory
```

اکنون باید مقدار به‌روز شده 2097152 را برگرداند:

```shell
1) "maxmemory"
2) "2097152"
```

به همین ترتیب، `maxmemory-policy` نیز به‌روز شده است:

```shell
127.0.0.1:6379> CONFIG GET maxmemory-policy
```

اکنون مقدار مورد نظر `allkeys-lru` را نشان می‌دهد:

```shell
1) "maxmemory-policy"
2) "allkeys-lru"
```

با حذف منابع ایجاد شده کار خود را پاک کنید:

```shell
kubectl delete pod/redis configmap/example-redis-config
```

## {{% heading "whatsnext" %}}


* بیشتر درباره [ConfigMapها](/docs/tasks/configure-pod-container/configure-pod-configmap/) بیاموزید.
* مثالی از [به‌روزرسانی پیکربندی از طریق ConfigMap](/docs/tutorials/configuration/updating-configuration-via-a-configmap/) را دنبال کنید.
