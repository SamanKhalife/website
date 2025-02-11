```markdown
---
reviewers:
- fgrzadkowski
- jszczepkowski
- directxman12
title: مقیاس‌گذاری افقی پادها (Horizontal Pod Autoscaling)
feature:
  title: مقیاس‌گذاری افقی
  description: >
    برنامه خود را به راحتی با یک فرمان ساده، با یک UI یا به صورت خودکار بر اساس استفاده از CPU مقیاس‌گذاری کنید.
content_type: concept
weight: 90
---

<!-- overview -->

در کوبرنتیز، _HorizontalPodAutoscaler_ به طور خودکار یک منبع کاری (مانند
{{< glossary_tooltip text="Deployment" term_id="deployment" >}} یا
{{< glossary_tooltip text="StatefulSet" term_id="statefulset" >}}) را به‌روزرسانی می‌کند، با هدف مقیاس‌گذاری خودکار حجم کار برای مطابقت با تقاضا.

مقیاس‌گذاری افقی به این معنی است که در پاسخ به افزایش بار، تعداد بیشتری از
{{< glossary_tooltip text="Pods" term_id="pod" >}} مستقر می‌شوند.
این متفاوت از مقیاس‌گذاری _عمودی_ است که برای کوبرنتیز به معنی اختصاص منابع بیشتر (برای مثال: حافظه یا CPU) به پادهای در حال اجرا برای حجم کار است.

اگر بار کاهش یابد و تعداد پادها بالاتر از حداقل تنظیم شده باشد، HorizontalPodAutoscaler منبع کاری (Deployment، StatefulSet، یا منبع مشابه دیگر) را دستور به کاهش مقیاس می‌دهد.

مقیاس‌گذاری افقی پادها برای اشیایی که نمی‌توانند مقیاس شوند (برای مثال:
{{< glossary_tooltip text="DaemonSet" term_id="daemonset" >}}) اعمال نمی‌شود.

HorizontalPodAutoscaler به عنوان یک منبع API کوبرنتیز و یک
{{< glossary_tooltip text="controller" term_id="controller" >}} پیاده‌سازی شده است.
این منبع رفتار کنترلر را تعیین می‌کند.
کنترلر مقیاس‌گذاری افقی پادها، که در داخل
{{< glossary_tooltip text="control plane" term_id="control-plane" >}} کوبرنتیز اجرا می‌شود، به صورت دوره‌ای مقیاس مورد نظر هدف خود (برای مثال، یک Deployment) را بر اساس مشاهدات متریک‌ها مانند استفاده متوسط CPU، استفاده متوسط حافظه، یا هر متریک سفارشی دیگری که تعیین می‌کنید، تنظیم می‌کند.

یک [مثال گام به گام](/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) از استفاده از مقیاس‌گذاری افقی پادها وجود دارد.

<!-- body -->

## چگونه یک HorizontalPodAutoscaler کار می‌کند؟

{{< mermaid >}}
graph BT

hpa[Horizontal Pod Autoscaler] --> scale[Scale]

subgraph rc[RC / Deployment]
    scale
end

scale -.-> pod1[Pod 1]
scale -.-> pod2[Pod 2]
scale -.-> pod3[Pod N]

classDef hpa fill:#D5A6BD,stroke:#1E1E1D,stroke-width:1px,color:#1E1E1D;
classDef rc fill:#F9CB9C,stroke:#1E1E1D,stroke-width:1px,color:#1E1E1D;
classDef scale fill:#B6D7A8,stroke:#1E1E1D,stroke-width:1px,color:#1E1E1D;
classDef pod fill:#9FC5E8,stroke:#1E1E1D,stroke-width:1px,color:#1E1E1D;
class hpa hpa;
class rc rc;
class scale scale;
class pod1,pod2,pod3 pod
{{< /mermaid >}}

شکل ۱. HorizontalPodAutoscaler مقیاس یک Deployment و ReplicaSet آن را کنترل می‌کند

کوبرنتیز مقیاس‌گذاری افقی پادها را به عنوان یک حلقه کنترلی که به صورت متناوب اجرا می‌شود پیاده‌سازی می‌کند
(این یک فرآیند پیوسته نیست). بازه زمانی توسط پارامتر
`--horizontal-pod-autoscaler-sync-period` به
[`kube-controller-manager`](/docs/reference/command-line-tools-reference/kube-controller-manager/)
تنظیم می‌شود (و بازه زمانی پیش‌فرض ۱۵ ثانیه است).

یک بار در هر دوره، مدیر کنترلر میزان استفاده از منابع را با متریک‌های مشخص شده در هر تعریف HorizontalPodAutoscaler مقایسه می‌کند. مدیر کنترلر
منبع هدف تعیین شده توسط `scaleTargetRef` را پیدا می‌کند،
سپس پادها را بر اساس برچسب‌های `.spec.selector` منبع هدف انتخاب می‌کند،
و متریک‌ها را از API متریک‌های منبع (برای متریک‌های منبع هر پاد)،
یا API متریک‌های سفارشی (برای سایر متریک‌ها) بدست می‌آورد.

- برای متریک‌های منبع هر پاد (مانند CPU)، کنترلر متریک‌ها را
  از API متریک‌های منبع برای هر پاد هدف توسط HorizontalPodAutoscaler بدست می‌آورد.
  سپس، اگر مقدار هدف استفاده تنظیم شده باشد، کنترلر مقدار استفاده
  را به عنوان درصدی از
  [درخواست منبع](/docs/concepts/configuration/manage-resources-containers/#requests-and-limits)
  معادل روی کانتینرهای هر پاد محاسبه می‌کند. اگر مقدار خام هدف تنظیم شده باشد، مقادیر متریک خام به صورت مستقیم استفاده می‌شوند.
  سپس کنترلر میانگین استفاده یا مقدار خام (بسته به نوع هدف تعیین شده) را در میان تمامی پادهای هدف می‌گیرد و نسبت مورد استفاده برای مقیاس‌گذاری تعداد پادهای مورد نظر را تولید می‌کند.

  لطفاً توجه داشته باشید که اگر برخی از کانتینرهای پاد درخواست منبع مرتبط را تنظیم نکرده باشند،
  استفاده از CPU برای پاد تعریف نمی‌شود و اتوسکیلر هیچ اقدامی برای آن متریک انجام نمی‌دهد. برای اطلاعات بیشتر درباره نحوه عملکرد الگوریتم اتوسکیلینگ به بخش [جزئیات الگوریتم](#algorithm-details) مراجعه کنید.

- برای متریک‌های سفارشی هر پاد، کنترلر به طور مشابه با متریک‌های منبع هر پاد عمل می‌کند،
  با این تفاوت که با مقادیر خام کار می‌کند، نه مقادیر استفاده.

- برای متریک‌های اشیاء و متریک‌های خارجی، یک متریک واحد بدست می‌آید که شیء مورد نظر را توصیف می‌کند. این متریک با مقدار هدف مقایسه می‌شود تا نسبت مشابهی تولید شود. در API نسخه `autoscaling/v2`،
  این مقدار می‌تواند به صورت اختیاری بر تعداد پادها تقسیم شود قبل از اینکه مقایسه انجام شود.

استفاده معمول از HorizontalPodAutoscaler این است که آن را برای دریافت متریک‌ها از
{{< glossary_tooltip text="aggregated APIs" term_id="aggregation-layer" >}}
(`metrics.k8s.io`، `custom.metrics.k8s.io`، یا `external.metrics.k8s.io`) پیکربندی کنید. API `metrics.k8s.io` معمولاً توسط یک افزودنی به نام Metrics Server فراهم می‌شود که نیاز به راه‌اندازی جداگانه دارد.
برای اطلاعات بیشتر درباره متریک‌های منابع، به
[Metrics Server](/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/#metrics-server) مراجعه کنید.

[پشتیبانی از APIهای متریک](#support-for-metrics-apis) توضیحات مربوط به تضمین‌های پایداری و وضعیت پشتیبانی این
APIهای مختلف را ارائه می‌دهد.

کنترلر HorizontalPodAutoscaler به منابع کاری مربوطه که از مقیاس‌گذاری پشتیبانی می‌کنند دسترسی دارد (مانند Deployments
و StatefulSet). این منابع هر کدام دارای یک زیرمنبع به نام `scale` هستند، یک رابط که به شما اجازه می‌دهد به صورت پویا تعداد پادها را تنظیم کنید و وضعیت فعلی هر یک از آنها را بررسی کنید.
برای اطلاعات عمومی درباره زیرمنابع در API کوبرنتیز، به
[مفاهیم API کوبرنتیز](/docs/reference/using-api/api-concepts/) مراجعه کنید.
### جزئیات الگوریتم

از دیدگاه بسیار ابتدایی، کنترل‌کننده HorizontalPodAutoscaler بر اساس نسبت بین مقدار متریک مورد نظر و مقدار فعلی متریک عمل می‌کند:

```
desiredReplicas = ceil[currentReplicas * ( currentMetricValue / desiredMetricValue )]
```

به عنوان مثال، اگر مقدار فعلی متریک `200m` و مقدار مطلوب `100m` باشد، تعداد رپلیکای‌ها دو برابر می‌شود، زیرا `200.0 / 100.0 == 2.0` است. اگر مقدار فعلی به جای آن `50m` باشد، تعداد رپلیکای‌ها نصف می‌شود، چرا که `50.0 / 100.0 == 0.5` است. سیستم کنترل هرگونه عملیات مقیاس‌پذیری را انجام نمی‌دهد اگر نسبت به طور کافی نزدیک به 1.0 باشد (در تنظیمات جهانی، به طور پیش‌فرض 0.1).

هنگامی که `targetAverageValue` یا `targetAverageUtilization` مشخص می‌شود، `currentMetricValue` با محاسبه میانگین متریک داده شده در تمام پادها در هدف مقیاس HorizontalPodAutoscaler محاسبه می‌شود.

قبل از بررسی تحمل و تصمیم‌گیری در مورد مقادیر نهایی، سیستم کنترل نیز در نظر می‌گیرد که آیا هر متریک مفقود است و چند پاد `Ready` هستند. همه پادهایی که زمان حذف آن‌ها تنظیم شده است (اشیاء با یک زمان حذف در حال حاضر در حال خاموش شدن / حذف هستند) نادیده گرفته می‌شوند و همه پادهای ناکامی دور می‌شوند.

اگر یک پاد خاص متریک‌های خود را از دست داده باشد، برای بعداً نگه داشته می‌شود؛ پادهایی با متریک‌های گم‌شده برای تنظیم مقدار مقیاس نهایی استفاده می‌شوند.

با توجه به محدودیت‌های فنی، کنترل‌کننده HorizontalPodAutoscaler نمی‌تواند دقیقاً تعیین کند که زمانی که یک پاد آماده می‌شود برای اولین بار، به عنوان مثال وقتی که تشخیص داده می‌شود که یک پاد "هنوز آماده نشده" است اگر هنوز آماده نشده است و در یک بازه زمانی کوتاه و پیکربندی شده از زمان اولین بار که شروع شده است تغییر کرده است. این مقدار با استفاده از پرچم `--horizontal-pod-autoscaler-initial-readiness-delay` پیکربندی شده و پیش‌فرض آن 30 ثانیه است.

پس از آنکه یک پاد آماده شده است، هر گونه گذر به آماده را به عنوان اولین می‌شمارد اگر آن اتفاق در یک بازه زمانی بیشتر و پیکربندی شده از زمانی که شروع شده است تغییر کرده است. این مقدار با استفاده از پرچم `--horizontal-pod-autoscaler-cpu-initialization-period` پیکربندی شده و پیش‌فرض آن 5 دقیقه است.

نسبت مقیاس مبنای `currentMetricValue / desiredMetricValue` سپس با استفاده از پادهای باقی‌مانده که از بالا گذشته یا پس زده شده محاسبه می‌شود.

اگر متریک‌هایی گم‌شده وجود داشته باشد، سیستم کنترل میانگین را دوباره با احتیاط بیشتر محاسبه می‌کند، اگر پادهای گم‌شده به تنظیمات خود را مصرف کنند که 100٪ از مقدار مطلوب در صورت مقیاس کم شدن و 0٪ در صورت مقیاس بالا. این باعث کاهش شدت هر گونه مقیاس ممکن می‌شود.

علاوه بر این، اگر پادهای هنوز آماده نشده وجود داشته باشند و بار کاری بدون در نظر گرفتن متریک‌های گم‌شده یا پادهای هنوز آماده نشده می‌شود، کنترل‌کننده فرضی کننده است که پادهای هنوز آماده نشده 0٪ از متریک مطلوب را مصرف می‌کنند، که باعث کاهش شدت مقیاس بالا می‌شود.

پس از در نظر گرفتن پادهای هنوز آماده نشده و متریک‌های گم‌شده، کنترل‌کننده نسبت استفاده جدید را محاسبه می‌کند. اگر نسبت جدید جهت مقیاس را معکوس کند یا در محدوده تحمل باشد، کنترل‌کننده هیچ عملیات مقیاس‌پذیری انجام نمی‌دهد. در سایر موارد، نسبت جدید برای تصمیم‌گیری در مورد هر تغییر در تعداد پادها استفاده می‌شود.

توجه داشته باشید که مقدار اصلی برای میانگین استفاده در وضعیت HorizontalPodAutoscaler گزارش داده می‌شود، بدون در نظر گرفتن پادهای هنوز آماده نشده یا متریک‌های گم‌شده، حتی زمانی که نسبت استفاده جدید بر

ای تصمیم‌گیری استفاده می‌شود.

اگر چندین متریک در یک HorizontalPodAutoscaler مشخص شده باشد، این محاسبه برای هر متریک انجام می‌شود و سپس بزرگترین تعداد رپلیکای مطلوب انتخاب می‌شود. اگر هر یک از این متریک‌ها نتواند به یک تعداد رپلیکای مطلوب تبدیل شود (به عنوان مثال به دلیل خطا در بازیابی متریک‌ها از APIs متریک) و یک مقیاس پایین توسط متریک‌هایی که می‌توانند بازیابی شوند پیشنهاد شود، مقیاس کاهش می‌یابد. این بدان معنی است که HorizontalPodAutoscaler هنوز قادر به مقیاس بالا است اگر یک یا چند متریک `desiredReplicas` بزرگتر از مقدار فعلی را ارائه دهد.

در نهایت، درست قبل از اینکه HPA هدف را مقیاس پذیر کند، توصیه مقیاس‌پذیری ثبت می‌شود. کنترل‌کننده تمام توصیه‌ها را در یک پنجره پیکربندی شده در نظر می‌گیرد و بالاترین توصیه را از درون آن پنجره انتخاب می‌کند. این مقدار با استفاده از پرچم `--horizontal-pod-autoscaler-downscale-stabilization` پیکربندی شده و پیش‌فرض آن 5 دقیقه است. این به این معنی است که مقیاس‌پذیری به آرامی انجام می‌شود، تاثیر نوسانات سریع مقادیر متریک را ملایم می‌کند.

## شیء API

Horizontal Pod Autoscaler یک منبع API در گروه Kubernetes `autoscaling` است. نسخه پایدار فعلی می‌تواند در نسخه `autoscaling/v2` API یافت شود که شامل پشتیبانی از مقیاس‌پذیری بر روی حافظه و متریک‌های سفارشی می‌شود. فیلدهای جدید معرفی شده در `autoscaling/v2` به عنوان حفظ شده‌ها به عنوان حفظ وقتی که با `autoscaling/v1` کار می‌کنید.

هنگامی که یک شیء API HorizontalPodAutoscaler ایجاد می‌کنید، مطمئن شوید که نام مشخص شده یک [نام زیردامنه DNS معتبر](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names) است. جزئیات بیشتر در مورد شیء API می‌توان در [HorizontalPodAutoscaler Object](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#horizontalpodautoscaler-v2-autoscaling) یافت.

## پایداری مقیاس بارگذاری {#flapping}

در مدیریت مقیاس یک گروه از رپلیکاها با استفاده از HorizontalPodAutoscaler، امکان دارد که تعداد رپلیکای موجود به طور مکرر به دلیل طبیعت پویا متریک‌های ارزیابی شده، تغییر کند. گاهی اوقات به این موضوع اشاره می‌شود به عنوان _thrashing_ یا _flapping_، که مشابه با مفهوم _hysteresis_ در سایبرنتیک است.

## مقیاس‌پذیری خودکار در طول به‌روزرسانی غلتان

Kubernetes به شما اجازه می‌دهد که به‌روزرسانی غلتانی را بر روی یک Deployment انجام دهید. در آن مورد، Deployment ReplicaSets زیرین را برای شما مدیریت می‌کند. هنگامی که مقیاس‌پذیری را برای یک Deployment پیکربندی می‌کنید، یک HorizontalPodAutoscaler را به یک Deployment تکیه می‌دهید. کنترل‌کننده اجرای Deployment مسئول تنظیم `replicas` ReplicaSets زیرین است تا آنها در طول پخش و همچنین بعد از آن به یک تعداد مناسب اضافه شوند.

اگر به‌روزرسانی غلتانی از StatefulSet که تعداد رپلیکای آن به صورت خودکار تنظیم شده باشد، StatefulSet به طور مستقیم مجموعه‌های خود را مدیریت می‌کند (هیچ منبع وسیله مشابه ReplicaSet میانی وجود ندارد).

## پشتیبانی از متریک‌های منابع

هر هدف HPA می‌تواند بر اساس استفاده از منابع پادها در هدف مقیاس داده شود. هنگام تعیین مشخصات پاد، درخواست‌های منابع مانند `cpu` و `memory` باید مشخص شوند. این برای تعیین استفاده از مقیاس مبتنی بر مصرف منابع توسط کنترل‌کننده HPA استفاده می‌شود برای مقیاس به بالا یا پایین. برای استفاده از مقیاس مصرف مبتنی بر منابع، منبع متریک مانند این گونه مشخص شود:

```yaml
type: Resource
resource:
  name: cpu
  target:
    type: Utilization
    averageUtilization: 60
