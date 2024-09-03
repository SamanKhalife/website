```markdown
---
reviewers:
- fgrzadkowski
- jszczepkowski
- justinsb
- directxman12
title: مستند HorizontalPodAutoscaler
content_type: task
weight: 100
min-kubernetes-server-version: 1.23
---

<!-- overview -->

یک [HorizontalPodAutoscaler](/docs/tasks/run-application/horizontal-pod-autoscale/)
(مخفف HPA)
به طور خودکار می‌تواند منابع یک منبع کار (مانند
{{< glossary_tooltip text="Deployment" term_id="deployment" >}} یا
{{< glossary_tooltip text="StatefulSet" term_id="statefulset" >}})
را بروزرسانی کند با هدف مقیاس‌پذیری خودکار منابع برای مطابقت با تقاضا.

مقیاس افقی به معنای پاسخ به بار افزایش یافته برای اجرای بیشتر
{{< glossary_tooltip text="Pods" term_id="pod" >}}
است. این متفاوت از مقیاس
_عمودی_
است که برای Kubernetes می‌تواند به معنای اختصاص منابع (مثلاً حافظه یا پردازنده) به Pods هایی که در حال حاضر برای منبع کار در حال اجرا هستند، باشد.

اگر بار کاهش یابد و تعداد Pods بالاتر از حداقل تنظیم شده باشد، HorizontalPodAutoscaler
منابع کار را (Deployment، StatefulSet، یا منبع مشابه دیگر) را به مقیاس
کاهش می‌دهد.

این مستند شما را از طریق یک مثال از فعال‌سازی HorizontalPodAutoscaler برای
مدیریت خودکار مقیاس یک برنامه وب نمایش می‌دهد. این بار نمونه‌کار Apache
httpd با اجرای کد PHP است.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}} اگر شما از یک استقرار قدیمی‌تر Kubernetes
دوباره شده، به مدارک تاریخ مراجعه کنید (منظور داشته تا
[نسخه‌های مستندات موجود](/docs/home/supported-doc-versions/)).

برای پیروی از این آموزش، شما نیاز دارید تا از یک خوشه استفاده کنید که دارای
[Metrics Server](https://github.com/kubernetes-sigs/metrics-server#readme) استقرار یافته و پیکربندی شده باشد.
Metrics Server Kubernetes می‌سیرد منابع می‌یابد که توسط توسل های معلول می‌کند، و تمام این سنگ
از طریق [Kubernetes API](/docs/concepts/overview/kubernetes-api/)
می‌کند، با استفاده از [APIService](/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/)
برای اضافه کردن نوع جدید یک نوع منابعی که نشان می‌دهد.

برای یادگیری از نحوه استقرار Metrics Server، نگاه به
[مستندات می‌سیر سرور](https://github.com/kubernetes-sigs/metrics-server#deployment).

اگر شما از {{< glossary_tooltip term_id="minikube" >}} استفاده می‌کنید، از گزینهٔ زیر برای فعال کردن metrics-server استفاده کنید:

```shell
minikube addons enable metrics-server
```

<!-- steps -->

## اجرا و افشا سرور php-apache

برای نمایش HorizontalPodAutoscaler، اولین چیزی که باید انجام دهید است استارت
یک Deployment است که یک ظرف به کاربر مشابه به صورت پیدا کنجا،
## افزایش بار {#increase-load}

در مرحله بعد، ببینید که چگونه autoscaler به افزایش بار واکنش نشان می‌دهد.
برای انجام این کار، یک Pod دیگر را به عنوان یک کلاینت شروع خواهید کرد. کانتینر درون کلاینت Pod
در یک حلقه بی‌نهایت اجرا می‌شود و به سرویس php-apache درخواست‌هایی ارسال می‌کند.

```shell
# این فرمان را در یک ترمینال جداگانه اجرا کنید
# تا تولید بار ادامه یابد و شما بتوانید با بقیه مراحل پیش بروید
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```

اکنون اجرا کنید:
```shell
# برای پایان دادن به watch کلید Ctrl+C را بزنید
kubectl get hpa php-apache --watch
```

در عرض یک دقیقه یا بیشتر، باید بار CPU بالاتری مشاهده کنید؛ به عنوان مثال:

```
NAME         REFERENCE                     TARGET      MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache/scale   305% / 50%  1         10        1          3m
```

و سپس، تعداد بیشتری از replicaها. برای مثال:
```
NAME         REFERENCE                     TARGET      MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache/scale   305% / 50%  1         10        7          3m
```

اینجا، مصرف CPU به 305% از درخواست رسیده است.
در نتیجه، Deployment به 7 replica تنظیم شده است:

```shell
kubectl get deployment php-apache
```

باید تعداد replica‌ها را ببینید که با شکل HorizontalPodAutoscaler مطابقت دارد
```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
php-apache   7/7      7           7           19m
```

{{< note >}}
ممکن است چند دقیقه طول بکشد تا تعداد replicaها پایدار شود. از آنجایی که مقدار بار به هیچ وجه کنترل نمی‌شود، ممکن است تعداد نهایی replicaها با این مثال متفاوت باشد.
{{< /note >}}

## توقف تولید بار {#stop-load}

برای اتمام مثال، ارسال بار را متوقف کنید.

در ترمینالی که Podی که تصویر `busybox` را اجرا می‌کند ایجاد کردید، تولید بار را با تایپ `<Ctrl> + C` متوقف کنید.

سپس وضعیت نتیجه را بررسی کنید (پس از یک دقیقه یا بیشتر):

```shell
# برای پایان دادن به watch کلید Ctrl+C را بزنید
kubectl get hpa php-apache --watch
```

خروجی مشابه زیر است:

```
NAME         REFERENCE                     TARGET       MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache/scale   0% / 50%     1         10        1          11m
```

و Deployment نیز نشان می‌دهد که مقیاس کاهش یافته است:

```shell
kubectl get deployment php-apache
```

```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
php-apache   1/1     1            1           27m
```

زمانی که استفاده از CPU به 0 کاهش یافت، HPA به طور خودکار تعداد replicaها را به 1 کاهش داد.

مقیاس‌بندی replicaها ممکن است چند دقیقه طول بکشد.

<!-- discussion -->

## مقیاس‌بندی خودکار بر اساس معیارهای متعدد و معیارهای سفارشی

می‌توانید معیارهای اضافی را برای استفاده هنگام مقیاس‌بندی Deployment `php-apache` معرفی کنید
با استفاده از نسخه API `autoscaling/v2`.

ابتدا، YAML مربوط به HorizontalPodAutoscaler خود را در قالب `autoscaling/v2` بگیرید:

```shell
kubectl get hpa php-apache -o yaml > /tmp/hpa-v2.yaml
```

فایل `/tmp/hpa-v2.yaml` را در یک ویرایشگر باز کنید، و باید YAML را مشاهده کنید که به این شکل است:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
status:
  observedGeneration: 1
  lastScaleTime: <some-time>
  currentReplicas: 1
  desiredReplicas: 1
  currentMetrics:
  - type: Resource
    resource:
      name: cpu
      current:
        averageUtilization: 0
        averageValue: 0
```

