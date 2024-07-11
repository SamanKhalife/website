---
title: پلاگین‌های دستگاه
description: >
  پلاگین‌های دستگاه به شما اجازه می‌دهند که خوشه خود را با پشتیبانی از دستگاه‌ها یا منابعی که نیاز به تنظیمات خاص فروشنده دارند، مانند GPUها، NICها، FPGAها، یا حافظه اصلی غیر فرار، پیکربندی کنید.
content_type: concept
weight: 20
---

<!-- overview -->
{{< feature-state for_k8s_version="v1.26" state="stable" >}}

Kubernetes یک چارچوب پلاگین دستگاه فراهم می‌کند که می‌توانید از آن برای تبلیغ منابع سخت‌افزاری سیستم به {{< glossary_tooltip term_id="kubelet" >}} استفاده کنید.

به جای سفارشی‌سازی کد Kubernetes، فروشندگان می‌توانند یک پلاگین دستگاه پیاده‌سازی کنند که شما می‌توانید آن را به صورت دستی یا به عنوان یک {{< glossary_tooltip term_id="daemonset" >}} پیاده‌سازی کنید. دستگاه‌های هدف شامل GPUها، NICهای با عملکرد بالا، FPGAها، آداپتورهای InfiniBand و سایر منابع محاسباتی مشابه هستند که ممکن است به مقداردهی اولیه و تنظیمات خاص فروشنده نیاز داشته باشند.

<!-- body -->

## ثبت پلاگین دستگاه

kubelet یک سرویس `Registration` gRPC صادر می‌کند:

```gRPC
service Registration {
	rpc Register(RegisterRequest) returns (Empty) {}
}
```

یک پلاگین دستگاه می‌تواند از طریق این سرویس gRPC با kubelet ثبت شود. در طول ثبت، پلاگین دستگاه باید اطلاعات زیر را ارسال کند:

