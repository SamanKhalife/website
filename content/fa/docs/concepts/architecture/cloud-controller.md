---
title: مدیر کنترلر ابری (Cloud Controller Manager)
content_type: مفهوم
weight: 40
---

<!-- مرور -->

{{< feature-state state="beta" for_k8s_version="v1.11" >}}

تکنولوژی‌های زیرساخت ابری به شما این امکان را می‌دهند که Kubernetes را بر روی ابرهای عمومی، خصوصی و هیبرید اجرا کنید. Kubernetes به این باور است که زیرساخت اتوماتیک و مبتنی بر API بدون اتصال شدید بین اجزا باید انجام شود.

{{< glossary_definition term_id="cloud-controller-manager" length="all" prepend="مدیر کنترلر ابری (Cloud Controller Manager) یک ساختار استفاده می‌شود که از طریق مکانیزم پلاگین به ابرهای مختلف این امکان را می‌دهد تا پلتفرم‌های خود را با Kubernetes ادغام کنند." >}}

<!-- محتوا -->

## طراحی

![اجزا Kubernetes](/images/docs/components-of-kubernetes.svg)

مدیر کنترلر ابری به عنوان یک مجموعه تکراری از فرآیندها (معمولاً این‌ها کانتینرها در پادها هستند) در صفحه کنترل اجرا می‌شود. هر مدیر کنترلر ابری، چندین {{< glossary_tooltip text="کنترلر" term_id="controller" >}} را در یک فرآیند واحد اجرا می‌کند.

{{< note >}}
شما همچنین می‌توانید مدیر کنترلر ابری را به عنوان یک {{< glossary_tooltip text="افزونه" term_id="addons" >}} Kubernetes اجرا کنید به جای آنکه جزء صفحه کنترل باشد.
{{< /note >}}

## عملکردهای مدیر کنترلر ابری {#functions-of-the-ccm}

کنترلرهای داخل مدیر کنترلر ابری شامل موارد زیر می‌شود:

### کنترلر گره (Node controller)

کنترلر گره مسئول به روزرسانی اشیاء {{< glossary_tooltip text="گره" term_id="node" >}} هنگامی که سرورهای جدید در زیرساخت ابری شما ایجاد می‌شوند است. کنترلر گره اطلاعات درباره میزبان‌های در حال اجرای داخل استفاده از ابر ارائه می‌دهد. کنترلر گره وظایف زیر را انجام می‌دهد:

1. به روزرسانی یک شیء گره با شناسه یکتای سروری که از API ارائه دهنده ابر دریافت شده است.
1. ارائه یادداشت و برچسب‌گذاری شیء گره با اطلاعات خاص ابری مانند منطقه‌ای که گره در آن اجرا می‌شود و منابع (CPU، حافظه و غیره) که در دسترس آن دارد.
1. دریافت نام میزبان و آدرس‌های شبکه گره.
1. تأیید سلامتی گره. در صورتی که یک گره پاسخ ندهد، این کنترلر با API ارائه دهنده ابری شما بررسی می‌کند که آیا سرور غیر فعال / حذف شده / خاتمه یافته است. اگر گره از ابر حذف شده باشد، کنترلر شیء گره را از خوشه Kubernetes شما حذف می‌کند.

بعضی پیاده سازی‌های ارائه دهنده ابر این کار را به دو کنترلر گره و کنترلر چرخه عمر گره جداگانه تقسیم می‌کنند.

### کنترلر مسیر (Route controller)

کنترلر مسئول پیکربندی مسیرها در ابر است تا کانتینرها در گره‌های مختلف در خوشه Kubernetes شما بتوانند با یکدیگر ارتباط برقرار کنند.

بسته به ارائه دهنده ابر، کنترلر مسیر ممکن است بلوک‌های آدرس IP برای شبکه پاد را اختصاص دهد.

### کنترلر خدمات (Service controller)

{{< glossary_tooltip text="خدمات" term_id="service" >}} با اجزای زیرساخت ابری مانند توزیع‌کننده بار مدیریت شده، آدرس‌های IP، فیلترینگ بسته شبکه و بررسی سلامت هدف ادغام می‌کنند. کنترلر خدمات با API‌های ارائه دهنده ابر شما تعامل می‌کند تا زمانی که یک منبع خدمتی را اعلام می‌کنید، توزیع‌کننده بارها و سایر اجزای زیرساخت راه اندازی کند که نیاز دارید.

## مجوزدهی