توجه داشته باشید که فیلد `targetCPUUtilizationPercentage` با یک آرایه به نام `metrics` جایگزین شده است.
معیار مصرف CPU یک *معیار منابع* است، زیرا به عنوان درصدی از منابع
مشخص شده در کانتینرهای pod نشان داده شده است. توجه داشته باشید که می‌توانید معیارهای منابع دیگری به جز CPU مشخص کنید. به طور پیش‌فرض،
تنها معیار منابع دیگری که پشتیبانی می‌شود حافظه است. این منابع از خوشه‌ای به خوشه دیگر تغییر نمی‌کنند،
و باید همیشه در دسترس باشند، به شرطی که API `metrics.k8s.io` در دسترس باشد.

همچنین می‌توانید معیارهای منابع را به صورت مقادیر مستقیم، به جای به عنوان درصدی از مقدار
درخواست شده، با استفاده از `target.type` به عنوان `AverageValue` به جای `Utilization`، و
تنظیم فیلد مربوطه `target.averageValue` به جای `target.averageUtilization` مشخص کنید.

دو نوع معیار دیگر وجود دارد که هر دو به عنوان *معیارهای سفارشی* در نظر گرفته می‌شوند: معیارهای pod و
معیارهای شیء. این معیارها ممکن است نام‌هایی داشته باشند که به خوشه خاصی اختصاص دارند، و نیاز به یک
راه‌اندازی پیشرفته‌تر برای مانیتورینگ خوشه دارند.

