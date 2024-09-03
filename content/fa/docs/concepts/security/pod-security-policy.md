---
title: سیاست‌های امنیتی پاد
content_type: مفهوم
weight: 30
---

<!-- مرور -->

{{% alert title="ویژگی حذف شده" color="warning" %}}
PodSecurityPolicy در Kubernetes نسخه ۱.۲۱ منسوخ شد و در نسخه ۱.۲۵ از Kubernetes حذف شد.
{{% /alert %}}

به جای استفاده از PodSecurityPolicy، می‌توانید محدودیت‌های مشابه را بر روی Pods با استفاده از یکی یا هر دوی این موارد اعمال کنید:

- [Pod Security Admission](/docs/concepts/security/pod-security-admission/)
- یک افزونه ورود سوم که خودتان آن را نصب و پیکربندی می‌کنید

برای راهنمای مهاجرت، [Migrate from PodSecurityPolicy to the Built-In PodSecurity Admission Controller](/docs/tasks/configure-pod-container/migrate-from-psp/) را مشاهده کنید.
برای اطلاعات بیشتر درباره حذف این API، به [PodSecurityPolicy Deprecation: Past, Present, and Future](/blog/2021/04/06/podsecuritypolicy-deprecation-past-present-and-future/) مراجعه کنید.

اگر از نسخه Kubernetes v{{< skew currentVersion >}} استفاده نمی‌کنید، مستندات نسخه خود را بررسی کنید.