* نام سوکت Unix آن.
* نسخه API پلاگین دستگاه که بر اساس آن ساخته شده است.
* `ResourceName` که می‌خواهد تبلیغ کند. این `ResourceName` باید از
  [طرح نام‌گذاری منابع توسعه‌یافته](/docs/concepts/configuration/manage-resources-containers/#extended-resources)
  به صورت `vendor-domain/resourcetype` پیروی کند.
  (برای مثال، یک GPU NVIDIA به صورت `nvidia.com/gpu` تبلیغ می‌شود.)

پس از یک ثبت موفق، پلاگین دستگاه لیست دستگاه‌هایی که مدیریت می‌کند را به kubelet ارسال می‌کند و سپس kubelet مسئول تبلیغ این منابع به سرور API به عنوان بخشی از به‌روزرسانی وضعیت گره kubelet است.
برای مثال، پس از اینکه یک پلاگین دستگاه `hardware-vendor.example/foo` را با kubelet ثبت می‌کند و دو دستگاه سالم در یک گره گزارش می‌دهد، وضعیت گره به‌روزرسانی می‌شود تا تبلیغ کند که گره دارای 2 دستگاه "Foo" نصب شده و در دسترس است.

سپس، کاربران می‌توانند دستگاه‌ها را به عنوان بخشی از مشخصات یک Pod درخواست کنند
(به [`container`](/docs/reference/kubernetes-api/workload-resources/pod-v1/#Container) مراجعه کنید).
درخواست منابع توسعه‌یافته مشابه مدیریت درخواست‌ها و محدودیت‌ها برای منابع دیگر است، با تفاوت‌های زیر:
* منابع توسعه‌یافته فقط به عنوان منابع صحیح پشتیبانی می‌شوند و نمی‌توانند بیش از حد تخصیص داده شوند.
* دستگاه‌ها نمی‌توانند بین کانتینرها به اشتراک گذاشته شوند.

### مثال {#example-pod}

فرض کنید یک خوشه Kubernetes یک پلاگین دستگاه اجرا می‌کند که منبع `hardware-vendor.example/foo` را روی برخی گره‌ها تبلیغ می‌کند. در اینجا مثالی از یک pod که این منبع را برای اجرای یک کار نمایشی درخواست می‌کند:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
    - name: demo-container-1
      image: registry.k8s.io/pause:2.0
      resources:
        limits:
          hardware-vendor.example/foo: 2
#
# این Pod به 2 دستگاه hardware-vendor.example/foo نیاز دارد
# و فقط می‌تواند روی گره‌ای برنامه‌ریزی شود که بتواند
# این نیاز را برآورده کند.
#
# اگر گره بیش از 2 از این دستگاه‌ها را در دسترس داشته باشد،
# باقی‌مانده برای استفاده سایر Podها در دسترس خواهد بود.
```

## پیاده‌سازی پلاگین دستگاه

روند کلی کار یک پلاگین دستگاه شامل مراحل زیر است:

1. مقداردهی اولیه. در این مرحله، پلاگین دستگاه تنظیمات خاص فروشنده و مقداردهی اولیه را انجام می‌دهد تا مطمئن شود دستگاه‌ها در حالت آماده به کار هستند.

1. پلاگین یک سرویس gRPC را با یک سوکت Unix تحت مسیر `/var/lib/kubelet/device-plugins/` شروع می‌کند که رابط‌های زیر را پیاده‌سازی می‌کند:

   ```gRPC
   service DevicePlugin {
         // GetDevicePluginOptions گزینه‌هایی را که باید با مدیر دستگاه منتقل شوند، برمی‌گرداند.
         rpc GetDevicePluginOptions(Empty) returns (DevicePluginOptions) {}

         // ListAndWatch جریان لیستی از دستگاه‌ها را برمی‌گرداند
         // هرگاه وضعیت دستگاه تغییر کند یا دستگاه ناپدید شود، ListAndWatch
         // لیست جدید را برمی‌گرداند
         rpc ListAndWatch(Empty) returns (stream ListAndWatchResponse) {}

         // Allocate در طول ایجاد کانتینر فراخوانی می‌شود تا پلاگین دستگاه
         // بتواند عملیات خاص دستگاه را انجام دهد و kubelet را از مراحل
         // موجود کردن دستگاه در کانتینر مطلع کند
         rpc Allocate(AllocateRequest) returns (AllocateResponse) {}

         // GetPreferredAllocation مجموعه‌ای ترجیحی از دستگاه‌ها را برای تخصیص
         // از لیست موجود برمی‌گرداند. تخصیص ترجیحی نتیجه‌شده
         // تضمین نمی‌شود که تخصیصی باشد که در نهایت توسط
         // مدیر دستگاه انجام شده است. این فقط برای کمک به مدیر دستگاه طراحی شده است
         // تصمیم‌گیری بهتر تخصیص در صورت امکان.
         rpc GetPreferredAllocation(PreferredAllocationRequest) returns (PreferredAllocationResponse) {}

         // PreStartContainer در صورت مشخص شدن توسط پلاگین دستگاه در مرحله ثبت‌نام،
         // قبل از شروع هر کانتینر فراخوانی می‌شود. پلاگین دستگاه می‌تواند
         // عملیات خاص دستگاه مانند بازنشانی دستگاه را قبل از موجود کردن دستگاه‌ها در کانتینر انجام دهد.
         rpc PreStartContainer(PreStartContainerRequest) returns (PreStartContainerResponse) {}
   }
   ```

   {{< note >}}
   پلاگین‌ها ملزم نیستند که پیاده‌سازی‌های مفیدی برای
   `GetPreferredAllocation()` یا `PreStartContainer()` ارائه دهند. علامت‌هایی که
   نشان‌دهنده در دسترس بودن این تماس‌ها هستند، باید در پیام `DevicePluginOptions`
   که با فراخوانی `GetDevicePluginOptions()` ارسال می‌شود، تنظیم شوند. `kubelet`
   همیشه `GetDevicePluginOptions()` را فراخوانی می‌کند تا ببیند کدام عملکردهای اختیاری
   در دسترس هستند، قبل از اینکه مستقیماً هر یک از آن‌ها را فراخوانی کند.
   {{< /note >}}

1. پلاگین با kubelet از طریق سوکت Unix در مسیر `/var/lib/kubelet/device-plugins/kubelet.sock` ثبت می‌شود.

   {{< note >}}
   ترتیب کار مهم است. یک پلاگین باید قبل از ثبت‌نام موفقیت‌آمیز با kubelet
   سرویس gRPC خود را شروع کند.
   {{< /note >}}

1. پس از ثبت موفق، پلاگین دستگاه در حالت سرویس‌دهی اجرا می‌شود، در این حالت سلامت دستگاه را نظارت می‌کند و به kubelet هرگونه تغییر وضعیت دستگاه را گزارش می‌دهد.
   همچنین مسئول پاسخ‌دهی به درخواست‌های gRPC `Allocate` است. در طول `Allocate`، پلاگین دستگاه ممکن است آماده‌سازی خاص دستگاه را انجام دهد؛ برای مثال، پاکسازی GPU یا مقداردهی اولیه QRNG.
   اگر عملیات موفق باشد، پلاگین دستگاه یک `AllocateResponse` برمی‌گرداند که شامل تنظیمات زمان اجرای کانتینر برای دسترسی به دستگاه‌های تخصیص داده شده است. kubelet این اطلاعات را به زمان اجرای کانتینر منتقل می‌کند.

   یک `AllocateResponse` شامل صفر یا بیشتر `ContainerAllocateResponse` است. در این‌ها، پلاگین دستگاه تغییراتی که باید در تعریف کانتینر برای فراهم کردن دسترسی به دستگاه اعمال شود را تعریف می‌کند. این تغییرات شامل موارد زیر است:

   * [Annotations](/docs/concepts/overview/working-with-objects/annotations/)
   * گره‌های دستگاه
   * متغیرهای محیطی
   * mounts
   * نام‌های دستگاه CDI کاملاً واجد شرایط

   {{< note >}}
   پردازش نام‌های دستگاه CDI کاملاً واجد شرایط توسط مدیر دستگاه نیاز دارد که
   دروازه ویژگی `DevicePluginCDIDevices` [دروازه ویژگی](/docs/reference/command-line-tools-reference/feature-gates/)
   برای هر دو kubelet و kube-apiserver فعال باشد. این به عنوان یک ویژگی آلفا در Kubernetes
   v1.28 اضافه شد و در v1.29 به بتا ارتقا یافت.
   {{< /note >}}

### مدیریت راه‌اندازی مجدد kubelet

یک پلاگین دستگاه انتظار می‌رود که راه‌اندازی مجدد kubelet را شناسایی کرده و خود را با نمونه جدید kubelet دوباره ثبت کند. یک نمونه جدید kubelet تمام سوکت‌های Unix موجود در مسیر `/var/lib/kubelet/device-plugins` را هنگام راه‌اندازی حذف می‌کند. یک پلاگین دستگاه می‌تواند حذف سوکت Unix خود را مانیتور کرده و در صورت وقوع چنین رویدادی خود را دوباره ثبت کند.

## استقرار پلاگین دستگاه

شما می‌توانید یک پلاگین دستگاه را به صورت DaemonSet، به عنوان یک بسته برای سیستم عامل گره‌های خود، یا به صورت دستی استقرار دهید.

دایرکتوری استاندارد `/var/lib/kubelet/device-plugins` به دسترسی دارای امتیاز نیاز دارد، بنابراین یک پلاگین دستگاه باید در یک محیط امنیتی دارای امتیاز اجرا شود.
اگر در حال استقرار یک پلاگین دستگاه به صورت DaemonSet هستید، باید `/var/lib/kubelet/device-plugins` به عنوان یک {{< glossary_tooltip term_id="volume" >}} در [PodSpec](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podspec-v1-core) پلاگین نصب شود.

اگر رویکرد DaemonSet را انتخاب کنید، می‌توانید به Kubernetes تکیه کنید تا: Pod پلاگین دستگاه را روی گره‌ها قرار دهد، daemon Pod را پس از خرابی راه‌اندازی مجدد کند، و به خودکارسازی به‌روزرسانی‌ها کمک کند.

## سازگاری API

قبلاً، نسخه‌بندی مورد نیاز بود که نسخه API پلاگین دستگاه دقیقاً با نسخه Kubelet مطابقت داشته باشد. از زمان ارتقاء این ویژگی به Beta در v1.12 این دیگر یک الزام سخت نیست. API نسخه‌بندی شده و از زمان ارتقاء به Beta پایدار بوده است. به همین دلیل، ارتقاء kubelet باید بدون مشکل باشد، اما هنوز ممکن است تغییراتی در API قبل از تثبیت وجود داشته باشد که باعث شود ارتقاء بدون مشکل تضمین نشود.

{{< note >}}
با وجود اینکه مؤلفه Device Manager از Kubernetes یک ویژگی به طور کلی در دسترس است، _API پلاگین دستگاه_ پایدار نیست. برای اطلاعات بیشتر در مورد نسخه‌های API پلاگین دستگاه و سازگاری نسخه‌ها، [نسخه‌های API پلاگین دستگاه](/docs/reference/node/device-plugin-api-versions/) را مطالعه کنید.
{{< /note >}}

به عنوان یک پروژه، Kubernetes توصیه می‌کند که توسعه‌دهندگان پلاگین دستگاه:

* به تغییرات API پلاگین دستگاه در نسخه‌های آینده توجه کنند.
* پشتیبانی از نسخه‌های متعدد API پلاگین دستگاه برای سازگاری به عقب/جلو.

برای اجرای پلاگین‌های دستگاه روی گره‌هایی که نیاز به ارتقاء به یک نسخه جدیدتر از Kubernetes با نسخه API پلاگین دستگاه جدید دارند، پلاگین‌های دستگاه خود را به گونه‌ای ارتقاء دهید که از هر دو نسخه پشتیبانی کنند قبل از ارتقاء این گره‌ها. این رویکرد تضمین می‌کند که تخصیص دستگاه‌ها در طول ارتقاء به صورت پیوسته عمل می‌کند.

## نظارت بر منابع پلاگین دستگاه

{{< feature-state for_k8s_version="v1.28" state="stable" >}}

برای نظارت بر منابعی که توسط پلاگین‌های دستگاه فراهم شده‌اند، عامل‌های نظارت باید بتوانند مجموعه دستگاه‌هایی که در گره در حال استفاده هستند را کشف کنند و متادیتایی برای توصیف اینکه کدام کانتینر باید متریک را مرتبط کند، به‌دست آورند. [Prometheus](https://prometheus.io/) متریک‌هایی که توسط عامل‌های نظارت دستگاه افشا می‌شود باید دستورالعمل‌های
[Kubernetes Instrumentation Guidelines](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-instrumentation/instrumentation.md) را دنبال کنند، و کانتینرها را با استفاده از برچسب‌های `pod`، `namespace` و `container` Prometheus شناسایی کنند.

kubelet یک سرویس gRPC فراهم می‌کند که امکان کشف دستگاه‌های در حال استفاده و ارائه متادیتا برای این دستگاه‌ها را فراهم می‌کند:

```gRPC
// PodResourcesLister سرویسی است که توسط kubelet فراهم شده و اطلاعاتی در مورد
// منابع گره‌ای که توسط podها و کانتینرها روی گره مصرف می‌شوند، فراهم می‌کند.
service PodResourcesLister {
    rpc List(ListPodResourcesRequest) returns (ListPodResourcesResponse) {}
    rpc GetAllocatableResources(AllocatableResourcesRequest) returns (AllocatableResourcesResponse) {}
    rpc Get(GetPodResourcesRequest) returns (GetPodResourcesResponse) {}
}
```

### نقطه پایانی `List` gRPC {#grpc-endpoint-list}

نقطه پایانی `List` اطلاعاتی در مورد منابع podهای در حال اجرا فراهم می‌کند، با جزئیاتی مانند شناسه CPUهای اختصاصی تخصیص یافته، شناسه دستگاه همانطور که توسط پلاگین‌های دستگاه گزارش شده و شناسه گره NUMA که این دستگاه‌ها در آن تخصیص یافته‌اند. همچنین، برای ماشین‌های مبتنی بر NUMA، اطلاعاتی در مورد حافظه و صفحات بزرگ رزرو شده برای یک کانتینر فراهم می‌کند.

از Kubernetes v1.27، نقطه پایانی `List` می‌تواند اطلاعاتی در مورد منابع podهای در حال اجرا که در `ResourceClaims` توسط API `DynamicResourceAllocation` تخصیص یافته‌اند، فراهم کند. برای فعال کردن این ویژگی، `kubelet` باید با پرچم‌های زیر شروع شود:

```
--feature-gates=DynamicResourceAllocation=true,KubeletPodResourcesDynamicResources=true
```

```gRPC
// ListPodResourcesResponse پاسخی است که توسط تابع List بازگردانده می‌شود.
message ListPodResourcesResponse {
    repeated PodResources pod_resources = 1;
}

// PodResources حاوی اطلاعاتی در مورد منابع گره‌ای اختصاص یافته به یک pod است.
message PodResources {
    string name = 1;
    string namespace = 2;
    repeated ContainerResources containers = 3;
}

// ContainerResources حاوی اطلاعاتی در مورد منابع اختصاص یافته به یک کانتینر است.
message ContainerResources {
    string name = 1;
    repeated ContainerDevices devices = 2;
    repeated int64 cpu_ids = 3;
    repeated ContainerMemory memory = 4;
    repeated DynamicResource dynamic_resources = 5;
}

// ContainerMemory حاوی اطلاعاتی در مورد حافظه و صفحات بزرگ اختصاص یافته به یک کانتینر است.
message ContainerMemory {
    string memory_type = 1;
    uint64 size = 2;
    TopologyInfo topology = 3;
}

// Topology اطلاعات توپوگرافی سخت‌افزار منابع را توصیف می‌کند.
message TopologyInfo {
        repeated NUMANode nodes = 1;
}

// NUMA نمایشی از گره NUMA.
message NUMANode {
        int64 ID = 1;
}

// ContainerDevices حاوی اطلاعاتی در مورد دستگاه‌های اختصاص یافته به یک کانتینر است.
message ContainerDevices {
    string resource_name = 1;
    repeated string device_ids = 2;
    TopologyInfo topology = 3;
}

// DynamicResource حاوی اطلاعاتی در مورد دستگاه‌های اختصاص یافته به یک کانتینر توسط تخصیص منابع پویا.
message DynamicResource {
    string class_name = 1;
    string claim_name = 2;
    string claim_namespace = 3;
    repeated ClaimResource claim_resources = 4;
}

// ClaimResource حاوی اطلاعاتی در مورد منابع هر پلاگین است.
message ClaimResource {
    repeated CDIDevice cdi_devices = 1 [(gogoproto.customname) = "CDIDevices"];
}

// CDIDevice اطلاعات دستگاه CDI را مشخص می‌کند.
message CDIDevice {
    // نام دستگاه CDI به طور کامل واجد شرایط
    // به عنوان مثال: vendor.com/gpu=gpudevice1
    // جزئیات بیشتر را در مشخصات CDI ببینید:
    // https://github.com/container-orchestrated-devices/container-device-interface/blob/main/SPEC.md
    string name = 1;
}
```
{{< note >}}
cpu_ids در `ContainerResources` در نقطه پایانی `List` مربوط به CPUهای اختصاصی تخصیص یافته به یک کانتینر خاص است. اگر هدف ارزیابی CPUهایی است که به استخر مشترک تعلق دارند، نقطه پایانی `List` نیاز به استفاده همراه با نقطه پایانی `GetAllocatableResources` دارد همانطور که در زیر توضیح داده شده است:
1. `GetAllocatableResources` را برای دریافت لیستی از تمام CPUهای قابل تخصیص فراخوانی کنید.
2. `GetCpuIds` را برای تمام `ContainerResources` در سیستم فراخوانی کنید.
3. همه CPUها را از تماس‌های `GetCpuIds` از تماس `GetAllocatableResources` کسر کنید.
{{< /note >}}

### نقطه پایانی gRPC `GetAllocatableResources` {#grpc-endpoint-getallocatableresources}

{{< feature-state state="stable" for_k8s_version="v1.28" >}}

`GetAllocatableResources` اطلاعاتی در مورد منابع اولیه موجود در گره کاری ارائه می‌دهد.
این اطلاعات بیشتری نسبت به آنچه kubelet به APIServer صادر می‌کند ارائه می‌دهد.

{{< note >}}
`GetAllocatableResources` باید تنها برای ارزیابی منابع [قابل تخصیص](/docs/tasks/administer-cluster/reserve-compute-resources/#node-allocatable) در یک گره استفاده شود. اگر هدف ارزیابی منابع آزاد/غیر تخصیص یافته است، باید همراه با نقطه پایانی `List()` استفاده شود. نتیجه‌ای که توسط `GetAllocatableResources` به دست می‌آید تا زمانی که منابع زیرین ارائه شده به kubelet تغییر نکند، باقی می‌ماند. این اتفاق به ندرت رخ می‌دهد اما هنگامی که رخ می‌دهد (به عنوان مثال: hotplug/hotunplug، تغییرات در سلامت دستگاه)، مشتری انتظار دارد که نقطه پایانی `GetAllocatableResources` را فراخوانی کند.

با این حال، فراخوانی نقطه پایانی `GetAllocatableResources` در مورد به‌روزرسانی CPU و/یا حافظه کافی نیست و kubelet نیاز به راه‌اندازی مجدد دارد تا ظرفیت و منابع قابل تخصیص صحیح را منعکس کند.
{{< /note >}}

```gRPC
// AllocatableResourcesResponses حاوی اطلاعاتی در مورد تمام دستگاه‌هایی است که توسط kubelet شناخته شده است.
message AllocatableResourcesResponse {
    repeated ContainerDevices devices = 1;
    repeated int64 cpu_ids = 2;
    repeated ContainerMemory memory = 3;
}
```

`ContainerDevices` اطلاعات توپوگرافی را افشا می‌کنند که نشان می‌دهد دستگاه به کدام سلول‌های NUMA وابسته است. سلول‌های NUMA با استفاده از یک شناسه عددی مبهم شناسایی می‌شوند که مقدار آن با آنچه پلاگین‌های دستگاه گزارش می‌دهند [هنگام ثبت خود در kubelet](/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/#device-plugin-integration-with-the-topology-manager) سازگار است.

سرویس gRPC بر روی یک سوکت یونیکس در مسیر `/var/lib/kubelet/pod-resources/kubelet.sock` سرو می‌شود.
عامل‌های نظارت بر منابع پلاگین دستگاه می‌توانند به عنوان یک daemon یا به صورت DaemonSet مستقر شوند.
دایرکتوری استاندارد `/var/lib/kubelet/pod-resources` به دسترسی دارای امتیاز نیاز دارد، بنابراین عامل‌های نظارت باید در یک محیط امنیتی دارای امتیاز اجرا شوند. اگر یک عامل نظارت دستگاه به صورت DaemonSet اجرا می‌شود، `/var/lib/kubelet/pod-resources` باید به عنوان یک {{< glossary_tooltip term_id="volume" >}} در [PodSpec](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podspec-v1-core) عامل نظارت دستگاه نصب شود.

{{< note >}}

هنگام دسترسی به `/var/lib/kubelet/pod-resources/kubelet.sock` از DaemonSet یا هر برنامه دیگری که به صورت کانتینر روی میزبان مستقر شده و سوکت را به عنوان یک volume نصب کرده است، بهتر است به جای `/var/lib/kubelet/pod-resources/kubelet.sock`، دایرکتوری `/var/lib/kubelet/pod-resources/` را نصب کنید. این کار اطمینان حاصل می‌کند که پس از راه‌اندازی مجدد kubelet، کانتینر قادر به اتصال مجدد به این سوکت خواهد بود.

نصب‌های کانتینر توسط inode که به سوکت یا دایرکتوری ارجاع می‌دهد، مدیریت می‌شوند، بسته به آنچه نصب شده است. هنگامی که kubelet راه‌اندازی مجدد می‌شود، سوکت حذف می‌شود و یک سوکت جدید ایجاد می‌شود، در حالی که دایرکتوری دست نخورده باقی می‌ماند. بنابراین inode اصلی برای سوکت غیرقابل استفاده می‌شود. inode برای دایرکتوری همچنان کار خواهد کرد.

{{< /note >}}

### نقطه پایانی gRPC `Get` {#grpc-endpoint-get}

{{< feature-state state="alpha" for_k8s_version="v1.27" >}}

نقطه پایانی `Get` اطلاعاتی در مورد منابع یک Pod در حال اجرا ارائه می‌دهد. اطلاعات مشابهی را که در نقطه پایانی `List` توضیح داده شده است افشا می‌کند. نقطه پایانی `Get` نیاز به `PodName` و `PodNamespace` از Pod در حال اجرا دارد.

```gRPC
// GetPodResourcesRequest حاوی اطلاعاتی در مورد pod است.
message GetPodResourcesRequest {
    string pod_name = 1;
    string pod_namespace = 2;
}
```

برای فعال کردن این ویژگی، باید سرویس‌های kubelet خود را با پرچم زیر شروع کنید:

```
--feature-gates=KubeletPodResourcesGet=true
```

نقطه پایانی `Get` می‌تواند اطلاعات مربوط به Pod را که به منابع پویا اختصاص یافته توسط API تخصیص منابع پویا مربوط است، ارائه دهد. برای فعال کردن این ویژگی، باید اطمینان حاصل کنید که سرویس‌های kubelet شما با پرچم‌های زیر شروع می‌شوند:

```
--feature-gates=KubeletPodResourcesGet=true,DynamicResourceAllocation=true,KubeletPodResourcesDynamicResources=true
```

## یکپارچه‌سازی پلاگین دستگاه با Topology Manager

{{< feature-state for_k8s_version="v1.27" state="stable" >}}

Topology Manager یک مؤلفه kubelet است که اجازه می‌دهد منابع به صورت هماهنگ در یک ترتیب توپوگرافی هم‌راستا شوند. برای انجام این کار، API پلاگین دستگاه به گونه‌ای گسترش یافت که یک ساختار `TopologyInfo` را شامل شود.

```gRPC
message TopologyInfo {
    repeated NUMANode nodes = 1;
}

message NUMANode {
    int64 ID = 1;
}
```

پلاگین‌های دستگاهی که می‌خواهند از Topology Manager استفاده کنند می‌توانند به عنوان بخشی از ثبت دستگاه، همراه با شناسه‌های دستگاه و سلامت دستگاه، یک ساختار `TopologyInfo` پر شده ارسال کنند.
مدیر دستگاه سپس از این اطلاعات برای مشاوره با Topology Manager و اتخاذ تصمیمات تخصیص منابع استفاده خواهد کرد.

`TopologyInfo` پشتیبانی از تنظیم یک فیلد `nodes` به صورت `nil` یا لیستی از گره‌های NUMA را فراهم می‌کند. این به پلاگین دستگاه اجازه می‌دهد تا یک دستگاه را که در چندین گره NUMA گسترده شده است تبلیغ کند.

تنظیم `TopologyInfo` به صورت `nil` یا ارائه یک لیست خالی از گره‌های NUMA برای یک دستگاه خاص نشان می‌دهد که پلاگین دستگاه برای آن دستگاه ترجیحی برای وابستگی NUMA ندارد.

یک مثال از ساختار `TopologyInfo` که توسط یک پلاگین دستگاه برای یک دستگاه پر شده است:

```
pluginapi.Device{ID: "25102017", Health: pluginapi.Healthy, Topology:&pluginapi.TopologyInfo{Nodes: []*pluginapi.NUMANode{&pluginapi.NUMANode{ID: 0,},}}}
```

## مثال‌های پلاگین دستگاه {#examples}

{{% thirdparty-content %}}

در اینجا چند مثال از پیاده‌سازی‌های پلاگین دستگاه آورده شده است:

* [Akri](https://github.com/project-akri/akri)، که به شما اجازه می‌دهد به راحتی دستگاه‌های مختلف برگ مانند دوربین‌های IP و دستگاه‌های USB را افشا کنید.
* [پلاگین دستگاه AMD GPU](https://github.com/ROCm/k8s-device-plugin)
* [پلاگین دستگاه عمومی](https://github.com/squat/generic-device-plugin) برای دستگاه‌های عمومی لینوکس و دستگاه‌های USB
* [پلاگین‌های دستگاه Intel](https://github.com/intel/intel-device-plugins-for-kubernetes) برای دستگاه‌های Intel GPU، FPGA، QAT، VPU، SGX، DSA، DLB و IAA
* [پلاگین‌های دستگاه KubeVirt](https://github.com/kubevirt/kubernetes-device-plugins) برای مجازی‌سازی با کمک سخت‌افزار
* [پلاگین دستگاه NVIDIA GPU برای Container-Optimized OS](https://github.com/GoogleCloudPlatform/container-engine-accelerators/tree/master/cmd/nvidia_gpu)
* [پلاگین دستگاه RDMA](https://github.com/hustcat/k8s-rdma-device-plugin)
* [پلاگین دستگاه SocketCAN](https://github.com/collabora/k8s-socketcan)
* [پلاگین دستگاه Solarflare](https://github.com/vikaschoudhary16/sfc-device-plugin)
* [پلاگین دستگاه شبکه SR-IOV](https://github.com/intel/sriov-network-device-plugin)
* [پلاگین‌های دستگاه Xilinx FPGA](https://github.com/Xilinx/FPGA_as_a_Service/tree/master/k8s-device-plugin) برای دستگاه‌های Xilinx FPGA

## {{% heading "whatsnext" %}}

* درباره [زمان‌بندی منابع GPU](/docs/tasks/manage-gpus/s

cheduling-gpus/) با استفاده از پلاگین‌های دستگاه بیشتر بدانید
* درباره [تبلیغ منابع گسترده‌شده](/docs/tasks/administer-cluster/extended-resource-node/) در یک گره بیشتر بدانید
* درباره [مدیر Topology](/docs/tasks/administer-cluster/topology-manager/) بیشتر بدانید
* درباره استفاده از [شتاب‌دهنده سخت‌افزاری برای ورود TLS](/blog/2019/04/24/hardware-accelerated-ssl/tls-termination-in-ingress-controllers-using-kubernetes-device-plugins-and-runtimeclass/) با استفاده از پلاگین‌های دستگاه و RuntimeClass در Kubernetes بیشتر بدانید
