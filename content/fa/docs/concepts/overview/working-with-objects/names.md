---
reviewers:
- mikedanese
- thockin
title: نام‌ها و شناسه‌های شیء
content_type: مفهوم
weight: 30
---

<!-- overview -->

هر {{< glossary_tooltip text="object" term_id="object" >}} در خوشه‌ی شما یک [_نام_](#names) دارد که برای این نوع منبع منحصر به فرد است.
هر شیء Kubernetes همچنین یک [_UID_](#uids) دارد که در کل خوشه‌ی شما منحصر به فرد است.

به عنوان مثال، شما می‌توانید فقط یک Pod با نام `myapp-1234` در همان [فضای‌نام](/docs/concepts/overview/working-with-objects/namespaces/) داشته باشید، اما می‌توانید یک Pod و یک Deployment داشته باشید که هر کدام با نام `myapp-1234` نامگذاری شوند.

برای ویژگی‌های تولیدشده توسط کاربر که منحصر به فرد نیستند، Kubernetes برچسب‌ها و حاشیه‌نویسی‌ها را فراهم می‌کند ([labels](/docs/concepts/overview/working-with-objects/labels/) و [annotations](/docs/concepts/overview/working-with-objects/annotations/)).


<!-- body -->

## نام‌ها

{{< glossary_definition term_id="name" length="all" >}}

**نام‌ها باید در سراسر تمام [نسخه‌های API](/docs/concepts/overview/kubernetes-api/#api-groups-and-versioning) یک منبع به صورت یکتا باشند. منابع API توسط گروه API، نوع منبع، فضای‌نام (برای منابع فضای‌نامی) و نام متمایز می‌شوند. به عبارت دیگر، نسخه API در این زمینه بی‌اثر است.**

{{< note >}}
در مواردی که اشیاء نمایانگر یک موجودیت فیزیکی مانند یک Node است که میزبان فیزیکی را نمایان می‌دهد، هنگامی که میزبان با همان نام بدون حذف و ایجاد مجدد Node ایجاد می‌شود، Kubernetes میزبان جدید را به عنوان میزبان قبلی می‌شناسد که ممکن است منجر به ناهم‌اهنگی‌ها شود.
{{< /note >}}

در زیر، چهار نوع محدودیت معمول برای نام منابع آورده شده است.

### نام‌های زیردامنه DNS

بیشتر انواع منابع نیاز به نامی دارند که می‌تواند به عنوان یک زیردامنه DNS استفاده شود، مطابق با [RFC 1123](https://tools.ietf.org/html/rfc1123).
این به معنای آن است که نام باید:

- حداکثر شامل 253 نویسه باشد
- فقط شامل حروف الفبایی کوچک، اعداد، '-' یا '.'
- با یک حرف الفبایی عددی شروع شود
- با یک حرف الفبایی عددی پایان یابد

### نام‌های برچسب RFC 1123 {#dns-label-names}

بعضی از انواع منابع نیاز دارند که نام آن‌ها به استاندارد برچسب DNS
مطابق با [RFC 1123](https://tools.ietf.org/html/rfc1123) باشد.
این به معنای آن است که نام باید:

- حداکثر شامل 63 نویسه باشد
- فقط شامل حروف الفبایی کوچک یا '-'
- با یک حرف الفبایی عددی شروع شود
- با یک حرف الفبایی عددی پایان یابد

### نام‌های برچسب RFC 1035

بعضی از انواع منابع نیاز دارند که نام آن‌ها به استاندارد برچسب DNS
مطابق با [RFC 1035](https://tools.ietf.org/html/rfc1035) باشد.
این به معنای آن است که نام باید:

- حداکثر شامل 63 نویسه باشد
- فقط شامل حروف الفبایی کوچک یا '-'
- با یک حرف الفبایی شروع شود
- با یک حرف الفبایی عددی پایان یابد

{{< note >}}
تنها تفاوت بین استانداردهای برچسب RFC 1035 و RFC 1123 این است که برچسب‌های RFC 1123 اجازه شروع با رقم را دارند، در حالی که برچسب‌های RFC 1035 تنها با یک حرف الفبایی کوچک شروع می‌شوند.
{{< /note >}}

### نام‌های بخش مسیر

بعضی از انواع منابع نیاز دارند که نام آن‌ها بتواند به طور ایمن به عنوان یک بخش مسیر کدگذاری شود. به عبارت دیگر، نام نباید "." یا ".." باشد و نباید شامل "/" یا "%" باشد.

اینجا یک نمونه نمایشی برای یک Pod به نام `nginx-demo` آمده است.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```


{{< note >}}
بعضی از انواع منابع محدودیت‌های اضافی درباره نام خود دارند.
{{< /note >}}

## UIDs

{{< glossary_definition term_id="uid" length="all" >}}

UIDهای Kubernetes شناسه‌گذارهای یکتا جهانی هستند (معروف به UUIDها).
UUIDها به عنوان استاندارد ISO/IEC 9834-8 و ITU-T X.667 استاندارد شده‌اند.


## {{% heading "whatsnext" %}}

* درباره [labels](/docs/concepts/overview/working-with-objects/labels/) و [annotations](/docs/concepts/overview/working-with-objects/annotations/) در Kubernetes بخوانید.
* مستند [شناسه‌گذارها و نام‌ها در Kubernetes](https://git.k8s.io/design-proposals-archive/architecture/identifiers.md) را ببینید.
