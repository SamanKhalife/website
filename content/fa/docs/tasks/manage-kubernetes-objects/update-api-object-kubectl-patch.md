---
title: به‌روزرسانی شی‌های API در مکان با استفاده از kubectl patch
description: استفاده از kubectl patch برای به‌روزرسانی شی‌های API Kubernetes در مکان. اعمال یک پچ ادغام استراتژیک یا پچ ادغام JSON.
content_type: task
weight: 50
---

<!-- مرور -->

این وظیفه نشان می‌دهد چگونه از `kubectl patch` برای به‌روزرسانی یک شی API در مکان استفاده کنید. تمرین‌های این وظیفه نحوه استفاده از پچ ادغام استراتژیک و پچ ادغام JSON را نمایش می‌دهند.

## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!-- مراحل -->

## استفاده از پچ ادغام استراتژیک برای به‌روزرسانی یک Deployment

اینجا فایل پیکربندی یک Deployment که دارای دو replica است را مشاهده می‌کنید. هر replica یک Pod با یک container است:

{{% code_sample file="application/deployment-patch.yaml" %}}

ایجاد Deployment:

```shell
kubectl apply -f https://k8s.io/examples/application/deployment-patch.yaml
```

مشاهده Pods مرتبط با Deployment شما:

```shell
kubectl get pods
```

خروجی نشان می‌دهد که Deployment دارای دو Pod است. `1/1` نشان می‌دهد که هر Pod دارای یک container است:

```
NAME                        READY     STATUS    RESTARTS   AGE
patch-demo-28633765-670qr   1/1       Running   0          23s
patch-demo-28633765-j5qs3   1/1       Running   0          23s
```

نام‌های Pods در حال اجرا را یادداشت کنید. بعداً خواهید دید که این Pods متوقف شده و توسط Pods جدیدی جایگزین می‌شوند.

در این لحظه، هر Pod دارای یک Container است که تصویر nginx را اجرا می‌کند. حال فرض کنید می‌خواهید هر Pod دارای دو container باشد: یکی که تصویر nginx و دیگری که تصویر redis را اجرا می‌کند.

یک فایل با نام `patch-file.yaml` با این محتوا ایجاد کنید:

```yaml
spec:
  template:
    spec:
      containers:
      - name: patch-demo-ctr-2
        image: redis
```

پچ کردن Deployment:

```shell
kubectl patch deployment patch-demo --patch-file patch-file.yaml
```

مشاهده Deployment پچ‌شده:

```shell
kubectl get deployment patch-demo --output yaml
```

خروجی نشان می‌دهد که PodSpec در Deployment دارای دو Container است:

```yaml
containers:
- image: redis
  imagePullPolicy: Always
  name: patch-demo-ctr-2
  ...
- image: nginx
  imagePullPolicy: Always
  name: patch-demo-ctr
  ...
```

مشاهده Pods مرتبط با Deployment پچ‌شده شما:

```shell
kubectl get pods
```

خروجی نشان می‌دهد که Pods در حال اجرا نام‌های متفاوتی نسبت به Podsی که قبلاً در حال اجرا بودند دارند. Deployment Pods قدیمی را متوقف کرده و دو Pods جدیدی را ایجاد کرده که با مشخصات به‌روزرسانی شده Deployment سازگار هستند. `2/2` نشان می‌دهد که هر Pod دارای دو Container است:

```
NAME                          READY     STATUS    RESTARTS   AGE
patch-demo-1081991389-2wrn5   2/2       Running   0          1m
patch-demo-1081991389-jmg7b   2/2       Running   0          1m
```

نگاهی نزدیک به یکی از Podsهای patch-demo بیاندازید:

```shell
kubectl get pod <your-pod-name> --output yaml
```

خروجی نشان می‌دهد که Pod دارای دو Container است: یکی که nginx و دیگری که redis را اجرا می‌کند:

```
containers:
- image: redis
  ...
- image: nginx
  ...
```

### نکات در مورد پچ ادغام استراتژیک

