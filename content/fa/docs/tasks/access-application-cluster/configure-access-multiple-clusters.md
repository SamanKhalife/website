---
title: پیکربندی دسترسی به چندین کلاستر
content_type: task
weight: 30
card:
  name: tasks
  weight: 25
  title: پیکربندی دسترسی به کلاسترها
---

<!-- overview -->

این صفحه نشان می‌دهد که چگونه می‌توان با استفاده از فایل‌های پیکربندی دسترسی به چندین کلاستر را پیکربندی کرد. پس از تعریف کلاسترها، کاربران و کانتکست‌ها در یک یا چند فایل پیکربندی، می‌توانید با استفاده از دستور `kubectl config use-context` به سرعت بین کلاسترها جابجا شوید.

{{< note >}}
فایلی که برای پیکربندی دسترسی به یک کلاستر استفاده می‌شود، گاهی اوقات فایل *kubeconfig* نامیده می‌شود. این یک روش عمومی برای اشاره به فایل‌های پیکربندی است و به این معنا نیست که فایلی به نام `kubeconfig` وجود دارد.
{{< /note >}}

{{< warning >}}
فقط از فایل‌های kubeconfig از منابع معتبر استفاده کنید. استفاده از فایل kubeconfig که به صورت خاص ساخته شده است می‌تواند منجر به اجرای کد مخرب یا افشای فایل‌ها شود.
اگر مجبور به استفاده از فایل kubeconfig غیرمعتبر هستید، ابتدا آن را به دقت بررسی کنید، همانطور که یک اسکریپت شل را بررسی می‌کنید.
{{< /warning >}}

## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}}

