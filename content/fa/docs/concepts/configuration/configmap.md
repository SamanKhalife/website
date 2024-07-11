---
title: "ConfigMaps"
api_metadata:
- apiVersion: "v1"
  kind: "ConfigMap"
content_type: concept
weight: 20
---

<!-- overview -->

{{< glossary_definition term_id="configmap" prepend="یک ConfigMap" length="all" >}}

{{< caution >}}
ConfigMap امنیت یا رمزگذاری را فراهم نمی‌کند.
اگر داده‌هایی که می‌خواهید ذخیره کنید محرمانه هستند، از
{{< glossary_tooltip text="Secret" term_id="secret" >}} به جای یک ConfigMap استفاده کنید
یا از ابزارهای اضافی (سومی) برای نگه‌داری داده‌های خود استفاده کنید.
{{< /caution >}}

<!-- body -->
## دلایل استفاده

برای تنظیم داده‌های پیکربندی جداگانه از کد برنامه از ConfigMap استفاده کنید.

به عنوان مثال، فرض کنید که شما در حال توسعه یک برنامه هستید که می‌توانید آن را در کامپیوتر خود اجرا کنید
(برای توسعه) و در ابر (برای پردازش ترافیک واقعی). شما کد را می‌نویسید تا به یک متغیر محیطی به نام `DATABASE_HOST`
نگاه کنید. به صورت محلی، این متغیر را به `localhost` تنظیم می‌کنید. در ابر، آن را به
یک {{< glossary_tooltip text="Service" term_id="service" >}} Kubernetes که جزئیات مربوط به پایگاه داده را به خوشی در کلاستر شما ارائه می‌دهد، تنظیم می‌کنید.
این به شما اجازه می‌دهد که یک تصویر کانتینر را که در ابر در حال اجرا است بگیرید و
کد دقیقاً همان کدی که به آن نیاز دارید را به صورت محلی اشکال زدایی کنید اگر نیاز باشد.

{{< note >}}
یک ConfigMap برای نگه‌داری بلوک‌های بزرگی از داده طراحی نشده است. داده‌های ذخیره شده در یک
ConfigMap نمی‌توانند بیش از ۱ مگابایت باشند. اگر نیاز به ذخیره تنظیماتی دارید که
بزرگتر از این حد باشند، ممکن است بخواهید یک ولوم را mount کنید یا از
یک پایگاه داده یا سرویس فایل جداگانه استفاده کنید.
{{< /note >}}

## Object ConfigMap

ConfigMap یک {{< glossary_tooltip text="API object" term_id="object" >}}
است که به شما امکان می‌دهد تا پیکربندی را برای استفاده از سایر اشیاء ذخیره کنید. برخلاف اکثر
اشیاء Kubernetes که یک `spec` دارند، ConfigMap دارای فیلدهای `data` و `binaryData`
است. این فیلدها جفت‌های کلید-مقدار را به عنوان مقادیر خود قبول می‌کنند. هر دو فیلد `data`
و `binaryData` اختیاری هستند. فیلد `data` طراحی شده است که شامل رشته‌های UTF-8 باشد در حالی که فیلد
`binaryData` طراحی شده است که داده‌های دودویی به صورت رشته‌های base64-encoded باشند.

نام یک ConfigMap باید یک
[نام subdomain DNS معتبر](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names)
باشد.

هر کلید تحت فیلد `data` یا `binaryData` باید از حروف الفبایی، عددی، `-`، `_` یا `.` تشکیل شده و نباید
با کلیدهای موجود در فیلد `binaryData` همپوشانی داشته باشد.

