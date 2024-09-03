---
title: "تکمیل خودکار در shell fish"
description: "پیکربندی اختیاری برای فعال‌سازی تکمیل خودکار در shell fish."
headless: true
_build:
  list: never
  render: never
  publishResources: false
---

{{< note >}}
برای تکمیل خودکار در Fish نیاز به استفاده از kubectl نسخه 1.23 یا بالاتر است.
{{< /note >}}

اسکریپت تکمیل kubectl برای Fish می‌تواند با دستور `kubectl completion fish` ایجاد شود. منبع‌سازی این اسکریپت تکمیل در شل شما، تکمیل خودکار kubectl را فعال می‌کند.

برای این کار در تمام نشست‌های شل خود، خط زیر را به فایل `~/.config/fish/config.fish` اضافه کنید:

```shell
kubectl completion fish | source
```

بعد از بازنشانی شل خود، تکمیل خودکار kubectl باید کار کند.