```

با این متریک، کنترل‌کننده HPA میانگین استفاده از منابع پادها در هدف مقیاس را در 60٪ نگه می‌دارد. استفاده نرخ بین استفاده فعلی از منابع به منابع درخواست شده پاد است. از [Algorithm](#algorithm-details) برای جزئیات بیشتر در مورد نحوه محاسبه و میانگین‌گیری استفاده شود.

{{< note >}}
از آنجا که مصارف منابع تمام ظروف جمع شده‌ان

د، مصرف کل پاد ممکن است نمایندگی دقیقی از استفاده منابع ظروف فردی نباشد. این ممکن است منجر به مواردی شود که یک ظرف تک نمونه با مصرف بالا در حال اجرا است و HPA برای مقیاس به بیرون نمی‌روید زیرا مصرف کل پاد هنوز هم در حدود مجاز است.
{{< /note >}}
### معیارهای منابع ظروف کانتینر

{{< feature-state feature_gate_name="HPAContainerMetrics" >}}

API HorizontalPodAutoscaler همچنین یک منبع متریک ظروف ارائه می‌دهد که HPA می‌تواند مصرف منابع هر کانتینر را در یک مجموعه از پادها ردیابی کند، به منظور مقیاس منبع هدف. این به شما این امکان را می‌دهد که آستانه‌های مقیاس‌پذیری را برای کانتینرهای مهم‌تر در یک پاد خاص پیکربندی کنید. به عنوان مثال، اگر یک برنامه وب و یک کانتینر کناری وجود دارد که وظیفه ثبت لاگ را انجام می‌دهد، می‌توانید بر اساس استفاده از منابع برنامه وب مقیاس دهی کنید و از استفاده کانتینر کناری و مصرف منابع آن صرف نظر کنید.

اگر منابع هدف را با مشخصات جدید پاد با مجموعه مختلفی از کانتینرها به روز کنید، باید مشخصات HPA را اصلاح کنید اگر کانتینر جدید اضافه شده همچنین برای مقیاس‌پذیری استفاده شود. اگر کانتینر مشخص شده در منبع متریک موجود نباشد یا فقط در یک زیرمجموعه از پادها وجود داشته باشد، آن پادها نادیده گرفته می‌شوند و توصیه مجدد محاسبه می‌شود. برای استفاده از منابع ظروف برای مقیاس دادن خودکار، یک منبع متریک به صورت زیر تعریف کنید:

```yaml
type: ContainerResource
containerResource:
  name: cpu
  container: application
  target:
    type: Utilization
    averageUtilization: 60