اولین نوع از این معیارهای جایگزین *معیارهای pod* است. این معیارها pods را توصیف می‌کنند، و
به طور متوسط بین pods تقسیم می‌شوند و با یک مقدار هدف مقایسه می‌شوند تا تعداد replicaها تعیین شود.
آنها بسیار شبیه معیارهای منابع کار می‌کنند، به جز اینکه *فقط* از یک نوع `target` به نام `AverageValue` پشتیبانی می‌کنند.
معیارهای pod با استفاده از یک بلوک معیار مانند این مشخص می‌شوند:

```yaml
type: Pods
pods:
  metric:
    name: packets-per-second
  target:
    type: AverageValue
    averageValue: 1k
```

نوع دوم معیار جایگزین *معیارهای شیء* است. این معیارها یک شیء متفاوت در همان namespace را توصیف می‌کنند،
به جای توصیف pods. معیارها لزوماً از شیء گرفته نمی‌شوند؛ آنها فقط آن را توصیف می‌کنند. معیارهای شیء از
انواع `target` هم `Value` و هم `AverageValue` پشتیبانی می‌کنند. با `Value`، هدف به طور مستقیم با معیار برگشتی از API مقایسه می‌شود. با `AverageValue`، مقدار برگشتی از API معیارهای سفارشی قبل از مقایسه با هدف به تعداد pods تقسیم می‌شود. مثال زیر نمایشی از YAML مربوط به معیار `requests-per-second` است.

```yaml
type: Object
object:
  metric:
    name: requests-per-second
  describedObject:
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    name: main-route
  target:
    type: Value
    value: 2k
```
## مقیاس‌بندی خودکار بر اساس معیارهای متعدد

اگر چندین بلوک معیار مشابه ارائه دهید، HorizontalPodAutoscaler هر معیار را به نوبت در نظر می‌گیرد.
HorizontalPodAutoscaler تعداد پیشنهادی replicaها را برای هر معیار محاسبه کرده و سپس
بالاترین تعداد replica را انتخاب می‌کند.

برای مثال، اگر سیستم مانیتورینگ شما معیارهایی در مورد ترافیک شبکه جمع‌آوری کند،
می‌توانید تعریف بالا را با استفاده از `kubectl edit` به صورت زیر به‌روزرسانی کنید:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Pods
    pods:
      metric:
        name: packets-per-second
      target:
        type: AverageValue
        averageValue: 1k
  - type: Object
    object:
      metric:
        name: requests-per-second
      describedObject:
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        name: main-route
      target:
        type: Value
        value: 10k
status:
  observedGeneration: 1
  lastScaleTime: <some-time>
  currentReplicas: 1
  desiredReplicas: 1
  currentMetrics:
  - type: Resource
    resource:
      name: cpu
    current:
      averageUtilization: 0
      averageValue: 0
  - type: Object
    object:
      metric:
        name: requests-per-second
      describedObject:
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        name: main-route
      current:
        value: 10k
