---
title: نسخه‌ها در CustomResourceDefinitions
reviewers:
- sttts
- liggitt
content_type: task
weight: 30
min-kubernetes-server-version: v1.16
---

<!-- overview -->
این صفحه توضیح می‌دهد چگونه اطلاعات نسخه‌بندی را به
[CustomResourceDefinitions](/docs/reference/kubernetes-api/extend-resources/custom-resource-definition-v1/) اضافه کنید تا سطح پایداری CustomResourceDefinitions خود را نشان دهید یا API خود را به نسخه جدیدی ارتقا دهید با تبدیل بین نمایش‌های API. همچنین نحوه ارتقا یک شیء از یک نسخه به نسخه دیگر را توضیح می‌دهد.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}}

شما باید درک اولیه‌ای از [منابع سفارشی](/docs/concepts/extend-kubernetes/api-extension/custom-resources/) داشته باشید.

{{< version-check >}}

<!-- steps -->

## مرور کلی

API CustomResourceDefinition یک جریان کاری برای معرفی و ارتقا به نسخه‌های جدید یک CustomResourceDefinition فراهم می‌کند.

هنگامی که یک CustomResourceDefinition ایجاد می‌شود، اولین نسخه در لیست `spec.versions` در CustomResourceDefinition به سطح پایداری مناسب و یک شماره نسخه تنظیم می‌شود. به عنوان مثال، `v1beta1` نشان می‌دهد که اولین نسخه هنوز پایدار نیست. تمام اشیاء منبع سفارشی ابتدا در این نسخه ذخیره می‌شوند.

پس از ایجاد CustomResourceDefinition، مشتریان می‌توانند استفاده از API `v1beta1` را آغاز کنند.

بعداً ممکن است نیاز به اضافه کردن نسخه جدیدی مانند `v1` باشد.

افزودن یک نسخه جدید:

1. انتخاب یک استراتژی تبدیل. از آنجایی که اشیاء منبع سفارشی نیاز به ارائه در هر دو نسخه دارند، به این معناست که گاهی اوقات در نسخه‌ای متفاوت از نسخه ذخیره‌شده ارائه می‌شوند. برای ممکن ساختن این امر، اشیاء منبع سفارشی باید گاهی اوقات بین نسخه‌ای که ذخیره شده‌اند و نسخه‌ای که ارائه می‌شوند تبدیل شوند. اگر تبدیل شامل تغییرات اسکیما و نیاز به منطق سفارشی باشد، باید از یک وب‌هوک تبدیل استفاده شود. اگر تغییرات اسکیما وجود نداشته باشد، استراتژی تبدیل پیش‌فرض `None` ممکن است استفاده شود و تنها فیلد `apiVersion` هنگام ارائه نسخه‌های مختلف تغییر خواهد کرد.
1. اگر از وب‌هوک‌های تبدیل استفاده می‌شود، وب‌هوک تبدیل را ایجاد و مستقر کنید. جزئیات بیشتر را در [Webhook conversion](#webhook-conversion) ببینید.
1. CustomResourceDefinition را به‌روزرسانی کنید تا نسخه جدید را در لیست `spec.versions` با `served:true` اضافه کنید. همچنین، فیلد `spec.conversion` را به استراتژی تبدیل انتخاب‌شده تنظیم کنید. اگر از وب‌هوک تبدیل استفاده می‌کنید، فیلد `spec.conversion.webhookClientConfig` را برای فراخوانی وب‌هوک پیکربندی کنید.

پس از افزودن نسخه جدید، مشتریان می‌توانند به تدریج به نسخه جدید مهاجرت کنند. برای برخی مشتریان استفاده از نسخه قدیمی در حالی که دیگران از نسخه جدید استفاده می‌کنند کاملاً امن است.

مهاجرت اشیاء ذخیره‌شده به نسخه جدید:

1. بخش [ارتقاء اشیاء موجود به نسخه ذخیره‌شده جدید](#upgrade-existing-objects-to-a-new-stored-version) را ببینید.

استفاده از هر دو نسخه قدیمی و جدید قبل، در حین و بعد از ارتقاء اشیاء به نسخه ذخیره‌شده جدید برای مشتریان امن است.

حذف یک نسخه قدیمی:

1. اطمینان حاصل کنید که همه مشتریان به طور کامل به نسخه جدید مهاجرت کرده‌اند. می‌توان لاگ‌های kube-apiserver را بررسی کرد تا مشتریانی که هنوز از طریق نسخه قدیمی دسترسی دارند شناسایی شوند.
1. `served` را به `false` برای نسخه قدیمی در لیست `spec.versions` تنظیم کنید. اگر هنوز برخی مشتریان به طور غیرمنتظره از نسخه قدیمی استفاده می‌کنند، ممکن است شروع به گزارش خطا در تلاش برای دسترسی به اشیاء منبع سفارشی در نسخه قدیمی کنند. اگر این اتفاق افتاد، دوباره از `served:true` در نسخه قدیمی استفاده کنید، مشتریان باقی‌مانده را به نسخه جدید مهاجرت دهید و این مرحله را تکرار کنید.
1. اطمینان حاصل کنید که مرحله [ارتقاء اشیاء موجود به نسخه ذخیره‌شده جدید](#upgrade-existing-objects-to-a-new-stored-version) انجام شده است.
   1. اطمینان حاصل کنید که `storage` برای نسخه جدید در لیست `spec.versions` در CustomResourceDefinition به `true` تنظیم شده است.
   1. اطمینان حاصل کنید که نسخه قدیمی دیگر در `status.storedVersions` CustomResourceDefinition فهرست نشده است.
1. نسخه قدیمی را از لیست `spec.versions` در CustomResourceDefinition حذف کنید.
1. پشتیبانی تبدیل برای نسخه قدیمی در وب‌هوک‌های تبدیل را حذف کنید.

## مشخص‌کردن چندین نسخه

فیلد `versions` در API CustomResourceDefinition می‌تواند برای پشتیبانی از چندین نسخه منابع سفارشی که توسعه داده‌اید استفاده شود. نسخه‌ها می‌توانند اسکیماهای متفاوتی داشته باشند و وب‌هوک‌های تبدیل می‌توانند منابع سفارشی را بین نسخه‌ها تبدیل کنند. تبدیل وب‌هوک‌ها باید در هر جایی که قابل‌اجرا است از [کنوانسیون‌های API کوبرنتیز](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md) پیروی کنند. به‌ویژه، مستندات [تغییر API](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api_changes.md) را برای مجموعه‌ای از نکات و پیشنهادات مفید ببینید.

{{< note >}}
در `apiextensions.k8s.io/v1beta1`، یک فیلد `version` به جای `versions` وجود داشت. فیلد `version` منسوخ شده و اختیاری است، اما اگر خالی نباشد، باید با اولین آیتم در فیلد `versions` مطابقت داشته باشد.
{{< /note >}}

این مثال یک CustomResourceDefinition با دو نسخه را نشان می‌دهد. برای مثال اول، فرض بر این است که تمام نسخه‌ها اسکیما یکسانی دارند و نیازی به تبدیل بین آن‌ها نیست. نظرات در YAML زمینه بیشتری ارائه می‌دهند.

{{< tabs name="CustomResourceDefinition_versioning_example_1" >}}
{{% tab name="apiextensions.k8s.io/v1" %}}
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: crontabs.example.com
spec:
  # group name to use for REST API: /apis/<group>/<version>
  group: example.com
  # list of versions supported by this CustomResourceDefinition
  versions:
  - name: v1beta1
    # Each version can be enabled/disabled by Served flag.
    served: true
    # One and only one version must be marked as the storage version.
    storage: true
    # A schema is required
    schema:
      openAPIV3Schema:
        type: object
        properties:
          host:
            type: string
          port:
            type: string
  - name: v1
    served: true
    storage: false
    schema:
      openAPIV3Schema:
        type: object
        properties:
          host:
            type: string
          port:
            type: string
  # The conversion section is introduced in Kubernetes 1.13+ with a default value of
  # None conversion (strategy sub-field set to None).
  conversion:
    # None conversion assumes the same schema for all versions and only sets the apiVersion
    # field of custom resources to the proper value
    strategy: None
  # either Namespaced or Cluster
  scope: Namespaced
  names:
    # plural name to be used in the URL: /apis/<group>/<version>/<plural>
    plural: crontabs
    # singular name to be used as an alias on the CLI and for display
    singular: crontab
    # kind is normally the CamelCased singular type. Your resource manifests use this.
    kind: CronTab
    # shortNames allow shorter string to match your resource on the CLI
    shortNames:
    - ct
```
```markdown
{{% /tab %}}
{{% tab name="apiextensions.k8s.io/v1beta1" %}}
```yaml
# Deprecated in v1.16 in favor of apiextensions.k8s.io/v1
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: crontabs.example.com
spec:
  group: example.com
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
  scope: Namespaced
  versions:
  - name: v1alpha1
    served: true
    storage: false
    deprecated: true
    deprecationWarning: "example.com/v1alpha1 CronTab is deprecated; see http://example.com/v1alpha1-v1 for instructions to migrate to example.com/v1 CronTab"
  - name: v1beta1
    served: true
    deprecated: true
  - name: v1
    served: true
    storage: true
  validation:
    openAPIV3Schema: ...
```
{{% /tab %}}
{{< /tabs >}}

CustomResourceDefinition را می‌توانید در یک فایل YAML ذخیره کرده و سپس با استفاده از `kubectl apply` آن را ایجاد کنید.

```shell
kubectl apply -f my-versioned-crontab.yaml
```

پس از ایجاد، سرور API شروع به ارائه هر نسخه فعال در یک نقطه پایان HTTP REST می‌کند. در مثال بالا، نسخه‌های API در `/apis/example.com/v1beta1` و `/apis/example.com/v1` در دسترس هستند.

### اولویت نسخه

صرف‌نظر از ترتیب تعریف نسخه‌ها در یک CustomResourceDefinition، نسخه با بالاترین اولویت به‌طور پیش‌فرض توسط kubectl برای دسترسی به اشیاء استفاده می‌شود. اولویت با تجزیه فیلد _name_ برای تعیین شماره نسخه، پایداری (GA، Beta یا Alpha)، و ترتیب در آن سطح پایداری تعیین می‌شود.

الگوریتم مورد استفاده برای مرتب‌سازی نسخه‌ها به گونه‌ای طراحی شده است که نسخه‌ها را به همان روشی مرتب کند که پروژه Kubernetes نسخه‌های Kubernetes را مرتب می‌کند. نسخه‌ها با `v` شروع می‌شوند که به دنبال آن یک شماره، یک نشان‌گذاری اختیاری `beta` یا `alpha`، و اطلاعات نسخه‌گذاری عددی اختیاری دنبال می‌شود. به‌طور کلی، یک رشته نسخه ممکن است شبیه `v2` یا `v2beta1` باشد. نسخه‌ها با استفاده از الگوریتم زیر مرتب می‌شوند:

- ورودی‌هایی که از الگوهای نسخه Kubernetes پیروی می‌کنند قبل از آنهایی که پیروی نمی‌کنند مرتب می‌شوند.
- برای ورودی‌هایی که از الگوهای نسخه Kubernetes پیروی می‌کنند، قسمت‌های عددی رشته نسخه از بزرگ به کوچک مرتب می‌شوند.
- اگر رشته‌های `beta` یا `alpha` به دنبال اولین قسمت عددی باشند، به ترتیب پس از رشته معادل بدون پسوند `beta` یا `alpha` مرتب می‌شوند (که فرض می‌شود نسخه GA است).
- اگر عدد دیگری پس از `beta` یا `alpha` بیاید، آن اعداد نیز از بزرگ به کوچک مرتب می‌شوند.
- رشته‌هایی که با فرمت فوق مطابقت ندارند به صورت الفبایی مرتب می‌شوند و قسمت‌های عددی به طور ویژه‌ای در نظر گرفته نمی‌شوند. توجه کنید که در مثال زیر، `foo1` بالاتر از `foo10` مرتب شده است. این با مرتب‌سازی قسمت عددی ورودی‌هایی که از الگوهای نسخه Kubernetes پیروی می‌کنند متفاوت است.

این ممکن است منطقی به نظر برسد اگر به لیست نسخه‌های مرتب‌شده زیر نگاه کنید:

```none
- v10
- v2
- v1
- v11beta2
- v10beta3
- v3beta1
- v12alpha1
- v11alpha2
- foo1
- foo10
```

برای مثال در [مشخص‌کردن چندین نسخه](#specify-multiple-versions)، ترتیب نسخه `v1`، به دنبال آن `v1beta1` است. این باعث می‌شود که دستور kubectl از `v1` به عنوان نسخه پیش‌فرض استفاده کند مگر اینکه شیء ارائه‌شده نسخه را مشخص کند.

### منسوخ شدن نسخه

{{< feature-state state="stable" for_k8s_version="v1.19" >}}

از نسخه v1.19، یک CustomResourceDefinition می‌تواند نشان دهد که یک نسخه خاص از منبعی که تعریف می‌کند منسوخ شده است.
هنگامی که درخواست‌های API به یک نسخه منسوخ از آن منبع انجام می‌شود، یک پیام هشدار در پاسخ API به عنوان یک هدر بازگردانده می‌شود.
پیام هشدار برای هر نسخه منسوخ از منبع می‌تواند در صورت تمایل سفارشی شود.

یک پیام هشدار سفارشی باید نشان‌دهنده گروه API، نسخه و نوع منسوخ شده باشد، و باید نشان دهد که از چه گروه API، نسخه و نوعی باید استفاده شود، در صورت موجود بودن.

{{< tabs name="CustomResourceDefinition_versioning_deprecated" >}}
{{% tab name="apiextensions.k8s.io/v1" %}}
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.example.com
spec:
  group: example.com
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
  scope: Namespaced
  versions:
  - name: v1alpha1
    served: true
    storage: false
    # این نشان می‌دهد که نسخه v1alpha1 منبع سفارشی منسوخ شده است.
    # درخواست‌های API به این نسخه یک هشدار هدر در پاسخ سرور دریافت می‌کنند.
    deprecated: true
    # این پیام هشدار پیش‌فرض برای مشتریان API که درخواست‌های API v1alpha1 انجام می‌دهند را لغو می‌کند.
    deprecationWarning: "example.com/v1alpha1 CronTab is deprecated; see http://example.com/v1alpha1-v1 for instructions to migrate to example.com/v1 CronTab"
    schema: ...
  - name: v1beta1
    served: true
    # این نشان می‌دهد که نسخه v1beta1 منبع سفارشی منسوخ شده است.
    # درخواست‌های API به این نسخه یک هشدار هدر در پاسخ سرور دریافت می‌کنند.
    # یک پیام هشدار پیش‌فرض برای این نسخه بازگردانده می‌شود.
    deprecated: true
    schema: ...
  - name: v1
    served: true
    storage: true
    schema: ...
```
```markdown
{{% /tab %}}
{{% tab name="apiextensions.k8s.io/v1beta1" %}}
```yaml
# Deprecated in v1.16 in favor of apiextensions.k8s.io/v1
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: crontabs.example.com
spec:
  group: example.com
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
  scope: Namespaced
  validation: ...
  versions:
  - name: v1alpha1
    served: true
    storage: false
    # This indicates the v1alpha1 version of the custom resource is deprecated.
    # API requests to this version receive a warning header in the server response.
    deprecated: true
    # This overrides the default warning returned to API clients making v1alpha1 API requests.
    deprecationWarning: "example.com/v1alpha1 CronTab is deprecated; see http://example.com/v1alpha1-v1 for instructions to migrate to example.com/v1 CronTab"
  - name: v1beta1
    served: true
    # This indicates the v1beta1 version of the custom resource is deprecated.
    # API requests to this version receive a warning header in the server response.
    # A default warning message is returned for this version.
    deprecated: true
  - name: v1
    served: true
    storage: true
```
{{% /tab %}}
{{< /tabs >}}

### حذف نسخه

یک نسخه قدیمی از یک CustomResourceDefinition نمی‌تواند از مانیفست حذف شود تا زمانی که داده‌های ذخیره‌شده موجود به نسخه جدیدتر API برای همه کلاسترهایی که نسخه قدیمی منبع سفارشی را ارائه می‌دهند مهاجرت کند و نسخه قدیمی از `status.storedVersions` CustomResourceDefinition حذف شود.

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
  name: crontabs.example.com
spec:
  group: example.com
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
  scope: Namespaced
  versions:
  - name: v1beta1
    # This indicates the v1beta1 version of the custom resource is no longer served.
    # API requests to this version receive a not found error in the server response.
    served: false
    schema: ...
  - name: v1
    served: true
    # The new served version should be set as the storage version
    storage: true
    schema: ...
```

## تبدیل Webhook

{{< feature-state state="stable" for_k8s_version="v1.16" >}}

{{< note >}}
تبدیل Webhook از نسخه 1.15 به عنوان بتا و از نسخه 1.13 به عنوان آلفا در دسترس است. ویژگی `CustomResourceWebhookConversion` باید فعال باشد که به صورت خودکار برای بسیاری از کلاسترها برای ویژگی‌های بتا فعال است. لطفاً به مستندات [feature gate](/docs/reference/command-line-tools-reference/feature-gates/) برای اطلاعات بیشتر مراجعه کنید.
{{< /note >}}

مثال بالا دارای تبدیل None بین نسخه‌ها است که فقط فیلد `apiVersion` را در تبدیل تنظیم می‌کند و بقیه شیء را تغییر نمی‌دهد. سرور API همچنین از تبدیل‌های Webhook که یک سرویس خارجی را در صورت نیاز به تبدیل فراخوانی می‌کند، پشتیبانی می‌کند. به عنوان مثال هنگامی که:

* منبع سفارشی در یک نسخه متفاوت از نسخه ذخیره شده درخواست می‌شود.
* یک Watch در یک نسخه ایجاد شده است اما شیء تغییر یافته در نسخه دیگری ذخیره شده است.
* درخواست PUT منبع سفارشی در نسخه‌ای متفاوت از نسخه ذخیره شده است.

برای پوشش تمام این موارد و بهینه‌سازی تبدیل توسط سرور API، درخواست‌های تبدیل ممکن است حاوی چندین شیء باشند تا تماس‌های خارجی را به حداقل برسانند. Webhook باید این تبدیل‌ها را به صورت مستقل انجام دهد.

### نوشتن یک سرور تبدیل Webhook

لطفاً به پیاده‌سازی [سرور تبدیل Webhook منبع سفارشی](https://github.com/kubernetes/kubernetes/tree/v1.25.3/test/images/agnhost/crd-conversion-webhook/main.go) که در یک تست e2e Kubernetes تأیید شده است، مراجعه کنید. Webhook درخواست‌های `ConversionReview` ارسال شده توسط سرورهای API را پردازش می‌کند و نتایج تبدیل را در `ConversionResponse` باز می‌گرداند. توجه داشته باشید که درخواست حاوی لیستی از منابع سفارشی است که باید به صورت مستقل بدون تغییر ترتیب اشیاء تبدیل شوند. سرور مثال به گونه‌ای سازمان‌دهی شده است که برای تبدیل‌های دیگر قابل استفاده مجدد باشد. بیشتر کدهای مشترک در [فایل فریمورک](https://github.com/kubernetes/kubernetes/tree/v1.25.3/test/images/agnhost/crd-conversion-webhook/converter/framework.go) قرار دارند که تنها یک [تابع](https://github.com/kubernetes/kubernetes/tree/v1.25.3/test/images/agnhost/crd-conversion-webhook/converter/example_converter.go#L29-L80) برای تبدیل‌های مختلف پیاده‌سازی می‌شود.

{{< note >}}
سرور تبدیل Webhook مثال فیلد `ClientAuth` را [خالی](https://github.com/kubernetes/kubernetes/tree/v1.25.3/test/images/agnhost/crd-conversion-webhook/config.go#L47-L48) می‌گذارد که به طور پیش‌فرض به `NoClientCert` تنظیم می‌شود. این به این معناست که سرور Webhook هویت مشتریان را احراز هویت نمی‌کند، فرضاً سرورهای API. اگر به TLS متقابل یا روش‌های دیگر برای احراز هویت مشتریان نیاز دارید، ببینید چگونه [سرورهای API را احراز هویت کنید](/docs/reference/access-authn-authz/extensible-admission-controllers/#authenticate-apiservers).
{{< /note >}}

#### تغییرات مجاز

یک Webhook تبدیل نباید چیزی داخل `metadata` شیء تبدیل شده غیر از `labels` و `annotations` را تغییر دهد. تلاش برای تغییر `name`، `UID` و `namespace` رد شده و درخواست باعث ایجاد خطا می‌شود که باعث تبدیل شد. تمام تغییرات دیگر نادیده گرفته می‌شوند.

### استقرار سرویس تبدیل Webhook

مستندات برای استقرار سرویس تبدیل Webhook همانند سرویس [مثال پذیرش Webhook](/docs/reference/access-authn-authz/extensible-admission-controllers/#deploy-the-admission-webhook-service) است. فرض برای بخش‌های بعدی این است که سرور تبدیل Webhook به یک سرویس به نام `example-conversion-webhook-server` در namespace `default` استقرار یافته و ترافیک را در مسیر `/crdconvert` ارائه می‌دهد.

{{< note >}}
وقتی سرور Webhook به عنوان یک سرویس در کلاستر Kubernetes استقرار می‌یابد، باید از طریق یک سرویس در پورت 443 ارائه شود (خود سرور می‌تواند یک پورت دلخواه داشته باشد اما شیء سرویس باید آن را به پورت 443 نگاشت کند). ارتباط بین سرور API و سرویس Webhook ممکن است در صورتی که پورت دیگری برای سرویس استفاده شود، شکست بخورد.
{{< /note >}}
```markdown
### پیکربندی CustomResourceDefinition برای استفاده از webhook های تبدیل

مثال تبدیل `None` می‌تواند با تغییر بخش `conversion` در `spec` برای استفاده از webhook تبدیل گسترش یابد:

{{< tabs name="CustomResourceDefinition_versioning_example_2" >}}
{{% tab name="apiextensions.k8s.io/v1" %}}
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # نام باید با فیلدهای spec زیر مطابقت داشته باشد و به فرم: <plural>.<group> باشد
  name: crontabs.example.com
spec:
  # نام گروه برای استفاده در REST API: /apis/<group>/<version>
  group: example.com
  # لیست نسخه‌هایی که توسط این CustomResourceDefinition پشتیبانی می‌شوند
  versions:
  - name: v1beta1
    # هر نسخه می‌تواند توسط پرچم Served فعال/غیرفعال شود.
    served: true
    # فقط یک نسخه باید به عنوان نسخه ذخیره‌سازی علامت‌گذاری شود.
    storage: true
    # هر نسخه می‌تواند طرح خود را تعریف کند وقتی هیچ طرح سطح بالا تعریف نشده است.
    schema:
      openAPIV3Schema:
        type: object
        properties:
          hostPort:
            type: string
  - name: v1
    served: true
    storage: false
    schema:
      openAPIV3Schema:
        type: object
        properties:
          host:
            type: string
          port:
            type: string
  conversion:
    # استراتژی Webhook به سرور API دستور می‌دهد که برای هر تبدیلی بین منابع سفارشی یک webhook خارجی را فراخوانی کند.
    strategy: Webhook
    # webhook زمانی که استراتژی Webhook است لازم است و endpoint webhook که باید توسط سرور API فراخوانی شود را پیکربندی می‌کند.
    webhook:
      # conversionReviewVersions نشان می‌دهد که چه نسخه‌هایی از ConversionReview توسط webhook قابل فهم/ترجیح هستند.
      # اولین نسخه در لیست که توسط سرور API فهمیده شود به webhook ارسال می‌شود.
      # webhook باید با یک شیء ConversionReview در همان نسخه‌ای که دریافت کرده پاسخ دهد.
      conversionReviewVersions: ["v1","v1beta1"]
      clientConfig:
        service:
          namespace: default
          name: example-conversion-webhook-server
          path: /crdconvert
        caBundle: "Ci0tLS0tQk...<base64-encoded PEM bundle>...tLS0K"
  # یا Namespaced یا Cluster
  scope: Namespaced
  names:
    # نام جمع برای استفاده در URL: /apis/<group>/<version>/<plural>
    plural: crontabs
    # نام مفرد برای استفاده به عنوان نام مستعار در CLI و برای نمایش
    singular: crontab
    # kind معمولاً نوع مفرد CamelCased است. مانیفست‌های منبع شما از این استفاده می‌کنند.
    kind: CronTab
    # shortNames اجازه می‌دهد رشته کوتاه‌تری با منبع شما در CLI مطابقت داشته باشد
    shortNames:
    - ct
```
{{% /tab %}}
{{% tab name="apiextensions.k8s.io/v1beta1" %}}
```yaml
# منسوخ شده در v1.16 به نفع apiextensions.k8s.io/v1
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  # نام باید با فیلدهای spec زیر مطابقت داشته باشد و به فرم: <plural>.<group> باشد
  name: crontabs.example.com
spec:
  # نام گروه برای استفاده در REST API: /apis/<group>/<version>
  group: example.com
  # object fields که در OpenAPI schema های زیر مشخص نشده‌اند را حذف می‌کند.
  preserveUnknownFields: false
  # لیست نسخه‌هایی که توسط این CustomResourceDefinition پشتیبانی می‌شوند
  versions:
  - name: v1beta1
    # هر نسخه می‌تواند توسط پرچم Served فعال/غیرفعال شود.
    served: true
    # فقط یک نسخه باید به عنوان نسخه ذخیره‌سازی علامت‌گذاری شود.
    storage: true
    # هر نسخه می‌تواند طرح خود را تعریف کند وقتی هیچ طرح سطح بالا تعریف نشده است.
    schema:
      openAPIV3Schema:
        type: object
        properties:
          hostPort:
            type: string
  - name: v1
    served: true
    storage: false
    schema:
      openAPIV3Schema:
        type: object
        properties:
          host:
            type: string
          port:
            type: string
  conversion:
    # استراتژی Webhook به سرور API دستور می‌دهد که برای هر تبدیلی بین منابع سفارشی یک webhook خارجی را فراخوانی کند.
    strategy: Webhook
    # webhookClientConfig زمانی که استراتژی Webhook است لازم است و endpoint webhook که باید توسط سرور API فراخوانی شود را پیکربندی می‌کند.
    webhookClientConfig:
      service:
        namespace: default
        name: example-conversion-webhook-server
        path: /crdconvert
      caBundle: "Ci0tLS0tQk...<base64-encoded PEM bundle>...tLS0K"
  # یا Namespaced یا Cluster
  scope: Namespaced
  names:
    # نام جمع برای استفاده در URL: /apis/<group>/<version>/<plural>
    plural: crontabs
    # نام مفرد برای استفاده به عنوان نام مستعار در CLI و برای نمایش
    singular: crontab
    # kind معمولاً نوع مفرد CamelCased است. مانیفست‌های منبع شما از این استفاده می‌کنند.
    kind: CronTab
    # shortNames اجازه می‌دهد رشته کوتاه‌تری با منبع شما در CLI مطابقت داشته باشد
    shortNames:
    - ct
```
{{% /tab %}}
{{< /tabs >}}

شما می‌توانید CustomResourceDefinition را در یک فایل YAML ذخیره کرده و سپس با استفاده از `kubectl apply` آن را اعمال کنید.

```shell
kubectl apply -f my-versioned-crontab-with-conversion.yaml
```

مطمئن شوید که سرویس تبدیل قبل از اعمال تغییرات جدید در حال اجرا است.
### تماس با وب‌هوک

هنگامی که سرور API تصمیم می‌گیرد که یک درخواست باید به یک وب‌هوک تبدیل شود، نیاز دارد که بداند چگونه با وب‌هوک تماس بگیرد. این در بخش `webhookClientConfig` در تنظیمات وب‌هوک مشخص می‌شود.

وب‌هوک‌های تبدیل می‌توانند از طریق یک URL یا مرجع سرویس فراخوانی شوند و اختیاری استفاده از یک مجموعه گواهی CA سفارشی برای تأیید اتصال TLS را دارند.

### URL

`url` مکان وب‌هوک را به شکل استاندارد URL (`scheme://host:port/path`) مشخص می‌کند.

`host` نباید به یک سرویس در حال اجرا در خوشه اشاره کند؛ به جای آن، از فیلد `service` استفاده کنید. `host` ممکن است از طریق DNS خارجی در برخی از سرورهای API رفع شود (به عنوان مثال، `kube-apiserver` نمی‌تواند DNS درون خوشه را حل کند زیرا این یک نقض لایه‌ای است). `host` ممکن است یک آدرس IP نیز باشد.

لطفا توجه داشته باشید که استفاده از `localhost` یا `127.0.0.1` به عنوان `host` خطرناک است مگر اینکه به دقت زیادی وب‌هوک را در همه میزبان‌هایی که سرورهای API در آن‌ها نیاز به انجام تماس با این وب‌هوک دارند، اجرا کنید. نصب‌های چنینی معمولاً قابل حمل نیست یا به راحتی در یک خوشه جدید قابل اجرا نیستند.

طرحی از یک وب‌هوک تبدیل که به یک URL فراخوانی شده تنظیم شده است (و انتظار می‌رود گواهی TLS با استفاده از ریشه‌های اعتماد سیستم تأیید شود، بنابراین `caBundle` مشخص نشده است):

{{< tabs name="مثال_نسخه‌بندی_CustomResourceDefinition_3" >}}
{{% tab name="apiextensions.k8s.io/v1" %}}
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
...
spec:
  ...
  conversion:
    strategy: Webhook
    webhook:
      clientConfig:
        url: "https://my-webhook.example.com:9443/my-webhook-path"
...
```
{{% /tab %}}
{{% tab name="apiextensions.k8s.io/v1beta1" %}}
```yaml
# قدیمی در v1.16 به جای apiextensions.k8s.io/v1
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
...
spec:
  ...
  conversion:
    strategy: Webhook
    webhookClientConfig:
      url: "https://my-webhook.example.com:9443/my-webhook-path"
...
```
{{% /tab %}}
{{< /tabs >}}

### مرجع سرویس

بخش `service` داخل `webhookClientConfig` یک مرجع به سرویس برای یک وب‌هوک تبدیل است. اگر وب‌هوک در داخل خوشه در حال اجرا است، آن‌گاه باید به جای `url` از `service` استفاده کنید. فضای نام سرویس و نام آن الزامی است. پورت اختیاری است و به طور پیش‌فرض 443 است. مسیر نیز اختیاری است و به طور پیش‌فرض "/".

طرحی از یک وب‌هوک که تنظیم شده است تا یک سرویس را در پورت "1234" با زیرمسیر "/my-path" فراخوانی کند، و اتصال TLS را در برابر ServerName `my-service-name.my-service-namespace.svc` با استفاده از یک مجموعه گواهی CA سفارشی تأیید کند.

{{< tabs name="مثال_نسخه‌بندی_CustomResourceDefinition_4" >}}
{{% tab name="apiextensions.k8s.io/v1" %}}
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
...
spec:
  ...
  conversion:
    strategy: Webhook
    webhook:
      clientConfig:
        service:
          namespace: my-service-namespace
          name: my-service-name
          path: /my-path
          port: 1234
        caBundle: "Ci0tLS0tQk...<base64-encoded PEM bundle>...tLS0K"
...
```
{{% /tab %}}
{{% tab name="apiextensions.k8s.io/v1beta1" %}}
```yaml
# قدیمی در v1.16 به جای apiextensions.k8s.io/v1
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
...
spec:
  ...
  conversion:
    strategy: Webhook
    webhookClientConfig:
      service:
        namespace: my-service-namespace
        name: my-service-name
        path: /my-path
        port: 1234
      caBundle: "Ci0tLS0tQk...<base64-encoded PEM bundle>...tLS0K"
...
```
{{% /tab %}}
{{< /tabs >}}
## درخواست و پاسخ وب‌هوک

### درخواست

وب‌هوک‌ها یک درخواست POST ارسال می‌کنند، با `Content-Type: application/json`، و با یک شی `ConversionReview` در گروه API `apiextensions.k8s.io` که به صورت JSON سریالی‌سازی شده است، به عنوان بدنه درخواست.

وب‌هوک‌ها می‌توانند با فیلد `conversionReviewVersions` در CustomResourceDefinition خود، نسخه‌های مورد قبول `ConversionReview` را مشخص کنند:

{{< tabs name="conversionReviewVersions" >}}
{{% tab name="apiextensions.k8s.io/v1" %}}
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
...
spec:
  ...
  conversion:
    strategy: Webhook
    webhook:
      conversionReviewVersions: ["v1", "v1beta1"]
      ...
```

`conversionReviewVersions` یک فیلد ضروری است هنگام ایجاد Custom Resource Definition `apiextensions.k8s.io/v1`.
وب‌هوک‌ها موظف به پشتیبانی حداقل یک نسخه `ConversionReview` که توسط سرور API فعلی و سرور API قبلی مورد تفهیم قرار گرفته است، هستند.
{{% /tab %}}
{{% tab name="apiextensions.k8s.io/v1beta1" %}}
```yaml
# قدیمی در v1.16 به جای apiextensions.k8s.io/v1
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
...
spec:
  ...
  conversion:
    strategy: Webhook
    conversionReviewVersions: ["v1", "v1beta1"]
    ...
```

اگر نسخه‌های `conversionReviewVersions` مشخص نشوند، پیش‌فرض در ایجاد Custom Resource Definition `apiextensions.k8s.io/v1beta1` `v1beta1` است.
{{% /tab %}}
{{< /tabs >}}

سرورهای API نخستین نسخه `ConversionReview` در لیست `conversionReviewVersions` را که پشتیبانی می‌کنند ارسال می‌کنند.
اگر هیچ یک از نسخه‌های موجود در لیست توسط سرور API پشتیبانی نشوند، امکان ایجاد Custom Resource Definition وجود ندارد.
اگر سرور API با تنظیمات وب‌هوک تبدیلی روبرو شود که پیش‌تر ایجاد شده است و هیچ یک از نسخه‌های `ConversionReview` را که سرور API می‌داند چگونه ارسال کند، پشتیبانی نمی‌کند، تلاش برای فراخوانی وب‌هوک شکست خواهد خورد.

این مثال نشان می‌دهد که داده‌های موجود در یک شی `ConversionReview` برای یک درخواست تبدیل اشیا `CronTab` به `example.com/v1`:

{{< tabs name="ConversionReview_request" >}}
{{% tab name="apiextensions.k8s.io/v1" %}}
```yaml
{
  "apiVersion": "apiextensions.k8s.io/v1",
  "kind": "ConversionReview",
  "request": {
    # شناسه تصادفی که منحصر به فرد این فراخوانی تبدیل است
    "uid": "705ab4f5-6393-11e8-b7cc-42010a800002",
    
    # گروه و نسخه API که اشیاء باید به آن تبدیل شوند
    "desiredAPIVersion": "example.com/v1",
    
    # لیست اشیاء برای تبدیل.
    # ممکن است شامل یک یا چند شیء، در یک یا چند نسخه باشد.
    "objects": [
      {
        "kind": "CronTab",
        "apiVersion": "example.com/v1beta1",
        "metadata": {
          "creationTimestamp": "2019-09-04T14:03:02Z",
          "name": "local-crontab",
          "namespace": "default",
          "resourceVersion": "143",
          "uid": "3415a7fc-162b-4300-b5da-fd6083580d66"
        },
        "hostPort": "localhost:1234"
      },
      {
        "kind": "CronTab",
        "apiVersion": "example.com/v1beta1",
        "metadata": {
          "creationTimestamp": "2019-09-03T13:02:01Z",
          "name": "remote-crontab",
          "resourceVersion": "12893",
          "uid": "359a83ec-b575-460d-b553-d859cedde8a0"
        },
        "hostPort": "example.com:2345"
      }
    ]
  }
}
```
{{% /tab %}}
{{% tab name="apiextensions.k8s.io/v1beta1" %}}
```yaml
{
  # قدیمی در v1.16 به جای apiextensions.k8s.io/v1
  "apiVersion": "apiextensions.k8s.io/v1beta1",
  "kind": "ConversionReview",
  "request": {
    # شناسه تصادفی که منحصر به فرد این فراخوانی تبدیل است
    "uid": "705ab4f5-6393-11e8-b7cc-42010a800002",
    
    # گروه و نسخه API که اشیاء باید به آن تبدیل شوند
    "desiredAPIVersion": "example.com/v1",
    
    # لیست اشیاء برای تبدیل.
    # ممکن است شامل یک یا چند شیء، در یک یا چند نسخه باشد.
    "objects": [
      {
        "kind": "CronTab",
        "apiVersion": "example.com/v1beta1",
        "metadata": {
          "creationTimestamp": "2019-09-04T14:03:02Z",
          "name": "local-crontab",
          "namespace": "default",
          "resourceVersion": "143",
          "uid": "3415a7fc-162b-4300-b5da-fd6083580d66"
        },
        "hostPort": "localhost:1234"
      },
      {
        "kind": "CronTab",
        "apiVersion": "example.com/v1beta1",
        "metadata": {
          "creationTimestamp": "2019-09-03T13:02:01Z",
          "name": "remote-crontab",
          "resourceVersion": "12893",
          "uid": "359a83ec-b575-460d-b553-d859cedde8a0"
        },
        "hostPort": "example.com:2345"
      }
    ]
  }
}
```
{{% /tab %}}
{{< /tabs >}}
### پاسخ

وب‌هوک‌ها با کد وضعیت HTTP 200 (`Content-Type: application/json`) و با بدنه‌ای شامل یک شی `ConversionReview` (در همان نسخه‌ای که ارسال شده) و با بخش `response` پر شده، سریالیزه شده به صورت JSON پاسخ می‌دهند.

اگر تبدیل موفق باشد، وب‌هوک باید یک بخش `response` شامل فیلدهای زیر را بازگرداند:
- `uid`، کپی شده از `request.uid` که به وب‌هوک ارسال شده است
- `result`، تنظیم شده به `{"status":"Success"}`
- `convertedObjects`، شامل همه اشیاء از `request.objects` که به `request.desiredAPIVersion` تبدیل شده‌اند

مثالی از یک پاسخ موفق حداقلی از وب‌هوک:

{{< tabs name="ConversionReview_response_success" >}}
{{% tab name="apiextensions.k8s.io/v1" %}}
```yaml
{
  "apiVersion": "apiextensions.k8s.io/v1",
  "kind": "ConversionReview",
  "response": {
    # باید با <request.uid> مطابقت داشته باشد
    "uid": "705ab4f5-6393-11e8-b7cc-42010a800002",
    "result": {
      "status": "Success"
    },
    # اشیاء باید با ترتیب request.objects مطابقت داشته باشند و apiVersion باید به <request.desiredAPIVersion> تنظیم شده باشد.
    # kind، metadata.uid، metadata.name و metadata.namespace باید توسط وب‌هوک تغییر نکنند.
    # فیلدهای metadata.labels و metadata.annotations ممکن است توسط وب‌هوک تغییر کنند.
    # تمام تغییرات دیگر در فیلدهای metadata توسط وب‌هوک نادیده گرفته می‌شود.
    "convertedObjects": [
      {
        "kind": "CronTab",
        "apiVersion": "example.com/v1",
        "metadata": {
          "creationTimestamp": "2019-09-04T14:03:02Z",
          "name": "local-crontab",
          "namespace": "default",
          "resourceVersion": "143",
          "uid": "3415a7fc-162b-4300-b5da-fd6083580d66"
        },
        "host": "localhost",
        "port": "1234"
      },
      {
        "kind": "CronTab",
        "apiVersion": "example.com/v1",
        "metadata": {
          "creationTimestamp": "2019-09-03T13:02:01Z",
          "name": "remote-crontab",
          "resourceVersion": "12893",
          "uid": "359a83ec-b575-460d-b553-d859cedde8a0"
        },
        "host": "example.com",
        "port": "2345"
      }
    ]
  }
}
```
{{% /tab %}}
{{% tab name="apiextensions.k8s.io/v1beta1" %}}
```yaml
{
  # Deprecated in v1.16 in favor of apiextensions.k8s.io/v1
  "apiVersion": "apiextensions.k8s.io/v1beta1",
  "kind": "ConversionReview",
  "response": {
    # باید با <request.uid> مطابقت داشته باشد
    "uid": "705ab4f5-6393-11e8-b7cc-42010a800002",
    "result": {
      "status": "Failed"
    },
    # اشیاء باید با ترتیب request.objects مطابقت داشته باشند و apiVersion باید به <request.desiredAPIVersion> تنظیم شده باشد.
    # kind، metadata.uid، metadata.name و metadata.namespace باید توسط وب‌هوک تغییر نکنند.
    # فیلدهای metadata.labels و metadata.annotations ممکن است توسط وب‌هوک تغییر کنند.
    # تمام تغییرات دیگر در فیلدهای metadata توسط وب‌هوک نادیده گرفته می‌شود.
    "convertedObjects": [
      {
        "kind": "CronTab",
        "apiVersion": "example.com/v1",
        "metadata": {
          "creationTimestamp": "2019-09-04T14:03:02Z",
          "name": "local-crontab",
          "namespace": "default",
          "resourceVersion": "143",
          "uid": "3415a7fc-162b-4300-b5da-fd6083580d66"
        },
        "host": "localhost",
        "port": "1234"
      },
      {
        "kind": "CronTab",
        "apiVersion": "example.com/v1",
        "metadata": {
          "creationTimestamp": "2019-09-03T13:02:01Z",
          "name": "remote-crontab",
          "resourceVersion": "12893",
          "uid": "359a83ec-b575-460d-b553-d859cedde8a0"
        },
        "host": "example.com",
        "port": "2345"
      }
    ]
  }
}
```
{{% /tab %}}
{{< /tabs >}}

اگر تبدیل ناموفق باشد، وب‌هوک باید یک بخش `response` شامل فیلدهای زیر را بازگرداند:
- `uid`، کپی شده از `request.uid` که به وب‌هوک ارسال شده است
- `result`، تنظیم شده به `{"status":"Failed"}`

{{< warning >}}
شکست تبدیل می‌تواند باعث اختلال در دسترسی خواندن و نوشتن به منابع سفارشی شود،
شامل امکان به‌روزرسانی یا حذف منابع. این شکست‌های تبدیل باید به هر اندازه‌ای که ممکن است، از قبل جلوگیری شود و نباید برای اعمال محدودیت‌های اعتبارسنجی استفاده شود (از اسکیمای اعتبارسنجی یا تایید وب‌هوک به جای آن استفاده کنید).
{{< /warning >}}

مثالی از یک پاسخ از وب‌هوک که نشان می‌دهد درخواست تبدیل ناموفق بوده، با یک پیام اختیاری:

{{< tabs name="ConversionReview_response_failure" >}}
{{% tab name="apiextensions.k8s.io/v1" %}}
```yaml
{
  "apiVersion": "apiextensions.k8s.io/v1",
  "kind": "ConversionReview",
  "response": {
    "uid": "<مقدار از request.uid>",
    "result": {
      "status": "Failed",
      "message": "hostPort could not be parsed into a separate host and port"
    }
  }
}
```
{{% /tab %}}
{{% tab name="apiextensions.k8s.io/v1beta1" %}}
```yaml
{
  # Deprecated in v1.16 in favor of apiextensions.k8s.io/v1
  "apiVersion": "apiextensions.k8s.io/v1beta1",
  "kind": "ConversionReview",
  "response": {
    "uid": "<مقدار از request.uid>",
    "result": {
      "status": "Failed",
      "message": "hostPort could not be parsed into a separate host and port"
    }
  }
}
```
{{% /tab %}}
{{< /tabs >}}

## نو

شتن، خواندن و به‌روزرسانی اشیاء CustomResourceDefinition با نسخه‌های

زمانی که یک شی نوشته می‌شود، در نسخه‌ای که در زمان نوشتن نسخه ذخیره‌سازی تعیین شده است، ذخیره می‌شود. اگر نسخه ذخیره‌سازی تغییر کند، اشیاء موجود به طور خودکار تبدیل نمی‌شوند. با این حال، اشیاء تازه‌ساخته یا به‌روزرسانی‌شده در نسخه جدید ذخیره می‌شوند. امکان دارد که یک شی در نسخه‌ای ذخیره شود که دیگر سرویس نمی‌شود.

زمانی که یک شی را می‌خوانید، شما نسخه را به عنوان بخشی از مسیر مشخص می‌کنید. می‌توانید از هر نسخه‌ای که در حال حاضر سرویس می‌شود، شیء را درخواست کنید. اگر نسخه‌ای را مشخص کنید که با نسخه ذخیره‌شده شی‌ء متفاوت است، Kubernetes شی را به شما در نسخه مورد نظر بازمی‌گرداند، اما شی در حافظه دیسک تغییر نمی‌کند.

چه اتفاقی برای شی که در حال خدمت به درخواست خواندن است رخ می‌دهد، به وابستگی به آنچه در CRD's `spec.conversion` مشخص شده است:
- اگر مقدار پیش‌فرض `strategy` به `None` مشخص شده باشد، تنها تغییراتی که به شیء اعمال می‌شود، تغییر رشته `apiVersion` است و شاید [pruning unknown fields](/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#field-pruning) (بسته به پیکربندی). توجه داشته باشید که این احتمالاً به نتایج خوبی منجر نمی‌شود اگر اسکیماها بین نسخه‌های ذخیره‌سازی و نسخه‌های درخواستی متفاوت باشند. به ویژه، این استراتژی را نباید استفاده کنید اگر داده مشابه در فیلدهای مختلف بین نسخه‌ها نمایش داده شود.
- اگر [تبدیل وب‌هوک](#webhook-conversion) مشخص شده باشد، آنگاه این مکانیسم کنترل تبدیل را کنترل می‌کند.

اگر یک شی موجود را به‌روزرسانی کنید، آن را در نسخه‌ای که در حال حاضر نسخه ذخیره‌سازی است، دوباره می‌نویسید. این تنها راهی است که اشیاء می‌توانند از یک نسخه به نسخه دیگر تغییر کنند.

برای توضیح این موضوع، دنباله‌ای از رویدادهای فرضی زیر را در نظر بگیرید:

1. نسخه ذخیره‌سازی `v1beta1` است. شما یک شی می‌سازید. این در نسخه `v1beta1` ذخیره می‌شود.
2. شما نسخه `v1` را به CustomResourceDefinition اضافه می‌کنید و آن را به عنوان نسخه ذخیره‌سازی معرفی می‌کنید. در اینجا، اسکیماها برای `v1` و `v1beta1` یکسان هستند، که به طور معمول در مورد تبدیل API به پایدار در جامعه Kubernetes رخ می‌دهد.
3. شما شی را در نسخه `v1beta1` می‌خوانید، سپس دوباره شی را در نسخه `v1` می‌خوانید. هر دو شی متفاوت به جز فیلد apiVersion هستند.
4. یک شی جدید ایجاد می‌کنید. این در نسخه `v1` ذخیره می‌شود. اکنون دو شی دارید، یکی از آن‌ها در `v1beta1` و دیگری در `v1`.
5. شما اولین شی را به‌روزرسانی می‌کنید. اکنون در نسخه `v1` ذخیره می‌شود زیرا این نسخه ذخیره‌سازی فعلی است.

### نسخه‌های قبلی ذخیره‌سازی

سرور API هر نسخه‌ای که هرگز به عنوان نسخه ذخیره‌سازی علامت‌گذاری نشده است را در فیلد وضعیت `storedVersions` ذخیره می‌کند. اشیاء ممکن است در هر نسخه‌ای که هرگز به عنوان نسخه ذخیره‌سازی علامت‌گذاری نشده است، در ذخیره باشند. هیچ شیء نمی‌تواند در ذخیره در نسخه‌ای وجود داشته باشد که هرگز نسخه ذخیره‌سازی نبوده است.

## ارتقا شی‌های موجود به نسخه جدید ذخیره‌سازی

هنگام قدیمی کردن نسخه‌ها و حذف پشتیبانی، یک روش ارتقاء ذخیره را انتخاب کنید.

*گزینه 1:* استفاده از مهاجر ذخیره نسخه

1. اجرای [مهاجر ذخیره نسخه](https://github.com/kubernetes-sigs/kube-storage-version-migrator)
2. حذف نسخه قدیمی از فیلد `status.storedVersions` CustomResourceDefinition.

*گزینه 2:* به‌روزرسانی دستی اشیاء موجود به نسخه جدید ذخیره‌سازی

مثالی از روند به‌روزرسانی به `v1` از `v1beta1`:

1. تنظیم `v1` را به عنوان ذخ

یره در فایل CustomResourceDefinition و آن را با استفاده از kubectl اعمال کنید. `storedVersions` در حال حاضر `v1beta1, v1` است.
2. یک روند به‌روزرسانی ایجاد کنید تا تمام اشیاء موجود را فهرست کنید و آن‌ها را با محتوای مشابه نوشته شوند. این باعث می‌شود که بک‌اند اشیاء را در نسخه ذخیره کند که در حال حاضر `v1` است.
3. `v1beta1` را از CustomResourceDefinition `status.storedVersions` حذف کنید.

{{< note >}}
پرچم `--subresource` با دستورات get، patch، edit و replace kubectl برای دریافت و به‌روزرسانی زیرمنابع، `status` و `scale` برای همه منابع API که از آن‌ها پشتیبانی می‌کنند، استفاده می‌شود. این پرچم از نسخه kubectl v1.24 به بعد در دسترس است. قبلاً خواندن زیرمنابع (مانند `status`) از طریق kubectl شامل استفاده از `kubectl --raw` بود و به‌روزرسانی زیرمنابع با استفاده از kubectl به هیچ وجه ممکن نبود. از v1.24 به بعد، ابزار `kubectl` برای ویرایش یا پچ زیرمنابع روی یک شی CRD می‌تواند استفاده شود. برای مشاهده [چگونگی پچ کردن یک اجرا با استفاده از پرچم زیرمنبع](/docs/tasks/manage-kubernetes-objects/update-api-object-kubectl-patch/#scale-kubectl-patch).

این صفحه بخشی از مستندات Kubernetes v{{< skew currentVersion >}} است.
اگر یک نسخه متفاوت از Kubernetes را اجرا می‌کنید، با مستندات آن نسخه مشورت کنید.

اینجا نمونه‌ای از چگونگی پچ کردن زیرمنبع `status` برای یک شی CRD با استفاده از `kubectl` است:
```bash
kubectl patch customresourcedefinitions <CRD_Name> --subresource='status' --type='merge' -p '{"status":{"storedVersions":["v1"]}}'
```
{{< /note >}}