```

در مثال بالا، کنترل‌کننده HPA هدف را به گونه‌ای مقیاس دهی می‌کند که میانگین استفاده از cpu در کانتینر `application` تمام پادها به 60% است.

{{< note >}}
اگر نام یک کانتینر را که HorizontalPodAutoscaler در حال ردیابی آن است، تغییر دهید، باید این تغییر را به ترتیب خاصی اعمال کنید تا مطمئن شوید که قابلیت مقیاس‌پذیری هنوز هم در دسترس و موثر است در حالی که تغییر اعمال می‌شود. قبل از به‌روزرسانی منبع که کانتینر را تعریف می‌کند (مانند یک Deployment)، باید HPA مرتبط را بروزرسانی کنید تا هر دو نام کانتینر جدید و قدیمی را ردیابی کند. به این ترتیب، HPA قادر است در طول فرآیند به‌روزرسانی، پیشنهاد مقیاس‌پذیری را محاسبه کند.

پس از اعمال تغییر نام کانتینر به منبع کاربار، با حذف نام قدیمی کانتینر از مشخصات HPA، کار را پاک کنید.
{{< /note >}}

## مقیاس‌پذیری بر متریک‌های سفارشی

{{< feature-state for_k8s_version="v1.23" state="stable" >}}

(نسخه `autoscaling/v2beta2` به عنوان یک ویژگی بتا، قبلاً این قابلیت را ارائه می‌داد)

با استفاده از نسخه `autoscaling/v2` API، می‌توانید یک HorizontalPodAutoscaler را بر اساس یک متریک سفارشی پیکربندی کنید (که به Kubernetes یا هیچ کامپوننت Kubernetes دیگری تعلق ندارد). سپس کنترل‌کننده HorizontalPodAutoscaler برای این متریک‌های سفارشی از API Kubernetes استعلام می‌کند.

برای الزامات راه‌های متریک‌ها بررسی کنید [پشتیبانی از APIs متریک](#support-for-metrics-apis).

## مقیاس‌پذیری بر متریک‌های چندگانه

{{< feature-state for_k8s_version="v1.23" state="stable" >}}

(نسخه `autoscaling/v2beta2` به عنوان یک ویژگی بتا، قبلاً این قابلیت را ارائه می‌داد)

با استفاده از نسخه `autoscaling/v2` API، می‌توانید چندین متریک برای HorizontalPodAutoscaler مشخص کنید تا بر اساس آن‌ها مقیاس دهی کند. سپس کنترل‌کننده HorizontalPodAutoscaler هر متریک را ارزیابی کرده و مقیاس جدیدی بر اساس آن پیشنهاد می‌دهد. HorizontalPodAutoscaler حداکثر مقیاس پیشنهادی برای هر متریک را گرفته و بارکاری را به اندازه آن تنظیم می‌کند (پیش‌فرض این است که این بزرگتر از حداکثر کلی است که شما پیکربندی کرده‌اید).

## پشتیبانی از APIs متریک‌ها

به طور پیش‌فرض، کنترل‌کننده HorizontalPodAutoscaler از یک سری از APIs متریک‌ها معلوماتی دریافت می‌کند. برای دسترسی به این APIs، مدیران خوشه باید اطمینان حاصل کنند که:

- لایه تجمع API فعال است.

- APIs مربوطه ثبت شده‌اند:

  - برای متریک‌های منابع، این `API` `metrics.k8s.io` است [API](/docs/reference/external-api/metrics.v1beta1/). به طور کلی توسط [metrics-server](https://github.com/kubernetes-sigs/metrics-server) ا

رائه می‌شود. می‌تواند به عنوان یک افزونه خوشه راه‌اندازی شود.

  - برای متریک‌های سفارشی، این `API` `custom.metrics.k8s.io` است [API](/docs/reference/external-api/custom-metrics.v1beta2/).
    توسط "adapter" API servers ارائه شده توسط راه‌حل‌های متریکس. با پایپ‌لاین متریک خود در کنار کشف کنید که آیا یک آداپتور متریک Kubernetes در دسترس است.

  - برای متریک‌های خارجی، این `API` `external.metrics.k8s.io` است [API](/docs/reference/external-api/external-metrics.v1beta1/).
    ممکن است توسط آداپتورهای متریک خاص ارائه شود که در بالا ذکر شدند.

برای اطلاعات بیشتر در مورد این مسیرهای مختلف متریک و تفاوت‌های آن‌ها لطفاً پیشنهادات طراحی مربوط به [HPA V2](https://git.k8s.io/design-proposals-archive/autoscaling/hpa-v2.md)،
[custom.metrics.k8s.io](https://git.k8s.io/design-proposals-archive/instrumentation/custom-metrics-api.md)
و [external.metrics.k8s.io](https://git.k8s.io/design-proposals-archive/instrumentation/external-metrics-api.md) را مشاهده کنید.

برای مثال‌های استفاده از آن‌ها، به [راهنمای استفاده از متریک‌های سفارشی](/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-multiple-metrics-and-custom-metrics)
و [راهنمای استفاده از متریک‌های خارجی](/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-metrics-not-related-to-kubernetes-objects) مراجعه کنید.

## رفتار قابل تنظیم مقیاس‌پذیری

{{< feature-state for_k8s_version="v1.23" state="stable" >}}

(نسخه `autoscaling/v2beta2` به عنوان یک ویژگی بتا، قبلاً این قابلیت را ارائه می‌داد)

اگر از API HorizontalPodAutoscaler `v2` استفاده کنید، می‌توانید از فیلد `behavior`
(راهنمایی ببینید [API reference](/docs/reference/kubernetes-api/workload-resources/horizontal-pod-autoscaler-v2/#HorizontalPodAutoscalerSpec))
برای پیکربندی رفتارهای جداگانه برای مقیاس‌پذیری به‌بالا و به‌پایین استفاده کنید.
این رفتارها را با تنظیم `scaleUp` و/یا `scaleDown`
زیر `behavior` مشخص می‌کنید.

شما می‌توانید یک _پنجره ثابت‌سازی_ مشخص کنید که از [پرشینگ](#flapping)
تعداد کپی برای هدف مقیاس‌پذیر است. سیاست‌های مقیاس‌پذیری همچنین به شما این امکان را می‌دهد که نرخ تغییر کپی‌ها را در حالت مقیاس‌پذیری کنترل کنید.

### سیاست‌های مقیاس‌پذیری

یک یا چند سیاست مقیاس‌پذیری می‌تواند در بخش `behavior` از مشخصه تعیین شود. زمانی که چندین سیاست مشخص شده باشند، سیاستی که بیشترین تغییر را اجازه می‌دهد، سیاستی است که به طور پیش‌فرض انتخاب می‌شود. مثال زیر این رفتار را در حالت کاهش مقیاس نمایش می‌دهد:

```yaml
behavior:
  scaleDown:
    policies:
    - type: Pods
      value: 4
      periodSeconds: 60
    - type: Percent
      value: 10
      periodSeconds: 60
