---
reviewers:
- mikedanese
title: برچسب‌ها و انتخاب‌گرها
content_type: concept
weight: 40
---

<!-- overview -->

برچسب‌ها (_Labels_) جفت‌های کلید/مقدار هستند که به اشیاءی مانند Pods پیوست می‌گیرند. برچسب‌ها برای مشخص کردن ویژگی‌های شناسایی‌ای از اشیاء که معنادار و مرتبط برای کاربران هستند، اما به طور مستقیم معنایی به سیستم اصلی ندارند، استفاده می‌شوند. برچسب‌ها می‌توانند برای سازماندهی و انتخاب زیرمجموعه‌هایی از اشیاء استفاده شوند. برچسب‌ها می‌توانند در زمان ایجاد اشیاء متصل شوند و پس از آن در هر زمان اضافه و تغییر یابند. هر اشیا می‌تواند مجموعه‌ای از برچسب‌های کلید/مقدار را داشته باشد که هر کلید برای یک شیء مشخص باید منحصر به فرد باشد.

```json
"metadata": {
  "labels": {
    "key1" : "value1",
    "key2" : "value2"
  }
}
```

برچسب‌ها امکان پرس و جوی کارآمد و نظارتی را فراهم می‌کنند و برای استفاده در رابط‌های کاربری (UIs) و خطوط فرمان (CLIs) ایده‌آل هستند. اطلاعات غیرشناسایی باید با استفاده از [annotations](/docs/concepts/overview/working-with-objects/annotations/) ثبت شوند.

<!-- body -->

## انگیزه

برچسب‌ها به کاربران اجازه می‌دهند تا ساختارهای سازمانی خود را به اشیاء سیستم به طور گسسته نقش بدهند، بدون اینکه نیاز به ذخیره‌سازی این نقش‌ها توسط مشتریان باشد.

نصب‌های سرویس و خطوط لوله پردازش دسته‌ای اغلب موجودیت‌های چندبعدی هستند (مانند چندین بخش یا نصب‌ها، مسیرهای انتشار متعدد، طبقات متعدد، چندین میکروسرویس در هر طبقه). مدیریت اغلب نیاز به عملیات عرضه می‌کند که شکست استخراج از نمایشگرهای سلسله مراتبی سخت محیط را دور می‌کند، به ویژه سلسله مراتب سفت تعیین شده توسط زیرساخت به جای کاربران.

نمونه‌های برچسب:

* `"release" : "stable"`, `"release" : "canary"`
* `"environment" : "dev"`, `"environment" : "qa"`, `"environment" : "production"`
* `"tier" : "frontend"`, `"tier" : "backend"`, `"tier" : "cache"`
* `"partition" : "customerA"`, `"partition" : "customerB"`
* `"track" : "daily"`, `"track" : "weekly"`

این نمونه‌ها از [برچسب‌های متداول استفاده‌شده](/docs/concepts/overview/working-with-objects/common-labels/) هستند؛ شما آزادید که مفاد قواعد خود را توسعه دهید. به خاطر داشته باشید که کلید برچسب برای یک شیء مشخص باید منحصر به فرد باشد.

## دستورالعمل و مجموعه کاراکترها

برچسب‌ها جفت‌های کلید/مقدار هستند. کلیدهای برچسب معتبر دارای دو بخش هستند: پیشوند اختیاری و نام، جداشده توسط یک خط کج (`/`). بخش نام لازم است و باید حداکثر 63 نویسه باشد، که با یک نویسه حروف الفبایی عددی (`[a-z0-9A-Z]`) آغاز و پایان می‌یابد و با خط‌ها (`-`)، زیرخط‌ها (`_`)، نقاط (`.`) و عددی-حرفی‌ها بین آنها است. پیشوند اختیاری است. اگر مشخص شود، پیشوند باید زیرمجموعه‌ای از دامنه فرعی DNS باشد: یک سری برچسب DNS که توسط نقاط (`.`) جدا شده‌اند، کل نهایی باید کمتر از 253 نویسه باشد و به دنبال آن یک خط کج (`/`) وجود دارد.

اگر پیشوند حذف شود، کلید برچسب در نظر گرفته می‌شود که خصوصی به کاربر باشد. اجزاء سیستمی خودکار (مانند `kube-scheduler`، `kube-controller-manager`، `kube-apiserver`، `kubectl` یا سیستم‌های خودکار شخص ثالث دیگر) که برچسب‌ها را به اشیاء کاربر نهایی اضافه می‌کنند، باید یک پیشوند مشخص کنند.

پیشوند `kubernetes.io/` و `k8s.io/` [رزرو شده‌اند](/docs/reference/labels-annotations-taints/) برای اجزاء هسته Kubernetes.

مقدار برچسب معتبر:

* باید حداکثر 63 نویسه باشد (می‌تواند خالی باشد).
* مگر اینکه خالی باشد، باید با یک نویسه حروف الفبایی عددی (`[a-z0-9A-Z]`) آغاز و پایان یابد.
* می‌تواند شامل خط‌ها (`-`)، زیرخط‌ها (`_`)، نقاط (`.`) و عددی-حرفی‌ها باشد.

به عنوان مثال، در اینجا یک ن

مایشنامه برای یک Pod است که دارای دو برچسب `environment: production` و `app: nginx` است:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
    environment: production
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

## انتخاب‌گرهای برچسب

برخلاف [نام‌ها و شناسه‌ها](/docs/concepts/overview/working-with-objects/names/)، برچسب‌ها اختصاصیت را فراهم نمی‌کنند. به طور کلی، انتظار می‌رود که اشیاء زیادی همراه با همان برچسب(ها) حمل شوند.

با استفاده از _انتخاب‌گر برچسب_، مشتری/کاربر می‌تواند یک مجموعه از اشیاء را شناسایی کند. انتخاب‌گر برچسب پیش‌فرض گروه‌بندی اصلی در Kubernetes است.

API در حال حاضر دو نوع انتخاب‌گره را پشتیبانی می‌کند: مبتنی بر _برابری_ و مبتنی بر _مجموعه_. یک انتخاب‌گر برچسب می‌تواند از چندین _نیازمندی_ که با ویرگول جدا شده‌اند، تشکیل شود. در صورت وجود نیازمندی‌های چندگانه، همه باید برآورده شوند، بنابراین جداکننده کاما به عنوان _AND_ منطقی (`&&`) عمل می‌کند.

معنی انتخاب‌گرهای خالی یا غیرمشخص به سیاق و نوع API که از انتخاب‌گرها استفاده می‌کنند، وابسته است و باید اعتبار و معنای آنها را مستند کند.

{{< note >}}
برای برخی از انواع API مانند ReplicaSets، انتخاب‌گرهای برچسب دو نمونه نباید در یک فضای نام همپوشانی داشته باشند، یا کنترل‌کننده ممکن است آن را به عنوان دستورالعمل‌های متضاد دیده و ناتوان در تعیین اینکه چند نسخه مجازی باید حاضر باشند.
{{< /note >}}

{{< caution >}}
برای هر دو شرایط مبتنی بر برابری و مبتنی بر مجموعه عملگر منطقی _OR_ (`||`) وجود ندارد. مطمئن شوید که عبارات فیلتر شما به درستی ساختاردهی شده‌اند.
{{< /caution >}}

### _Equality-based_ requirement

نیازمندی‌های _برابری_ یا _نابرابری_ به فیلتر کردن بر اساس کلیدها و مقادیر برچسب اجازه می‌دهند. اشیاء مطابقت باید تمام محدودیت‌های برچسب مشخص شده را برآورده کنند، اگرچه ممکن است برچسب‌های اضافی هم داشته باشند. سه نوع عملگر مجاز هستند `=`,`==`,`!=`. دو عملگر اول معادل با _برابری_ هستند (و مترادفند)، در حالی که عملگر آخر معادل _نابرابری_ است. به عنوان مثال:

```
environment = production
tier != frontend
```

اولیه همه منابع با کلید برابر `environment` و مقدار برابر `production` را انتخاب می‌کند. دومی همه منابع با کلید برابر `tier` و مقدار متفاوت از `frontend` را انتخاب می‌کند، همچنین همه منابع بدون برچسب با کلید `tier` را. می‌توان با استفاده از عملگر کاما برای منابع در `production` به استثنای `frontend` فیلتر کرد: `environment=production,tier!=frontend`

یکی از موارد استفاده از نیازمندی بر اساس برچسب برابری برای Pods مشخص کردن معیارهای انتخاب راس‌ها است. به عنوان مثال، Pod نمونه زیر راس‌ها را با برچسب "`accelerator=nvidia-tesla-p100`" انتخاب می‌کند.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "registry.k8s.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1
  nodeSelector:
    accelerator: nvidia-tesla-p100
