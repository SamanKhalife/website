---
title: "تکمیل خودکار Bash در macOS"
description: "تنظیمات اختیاری برای تکمیل خودکار Bash در macOS."
headless: true
_build:
  list: never
  render: never
  publishResources: false
---

### مقدمه

اسکریپت تکمیل kubectl برای Bash می‌تواند با دستور `kubectl completion bash` ایجاد شود.
منبع‌سازی این اسکریپت در شل شما، تکمیل خودکار kubectl را فعال می‌کند.

اما اسکریپت تکمیل kubectl وابسته به
[**bash-completion**](https://github.com/scop/bash-completion)
است، به همین دلیل شما باید این نرم‌افزار را قبلاً نصب کرده باشید.

{{< warning>}}
دو نسخه از bash-completion وجود دارد، v1 و v2. V1 برای Bash 3.2 است
(که به طور پیش‌فرض در macOS استفاده می‌شود) و v2 برای Bash 4.1+ است.
اسکریپت تکمیل kubectl با bash-completion v1 و Bash 3.2 **به درستی کار نمی‌کند**.
برای استفاده درست از تکمیل خودکار kubectl در macOS، شما باید Bash 4.1+ را نصب و استفاده کنید
([*دستورالعمل‌ها*](https://itnext.io/upgrading-bash-on-macos-7138bd1066ba)).
دستورات زیر فرض می‌کنند که شما از Bash 4.1+ استفاده می‌کنید.
{{< /warning >}}

### ارتقاء Bash

دستورات اینجا فرض می‌کنند که شما از Bash 4.1+ استفاده می‌کنید. شما می‌توانید نسخه Bash خود را با اجرای دستور زیر بررسی کنید:

```bash
echo $BASH_VERSION
```

اگر خیلی قدیمی باشد، می‌توانید آن را با استفاده از Homebrew نصب یا ارتقا دهید:

```bash
brew install bash
```

شل خود را بازنشانی کرده و اطمینان حاصل کنید که نسخه مورد نظر در حال استفاده است:

```bash
echo $BASH_VERSION $SHELL
```

به طور معمول، Homebrew آن را در `/usr/local/bin/bash` نصب می‌کند.

### نصب bash-completion

{{< note >}}
همانطور که گفته شد، این دستورات فرض می‌کنند که شما از Bash 4.1+ استفاده می‌کنید، به این معنی که شما باید bash-completion v2 را نصب کنید
(بر خلاف Bash 3.2 و bash-completion v1 که در آن صورت تکمیل خودکار kubectl کار نمی‌کند).
{{< /note >}}

شما می‌توانید با اجرای `type _init_completion` بررسی کنید که bash-completion v2 در حال حاضر نصب شده است یا خیر.
اگر نصب نیست، می‌توانید آن را با استفاده از Homebrew نصب کنید:

```bash
brew install bash-completion@2
```

همانطور که در خروجی این دستور ذکر شده است، خط زیر را به فایل `~/.bash_profile` خود اضافه کنید:

```bash
brew_etc="$(brew --prefix)/etc" && [[ -r "${brew_etc}/profile.d/bash_completion.sh" ]] && . "${brew_etc}/profile.d/bash_completion.sh"
```

شل خود را بازنشانی کرده و اطمینان حاصل کنید که bash-completion v2 به درستی نصب شده با استفاده از `type _init_completion`.

### فعال‌سازی تکمیل خودکار kubectl

حالا باید اطمینان حاصل کنید که اسکریپت تکمیل kubectl در تمام نشست‌های شل شما منبع‌سازی شود. چندین روش برای انجام این کار وجود دارد:

- منبع‌سازی اسکریپت تکمیل در فایل `~/.bash_profile` خود:

    ```bash
    echo 'source <(kubectl completion bash)' >>~/.bash_profile
    ```

- اضافه کردن اسکریپت تکمیل به دایرکتوری `/usr/local/etc/bash_completion.d`:

    ```bash
    kubectl completion bash >/usr/local/etc/bash_completion.d/kubectl
    ```

- اگر برای kubectl نام مستعاری دارید، می‌توانید تکمیل خودکار شل را برای آن کار کنید:

    ```bash
    echo 'alias k=kubectl' >>~/.bash_profile
    echo 'complete -o default -F __start_kubectl k' >>~/.bash_profile
    ```

- اگر kubectl را با استفاده از Homebrew نصب کرده‌اید (همانطور که
  [اینجا](/docs/tasks/tools/install-kubectl-macos/#install-with-homebrew-on-macos)
  توضیح داده شده است)،
  در آن صورت اسکریپت تکمیل kubectl باید از پیش در `/usr/local/etc/bash_completion.d/kubectl` باشد.
  در این حالت، نیازی به انجام هیچ کاری ندارید.

   {{< note >}}
   نصب Homebrew از bash-completion v2 تمام فایل‌ها را در
   `BASH_COMPLETION_COMPAT_DIR` منبع‌سازی می‌کند، به همین دلیل روش‌های دوم و سوم کار می‌کنند.
   {{< /note >}}

با انجام هر کدام از این مراحل و بازنشانی شل شما، تکمیل خودکار kubectl باید کار کند.
