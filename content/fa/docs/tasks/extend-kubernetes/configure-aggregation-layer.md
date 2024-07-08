---
title: تنظیم لایه تجمیع
reviewers:
- lavalamp
- cheftako
- chenopis
content_type: task
weight: 10
---

<!-- overview -->

تنظیم [لایه تجمیع](/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/)
به سرور API Kubernetes اجازه می‌دهد تا با API‌های اضافی که بخشی از API‌های اصلی Kubernetes نیستند، گسترش یابد.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

{{< note >}}
چند نیازمندی برای راه‌اندازی لایه تجمیع در محیط شما وجود دارد تا از تأیید متقابل TLS بین پراکسی و سرورهای API توسعه پشتیبانی کند.
Kubernetes و kube-apiserver دارای چندین CA هستند، بنابراین اطمینان حاصل کنید که پراکسی توسط CA لایه تجمیع امضا شده است و نه توسط چیزی دیگر مانند CA عمومی Kubernetes.
{{< /note >}}

{{< caution >}}
استفاده مجدد از همان CA برای انواع مختلف مشتری می‌تواند تأثیر منفی بر توانایی کلاستر در عملکرد داشته باشد. برای اطلاعات بیشتر، به [استفاده مجدد از CA و تضادها](#ca-reusage-and-conflicts) مراجعه کنید.
{{< /caution >}}

<!-- steps -->

## جریان تأیید اعتبار

برخلاف تعریف منابع سفارشی (CRD)، API تجمیع شامل یک سرور دیگر - سرور API توسعه شما - علاوه بر سرور استاندارد Kubernetes می‌باشد.
سرور API Kubernetes باید با سرور API توسعه شما ارتباط برقرار کند، و سرور API توسعه شما باید با سرور API Kubernetes ارتباط برقرار کند.
برای اینکه این ارتباط ایمن شود، سرور API Kubernetes از گواهی‌های x509 برای تأیید خود به سرور API توسعه استفاده می‌کند.

این بخش توضیح می‌دهد که جریان‌های تأیید اعتبار و مجوز چگونه کار می‌کنند و چگونه آن‌ها را تنظیم کنید.

جریان کلی به شرح زیر است:

1. سرور API Kubernetes: کاربر درخواست‌کننده را تأیید اعتبار کرده و حقوق آنها را برای مسیر API درخواست شده مجوز می‌دهد.
2. سرور API Kubernetes: درخواست را به سرور API توسعه پراکسی می‌کند.
3. سرور API توسعه: درخواست از سرور API Kubernetes را تأیید اعتبار می‌کند.
4. سرور API توسعه: درخواست از کاربر اصلی را مجوز می‌دهد.
5. سرور API توسعه: اجرا می‌کند.

بخش‌های باقیمانده این مراحل را به تفصیل توضیح می‌دهند.

جریان را می‌توان در نمودار زیر مشاهده کرد.

![جریان‌های تأیید اعتبار تجمیع](/images/docs/aggregation-api-auth-flow.png)

منبع برای خطوط شنا در بالا را می‌توان در منبع این سند یافت.

<!--
Swimlanes generated at https://swimlanes.io with the source as follows:

-----BEGIN-----
title: Welcome to swimlanes.io


User -> kube-apiserver / aggregator:

note:
1. The user makes a request to the Kube API server using any recognized credential (e.g. OIDC or client certs)

kube-apiserver / aggregator -> kube-apiserver / aggregator: authentication

note:
2. The Kube API server authenticates the incoming request using any configured
   authentication methods (e.g. OIDC or client certs)

kube-apiserver / aggregator -> kube-apiserver / aggregator: authorization

note:
3. The Kube API server authorizes the requested URL using any configured authorization method (e.g. RBAC)

kube-apiserver / aggregator -> aggregated apiserver:

note:
4. The aggregator opens a connection to the aggregated API server using
   `--proxy-client-cert-file`/`--proxy-client-key-file` client certificate/key to secure the channel
5. The aggregator sends the user info from step 1 to the aggregated API server as
   http headers, as defined by the following flags:
  * `--requestheader-username-headers`
  * `--requestheader-group-headers`
  * `--requestheader-extra-headers-prefix`

aggregated apiserver -> aggregated apiserver: authentication

note:
6. The aggregated apiserver authenticates the incoming request using the auth proxy authentication method:
  * verifies the request has a recognized auth proxy client certificate
  * pulls user info from the incoming request's http headers

By default, it pulls the configuration information for this from a configmap
in the kube-system namespace that is published by the kube-apiserver,
containing the info from the `--requestheader-...` flags provided to the
kube-apiserver (CA bundle to use, auth proxy client certificate names to allow,
http header names to use, etc)

aggregated apiserver -> kube-apiserver / aggregator: authorization

note:
7. The aggregated apiserver authorizes the incoming request by making a
   SubjectAccessReview call to the kube-apiserver

aggregated apiserver -> aggregated apiserver: admission

note:
8. For mutating requests, the aggregated apiserver runs admission checks.
   by default, the namespace lifecycle admission plugin ensures namespaced
   resources are created in a namespace that exists in the kube-apiserver
-----END-----
-->

### احراز هویت و مجوزدهی در Kubernetes Apiserver

درخواستی به مسیری از API که توسط یک سرور API توسعه‌دهنده (extension apiserver) ارائه می‌شود، همانند تمام درخواست‌های API آغاز می‌شود: ارتباط با Kubernetes apiserver. این مسیر قبلاً توسط سرور API توسعه‌دهنده با Kubernetes apiserver ثبت شده است.

کاربر با Kubernetes apiserver ارتباط برقرار کرده و درخواست دسترسی به مسیر را می‌کند. Kubernetes apiserver از احراز هویت و مجوزدهی استاندارد تنظیم شده با Kubernetes apiserver برای احراز هویت کاربر و مجوز دسترسی به مسیر خاص استفاده می‌کند.

برای یک مرور کلی از احراز هویت به یک کلاستر Kubernetes، به ["احراز هویت به یک کلاستر"](/docs/reference/access-authn-authz/authentication/) مراجعه کنید.
برای یک مرور کلی از مجوزدهی دسترسی به منابع کلاستر Kubernetes، به ["مرور مجوزدهی"](/docs/reference/access-authn-authz/authorization/) مراجعه کنید.

تا این نقطه همه چیز درخواست‌های استاندارد API Kubernetes، احراز هویت و مجوزدهی بوده است.

Kubernetes apiserver اکنون آماده ارسال درخواست به سرور API توسعه‌دهنده است.

### پروکسی کردن درخواست توسط Kubernetes Apiserver

Kubernetes apiserver اکنون درخواست را به سرور API توسعه‌دهنده که ثبت شده برای مدیریت درخواست ارسال یا پروکسی می‌کند. برای انجام این کار، باید چند نکته را بداند:

1. چگونه باید Kubernetes apiserver به سرور API توسعه‌دهنده احراز هویت کند و به سرور API توسعه‌دهنده اطلاع دهد که درخواست که از طریق شبکه می‌آید از یک Kubernetes apiserver معتبر است؟
2. چگونه باید Kubernetes apiserver به سرور API توسعه‌دهنده اطلاع دهد که نام کاربری و گروهی که درخواست اصلی با آن احراز هویت شده است، چیست؟

برای فراهم کردن این دو مورد، باید Kubernetes apiserver را با استفاده از چند فلگ تنظیم کنید.

#### احراز هویت مشتری در Kubernetes Apiserver

Kubernetes apiserver از طریق TLS به سرور API توسعه‌دهنده متصل می‌شود و خود را با استفاده از یک گواهی مشتری احراز هویت می‌کند. باید موارد زیر را به Kubernetes apiserver در زمان راه‌اندازی فراهم کنید، با استفاده از فلگ‌های مربوطه:

* فایل کلید خصوصی از طریق `--proxy-client-key-file`
* فایل گواهی مشتری امضا شده از طریق `--proxy-client-cert-file`
* گواهی CA که فایل گواهی مشتری را امضا کرده است از طریق `--requestheader-client-ca-file`
* مقادیر معتبر نام‌های مشترک (CNs) در گواهی مشتری امضا شده از طریق `--requestheader-allowed-names`

Kubernetes apiserver از فایل‌هایی که توسط `--proxy-client-*-file` مشخص شده‌اند برای احراز هویت به سرور API توسعه‌دهنده استفاده خواهد کرد. برای اینکه درخواست توسط سرور API توسعه‌دهنده مطابق با استاندارد معتبر در نظر گرفته شود، باید شرایط زیر برآورده شوند:

1. اتصال باید با استفاده از گواهی مشتری که توسط CA امضا شده است که گواهی آن در `--requestheader-client-ca-file` است، برقرار شود.
2. اتصال باید با استفاده از گواهی مشتری که CN آن یکی از موارد لیست شده در `--requestheader-allowed-names` است، برقرار شود.

{{< note >}}
می‌توانید این گزینه را به صورت خالی تنظیم کنید یعنی `--requestheader-allowed-names=""`. این نشان می‌دهد که به سرور API توسعه‌دهنده هر CN قابل قبول است.
{{< /note >}}

وقتی با این گزینه‌ها راه‌اندازی شد، Kubernetes apiserver:

1. از آنها برای احراز هویت به سرور API توسعه‌دهنده استفاده می‌کند.
2. یک configmap در namespace `kube-system` به نام `extension-apiserver-authentication` ایجاد می‌کند که در آن گواهی CA و CNهای مجاز را قرار می‌دهد. این موارد سپس توسط سرورهای API توسعه‌دهنده قابل بازیابی و استفاده برای اعتبارسنجی درخواست‌ها هستند.

توجه داشته باشید که همان گواهی مشتری توسط Kubernetes apiserver برای احراز هویت در برابر _همه_ سرورهای API توسعه‌دهنده استفاده می‌شود. Kubernetes apiserver گواهی مشتری جداگانه‌ای برای هر سرور API توسعه‌دهنده ایجاد نمی‌کند، بلکه از یک گواهی استفاده می‌کند تا به عنوان Kubernetes apiserver احراز هویت کند. همین گواهی برای همه درخواست‌های سرور API توسعه‌دهنده استفاده می‌شود.

#### نام کاربری و گروه درخواست اصلی

وقتی Kubernetes apiserver درخواست را به سرور API توسعه‌دهنده پروکسی می‌کند، به سرور API توسعه‌دهنده اطلاع می‌دهد که نام کاربری و گروهی که درخواست اصلی با آن احراز هویت شده است، چیست. این اطلاعات را در هدرهای http درخواست پروکسی شده قرار می‌دهد. باید نام‌های هدرها را به Kubernetes apiserver اطلاع دهید.

* هدر برای ذخیره نام کاربری از طریق `--requestheader-username-headers`
* هدر برای ذخیره گروه از طریق `--requestheader-group-headers`
* پیشوندی برای افزودن به تمام هدرهای اضافی از طریق `--requestheader-extra-headers-prefix`

این نام‌های هدر نیز در configmap `extension-apiserver-authentication` قرار می‌گیرند، بنابراین می‌توانند توسط سرورهای API توسعه‌دهنده بازیابی و استفاده شوند.

### احراز هویت درخواست توسط سرور API توسعه‌دهنده

سرور API توسعه‌دهنده، هنگام دریافت یک درخواست پروکسی شده از Kubernetes apiserver، باید تأیید کند که درخواست واقعاً از یک پروکسی معتبر احراز هویت کننده آمده است، که این نقش توسط Kubernetes apiserver ایفا می‌شود. سرور API توسعه‌دهنده آن را از طریق موارد زیر اعتبارسنجی می‌کند:

1. بازیابی موارد زیر از configmap در namespace `kube-system`، همان‌طور که در بالا توضیح داده شد:
    * گواهی CA مشتری
    * لیست نام‌های مجاز (CNها)
    * نام‌های هدر برای نام کاربری، گروه و اطلاعات اضافی
2. بررسی اینکه اتصال TLS با استفاده از گواهی مشتری که:
    * توسط CA که گواهی آن با گواهی CA بازیابی شده مطابقت دارد، امضا شده است.
    * CN آن در لیست CNهای مجاز قرار دارد، مگر اینکه لیست خالی باشد که در این صورت همه CNها مجاز هستند.
    * استخراج نام کاربری و گروه از هدرهای مناسب

اگر موارد بالا عبور کنند، درخواست یک درخواست پروکسی شده معتبر از یک پروکسی احراز هویت کننده معتبر است، در این مورد Kubernetes apiserver.

توجه داشته باشید که این مسئولیت اجرای سرور API توسعه‌دهنده است که موارد فوق را فراهم کند. بسیاری از آنها این کار را به صورت پیش‌فرض انجام می‌دهند و از پکیج `k8s.io/apiserver/` استفاده می‌کنند. برخی دیگر ممکن است گزینه‌هایی برای جایگزینی آن با استفاده از گزینه‌های خط فرمان ارائه دهند.

برای داشتن اجازه برای بازیابی configmap، یک سرور API توسعه‌دهنده نیاز به نقش مناسب دارد. یک نقش پیش‌فرض به نام `extension-apiserver-authentication-reader` در namespace `kube-system` وجود دارد که می‌توان آن را اختصاص داد.

### مجوزدهی درخواست توسط Extension Apiserver

سرور API توسعه‌دهنده اکنون می‌تواند تأیید کند که کاربر/گروه بازیابی شده از هدرها مجاز به اجرای درخواست داده شده هستند. این کار را با ارسال یک درخواست استاندارد [SubjectAccessReview](/docs/reference/access-authn-authz/authorization/) به Kubernetes apiserver انجام می‌دهد.

برای اینکه سرور API توسعه‌دهنده خود مجاز به ارسال درخواست `SubjectAccessReview` به Kubernetes apiserver باشد، به مجوزهای صحیح نیاز دارد. Kubernetes شامل یک `ClusterRole` پیش‌فرض به نام `system:auth-delegator` است که مجوزهای مناسب را دارد. می‌توان این نقش را به حساب سرویس سرور API توسعه‌دهنده اعطا کرد.

### اجرای درخواست توسط Extension Apiserver

اگر `SubjectAccessReview` تأیید شود، سرور API توسعه‌دهنده درخواست را اجرا می‌کند.

## فعال‌سازی فلگ‌های Kubernetes Apiserver

لایه تجمیع را از طریق فلگ‌های `kube-apiserver` زیر فعال کنید. ممکن است این موارد قبلاً توسط ارائه‌دهنده شما انجام شده باشند.

```
--requestheader-client-ca-file=<path to aggregator CA cert>
--requestheader-allowed-names=front-proxy-client
--requestheader-extra-headers-prefix=X-Remote-Extra-
--requestheader-group-headers=X-Remote-Group
--requestheader-username-headers=X-Remote-User
--proxy-client-cert-file=<path to aggregator proxy cert>
--proxy-client-key-file=<path to aggregator proxy key>
```

### استفاده مجدد و تضادهای CA

Kubernetes apiserver دو گزینه CA مشتری دارد:

* `--client-ca-file`
* `--requestheader-client-ca-file`

هر یک از این گزینه‌ها به صورت مستقل عمل می‌کنند و در صورت استفاده نادرست می‌توانند با یکدیگر تضاد داشته باشند.

* `--client-ca-file`: وقتی درخواستی به Kubernetes apiserver می‌رسد، اگر این گزینه فعال باشد، Kubernetes apiserver گواهی درخواست را بررسی می‌کند. اگر توسط یکی از گواهی‌های CA در فایل مرجع شده توسط `--client-ca-file` امضا شده باشد، درخواست به عنوان یک درخواست معتبر تلقی می‌شود و کاربر مقدار نام مشترک `CN=` است، در حالی که گروه سازمان `O=` است. به [مستندات احراز هویت TLS](/docs/reference/access-authn-authz/authentication/#x509-client-certificates) مراجعه کنید.
* `--requestheader-client-ca-file`: وقتی درخواستی به Kubernetes apiserver می‌رسد، اگر این گزینه فعال باشد، Kubernetes apiserver گواهی درخواست را بررسی می‌کند. اگر توسط یکی از گواهی‌های CA در فایل مرجع شده توسط `--requestheader-client-ca-file` امضا شده باشد، درخواست به عنوان یک درخواست بالقوه معتبر تلقی می‌شود. Kubernetes apiserver سپس بررسی می‌کند که آیا نام مشترک `CN=` یکی از نام‌های موجود در لیست ارائه شده توسط `--requestheader-allowed-names` است یا خیر. اگر نام مجاز باشد، درخواست تأیید می‌شود؛ اگر نباشد، درخواست رد می‌شود.

اگر _هر دو_ `--client-ca-file` و `--requestheader-client-ca-file` فراهم شده باشند، ابتدا درخواست CA `--requestheader-client-ca-file` و سپس `--client-ca-file` را بررسی می‌کند. معمولاً CAهای مختلف، یا CAهای ریشه یا CAهای میانی، برای هر یک از این گزینه‌ها استفاده می‌شوند؛ درخواست‌های مشتری معمولی با `--client-ca-file` مطابقت دارند، در حالی که درخواست‌های تجمیع با `--requestheader-client-ca-file` مطابقت دارند. اما اگر هر دو از _همان_ CA استفاده کنند، درخواست‌های مشتری که معمولاً از طریق `--client-ca-file` عبور می‌کنند، رد خواهند شد، زیرا CA با CA در `--requestheader-client-ca-file` مطابقت خواهد داشت، اما نام مشترک `CN=` با یکی از نام‌های قابل قبول در `--requestheader-allowed-names` **مطابقت نخواهد داشت**. این می‌تواند باعث شود که kubelets و سایر اجزای کنترل پلین، و همچنین کاربران نهایی، نتوانند به Kubernetes apiserver احراز هویت کنند.

به همین دلیل، از گواهی‌های CA مختلف برای گزینه `--client-ca-file` - برای مجوزدهی اجزای کنترل پلین و کاربران نهایی - و گزینه `--requestheader-client-ca-file` - برای مجوزدهی درخواست‌های سرور API تجمیع - استفاده کنید.

{{< warning >}}
از استفاده مجدد یک CA که در یک زمینه متفاوت استفاده شده است خودداری کنید مگر اینکه خطرات و مکانیسم‌های محافظت از استفاده از CA را به خوبی درک کنید.
{{< /warning >}}

اگر kube-proxy را بر روی سیستمی که API server را اجرا می‌کند، اجرا نمی‌کنید، باید مطمئن شوید که سیستم با فلگ `kube-apiserver` زیر فعال شده است:

```
--enable-aggregator-routing=true
```

### ثبت اشیاء APIService

می‌توانید به صورت پویا پیکربندی کنید که چه درخواست‌های مشتری به سرور API توسعه‌دهنده پروکسی شوند. نمونه‌ای از ثبت به صورت زیر است:

```yaml
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  name: <name of the registration object>
spec:
  group: <API group name this extension apiserver hosts>
  version: <API version this extension apiserver hosts>
  groupPriorityMinimum: <priority this APIService for this group, see API documentation>
  versionPriority: <prioritizes ordering of this version within a group, see API documentation>
  service:
    namespace: <namespace of the extension apiserver service>
    name: <name of the extension apiserver service>
  caBundle: <pem encoded ca cert that signs the server cert used by the webhook>
```

نام یک شیء APIService باید یک [نام قطعه مسیر](/docs/concepts/overview/working-with-objects/names#path-segment-names) معتبر باشد.

#### تماس با سرور API توسعه‌دهنده

پس از اینکه Kubernetes apiserver تعیین کرد که درخواستی باید به سرور API توسعه‌دهنده ارسال شود، باید بداند چگونه با آن تماس بگیرد.

قسمت `service` یک مرجع به سرویس برای یک سرور API توسعه‌دهنده است. namespace و نام سرویس الزامی هستند. پورت اختیاری است و به طور پیش‌فرض 443 است.

در اینجا مثالی از یک سرور API توسعه‌دهنده است که پیکربندی شده است تا در پورت "1234" فراخوانی شود و اتصال TLS را با نام سرور `my-service-name.my-service-namespace.svc` با استفاده از باندل CA سفارشی تأیید کند.

```yaml
apiVersion: apiregistration.k8s.io/v1
kind: APIService
...
spec:
  ...
  service:
    namespace: my-service-namespace
    name: my-service-name
    port: 1234
  caBundle: "Ci0tLS0tQk...<base64-encoded PEM bundle>...tLS0K"
...
```

## {{% heading "whatsnext" %}}

* [راه‌اندازی یک سرور API توسعه‌دهنده](/docs/tasks/extend-kubernetes/setup-extension-api-server/) برای کار با لایه تجمیع.
* برای یک مرور کلی، به [توسعه API Kubernetes با لایه تجمیع](/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/) مراجعه کنید.
* بیاموزید چگونه [API Kubernetes را با استفاده از تعاریف منابع سفارشی توسعه دهید](/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/).