برای بررسی اینکه {{< glossary_tooltip text="kubectl" term_id="kubectl" >}} نصب شده است، دستور `kubectl version --client` را اجرا کنید. نسخه kubectl باید [در یکی از نسخه‌های جزیی](/releases/version-skew-policy/#kubectl) سرور API کلاستر شما باشد.

<!-- steps -->

## تعریف کلاسترها، کاربران و کانتکست‌ها

فرض کنید شما دو کلاستر دارید، یکی برای کارهای توسعه و دیگری برای کارهای آزمایشی.
در کلاستر `توسعه`، توسعه‌دهندگان فرانت‌اند در یک فضای نام به نام `frontend` کار می‌کنند و توسعه‌دهندگان ذخیره‌سازی در یک فضای نام به نام `storage` کار می‌کنند. در کلاستر `آزمایش`، توسعه‌دهندگان در فضای نام پیش‌فرض کار می‌کنند یا به دلخواه خود فضاهای نام کمکی ایجاد می‌کنند. دسترسی به کلاستر توسعه نیاز به احراز هویت توسط گواهی دارد. دسترسی به کلاستر آزمایشی نیاز به احراز هویت توسط نام کاربری و رمز عبور دارد.

یک دایرکتوری به نام `config-exercise` ایجاد کنید. در دایرکتوری `config-exercise`، فایلی به نام `config-demo` با این محتوا ایجاد کنید:

```yaml
apiVersion: v1
kind: Config
preferences: {}

clusters:
- cluster:
  name: development
- cluster:
  name: test

users:
- name: developer
- name: experimenter

contexts:
- context:
  name: dev-frontend
- context:
  name: dev-storage
- context:
  name: exp-test
```

یک فایل پیکربندی کلاسترها، کاربران و کانتکست‌ها را توصیف می‌کند. فایل `config-demo` شما دارای چارچوبی برای توصیف دو کلاستر، دو کاربر و سه کانتکست است.

به دایرکتوری `config-exercise` خود بروید. این دستورات را برای اضافه کردن جزئیات کلاستر به فایل پیکربندی خود وارد کنید:

```shell
kubectl config --kubeconfig=config-demo set-cluster development --server=https://1.2.3.4 --certificate-authority=fake-ca-file
kubectl config --kubeconfig=config-demo set-cluster test --server=https://5.6.7.8 --insecure-skip-tls-verify
```

جزئیات کاربر را به فایل پیکربندی خود اضافه کنید:

{{< caution >}}
ذخیره رمزهای عبور در پیکربندی کلاینت Kubernetes پرخطر است. یک جایگزین بهتر استفاده از افزونه اعتباری و ذخیره آنها به صورت جداگانه است. مشاهده کنید: [افزونه‌های اعتباری client-go](/docs/reference/access-authn-authz/authentication/#client-go-credential-plugins)
{{< /caution >}}

```shell
kubectl config --kubeconfig=config-demo set-credentials developer --client-certificate=fake-cert-file --client-key=fake-key-file
kubectl config --kubeconfig=config-demo set-credentials experimenter --username=exp --password=some-password
```

{{< note >}}
- برای حذف یک کاربر می‌توانید دستور `kubectl --kubeconfig=config-demo config unset users.<name>` را اجرا کنید.
- برای حذف یک کلاستر، می‌توانید دستور `kubectl --kubeconfig=config-demo config unset clusters.<name>` را اجرا کنید.
- برای حذف یک کانتکست، می‌توانید دستور `kubectl --kubeconfig=config-demo config unset contexts.<name>` را اجرا کنید.
{{< /note >}}

جزئیات کانتکست را به فایل پیکربندی خود اضافه کنید:

```shell
kubectl config --kubeconfig=config-demo set-context dev-frontend --cluster=development --namespace=frontend --user=developer
kubectl config --kubeconfig=config-demo set-context dev-storage --cluster=development --namespace=storage --user=developer
kubectl config --kubeconfig=config-demo set-context exp-test --cluster=test --namespace=default --user=experimenter
```

فایل `config-demo` خود را باز کنید تا جزئیات اضافه شده را ببینید. به عنوان یک جایگزین برای باز کردن فایل `config-demo`، می‌توانید از دستور `config view` استفاده کنید.

```shell
kubectl config --kubeconfig=config-demo view
```

خروجی دو کلاستر، دو کاربر و سه کانتکست را نشان می‌دهد:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: fake-ca-file
    server: https://1.2.3.4
  name: development
- cluster:
    insecure-skip-tls-verify: true
    server: https://5.6.7.8
  name: test
contexts:
- context:
    cluster: development
    namespace: frontend
    user: developer
  name: dev-frontend
- context:
    cluster: development
    namespace: storage
    user: developer
  name: dev-storage
- context:
    cluster: test
    namespace: default
    user: experimenter
  name: exp-test
current-context: ""
kind: Config
preferences: {}
users:
- name: developer
  user:
    client-certificate: fake-cert-file
    client-key: fake-key-file
- name: experimenter
  user:
    # Documentation note (this comment is NOT part of the command output).
    # Storing passwords in Kubernetes client config is risky.
    # A better alternative would be to use a credential plugin
    # and store the credentials separately.
    # See https://kubernetes.io/docs/reference/access-authn-authz/authentication/#client-go-credential-plugins
    password: some-password
    username: exp
```

فایل‌های `fake-ca-file`، `fake-cert-file` و `fake-key-file` بالا محل‌هایی هستند برای مسیرهای گواهی‌نامه‌ها. باید این‌ها را به مسیرهای واقعی گواهی‌نامه‌ها در محیط خود تغییر دهید.

گاهی اوقات ممکن است بخواهید از داده‌های Base64-رمزنگاری‌شده به جای فایل‌های گواهی‌نامه جداگانه استفاده کنید؛ در این صورت باید به کلیدها پسوند `-data` اضافه کنید، به عنوان مثال، `certificate-authority-data`، `client-certificate-data`، `client-key-data`.

هر کانتکست یک سه‌گانه (کلاستر، کاربر، فضای نام) است. برای مثال، کانتکست `dev-frontend` می‌گوید: "از اعتبارنامه‌های کاربر `developer` برای دسترسی به فضای نام `frontend` کلاستر `development` استفاده کن".

کانتکست فعلی را تنظیم کنید:

```shell
kubectl config --kubeconfig=config-demo use-context dev-frontend
```

حالا هر زمان که دستور `kubectl` را وارد کنید، عملیات بر روی کلاستر و فضای نام موجود در کانتکست `dev-frontend` اعمال خواهد شد و دستور از اعتبارات کاربر موجود در کانتکست `dev-frontend` استفاده خواهد کرد.

برای دیدن اطلاعات پیکربندی مرتبط فقط با کانتکست فعلی، از پرچم `--minify` استفاده کنید.

```shell
kubectl config --kubeconfig=config-demo view --minify
```

خروجی، اطلاعات پیکربندی مرتبط با کانتکست `dev-frontend` را نشان می‌دهد:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: fake-ca-file
    server: https://1.2.3.4
  name: development
contexts:
- context:
    cluster: development
    namespace: frontend
    user: developer
  name: dev-frontend
current-context: dev-frontend
kind: Config
preferences: {}
users:
- name: developer
  user:
    client-certificate: fake-cert-file
    client-key: fake-key-file
```

حالا فرض کنید می‌خواهید مدتی در کلاستر آزمایشی کار کنید.

کانتکست فعلی را به `exp-test` تغییر دهید:

```shell
kubectl config --kubeconfig=config-demo use-context exp-test
```

حالا هر دستور `kubectl` که وارد کنید، بر روی فضای نام پیش‌فرض کلاستر `test` اعمال خواهد شد و دستور از اعتبارات کاربر موجود در کانتکست `exp-test` استفاده خواهد کرد.

اطلاعات پیکربندی مرتبط با کانتکست فعلی جدید، `exp-test`، را ببینید.

```shell
kubectl config --kubeconfig=config-demo view --minify
```

و در نهایت، فرض کنید می‌خواهید مدتی در فضای نام `storage` در کلاستر `development` کار کنید.

کانتکست فعلی را به `dev-storage` تغییر دهید:

```shell
kubectl config --kubeconfig=config-demo use-context dev-storage
```

اطلاعات پیکربندی مرتبط با کانتکست فعلی جدید، `dev-storage`، را ببینید.

```shell
kubectl config --kubeconfig=config-demo view --minify
```

## ایجاد فایل پیکربندی دوم

در دایرکتوری `config-exercise` خود، یک فایل با نام `config-demo-2` با این محتوا ایجاد کنید:

```yaml
apiVersion: v1
kind: Config
preferences: {}

contexts:
- context:
    cluster: development
    namespace: ramp
    user: developer
  name: dev-ramp-up
```

فایل پیکربندی فوق، یک کانتکست جدید به نام `dev-ramp-up` را تعریف می‌کند.

## تنظیم متغیر محیطی KUBECONFIG

بررسی کنید که آیا یک متغیر محیطی به نام `KUBECONFIG` دارید یا خیر. اگر دارید، مقدار فعلی متغیر محیطی `KUBECONFIG` را ذخیره کنید تا بعداً بتوانید آن را بازگردانید. به عنوان مثال:

### Linux

```shell
export KUBECONFIG_SAVED="$KUBECONFIG"
```

### Windows PowerShell

```powershell
$Env:KUBECONFIG_SAVED=$ENV:KUBECONFIG
```

متغیر محیطی `KUBECONFIG` یک لیست از مسیرهای فایل‌های پیکربندی است. این لیست برای لینوکس و مک با دو نقطه؛ و برای ویندوز با نقطه و کاما جدا شده است. اگر یک متغیر محیطی `KUBECONFIG` دارید، با فایل‌های پیکربندی در این لیست آشنا شوید.

موقتاً دو مسیر به متغیر محیطی `KUBECONFIG` خود اضافه کنید. به عنوان مثال:

### Linux

```shell
export KUBECONFIG="${KUBECONFIG}:config-demo:config-demo-2"
```

### Windows PowerShell

```powershell
$Env:KUBECONFIG=("config-demo;config-demo-2")
```

در دایرکتوری `config-exercise` خود، این دستور را وارد کنید:

```shell
kubectl config view
```

خروجی، اطلاعات ترکیب شده از تمام فایل‌های موجود در متغیر محیطی `KUBECONFIG` را نشان می‌دهد. به خصوص، توجه کنید که اطلاعات ترکیب شده شامل کانتکست `dev-ramp-up` از فایل `config-demo-2` و سه کانتکست از فایل `config-demo` می‌باشد:

```yaml
contexts:
- context:
    cluster: development
    namespace: frontend
    user: developer
  name: dev-frontend
- context:
    cluster: development
    namespace: ramp
    user: developer
  name: dev-ramp-up
- context:
    cluster: development
    namespace: storage
    user: developer
  name: dev-storage
- context:
    cluster: test
    namespace: default
    user: experimenter
  name: exp-test
```

برای اطلاعات بیشتر درباره چگونگی ادغام فایل‌های kubeconfig، به [Organizing Cluster Access Using kubeconfig Files](/docs/concepts/configuration/organize-cluster-access-kubeconfig/) مراجعه کنید.

## بررسی دایرکتوری $HOME/.kube

اگر در حال حاضر یک کلاستر دارید و می‌توانید از `kubectl` برای تعامل با کلاستر استفاده کنید، احتمالاً یک فایل با نام `config` در دایرکتوری `$HOME/.kube` دارید.

به `$HOME/.kube` بروید و ببینید که چه فایل‌هایی در آنجا وجود دارد. به طور معمول، یک فایل با نام `config` وجود دارد. ممکن است فایل‌های پیکربندی دیگری هم در این دایرکتوری وجود داشته باشند. با محتوای این فایل‌ها آشنا شوید.

## اضافه کردن فایل $HOME/.kube/config به متغیر محیطی KUBECONFIG

اگر یک فایل `$HOME/.kube/config` دارید و هنوز در متغیر محیطی `KUBECONFIG` شما لیست نشده است، آن را به متغیر محیطی `KUBECONFIG` اضافه کنید. به عنوان مثال:

### Linux

```shell


export KUBECONFIG="${KUBECONFIG}:${HOME}/.kube/config"
```

### Windows PowerShell

```powershell
$Env:KUBECONFIG="$Env:KUBECONFIG;$HOME\.kube\config"
```

اطلاعات پیکربندی ترکیب شده از همه فایل‌هایی که در حال حاضر در متغیر محیطی `KUBECONFIG` لیست شده‌اند را ببینید. در دایرکتوری `config-exercise` خود، دستور زیر را وارد کنید:

```shell
kubectl config view
```

## پاکسازی

متغیر محیطی `KUBECONFIG` را به مقدار اصلی خود بازگردانید. به عنوان مثال:

### Linux

```shell
export KUBECONFIG="$KUBECONFIG_SAVED"
```

### Windows PowerShell

```powershell
$Env:KUBECONFIG=$ENV:KUBECONFIG_SAVED
```

## بررسی موضوع ممثل توسط kubeconfig

معمولاً واضح نیست که بعد از احراز هویت در کلاستر، ویژگی‌های زیر (نام کاربری، گروه‌ها) را خواهید دریافت. این می‌تواند چالشی بیشتر باشد اگر در عین حال بیش از یک کلاستر را مدیریت می‌کنید.

دستور فرعی `kubectl` برای بررسی ویژگی‌های موضوعی، مانند نام کاربری، برای کانتکست کاربری Kubernetes انتخاب شده شما وجود دارد: `kubectl auth whoami`.

برای یادگیری بیشتر در این خصوص، به [API access to authentication information for a client](/docs/reference/access-authn-authz/authentication/#self-subject-review) مراجعه کنید.