```

`periodSeconds` طول زمان در گذشته است که برای آن سیاست باید درست باشد. حداکثر مقداری که می‌توانید برای `periodSeconds` تنظیم کنید 1800 است (نیم ساعت). سیاست اول _(Pods)_ حداکثر 4 کپی را در یک دقیقه می‌تواند مقیاس کم کند. سیاست دوم _(Percent)_ حداکثر 10% از کپی‌های فعلی را در یک دقیقه می‌تواند کاهش دهد.

از آنجا که به طور پیش‌فرض سیاستی که بیشترین تغییر را اجازه می‌دهد انتخاب می‌شود، سیاست دوم تنها زمانی استفاده می‌شود که تعداد کپی‌های کنترل‌کننده بیشتر از 40 باشد. با 40 یا کمتر کپی، سیاست اول اعمال می‌شود. به عنوان مثال اگر 80 کپی وجود داشته باشد و هدف باید از 80 کپی به 10 کپی کاهش یابد در مرحله اول 8 کپی کاهش می‌یابد. در مرحله بعدی زمانی که تعداد کپی‌ها 72 است، 10% از کپی‌ها 7.2 است اما عدد به بالا گرد می‌شود تا 8. در هر حلقه از کنترل‌کننده مقیاس‌پذیر، تعداد کپی‌های تغییر می‌یابد که بر اساس تعداد کپی‌های فعلی محاسبه می‌شود. زمانی که تعداد کپی‌ها کمتر از 40 می‌شود، سیاست اول _(Pods)_ اعمال می‌شود و 4 کپی در هر بار کاهش یافته‌اند.

انتخاب سیاست می‌تواند با مشخص کردن فیلد `selectPolicy` برای یک جهت مقیاس‌پذیری تغییر یابد. با تنظیم مقدار به `Min` که سیاستی که کمترین تغییر در تعداد کپی را اجازه می‌دهد را انتخاب می‌

کند. تنظیم مقدار به `Disabled` مقیاس‌پذیری را به طور کامل در آن جهت غیرفعال می‌کند.
### پنجره ثابت‌سازی

پنجره ثابت‌سازی برای محدود کردن [پرشینگ](#flapping) تعداد نمونه‌های کپی هنگامی که متریک‌های استفاده شده برای مقیاس‌پذیری پرشتاب هستند، استفاده می‌شود. الگوریتم خودکار مقیاس‌پذیری از این پنجره برای استنباط حالت مطلوب پیشین استفاده می‌کند و از تغییرات غیرمطلوب در مقیاس کار بکشد.

به عنوان مثال، در قطعه کد زیر، یک پنجره ثابت‌سازی برای `scaleDown` مشخص شده است.

```yaml
behavior:
  scaleDown:
    stabilizationWindowSeconds: 300