از نسخه v1.19 به بعد، می‌توانید یک فیلد `immutable` به تعریف ConfigMap
اضافه کنید تا یک [ConfigMap غیرقابل تغییر](#configmap-immutable) ایجاد کنید.

## ConfigMaps و Pods

می‌توانید یک `spec` Pod را بنویسید که به یک ConfigMap ارجاع دهد و container(s) را در آن Pod بر اساس داده‌های موجود در ConfigMap پیکربندی کند. Pod و ConfigMap باید در
همان {{< glossary_tooltip text="namespace" term_id="namespace" >}} باشند.

{{< note >}}
`spec` یک {{< glossary_tooltip text="static Pod" term_id="static-pod" >}} نمی‌تواند به یک ConfigMap
یا هر اشیاء API دیگری ارجاع دهد.
{{< /note >}}

در اینجا یک ConfigMap نمونه است که برخی از کلیدها دارای مقادیر تکی هستند،
و سایر کلیدها که مقدار به نظر می‌رسد بخشی از یک فرمت پیکربندی هستند.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # کلید‌های شبیه به ویژگی؛ هر کلید به یک مقدار ساده نگاشته می‌شود
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # کلیدهای شبیه به فایل
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
```

چهار روش مختلف وجود دارد که می‌توانید از یک ConfigMap برای پیکربندی
یک container درون یک Pod استفاده کنید:

1. درون یک دستور و args container
1. متغیرهای محیطی برای یک container
1. اضافه کردن یک فایل در volume فقط خواندنی، برای برنامه برای خواندن
1. نوشتن کد برای اجرا درون Pod که از API Kubernetes برای خواندن یک ConfigMap استفاده می‌کند



این روش‌های مختلف به روش‌های مختلف برای مدل‌سازی داده‌های مصرفی منجر می‌شوند.
برای سه روش اول، {{< glossary_tooltip text="kubelet" term_id="kubelet" >}} داده‌ها را از
ConfigMap استفاده می‌کند زمانی که container(s) برای یک Pod راه‌اندازی می‌کند.

روش چهارم به این معنی است که شما باید کدی را بنویسید که ConfigMap و داده‌های آن را بخواند.
با این حال، از آنجا که شما مستقیماً از API Kubernetes استفاده می‌کنید، برنامه شما می‌تواند
مشترک شود تا زمانی که ConfigMap تغییر کند، و واکنش نشان دهد
هنگامی که این اتفاق می‌افتد. با دسترسی مستقیم به API Kubernetes، این
تکنیک همچنین به شما امکان می‌دهد تا به یک ConfigMap در یک فضای نام دیگر دسترسی پیدا کنید.

اینجا یک Pod نمونه است که از مقادیر `game-demo` برای پیکربندی یک Pod استفاده می‌کند:

{{% code_sample file="configmap/configure-pod.yaml" %}}

یک ConfigMap تفاوتی بین مقادیر ویژگی تک خط و
مقادیر شبیه به فایل ندارد.
مهم این است که چگونه Pods و اشیاء دیگر این مقادیر را مصرف می‌کنند.

برای این مثال، تعریف یک volume و mount کردن آن درون container `demo` به عنوان `/config` دو فایل ایجاد می‌کند،
`/config/game.properties` و `/config/user-interface.properties`،
هرچند که چهار کلید در ConfigMap وجود دارد. این به این دلیل است که تعریف Pod یک آرایه `items` را در بخش `volumes`
مشخص می‌کند. اگر آرایه `items` را به طور کامل حذف کنید، هر کلید در ConfigMap تبدیل به یک فایل با همان نام کلیدی می‌شود، و شما 4 فایل دریافت می‌کنید.

## استفاده از ConfigMaps

ConfigMaps می‌توانند به عنوان volume‌های داده‌ای mount شوند. ConfigMaps همچنین می‌توانند توسط دیگر
بخش‌های سیستم استفاده شوند، بدون اینکه مستقیماً به Pod ارائه دهند. به عنوان مثال،
ConfigMaps می‌توانند داده‌هایی را که باید بخش‌های دیگر سیستم از آن‌ها برای پیکربندی استفاده کنند نگه‌داری کنند.

رایج‌ترین روش برای استفاده از ConfigMaps پیکربندی تنظیمات برای
container‌های در حال اجرا در Pod در همان namespace است. همچنین می‌توانید از یک
ConfigMap به صورت مجزا استفاده کنید.

به عنوان مثال، شما ممکن است با {{< glossary_tooltip text="addons" term_id="addons" >}}
یا {{< glossary_tooltip text="operators" term_id="operator-pattern" >}} روبه‌رو شوید که عملکرد خود را بر اساس یک ConfigMap تغییر می‌دهند.

### استفاده از ConfigMaps به عنوان فایل‌ها از داخل یک Pod

برای استفاده از یک ConfigMap به عنوان یک volume در یک Pod:

1. یک ConfigMap ایجاد کنید یا از یک ConfigMap موجود استفاده کنید. چندین Pod می‌توانند به همان ConfigMap ارجاع دهند.
2. تعریف Pod خود را تغییر دهید تا یک volume در زیر `.spec.volumes[]` اضافه کنید. نام volume را هرچیزی بگذارید و فیلد `.spec.volumes[].configMap.name` را برای ارجاع به شیء ConfigMap خود تنظیم کنید.
3. به هر container که نیاز به ConfigMap دارد، یک `.spec.containers[].volumeMounts[]` اضافه کنید. `.spec.containers[].volumeMounts[].readOnly = true` را مشخص کنید و `mountPath` را به یک نام دایرکتوری غیر استفاده شده تنظیم کنید که می‌خواهید ConfigMap در آن ظاهر شود.
4. تصویر یا خط فرمان خود را تغییر دهید تا برنامه به دنبال فایل‌ها در آن دایرکتوری بگردد. هر کلید در نقشه `data` ConfigMap به عنوان نام فایل تحت `mountPath` تبدیل می‌شود.

این یک مثال از یک Pod است که یک ConfigMap را در یک volume mount می‌کند:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    configMap:
      name: myconfigmap
```

هر ConfigMap که می‌خواهید استفاده کنید باید به صورت `.spec.volumes` ارجاع شود.

اگر در Pod چندین container وجود داشته باشد، هر container نیاز به بلاک `volumeMounts` خود دارد، اما فقط یک `.spec.volumes` برای هر ConfigMap لازم است.

#### ConfigMaps مونت شده به طور خودکار به روز می‌شوند

زمانی که یک ConfigMap که در حال حاضر در یک volume مصرف می‌شود، به روزرسانی می‌شود، کلیدهای پروژه به طور نهایی نیز به روزرسانی می‌شوند. kubelet بررسی می‌کند که آیا ConfigMap مونت شده در هر همگامی دوره‌ای تازه است. با این حال، kubelet از کش محلی خود برای دریافت مقدار کنونی ConfigMap استفاده می‌کند. نوع کش قابل پیکربندی با استفاده از فیلد `configMapAndSecretChangeDetectionStrategy` در `struct KubeletConfiguration` است. یک ConfigMap می‌تواند به روش‌های watch (پیش‌فرض)، ttl-based، یا با انتقال کل درخواست‌ها به‌طور مستقیم به سرور API propagate شود. بنابراین، تأخیر کل از زمان به‌روزرسانی ConfigMap تا زمانی که کلید‌های جدید به Pod پروژه شوند، می‌تواند به عنوان دوره sync kubelet + تأخیر گسترش کش تأخیر تعیین شود، که به انتخاب نوع کش وابسته است (معادل به تأخیر گسترش گسترش watch، ttl کش، یا صفر).

ConfigMaps مصرف شده به عنوان متغیرهای محیطی به طور خودکار به روز نمی‌شوند و نیاز به restart pod دارند.

{{< note >}}
یک container که از یک ConfigMap به عنوان یک volume mount استفاده می‌کند، نمی‌تواند به‌روزرسانی ConfigMap بگیرد.
{{< /note >}}

### استفاده از ConfigMaps به عنوان متغیرهای محیطی

برای استفاده از یک ConfigMap به عنوان یک متغیر محیطی در یک Pod:

1. برای هر container در مشخصات Pod خود، یک متغیر محیطی اضافه کنید برای هر کلید Configmap که می‌خواهید به `env[].valueFrom.configMapKeyRef` ارجاع دهید.
2. تصویر و/یا خط فرمان خود را تغییر دهید تا برنامه به دنبال مقادیر در متغیرهای محیطی مشخص شده بگردد.

این یک مثال از تعریف یک ConfigMap به عنوان یک متغیر محیطی Pod است:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-configmap
spec:
  containers:
  - name: envars-test-container
    image: nginx
    env:
    - name: CONFIGMAP_USERNAME
      valueFrom:
        configMapKeyRef:
          name: myconfigmap
          key: username

```

مهم است که توجه داشته باشید که محدوده کاراکترهای مجاز برای نام‌های متغیرهای محیطی در Pod ها [محدود شده است](/docs/tasks/inject-data-application/define-environment-variable-container/#using-environment-variables-inside-of-your-config). اگر کلیدها با قوانین مطابقت نداشته باشند، آن کلیدها برای container شما در دسترس نخواهند بود، اگرچه این Pod اجازه شروع می‌دهد.

## ConfigMaps غیرقابل تغییر {#configmap-immutable}

{{< feature-state for_k8s_version="v1.21" state="stable" >}}

ویژگی Kubernetes _Secrets و ConfigMaps غیرقابل تغییر_ گزینه‌ای برای تنظیم فراهم می‌کند
Secrets و ConfigMaps فردی به عنوان غیرقابل تغییر. برای خوشه‌هایی که به طور گسترده از ConfigMaps استفاده می‌کنند
(حداقل ده‌ها هزار ConfigMap یکتا به نقطه پرداخت کردنی تغییرات برای داده ها) در اینجا به مراحلی که ممکن است

- شما از به روزرسانی دستی ConfigMap خواهید برد به API، به

 شما انجام
  همچنین باعث می‌شود که به منجر شدن
  می‌کند مانع خدمات درون شود تراز کلیه یک سیستم نیاز به روش غیردیگران.