```

### _Set-based_ requirement

نیازمندی‌های برچسب مبتنی بر مجموعه اجازه می‌دهند کلیدها را بر اساس یک مجموعه از مقادیر فیلتر کنند. سه نوع عملگر پشتیبانی می‌شوند: `in`، `notin` و `exists` (تنها شناسه کلید). به عنوان مثال:

```
environment in (production, qa)
tier notin (frontend, backend)
partition
!partition
```

- نمونه اول همه منابع با کلید برابر `environment` و مقدار برابر با `production` یا `qa` را انتخاب می‌کند.
- نمونه دوم همه منابع با کلید برابر `tier` و مقادیر به جز `frontend` و `backend` را انتخاب می‌کند، همچنین همه منابع بدون برچسب با کلید `tier` را.
- نمونه سوم همه منابع شامل یک برچسب با کلید `partition` را انتخاب می‌کند؛ هیچ مقداری بررسی نمی‌شود.
- نمونه چهارم همه منابع بدون برچسب با کلید `partition` را انتخاب می‌کند؛ هیچ مقداری بررسی نمی‌شود.

به طور مشابه، جداکننده کاما به عنوان عملگر _AND_ عمل می‌کند. بنابراین، فیلتر کردن منابع با کلید `partition` (نه مهم که مقدار) و با `environment` متفاوت از `qa` با استفاده از `partition,environment notin (qa)` امکان‌پذیر است. انتخاب‌گر برچسب مبتنی بر مجموعه یک شکل کلی از برابری است، زیرا `environment=production` معادل `environment in (production)` است؛ به همین ترتیب برای `!=` و `notin`.

نیازمندی‌های برچسب مبتنی بر مجموعه می‌توانند با نیازمندی‌های برابری مخلوط شوند. به عنوان مثال: `partition in (customerA, customerB),environment!=qa`.

## API

### فیلتر کردن LIST و WATCH

عملیات LIST و WATCH می‌توانند با استفاده از یک انتخاب‌گر برچسب برای فیلتر کردن مجموعه‌های اشیاء با استفاده از یک پارامتر پرس و جو مشخص شوند. هر دو نیازمندی اجازه داده شده است (که در اینجا به عنوان یک رشته پرس و جو URL نشان داده شده‌اند):

* نیازمندی‌های _برابری_ : `?labelSelector=environment%3Dproduction,tier%3Dfrontend`
* نیازمندی‌های _برچسب مبتنی بر مجموعه_ : `?labelSelector=environment+in+%28production%2Cqa%29%2Ctier+in+%28frontend%29`

هر دو نوع استایل انتخاب‌گر برچسب می‌تواند برای لیست یا نظارت بر منابع از طریق یک مشتری REST استفاده شود. به عنوان مثال، با استفاده از انتخاب‌گر _برابری_ می‌توانید به `apiserver` متصل شوید و از نیازمندی‌های برچسب استفاده کنید:

```shell
kubectl get pods -l environment=production,tier=frontend
```

یا با استفاده از نیازمندی‌های _برچسب مبتنی بر مجموعه_:

```shell
kubectl get pods -l 'environment in (production),tier in (frontend)'
```

همانطور که قبلاً اشاره شده، نیازمندی‌های برچسب مبتنی بر مجموعه بیانگر هستند. به عنوان مثال، آنها می‌توانند اپراتور _OR_ را در مقادیر پیاده کنند:

```shell
kubectl get pods -l 'environment in (production, qa)'
```

#### منابع پشتیبانی‌کننده از نیازمندی‌های برچسب مبتنی بر مجموعه

منابع جدیدی مانند [`Job`](/docs/concepts/workloads/controllers/job/),
[`Deployment`](/docs/concepts/workloads/controllers/deployment/),
[`ReplicaSet`](/docs/concepts/workloads/controllers/replicaset/) و
[`DaemonSet`](/docs/concepts/workloads/controllers/daemonset/)
همچنین از نیازمندی‌های برچسب مبتنی بر مجموعه پشتیبانی می‌کنند.

```yaml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - { key: tier, operator: In, values: [cache] }
    - { key: environment, operator: NotIn, values: [dev] }
```

`matchLabels` یک نگاشت از جفت `{key,value}` است. یک `{key,value}` تنها در نگاشت `matchLabels` معادل یک عنصر از `matchExpressions` است، که فیلد `key` آن "key"، `operator` آن "In" و آرایه `values` آن فقط "value" است. `matchExpressions` یک لیست از نیازمندی‌های انتخابگر Pod است. اپراتورهای معتبر شامل In، NotIn، Exists و DoesNotExist هستند. مجموعه مقادیر باید غیر خالی باشد در مورد In و NotIn. تمام نیازمندی‌ها، هم از `matchLabels` و هم از `matchExpressions` با هم AND شده‌اند -- همه باید برآورده شوند تا منطبق شوند.

#### انتخاب مجموعه‌های راس‌ها

یکی از موارد استفاده از انتخاب بر اساس برچسب‌ها، محدود کردن مجموعه راس‌هایی است که یک Pod می‌تواند در آنها برنامه‌ریزی شود. برای کسب اطلاعات بیشتر، به مستندات در مورد [انتخاب راس](/docs/concepts/scheduling-eviction/assign-pod-node/) مراجعه کنید.

## استفاده موثر از برچسب‌ها

می‌توانید یک برچسب تنها را به هر منبع اعمال کنید، اما این همیشه بهترین شیوه نیست. در موارد زیادی، باید از چندین برچسب برای تمایز مجموعه‌های منابع از یکدیگر استفاده شود.

به عنوان مثال، برنامه‌های مختلف از مقادیر مختلف برای برچسب `app` استفاده می‌کنند، اما یک برنامه چندمرحله‌ای مانند [مثال guestbook](https://github.com/kubernetes/examples/tree/master/guestbook/) نیاز به تمایز هر مرحله دارد. Frontend می‌تواند دارای برچسب‌های زیر باشد:

```yaml
labels:
  app: guestbook
  tier: frontend
