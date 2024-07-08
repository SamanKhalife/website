---
title: "گسترش API Kubernetes با CustomResourceDefinitions"
reviewers:
- deads2k
- jpbetz
- liggitt
- roycaihw
- sttts
content_type: task
min-kubernetes-server-version: 1.16
weight: 20
---

<!-- overview -->
این صفحه نحوه نصب یک
[منبع سفارشی](/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
را در API Kubernetes با ایجاد یک
[CustomResourceDefinition](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#customresourcedefinition-v1-apiextensions-k8s-io)
نشان می‌دهد.

## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}
اگر از نسخه‌ای قدیمی‌تر از Kubernetes استفاده می‌کنید که هنوز پشتیبانی می‌شود، به مستندات آن نسخه مراجعه کنید تا راهنمایی‌هایی که برای خوشه شما مناسب است را ببینید.

<!-- steps -->

## ایجاد CustomResourceDefinition

زمانی که یک CustomResourceDefinition (CRD) جدید ایجاد می‌کنید، سرور API Kubernetes
یک مسیر منبع RESTful جدید برای هر نسخه‌ای که مشخص می‌کنید ایجاد می‌کند. منبع سفارشی
ایجاد شده از یک شی CRD می‌تواند در حوزه نام یا حوزه خوشه قرار داشته باشد، همان‌طور که در
فیلد `spec.scope` CRD مشخص شده است. همانند اشیاء موجود داخلی، حذف یک فضای نام،
تمام اشیاء سفارشی در آن فضای نام را حذف می‌کند.
خود CustomResourceDefinitions از نوع بدون نام و برای همه فضاهای نام موجود است.

به عنوان مثال، اگر شما این CustomResourceDefinition زیر را در `resourcedefinition.yaml` ذخیره کنید:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # نام باید با فیلدهای مشخص شده زیر همخوانی داشته باشد، و به صورت: <جمع>.<گروه> باشد
  name: crontabs.stable.example.com
spec:
  # نام گروه برای استفاده در REST API: /apis/<گروه>/<نسخه>
  group: stable.example.com
  # لیست نسخه‌های پشتیبانی شده توسط این CustomResourceDefinition
  versions:
    - name: v1
      # هر نسخه می‌تواند توسط پرچم Served فعال/غیرفعال شود.
      served: true
      # یک و تنها یک نسخه باید به عنوان نسخه ذخیره شده مشخص شود.
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  # به صورت Namespaced یا Cluster
  scope: Namespaced
  names:
    # نام جمع برای استفاده در URL: /apis/<گروه>/<نسخه>/<جمع>
    plural: crontabs
    # نام مفرد برای استفاده به عنوان یک نام مستعار در CLI و برای نمایش
    singular: crontab
    # kind به طور معمول نوع CamelCased از نوع تک جمع است. اشیای منابع شما از این استفاده می‌کنند.
    kind: CronTab
    # shortNames اجازه می‌دهد رشته کوتاه‌تری را برای مطابقت منبع خود در CLI استفاده کنید
    shortNames:
    - ct
```

و آن را ایجاد کنید:

```shell
kubectl apply -f resourcedefinition.yaml
```

آنگاه یک نقطه دسترسی API RESTful جدید در حوزه نام‌دار ایجاد می‌شود در:

```
/apis/stable.example.com/v1/namespaces/*/crontabs/...
```

این URL نقطه دسترسی می‌تواند برای ایجاد و مدیریت اشیاء سفارشی استفاده شود.
`kind` این اشیاء از مشخصات اشیاء CronTab از شی CustomResourceDefinitionی که در بالا ایجاد کرده‌اید خواهد بود.

ممکن است چند ثانیه طول بکشد تا نقطه دسترسی ایجاد شود.
می‌توانید شرایط `Established` CRD خود را برای صحیح بودن بررسی کنید یا اطلاعات کشف سرور API خود را برای منبع خود بررسی کنید.

## ایجاد اشیاء سفارشی

پس از ایجاد شی CustomResourceDefinition، می‌توانید اشیاء سفارشی ایجاد کنید.
اشیاء سفارشی می‌توانند شامل فیلدهای سفارشی باشند. این فیلدها می‌توانند JSON دلخواهی را
شامل شوند.
در مثال زیر، فیلدهای سفارشی `cronSpec` و `image` در یک شی سفارشی با نوع `CronTab` تنظیم شده‌اند.
نوع `CronTab` از مشخصات اشیاء CronTab از شی CustomResourceDefinitionی که در بالا ایجاد کرده‌اید می‌آید.

اگر این YAML را در `my-crontab.yaml` ذخیره کنید:

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
```

و آن را ایجاد کنید:

```shell
kubectl apply -f my-crontab.yaml
```

می‌توانید سپس اشیاء CronTab خود را با استفاده از kubectl مدیریت کنید. به عنوان مثال:

```shell
kubectl get crontab
```

باید یک لیستی مانند این چاپ کند:

```none
NAME                 AGE
my-new-cron-object   6s
```

نام منابع حساس به کوچکی/بزرگی نیست و می‌توانید از شکل‌های تک جمع یا جمع تعریف شده در CRD استفاده کنید، همچنین از هر نام کوتاهی.

همچنین می‌توانید

 داده‌های YAML خام را مشاهده کنید:

```shell
kubectl get ct -o yaml
```

باید ببینید که حاوی فیلدهای سفارشی `cronSpec` و `image` از YAML استفاده شده برای ایجاد آن است:

```yaml
apiVersion: v1
items:
- apiVersion: stable.example.com/v1
  kind: CronTab
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"stable.example.com/v1","kind":"CronTab","metadata":{"annotations":{},"name":"my-new-cron-object","namespace":"default"},"spec":{"cronSpec":"* * * * */5","image":"my-awesome-cron-image"}}
    creationTimestamp: "2021-06-20T07:35:27Z"
    generation: 1
    name: my-new-cron-object
    namespace: default
    resourceVersion: "1326"
    uid: 9aab1d66-628e-41bb-a422-57b8b3b1f5a9
  spec:
    cronSpec: '* * * * */5'
    image: my-awesome-cron-image
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

## حذف یک CustomResourceDefinition

زمانی که یک CustomResourceDefinition را حذف می‌کنید، سرور مسیر API RESTful را حذف می‌کند
و تمام اشیاء سفارشی ذخیره شده در آن را حذف می‌کند.

```shell
kubectl delete -f resourcedefinition.yaml
kubectl get crontabs
```

```none
Error from server (NotFound): Unable to list {"stable.example.com" "v1" "crontabs"}: the server could not
find the requested resource (get crontabs.stable.example.com)
```

اگر در آینده همان CustomResourceDefinition را دوباره ایجاد کنید، خالی شروع می‌شود.

## تعیین یک طرح ساختاری

CustomResources داده‌های ساختاری را در فیلدهای سفارشی ذخیره می‌کنند (همراه با
فیلدهای داخلی `apiVersion`، `kind` و `metadata` که سرور API به طور ضمنی اعتبارسنجی می‌کند).
با [OpenAPI v3.0 validation](#validation) می‌توان یک طرح را مشخص کرد که در ایجاد و بروزرسانی اعتبارسنجی می‌شود، جزئیات و محدودیت‌های چنین طرحی را مقایسه کنید در زیر.

با `apiextensions.k8s.io/v1` تعریف یک طرح ساختاری برای CustomResourceDefinitions الزامی است. در نسخه بتا CustomResourceDefinition، طرح ساختاری اختیاری بود.

یک طرح ساختاری یک [OpenAPI v3.0 validation schema](#validation) است که:

1. نوع غیرخالی را برای ریشه (با استفاده از `type` در OpenAPI) برای هر فیلد مشخص شده از یک گره شی (با استفاده از `properties` یا `additionalProperties` در OpenAPI) و برای هر مورد در یک گره آرایه (با استفاده از `items` در OpenAPI) مشخص می‌کند، به استثنای:
   * یک گره با `x-kubernetes-int-or-string: true`
   * یک گره با `x-kubernetes-preserve-unknown-fields: true`
2. برای هر فیلد در یک شی و هر مورد در یک آرایه که در هرکدام از `allOf`، `anyOf`، `oneOf` یا `not` مشخص شده است، طرح همچنین فیلد/مورد را خارج از این مفسره‌های منطقی مشخص می‌کند (مقایسه مثال 1 و 2).
3. `description`، `type`، `default`، `additionalProperties`، `nullable` را درون `allOf`، `anyOf`، `oneOf` یا `not` تنظیم نمی‌کند، به استثنای دو الگو برای `x-kubernetes-int-or-string: true` (مراجعه کنید به پایین).
4. اگر `metadata` مشخص شود، تنها محدودیت‌های `metadata.name` و `metadata.generateName` مجاز هستند.

مثال غیرساختاری 1:

```none
allOf:
- properties:
    foo:
      ...
```

در تضاد با قانون 2 است. درست زیرین:

```none
properties:
  foo:
    ...
allOf:
- properties:
    foo:
      ...
```

مثال غیرساختاری 2:

```none
allOf:
- items:
    properties:
      foo:
        ...
```

در تضاد با قانون 2 است. درست زیرین:

```none
items:
  properties:
    foo:
      ...
allOf:
- items:
    properties:
      foo:
        ...
```

مثال غیرساختاری 3:

```none
properties:
  foo:
    pattern: "abc"
  metadata:
    type: object
    properties:
      name:
        type: string
        pattern: "^a"
      finalizers:
        type: array
        items:
          type: string
          pattern: "my-finalizer"
anyOf:
- properties:
    bar:
      type: integer
      minimum: 42
  required: ["bar"]
  description: "foo bar object"
```

یک طرح ساختاری نیست به دلیل نقض‌های زیر:

* نوع در ریشه گم شده است (قانون 1).
* نوع `foo` گم شده است (قانون 1).
* `bar` در داخل `anyOf` خارج نشده است (قانون 2).
* نوع `bar` در داخل `anyOf` مشخص شده است (قانون 3).
* توضیحات در داخل `anyOf` تنظیم شده است (قانون 3).
* `metadata.finalizers` ممکن است محدود نشده باشد (قانون 4).

به عنوان مقایسه، طرح ساختاری زیرین، طرح متناظر است:

```yaml
type: object
description: "foo bar object"
properties:
  foo:
    type: string
    pattern: "abc"
  bar:
    type: integer
  metadata:
    type: object
    properties:
      name:
        type: string
        pattern: "^a"
anyOf:
- properties:
    bar:
      minimum: 42
  required: ["bar"]
```

نقض‌های قوانین طرح ساختاری در شرایط `NonStructural` در CustomResourceDefinition گزارش می‌شوند.

### حذف فیلد

CustomResourceDefinitions داده‌های منابع اعتبارسنجی شده را در فضای ذخیره‌سازی دائمی خوشه مانند {{< glossary_tooltip term_id="etcd" text="etcd">}} ذخیره می‌کنند.
مانند منابع اصلی Kubernetes مانند {{< glossary_tooltip text="ConfigMap" term_id="configmap" >}},
اگر یک فیلد را که سرور API شناسایی نمی‌کند مشخص کنید، فیلد ناشناخته قبل از ذخیره‌سازی _pruned_ (حذف) می‌شود.

CRDs که از `apiextensions.k8s.io/v1beta1` به `apiextensions.k8s.io/v1` تبدیل شده باشند
ممکن است از طرح‌های ساختاری بی‌خبر باشند و `spec.preserveUnknownFields` ممکن است `true` باشد.

برای CustomResourceDefinition‌های قدیمی‌ای که به عنوان `apiextensions.k8s.io/v1beta1` با `spec.preserveUnknownFields` به `true` ایجاد شده‌اند، این نکات همچنین درست است:

* حذف فعال نیست.
* می‌توانید داده‌های دلخواه ذخیره کنید.

برای سازگاری با `apiextensions.k8s.io/v1`، تعریف‌های منابع سفارشی خود را به روز کنید به:

1. استفاده از یک طرح ساختاری OpenAPI.
2. `spec.preserveUnknownFields` را به `false` تنظیم کنید.

اگر YAML زیر را به `my-crontab.yaml` ذخیره کنید:

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
  someRandomField: 42
```

و آن را ایجاد کنید:

```shell
kubectl create --validate=false -f my-crontab.yaml -o yaml
```

خروجی شما مشابه زیر است:

```yaml
apiVersion: stable.example.com/v1
kind: CronTab
metadata:
  creationTimestamp: 2017-05-31T12:56:35Z
  generation: 1
  name: my-new-cron-object
  namespace: default
  resourceVersion: "285"
  uid: 9423255b-4600-11e7-af6a-28d2447dc82b
spec:
  cronSpec: '* * * * */5'
  image: my-awesome-cron-image
```

توجه داشته باشید که فیلد `someRandomField` حذف شد.

این مثال با غیرفعال کردن اعتبارسنجی سمت مشتری برای نمایش رفتار سرور API، با افزودن گزینه خط فرمان `--validate=false` انجام شد.
زیرا [طرح‌های اعتبارسنجی OpenAPI همچنین منتشر شده‌اند](#publish-validation-schema-in-openapi)
به مشتری‌ها، `kubectl` نیز فیلدهای ناشناخته را بررسی می‌کند و اشیاء را قبل از ارسال به سرور API رد می‌کند.

#### کنترل حذف

به طور پیش‌فرض، همه فیلدهای مشخص نشده برای یک منبع سفارشی، در تمام نسخه‌ها، حذف می‌شوند. اما امکان دارد برای زیر‌درخت‌های خاصی از فیلدها از این کار خارج شوید با اضافه کردن `x-kubernetes-preserve-unknown-fields: true` در
[طرح ساختاری اعتبارسنجی OpenAPI v3](#specifying-a-structural-schema).

به عنوان مثال:

```yaml
type: object
properties:
  json:
    x-kubernetes-preserve-unknown-fields: true
```

فیلد `json` می‌تواند هر مقدار JSON را ذخیره کند، بدون حذف چیزی.

همچنین می‌توانید جزئیات JSON مجاز را مشخص کنید؛ به عنوان مثال:

```yaml
type: object
properties:
  json:
    x-kubernetes-preserve-unknown-fields: true
    type: object
    description: این JSON دلخواه است
```

با این حال، فقط انواع `object` مجاز هستند.

حذف دوباره برای هر خاصیت مشخص شده (یا `additionalProperties`):

```yaml
type: object
properties:
  json:
    x-kubernetes-preserve-unknown-fields: true
    type: object
    properties:
      spec:
        type: object
        properties:
          foo:
            type: string
          bar:
            type: string
```

با این، مقدار زیر:

```yaml
json:
  spec:
    foo: abc
    bar: def
    something: x
  status:
    something: x
```

به این شکل حذف می‌شود:

```yaml
json:
  spec:
    foo: abc
    bar: def
  status:
    something: x
```

این بدان معنی است که فیلد `something` در شی `spec` مشخص شده، حذف می‌شود، اما همه چیز بیرون از آن حذف نمی‌شود.

### IntOrString

گره‌ها در یک طرح با `x-kubernetes-int-or-string: true` از قانون 1 مستثنی شده‌اند، به طوری که
مقادیر زیر ساختاری هستند:

```yaml
type: object
properties:
  foo:
    x-kubernetes-int-or-string: true
```

همچنین این گره‌ها به طور جزئی از قانون 3 مستثنی می‌شوند به معنای این که دو الگوی زیر مجاز هستند
(دقیقا اینها، بدون تغییرات در ترتیب به فیلدهای اضافی):

```none
x-kubernetes-int-or-string: true
anyOf:
  - type: integer
  - type: string
...
```

و

```none
x-kubernetes-int-or-string: true
allOf:
  - anyOf:
      - type: integer
      - type: string
  - ... # صفر یا بیشتر
...
```

با یکی از این مشخصات، هر دو integer و string اعتبارسنجی می‌شوند.

در [انتشار طرح اعتبارسنجی](#publish-validation-schema-in-openapi)،
`x-kubernetes-int-or-string: true` به یکی از الگوهای نشان داده شده بالا گسترده می‌شود.

### RawExtension

RawExtensions (مانند [`runtime.RawExtension`](/docs/reference//kubernetes-api/workload-resources/controller-revision-v1#RawExtension))
شی‌های کامل Kubernetes را نگه می‌دارد، به این معنی که دارای فیلدهای `apiVersion` و `kind` هستند.

امکان دارد این شی‌های جانشین (هم کاملاً بدون محدودیت یا جزئی مشخص شده)
با تنظیم `x-kubernetes-embedded-resource: true` مشخص شوند. به عنوان مثال:

```yaml
type: object
properties:
  foo:
    x-kubernetes-embedded-resource: true
    x-kubernetes-preserve-unknown-fields: true
```

در اینجا، فیلد `foo` یک شی کامل را نگه می‌دارد، به عنوان مثال:

```none
foo:
  apiVersion: v1
  kind: Pod
  spec:
    ...
```

زیرا `x-kubernetes-preserve-unknown-fields: true` همراه است، هیچ چیز حذف نمی‌شود.
استفاده از `x-kubernetes-preserve-unknown-fields: true` اختیاری است.

با `x-kubernetes-embedded-resource: true`، `apiVersion`، `kind` و `metadata` به طور ضمنی مشخص و اعتبارسنجی می‌شوند.

## ارائه چند نسخه از یک CRD

برای اطلاعات بیشتر در مورد ارائه چند نسخه از CustomResourceDefinition خود
می‌توانید به [نسخه‌بندی تعریف منبع سفارشی](/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definition-versioning/)
مراجعه کنید و اشیاء خود را از یک نسخه به نسخه دیگر منتقل کنید.

<!-- discussion -->
## مباحث پیشرفته

### Finalizers

*Finalizers* به کنترل‌گرها اجازه می‌دهند تا هوک‌های پیش حذف ناهمزمان را پیاده‌سازی کنند.
اشیاء سفارشی حمایت می‌کنند finalizers شبیه به اشیاء داخلی.

می‌توانید یک finalizer را به یک شیء سفارشی اضافه کنید، مانند:

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  finalizers:
  - stable.example.com/finalizer
```

شناسه‌های finalizer سفارشی شامل یک نام دامنه، یک خط اسلش و نام
finalizer است. هر کنترل‌گری می‌تواند یک finalizer را به لیست finalizerهای هر شیء اضافه کند.

اولین درخواست حذف بر روی یک شیء با finalizers یک مقدار برای
`metadata.deletionTimestamp` تنظیم می‌کند، اما آن را حذف نمی‌کند. هنگامی که این مقدار تنظیم شود،
موارد در لیست `finalizers` فقط می‌توانند حذف شوند. در حالی که هر finalizers باقی‌مانده باشد، همچنین
امکان اجباری حذف یک شیء وجود ندارد.

زمانی که فیلد `metadata.deletionTimestamp` تنظیم شده است، کنترل‌گرهایی که شیء را نظارت می‌کنند، هر finalizer را اجرا می‌کنند و پس از انجام کار، finalizer را از لیست حذف می‌کنند. مسئولیت هر کنترل‌گر است که finalizer خود را از لیست حذف کند.

مقدار `metadata.deletionGracePeriodSeconds` کنترل می‌کند فاصله زمانی بین به‌روزرسانی‌های پرس و جو.

هنگامی که لیست finalizers خالی شود، به این معنی است که تمام finalizers اجرا شده‌اند، منبع توسط Kubernetes حذف می‌شود.

### اعتبارسنجی

منابع سفارشی از طریق
[طرح‌های اعتبارسنجی OpenAPI v3](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#schemaObject)
توسط x-kubernetes-validations در هنگام فعال‌سازی قوانین اعتبارسنجی، و شما
می‌توانید با استفاده از
[admission webhooks](/docs/reference/access-authn-authz/admission-controllers/#validatingadmissionwebhook)
اعتبارسنجی اضافی اضافه کنید.

به علاوه، محدودیت‌های زیر بر روی طرح اعمال می‌شود:

- این فیلدها نمی‌توانند تنظیم شوند:

  - `definitions`,
  - `dependencies`,
  - `deprecated`,
  - `discriminator`,
  - `id`,
  - `patternProperties`,
  - `readOnly`,
  - `writeOnly`,
  - `xml`,
  - `$ref`.

- فیلد `uniqueItems` نمی‌تواند به `true` تنظیم شود.
- فیلد `additionalProperties` نمی‌تواند به `false` تنظیم شود.
- فیلد `additionalProperties` انحصاری است با `properties`.

پسوند `x-kubernetes-validations` می‌تواند برای اعتبارسنجی منابع سفارشی با استفاده از
[Common Expression Language (CEL)](https://github.com/google/cel-spec) اعتبارسنجی شود هنگامی که
قوانین اعتبارسنجی فعال‌سازی شده و طرح CustomResourceDefinition یک
[طرح ساختاری](#specifying-a-structural-schema) است.

برای محدودیت‌ها و ویژگی‌های CustomResourceDefinition به بخش
[طرح‌های ساختاری](#specifying-a-structural-schema) مراجعه کنید.

طرح در CustomResourceDefinition تعریف شده است. در مثال زیر،
CustomResourceDefinition اعمال می‌شود:
اعتبارسنجی‌های زیر را در شیء سفارشی.

- `spec.cronSpec` باید یک رشته باشد و باید به صورت توصیف شده توسط عبارت منظم باشد.
- `spec.replicas` باید یک عدد صحیح باشد و باید مقدار حداقل 1 و حداکثر 10 را داشته باشد.

CustomResourceDefinition را در `resourcedefinition.yaml` ذخیره کنید:

```

yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        # openAPIV3Schema is the schema for validating custom objects.
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                  pattern: '^(\d+|\*)(/\d+)?(\s+(\d+|\*)(/\d+)?){4}$'
                image:
                  type: string
                replicas:
                  type: integer
                  minimum: 1
                  maximum: 10
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
    - ct
```

و آن را ایجاد کنید:

```shell
kubectl apply -f resourcedefinition.yaml
```

درخواست ایجاد شیء سفارشی از نوع CronTab رد می‌شود اگر مقادیر نامعتبری در فیلدهای آن باشد.
در مثال زیر، شیء سفارشی حاوی مقادیر نامعتبر است:

- `spec.cronSpec` با عبارت منظم مطابقت ندارد.
- `spec.replicas` بیشتر از 10 است.

اگر شما YAML زیر را در `my-crontab.yaml` ذخیره کنید:

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * *"
  image: my-awesome-cron-image
  replicas: 15
```

و سعی کنید آن را ایجاد کنید:

```shell
kubectl apply -f my-crontab.yaml
```

در این صورت یک خطای دریافت می‌کنید:

```console
The CronTab "my-new-cron-object" is invalid: []: Invalid value: map[string]interface {}{"apiVersion":"stable.example.com/v1", "kind":"CronTab", "metadata":map[string]interface {}{"name":"my-new-cron-object", "namespace":"default", "deletionTimestamp":interface {}(nil), "deletionGracePeriodSeconds":(*int64)(nil), "creationTimestamp":"2017-09-05T05:20:07Z", "uid":"e14d79e7-91f9-11e7-a598-f0761cb232d1", "clusterName":""}, "spec":map[string]interface {}{"cronSpec":"* * * *", "image":"my-awesome-cron-image", "replicas":15}}:
validation failure list:
spec.cronSpec in body should match '^(\d+|\*)(/\d+)?(\s+(\d+|\*)(/\d+)?){4}$'
spec.replicas in body should be less than or equal to 10
```

اگر فیلدها مقادیر معتبر داشته باشند، درخواست ایجاد شیء قبول می‌شود.

YAML زیر را در `my-crontab.yaml` ذخیره کرده و ایجاد کنید:

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
  replicas: 5
```

و آن را ایجاد کنید:

```shell
kubectl apply -f my-crontab.yaml
crontab "my-new-cron-object" created
```
### ارتقاء اعتبارسنجی

{{< feature-state feature_gate_name="CRDValidationRatcheting" >}}

در صورت استفاده از نسخه Kubernetes قدیمی‌تر از v1.30، شما باید به طور صریح
ویژگی `CRDValidationRatcheting`
[feature gate](/docs/reference/command-line-tools-reference/feature-gates/) را فعال کنید
تا بتوانید از این رفتار استفاده کنید، که سپس برای همه CustomResourceDefinitions در
خوشه شما اعمال می‌شود.

با فعال کردن این ویژگی، Kubernetes اجازه می‌دهد که بروزرسانی‌ها به منابعی که بعد از بروزرسانی
معتبر نیستند را پذیرفته کند، شرط آنکه هر بخش از منبع که در اثر عملیات بروزرسانی اعتبارسنجی
نشد، تغییر نکرده باشد. به عبارت دیگر، هر بخش نامعتبر از منبع که هنوز هم نامعتبر باقی مانده
باشد، قبلاً نادرست بوده است.
شما نمی‌توانید از این مکانیزم برای بروزرسانی یک منبع معتبر استفاده کنید که تبدیل به نامعتبر شود.

این ویژگی به نویسندگان CRD اجازه می‌دهد که به اطمینان از اضافه کردن اعتبارسنجی‌های جدید به
طرح OpenAPIV3 در شرایط خاص اطمینان حاصل کنند. کاربران می‌توانند به طور ایمن به طرح جدید
به روز رسانی کنند بدون افزایش نسخه شی یا شکستن جریان کاری.

با اینکه اکثر اعتبارسنجی‌های قرار داده شده در طرح OpenAPIV3 یک CRD را پشتیبانی می‌کند
از رچتینگ پشتیبانی می‌کند، اما چند استثنا وجود دارد. اعتبارسنجی‌های زیر از رچتینگ
تحت پیاده‌سازی در Kubernetes
{{< skew currentVersion >}} پشتیبانی نمی‌شود و اگر نقض شوند، همچنان خطاها به طور
معمول ارائه خواهند داد:

- Quantors
  - `allOf`
  - `oneOf`
  - `anyOf`
  - `not`
  -  هر اعتبارسنجی در نسلی از این فیلدها
- `x-kubernetes-validations`
  برای Kubernetes 1.28، CRD از اعتبارسنجی معمولی (x-kubernetes-validations) نادیده
  گرفته شده است. آغاز شده است با افزایشی که 1.29 دارد، فقط به طور معمول رچتینگ می‌شود
  اگر ارجاع به `oldSelf` نداشته باشد.

  قوانین ترانزیشن هیچ وقت رچتینگ نمی‌شوند: تنها خطاهای ارتفاع شده توسط قوانینی که
  ندارند `oldSelf` به صورت خودکار رچتینگ خواهند شد اگر مقادیر آنها تغییر نکند.

  برای نوشتن منطق رچتینگ مخصوص CEL، چک کنید [optionalOldSelf](#field-optional-oldself).
- `x-kubernetes-list-type`
  خطاهای ناشی از تغییر نوع لیست زیرمجموعه از یک زیرمجموعه لیست همیشه رچتینگ نخواهد
  شد. برای مثال افزودن `set` روی یک لیست با تکرارها همیشه نتیجه در خطا خواهد داشت.
- `x-kubernetes-map-keys`
  خطاهای ناشی از تغییر کلیدهای نقشه یک طرح لیست رچتینگ نخواهد شد.
- `required`
  خطاهای ناشی از تغییر لیستی از فیلدهای مورد نیاز رچتینگ نخواهد شد.
- `properties`
  اضافه کردن/حذف/تغییر نام‌های ویژگی‌ها رچتینگ نمی‌شود، اما تغییرات در اعتبارسنجی‌ها
  در هر زیرمجموعه ویژگی‌ها و زیرمجموعه‌ها ممکن است رچتینگ شود اگر نام ویژگی یکسان
  بماند.
- `additionalProperties`
  حذف اعتبارسنجی `additionalProperties` مشخص شده پیشین رچتینگ نخواهد شد.
- `metadata`
  خطاهایی که از اعتبارسنجی داخلی Kubernetes برای یک شی‌ء `metadata` حاصل می‌شود
  رچتینگ نمی‌شود (مانند نام شیء، یا کاراکترها در مقدار برچسب). اگر شما قوانین اضافی خود را
  برای متادیتا یک منبع سفارشی مشخص کنید، این اعتبارسنجی اضافی رچتینگ خواهد شد.

### قوانین اعتبارسنجی

{{< feature-state state="stable" for_k8s_version="v1.29" >}}

قوانین اعتبارسنجی از [Common Expression Language (CEL)](https://github.com/google/cel-spec)
برای اعتبارسنجی ارزش‌های منابع سفارشی استفاده می‌کنند. قوانین اعتبارسنجی در
طرح CustomResourceDefinition با استفاده از توسعه `x-kubernetes-validations` اضافه می‌شوند.

قاعده در محدوده موقعیت توسعه `x-kubernetes-validations` در طرح است.
و متغیر `self` در عبارت CEL به مقدار بسته می‌شود.

تمام قوانین اعتبارسنجی به شی کنونی محدود شده‌اند: هیچ قوانین اعتبارسنجی

 متقابل یا حالتی پشتیبانی
نمی‌شوند.

به عنوان مثال:

```none
  ...
  openAPIV3Schema:
    type: object
    properties:
      spec:
        type: object
        x-kubernetes-validations:
          - rule: "self.minReplicas <= self.replicas"
            message: "replicas should be greater than or equal to minReplicas."
          - rule: "self.replicas <= self.maxReplicas"
            message: "replicas should be smaller than or equal to maxReplicas."
        properties:
          ...
          minReplicas:
            type: integer
          replicas:
            type: integer
          maxReplicas:
            type: integer
        required:
          - minReplicas
          - replicas
          - maxReplicas
```

یک درخواست برای ایجاد این منبع سفارشی را رد خواهد کرد:

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  minReplicas: 0
  replicas: 20
  maxReplicas: 10
```

با پاسخ:

```
The CronTab "my-new-cron-object" is invalid:
* spec: Invalid value: map[string]interface {}{"maxReplicas":10, "minReplicas":0, "replicas":20}: replicas should be smaller than or equal to maxReplicas.
```

`x-kubernetes-validations` ممکن است چندین قانون داشته باشد.
`rule` زیر `x-kubernetes-validations` نشان دهنده عبارت است که توسط CEL ارزیابی می‌شود.
`message` نمایش پیام هنگامی که اعتبارسنجی ناموفق است. اگر پیام تنظیم نشود،
پاسخ بالا به صورت زیر خواهد بود:

```
The CronTab "my-new-cron-object" is invalid:
* spec: Invalid value: map[string]interface {}{"maxReplicas":10, "minReplicas":0, "replicas":20}: failed rule: self.replicas <= self.maxReplicas
```

{{< note >}}
شما می‌توانید به سرعت عبارات CEL را در [CEL Playground](https://playcel.undistro.io)
تست کنید.
{{< /note >}}

قوانین اعتبارسنجی هنگام ایجاد/به روزرسانی CRDها کامپایل می‌شوند.
درخواست ایجاد/به روزرسانی CRD با شکست کامپایل قوانین اعتبارسنجی شکست خواهد خورد.
فرآیند کامپایل شامل بررسی نوع نیز می‌شود.

شکست کامپایل:

- `no_matching_overload`: این تابع هیچ بارگزاری متناظر با انواع آرگومان‌ها ندارد.

  به عنوان مثال، یک قاعده مانند `self == true` در برابر یک فیلد از نوع عدد صحیح خطا خواهد گرفت:

  ```none
  Invalid value: apiextensions.ValidationRule{Rule:"self == true", Message:""}: compilation failed: ERROR: \<input>:1:6: found no matching overload for '_==_' applied to '(int, bool)'
  ```

- `no_such_field`: شامل فیلد مورد نظر نمی‌شود.

  به عنوان مثال، یک قاعده مانند `self.nonExistingField > 0` در برابر یک فیلد غیر موجود خطا
  زیر را به همراه خواهد داشت:

  ```none
  Invalid value: apiextensions.ValidationRule{Rule:"self.nonExistingField > 0", Message:""}: compilation failed: ERROR: \<input>:1:5: undefined field 'nonExistingField'
  ```

- `invalid argument`: آرگومان نامعتبر به ماکروها.

  به عنوان مثال، یک قاعده مانند `has(self)` خطا خواهد گرفت:

  ```none
  Invalid value: apiextensions.ValidationRule{Rule:"has(self)", Message:""}: compilation failed: ERROR: <input>:1:4: invalid argument to has() macro
  ```
### نمونه‌های قوانین اعتبارسنجی:

| قانون                                                                                      | هدف                                                                                       |
| ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `self.minReplicas <= self.replicas && self.replicas <= self.maxReplicas`                    | اعتبارسنجی که سه فیلد تعریف کننده تعداد کپی‌ها به درستی مرتب شده‌اند                  |
| `'Available' in self.stateCounts`                                                         | اعتبارسنجی بررسی می‌کند که یک ورودی با کلید 'Available' در یک نقشه وجود دارد         |
| `(size(self.list1) == 0) != (size(self.list2) == 0)`                                        | اعتبارسنجی که یکی از دو لیست غیر خالی است، اما هر دو نیز خالی نیستند                 |
| <code>!('MY_KEY' in self.map1) &#124;&#124; self['MY_KEY'].matches('^[a-zA-Z]*$')</code>  | اعتبارسنجی ارزیابی مقدار یک کلید خاص در یک نقشه، اگر در نقشه باشد                     |
| `self.envars.filter(e, e.name == 'MY_ENV').all(e, e.value.matches('^[a-zA-Z]*$')`          | اعتبارسنجی فیلد 'value' یک ورودی listMap با فیلد کلیدی 'name' برابر 'MY_ENV'          |
| `has(self.expired) && self.created + self.ttl < self.expired`                             | اعتبارسنجی که تاریخ 'منقضی' پس از تاریخ 'ساخت' به علاوه 'مدت زمان زندگی' باشد          |
| `self.health.startsWith('ok')`                                                            | اعتبارسنجی که یک رشته 'سلامتی' دارای پیشوند 'ok' باشد                                   |
| `self.widgets.exists(w, w.key == 'x' && w.foo < 10)`                                      | اعتبارسنجی که ویژگی 'foo' یک مورد listMap با کلید 'x' کمتر از 10 باشد                    |
| `type(self) == string ? self == '100%' : self == 1000`                                    | اعتبارسنجی یک فیلد int-or-string برای هر دو حالت int و string                              |
| `self.metadata.name.startsWith(self.prefix)`                                              | اعتبارسنجی که نام یک شیء دارای پیشوند مقدار دیگری باشد                                       |
| `self.set1.all(e, !(e in self.set2))`                                                     | اعتبارسنجی که دو listSet متمایز هستند                                                            |
| `size(self.names) == size(self.details) && self.names.all(n, n in self.details)`           | اعتبارسنجی که نقشه 'details' به کلیدهای لیست 'names' متصل شده باشد                           |
| `size(self.clusters.filter(c, c.name == self.primary)) == 1`                              | اعتبارسنجی که ویژگی 'اصلی' یک بار و فقط یک بار در 'clusters' listMap باشد                      |

مراجعه: [پشتیبانی ارزیابی در CEL](https://github.com/google/cel-spec/blob/v0.6.0/doc/langdef.md#evaluation)

- اگر قانون به ریشه یک منبع محدود شود، ممکن است انتخاب فیلد شود به هر فیلدی
  اعلامی در طرح OpenAPIv3 شما همچنین، انتخاب فیلد در 'spec' و 'status' در
  به یک عبارت نیز انتخاب شده است:

  ```none
    ...
    openAPIV3Schema:
      type: object
      x-kubernetes-validations:
        - rule: "self.status.availableReplicas >= self.spec.minReplicas"
      properties:
          spec:
            type: object
            properties:
              minReplicas:
                type: integer
              ...
          status:
            type: object
            properties:
              availableReplicas:
                type: integer
  ```

- اگر قانون به یک شیء با ویژگی‌ها محدود شود، ویژگی‌های قابل دسترسی از شیء قابل انتخاب هستند
  از طریق `self.field` و حضور فیلد می‌تواند از طریق `has(self.field)` بررسی شود. فیلدهای با مقدار نال به عنوان
  فیلدهای غایب در عبارات CEL تلقی می‌شوند.

  ```none
    ...
    openAPIV3Schema:
      type: object
      properties:
        spec:
          type: object
          x-kubernetes-validations:
            - rule: "has(self.foo)"
          properties:
            ...
            foo:
              type: integer
  ```

- اگر قانون به یک شیء با additionalProperties (به عبارتی نقشه) محدود شود، مقدار نقشه
  قابل دسترسی از طریق `self[mapKey]`، حاوی نقشه می‌تواند از طریق `mapKey in self` بررسی شود و تمام
  ورودی‌های نقشه از طریق ماکروها و توابع CEL مانند `self.all(...)` قابل دسترسی هستند.

  ```none
    ...
    openAPIV3Schema:
      type: object
      properties:
        spec:
          type: object
          x-kubernetes-validations:
            - rule: "self['xyz'].foo > 0"
          additionalProperties:
            ...
            type: object
            properties:
              foo:
                type: integer
  ```

- اگر قانون به یک آرایه محدود شود، عناصر آرایه از طریق `self[i]` قابل دسترسی هستند و
  همچنین از طریق ماکروها و توابع.

  ```none
    ...
    openAPIV3Schema:
      type: object
      properties:
        ...
        foo:
          type: array
          x-kubernetes-validations:
            - rule: "size(self) == 1"
          items:
            type: string
  ```

- اگر قانون به یک مقدار اسکالر محدود شود، `self` به مقدار اسکالر متصل می‌شود.

  ```none
    ...
    openAPIV3Schema:
      type: object
      properties:
        spec:
          type: object
          properties:
            ...
            foo:
              type: integer
              x-kubernetes-validations:
              - rule: "self > 0"
  ```

مثال‌ها:

| نوع قانون محدود به فیلد    | نمونه قانون             |
| -----------------------| -----------------------|
| شیء ریشه            | `self.status.actual <= self.spec.maxDesired` |
| نقشه شیء             | `self.components['Widget'].priority < 10` |
| لیست اعداد صحیح       | `self.values.all(value, value >= 0 && value < 100)` |
| رشته                 | `self.startsWith('kube')` |

`apiVersion`، `kind`، `metadata.name` و `metadata.generateName` هم

واره از
ریشه شیء و از هر موارد `x-kubernetes-embedded-resource` نیز دسترسی پذیر هستند. خود از هم متفاوت
خواهند بود:

  ```none
    ...
    openAPIV3Schema:
      type: object
      x-kubernetes-validations:
        - rule: "self.status.availableReplicas >= self.spec.minReplicas"
      properties:
          spec:
            type: object
            properties:
              minReplicas:
                type: integer
              ...
          status:
            type: object
            properties:
              availableReplicas:
                type: integer
  ```

- If the Rule is scoped to an object with additionalProperties (i.e. a map) the value of the map
  are accessible via `self[mapKey]`, map containment can be checked via `mapKey in self` and all
  entries of the map are accessible via CEL macros and functions such as `self.all(...)`
### جدول مطابقت انواع تعریف‌شده بین OpenAPIv3 و CEL

| نوع OpenAPIv3                                     | نوع CEL                                                                                                                     |
| -------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| شیء با خصوصیت‌ها                                 | object / "message type"                                                                                                      |
| شیء با خصوصیات اضافی                            | map                                                                                                                          |
| شیء با نوع x-kubernetes-embedded-type             | object / "message type"، 'apiVersion'، 'kind'، 'metadata.name' و 'metadata.generateName' به صورت ضمنی در طرح شامل می‌شوند |
| شیء با نوع x-kubernetes-preserve-unknown-fields   | object / "message type"، فیلدهای ناشناخته در عبارت CEL قابل دسترس نیستند                                           |
| x-kubernetes-int-or-string                         | شیء پویا که می‌تواند یک int یا یک string باشد، `type(value)` برای بررسی نوع استفاده می‌شود                           |
| آرایه                                             | list                                                                                                                         |
| آرایه با نوع x-kubernetes-list-type=map           | list با تضمین برابری و یکتایی بر اساس نقشه                                                                                   |
| آرایه با نوع x-kubernetes-list-type=set           | list با تضمین برابری ورودی یکتایی بر اساس مجموعه                                                                                 |
| بولین                                             | boolean                                                                                                                      |
| عدد (همه فرمت‌ها)                                 | double                                                                                                                       |
| عدد صحیح (همه فرمت‌ها)                           | int (64)                                                                                                                     |
| مقدار null                                        | null_type                                                                                                                    |
| رشته                                              | string                                                                                                                       |
| رشته با فرمت byte (base64 encoded)                | bytes                                                                                                                        |
| رشته با فرمت date                                 | timestamp (google.protobuf.Timestamp)                                                                                        |
| رشته با فرمت datetime                             | timestamp (google.protobuf.Timestamp)                                                                                        |
| رشته با فرمت duration                             | duration (google.protobuf.Duration)                                                                                          |

منابع:

- [CEL types](https://github.com/google/cel-spec/blob/v0.6.0/doc/langdef.md#values)
- [OpenAPI types](https://swagger.io/specification/#data-types)
- [Kubernetes Structural Schemas](/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#specifying-a-structural-schema)

### فیلد messageExpression

شبیه به فیلد `message` که رشته مورد استفاده برای عدم موفقیت یک قاعده اعتبارسنجی را تعریف می‌کند، `messageExpression` به شما امکان می‌دهد که از یک عبارت CEL برای ساخت رشته پیام استفاده کنید. این به شما اجازه می‌دهد تا اطلاعات توضیحی بیشتری را به پیام عدم موفقیت اعتبارسنجی اضافه کنید. `messageExpression` باید به یک رشته ارزیابی شود و ممکن است از متغیرهای مشابهی استفاده کند که برای فیلد `rule` در دسترس هستند. به عنوان مثال:

```yaml
x-kubernetes-validations:
- rule: "self.x <= self.maxLimit"
  messageExpression: '"x exceeded max limit of " + string(self.maxLimit)'
```

لطفاً توجه داشته باشید که اتصال رشته CEL (`+` عامل) به طور خودکار به رشته تبدیل نمی‌شود. اگر
شما یک مقدار اسکالر نیستید، از تابع `string(<value>)` برای تبدیل اسکالر به رشته استفاده کنید
مانند مثال بالا.

`messageExpression` باید به یک رشته ارزیابی شود و این در حالی که CRD نوشته می‌شود بررسی می‌شود. توجه کنید که ممکن است
به تنظیم `message` و `messageExpression` برای همان قاعده نیاز داشته باشید، و اگر هر دو حاضر باشند، `messageExpression`
استفاده خواهد شد. با این حال، اگر `messageExpression` به خطایی ارزیابی شود، رشته تعریف شده در `message`
به جای آن استفاده خواهد شد، و خطای `messageExpression` ثبت خواهد شد. این پشتیبانی پشتیبانی می‌شود اگر
عبارت CEL تعریف شده در `messageExpression` رشته خالی یا شامل خطوط
شکسته است.

اگر یکی از شرایط فوق برآورده شود و هیچ `message` مشخص نشده باشد، پیام عدم موفقیت پیش‌فرض
استفاده خواهد شد.

`messageExpression` یک عبارت CEL است، بنابراین محدودیت‌های فهرست شده در [استفاده از منابع توابع ارزیابی](#resource-use-by-validation-functions) را دارا است. اگر ارزیابی به دلیل محدودیت منابع
در حین اجرای `messageExpression` متوقف شود، دیگر قوانین اعتبارسنجی اجرا نخواهند شد.

تنظیم `messageExpression` اختیاری است.

### فیلد message

اگر می‌خواهید یک پیام استاتیک تعیین کنید، می‌توانید به جای `messageExpression`، `message` را ارائه دهید.
مقدار `message` به عنوان یک رشته خطای ناشناخته استفاده می‌شود اگر اعتبارسنجی ناموفق باشد.

تنظیم `message` اختیاری است.

### فیلد reason

می‌توانید یک دلیل خطای اعتبارسنجی قابل خواندن توسط ماشین را به یک `validation` اضافه کنید، برای بازگشت
هرگاه یک درخواست این قاعده اعتبارسنجی ناموفق باشد.

به عنوان مثال:

```yaml
x-kubernetes-validations:
- rule: "self.x <= self.maxLimit"
  reason: "FieldValueInvalid"
```

کد وضعیت HTTP که به خواننده برمی‌گردد مطابقت دارد با دلیل اولین قاعده اعتبارسنجی ناموفق.
دلایل پشتیبانی شده در حال حاضر: "FieldValueInvalid"، "FieldValueForbidden"، "FieldValueRequired"، "FieldValueDuplicate".
اگر تنظیم نشود یا دلایل ناموجود

، به طور پیش‌فرض برای استفاده از "FieldValueInvalid".

تنظیم `reason` اختیاری است.

### فیلد fieldPath

می‌توانید مسیر فیلد را که هنگام عدم موفقیت اعتبارسنجی برمی‌گردد مشخص کنید.

به عنوان مثال:

```yaml
x-kubernetes-validations:
- rule: "self.foo.test.x <= self.maxLimit"
  fieldPath: ".foo.test.x"
```

در مثال بالا، اعتبارسنجی مقادیر فیلد `x` باید کمتر از مقدار `maxLimit` باشد.
اگر `fieldPath` مشخص نشود، هنگام عدم موفقیت اعتبارسنجی، `fieldPath` به طور پیش‌فرض محلی از `self` بازمی‌گردد.
با `fieldPath` مشخص شده، خطای برگشتی به طور مناسب به مکان فیلد `x` اشاره خواهد کرد.

مقدار `fieldPath` باید یک مسیر JSON نسبی باشد که در محدوده این افزونه x-kubernetes-validations در طرح قرار دارد. 
به علاوه، باید به یک فیلد موجود در طرح اشاره کند.
به عنوان مثال زمانی که اعتبارسنجی بررسی می‌کند که آیا یک ویژگی خاص `foo` تحت یک نقشه `testMap` وجود دارد، شما می‌توانید
`fieldPath` را به `".testMap.foo"` یا `.testMap['foo']'`.
اگر اعتبارسنجی نیاز به بررسی ویژگی‌های یکتا در دو لیست دارد، می‌توانید `fieldPath` را به هر دو از لیست‌ها تنظیم کنید.
به عنوان مثال، می‌تواند تنظیم شود به `.testList1` یا `.testList2`.
این عملیات فرزند را برای اشاره به یک فیلد موجود فعلی پشتیبانی می‌کند.
برای اطلاعات بیشتر به [پشتیبانی JSONPath در Kubernetes](/docs/reference/kubectl/jsonpath/) مراجعه کنید.
فیلد `fieldPath` اندیس‌گذاری آرایه‌ها را به صورت عددی پشتیبانی نمی‌کند.

تنظیم `fieldPath` اختیاری است.

### فیلد optionalOldSelf

{{< feature-state feature_gate_name="CRDValidationRatcheting" >}}

اگر خوشه شما از [CRD validation ratcheting](#validation-ratcheting) فعال نباشد،
API CustomResourceDefinition شامل این فیلد نخواهد بود، و سعی بر تنظیم آن ممکن است
به خطایی منجر شود.

فیلد `optionalOldSelf` یک فیلد منطقی بولی است که رفتار [قواعد انتقال](#transition-rules) توصیف شده
را تغییر می‌دهد. به طور معمول، یک قاعده انتقال نمی‌تواند ارزیابی شود اگر `oldSelf` قابل تعیین نباشد:
هنگام ایجاد شیء یا زمانی که یک مقدار جدید در یک به‌روزرسانی معرفی می‌شود.

اگر `optionalOldSelf` به true تنظیم شود، آنگاه قواعد انتقال همیشه ارزیابی خواهند شد و نوع `oldSelf` به نوع [`Optional`](https://pkg.go.dev/github.com/google/cel-go/cel#OptionalTypes) CEL تغییر خواهد یافت.

`optionalOldSelf` در مواردی کاربردی است که نویسندگان طرح نیاز به ابزاری متناسب بیشتر از رفتار بررسی شده برای تقویت محدودیت‌های جدید، عموماً دارند
از طریق استفاده از تاثیر قوانین معرفی شده [برای](#validation-ratcheting)
مقدارهای جدید، هنوز هم اجازه داده می‌شود که مقادیر قدیمی به عنوان "مادرسازی شده" یا "نوسان" با استفاده از اعتبارسنجی قدیمی شوند.

مثال استفاده:

| CEL                                     | توضیحات |
|-----------------------------------------|-------------|
| `self.foo == "foo" || (oldSelf.hasValue() && oldSelf.value().foo != "foo")` | قاعده نوسانی. یک‌بار مقدار به "foo" تنظیم شود، باید فقط مقدار foo باقی بماند. اما اگر قبل از اینکه محدودیت "foo" معرفی شود، وجود داشته باشد، ممکن است از هر مقداری استفاده شود. |
| [oldSelf.orValue(""), self].all(x, ["OldCase1", "OldCase2"].exists(case, x == case)) || ["NewCase1", "NewCase2"].exists(case, self == case) || ["NewCase"].has(self)` | "اعتبارسنجی برای موارد شناخته شده enum در صورت حذف اگر oldSelf از آنها استفاده می‌کرد" |
| oldSelf.optMap(o, o.size()).orValue(0) < 4 || self.size() >= 4 | اعتبارسنجی نوسانی از حداقل تازه افزایش یافته نقشه یا اندازه لیست |

### توابع اعتبارسنجی {#available-validation-functions}

توابع موجود شامل:

- توابع استاندارد CEL، تعریف شده در [لیست تعریفات استاندارد](https://github.com/google/cel-spec/blob/v0.7.0/doc/langdef.md#list-of-standard-definitions)
- ماکروهای استاندارد CEL [ماکروها](https://github.com/google/cel-spec/blob/v0.7.0/doc/langdef.md#macros)
- کتابخانه [تابع رشته گسترده CEL](https://pkg.go.dev/github.com/google/cel-go@v0.11.2/ext#Strings)
- کتابخانه افزونه Kubernetes [کتابخانه CEL](https://pkg.go.dev/k8s.io/apiextensions-apiserver@v0.24.0/pkg/apiserver/schema/cel/library#pkg-functions)

### قواعد انتقال

قوانینی که شامل یک عبارت اشاره‌گر به شناسه `oldSelf` باشند به‌طور ضمنی به عنوان _قواعد انتقال_ در نظر گرفته می‌شوند. قواعد انتقال به نویسندگان طرح اجازه می‌دهند تا از برخی انتقال‌ها بین دو حالت معتبر جلوگیری کنند. برای مثال:

```yaml
type: string
enum: ["low", "medium", "high"]
x-kubernetes-validations:
- rule: "!(self == 'high' && oldSelf == 'low') && !(self == 'low' && oldSelf == 'high')"
  message: cannot transition directly between 'low' and 'high'
```

بر خلاف قوانین دیگر، قوانین انتقال فقط به عملیات‌هایی اعمال می‌شوند که معیارهای زیر را دارند:

- عملیات یک شیء موجود را به‌روزرسانی می‌کند. قوانین انتقال هرگز به عملیات ایجاد اعمال نمی‌شوند.

- هر دو مقدار قدیمی و جدید وجود دارند. همچنان می‌توان بررسی کرد که آیا یک مقدار اضافه یا حذف شده است با قرار دادن یک قانون انتقال در گره والد. قوانین انتقال هرگز به ایجاد منابع سفارشی اعمال نمی‌شوند. وقتی که در یک فیلد اختیاری قرار می‌گیرد، یک قانون انتقال به عملیات به‌روزرسانی که فیلد را تنظیم یا لغو می‌کند اعمال نخواهد شد.

- مسیر به گره طرح که توسط یک قانون انتقال اعتبارسنجی می‌شود باید به یک گره حل شود که قابل مقایسه بین شیء قدیمی و شیء جدید باشد. برای مثال، موارد لیست و نوادگان آنها (`spec.foo[10].bar`) به‌طور ضروری نمی‌توانند بین یک شیء موجود و یک به‌روزرسانی بعدی به همان شیء مقایسه شوند.

در نوشتن CRD خطاهایی تولید خواهند شد اگر یک گره طرح شامل یک قانون انتقال باشد که هرگز قابل اعمال نباشد، مانند "*path*: update rule *rule* cannot be set on schema because the schema or its parent schema is not mergeable".

قوانین انتقال فقط در _بخش‌های قابل همبستگی_ یک طرح مجاز هستند. 
یک بخش از طرح زمانی قابل همبستگی است که تمام طرح‌های والد `array` از نوع `x-kubernetes-list-type=map` باشند؛ 
هر گونه `set` یا `atomic` والد `array` همبستگی بدون ابهام `self` با `oldSelf` را غیرممکن می‌کند.

در اینجا چند مثال برای قوانین انتقال آورده شده است:

| مورد استفاده                                                      | قانون
| --------                                                          | --------
| عدم تغییرپذیری                                                    | `self.foo == oldSelf.foo`
| جلوگیری از تغییر/حذف بعد از تخصیص                                 | `oldSelf != 'bar' \|\| self == 'bar'` یا `!has(oldSelf.field) \|\| has(self.field)`
| مجموعه فقط اضافه‌شدنی                                              | `self.all(element, element in oldSelf)`
| اگر مقدار قبلی X بود، مقدار جدید فقط می‌تواند A یا B باشد، نه Y یا Z | `oldSelf != 'X' \|\| self in ['A', 'B']`
| شمارنده‌های یکنواخت (غیرکاهشی)                                    | `self >= oldSelf`

### استفاده از منابع توسط توابع اعتبارسنجی

هنگامی که یک CustomResourceDefinition ایجاد یا به‌روزرسانی می‌کنید که از قوانین اعتبارسنجی استفاده می‌کند،
سرور API تأثیر احتمالی اجرای آن قوانین اعتبارسنجی را بررسی می‌کند. اگر یک قانون 
به‌طور تخمینی بسیار پرهزینه برای اجرا باشد، سرور API عملیات ایجاد 
یا به‌روزرسانی را رد می‌کند و یک پیام خطا برمی‌گرداند.
یک سیستم مشابه در زمان اجرا استفاده می‌شود که اقداماتی که مفسر انجام می‌دهد را مشاهده می‌کند. اگر مفسر
دستورالعمل‌های زیادی را اجرا کند، اجرای قانون متوقف خواهد شد و یک خطا نتیجه می‌شود.
هر CustomResourceDefinition نیز اجازه دارد تا مقدار مشخصی از منابع را برای اتمام اجرای تمامی
قوانین اعتبارسنجی خود استفاده کند. اگر مجموع قوانین آن در زمان ایجاد به‌طور تخمینی از آن حد فراتر رود،
یک خطای اعتبارسنجی نیز رخ خواهد داد.

اگر فقط قوانین را مشخص کنید که همیشه به همان مقدار زمان نیاز دارند، احتمال مواجهه با مشکلات بودجه منابع اعتبارسنجی نخواهید داشت.
برای مثال، قانونی که تأکید می‌کند `self.foo == 1` به‌خودی‌خود هیچ 
ریسکی برای رد شدن بر روی بودجه منابع اعتبارسنجی ندارد.
اما اگر `foo` یک رشته باشد و یک قانون اعتبارسنجی تعریف کنید `self.foo.contains("someString")`، آن قانون بیشتر طول می‌کشد تا اجرا شود بسته به طول `foo`.
مثال دیگر این است که اگر `foo` یک آرایه باشد، و یک قانون اعتبارسنجی `self.foo.all(x, x > 5)` مشخص کنید.
سیستم هزینه همیشه بدترین سناریو را فرض می‌کند اگر محدودیتی بر طول `foo` مشخص نشده باشد، و این برای هر چیزی که قابل پیمایش باشد (لیست‌ها، نقشه‌ها و غیره) اتفاق می‌افتد.

به همین دلیل، به‌عنوان بهترین عمل در نظر گرفته می‌شود که از طریق `maxItems`، `maxProperties` و
`maxLength` برای هر چیزی که در یک قانون اعتبارسنجی پردازش می‌شود محدودیت قرار دهید تا از خطاهای اعتبارسنجی در هنگام تخمین هزینه جلوگیری کنید. برای مثال، با توجه به این طرح با یک قانون:

```yaml
openAPIV3Schema:
  type: object
  properties:
    foo:
      type: array
      items:
        type: string
      x-kubernetes-validations:
        - rule: "self.all(x, x.contains('a string'))"
```

سپس سرور API این قانون را به دلیل بودجه اعتبارسنجی با خطای زیر رد می‌کند:

```
spec.validation.openAPIV3Schema.properties[spec].properties[foo].x-kubernetes-validations[0].rule: Forbidden:
CEL rule exceeded budget by more than 100x (try simplifying the rule, or adding maxItems, maxProperties, and
maxLength where arrays, maps, and strings are used)
```

این رد به این دلیل است که `self.all` مستلزم فراخوانی `contains()` بر هر رشته در `foo` است،
که به نوبه خود رشته داده‌شده را بررسی می‌کند تا ببیند آیا شامل `'a string'` است. بدون محدودیت‌ها، این
یک قانون بسیار پرهزینه است.

اگر هیچ محدودیت اعتبارسنجی مشخص نکنید، هزینه تخمینی این قانون از 
حد هزینه هر قانون فراتر خواهد رفت. اما اگر محدودیت‌ها را در مکان‌های مناسب اضافه کنید، قانون مجاز خواهد بود:

```yaml
openAPIV3Schema:
  type: object
  properties:
    foo:
      type: array
      maxItems: 25
      items:
        type: string
        maxLength: 10
      x-kubernetes-validations:
        - rule: "self.all(x, x.contains('a string'))"
```

سیستم تخمین هزینه تعداد دفعاتی که قانون اجرا خواهد شد را علاوه بر
هزینه تخمینی خود قانون در نظر می‌گیرد. به عنوان مثال، قانون زیر همان هزینه تخمینی را با
مثال قبلی خواهد داشت (با وجود اینکه قانون اکنون بر روی آیتم‌های فردی آرایه تعریف شده است):

```yaml
openAPIV3Schema:
  type: object
  properties:
    foo:
      type: array
      maxItems: 25
      items:
        type: string
        x-kubernetes-validations:
          - rule: "self.contains('a string'))"
        maxLength: 10
```

اگر یک لیست داخل یک لیست یک قانون اعتبارسنجی که از `self.all` استفاده می‌کند، داشته باشد، این به‌طور قابل‌توجهی بیشتر هزینه‌بر است
نسبت به یک لیست غیر تو در تو با همان قانون. یک قانونی که بر روی یک لیست غیر تو در تو مجاز بوده ممکن است
محدودیت‌های کمتری بر روی هر دو لیست تو در تو تنظیم شود تا مجاز باشد. برای مثال، حتی بدون تنظیم محدودیت‌ها،
قانون زیر مجاز است:

```yaml
openAPIV3Schema:
  type: object
  properties:
    foo:
      type: array
      items:
        type: integer
    x-kubernetes-validations:
      - rule: "self.all(x, x == 5)"
```

اما همان قانون در طرح زیر (با اضافه شدن یک آرایه تو در تو) خطای اعتبارسنجی ایجاد می‌کند:

```yaml
openAPIV3Schema:
  type: object
  properties:
    foo:
      type: array
      items:
        type: array
        items:
          type: integer
        x-kubernetes-validations:
          - rule: "self.all(x, x == 5)"
```

این به این دلیل است که هر آیتم از `foo` خودش یک آرایه است و هر زیرآرایه به نوبه خود `self.all` را فراخوانی می‌کند.
از لیست‌ها و نقشه‌های تو در تو اگر ممکن است در جایی که از قوانین اعتبارسنجی استفاده می‌شود، اجتناب کنید.

### پیش‌فرض‌گذاری

{{< note >}}
برای استفاده از پیش‌فرض‌گذاری، CustomResourceDefinition شما باید از نسخه API `apiextensions.k8s.io/v1` استفاده کند.
{{< /note >}}

پیش‌فرض‌گذاری اجازه می‌دهد تا مقادیر پیش‌فرض در [طرح اعتبارسنجی OpenAPI v3](#validation) مشخص شود:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        # openAPIV3Schema طرح برای اعتبارسنجی اشیاء سفارشی است.
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                  pattern: '^(\d+|\*)(/\d+)?(\s+(\d+|\*)(/\د+)?){4}$'
                  default: "5 0 * * *"
                image:
                  type: string
                replicas:
                  type: integer
                  minimum: 1
                  maximum: 10
                  default: 1
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
    - ct
```

با این تنظیمات، هر دو `cronSpec` و `replicas` به صورت پیش‌فرض قرار داده می‌شوند:

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  image: my-awesome-cron-image
```

منجر به

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "5 0 * * *"
  image: my-awesome-cron-image
  replicas: 1
```

پیش‌فرض‌گذاری روی شیء اتفاق می‌افتد

* در درخواست به سرور API با استفاده از پیش‌فرض‌های نسخه درخواست،
* هنگام خواندن از etcd با استفاده از پیش‌فرض‌های نسخه ذخیره‌سازی،
* پس از پلاگین‌های پذیرش تغییر با پچ‌های غیرخالی با استفاده از پیش‌فرض‌های نسخه شیء پذیرش webhook.

پیش‌فرض‌هایی که هنگام خواندن داده‌ها از etcd اعمال می‌شوند به‌طور خودکار به etcd نوشته نمی‌شوند.
یک درخواست به‌روزرسانی از طریق API برای نگهداری این پیش‌فرض‌ها در etcd مورد نیاز است.

مقادیر پیش‌فرض باید برش داده شوند (به استثنای پیش‌فرض‌ها برای فیلدهای `metadata`) و باید
بر اساس طرح ارائه شده اعتبارسنجی شوند.

مقادیر پیش‌فرض برای فیلدهای `metadata` از گره‌های `x-kubernetes-embedded-resources: true` (یا قسمت‌هایی از
یک مقدار پیش‌فرض که شامل `metadata` می‌شوند) در هنگام ایجاد CustomResourceDefinition برش داده نمی‌شوند، بلکه
در طول مرحله برش در هنگام پردازش درخواست‌ها انجام می‌شود.

#### پیش‌فرض‌گذاری و Nullable

مقادیر null برای فیلدهایی که یا پرچم nullable را مشخص نمی‌کنند، یا به آن مقدار
`false` می‌دهند، قبل از پیش‌فرض‌گذاری برش داده خواهند شد. اگر یک پیش‌فرض وجود داشته باشد، اعمال خواهد شد. هنگامی که nullable `true` است، مقادیر null حفظ می‌شوند و پیش‌فرض‌گذاری نخواهند شد.

برای مثال، با توجه به طرح OpenAPI زیر:

```yaml
type: object
properties:
  spec:
    type: object
    properties:
      foo:
        type: string
        nullable: false
        default: "default"
      bar:
        type: string
        nullable: true
      baz:
        type: string
```

ایجاد یک شیء با مقادیر null برای `foo` و `bar` و `baz`

```yaml
spec:
  foo: null
  bar: null
  baz: null
```

منجر به

```yaml
spec:
  foo: "default"
  bar: null
```

با `foo` که برش داده شده و پیش‌فرض‌گذاری شده زیرا فیلد non-nullable است، `bar` حفظ مقدار null
به دلیل `nullable: true`، و `baz` برش داده شده زیرا فیلد non-nullable است و پیش‌فرض ندارد.

### انتشار طرح اعتبارسنجی در OpenAPI

CustomResourceDefinition [طرح‌های اعتبارسنجی OpenAPI v3](#validation) که
[ساختاری](#specifying-a-structural-schema) هستند و [برش‌دهی را فعال می‌کنند](#field-pruning) به عنوان
[OpenAPI v3](/docs/concepts/overview/kubernetes-api/#openapi-and-swagger-definitions) و
OpenAPI v2 از سرور API Kubernetes منتشر می‌شوند. توصیه می‌شود از سند OpenAPI v3 استفاده کنید
زیرا نمایشی بدون از دست دادن از طرح اعتبارسنجی OpenAPI v3 CustomResourceDefinition است
در حالی که OpenAPI v2 نمایشی با از دست دادن تبدیل می‌شود.

ابزار خط فرمان [kubectl](/docs/reference/kubectl/) طرح منتشر شده را مصرف می‌کند تا
اعتبارسنجی سمت مشتری (`kubectl create` و `kubectl apply`) را انجام دهد، توضیح طرح (`kubectl explain`)
روی منابع سفارشی. طرح منتشر شده می‌تواند برای مقاصد دیگر نیز مصرف شود، مانند تولید مشتری یا مستندسازی.

#### سازگاری با OpenAPI V2

برای سازگاری با OpenAPI V2، طرح اعتبارسنجی OpenAPI v3 به صورت از دست‌دهنده به طرح OpenAPI v2 تبدیل می‌شود. این طرح در فیلدهای `definitions` و `paths` در [مشخصات OpenAPI v2](/docs/concepts/overview/kubernetes-api/#openapi-and-swagger-definitions) نشان داده می‌شود.

تغییرات زیر در طول تبدیل برای حفظ سازگاری با نسخه قبلی kubectl 1.13 اعمال می‌شود. این تغییرات مانع از آن می‌شود که kubectl بیش از حد سخت‌گیرانه باشد و طرح‌های OpenAPI معتبر را که نمی‌فهمد، رد کند. تبدیل طرح اعتبارسنجی تعریف‌شده در CRD را تغییر نمی‌دهد و بنابراین بر [اعتبارسنجی](#validation) در سرور API تأثیری ندارد.

1. فیلدهای زیر حذف می‌شوند زیرا توسط OpenAPI v2 پشتیبانی نمی‌شوند.
   - فیلدهای `allOf`، `anyOf`، `oneOf` و `not` حذف می‌شوند.
2. اگر `nullable: true` تنظیم شده باشد، `type`، `nullable`، `items` و `properties` حذف می‌شوند زیرا OpenAPI v2 قادر به بیان nullable نیست. برای جلوگیری از رد اشیاء خوب توسط kubectl، این لازم است.

### ستون‌های اضافی چاپگر

ابزار kubectl به قالب‌بندی خروجی سمت سرور متکی است. سرور API خوشه شما تصمیم می‌گیرد که کدام ستون‌ها توسط دستور `kubectl get` نمایش داده شوند. شما می‌توانید این ستون‌ها را برای CustomResourceDefinition سفارشی کنید. مثال زیر ستون‌های `Spec`، `Replicas` و `Age` را اضافه می‌کند.

CustomResourceDefinition را در فایل `resourcedefinition.yaml` ذخیره کنید:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
    - ct
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              cronSpec:
                type: string
              image:
                type: string
              replicas:
                type: integer
    additionalPrinterColumns:
    - name: Spec
      type: string
      description: The cron spec defining the interval a CronJob is run
      jsonPath: .spec.cronSpec
    - name: Replicas
      type: integer
      description: The number of jobs launched by the CronJob
      jsonPath: .spec.replicas
    - name: Age
      type: date
      jsonPath: .metadata.creationTimestamp
```

CustomResourceDefinition را ایجاد کنید:

```shell
kubectl apply -f resourcedefinition.yaml
```

یک نمونه با استفاده از `my-crontab.yaml` از بخش قبلی ایجاد کنید.

چاپ سمت سرور را اجرا کنید:

```shell
kubectl get crontab my-new-cron-object
```

توجه داشته باشید به ستون‌های `NAME`، `SPEC`، `REPLICAS` و `AGE` در خروجی:

```
NAME                 SPEC        REPLICAS   AGE
my-new-cron-object   * * * * *   1          7s
```

{{< note >}}
ستون `NAME` ضمنی است و نیازی به تعریف آن در CustomResourceDefinition نیست.
{{< /note >}}

### انتخابگرهای فیلد

[انتخابگرهای فیلد](/docs/concepts/overview/working-with-objects/field-selectors/) به مشتریان اجازه می‌دهد منابع سفارشی را بر اساس مقدار یک یا چند فیلد منبع انتخاب کنند.

همه منابع سفارشی از انتخابگرهای فیلد `metadata.name` و `metadata.namespace` پشتیبانی می‌کنند.

فیلدهایی که در {{< glossary_tooltip term_id="CustomResourceDefinition" text="CustomResourceDefinition" >}} اعلام شده‌اند نیز می‌توانند با انتخابگرهای فیلد استفاده شوند زمانی که در فیلد `spec.versions[*].selectableFields` از {{< glossary_tooltip term_id="CustomResourceDefinition" text="CustomResourceDefinition" >}} درج شده باشند.

#### فیلدهای قابل انتخاب برای منابع سفارشی {#crd-selectable-fields}

{{< feature-state state="alpha" for_k8s_version="v1.30" >}}
{{< feature-state feature_gate_name="CustomResourceFieldSelectors" >}}

برای استفاده از این رفتار، نیاز است که [feature gate](/docs/reference/command-line-tools-reference/feature-gates/) `CustomResourceFieldSelectors` را فعال کنید که سپس برای همه CustomResourceDefinitions در خوشه شما اعمال می‌شود.

فیلد `spec.versions[*].selectableFields` از {{< glossary_tooltip term_id="CustomResourceDefinition" text="CustomResourceDefinition" >}} ممکن است برای اعلام فیلدهای دیگر در یک منبع سفارشی که می‌توانند در انتخابگرهای فیلد استفاده شوند، استفاده شود. مثال زیر فیلدهای `.spec.color` و `.spec.size` را به عنوان فیلدهای قابل انتخاب اضافه می‌کند.

CustomResourceDefinition را در فایل `shirt-resource-definition.yaml` ذخیره کنید:

{{% code_sample file="customresourcedefinition/shirt-resource-definition.yaml" %}}

CustomResourceDefinition را ایجاد کنید:

```shell
kubectl apply -f https://k8s.io/examples/customresourcedefinition/shirt-resource-definition.yaml
```

چند پیراهن را با ویرایش `shirt-resources.yaml` تعریف کنید؛ برای مثال:

{{% code_sample file="customresourcedefinition/shirt-resources.yaml" %}}

منابع سفارشی را ایجاد کنید:

```shell
kubectl apply -f https://k8s.io/examples/customresourcedefinition/shirt-resources.yaml
```

همه منابع را دریافت کنید:

```shell
kubectl get shirts.stable.example.com
```

خروجی:

```
NAME       COLOR  SIZE
example1   blue   S
example2   blue   M
example3   green  M
```

پیراهن‌های آبی را بازیابی کنید (پیراهن‌هایی با `color` آبی):

```shell
kubectl get shirts.stable.example.com --field-selector spec.color=blue
```

باید خروجی بدهد:

```
NAME       COLOR  SIZE
example1   blue   S
example2   blue   M
```

فقط منابع با `color` سبز و `size` M را دریافت کنید:

```shell
kubectl get shirts.stable.example.com --field-selector spec.color=green,spec.size=M
```

باید خروجی بدهد:

```
NAME       COLOR  SIZE
example2   blue   M
```

#### اولویت

هر ستون شامل فیلد `priority` است. در حال حاضر، اولویت بین ستون‌های نمایش‌داده‌شده در نمای استاندارد یا نمای گسترده (با استفاده از فلگ `-o wide`) تفاوت ایجاد می‌کند.

- ستون‌هایی با اولویت `0` در نمای استاندارد نمایش داده می‌شوند.
- ستون‌هایی با اولویت بیشتر از `0` فقط در نمای گسترده نمایش داده می‌شوند.

#### نوع

فیلد `type` یک ستون می‌تواند یکی از موارد زیر باشد (مقایسه با [انواع داده‌های OpenAPI v3](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#dataTypes)):

- `integer` – اعداد غیر اعشاری
- `number` – اعداد اعشاری
- `string` – رشته‌ها
- `boolean` – `true` یا `false`
- `date` – به صورت زمانی که از این مهر زمانی گذشته است نمایش داده می‌شود.

اگر مقدار داخل یک CustomResource با نوع مشخص‌شده برای ستون مطابقت نداشته باشد، مقدار حذف می‌شود. از اعتبارسنجی CustomResource برای اطمینان از درستی نوع مقادیر استفاده کنید.

#### قالب

فیلد `format` یک ستون می‌تواند یکی از موارد زیر باشد:

- `int32`
- `int64`
- `float`
- `double`
- `byte`
- `date`
- `date-time`
- `password`

قالب ستون سبک مورد استفاده هنگام چاپ مقدار توسط `kubectl` را کنترل می‌کند.

### زیرمنابع

منابع سفارشی از زیرمنابع `/status` و `/scale` پشتیبانی می‌کنند.

زیرمنابع وضعیت و مقیاس را می‌توان با تعریف آنها در CustomResourceDefinition به صورت اختیاری فعال کرد.

#### زیرمنبع وضعیت

هنگامی که زیرمنبع وضعیت فعال شود، زیرمنبع `/status` برای منبع سفارشی نمایش داده می‌شود.

- بخش‌های وضعیت و مشخصات به ترتیب با JSONPaths `.status` و `.spec` درون یک منبع سفارشی نمایش داده می‌شوند.
- درخواست‌های `PUT` به زیرمنبع `/status` یک شیء منبع سفارشی را می‌گیرند و تغییرات در هر چیزی به جز بخش وضعیت را نادیده می‌گیرند.
- درخواست‌های `PUT` به زیرمنبع `/status` فقط بخش وضعیت منبع سفارشی را اعتبارسنجی می‌کنند.
- درخواست‌های `PUT`/`POST`/`PATCH` به منبع سفارشی تغییرات در بخش وضعیت را نادیده می‌گیرند.
- مقدار `.metadata.generation` برای همه تغییرات به جز تغییرات در `.metadata` یا `.status` افزایش می‌یابد.
- فقط ساختارهای زیر در ریشه طرح اعتبارسنجی OpenAPI CRD مجاز هستند:
  - `description`
  - `example`
  - `exclusiveMaximum`
  - `exclusiveMinimum`
  - `externalDocs`
  - `format`
  - `items`
  - `maximum`
  - `maxItems`
  - `maxLength`
  - `minimum`
  - `minItems`
  - `minLength`
  - `multipleOf`
  - `pattern`
  - `properties`
  - `required`
  - `title`
  - `type`
  - `uniqueItems`

#### زیرمنبع مقیاس

هنگامی که زیرمنبع مقیاس فعال شود، زیرمنبع `/scale` برای منبع سفارشی نمایش داده می‌شود. شیء `autoscaling/v1.Scale` به عنوان بار برای `/scale` ارسال می‌شود.

برای فعال‌کردن زیرمنبع مقیاس، فیلدهای زیر در CustomResourceDefinition تعریف می‌شوند.

- `specReplicasPath` مسیر JSONPath داخل یک منبع سفارشی را که با `scale.spec.replicas` مطابقت دارد تعریف می‌کند.
  - این مقدار الزامی است.
  - فقط JSONPathهای زیر `.spec` و با نشانه نقطه مجاز هستند.
  - اگر هیچ مقداری زیر `specReplicasPath` در منبع سفارشی وجود نداشته باشد، زیرمنبع `/scale` هنگام دریافت مقدار خطا می‌دهد.
- `statusReplicasPath` مسیر JSONPath داخل یک منبع سفارشی را که با `scale.status.replicas` مطابقت دارد تعریف می‌کند.
  - این مقدار الزامی است.
  - فقط JSONPathهای زیر `.status` و با نشانه نقطه مجاز هستند.
  - اگر هیچ مقداری زیر `statusReplicasPath` در منبع سفارشی وجود نداشته باشد، مقدار نمونه وضعیت در زیرمنبع `/scale` به‌طور پیش‌فرض 0 خواهد بود.
- `labelSelectorPath` مسیر JSONPath داخل یک منبع سفارشی را که با `Scale.Status.Selector` مطابقت دارد تعریف می‌کند.
  - این مقدار اختیاری است.
  - برای کار با HPA و VPA باید تنظیم شود.
  - فقط JSONPathهای زیر `.status` یا `.spec` و با نشانه نقطه مجاز هستند.
  - اگر هیچ مقداری زیر `labelSelectorPath` در منبع سفارشی وجود نداشته باشد، مقدار انتخابگر وضعیت در زیرمنبع `/scale` به‌طور پیش‌فرض خالی خواهد بود.
  - فیلدی که این JSONPath به آن اشاره می‌کند باید یک فیلد رشته‌ای باشد (نه یک ساختار انتخابگر پیچیده) که حاوی یک انتخابگر برچسب سریالی‌شده به صورت رشته‌ای است.

در مثال زیر، هر دو زیرمنبع وضعیت و مقیاس فعال شده‌اند.

CustomResourceDefinition را در فایل `resourcedefinition.yaml` ذخیره کنید:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
            status:
              type: object
              properties:
                replicas:
                  type: integer
                labelSelector:
                  type: string
      # subresources describes the subresources for custom resources.
      subresources:
        # status enables the status subresource.
        status: {}
        # scale enables the scale subresource.
        scale:
          # specReplicasPath defines the JSONPath inside of a custom resource that corresponds to Scale.Spec.Replicas.
          specReplicasPath: .spec.replicas
          # statusReplicasPath defines the JSONPath inside of a custom resource that corresponds to Scale.Status.Replicas.
          statusReplicasPath: .status.replicas
          # labelSelectorPath defines the JSONPath inside of a custom resource that corresponds to Scale.Status.Selector.
          labelSelectorPath: .status.labelSelector
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
    - ct
```

و آن را ایجاد کنید:

```shell
kubectl apply -f resourcedefinition.yaml
```

پس از ایجاد شیء CustomResourceDefinition، می‌توانید اشیاء سفارشی ایجاد کنید.

اگر YAML زیر را در `my-crontab.yaml` ذخیره کنید:

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
  replicas: 3
```

و آن را ایجاد کنید:

```shell
kubectl apply -f my-crontab.yaml
```

سپس نقاط پایانی RESTful جدید در سطح نام‌گذاری‌شده در آدرس‌های زیر ایجاد می‌شوند:

```none
/apis/stable.example.com/v1/namespaces/*/crontabs/status
```

و

```none
/apis/stable.example.com/v1/namespaces/*/crontabs/scale
```

یک منبع سفارشی می‌تواند با استفاده از دستور `kubectl scale` مقیاس‌بندی شود. برای مثال، دستور زیر مقدار `.spec.replicas` منبع سفارشی ایجادشده در بالا را به ۵ تنظیم می‌کند:

```shell
kubectl scale --replicas=5 crontabs/my-new-cron-object
crontabs "my-new-cron-object" scaled

kubectl get crontabs my-new-cron-object -o jsonpath='{.spec.replicas}'
5
```

می‌توانید از [PodDisruptionBudget](/docs/tasks/run-application/configure-pdb/) برای محافظت از منابع سفارشی که زیرمنبع مقیاس برای آنها فعال شده است، استفاده کنید.


### دسته‌ها

دسته‌ها لیستی از منابع گروه‌بندی‌شده هستند که منبع سفارشی به آن‌ها تعلق دارد (مثلاً `all`).
شما می‌توانید از `kubectl get <category-name>` برای لیست‌کردن منابع متعلق به دسته استفاده کنید.

مثال زیر `all` را به لیست دسته‌ها در CustomResourceDefinition اضافه می‌کند و نشان می‌دهد که چگونه می‌توان منبع سفارشی را با استفاده از `kubectl get all` خروجی داد.

CustomResourceDefinition زیر را در `resourcedefinition.yaml` ذخیره کنید:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
    - ct
    # categories is a list of grouped resources the custom resource belongs to.
    categories:
    - all
```

و آن را ایجاد کنید:

```shell
kubectl apply -f resourcedefinition.yaml
```

پس از ایجاد شیء CustomResourceDefinition، می‌توانید اشیاء سفارشی ایجاد کنید.

YAML زیر را در `my-crontab.yaml` ذخیره کنید:

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
```

و آن را ایجاد کنید:

```shell
kubectl apply -f my-crontab.yaml
```

می‌توانید دسته را هنگام استفاده از `kubectl get` مشخص کنید:

```shell
kubectl get all
```

و این شامل منابع سفارشی از نوع `CronTab` خواهد بود:

```none
NAME                          AGE
crontabs/my-new-cron-object   3s
```

## {{% heading "whatsnext" %}}

* درباره [منابع سفارشی](/docs/concepts/extend-kubernetes/api-extension/custom-resources/) بیشتر بخوانید.

* به [CustomResourceDefinition](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#customresourcedefinition-v1-apiextensions-k8s-io) مراجعه کنید.

* سرویس‌دهی به [نسخه‌های متعدد](/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definition-versioning/) یک CustomResourceDefinition را ببینید.