```

وقتی که متریک‌ها نشان می‌دهند که هدف باید کاهش یابد، الگوریتم به حالت‌های مطلوب پیشین نگاه می‌کند و بالاترین مقدار را از بازه مشخص شده استفاده می‌کند. در مثال بالا، تمام حالت‌های مطلوب در 5 دقیقه گذشته در نظر گرفته می‌شود.

این به معنای نزدیک به حداکثر متحرک بودن است و از داشتن الگوریتم مقیاس‌پذیری که به طور فراوان پاده‌ها را حذف می‌کند فقط برای ایجاد دوباره یک پاد معادل چند لحظه بعد جلوگیری می‌کند.

### رفتار پیش‌فرض

برای استفاده از مقیاس‌پذیری سفارشی، نیازی به تعیین همه فیلدها نیست. تنها مقادیری که باید سفارشی شوند باید تعیین شوند. این مقادیر سفارشی با مقادیر پیش‌فرض ترکیب می‌شوند. مقادیر پیش‌فرض با رفتار موجود در الگوریتم HPA همخوانی دارند.

```yaml
behavior:
  scaleDown:
    stabilizationWindowSeconds: 300
    policies:
    - type: Percent
      value: 100
      periodSeconds: 15
  scaleUp:
    stabilizationWindowSeconds: 0
    policies:
    - type: Percent
      value: 100
      periodSeconds: 15
    - type: Pods
      value: 4
      periodSeconds: 15
    selectPolicy: Max
