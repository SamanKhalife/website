
---
title: "تکمیل خودکار در PowerShell"
description: "پیکربندی اختیاری برای فعال‌سازی تکمیل خودکار در PowerShell."
headless: true
_build:
  list: never
  render: never
  publishResources: false
---

اسکریپت تکمیل kubectl برای PowerShell می‌تواند با دستور `kubectl completion powershell` ایجاد شود.

برای فعال‌سازی این اسکریپت تکمیل در تمام نشست‌های شل PowerShell، خط زیر را به فایل `$PROFILE` خود اضافه کنید:

```powershell
kubectl completion powershell | Out-String | Invoke-Expression
```

این دستور هر بار که PowerShell راه‌اندازی می‌شود، اسکریپت تکمیل خودکار را بازسازی می‌کند. همچنین می‌توانید اسکریپت تکمیل تولید شده را مستقیما به فایل `$PROFILE` خود اضافه کنید.

برای اضافه کردن اسکریپت تکمیل تولید شده به فایل `$PROFILE`، دستور زیر را در خط فرمان PowerShell خود اجرا کنید:

```powershell
kubectl completion powershell >> $PROFILE
```

بعد از بازنشانی شل PowerShell، تکمیل خودکار kubectl باید کار کند.
