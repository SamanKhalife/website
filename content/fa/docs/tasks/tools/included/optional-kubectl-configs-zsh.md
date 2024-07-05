---
title: "تکمیل خودکار در zsh"
description: "پیکربندی اختیاری برای فعال‌سازی تکمیل خودکار در zsh."
headless: true
_build:
  list: never
  render: never
  publishResources: false
---

اسکریپت تکمیل kubectl برای Zsh می‌تواند با دستور `kubectl completion zsh` ایجاد شود. منبع‌سازی این اسکریپت تکمیل در شل شما، تکمیل خودکار kubectl را فعال می‌کند.

برای این کار در تمام نشست‌های شل خود، خط زیر را به فایل `~/.zshrc` خود اضافه کنید:

```zsh
source <(kubectl completion zsh)
```

اگر برای kubectl یک نام مستعار (alias) دارید، تکمیل خودکار kubectl به صورت خودکار با آن کار خواهد کرد.

بعد از بازنشانی شل خود، تکمیل خودکار kubectl باید کار کند.

اگر خطایی مانند `2: command not found: compdef` دریافت کردید، آنگاه خط زیر را به ابتدای فایل `~/.zshrc` خود اضافه کنید:

```zsh
autoload -Uz compinit
compinit
```