```

سپس، HorizontalPodAutoscaler شما تلاش می‌کند تا اطمینان حاصل کند که هر pod تقریباً
50% از CPU درخواست شده را مصرف می‌کند، 1000 packet در ثانیه را سرویس می‌دهد، و همه pods پشت
Ingress مسیر اصلی 10000 درخواست در ثانیه را سرویس می‌دهند.

### مقیاس‌بندی خودکار بر اساس معیارهای خاص‌تر

بسیاری از سیستم‌های مانیتورینگ اجازه می‌دهند که معیارها را یا با نام یا با مجموعه‌ای از
توصیف‌کننده‌های اضافی به نام _labels_ توصیف کنید. برای همه انواع معیارهای غیرمنابعی (pod، شیء، و خارجی،
که در ادامه توضیح داده شده است)، می‌توانید یک انتخابگر برچسب اضافی که به سیستم معیار شما
پاس داده می‌شود را مشخص کنید. به عنوان مثال، اگر معیاری به نام `http_requests` با برچسب `verb`
جمع‌آوری کنید، می‌توانید بلوک معیار زیر را برای مقیاس‌بندی فقط بر اساس درخواست‌های GET مشخص کنید:

```yaml
type: Object
object:
  metric:
    name: http_requests
    selector: {matchLabels: {verb: GET}}
```

این انتخابگر از همان نحوی استفاده می‌کند که انتخابگرهای برچسب کامل Kubernetes. سیستم مانیتورینگ
تعیین می‌کند که چگونه چندین سری را به یک مقدار تبدیل کند، اگر نام و انتخابگر
با چندین سری مطابقت داشته باشند. انتخابگر افزایشی است و نمی‌تواند معیارهایی را
که اشیاء هدف نیستند را انتخاب کند (هدف pods در نوع `Pods`
و شیء توصیف‌شده در نوع `Object`).

### مقیاس‌بندی خودکار بر اساس معیارهای نامرتبط با اشیاء Kubernetes

برنامه‌های در حال اجرا بر روی Kubernetes ممکن است نیاز به مقیاس‌بندی خودکار بر اساس معیارهایی داشته باشند که ارتباط
واضحی با هیچ شیئی در خوشه Kubernetes ندارند، مانند معیارهایی که یک سرویس میزبان را توصیف می‌کنند که
هیچ ارتباط مستقیمی با namespace‌های Kubernetes ندارد. در Kubernetes 1.10 و نسخه‌های بعدی، می‌توانید این مورد استفاده را
با *معیارهای خارجی* برطرف کنید.

استفاده از معیارهای خارجی نیاز به دانش سیستم مانیتورینگ شما دارد؛ راه‌اندازی مشابه با
استفاده از معیارهای سفارشی است. معیارهای خارجی به شما اجازه می‌دهند که خوشه خود را بر اساس هر معیاری که در سیستم مانیتورینگ شما موجود است مقیاس‌بندی کنید. یک بلوک `metric` با یک
`name` و `selector` ارائه دهید، همانطور که در بالا توضیح داده شد، و از نوع معیار `External`
به جای `Object` استفاده کنید.
اگر چندین سری زمانی توسط `metricSelector` مطابقت داده شود،
مجموع مقادیر آنها توسط HorizontalPodAutoscaler استفاده می‌شود.
معیارهای خارجی از هر دو نوع هدف `Value` و `AverageValue` پشتیبانی می‌کنند، که دقیقاً به همان صورت که در نوع `Object`
است عمل می‌کنند.

برای مثال، اگر برنامه شما کارهایی را از یک سرویس صف میزبان پردازش کند، می‌توانید بخش زیر را
به manifest HorizontalPodAutoscaler خود اضافه کنید تا مشخص کنید که شما به یک کارگر برای هر 30 کار معوقه نیاز دارید.

```yaml
- type: External
  external:
    metric:
      name: queue_messages_ready
      selector:
        matchLabels:
          queue: "worker_tasks"
    target:
      type: AverageValue
      averageValue: 30