پچی که در تمرین قبلی انجام دادید به نام *پچ ادغام استراتژیک* معروف است. توجه داشته باشید که پچ لیست `containers` را جایگزین نکرد. به جای آن، یک Container جدید به لیست اضافه شد. به عبارت دیگر، لیست در پچ با لیست موجود ادغام شد. این همیشه اتفاقی نیست که وقتی از پچ ادغام استراتژیک بر روی یک لیست استفاده می‌شود، این اتفاق می‌افتد. در برخی موارد، لیست جایگزین می‌شود، نه ادغام می‌شود.

با پچ ادغام استراتژیک، یک لیست یا جایگزین می‌شود یا ادغام می‌شود، بسته به استراتژی پچ. استراتژی پچ توسط مقدار کلید `patchStrategy` در برچسب فیلد در کد منبع Kubernetes مشخص می‌شود. به عنوان مثال، فیلد `Containers` از ساختار `PodSpec` دارای `patchStrategy` به نام `merge` است:

```go
type PodSpec struct {
  ...
  Containers []Container `json:"containers" patchStrategy:"merge" patchMergeKey:"name" ...`
  ...
}
```

همچنین می‌توانید استراتژی پچ را در
[OpenApi spec](https://raw.githubusercontent.com/kubernetes/kubernetes/master/api/openapi-spec/swagger.json)
مشاهده کنید:

```yaml
"io.k8s.api.core.v1.PodSpec": {
    ...,
    "containers": {
        "description": "List of containers belonging to the pod.  ...."
    },
    "x-kubernetes-patch-merge-key": "name",
    "x-kubernetes-patch-strategy": "merge"
}
```

و همچنین می‌توانید استراتژی پچ را در
[مستندات API Kubernetes](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podspec-v1-core)
مشاهده کنید.

یک فایل با نام `patch-file-tolerations.yaml` با این محتوا ایجاد کنید:

```yaml
spec:
  template:
    spec:
      tolerations:
      - effect: NoSchedule
        key: disktype
        value: ssd
```

پچ کردن Deployment:

```shell
kubectl patch deployment patch-demo --

patch-file patch-file-tolerations.yaml
```

مشاهده Deployment پچ‌شده:

```shell
kubectl get deployment patch-demo --output yaml
```

خروجی نشان می‌دهد که PodSpec در Deployment فقط یک Toleration دارد:

```yaml
tolerations:
- effect: NoSchedule
  key: disktype
  value: ssd
```

توجه داشته باشید که لیست `tolerations` در PodSpec جایگزین شد، نه ادغام شد. این به دلیل این است که فیلد Tolerations در PodSpec دارای کلید `patchStrategy` در برچسب فیلد خود نیست. بنابراین پچ ادغام استراتژیک از استراتژی پچ پیش‌فرض، یعنی `replace` استفاده می‌کند.

```go
type PodSpec struct {
  ...
  Tolerations []Toleration `json:"tolerations,omitempty" protobuf:"bytes,22,opt,name=tolerations"`
  ...
}
```

## استفاده از پچ ادغام JSON برای به‌روزرسانی یک Deployment

پچ ادغام استراتژیک با پچ ادغام JSON متفاوت است. اگر از پچ ادغام JSON استفاده می‌کنید و می‌خواهید یک لیست را به‌روزرسانی کنید، باید کل لیست جدید را مشخص کنید و لیست جدید به‌طور کامل لیست موجود را جایگزین می‌کند.

دستور `kubectl patch` دارای پارامتر `type` است که می‌توانید به یکی از این مقادیر تنظیم کنید:

| مقدار پارامتر | نوع ادغام |
|----------------|------------|
| json           | [پچ JSON، RFC 6902](https://tools.ietf.org/html/rfc6902) |
| merge          | [پچ ادغام JSON، RFC 7386](https://tools.ietf.org/html/rfc7386) |
| strategic      | پچ ادغام استراتژیک |

برای مقایسه بین پچ JSON و پچ ادغام JSON، به [پچ JSON و پچ ادغام JSON](https://erosb.github.io/post/json-patch-vs-merge-patch/) مراجعه کنید.

مقدار پیش‌فرض برای پارامتر `type` استراتژیک است. بنابراین در تمرین قبلی، شما یک پچ ادغام استراتژیک انجام دادید.

حال، یک پچ ادغام JSON روی همان Deployment خود انجام دهید. یک فایل با نام `patch-file-2.yaml` با این محتوا ایجاد کنید:

```yaml
spec:
  template:
    spec:
      containers:
      - name: patch-demo-ctr-3
        image: gcr.io/google-samples/hello-app:2.0
```

در دستور پچ خود، پارامتر `type` را به `merge` تنظیم کنید:

```shell
kubectl patch deployment patch-demo --type merge --patch-file patch-file-2.yaml
```

مشاهده Deployment پچ‌شده:

```shell
kubectl get deployment patch-demo --output yaml
```

لیست `containers` که در پچ مشخص کردید، تنها یک Container را دارد. خروجی نشان می‌دهد که لیست شما از یک Container جایگزین لیست موجود `containers` شده است.

```yaml
spec:
  containers:
  - image: gcr.io/google-samples/hello-app:2.0
    ...
    name: patch-demo-ctr-3
```

لیست Pods در حال اجرا را نمایش دهید:

```shell
kubectl get pods
```

در خروجی، می‌توانید ببینید که Pods‌های موجود متوقف شده‌اند و Pods‌های جدیدی ایجاد شده‌اند. `1/1` نشان می‌دهد که هر Pod جدید تنها یک Container را اجرا می‌کند.

```shell
NAME                          READY     STATUS    RESTARTS   AGE
patch-demo-1307768864-69308   1/1       Running   0          1m
patch-demo-1307768864-c86dc   1/1       Running   0          1m
```

## استفاده از پچ ادغام استراتژیک برای به‌روزرسانی یک Deployment با استفاده از استراتژی retainKeys

اینجا فایل پیکربندی یک Deployment را که از استراتژی `RollingUpdate` استفاده می‌کند، مشاهده می‌کنید:

{{% code_sample file="application/deployment-retainkeys.yaml" %}}

ایجاد Deployment:

```shell
kubectl apply -f https://k8s.io/examples/application/deployment-retainkeys.yaml
```

در این لحظه، Deployment ایجاد شده و از استراتژی `RollingUpdate` استفاده می‌کند.

یک فایل با نام `patch-file-no-retainkeys.yaml` با این محتوا ایجاد کنید:

```yaml
spec:
  strategy:
    type: Recreate
```

پچ کردن Deployment:

```shell
kubectl patch deployment retainkeys-demo --type strategic --patch-file patch-file-no-retainkeys.yaml
```

در خروجی، می‌توانید ببینید که امکان تنظیم `type` به `Recreate` وقتی که برای `spec.strategy.rollingUpdate` مقداری تعریف شده است وجود ندارد:

```
The Deployment "retainkeys-demo" is invalid: spec.strategy.rollingUpdate: Forbidden: may not be specified when strategy `type` is 'Recreate'
```

روش برای حذف مقدار `spec.strategy.rollingUpdate` هنگام به‌روزرسانی مقدار `type` استفاده از استراتژی `retainKeys` برای پچ ادغام استراتژیک است.

فایل دیگری با نام `patch-file-retainkeys.yaml` با این محتوا ایجاد کنید:

```yaml
spec:
  strategy:
    $retainKeys:
    - type
    type: Recreate
```

با این پچ، مشخص می‌کنیم که فقط کلید `type` از شیء `strategy` را نگه داریم. بنابراین، کلید `rollingUpdate` در طول عملیات پچ حذف می‌شود.

پچ دیگری را با این پچ جدید بر روی Deployment خود انجام دهید:

```shell
kubectl patch deployment retainkeys-demo --type strategic --patch-file patch-file-retainkeys.yaml
```

محتوای Deployment را بررسی کنید:

```shell
kubectl get deployment retainkeys-demo --output yaml
```

خروجی نشان می‌دهد که شیء استراتژی در Deployment دیگر شامل کلید `rollingUpdate` نمی‌شود:

```yaml
spec:
  strategy:
    type: Recreate
  template:
```

### نکات درباره پچ ادغام استراتژیک با استفاده از استراتژی retainKeys

پچی که در تمرین قبلی انجام دادید به نام *پچ ادغام استراتژیک با استفاده از استراتژی retainKeys* معروف است. این روش یک دستورالعمل جدید به نام `$retainKeys` را معرفی می‌کند که استراتژی‌های زیر را دارد:

- این شامل یک لیست از رشته‌ها است.
- همه فیلدهای نیازمند نگه‌داری باید در لیست `$retainKeys` حضور داشته باشند.
- فیلدهای موجود در لیست، با شیء زنده ادغام می‌شوند.
- همه فیلدهای موجودی که هنگام پچ کردن از بین می‌روند، پاک می‌شوند.
- همه فیلدهای موجود در لیست `$retainKeys` باید یک زیر مجموعه یا مانند فیلدهای موجود در پچ باشند.

استراتژی `retainKeys` برای همه اشیاء کار نمی‌کند. این فقط زمانی کار می‌کند که مقدار کلید `patchStrategy` در برچسب فیلد در کد منبع Kubernetes شامل `retainKeys` باشد. به عنوان مثال، فیلد `Strategy` ساختار `DeploymentSpec` دارای `patchStrategy` با مقدار `retainKeys` است:

```go
type DeploymentSpec struct {
  ...
  // +patchStrategy=retainKeys
  Strategy DeploymentStrategy `json:"strategy,omitempty" patchStrategy:"retainKeys" ...`
  ...
}
```

شما همچنین می‌توانید استراتژی `retainKeys` را در مشخصه OpenApi ببینید:

```yaml
"io.k8s.api.apps.v1.DeploymentSpec": {
    ...,
    "strategy": {
        "$ref": "#/definitions/io.k8s.api.apps.v1.DeploymentStrategy",
        "description": "The deployment strategy to use to replace existing pods with new ones.",
        "x-kubernetes-patch-strategy": "retainKeys"
    },
    ....
}
```
<!-- برای ویرایشگرها: به دلیل جلوگیری از خطاهای برجسته‌سازی نحوه‌های کد، از فرمت yaml استفاده کنید. -->

و شما می‌توانید استراتژی `retainKeys` را در [مستندات API Kubernetes](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#deploymentspec-v1-apps) ببینید.

### فرم‌های جایگزین دستور kubectl patch

دستور `kubectl patch` فایل YAML یا JSON می‌گیرد. این می‌تواند پچ را به‌صورت یک فایل یا مستقیماً در خط دستور بگیرد.

یک فایل به نام `patch-file.json` با این محتوا ایجاد کنید:

```json
{
   "spec": {
      "template": {
         "spec": {
            "containers": [
               {
                  "name": "patch-demo-ctr-2",
                  "image": "redis"
               }
            ]
         }
      }
   }
}
```

دستورهای زیر هم‌ارز هستند:

```shell
kubectl patch deployment patch-demo --patch-file patch-file.yaml
kubectl patch deployment patch-demo --patch 'spec:\n template:\n  spec:\n   containers:\n   - name: patch-demo-ctr-2\n     image: redis'

kubectl patch deployment patch-demo --patch-file patch-file.json
kubectl patch deployment patch-demo --patch '{"spec": {"template": {"spec": {"containers": [{"name": "patch-demo-ctr-2","image": "redis"}]}}}}'
```

### به‌روزرسانی تعداد نمونه‌های یک شیء با استفاده از `kubectl patch` با `--subresource` {#scale-kubectl-patch}

{{< feature-state for_k8s_version="v1.24" state="alpha" >}}

پرچم `--subresource=[نام-زیر-منبع]` با دستورات kubectl مانند get، patch، edit و replace برای گرفتن و به‌روزرسانی زیرمنبع‌های `status` و `scale` از منابع (قابل استفاده برای نسخه kubectl v1.24 یا بیشتر) استفاده می‌شود. این پرچم با همهٔ منابع API (ویژگی‌های درونی و CR) که دارای زیرمنبع `status` یا `scale` هستند، استفاده می‌شود. Deployment یکی از مثال‌هایی است که این زیرمنبع‌ها را پشتیبانی می‌کند.

در اینجا یک نمونه از یک Deployment که دارای دو نمونه‌ از پوستری است، مشاهده می‌کنید:

{{% code_sample file="application/deployment.yaml" %}}

ایجاد Deployment:

```shell
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
```

مشاهده Pods مرتبط با Deployment شما:

```shell
kubectl get pods -l app=nginx
```

در خروجی، می‌توانید ببینید که Deployment دارای دو Pod است. به عنوان مثال:

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7fb96c846b-22567   1/1     Running   0          47s
nginx-deployment-7fb96c846b-mlgns   1/1     Running   0          47s
```

حالا، آن Deployment را با پرچم `--subresource=[نام-زیر-منبع]` پچ کنید:

```shell
kubectl patch deployment nginx-deployment --subresource='scale' --type='merge' -p '{"spec":{"replicas":3}}'
```

خروجی این است:

```shell
scale.autoscaling/nginx-deployment patched
```

مشاهده Pods مرتبط با Deployment پچ‌شده شما:

```shell
kubectl get pods -l app=nginx
```

در خروجی، می‌توانید ببینید که یک Pod جدید ایجاد شده است، پس حالا شما ۳ Pod در حال اجرا دارید.

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7fb96c846b-22567   1/1     Running   0          107s
nginx-deployment-7fb96c846b-lxfr2   1/1     Running   0          14s
nginx-deployment-7fb96c846b-mlgns   1/1     Running   0          107s
```

مشاهده Deployment پچ‌شده شما:

```shell
kubectl get deployment nginx-deployment -o yaml
```

```yaml
...
spec:
  replicas: 3
  ...
status:
  ...
  availableReplicas: 3
  readyReplicas: 3
  replicas: 3
```

{{< note >}}
اگر `kubectl patch` را اجرا کرده و برای منبعی پچ با پ

رچم `--subresource` را مشخص کنید که زیرمنبع مربوطه را پشتیبانی نمی‌کند، سرور API خطای 404 Not Found باز می‌گرداند.
{{< /note >}}

## خلاصه

در این تمرین، از `kubectl patch` برای تغییر تنظیمات زنده یک شیء Deployment استفاده کردید. شما تغییری در فایل پیکربندی اصلی که برای ایجاد شیء Deployment استفاده کرده‌اید، ایجاد نکردید. دستورات دیگر برای به‌روزرسانی اشیاء API شامل
[kubectl annotate](/docs/reference/generated/kubectl/kubectl-commands/#annotate)،
[kubectl edit](/docs/reference/generated/kubectl/kubectl-commands/#edit)،
[kubectl replace](/docs/reference/generated/kubectl/kubectl-commands/#replace)،
[kubectl scale](/docs/reference/generated/kubectl/kubectl-commands/#scale)، و
[kubectl apply](/docs/reference/generated/kubectl/kubectl-commands/#apply)
است.

{{< note >}}
پچ ادغام استراتژیک برای منابع سفارشی پشتیبانی نمی‌شود.
{{< /note >}}

## {{% heading "whatsnext" %}}

* [مدیریت اشیاء Kubernetes](/docs/concepts/overview/working-with-objects/object-management/)
* [مدیریت اشیاء Kubernetes با دستورات امپراتوری](/docs/tasks/manage-kubernetes-objects/imperative-command/)
* [مدیریت اشیاء Kubernetes امپراتوری با فایل‌های پیکربندی](/docs/tasks/manage-kubernetes-objects/imperative-config/)
* [مدیریت اشیاء Kubernetes اظهاری با فایل‌های پیکربندی](/docs/tasks/manage-kubernetes-objects/declarative-config/)