```

برای کاهش مقیاس، پنجره ثابت‌سازی _300_ ثانیه است (یا مقدار پرچین _--horizontal-pod-autoscaler-downscale-stabilization_ ارائه شده است). تنها یک سیاست برای کاهش مقیاس وجود دارد که به آن اجازه می‌دهد که 100٪ از نمونه‌های فعلی پرشتاب برداشته شود که به معنای این است که هدف مقیاس می‌تواند به نمونه‌های حداقل مجاز کاهش یابد.
برای بزرگ کردن مقیاس بالا، هیچ پنجره ثابت‌سازی وجود ندارد. وقتی که متریک‌ها نشان می‌دهند که هدف باید بالا رود، هدف به طور فوری بالا می‌رود. دو سیاست وجود دارد که در هر 15 ثانیه حداکثر 4 پاد یا 100٪ از نمونه‌های فعلی را اضافه کنند تا HPA به وضعیت ثابت خود برسد.

### مثال: تغییر پنجره ثابت‌سازی برای کاهش مقیاس

برای ارائه پنجره ثابت‌سازی کاهش مقیاس سفارشی به مدت 1 دقیقه، رفتار زیر به HPA اضافه می‌شود:

```yaml
behavior:
  scaleDown:
    stabilizationWindowSeconds: 60
