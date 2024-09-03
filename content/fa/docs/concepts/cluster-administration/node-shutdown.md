---
title: خاموشی نود‌ها
content_type: مفهوم
weight: 10
---

<!-- مرور -->
در یک خوشه Kubernetes، یک {{< glossary_tooltip text="node" term_id="node" >}}
ممکن است به صورت برنامه‌ریزی شده و خوشه‌ای یا ناگهانی و ناخواسته به دلیل دلایلی نظیر قطع برق یا چیز دیگر خاموش شود. خاموشی یک نود می‌تواند منجر به شکست بارکاری‌ها شود اگر نود قبل از خاموش شدن خالی نشود. خاموشی یک نود می‌تواند به دو صورت **خوشه‌ای** یا **غیر خوشه‌ای** باشد.

<!-- متن -->
## خاموشی خوشه‌ای نود {#graceful-node-shutdown}

{{< feature-state feature_gate_name="GracefulNodeShutdown" >}}

kubelet سعی می‌کند که خاموشی سیستمی نود را تشخیص دهد و پادهای در حال اجرا روی نود را خاتمه دهد.

kubelet اطمینان حاصل می‌کند که پادها در طول خاموشی نود فرآیند
[خاتمه پاد](/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination)
معمول را دنبال می‌کنند. در طول خاموشی نود، kubelet قادر به پذیرفتن پادهای جدید نیست (حتی اگر این پادها قبلا به نود متصل شده باشند).

