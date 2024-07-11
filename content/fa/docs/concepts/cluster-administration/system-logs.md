---
reviewers:
- dims
- 44past4
title: لاگ‌های سیستم
content_type: concept
weight: 80
---

<!-- overview -->

لاگ‌های مؤلفه‌های سیستم رویدادهای رخ داده در خوشه Kubernetes را ثبت می‌کنند که برای اشکال‌زدایی بسیار مفید هستند.
شما می‌توانید پیچیدگی لاگ را به گونه‌ای پیکربندی کنید که جزئیات بیشتر یا کمتری را ببینید.
لاگ‌ها می‌توانند به عنوان یک مؤشر خشن نشان دهنده خطاها در یک مؤلفه باشند یا به صورت دقیق‌تر به نمایش اطلاعات کامل گام به گام از رویدادها بپردازند (مانند لاگ‌های دسترسی HTTP، تغییرات وضعیت پاد، اقدامات کنترل‌گر یا تصمیم‌گیری‌های برنامه‌ریز).

<!-- body -->

{{< warning >}}
بر خلاف پرچم‌های خط فرمان که در اینجا توضیح داده شده‌اند، *خروجی لاگ* خود تحت ضمانت پایداری API Kubernetes نیست:
ورودی‌های لاگ فردی و فرمت آنها ممکن است از یک نسخه به نسخه دیگر تغییر کند!
{{< /warning >}}

## Klog