```

### مثال: محدود کردن نرخ کاهش مقیاس

برای محدود کردن نرخی که Podها توسط HPA حذف می‌شوند به 10٪ در دقیقه، رفتار زیر به HPA اضافه می‌شود:

```yaml
behavior:
  scaleDown:
    policies:
    - type: Percent
      value: 10
      periodSeconds: 60
```

برای اطمینان از اینکه بیشتر از 5 پاد در دقیقه حذف نمی‌شوند، می‌توانید یک دومین سیاست کاهش مقیاس با اندازه ثابت 5 اضافه کنید و `selectPolicy` را به حداقل تنظیم کنید. تنظیم `selectPolicy` به `Min` به معنای این است که خودکار برای سیاستی که تعداد کمتری از پادها را تحت تأثیر قرار می‌دهد را انتخاب می‌کند:

```yaml
behavior:
  scaleDown:
    policies:
    - type: Percent
      value: 10
      periodSeconds: 60
    - type: Pods
      value: 5
      periodSeconds: 60
    selectPolicy: Min
```

### مثال: غیرفعال کردن کاهش مقیاس

مقدار `selectPolicy` به `Disabled` مقیاس‌پذیری جهت داده شده را خاموش می‌کند. بنابراین برای جلوگیری از کاهش مقیاس، سیاست زیر استفاده می‌شود:

```yaml
behavior:
  scaleDown:
    selectPolicy: Disabled