```

در صورت امکان، ترجیح داده می‌شود که از نوع هدف معیارهای سفارشی به جای معیارهای خارجی استفاده کنید، زیرا
برای مدیران خوشه آسان‌تر است که API معیارهای سفارشی را ایمن کنند. API معیارهای خارجی به طور بالقوه اجازه
دسترسی به هر معیاری را می‌دهد، بنابراین مدیران خوشه باید هنگام افشای آن دقت کنند.

## ضمیمه: شرایط وضعیت Horizontal Pod Autoscaler

هنگام استفاده از فرم `autoscaling/v2` از HorizontalPodAutoscaler، قادر خواهید بود
*شرایط وضعیت* را که توسط Kubernetes بر روی HorizontalPodAutoscaler تنظیم شده‌اند مشاهده کنید. این شرایط وضعیت نشان می‌دهند
که آیا HorizontalPodAutoscaler قادر به مقیاس‌بندی است یا خیر، و آیا در حال حاضر به هر نحوی محدود شده است یا خیر.

این شرایط در فیلد `status.conditions` ظاهر می‌شوند. برای دیدن شرایطی که بر HorizontalPodAutoscaler تأثیر می‌گذارند،
می‌توانیم از `kubectl describe hpa` استفاده کنیم:

```shell
kubectl describe hpa cm-test
```

```
Name:                           cm-test
Namespace:                      prom
Labels:                         <none>
Annotations:                    <none>
CreationTimestamp:              Fri, 16 Jun 2017 18:09:22 +0000
Reference:                      ReplicationController/cm-test
Metrics:                        ( current / target )
  "http_requests" on pods:      66m / 500m
Min replicas:                   1
Max replicas:                   4
ReplicationController pods:     1 current / 1 desired
Conditions:
  Type                  Status  Reason                  Message
  ----                  ------  ------                  -------
  AbleToScale           True    ReadyForNewScale        the last scale time was sufficiently old as to warrant a new scale
  ScalingActive         True    ValidMetricFound        the HPA was able to successfully calculate a replica count from pods metric http_requests
  ScalingLimited        False   DesiredWithinRange      the desired replica count is within the acceptable range
Events:
```

برای این HorizontalPodAutoscaler، می‌توانید چندین شرایط در حالت سالم ببینید. اولی،
`AbleToScale`، نشان می‌دهد که آیا HPA قادر به دریافت و به‌روزرسانی مقیاس‌ها است یا خیر و
آیا هر گونه شرایط مرتبط با backoff مقیاس‌بندی را جلوگیری می‌کنند یا خیر. دومی، `ScalingActive`،
نشان می‌دهد که آیا HPA فعال است (یعنی تعداد replica‌های هدف صفر نیست) و
قادر به محاسبه مقیاس‌های مورد نظر است. زمانی که این مقدار `False` باشد، به طور کلی مشکلاتی در
دریافت معیارها را نشان می‌دهد. در نهایت، آخرین شرط، `ScalingLimited`، نشان می‌دهد که مقیاس مورد نظر
توسط حداقل یا حداکثر HorizontalPodAutoscaler محدود شده است. این یک نشانگر است که
ممکن است بخواهید حداقل یا حداکثر تعداد replicaها را بر روی HorizontalPodAutoscaler خود افزایش یا کاهش دهید.

## مقادیر

همه معیارها در HorizontalPodAutoscaler و APIهای معیارها با استفاده از
یک نشانه‌گذاری عددی ویژه که در Kubernetes به عنوان
{{< glossary_tooltip term_id="quantity" text="quantity">}} شناخته می‌شود، مشخص شده‌اند. برای مثال،
مقدار `10500m` به صورت `10.5` در نشانه‌گذاری اعشاری نوشته می‌شود. APIهای معیارها
اعداد صحیح را بدون پسوند زمانی که ممکن است باز می‌گردانند، و به طور کلی
مقادیر را به واحدهای میلی‌واحد برمی‌گردانند. این به این معنی است که ممکن
## سناریوهای ممکن دیگر

### ایجاد مقیاس‌بند خودکار به صورت اعلامی

به جای استفاده از دستور `kubectl autoscale` برای ایجاد یک HorizontalPodAutoscaler به صورت دستوری، می‌توانیم از manifest زیر برای ایجاد آن به صورت اعلامی استفاده کنیم:

{{% code_sample file="application/hpa/php-apache.yaml" %}}

سپس، با اجرای دستور زیر مقیاس‌بند خودکار را ایجاد کنید:

```shell
kubectl create -f https://k8s.io/examples/application/hpa/php-apache.yaml
```

```
horizontalpodautoscaler.autoscaling/php-apache created
```