```

در حالی که Redis master و replica ممکن است برچسب‌های مختلف `tier` داشته باشند، و شاید حتی یک برچسب `role` اضافی:

```yaml
labels:
  app: guestbook
  tier: backend
  role: master
```

و

```yaml
labels:
  app: guestbook
  tier: backend
  role: replica
```

برچسب‌ها امکان شکاف‌زنی منابع را در هر ابعاد مشخص شده توسط یک برچسب فراهم می‌کنند:

```shell
kubectl apply -f examples/guestbook/all-in-one/guestbook-all-in-one.yaml
kubectl get pods -Lapp -Ltier -Lrole
```

```none
NAME                           READY  STATUS    RESTARTS   AGE   APP         TIER       ROLE
guestbook-fe-4nlpb             1/1    Running   0          1m    guestbook   frontend   <none>
guestbook-fe-ght6d             1/1    Running   0          1m    guestbook   frontend   <none>
guestbook-fe-jpy62             1/1    Running   0          1m    guestbook   frontend   <none>
guestbook-redis-master-5pg3b   1/1    Running   0          1m    guestbook   backend    master
guestbook-redis-replica-2q2yf  1/1    Running   0          1m    guestbook   backend    replica
guestbook-redis-replica-qgazl  1/1    Running   0          1m    guestbook   backend    replica
my-nginx-divi2                 1/1    Running   0          29m   nginx       <none>     <none>
my-nginx-o0ef1                 1/1    Running   0          29m   nginx       <none>     <none>
```

```shell
kubectl get pods -lapp=guestbook,role=replica
```

```none
NAME                           READY  STATUS   RESTARTS  AGE
guestbook-redis-replica-2q2yf  1/1    Running  0         3m
guestbook-redis-replica-qgazl  1/1    Running  0         3m
```

## به‌روزرسانی برچسب‌ها

گاهی اوقات ممکن است بخواهید برچسب‌های موجودیت‌ها و پادها را قبل از ایجاد موجودیت‌های جدید مجدداً برچسب‌گذاری کنید. این کار با `kubectl label` قابل انجام است. به عنوان مثال، اگر می‌خواهید همه پادهای NGINX خود را به عنوان Tier فرانت‌اند برچسب‌گذاری کنید، اجرا کنید:

```shell
kubectl label pods -l app=nginx tier=fe
```

```none
pod/my-nginx-2035384211-j5fhi labeled
pod/my-nginx-2035384211-u2c7e labeled
pod/my-nginx-2035384211-u3t6x labeled
```

این ابتدا تمام پادهای با برچسب "app=nginx" را فیلتر می‌کند و سپس آنها را با برچسب "tier=fe" برچسب‌گذاری می‌کند. برای مشاهده پادهای برچسب‌گذاری‌شده، اجرا کنید:

```shell
kubectl get pods -l app=nginx -L tier
```

```none
NAME                        READY     STATUS    RESTARTS   AGE       TIER
my-nginx-2035384211-j5fhi   1/1       Running   0          23m       fe
my-nginx-2035384211-u2c7e   1/1       Running   0          23m       fe
my-nginx-2035384211-u3t6

x   1/1       Running   0          23m       fe
```

این خروجی همه پادهای "app=nginx" را نمایش می‌دهد، با یک ستون برچسب اضافی از Tier پاده‌ها
(مشخص شده با `-L` یا `--label-columns`).

برای کسب اطلاعات بیشتر، لطفاً به [kubectl label](/docs/reference/generated/kubectl/kubectl-commands/#label) مراجعه کنید.

## {{% heading "whatsnext" %}}

- یاد بگیرید که چگونه [برچسب به یک راس اضافه کنید](/docs/tasks/configure-pod-container/assign-pods-nodes/#add-a-label-to-a-node)
- برچسب‌ها، انواع [معروف، حاشیه‌نویسی و تفریق](/docs/reference/labels-annotations-taints/)
- مشاهده [برچسب‌های توصیه‌شده](/docs/concepts/overview/working-with-objects/common-labels/)
- [اجرای استانداردهای امنیتی پاد با برچسب‌های فضا](/docs/tasks/configure-pod-container/enforce-standards-namespace-labels/)
- خواندن وبلاگ در مورد [نوشتن یک کنترل‌کننده برای برچسب‌های پاد](/blog/2021/06/21/writing-a-controller-for-pod-labels/)

