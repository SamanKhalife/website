---
reviewers:
- jayunit100
- jsturtevant
- marosset
- perithompson
title:    امنیت برای نودهای ویندوز
content_type: مفهوم
weight: 40
---

<!-- overview -->

این صفحه مشتمل بر ملاحظات امنیتی و شیوه‌های بهتر برای سیستم‌عامل ویندوز است.

<!-- body -->

## حفاظت از داده‌های Secret در نودها

در ویندوز، داده‌های Secret به صورت متن ساده در فضای ذخیره‌سازی محلی نود نوشته می‌شوند
(بر خلاف استفاده از سیستم‌های فایل tmpfs / در حافظه بر روی لینوکس). به عنوان اپراتور خوشه، شما باید دو اقدام اضافی زیر را انجام دهید:

1. استفاده از ACL فایل برای امنیت مکان فایل‌های Secret.
1. استفاده از رمزگذاری سطح حجم با استفاده از
   [BitLocker](https://docs.microsoft.com/windows/security/information-protection/bitlocker/bitlocker-how-to-deploy-on-windows-server).

## کاربران کانتینر

می‌توانید برای Pods یا کانتینرهای ویندوز،
[RunAsUsername](/docs/tasks/configure-pod-container/configure-runasusername)
را مشخص کنید تا فرآیندهای کانتینر را به عنوان کاربر خاص اجرا کند. این تقریباً معادل
[RunAsUser](/docs/concepts/security/pod-security-policy/#users-and-groups)
است.

کانتینرهای ویندوز دارای دو حساب کاربری پیش‌فرض هستند، ContainerUser و ContainerAdministrator.
تفاوت‌ها بین این دو حساب کاربری در
[زمان استفاده از حساب‌های کاربری ContainerAdmin و ContainerUser](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/container-security#when-to-use-containeradmin-and-containeruser-user-accounts)
در مستندات _Secure Windows containers_ شرکت Microsoft مورد بررسی قرار گرفته است.

کاربران محلی می‌توانند به تصاویر کانتینری در فرآیند ساخت کانتینر اضافه شوند.

{{< note >}}

* تصاویر مبتنی بر [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver)
  به صورت پیش‌فرض به عنوان `ContainerUser` اجرا می‌شوند
* تصاویر مبتنی بر [Server Core](https://hub.docker.com/_/microsoft-windows-servercore)
  به صورت پیش‌فرض به عنوان `ContainerAdministrator` اجرا می‌شوند

{{< /note >}}

کانتینرهای ویندوز همچنین می‌توانند به عنوان هویت‌های Active Directory با استفاده از
[Group Managed Service Accounts](/docs/tasks/configure-pod-container/configure-gmsa/)
اجرا شوند.

## جداسازی امنیتی سطح Pods

مکانیزم‌های خاص امنیتی Pods مانند SELinux، AppArmor، Seccomp یا قابلیت‌های POSIX سازگاری
در نودهای ویندوز پشتیبانی نمی‌شوند.

کانتینرهای مجازی [در حالت Privileged](/docs/concepts/windows/intro/#compatibility-v1-pod-spec-containers-securitycontext)
در ویندوز پشتیبانی نمی‌شوند.
به جای آن می‌توانید از کانتینرهای [HostProcess](/docs/tasks/configure-pod-container/create-hostprocess-pod)
در ویندوز استفاده کنید تا بسیاری از وظایفی که توسط کانتینرهای Privileged در لینوکس انجام می‌شود، انجام دهید.