```

## پشتیبانی از HorizontalPodAutoscaler در `kubectl`

HorizontalPodAutoscaler، مانند هر منبع API دیگر، به یک روش استاندارد توسط `kubectl` پشتیبانی می‌شود.
می‌توانید با استفاده از دستور `kubectl create` یک خودکار جدید ایجاد کنید.
می‌توانید لیست خودکارها را با دستور `kubectl get hpa` یا توضیحات دقیقتر را با `kubectl describe hpa` دریافت کنید.
در نهایت، می‌توانید با استفاده از `kubectl delete hpa` یک خودکار ر

ا حذف کنید.

علاوه بر این، یک دستور خاص `kubectl autoscale` برای ایجاد یک شی HorizontalPodAutoscaler وجود دارد.
به عنوان مثال، اجرای `kubectl autoscale rs foo --min=2 --max=5 --cpu-percent=80`
یک خودکار برای ReplicaSet _foo_ ایجاد می‌کند، با استفاده از استفاده از CPU مقداری که بین 2 و 5 نسخه تنظیم شده است
و درصد بار CPU مقصد را به `80%` تنظیم می‌کند.

## غیرفعال کردن ضمنی حالت نگهداری

می‌توانید به طور ضمنی HPA را برای هدف بدون نیاز به تغییر پیکربندی HPA خود غیرفعال کنید. اگر تعداد نمونه‌های کپی مطلوب هدف به 0 تنظیم شود و حداقل تعداد نمونه‌های کپی HPA بیشتر از 0 باشد، HPA تنظیمات هدف را تنظیم نمی‌کند (و شرایط `ScalingActive` را بر روی خود به `false` تنظیم می‌کند) تا زمانی که شما با تنظیم مجدد تعداد نمونه‌های کپی مطلوب یا حداقل تعداد نمونه‌های کپی HPA آن را دوباره فعال کنید.

### مهاجرت از استقرارها و StatefulSets به مقیاس‌پذیری افقی

وقتی که یک HPA فعال است، توصیه می‌شود که مقدار `spec.replicas` از Deployment و/یا StatefulSet از
{{< glossary_tooltip text="manifest(s)" term_id="manifest" >}} آن‌ها حذف شود. اگر این انجام نشود، هر بار
تغییری در آن شیء اعمال می‌شود، به عنوان مثال از طریق `kubectl apply -f
deployment.yaml`، این دستور به Kubernetes می‌گوید که تعداد فعلی از پاد را برای مقدار کلیدی `spec.replicas` مقیاس
تنظیم می‌کند. این ممکن است مطلوب نباشد و ممکن است در هنگامی که یک HPA فعال است، مشکل‌ساز باشد.

در نظر داشته باشید که حذف `spec.replicas` می‌تواند منجر به یکباره‌اضطراب کم نیروی کار شود
تعداد از کلید پیش‌فرض این کلید 1 (مرجع
[نمونه‌های استقرار](/docs/concepts/workloads/controllers/deployment#replicas)).
در بروزرسانی، تمام پادها به جز 1 شروع به روند ترکیب می‌کنند. هر برنامهٔ بعدی اجرا می‌کنید
کنید که به طور معمول رفتار و تنظیمات بروزرسانی را در مورد چگونگی که شما از چگونگی مدیریت مشترک اجرا می‌کنید، حفظ کنید.

{{< tabs name="fix_replicas_instructions" >}}
{{% tab name="Client Side Apply (this is the default)" %}}

1. `kubectl apply edit-last-applied deployment/<deployment_name>`
2. In the editor, remove `spec.replicas`. When you save and exit the editor, `kubectl`
   applies the update. No changes to Pod counts happen at this step.
3. You can now remove `spec.replicas` from the manifest. If you use source code management,
   also commit your changes or take whatever other steps for revising the source code
   are appropriate for how you track updates.
4. From here on out you can run `kubectl apply -f deployment.yaml`

{{% /tab %}}
{{% tab name="Server Side Apply" %}}

هنگام استفاده از [Server-Side Apply](/docs/reference/using-api/server-side-apply/)
می‌توانید دنبال راهنمایی [انتقال مالکیت](/docs/reference/using-api/server-side-apply/#transferring-ownership)
را دنبال کنید که این مورد استفاده را پوشش می‌دهد.

{{% /tab %}}
{{< /tabs >}}

## {{% heading "whatsnext" %}}

اگر مقیاس‌پذیری را در خوشه‌تان پیکربندی کرده‌اید، ممکن است بخواهید در نظر بگیرید که از
[مقیاس‌پذیری خوشه](/docs/concepts/cluster-administration/cluster-autoscaling/)
اطمینان حاصل کنید تا مطمئن شوید که شما در حال اجرای تعداد مناسبی از نودها هستید.

برای اطلاعات بیشتر در مورد HorizontalPodAutoscaler:

- یک [مثال پیمایش](/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) برای اتوماسیون افقی پاد پیمایش کنید.
- مستندات برای [`kubectl autoscale`](/docs/reference/generated/kubectl/kubectl-commands/#autoscale) را مطالعه کنید.
- اگر می‌خواهید یک تبدیل متریک‌های سفارشی خود را بنویسید، چکیده کنید
  [شواهد اولیه](https://github.com/kubernetes-sigs/custom-metrics-apiserver) برای شروع کردن.
- مستندات [مرجع API](/docs/reference/kubernetes-api/workload-resources/horizontal-pod-autoscaler-v2/) برای HorizontalPodAutoscaler را مطالعه کنید.