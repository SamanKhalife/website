---
title: "امنیت"
weight: 85
description: >
  مفاهیم برای حفظ امنیت بارگیری‌های نیتیوی ابری خود.
simple_list: true
---

این بخش از مستندات Kubernetes به شما کمک می‌کند تا بهتر یاد بگیرید که چگونه بارگیری‌ها را با امنیت بیشتر اجرا کنید و در مورد جنبه‌های اساسی حفظ امنیت یک کلاستر Kubernetes مطالعه کنید.

Kubernetes بر اساس یک معماری نیتیوی ابری استوار است و از مشورت‌های {{< glossary_tooltip text="CNCF" term_id="cncf" >}} درباره شیوه‌های خوب امنیت اطلاعات نیتیوی ابری بهره می‌برد.

برای اطلاعات بیشتر در مورد چگونگی امنیت کلاستر خود و برنامه‌هایی که روی آن اجرا می‌کنید، مطالعه کنید: [امنیت نیتیوی ابری و Kubernetes](/docs/concepts/security/cloud-native-security/).

## مکانیزم‌های امنیت Kubernetes {#security-mechanisms}

Kubernetes شامل چندین API و کنترل‌های امنیتی است، همچنین روش‌هایی برای تعریف [سیاست‌ها](#policies) که می‌توانند بخشی از نحوه مدیریت امنیت اطلاعات شما باشند.

### حفاظت از صفحه کنترل

یکی از مکانیزم‌های اصلی امنیتی برای هر کلاستر Kubernetes، [کنترل دسترسی به API Kubernetes](/docs/concepts/security/controlling-access) است.

Kubernetes انتظار دارد که شما TLS را پیکربندی کرده و استفاده کنید تا [رمزنگاری داده در حال انتقال](/docs/tasks/tls/managing-tls-in-a-cluster/) را در داخل صفحه کنترل و بین صفحه کنترل و مشتریان خود فراهم کنید. همچنین می‌توانید [رمزنگاری در استراحت](/docs/tasks/administer-cluster/encrypt-data/) را برای داده‌های ذخیره شده در صفحه کنترل Kubernetes فعال کنید؛ این جدا از استفاده از رمزنگاری در استراحت برای داده‌های بارگیری‌های خود نیز ممکن است یک ایده خوب باشد.

### رمزنگاری در استراحت

API [Secret](/docs/concepts/configuration/secret/) حفاظت پایه‌ای برای مقادیر پیکربندی که نیاز به محرمانگی دارند فراهم می‌کند.

### حفاظت از بارگیری‌ها

برای اطمینان از اینکه Pod‌ها و ظروف آنها به طور مناسب جدا شده‌اند، [استانداردهای امنیتی Pod](/docs/concepts/security/pod-security-standards/) را اجرا کنید. همچنین می‌توانید از [RuntimeClasses](/docs/concepts/containers/runtime-class) استفاده کنید تا جداسازی سفارشی را تعریف کنید اگر نیاز دارید.

[سیاست‌های شبکه](/docs/concepts/services-networking/network-policies/) به شما امکان می‌دهند تا ترافیک شبکه را بین Podها یا بین Podها و شبکه خارج از کلاستر کنترل کنید.

می‌توانید کنترل‌های امنیتی از اکوسیستم گسترده‌تر را برای پیاده‌سازی کنترل‌های پیشگیرانه یا تشخیصی در اطراف Podها، ظروف آنها و تصاویری که در آنها اجرا می‌شوند، پیاده‌سازی کنید.

### بررسی‌های حسابرسی

[ثبت بررسی‌های Kubernetes](/docs/tasks/debug/debug-cluster/audit/) مجموعه‌ای از رکوردهای زمانی امنیتی را فراهم می‌کند که دنباله‌ای از اقدامات در یک کلاستر را مستند می‌کند. کلاستر اقدامات توسط کاربران تولید شده را از طریق برنامه‌های که از API Kubernetes استفاده می‌کنند و توسط صفحه کنترل خود به ثبت می‌رساند.

## امنیت ارائه دهنده ابری

{{% thirdparty-content vendor="true" %}}

اگر یک کلاستر Kubernetes را بر روی سخت‌افزار خود یا ارائه دهنده ابری متفاوت اجرا می‌کنید، بهتر است بهترین شیوه‌های امنیتی خود را مطابق مستندات خود بررسی کنید. در زیر لینک‌هایی به برخی از مستندات امنیتی ارائه دهندگان معروف ابری وجود دارد:

{{< table caption="امنیت ارائه دهنده‌های ابری" >}}

تامین کننده IaaS        | لینک |
-------------------- | ------------ |
Alibaba Cloud | https://www.alibabacloud.com/trust-center |
Amazon Web Services | https://aws.amazon.com/security |
Google Cloud Platform | https://cloud.google.com/security |
Huawei Cloud | https://www.huaweicloud.com/intl/en-us/securecenter/overallsafety |
IBM Cloud | https://www.ibm.com/cloud/security |
Microsoft Azure | https://docs.microsoft.com/en-us/azure/security/azure-security |
Oracle Cloud Infrastructure | https://www.oracle.com/security |
VMware vSphere | https://www.vmware.com/security/hardening-guides |

{{< /table >}}

## سیاست‌ها

می‌توانید با استفاده از مکانیسم‌های امنیتی نیتیوی Kubernetes، مانند [NetworkPolicy](/docs/concepts/services-networking/network-policies/)
(کنترل اعلامی بر فیلترینگ بسته‌های شبکه) یا
[ValidatingAdmissionPolicy](/docs/reference/access-authn-authz/validating-admission-policy/) (محدودیت‌های اعلامی بر اینکه چه تغییراتی می‌تواند با استفاده از API Kubernetes ایجاد شود)، سیاست‌های امنیتی را تعریف کنید.

با این حال، می‌توانید همچنین بر سیاست‌های اجرایی از اکوسیستم گسترده‌تر اطراف Kubernetes تکیه کنید. Kubernetes مکانیزم‌های توسعه را فراهم می‌کند تا به این پروژه‌های اکوسیستم اجازه دهد تا کنترل‌های سیاستی خود را بر روی بررسی کد منبع، تأیید تصاویر ظروف، کنترل‌های دسترسی API، شبکه و غیره پیاده‌سازی کنند.

برای اطلاعات بیشتر در مورد مکانیسم‌های سیاستی و Kubernetes، به مطالعه [سیاست‌ها](/docs/concepts/policy/) بپردازید.

## {{% heading "whatsnext" %}}

در مورد موضوعات امنیتی مرتبط با Kubernetes بیشتر بیاموزید:

* [حفظ امنیت کلاستر خود](/docs/tasks/administer-cluster/securing-a-cluster/)
* [ثبت شده‌های شناخته شده](/docs/reference/issues-security/official-cve-feed/)
  در Kubernetes (و لینک‌های به اطلاعات بیشتر)
* [رمزنگاری داده در حال انتقال](/docs/tasks/tls/managing-tls-in-a-cluster/) برای صفحه کنترل
* [رمزنگاری داده در استراحت](/docs/tasks/administer-cluster/encrypt-data/)
* [کنترل دسترسی به API Kubernetes](/docs/concepts/security/controlling-access)
* [سیاست‌های شبکه](/docs/concepts/services-networking/network-policies/) برای Podها
* [رازها در Kubernetes](/docs/concepts/configuration/secret/)
* [استانداردهای امنیتی Pod](/docs/concepts/security/pod-security-standards/)
* [RuntimeClasses](/docs/concepts/containers/runtime-class)

مفاهیم را بیاموزید:

<!-- اگر این را تغییر دهید، همچنین پیش‌منبع صفحه از محتوای اولیه مستندات/en/docs/concepts/security/cloud-native-security.md مطابقت را بررسی کنید -->
* [امنیت نیتیوی ابری و Kubernetes](/docs/concepts/security/cloud-native-security/)

مدرک بگیرید:

* [متخصص امنیت معتبر Kubernetes](https://training.linuxfoundation.org/certification/certified-kubernetes-security-specialist/)
  دوره آموزشی و گواهینامه رسمی.

بیشتر بخوانید در این بخش:
