---
reviewers:
- mikedanese
title: بهترین شیوه‌های پیکربندی
content_type: concept
weight: 10
---

<!-- overview -->
این سند بهترین شیوه‌های پیکربندی را که در سراسر راهنمای کاربر، مستندات شروع به کار و مثال‌ها معرفی شده است، برجسته و یکپارچه می‌کند.

این یک سند زنده است. اگر به چیزی که در این لیست نیست اما ممکن است برای دیگران مفید باشد، مرتکب شدید، لطفاً دو برخورد کنید یا PR ارائه دهید.

<!-- body -->
## نکات کلی پیکربندی

- در تعریف پیکربندی‌ها، ورژن API پایدار جدید را مشخص کنید.

- فایل‌های پیکربندی باید قبل از ارسال به خوشه در کنترل نسخه ذخیره شوند. این امکان را به شما می‌دهد که در صورت لزوم تغییرات پیکربندی را به سرعت بازگردانید. همچنین به بازسازی و بازیابی خوشه کمک می‌کند.

- فایل‌های پیکربندی خود را با استفاده از YAML به جای JSON بنویسید. اگرچه این فرمت‌ها در تقریباً همه حالات قابل استفاده می‌باشند، YAML به نظر دوستانه‌تر است.

- اشیاء مرتبط را در یک فایل ترکیب کنید هر زمان که ممکن است منطقی باشد. یک فایل مدیریت کردنی‌تر از چندین فایل است.

- توجه داشته باشید که بسیاری از دستورات `kubectl` می‌توانند بر روی یک دایرکتوری فراخوانی شوند. به عنوان مثال، می‌توانید دستور `kubectl apply` را بر روی یک دایرکتوری از فایل‌های پیکربندی فراخوانی کنید.

- مقادیر پیش‌فرض را بی‌دلیل مشخص نکنید: پیکربندی ساده و کم‌تر، احتمال خطاها را کمتر می‌کند.

- توضیحات اشیاء را در annotations قرار دهید تا امکان تجزیه و تحلیل بهتر را فراهم کنید.

{{< note >}}
یک تغییر شکننده در مشخصات مقدار بولیای YAML 1.2 با مشخصات YAML 1.1 معرفی شده است. این یک
[مشکل معروف](https://github.com/kubernetes/kubernetes/issues/34146) در Kubernetes است.
YAML 1.2 تنها **true** و **false** را به عنوان مقادیر معتبر بولیای قبول می‌کند، در حالی که YAML 1.1
همچنین **yes**، **no**، **on** و **off** را به عنوان مقادیر بولیای قبول می‌کند. با این حال، Kubernetes از پارسرهای YAML استفاده می‌کند
[که اکثراً سازگار با YAML 1.1 هستند](https://github.com/kubernetes/kubernetes/issues/34146#issuecomment-252692024)، که به این معنی است که استفاده از **yes** یا **no** به جای **true** یا **false** در یک منیفست YAML ممکن است خطاها یا رفتارهای غیرمنتظره ایجاد کند. برای جلوگیری از این مشکل، توصیه می‌شود همیشه از **true** یا **false** برای مقادیر بولیای در منیفست‌های YAML استفاده کنید و رشته‌هایی که ممکن است با مقادیر بولیای به اشتباه، مانند **"yes"** یا **"no"** نقل قول کنید.

علاوه بر بولیای، تغییرات مشخصات اضافی بین نسخه‌های YAML نیز وجود دارد. لطفاً به
[تغییرات مشخصات YAML](https://spec.yaml.io/main/spec/1.2.2/ext/changes) برای لیست جامعی از آنها مراجعه کنید.
{{< /note >}}

## پادهای "بدون لباس" در مقابل ReplicaSets، Deployments و Jobs {#naked-pods-vs-replicasets-deployments-and-jobs}

- اگر می‌توانید از Pods "بدون لباس" (یعنی Podsی که به یک [ReplicaSet](/docs/concepts/workloads/controllers/replicaset/) یا
  [Deployment](/docs/concepts/workloads/controllers/deployment/) متصل نیستند) استفاده نکنید. پادهای بدون لباس در صورت شکست یک گره، مجدداً زمانبندی نمی‌شوند.

  یک Deployment، که هم یک ReplicaSet را ایجاد می‌کند تا اطمینان حاصل شود که تعداد مورد نظر از Pods همیشه در دسترس است، و هم استراتژی خاصی را برای جایگزینی Pods‌ها مشخص می‌کند (مانند
  [RollingUpdate](/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment))، تقریباً همیشه بهتر از ایجاد مستقیم Pods‌هاست، به جز برخی حالات صریح
  [`restartPolicy: Never`](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy). یک [Job](/docs/concepts/workloads/controllers/job/) نیز ممکن است مناسب باشد.

## خدمات

- یک [Service](/docs/concepts/services-networking/service/) را پیش از پشته‌های کاری پشتیبان آن (Deployments یا ReplicaSets مربوطه) و پیش از هر پشتیبانی که نیاز به دسترسی به آن دارد، ایجاد کنید. زمانی که Kubernetes یک کانتینر را شروع می‌کند، متغیرهای محیطی را به همه

 خدماتی که در زمان شروع کانتینر در حال اجرا بودند، ارائه می‌دهد. به عنوان مثال، اگر یک خدمت با نام `foo` وجود داشته باشد، تمام کانتینرها متغیرهای زیر را در محیط اولیه خود دریافت خواهند کرد:

  ```shell
  FOO_SERVICE_HOST=<میزبانی که خدمت بر روی آن اجرا می‌شود>
  FOO_SERVICE_PORT=<پورتی که خدمت بر روی آن اجرا می‌شود>
  ```

  *این نیاز به ترتیب * - هر خدمت `Service` که `Pod` می‌خواهد دسترسی داشته باشد، باید قبل از خود `Pod` ایجاد شود، وگرنه متغیرهای محیطی پر نخواهد شد.
  DNS این محدودیت را ندارد.

- یک افزونه اختیاری (هرچند به شدت توصیه می‌شود) [افزونه خوشه](/docs/concepts/cluster-administration/addons/) یک سرور DNS است. سرور DNS کلوستر Kubernetes را برای `خدمات` جدید نظارت می‌کند و مجموعه‌ای از رکوردهای DNS برای هر کدام ایجاد می‌کند. اگر DNS در سراسر خوشه فعال شده باشد، تمام کانتینرها باید قادر به رزولوشن نام `خدمات` به صورت خودکار باشند.

- برای یک Pod `hostPort` را مشخص نکنید مگر اینکه مطمئناً ضروری باشد. وقتی یک Pod را به `hostPort` متصل می‌کنید، تعداد مکان‌هایی که می‌تواند بر روی آن زمان‌بندی شود، محدود می‌شود، زیرا هر ترکیب <`hostIP`، `hostPort`، `protocol`> باید منحصر به فرد باشد. اگر `hostIP` و `protocol` را صراحتاً مشخص نکنید، Kubernetes از `0.0.0.0` به عنوان `hostIP` پیش‌فرض و `TCP` به عنوان `protocol` پیش‌فرض استفاده خواهد کرد.

  اگر تنها به دسترسی به پورت Pod برای اهداف اشکال‌زدایی نیاز دارید، می‌توانید از
  [پروکسی apiserver](/docs/tasks/access-application-cluster/access-cluster/#manually-constructing-apiserver-proxy-urls) یا [`kubectl port-forward`](/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)
  استفاده کنید.

  اگر به طور صریح نیاز به افشای پورت یک Pod بر روی گره دارید، در نظر داشته باشید که از
  [NodePort](/docs/concepts/services-networking/service/#type-nodeport) خدمت استفاده کنید، قبل از رسیدگی به
  `hostPort`.

- از استفاده از `hostNetwork` خودداری کنید، به دلایلی که به همان اندازه استفاده از `hostPort` است.

- از [خدمات headless](/docs/concepts/services-networking/service/#headless-services)
  (که دارای `ClusterIP` = `None` هستند) برای کشف خدمات استفاده کنید زمانی که به توزیع بار `kube-proxy`
  نیاز ندارید.

## استفاده از برچسب‌ها

- برچسب‌هایی را تعریف و استفاده کنید که ویژگی‌های معنایی برنامه یا Deployment شما را مشخص کنند، مانند `{ app.kubernetes.io/name:
  MyApp، tier: frontend، phase: test، deployment: v3 }`. شما می‌توانید از این برچسب‌ها برای انتخاب Pods مناسب برای منابع دیگر استفاده کنید؛ به عنوان مثال، یک خدمت که همه Pods `tier: frontend` را انتخاب می‌کند، یا تمام جزئیات `phase: test` از
  `app.kubernetes.io/name: MyApp`.

  یک خدمت می‌تواند با حذف برچسب‌های انتشاری از انتخاب کننده خود را چندین Deployment فراهم کند. زمانی که به روزرسانی یک سرویس در حال اجرا بدون وقفه نیاز دارید، از
  [Deployment](/docs/concepts/workloads/controllers/deployment/) استفاده کنید.

  وضعیت مطلوب یک شی توسط یک Deployment توصیف می‌شود، و اگر تغییرات به آن مشخصات *اعمال* شوند، کنترل‌کننده اجرایی وضعیت واقعی را به وضعیت مطلوب در یک نرخ کنترل‌شده تغییر می‌دهد.

- از [برچسب‌های مشترک Kubernetes](/docs/concepts/overview/working-with-objects/common-labels/)
  برای موارد معمول استفاده کنید. این برچسب‌های استاندارد اطلاعات فراداده را به گونه‌ای غنی می‌کنند که ابزارها، شامل `kubectl` و
  [پیشخوان](/docs/tasks/access-application-cluster/web-ui-dashboard)، به یک شیوه تعاملی کار می‌کنند.

- شما می‌توانید برچسب‌ها را برای اشکال‌زدایی تغییر دهید. زیرا کنترل‌کنندگان Kubernetes (مانند ReplicaSet) و
  Services به کمک برچسب‌های انتخاب کننده Pods مطابقت می‌یابند، حذف برچسب‌های مرتبط را از یک Pod، باعث می‌شود که از طریق کنترل‌کننده خود جدید Pod ایجاد شود. این روش مفیدی برای اشکال‌زدایی یک Pod قبلاً "زنده" در یک محیط "قرنطینه" است. برای حذف یا اضافه کردن برچسب‌ها

 به صورت تعاملی، از [`kubectl label`](/docs/reference/generated/kubectl/kubectl-commands#label) استفاده کنید.

## استفاده از kubectl

- از `kubectl apply -f <directory>` استفاده کنید. این دستور به دنبال پیکربندی Kubernetes در تمام پرونده‌های `.yaml`،
  `.yml` و `.json` در `<directory>` می‌گردد و آنها را به `apply` منتقل می‌کند.

- برچسب‌های انتخاب‌کننده برای عملیات‌های `get` و `delete` به جای نام‌های خاص شی استفاده کنید. بخش‌های [انتخاب‌کننده‌های برچسب](/docs/concepts/overview/working-with-objects/labels/#label-selectors)
  و [استفاده موثر از برچسب‌ها](/docs/concepts/overview/working-with-objects/labels/#using-labels-effectively) را ببینید.

- از `kubectl create deployment` و `kubectl expose` برای سریع‌تر ایجاد و برگون کردن Deployments و
  Services تک‌کانتینر استفاده کنید.
  برای مثال [استفاده از یک خدمت برای دسترسی به یک برنامه در یک خوشه](/docs/tasks/access-application-cluster/service-access-application-cluster/)