klog کتابخانه لاگ‌نویسی Kubernetes است. [klog](https://github.com/kubernetes/klog)
پیام‌های لاگ را برای مؤلفه‌های سیستم Kubernetes تولید می‌کند.

Kubernetes در حال ساده‌سازی لاگ‌نویسی در مؤلفه‌های خود است.
پرچم‌های خط فرمان زیر klog
[از Kubernetes v1.23 منسوخ شده‌اند](https://github.com/kubernetes/enhancements/tree/master/keps/sig-instrumentation/2845-deprecate-klog-specific-flags-in-k8s-components)
و در Kubernetes v1.26 حذف می‌شوند:

- `--add-dir-header`
- `--alsologtostderr`
- `--log-backtrace-at`
- `--log-dir`
- `--log-file`
- `--log-file-max-size`
- `--logtostderr`
- `--one-output`
- `--skip-headers`
- `--skip-log-headers`
- `--stderrthreshold`

خروجی همیشه به stderr نوشته می‌شود، بدون توجه به فرمت خروجی. هدایت خروجی انتظار می‌رود توسط مؤلفه‌ای که یک مؤلفه Kubernetes را فراخوانی می‌کند، انجام شود. این می‌تواند یک شل POSIX یا ابزاری مانند systemd باشد.

در برخی موارد، مانند یک کانتینر distroless یا یک سرویس سیستم ویندوز، این گزینه‌ها در دسترس نیستند. در این صورت، باید از باینری
[`kube-log-runner`](https://github.com/kubernetes/kubernetes/blob/d2a8a81639fcff8d1221b900f66d28361a170654/staging/src/k8s.io/component-base/logs/kube-log-runner/README.md)
به عنوان یک پوشش برای یک مؤلفه Kubernetes استفاده کنید تا خروجی را هدایت کنید. یک باینری پیش‌ساخته در چند تصویر اصلی Kubernetes به عنوان `/go-runner` و در آرشیوهای انتشار سرور و node به عنوان `kube-log-runner` وجود دارد.

این جدول نشان می‌دهد که چگونه فراخوانی‌های `kube-log-runner` متناسب با هدایت خروجی شل است:

| استفاده | شل POSIX (مانند bash) | `kube-log-runner <options> <cmd>`                           |
| ---------|-------------------------|-------------------------------------------------------------|
| ادغام stderr و stdout، نوشتن به stdout | `2>&1`                     | `kube-log-runner` (رفتار پیش‌فرض)                        |
| هدایت هر دو به فایل لاگ             | `1>>/tmp/log 2>&1`         | `kube-log-runner -log-file=/tmp/log`                        |
| کپی کردن به فایل لاگ و به stdout    | `2>&1 \| tee -a /tmp/log`  | `kube-log-runner -log-file=/tmp/log -also-stdout`           |
| هدایت فقط stdout به فایل لاگ       | `>/tmp/log`                | `kube-log-runner -log-file=/tmp/log -redirect-stderr=false` |

### خروجی Klog

یک نمونه از فرمت سنتی نیتیو klog:

```
I1025 00:15:15.525108       1 httplog.go:79] GET /api/v1/namespaces/kube-system/pods/metrics-server-v0.3.1-57c75779f-9p8wg: (1.512ms) 200 [pod_nanny/v0.0.0 (linux/amd64) kubernetes/$Format 10.56.1.19:51756]
```

رشته پیام ممکن است حاوی شکست خطی باشد:

```
I1025 00:15:15.525108       1 example.go:79] This is a message
which has a line break.
```

### لاگ‌نویسی ساختارمند

{{< feature-state for_k8s_version="v1.23" state="beta" >}}

{{< warning >}}
مهاجرت به پیام‌های لاگ ساختارمند یک فرآیند در حال انجام است. همه پیام‌های لاگ در این نسخه ساختارمند نیستند. هنگام تجزیه فایل‌های لاگ، باید همچنین پیام‌های لاگ ساختارمند نا ساخته را هندل کنید.

فرمت‌بندی لاگ و سریال‌سازی مقادیر تحت تغییر قرار می‌گیرند.
{{< /warning>}}

لاگ‌نویسی ساختارمند یک ساختار یکپارچه را در پیام‌های لاگ معرفی می‌کند که امکان استخراج برنامه‌ریزی شده اطلاعات را فراهم می‌کند. شما می‌توانید با کمترین تلاش و هزینه لاگ‌های ساختارمند را ذخیره و پردازش کنید. کدی که پیام لاگ را ت

ولید می‌کند تعیین می‌کند که آیا از خروجی سنتی klog غیرساختارمند استفاده می‌کند یا لاگ‌نویسی ساختارمند.

فرمت پیش‌فرض پیام‌های لاگ ساختارمند به صورت متنی است، با یک فرمت که با klog سنتی سازگاری دارد:

```
<klog header> "<message>" <key1>="<value1>" <key2>="<value2>" ...
```

نمونه:

```
I1025 00:15:15.525108       1 controller_utils.go:116] "وضعیت پاد به روز شده" pod="kube-system/kubedns" status="آماده"
```

رشته‌ها نقل قول شده‌اند. مقادیر دیگر با
[`%+v`](https://pkg.go.dev/fmt#hdr-Printing) قالب‌بندی شده‌اند، که ممکن است باعث ادامه پیام‌های لاگ به خط بعدی شود
[بسته به داده‌ها](https://github.com/kubernetes/kubernetes/issues/106428).

```
I1025 00:15:15.525108       1 example.go:116] "مثال" data="این متن دارای یک شکست خطی است\nو \"نقل قول\" دارد." someInt=1 someFloat=0.1 someStruct={StringField: خط اول،
در خط دوم.}
```

### لاگ‌نویسی زمینه‌ای

{{< feature-state for_k8s_version="v1.30" state="beta" >}}

لاگ‌نویسی زمینه‌ای بر روی لاگ‌نویسی ساختارمند متکی است. اصولاً درباره چگونگی استفاده توسعه‌دهندگان از تماس‌های لاگ‌نویسی است: کد مبتنی بر آن مفصل‌تر است و پشتیبانی از موردهای استفاده اضافی را که در [KEP لاگ‌نویسی زمینه‌ای](https://github.com/kubernetes/enhancements/tree/master/keps/sig-instrumentation/3077-contextual-logging) توضیح داده شده، پشتیبانی می‌کند.

اگر توسعه‌دهندگان از توابع اضافی مانند `WithValues` یا `WithName` در مؤلفه‌های خود استفاده کنند، آنگاه ورودی‌های لاگ حاوی اطلاعات اضافی می‌شود که توسط تماس کننده‌های آن تابع به آن منتقل می‌شود.

برای Kubernetes {{< skew currentVersion >}}، این ویژگی به واسطه پایه
[ویژگی](/docs/reference/command-line-tools-reference/feature-gates/) `ContextualLogging` فعال است و به صورت پیش‌فرض فعال است. زیرساخت برای این کار در نسخه 1.24 بدون تغییر در مؤلفه‌ها اضافه شده است. دستور
[`component-base/logs/example`](https://github.com/kubernetes/kubernetes/blob/v1.24.0-beta.0/staging/src/k8s.io/component-base/logs/example/cmd/logger.go)
نشان می‌دهد که چگونه از تماس‌های لاگ جدید استفاده کنید و چگونه یک مؤلفه را که از لاگ‌نویسی زمینه‌ای پشتیبانی می‌کند، عمل می‌کند.

```console
$ cd $GOPATH/src/k8s.io/kubernetes/staging/src/k8s.io/component-base/logs/example/cmd/
$ go run . --help
...
      --feature-gates mapStringBool  A set of key=value pairs that describe feature gates for alpha/experimental features. Options are:
                                     AllAlpha=true|false (ALPHA - default=false)
                                     AllBeta=true|false (BETA - default=false)
                                     ContextualLogging=true|false (BETA - default=true)
$ go run . --feature-gates ContextualLogging=true
...
I0222 15:13:31.645988  197901 example.go:54] "runtime" logger="example.myname" foo="bar" duration="1m0s"
I0222 15:13:31.646007  197901 example.go:55] "another runtime" logger="example" foo="bar" duration="1h0m0s" duration="1m0s"
```

کلید `logger` و `foo="bar"` توسط تماس کننده تابعی که پیام `runtime` را لاگ می‌کند و مقدار `duration="1m0s"`، بدون این که لازم باشد آن تابع را تغییر دهد.

با غیرفعال کردن لاگ‌نویسی زمینه‌ای، `WithValues` و `WithName` هیچ کاری نمی‌کنند و تماس‌های لاگ از طریق لاگر klog جهانی می‌روند. بنابراین اطلاعات اضافی در خروجی لاگ دیگر موجود نیست:

```console
$ go run . --feature-gates ContextualLogging=false
...
I0222 15:14:40.497333  198174 example.go:54] "runtime" duration="1m0s"
I0222 15:14:40.497346  198174 example.go:55] "another runtime" duration="1h0m0s" duration="1m0s"
```

### فرمت لاگ JSON

{{< feature-state for_k8s_version="v1.19" state="alpha" >}}

{{<warning >}}
خروجی JSON از بسیاری از پرچم‌های استاندارد klog پشتیبانی نمی‌کند. برای لیست پرچم‌هایی که توسط klog پشتیبانی نمی‌شوند، می‌توانید به
[مرجع ابزار خط فرمان](/docs/reference/command-line-tools-reference/) مراجعه کنید.

همه لاگ‌ها تضمین نمی‌شود که به فرمت JSON نوشته شوند (به عنوان مثال، در زمان شروع پردازش). اگر قصد دارید که لاگ‌ها را تجزیه کنید، اطمینان حاصل کنید که می‌توانید خطوط لاگ را که به فرمت JSON نیستند نیز پردازش کنید.

نام‌های فیلد و سریال‌سازی JSON ممکن است تغییر کند.
{{< /warning >}}

پرچم `--logging-format=json` فرمت لاگ‌ها را از فرمت اصلی klog به فرمت JSON تغییر می‌دهد.
نمونه‌ای از فرمت لاگ JSON (به صورت pretty printed):

```json
{
   "ts": 1580306777.04728,
   "v": 4,
   "msg": "وضعیت پاد به روز شده",
   "pod":{
      "name": "nginx-1",
      "namespace": "default"
   },
   "status": "آماده"
}
```

کلید‌هایی که معانی خاصی دارند:

* `ts` - زمان‌نمای Unix به عنوان شماره (اجباری، شناور)
* `v` - دقت (فقط برای اطلاعات و نه برای پیام‌های خطا، عدد صحیح)
* `err` - رشته خطا (اختیاری، رشته)
* `msg` - پیام (اجباری، رشته)

لیستی از مؤلفه‌هایی که در حال حاضر فرمت JSON را پشتیبانی می‌کنند:

* {{< glossary_tooltip term_id="kube-controller-manager" text="مدیرکنترل کوبه" >}}
* {{< glossary_tooltip term_id="kube-apiserver" text="کوبه-سرور" >}}
* {{< glossary_tooltip term_id="kube-scheduler" text="زمان‌بند کوبه" >}}
* {{< glossary_tooltip term_id="kubelet" text="کوبلت" >}}

### سطح دقت لاگ

پرچم `-v` سطح دقت لاگ را کنترل می‌کند. افزایش مقدار، تعداد رویدادهای لاگ شده را افزایش می‌دهد.
کاهش مقدار، تعداد رویدادهای لاگ شده را کاهش می‌دهد. تنظیمات دقت بالاتر، رویدادهای کمتر و خطرناکتر را لاگ می‌کند. تنظیم دقت صفر فقط رویدادهای بحرانی را لاگ می‌کند.

### مکان لاگ

دو نوع مؤلفه سیستم وجود دارد: آنهایی که در یک کانتینر اجرا می‌شوند و آنهایی
که در یک کانتینر اجرا نمی‌شوند. به عنوان مثال:

* زمان‌بند Kubernetes و متمرکز کوبه در یک کانتینر اجرا می‌شوند.
* کوبلت و {{<glossary_tooltip term_id="container-runtime" text="زمان‌بند کوبه">}}
  در کانتینر‌ها اجرا نمی‌شوند.

در دستگاه‌هایی با systemd، کوبلت و زمان‌بند کانتینر به journald می‌نویسند.
در غیر این صورت، آنها به فایل‌های `.log` در دایرکتوری `/var/log` می‌نویسند.
مؤلفه‌های سیستم درون کانتینرها همیشه به فایل‌های `.log` در دایرکتوری `/var/log` می‌نویسند
و مکانیسم پیش‌فرض لاگ‌نویسی را طرد می‌کنند.
مانند لاگ‌های کانتینر، شما باید لاگ‌های مؤلفه‌های سیستم را در دایرکتوری `/var/log` چرخانید.
در خوشه‌های Kubernetes ایجاد شده توسط اسکریپت `kube-up.sh`، چرخش لاگ توسط ابزار `logrotate` پیکربندی شده است.
ابزار `logrotate` لاگ‌ها را روزانه چرخانده می‌کند یا یک بار اندازه لاگ بیش از 100MB است.

## پرس و جوی لاگ

{{< feature-state feature_gate_name="NodeLogQuery" >}}

برای کمک به رفع مشکلات در دستگاه‌ها، Kubernetes v1.27 ویژگی‌ای را معرفی کرد که اجازه مشاهده لاگ‌های سرویس‌های در حال اجرا روی دستگاه‌ها را می‌دهد.
برای استفاده از این ویژگی، اطمینان حاصل کنید که پرچم `NodeLogQuery`
[feature gate](/docs/reference/command-line-tools-reference/feature-gates/) برای آن دستگاه فعال باشد، و که گزینه‌های پیکربندی کوبلت `enableSystemLogHandler` و `enableSystemLogQuery` هر دو به صورت درست تنظیم شده باشند. در لینوکس
فرض بر این است که لاگ‌های سرویس از طریق journald در دسترس است. در ویندوز فرض بر این است که لاگ‌های سرویس در ارائه دهنده لاگ برنامه قابل دسترسی است. در هر دو سیستم عامل، لاگ‌ها نیز از طریق خواندن فایل‌ها در دایرکتوری
`/var/log/` در دسترس است.

اگر مجوز برای تعامل با اشیاء دستگاه دارید، می‌توانید این ویژگی را

 بر روی تمام دستگاه‌های خود یا فقط یک زیرمجموعه از آنها امتحان کنید. در اینجا یک مثال برای بازیابی لاگ‌های سرویس کوبلت از یک دستگاه آورده شده است:

```shell
# فچ کردن لاگ‌های کوبلت از یک دستگاه به نام node-1.example
kubectl get --raw "/api/v1/nodes/node-1.example/proxy/logs/?query=kubelet"
```

همچنین می‌توانید فایل‌ها را فچ کنید، به شرط آنکه فایل‌ها در یک دایرکتوری باشند که کوبلت اجازه می‌دهد برای فچ. به عنوان مثال، می‌توانید یک لاگ از `/var/log` روی یک دستگاه لینوکس فچ کنید:

```shell
kubectl get --raw "/api/v1/nodes/<نام-دستگاه-را-وارد-کنید>/proxy/logs/?query=/<نام-فایل-لاگ-را-وارد-کنید>"
```

کوبلت از هوریستیک برای بازیابی لاگ‌ها استفاده می‌کند. این به کمک می‌آید اگر آگاه نباشید که آیا یک سرویس سیستمی خاص لاگ‌ها را به لاگر نیتیو سیستم عامل مانند journald می‌نویسد یا به یک فایل لاگ در
`/var/log/<نام-سرویس>` یا `/var/log/<نام-سرویس>.log` یا `/var/log/<نام-سرویس>/<نام-سرویس>.log` می‌نویسد.

لیست کامل گزینه‌های قابل استفاده:

گزینه | توضیحات
------ | -----------
`boot` | نمایش پیام‌های بوت از یک بوت سیستم خاص
`pattern` | الگوهای فیلتر لاگ را با استفاده از عبارات منظم متناسب با PERL فیلتر کنید
`query` | سرویس(ها) یا فایل(ها) را که می‌خواهید لاگشان را برگردانید (اجباری)
`sinceTime` | یک برچسب زمانی [RFC3339](https://www.rfc-editor.org/rfc/rfc3339) که نشان می‌دهد لاگ‌ها از آن زمان نمایش داده شده است (شامل)
`untilTime` | یک برچسب زمانی [RFC3339](https://www.rfc-editor.org/rfc/rfc3339) که نشان می‌دهد لاگ‌ها تا آن زمان نمایش داده شده است (شامل)
`tailLines` | تعیین می‌کند چند خط از انتهای لاگ برای بازیابی شود؛ پیش‌فرض این است که تمام لاگ را فچ می‌کند

نمونه‌ای از یک پرس‌وجوی پیچیده‌تر:

```shell
# فچ کردن لاگ‌های کوبلت از یک دستگاه به نام node-1.example که دارای کلمه "error" هستند
kubectl get --raw "/api/v1/nodes/node-1.example/proxy/logs/?query=kubelet&pattern=error"
```

## {{% heading "whatsnext" %}}

* درباره [معماری لاگ Kubernetes](/docs/concepts/cluster-administration/logging/) بخوانید
* درباره [لاگ‌نویسی ساختارمند](https://github.com/kubernetes/enhancements/tree/master/keps/sig-instrumentation/1602-structured-logging) بخوانید
* درباره [لاگ‌نویسی زمینه‌ای](https://github.com/kubernetes/enhancements/tree/master/keps/sig-instrumentation/3077-contextual-logging) بخوانید
* درباره [از بین بردن پرچم‌های klog](https://github.com/kubernetes/enhancements/tree/master/keps/sig-instrumentation/2845-deprecate-klog-specific-flags-in-k8s-components) بخوانید
* درباره [راهنمایی‌های جهت جداسازی شدت لاگ](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-instrumentation/logging.md) بخوانید
* درباره [پرس‌وجوی لاگ](https://kep.k8s.io/2258) بخوانید
