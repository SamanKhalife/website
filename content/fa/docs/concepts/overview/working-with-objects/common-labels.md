---
title: برچسب‌های پیشنهادی
content_type: concept
weight: 100
---

<!-- overview -->
می‌توانید اشیاء Kubernetes را با ابزارهای بیشتری به جز kubectl و داشبورد به صورت بصری مدیریت کنید. یک مجموعه مشترک از برچسب‌ها به ابزارها امکان کار بین‌افزاری را فراهم می‌کند، اشیاء را به یک شیوه مشترک که تمام ابزارها می‌توانند درک کنند، توصیف می‌کند.

علاوه بر پشتیبانی از ابزارها، برچسب‌های پیشنهادی برنامه‌ها را به گونه‌ای توصیف می‌کنند که می‌توان از آنها پرس‌وجو کرد.

<!-- body -->
اطلاعات فنی در ارتباط با مفهوم یک _برنامه_ است. Kubernetes یک پلتفرم به عنوان سرویس (PaaS) نیست و مفهوم رسمی یک برنامه را ندارد یا به آن اجبار نمی‌آورد. به جای اینکه برنامه‌ها را غیررسمی و با استفاده از متا دیتا شرح دهند. تعریف اینکه چه چیزی شامل یک برنامه است، گنگ است.

{{< note >}}
این برچسب‌های پیشنهادی هستند. آنها مدیریت برنامه‌ها را آسان‌تر می‌کنند، اما برای هیچ ابزار هسته‌ای ضروری نیستند.
{{< /note >}}

برچسب‌ها و حاشیه‌نویسی‌های مشترک دارای یک پیشوند مشترک هستند: `app.kubernetes.io`. برچسب‌های بدون پیشوند به صورت خصوصی برای کاربران هستند. پیشوند مشترک اطمینان می‌دهد که برچسب‌های مشترک با برچسب‌های کاربری سفارشی تداخل ندارند.

## برچسب‌ها

برای بهره‌مندی کامل از استفاده از این برچسب‌ها، باید آنها را بر روی هر شی منبع اعمال کنید.

| کلید                                | توضیحات               | مثال  | نوع |
| ----------------------------------- | --------------------- | ------ | ---- |
| `app.kubernetes.io/name`            | نام برنامه           | `mysql` | string |
| `app.kubernetes.io/instance`        | یک نام منحصربه‌فرد شناسایی نام نمونه یک برنامه | `mysql-abcxyz` | string |
| `app.kubernetes.io/version`         | نسخه فعلی برنامه (مانند یک [SemVer 1.0](https://semver.org/spec/v1.0.0.html), هش بازبینی، و غیره) | `5.7.21` | string |
| `app.kubernetes.io/component`       | اجزا در معماری     | `database` | string |
| `app.kubernetes.io/part-of`         | نام برنامه سطح بالاتر که این بخش از آن است | `wordpress` | string |
| `app.kubernetes.io/managed-by`      | ابزار استفاده شده برای مدیریت عملکرد یک برنامه | `Helm` | string |

برای نمایش عملکرد این برچسب‌ها، در نظر بگیرید که یک شی {{< glossary_tooltip text="StatefulSet" term_id="statefulset" >}} برای MySQL وجود دارد:

```yaml
# این یک متن نمونه است
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxyz
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
    app.kubernetes.io/managed-by: Helm
```

## برنامه‌ها و نمونه‌های برنامه

یک برنامه می‌تواند یک یا چند بار در یک خوشه Kubernetes نصب شود و در برخی موارد، همان فضای نام. به عنوان مثال، WordPress می‌تواند بیشتر از یک بار نصب شود که وب‌سایت‌های مختلف نصب‌های مختلف WordPress هستند.

نام یک برنامه و نام نمونه جداگانه ثبت می‌شود. به عنوان مثال، WordPress دارای `app.kubernetes.io/name` به نام `wordpress` است در حالی که دارای نام نمونه است، نمایش داده می‌شود به عنوان `app.kubernetes.io/instance` با یک مقدار از `wordpress-abcxyz`. این امکان را فراهم می‌کند تا برنامه و نمونه برنامه قابل شناسایی باشند. هر نمونه از یک برنامه باید یک نام منحصر به فرد داشته باشد.

## مثال‌ها

برای نشان دادن روش‌های مختلف استفاده از این برچسب‌ها، مثال‌های زیر شامل پیچیدگی‌های متفاوت هستند.

### یک سرویس ساده بدون حالت

مورد ساده یک سرویس بدون حالت که با استفاده از شیء `Deployment` و `Service` را پیاده سازی می‌کند. دو قطعه کد زیر نشان می‌دهد که برچسب‌ها چگونه به سادگی در نهایت استفاده می‌شوند.

`Deployment` برای نظارت بر پادهای اجرای برنامه استفاده می‌شود.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: myservice
    app.kubernetes.io/instance: myservice-abcxyz
...
```

`Service` برای ارائه برنامه استفاده می‌شود.
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: myservice
    app.kubernetes.io/instance: myservice-abcxyz


...
```

### برنامه وب با پایگاه داده

یک برنامه کمی پیچیده‌تر: یک برنامه وب (WordPress) که از یک پایگاه داده (MySQL) استفاده می‌کند، با استفاده از Helm نصب شده است. قطعات زیر نشان می‌دهند یک شروع از اشیاء استفاده شده برای پیاده‌سازی این برنامه.

شروع به `Deployment` برای WordPress استفاده می‌شود:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: wordpress
    app.kubernetes.io/instance: wordpress-abcxyz
    app.kubernetes.io/version: "4.9.4"
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: server
    app.kubernetes.io/part-of: wordpress
...
```

`Service` برای ارائه WordPress استفاده می‌شود:
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: wordpress
    app.kubernetes.io/instance: wordpress-abcxyz
    app.kubernetes.io/version: "4.9.4"
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: server
    app.kubernetes.io/part-of: wordpress
...
```

MySQL به عنوان `StatefulSet` با متادیتا برای هر دو آن و برنامه اصلی که به آن تعلق دارد نمایش داده می‌شود:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxyz
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
...
```

`Service` برای ارائه MySQL به عنوان بخشی از WordPress استفاده می‌شود:
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxyz
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
...
```

با `StatefulSet` MySQL و `Service` می‌توانید مشاهده کنید که اطلاعات درباره هر دو MySQL و WordPress، برنامه‌های گسترده‌تر، ارائه شده است.