این بخش دسترسی‌هایی را که مدیر کنترلر ابری بر روی اشیاء مختلف API نیاز دارد برای انجام عملیات خود شرح می‌دهد.

### کنترلر گره (Node controller) {#authorization-node-controller}

کنترلر گره تنها با اشیاء گره کار می‌کند. او نیاز به دسترسی کامل برای خواندن و اصلاح اشیاء گره دارد.

`v1/Node`:

- get
- list
- create
- update
- patch
- watch
- delete

### کنترلر مسیر (Route controller) {#authorization-route-controller}

کنترلر مسیر به ایجاد اشیاء گره گوش می‌دهد و مسیرها را به‌طور مناسب پیکربندی می‌کند. برای دسترسی به اشیاء گره، دسترسی Get نیاز دارد.

`v1/Node`:

- get

### کنترلر خدمات (Service controller) {#authorization-service-controller}

کنترلر خدمات برای رویدادهای ایجاد، به‌روزرسانی و حذف شیء خدمات نظارت می‌کند و سپس Endpoints مورد نظر را به‌طور مناسب پیکربندی می‌کند (برای EndpointSlices، kube-controller-manager این‌ها را به‌طور درخواستی مدیریت می‌کند).

برای دسترسی به خدمات، نیاز به دسترسی‌های list و watch دارد. برای به‌روزرسانی خدمات، نیاز به دسترسی‌های patch و update دارد.

برای راه‌اندازی منابع Endpoints برای خدمات، نیاز به دسترسی‌های create، list، get، watch و update دارد.

`v1/Service`:

- list
- get
- watch
- patch
- update

### موارد دیگر {#authorization-miscellaneous}

برای پیاده‌سازی هسته مرکزی مدیر کنترلر ابری، دسترسی به ایجاد اشیاء رویداد و برای اطمینان از عملیات امن، دسترسی به ایجاد ServiceAccounts نیاز دارد.

`v1/Event`:

- create
- patch
- update

`v1/ServiceAccount`:

- create

نقش ClusterRole {{< glossary_tooltip term_id="rbac" text="RBAC" >}} برای مدیر کنترلر ابری به صورت زیر است:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cloud-controller-manager
rules:
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - create
  - patch
  - update
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - '*'
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - list
  - patch
  - update
  - watch
- apiGroups:
  - ""
  resources:
  - serviceaccounts
  verbs:
  - create
- apiGroups:
  - ""
  resources:
  - persistentvolumes
  verbs:
  - get
  - list
  - update
  - watch
- apiGroups:
  - ""
  resources:
  - endpoints
  verbs:
  - create
  - get
  - list
  - watch
  - update
```

## {{% heading "whatsnext" %}}

* [مدیریت مدیر کنترلر ابری](/docs/tasks/administer-cluster/running-cloud-controller/#cloud-controller-manager)
  دارای دستورالعمل‌هایی برای اجرا و مدیریت مدیر کنترلر ابری است.

* برای ارتقا یک صفحه کنترل HA به استفاده از مدیر کنترلر ابری، [مهاجرت کنترل پنهان به استفاده از مدیر کنترلر ابری](/docs/tasks/administer-cluster/controller-manager-leader-migration/) را ببینید.

* آیا می‌خواهید بدانید که چگونه می‌توانید مدیر کنترلر ابری خود را پیاده‌سازی کنید یا یک پروژه موجود را گسترش دهید؟

  - مدیر کنترلر ابری از رابط‌های Go استفاده می‌کند، به خصوص رابط `CloudProvider` که در [`cloud.go`](https://github.com/kubernetes/cloud-provider/blob/release-1.21/cloud.go#L42-L69)
    از [kubernetes/cloud-provider](https://github.com/kubernetes/cloud-provider) تعریف شده است تا اجراهای از هر ابری را پیاده سازی کند.
  - پیاده‌سازی کنترلرهای مشترک بین‌المللی معرفی شده در این سند (گره، مسیر و خدمات) و برخی از چارچوب‌های پیاده‌سازی از جمله رابط ابر مشترک، قسمتی از هسته Kubernetes است.
    پیاده‌سازی‌های خاص به ارائه دهندگان ابر خارج از هسته Kubernetes است و رابط `CloudProvider` را پیاده‌سازی می‌کند.
  - برای اطلاعات بیشتر درباره توسعه پلاگین‌ها، [پیاده‌سازی مدیر کنترلر ابری](/docs/tasks/administer-cluster/developing-cloud-controller-manager/) را ببینید.