خاموشی خوشه‌ای نود بستگی به systemd دارد زیرا از
[قفل‌های مهار کننده systemd](https://www.freedesktop.org/wiki/Software/systemd/inhibit/) برای تأخیر خاموشی نود با مدت زمان داده شده استفاده می‌کند.

ویژگی خاموشی خوشه‌ای نود با استفاده از `GracefulNodeShutdown`
[دروازه ویژگی](/docs/reference/command-line-tools-reference/feature-gates/)
که به طور پیش‌فرض در نسخه ۱.۲۱ فعال است.

لطفا توجه داشته باشید که به طور پیش‌فرض، هر دو گزینه پیکربندی زیر،
`shutdownGracePeriod` و `shutdownGracePeriodCriticalPods` به صفر تنظیم شده‌اند، بنابراین ویژگی خاموشی خوشه‌ای نود فعال نمی‌شود.
برای فعال‌سازی این ویژگی، باید دو تنظیم پیکربندی kubelet را به درستی پیکربندی و به مقادیر غیر صفر تنظیم کنید.

هنگامی که systemd خاموشی نود را تشخیص می‌دهد یا اعلام می‌کند، kubelet شرایط `NotReady` را بر روی نود تنظیم می‌کند، با `reason` تنظیم شده به `"node is shutting down"`. kube-scheduler این شرایط را احترام می‌گذارد و هیچ پاد جدیدی را بر روی نود متأثر زمانبندی نمی‌کند؛ زمانبندی‌گرهای شخص ثالث انتظار می‌رود که منطق مشابه را دنبال کنند. این بدان معناست که پادهای جدیدی بر روی آن نود زمانبندی نمی‌شوند و بنابراین هیچ کدام شروع نمی‌شوند.

kubelet **همچنین** در طول مرحله `PodAdmission` پذیرفتن پادها را رد می‌کند اگر خاموشی نودی در حال انجام تشخیص شده باشد، به طوری که حتی پادهایی با
{{< glossary_tooltip text="toleration" term_id="toleration" >}} برای
`node.kubernetes.io/not-ready:NoSchedule` نیز در آنجا شروع نمی‌شوند.

همزمان با تنظیم kubelet این شرایط را بر روی نود خود از طریق API، kubelet همچنین شروع به خاتمه دادن هر گونه پادی که به طور محلی در حال اجرا است، می‌کند.

در طول خاموشی خوشه‌ای، kubelet پادها را در دو مرحله خاتمه می‌دهد:

1. پایان دادن به پادهای عادی در حال اجرا روی نود.
2. پایان دادن به [پادهای بحرانی](/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/#marking-pod-as-critical)
   در حال اجرا روی نود.

ویژگی خاموشی خوشه‌ای نود با دو گزینه
[`KubeletConfiguration`](/docs/tasks/administer-cluster/kubelet-config-file/) پیکربندی می‌شود:

* `shutdownGracePeriod`:
  * مدت کلی که نود باید خاموشی را تأخیر دهد را مشخص می‌کند. این مدت تأخیر کل برای خاتمه پادهای عادی و
    [پادهای بحرانی](/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/#marking-pod-as-critical)
    است.
* `shutdownGracePeriodCriticalPods`:
  * مدت استفاده شده برای خاتمه
    [پادهای بحرانی](/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/#marking-pod-as-critical)
    در طول خاموشی نود. این مقدار باید کمتر از `shutdownGracePeriod` باشد.

{{< note >}}

مواردی وجود دارد که خاتمه نود توسط سیستم لغو شده است (یا شاید به صورت دستی توسط یک مدیر). در هر یک از این شرایط، نود به حالت `Ready` باز می‌گردد. با این حال، پادهایی که قبلاً فرآیند خاتمه را آغاز کرده‌اند، توسط kubelet بازنشانده نخواهند شد و نیاز به برنامه‌ریزی مجدد دارند.

{{< /note >}}

به عنوان مثال، اگر `shutdownGracePeriod=30s` و
`shutdownGracePeriodCriticalPods=10s` باشد، kubelet خاموشی ن

ود را با تأخیر 30 ثانیه انجام می‌دهد. در طول خاموشی، 20 ثانیه اول (30-10) برای خاتمه به طور خوشه‌ای پادهای عادی اختصاص داده شده و آخرین 10 ثانیه برای خاتمه
[پادهای بحرانی](/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/#marking-pod-as-critical)
ذخیره می‌شود.

{{< note >}}
زمانی که پادها در طول خاموشی خوشه‌ای نود اخراج می‌شوند، به عنوان خاتمه علامت زده شده‌اند.
اجرای `kubectl get pods` وضعیت پادهای اخراج شده را به عنوان `Terminated` نشان می‌دهد.
و `kubectl describe pod` نشان می‌دهد که پاد به دلیل خاموشی نود برگرفته شده است:

```
Reason:         Terminated
Message:        Pod was terminated in response to imminent node shutdown.
```

{{< /note >}}

### خاموشی خوشه‌ای نود براساس اولویت پاد {#pod-priority-graceful-node-shutdown}

{{< feature-state feature_gate_name="GracefulNodeShutdownBasedOnPodPriority" >}}

برای ارائه انعطاف بیشتر در طول خاموشی خوشه‌ای نود در خصوص ترتیب دادن پادها، خاموشی خوشه‌ای نود احترام می‌گذارد به کلاس اولویت برای پادها، تهیه شده تا در صورت فعال بودن این ویژگی در خوشه، به طور صریح ترتیب پادها را تعیین کند. این ویژگی به مدیران خوشه اجازه می‌دهد تا ترتیب پادها را در طول خاموشی خوشه‌ای نود بر اساس
[کلاس‌های اولویت](/docs/concepts/scheduling-eviction/pod-priority-preemption/#priorityclass)
تعریف کنند.

ویژگی [خاموشی خوشه‌ای نود](#graceful-node-shutdown)، به عنوان توضیح داده شده، پادها را در دو مرحله، پادهای غیر بحرانی و سپس پادهای بحرانی، خاتمه می‌دهد. اگر نیاز به انعطاف بیشتر برای صریح تعریف ترتیب پادها در طول خاموشی وجود داشته باشد، می‌توان از خاموشی خوشه‌ای نود بر اساس اولویت پاد استفاده کرد.

وقتی خاموشی خوشه‌ای نود احترام به اولویت‌های پادها را دارد، این امکان وجود دارد که خاموشی خوشه‌ای نود را در چندین مرحله انجام دهد، هر مرحله پایان دادن به یک کلاس اولویت خاص از پادها را دارد. kubelet می‌تواند بافت‌ها و زمان خاموشی را برای هر فاز دقیق تنظیم کند.

با فرض این که کلاس‌های اولویت پاد سفارشی زیر را در یک خوشه داشته باشید،

|نام کلاس اولویت پاد|مقدار کلاس اولویت پاد|
|-------------------------|------------------------|
|`custom-class-a`         | 100000                 |
|`custom-class-b`         | 10000                  |
|`custom-class-c`         | 1000                   |
|`معمولی/تنظیم نشده`   | 0                      |

در پیکربندی [kubelet](/docs/reference/config-api/kubelet-config.v1beta1/)
تنظیمات برای `shutdownGracePeriodByPodPriority` به صورت زیر خواهد بود:

|مقدار کلاس اولویت پاد|مدت زمان خاتمه|
|------------------------|---------------|
| 100000                 |10 ثانیه       |
| 10000                  |180 ثانیه      |
| 1000                   |120 ثانیه      |
| 0                      |60 ثانیه       |

پیکربندی YAML مربوطه برای kubelet به شکل زیر خواهد بود:

```yaml
shutdownGracePeriodByPodPriority:
  - priority: 100000
    shutdownGracePeriodSeconds: 10
  - priority: 10000
    shutdownGracePeriodSeconds: 180
  - priority: 1000
    shutdownGracePeriodSeconds: 120
  - priority: 0
    shutdownGracePeriodSeconds: 60
```
به صورت Markdown در فارسی:

## زمان‌بندی متوقف کردن Pods بر اساس اولویت

در جدول بالا آمده است که هر Pod با مقدار `priority` بزرگتر مساوی 100000 فقط 10 ثانیه زمان دارد تا متوقف شود، هر Pod با مقدار بزرگتر مساوی 10000 و کمتر از 100000 180 ثانیه زمان دارد، هر Pod با مقدار بزرگتر مساوی 1000 و کمتر از 10000 120 ثانیه زمان دارد. در نهایت، سایر Pods 60 ثانیه زمان دارند تا متوقف شوند.

نیازی نیست که برای همه کلاس‌ها مقادیر متناظر را مشخص کنید. به عنوان مثال، می‌توانید از این تنظیمات استفاده کنید:

| مقدار کلاس اولویت Pod | زمان متوقف شدن |
|------------------------|---------------|
| 100000                 |300 ثانیه      |
| 1000                   |120 ثانیه      |
| 0                      |60 ثانیه       |

در مثال بالا، Pods با `custom-class-b` به همان سطح `custom-class-c` برای متوقف شدن تعلق خواهند داشت.

اگر در یک دامنه خاص Podsی وجود نداشته باشد، kubelet منتظر Pods در آن دامنه اولویت نمی‌ماند. به جای آن، kubelet به سرعت به دامنه مقدار اولویت بعدی می‌پردازد.

برای استفاده از این ویژگی، لازم است ویژگی `GracefulNodeShutdownBasedOnPodPriority` را فعال کرده و `ShutdownGracePeriodByPodPriority` را در پیکربندی kubelet به تنظیمات مورد نظر که شامل مقادیر کلاس اولویت Pod و دوره‌های متوقف شدن آن‌ها است، تنظیم کنید.

{{< note >}}
قابلیت در نظر گرفتن اولویت Pod در زمان متوقف شدن مهربانانه یک ویژگی Alpha در Kubernetes v1.23 معرفی شد. در Kubernetes {{< skew currentVersion >}} این ویژگی به صورت Beta است و به طور پیش‌فرض فعال است.
{{< /note >}}

معیارهای `graceful_shutdown_start_time_seconds` و `graceful_shutdown_end_time_seconds` زیرسیستم kubelet را برای نظارت بر خاموشی Node‌ها ارسال می‌کنند.

## رفتار مدیریت خاموش کردن غیر مهربانانه Node {#non-graceful-node-shutdown}

{{< feature-state feature_gate_name="NodeOutOfServiceVolumeDetach" >}}

عمل خاموش کردن یک Node ممکن است توسط مدیر Node Shutdown Manager kubelet شناسایی نشود، یا به دلیل خطاهای کاربری مانند تنظیم نادرست ShutdownGracePeriod و ShutdownGracePeriodCriticalPods. لطفاً برای جزئیات بیشتر به بخش بالا [Graceful Node Shutdown](#graceful-node-shutdown) مراجعه کنید.

هنگامی که یک Node خاموش می‌شود اما توسط Node Shutdown Manager kubelet شناسایی نمی‌شود، Pods که قسمتی از {{< glossary_tooltip text="StatefulSet" term_id="statefulset" >}} هستند در وضعیت Terminating در Node خاموش گیر می‌افتند و نمی‌توانند به یک Node دیگر حرکت کنند که در حال اجرا باشد. این به دلیل عدم دسترسی kubelet در Node خاموش برای حذف Pods است، بنابراین StatefulSet نمی‌تواند یک Pods جدید با همان نام ایجاد کند. اگر از Volumes توسط Pods استفاده شود، VolumeAttachments از Node خاموش اصلی حذف نمی‌شود بنابراین Volumes مورد استفاده توسط این Pods نمی‌توانند به یک Node جدید اتصال پیدا کنند. به عبارت دیگر، برنامه اجرایی در حال اجرای StatefulSet نمی‌تواند به درستی کار کند. اگر Node خاموش اصلی بالا بیاید، Pods توسط kubelet حذف شده و Pods جدید در یک Node دیگر ایجاد خواهند شد. اگر Node خاموش اصلی بالا نیاید، این Pods بر روی Node خاموش به طور دائم در وضعیت Terminating گیر خواهند افتاد.

برای مقابله با این موقعیت، کاربر می‌تواند به طور دستی Taint `node.kubernetes.io/out-of-service` را با تأثیر `NoExecute` یا `NoSchedule` به یک Node اضافه کند و آن را به عنوان خارج از خدمت علامت‌گذاری کند. اگر ویژگی `NodeOutOfServiceVolumeDetach` از طریق {{< glossary_tooltip text="kube-controller-manager" term_id="kube-controller-manager" >}} فعال باشد و یک Node با این Taint علامت‌گذاری شود، Pods در Node به طور خودکار حذف خواهند شد اگر تحمل مطابقتی روی آن‌ها نباشد و عملیات جدا کردن Volume برای Pods در حال Terminating در Node بلافاصله انجام خواهد شد. این امکان می‌دهد تا Pods در Node خارج از خدمت به سرعت در یک Node دیگر بازیابی شوند.

در زمان یک خاموشی غیر مهربانانه، Pods در دو مرحله خاتمه می‌یابند:

1. حذف اجباری Pods که تحمل مطابقتی `out-of-service` ندارند.
2. عملیات جدا کردن Volume برای Pods در حال Terminating به صورت فوری انجام می‌شود.

{{< note >}}
- قبل از اضافه کردن Taint `node.kubernetes.io/out-of-service`، باید اطمینان حاصل شود که Node در حال خاموش شدن یا خاموش شدن است (نه در حال بازنشسته ش

دن).
- کاربر باید پس از انتقال Pods به یک Node جدید و اطمینان از بازیابی Node خاموش اصلی، Taint خارج از خدمت را به صورت دستی حذف کند.
{{< /note >}}

### جداسازی اجباری ذخیره‌سازی در صورت زمان مقید {#storage-force-detach-on-timeout}

در هر شرایطی که حذف Pods در مدت زمان 6 دقیقه انجام نشده باشد، Kubernetes تلاش می‌کند تا Volumes که در آن نود ناسالم است که در آن لحظه از نظر سلامتی خود در معرض قرار دارد، از طریق Volume Unmount کردن اجباری به دور برساند. هر Workload هنوز در حال اجرایی بودن روی Node است که از Volume اجباری استفاده می‌کند، باعث نقض مشخصات [CSI](https://github.com/container-storage-interface/spec/blob/master/spec.md#controllerunpublishvolume) خواهد شد که می‌گوید `ControllerUnpublishVolume` "باید" پس از تماس با "NodeUnstageVolume" و "NodeUnpublishVolume" بر روی Volume صدا زده شود و پس از موفقیت دارد. در چنین شرایطی، Volumes در Node مورد نظر ممکن است با خطر از دست رفتن داده‌ها روبرو شوند.

رفتار جداسازی اجباری ذخیره‌سازی اختیاری است؛ کاربران ممکن است تصمیم بگیرند به جای این ویژگی، از ویژگی "Shutdown غیر مهربانانه Node" استفاده کنند.

می‌توان تنظیم کرد تا رفتار جداسازی اجباری ذخیره‌سازی غیرفعال باشد با تنظیم فیلد پیکربندی `disable-force-detach-on-timeout` در `kube-controller-manager`. غیرفعال کردن ویژگی جداسازی اجباری ذخیره‌سازی به این معنی است که یک Volume که بر روی یک Node ناسالم برای بیش از 6 دقیقه وجود دارد، ارتباط VolumeAttachment مربوطه آن حذف نخواهد شد.

بعد از اعمال این تنظیم، Pods ناسالم که هنوز به Volumes متصل هستند باید از طریق روش [Shutdown غیر مهربانانه Node](#non-graceful-node-shutdown) که در بالا ذکر شده، بازیابی شوند.

{{< note >}}
- در استفاده از روش [Shutdown غیر مهربانانه Node](#non-graceful-node-shutdown) باید موارد فراهم شده مراعات شود.
- انحراف از مراحل مستند شده ممکن است منجر به فساد داده شود.
{{< /note >}}

## {{% heading "whatsnext" %}}

بیشتر درباره موارد زیر بیاموزید:
* وبلاگ: [Shutdown غیر مهربانانه Node](/blog/2023/08/16/kubernetes-1-28-non-graceful-node-shutdown-ga/).
* معماری کلاستر: [Nodes](/docs/concepts/architecture/nodes/).
