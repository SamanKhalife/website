---
reviewers:
- bsalamat
- k82cn
- ahg-g
title: بسته‌بندی منابع
content_type: مفهوم
weight: 80
---

<!-- خلاصه -->

در پلاگین برنامه‌ریزی [`NodeResourcesFit`](/docs/reference/scheduling/config/#scheduling-plugins) از kube-scheduler، دو استراتژی امتیازدهی وجود دارد که پشتیبانی از بسته‌بندی منابع را انجام می‌دهند: `MostAllocated` و `RequestedToCapacityRatio`.

<!-- محتوا -->

## فعال‌سازی بسته‌بندی منابع با استفاده از استراتژی `MostAllocated`

استراتژی `MostAllocated` بر اساس استفاده از منابع، نمره‌بندی گره‌ها را انجام می‌دهد و گره‌هایی را که دارای تخصیص بیشتری هستند، ترجیح می‌دهد. برای هر نوع منبع، می‌توانید یک وزن تعیین کنید تا تأثیر آن بر نمره گره‌ها تغییر کند.

برای تنظیم استراتژی `MostAllocated` برای پلاگین `NodeResourcesFit`، از یک [پیکربندی برنامه‌ریزی](/docs/reference/scheduling/config) مشابه زیر استفاده کنید:

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- pluginConfig:
  - args:
      scoringStrategy:
        resources:
        - name: cpu
          weight: 1
        - name: memory
          weight: 1
        - name: intel.com/foo
          weight: 3
        - name: intel.com/bar
          weight: 3
        type: MostAllocated
    name: NodeResourcesFit
```

برای آگاهی از سایر پارامترها و پیکربندی‌های پیش‌فرض، به مستندات API برای [`NodeResourcesFitArgs`](/docs/reference/config-api/kube-scheduler-config.v1/#kubescheduler-config-k8s-io-v1-NodeResourcesFitArgs) مراجعه کنید.

## فعال‌سازی بسته‌بندی منابع با استفاده از استراتژی `RequestedToCapacityRatio`

استراتژی `RequestedToCapacityRatio` به کاربران امکان می‌دهد تا منابع را به همراه وزن‌های مخصوص، بر اساس نسبت درخواست به ظرفیت، نمره‌دهی کنند. این امکان را به کاربران می‌دهد تا منابع گسترده را با استفاده از پارامترهای مناسب برای بهبود استفاده از منابع کم در خوشه‌های بزرگ بسته‌بندی کنند. این استراتژی گره‌ها را بر اساس یک تابع پیکربندی شده از منابع تخصیص داده شده، ترجیح می‌دهد. رفتار `RequestedToCapacityRatio` در تابع امتیازدهی `NodeResourcesFit` توسط فیلد [scoringStrategy](/docs/reference/config-api/kube-scheduler-config.v1/#kubescheduler-config-k8s-io-v1-ScoringStrategy) کنترل می‌شود. درون فیلد `scoringStrategy`، می‌توانید دو پارامتر `requestedToCapacityRatio` و `resources` را پیکربندی کنید. شکل در پارامتر `requestedToCapacityRatio` به کاربر امکان می‌دهد تا تابع را بر اساس `utilization` و `score` به عنوان درخواست‌ شده یا پرسش شده بهترین انتظار داشته باشد. فیلد `resources` شامل هم نام منبعی است که در زمان نمره‌دهی در نظر گرفته می‌شود و وزن مربوط به آن را مشخص می‌کند.

در زیر یک پیکربندی نمونه آورده شده است که با استفاده از فیلد `requestedToCapacityRatio` رفتار بسته‌بندی برای منابع گسترده `intel.com/foo` و `intel.com/bar` را تنظیم می‌کند.

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- pluginConfig:
  - args:
      scoringStrategy:
        resources:
        - name: intel.com/foo
          weight: 3
        - name: intel.com/bar
          weight: 3
        requestedToCapacityRatio:
          shape:
          - utilization: 0
            score: 0
          - utilization: 100
            score: 10
        type: RequestedToCapacityRatio
    name: NodeResourcesFit
```

با اشاره به پرونده `KubeSchedulerConfiguration` با پرچم kube-scheduler `--config=/path/to/config/file`، پیکربندی را به برنامه‌ریز منتقل کنید.

برای آگاهی از سایر پارامترها و پیکربندی‌های پیش‌فرض، به مستندات API برای [`NodeResourcesFitArgs`](/docs/reference/config-api/kube-scheduler-config.v1/#kubescheduler-config-k8s-io-v1-NodeResourcesFitArgs) مراجعه کنید.

### تنظیم تابع امتیازدهی

`shape` برای تعیین رفتار تابع `RequestedToCapacityRatio` استفاده می‌شود.

```yaml
shape:
  - utilization: 0
    score: 0
  - utilization: 100
    score: 10
```

پارامترهای فوق، برای گره‌ها امتیاز 0 اختصاص می‌دهد اگر `utilization` برابر 0% باشد و برای `utilization` برابر 100%، امتیاز 10 را اختصاص می‌دهد که بسته‌بندی منابع را فعال می‌کند. برای فعال‌سازی حداقل درخواست‌شده، مقدار امتیاز باید برعکس شود به صورت زیر.

```yaml
shape:
  - utilization: 0
    score: 10
  - utilization: 100
    score: 0
```

`resources` یک پارامتر اختیاری است که به صورت پیش‌فرض به شکل زیر است:

```yaml
resources:
  - name: cpu
    weight: 1
  - name: memory
    weight: 1
```

این می‌تواند برای اضافه کردن منابع گسترده به صورت زیر استفاده شود:

```yaml
resources:
  - name: intel.com/foo
    weight: 5
  - name: cpu
    weight: 3
  - name: memory
    weight: 1
```

پارامتر `weight` اختیاری است و اگر مشخص نشود، به مقدار 1 تنظیم می‌شود. همچنین، نمی‌تواند به مقدار منفی تنظیم شود.

### امتیازدهی گره برای تخصیص ظرفیت

این بخش برای کسانی است که می‌خواهند جزئیات داخلی این ویژگی را بفهمند.
در زیر مثالی از نحوه محاسبه امتیاز گره برای مجموعه مقادیر داده شده آورده شده است.

منابع درخواست شده:

```
intel.com/foo: 2
memory: 256MB
cpu: 2
```

وزن منابع:

```
intel.com/foo: 5
memory: 1
cpu: 3
```

FunctionShapePoint {{0, 0}, {100, 10}}

مشخصات گره 1:

```
در دسترس:
  intel.com/foo: 4
  memory: 1 گیگابایت
  cpu: 8

استفاده شده:
  intel.com/foo: 1
  memory: 256MB
  cpu: 1
```

امتیاز گره:

```
intel.com/foo  = resourceScoringFunction((2+1),4)
               = (100 - ((4-3)*100/4)
               = (100 - 25)
               = 75                       # requested + used = 75% * available
               = rawScoringFunction(75)
               = 7                        # floor(75/10)

memory         = resourceScoringFunction((256+256),1024)
               = (100 -((1024-512)*100/1024))
               = 50                       # requested + used = 50% * available
               = rawScoringFunction(50)
               = 5                        # floor(50/10)

cpu            = resourceScoringFunction((2+1),8)
               = (100 -((8-3)*100/8))
               = 37.5                     # requested + used = 37.5% * available
               = rawScoringFunction(37.5)
               = 3                        # floor(37.5/10)

امتیاز گره   =  ((7 * 5) + (5 * 1) + (3 * 3)) / (5 + 1 + 3)
            =  5
```

مشخصات گره 2:

```
در دسترس:
  intel.com/foo: 8
  memory: 1 گیگابایت
  cpu: 8

استفاده شده:
  intel.com/foo: 2
  memory: 512MB
  cpu: 6
```

امتیاز گره:

```
intel.com/foo  = resourceScoringFunction((2+2),8)
               =  (100 - ((8-4)*100/8)
               =  (100 - 50)
               =  50
               =  rawScoringFunction(50)
               = 5

memory         = resourceScoringFunction((256+512),1024)
               = (100 -((1024-768)*100/1024))
               = 75
               = rawScoringFunction(75)
               = 7

cpu            = resourceScoringFunction((2+6),8)
               = (100 -((8-8)*100/8))
               = 100
               = rawScoringFunction(100)
               = 10

امتیاز گره   =  ((5 * 5) + (7 * 1) + (10 * 3)) / (5 + 1 + 3)
            =  7

```

## {{% heading "whatsnext" %}}

- بیشتر درباره [چارچوب برنامه‌ریزی](/docs/concepts/scheduling-eviction/scheduling-framework/) بخوانید
- بیشتر درباره [پیکربندی برنامه‌ریز](/docs/reference/scheduling/config/) بخوانید
