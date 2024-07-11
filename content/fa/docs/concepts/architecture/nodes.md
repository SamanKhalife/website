---
reviewers:
- caesarxuchao
- dchen1107
title: Nodes
api_metadata:
- apiVersion: "v1"
  kind: "Node"
content_type: concept
weight: 10
---

<!-- overview -->

کوبرنیتیز با قرار دادن کانتینرها در پادها بر روی _Node ها_ بارگذاری {{< glossary_tooltip text="workload" term_id="workload" >}} شما را اجرا می‌کند.
یک node می‌تواند یک ماشین مجازی یا فیزیکی باشد، بسته به کلاستر. هر node توسط
{{< glossary_tooltip text="control plane" term_id="control-plane" >}}
مدیریت می‌شود و حاوی خدمات لازم برای اجرای
{{< glossary_tooltip text="Pod ها" term_id="pod" >}} است.

معمولاً شما چندین node در یک کلاستر دارید؛ در یک محیط آموزشی یا با محدودیت منابع، ممکن است فقط یک node داشته باشید.

[اجزاء](/docs/concepts/overview/components/#node-components) در یک node شامل
{{< glossary_tooltip text="kubelet" term_id="kubelet" >}},
{{< glossary_tooltip text="container runtime" term_id="container-runtime" >}} و
{{< glossary_tooltip text="kube-proxy" term_id="kube-proxy" >}} هستند.

<!-- body -->

## مدیریت

دو روش اصلی برای اضافه کردن node به
{{< glossary_tooltip text="API server" term_id="kube-apiserver" >}} وجود دارد:

1. kubelet روی یک node به صورت خودکار به control plane ثبت نام می‌کند
2. شما (یا یک کاربر دیگر) به صورت دستی یک شیء Node اضافه می‌کنید

بعد از ایجاد یک {{< glossary_tooltip text="object" term_id="object" >}} Node،
یا زمانی که kubelet روی یک node به صورت خودکار ثبت نام می‌کند، control plane بررسی می‌کند که شیء Node جدید
معتبر است یا خیر. به عنوان مثال، اگر شما سعی کنید یک Node را از فرمان JSON مندرج زیر ایجاد کنید:

```json
{
  "kind": "Node",
  "apiVersion": "v1",
  "metadata": {
    "name": "10.240.79.157",
    "labels": {
      "name": "my-first-k8s-node"
    }
  }
}
```

کوبرنیتیز به طور داخلی یک شیء Node ایجاد می‌کند (نمایندگی). کوبرنیتیز بررسی می‌کند که آیا kubelet به API server ثبت نام کرده است که با فیلد `metadata.name`
شیء Node مطابقت دارد یا خیر. اگر node سالم باشد (یعنی تمام خدمات لازم در حال اجرا باشند)،
سپس این مرتبط به اجرای یک Pod واجد شرایط است. در غیر این صورت، این node برای هر فعالیتی در کلاستر نادیده گرفته می‌شود
تا زمانی که سالم شود.

{{< note >}}
کوبرنیتیز شیء برای node نامعتبر را نگه می‌دارد و ادامه می‌دهد تا ببیند آیا آن سالم می‌شود یا خیر.

شما یا یک {{< glossary_tooltip term_id="controller" text="controller">}} باید به صورت صریح شیء Node را حذف کنید تا این بررسی سلامتی متوقف شود.
{{< /note >}}

نام یک شیء Node باید یک
[نام subdomain DNS معتبر](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names) باشد.

### یکتایی نام Node

نام [نام](/docs/concepts/overview/working-with-objects/names#names) یک Node را مشخص می‌کند. دو Node نمی‌توانند همزمان همین نام را داشته باشند. همچنین کوبرنیتیز فرض می‌کند که یک منبع با همین نام یک شیء یکسان است. در مورد یک Node، به طور ضمنی فرض می‌شود که یک نمونه با استفاده از همان نام، وضعیت یکسانی دارد (مانند تنظیمات شبکه، محتوای دیسک root) و ویژگی‌هایی مانند برچسب‌های node. این ممکن است منجر به عدم انطباق شود اگر یک نمونه بدون تغییر نام تغییر داده شود. اگر نیاز به جایگزینی یا به روزرسانی قابل توجهی Node باشد، شیء Node موجود باید ابتدا از API server حذف شود و پس از به روزرسانی مجدداً اضافه شود.

### ثبت خودکار Node ها

وقتی که پرچم kubelet `--register-node` true است (پیش‌فرض)، kubelet سعی می‌کند
که خودش را با API server ثبت کند. این الگوی ترجیحی است که توسط بیشتر توزیع‌ها استفاده می‌شود.

برای ثبت خودکار، kubelet با گزینه‌های زیر شروع می‌شود:

- `--kubeconfig` - مسیر اعتبارات برای احراز هویت خود به API server.
- `--cloud-provider` - چگونگی ارتباط با {{< glossary_tooltip text="cloud provider" term_id="cloud-provider" >}}
  برای خواندن متادیتا درباره خودش.
- `--register-node` - به صورت خودکار با API server ثبت نام کنید.
- `--register-with-taints` - Node را با لیست داده شده از
  {{< glossary_tooltip text="taints" term_id="taint" >}} (جدا شده با کاما `<key>=<value>:<effect>`) ثبت کنید.

  اگر `register-node` false باشد، این عملیات انجام نمی‌شود.
- `--node-ip` - لیست اختیاری IP address های Node به صورت جدا شده با کاما.
  شما می‌توانید فقط یک آدرس برای هر خانواده آدرس مشخص کنید.
  به عنوان مثال، در یک کلاستر IPv4 تک-stack، شما مقدار این را به آدرس IPv4 که kubelet برای Node باید استفاده کند تنظیم می‌کنید.
  برای جزئیات در مورد اجرای یک کلاستر دو-stack IPv4/IPv6، [configure IPv4/IPv6 dual stack](/docs/concepts/services-networking/dual-stack/#configure-ipv4-ipv6-dual-stack)
  را ببینید.

  اگر این آرگومان را ارائه ندهید، kubelet از آدرس IPv4 پیش‌فرض Node استفاده می‌کند، اگر هر دو IPv4 ندارد
  kubelet از آدرس IPv6 پیش‌فرض Node استفاده می‌کند.
- `--node-labels` - {{< glossary_tooltip text="Labels" term_id="label" >}} برای افزودن هنگام ثبت نام Node در کلاستر
  (محدودیت‌های برچسب اعمال شده توسط
  [NodeRestriction admission plugin](/docs/reference/access-authn-authz/admission-controllers/#noderestriction)
  را ببینید).
- `--node-status-update-frequency` - مشخص می‌کند که چقدر اغلب kubelet وضعیت node خود را به API server ارسال می‌کند.

وقتی حالت [مجوز Node](/docs/reference/access-authn-authz/node/) و
پلاگین اعتبارسنجی [NodeRestriction admission plugin](/docs/reference/access-authn-authz/admission-controllers/#noderestriction)
فعال باشد، kubelet ها فقط مجاز به ایجاد/تغییر منبع Node خودشان هستند.

{{< note >}}
همانطور که در بخش [یکتایی نام Node](#node-name-uniqueness) ذکر شده است،
وقتی که نیاز به به روزرسانی پیکربندی Node باشد، بهتر است node را مجدداً با API server ثبت کنید.
به عنوان مثال، اگر kubelet با مجموعه جدید `--node-labels` دوباره راه‌اندازی شود، اما همان نام Node استفاده شود، تغییر اعمال نخواهد شد، زیرا برچسب‌ها فقط هنگام ثبت نام Node با API server تنظیم می‌شوند (یا تغییر می‌کنند).

Pod هایی که قبلاً بر روی Node زمان‌بندی شده‌اند ممکن است در صورت تغییر پیکربندی Node در هنگام راه‌اندازی kubelet رفتار نامطلوب داشته باشند یا مشکل‌ها ایجاد کنند. به عنوان مثال، Pod هایی که در حال اجرای آنها تغییر برچسب جدیدی به Node زده شده‌اند، ممکن است به نام یک‌دیگر پرچم بگیرند. با مجدد ثبت نام Node، اطمینان حاصل می‌شود که همه Pod ها خالی می‌شوند و به درستی دوباره زمان‌بندی می‌شوند.
{{< /note >}}

### مدیریت دستی Node

شما می‌توانید شیء Node را با استفاده از
{{< glossary_tooltip text="kubectl" term_id="kubectl" >}} ایجاد و تغییر دهید.

وقتی که می‌خواهید به صورت دستی شیء Node ایجاد کنید، پرچم kubelet `--register-node=false` را تنظیم کنید.

شما می‌توانید شیء Node را بدون توجه به تنظیم `--register-node` تغییر دهید.
به عنوان مثال، شما می‌توانید برچسب‌ها را بر روی یک شیء Node موجود تنظیم کنید یا آن را غیرقابل زمان‌بندی کنید.

شما می‌توانید از برچسب‌ها بر روی Node ها به همراه انتخاب‌کننده Node ها در Pod ها برای کنترل زمان‌بندی استفاده کنید.
به عنوان مثال، شما می‌توانید Pod را محدود به اجرای فقط روی یک زیرمجموعه از Node های موجود کنید.

نشان دادن یک Node به عنوان غیرقابل زمان‌بندی، از ایجاد پرچم از جمله جدید Pod ها بر روی آن Node جلوگیری می‌کند اما تأثیری بر روی Pod های موجود در Node ندارد. این مفید است به عنوان
مرحله آماده کننده قبل از راه‌اندازی مجدد Node یا نگهداری دیگر.

برای نشان دادن یک Node به عن

وان غیرقابل زمان‌بندی، اجرا کنید:

```shell
kubectl cordon $NODENAME
```

برای جزئیات بیشتر، [Safely Drain a Node](/docs/tasks/administer-cluster/safely-drain-node/)
را ببینید.

{{< note >}}
Pod هایی که قسمت یک {{< glossary_tooltip term_id="daemonset" >}} هستند ممکن است در زمان اجرای آنها بر روی یک Node غیرقابل زمان‌بندی تحمل کنند. DaemonSet ها به طور معمول خدمات محلی Node را ارائه می‌دهند
که باید بر روی Node اجرا شود حتی اگر در حال تخلیه از برنامه‌های کاربردی کاربردی هستند.
{{< /note >}}
```markdown
## وضعیت Node

وضعیت یک Node شامل اطلاعات زیر است:

* [آدرس‌ها](/docs/reference/node/node-status/#addresses)
* [شرایط](/docs/reference/node/node-status/#condition)
* [ظرفیت و قابل تخصیص](/docs/reference/node/node-status/#capacity)
* [اطلاعات](/docs/reference/node/node-status/#info)

شما می‌توانید از `kubectl` برای مشاهده وضعیت یک Node و سایر جزئیات استفاده کنید:

```shell
kubectl describe node <نام-Node-را-وارد-کنید>
```

برای جزئیات بیشتر، [وضعیت Node](/docs/reference/node/node-status/) را ببینید.

## Heartbeat های Node

Heartbeat های ارسالی توسط Node های Kubernetes به کلاستر شما کمک می‌کنند تا دسترسی هر Node را تعیین کنند
و در صورت شناسایی مشکلات، اقدام مناسب انجام دهند.

برای Node ها دو نوع heartbeat وجود دارد:

* به‌روزرسانی‌های [`.status`](/docs/reference/node/node-status/) یک Node.
* اشیاء [Lease](/docs/concepts/architecture/leases/) در فضای نام `kube-node-lease`
  {{< glossary_tooltip term_id="namespace" text="namespace">}}.
  هر Node یک شیء Lease مرتبط دارد.

## کنترل‌کننده Node

کنترل‌کننده Node یک جزء از کامپوننت‌های کنترل پلان Kubernetes است که مدیریت انواع مختلف Node ها را بر عهده دارد.

کنترل‌کننده Node نقش‌های متعددی در زندگی یک Node دارد. اولین آن تخصیص یک بلوک CIDR به Node هنگامی که ثبت می‌شود (اگر تخصیص CIDR فعال باشد) است.

دومین نقش آن حفظ لیست داخلی کنترل‌کننده Node از Node ها با لیست ابرپروایدر ماشین‌های موجود است. هنگام اجرا در محیط ابر
و هر زمان که یک Node ناسالم است، کنترل‌کننده Node از ابرپروایدر می‌پرسد که آیا ماشین مجازی برای آن Node هنوز موجود است یا خیر.
اگر نه، کنترل‌کننده Node Node را از لیست Node های خود حذف می‌کند.

سومین نقش آن نظارت بر سلامت Node هاست. کنترل‌کننده Node مسئول است برای:

- در صورتی که یک Node غیرقابل دسترسی شود، به‌روزرسانی شرط `Ready`
  در فیلد `.status` Node. در این صورت کنترل‌کننده Node شرط `Ready` را به `Unknown` تنظیم می‌کند.
- اگر یک Node به‌طور دائمی غیرقابل دسترسی باقی بماند: مقداردهی اولیه
  [اخراج API-initiated](/docs/concepts/scheduling-eviction/api-eviction/)
  برای همه Pod های موجود در Node غیرقابل دسترسی. به طور پیش‌فرض، کنترل‌کننده Node
  بین تنظیم کردن Node به `Unknown` و ارسال
  درخواست اولیه اخراج، 5 دقیقه انتظار می‌کند.

به طور پیش‌فرض، کنترل‌کننده Node وضعیت هر Node را هر 5 ثانیه یکبار بررسی می‌کند.
این دوره می‌تواند با استفاده از پرچم `--node-monitor-period` در
کامپوننت `kube-controller-manager` پیکربندی شود.

### محدودیت‌های نرخ در اخراج

در بیشتر موارد، کنترل‌کننده Node محدودیت نرخ اخراج را به `--node-eviction-rate` (پیش‌فرض 0.1) در ثانیه می‌گذارد، به این معنی که از 1 Node در هر 10 ثانیه اخراج Pod ها را ممنوع می‌کند.

رفتار اخراج Node تغییر می‌کند زمانی که یک Node در یک منطقه دسترسی به ناتوانی می‌یابد. کنترل‌کننده Node بررسی می‌کند که چه درصد از Node های در منطقه ناسالم است (شرط `Ready` `Unknown` یا `False`) در همان زمان:

- اگر برخورداری از قسمت ناسالم کمتر از `--unhealthy-zone-threshold`
  (پیش‌فرض 0.55) است، پس از آن نرخ اخراج کاهش می‌یابد.
- اگر کلاستر کوچک باشد (یعنی کمتر یا مساوی با `--large-cluster-size-threshold` Node ها - پیش‌فرض 50)، اخراجات متوقف می‌شوند.
- در غیر این صورت، نرخ اخراج به `--secondary-node-eviction-rate` (پیش‌فرض 0.01) در ثانیه کاهش می‌یابد.

دلیل پیاده‌سازی این سیاست‌ها در هر منطقه در دسترسی‌پذیری است زیرا یک منطقه دسترسی به کنترل پلان برخی از Node ها را ممکن است جدا کند. اگر کلاستر شما چند منطقه در دسترسی ابرپروایدر را نمی‌پذیرد،
پس مکانیسم اخراج هیچ حسابی از عدم دسترسی به هر منطقه را در نظر نمی‌گیرد.

یک دلیل اصلی برای پراکندگی Node ها در طول مناطق در دسترسی است تا بار کاری را بتوانید به مناطق سالم منتقل کنید زمانی که یک منطقه کامل به پایین می‌رود.
بنابراین، اگر تمام Node ها در یک منطقه ناسالم باشند، آنگاه کنتر

## پیگیری ظرفیت منابع {#node-capacity}

اشیاء Node اطلاعاتی را درباره ظرفیت منابع Node ردیابی می‌کنند: به عنوان مثال، مقدار حافظه موجود و تعداد CPU ها.
Node هایی که [خودشان را ثبت می‌کنند](#self-registration-of-nodes)، ظرفیت خود را هنگام ثبت نام گزارش می‌دهند. اگر شما [به صورت دستی](#manual-node-administration) یک Node اضافه کنید،
باید اطلاعات ظرفیت Node را هنگام اضافه کردن آن تنظیم کنید.

{{< glossary_tooltip text="scheduler" term_id="kube-scheduler" >}} Kubernetes اطمینان حاصل می‌کند که منابع کافی برای تمامی Pods ها در یک Node وجود دارد. این scheduler بررسی می‌کند که مجموع درخواست‌های کانتینرها در Node بیشتر از ظرفیت Node نباشد. این مجموع درخواست‌ها شامل تمام کانتینرهای مدیریت شده توسط kubelet است، اما هرگونه کانتینری که مستقیماً توسط runtime کانتینر شروع شده باشد و همچنین هر فرآیندی که خارج از کنترل kubelet اجرا شود، مستثنی می‌شود.

{{< note >}}
اگر می‌خواهید به طور صریح منابع را برای فرآیندهای غیر-Pod رزرو کنید، به
[رزرو منابع برای دیمون‌های سیستم](/docs/tasks/administer-cluster/reserve-compute-resources/#system-reserved)
مراجعه کنید.
{{< /note >}}

## Topology توپولوژی Node

{{< feature-state feature_gate_name="TopologyManager" >}}

اگر شما `TopologyManager`
[feature gate](/docs/reference/command-line-tools-reference/feature-gates/)
را فعال کرده‌اید، آنگاه kubelet می‌تواند هنگام تصمیم‌گیری‌های تخصیص منابع از شوروالی توپولوژی استفاده کند.
برای اطلاعات بیشتر، [کنترل سیاست‌های مدیریت توپولوژی در یک Node](/docs/tasks/administer-cluster/topology-manager/)
را ببینید.

## مدیریت حافظه Swap {#swap-memory}

{{< feature-state feature_gate_name="NodeSwap" >}}

برای فعال کردن Swap در یک Node، باید ویژگی `NodeSwap` را در kubelet فعال کنید (پیش‌فرض true) و پرچم خط فرمان `--fail-swap-on` یا تنظیم پیکربندی `failSwapOn`
[راست مرتبط](/docs/reference/config-api/kubelet-config.v1beta1/)
را به false تنظیم کنید.
برای اجازه به Pods برای استفاده از swap، `swapBehavior` نباید به `NoSwap` (که رفتار پیش‌فرض است) در پیکربندی kubelet تنظیم شود.

{{< warning >}}
هنگامی که ویژگی حافظه swap فعال شود، داده‌های Kubernetes مانند محتوای اشیاء Secret که به tmpfs نوشته شده‌اند، ممکن است به دیسک swap شود.
{{< /warning >}}

کاربر می‌تواند نیز به‌طور اختیاری `memorySwap.swapBehavior` را پیکربندی کند تا مشخص کند چگونه یک Node از حافظه swap استفاده خواهد کرد. به عنوان مثال،

```yaml
memorySwap:
  swapBehavior: LimitedSwap
```

- `NoSwap` (پیش‌فرض): بارهای کاری Kubernetes از swap استفاده نمی‌کنند.
- `LimitedSwap`: استفاده از حافظه swap توسط بارهای کاری Kubernetes تحت محدودیت قرار می‌گیرد.
  تنها Pods با QoS برازنده مجاز به استفاده از swap هستند.

اگر پیکربندی برای `memorySwap` مشخص نشده باشد و ویژگی gate فعال شود، به طور پیش‌فرض kubelet
همان رفتار `NoSwap` را اعمال می‌کند.

با `LimitedSwap`، Pods که تحت QoS برازنده قرار دارند (به عبارت دیگر، Pods `BestEffort`/`Guaranteed`) ممنوع هستند که از حافظه swap استفاده کنند.
برای حفظ این تضمینات امنیتی و سلامت Node، این Pods اجازه استفاده از حافظه swap را ندارند هنگامی که `LimitedSwap` فعال است.

قبل از جزئیات محاسبه حداکثر حافظه swap، لازم است که اصطلاحات زیر را تعریف کنیم:

* `nodeTotalMemory`: مجموع حافظه فیزیکی موجود در Node.
* `totalPodsSwapAvailable`: مجموع حافظه swap در Node که برای استفاده توسط Pods در دسترس است
  (بعضی از حافظه swap ممکن است برای استفاده سیستم رزرو شود).
* `containerMemoryRequest`: درخواست حافظه کانتینر.

محدودیت swap به شکل زیر پیکربندی می‌شود:
`(containerMemoryRequest / nodeTotalMemory) * totalPodsSwapAvailable`.

لطفاً توجه داشته باشید که برای کانتینرهای درون Pods با QoS برازنده، ممکن است از استفاده از swap خودداری کنید با تعیین درخواست‌های حافظه که برابر با حداکثر حافظه می‌شود.
کانتینرهایی که به این ترتیب پیکربندی شده‌اند، به حافظه swap دسترسی ندارند.

Swap فقط با **cgroup v2** پشتیبانی می‌شود، cgroup v1 پشتیبانی نمی‌شود.

برای اطلاعات بیشتر، و به منظور کمک به تست و ارائه بازخورد، لطفاً
مطالعه کنید [درباره Kubernetes 1.28: NodeSwap graduates to Beta1](/blog/2023/08/

24/swap-linux-beta/),
[KEP-2400](https://github.com/kubernetes/enhancements/issues/4128) و
[پیشنهاد طراحی](https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/2400-node-swap/README.md).
```
