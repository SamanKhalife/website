---
title: "تکمیل خودکار Bash در Linux"
description: "تنظیمات اختیاری برای تکمیل خودکار Bash در Linux."
headless: true
_build:
  list: never
  render: never
  publishResources: false
---

### مقدمه

اسکریپت تکمیل kubectl برای Bash می‌تواند با دستور `kubectl completion bash` ایجاد شود.
استفاده از اسکریپت تکمیل در شل شما، تکمیل خودکار kubectl را فعال می‌کند.

اما این اسکریپت تکمیل وابسته به
[**bash-completion**](https://github.com/scop/bash-completion)
است، به این معنی که ابتدا باید این نرم‌افزار را نصب کنید
(می‌توانید با اجرای دستور `type _init_completion` از این نرم‌افزار مطمئن شوید که bash-completion نصب است یا خیر).

### نصب bash-completion

bash-completion توسط بسیاری از مدیرهای بسته ارائه می‌شود
(برای جزئیات بیشتر [اینجا](https://github.com/scop/bash-completion#installation) را ببینید).
می‌توانید آن را با `apt-get install bash-completion` یا `yum install bash-completion` و غیره نصب کنید.

دستورات بالا `/usr/share/bash-completion/bash_completion` را ایجاد می‌کنند، که اسکریپت اصلی bash-completion است. بسته به مدیر بسته‌ی شما، شما باید این فایل را به صورت دستی در فایل `~/.bashrc` خود منبع کنید.

برای اطمینان، شل خود را بازنشانی کنید و `type _init_completion` را اجرا کنید.
اگر دستور موفق باشد، شما قبلا تنظیم شده‌اید، در غیر این صورت خط زیر را به فایل `~/.bashrc` خود اضافه کنید:

```bash
source /usr/share/bash-completion/bash_completion
```

شل خود را بازنشانی کنید و از صحت نصب bash-completion با اجرای `type _init_completion` اطمینان حاصل کنید.

### فعال‌سازی تکمیل خودکار kubectl

#### Bash

حالا باید اطمینان حاصل کنید که اسکریپت تکمیل kubectl در تمام نشست‌های شل شما منبع‌سازی شود. دو روش برای انجام این کار وجود دارد:

{{< tabs name="kubectl_bash_autocompletion" >}}
{{< tab name="کاربر" codelang="bash" >}}
echo 'source <(kubectl completion bash)' >>~/.bashrc
{{< /tab >}}
{{< tab name="سیستم" codelang="bash" >}}
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
sudo chmod a+r /etc/bash_completion.d/kubectl
{{< /tab >}}
{{< /tabs >}}

اگر برای kubectl یک نام مستعار دارید، می‌توانید تکمیل خودکار شل را برای آن کار کنید:

```bash
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
```

{{< note >}}
bash-completion تمام اسکریپت‌های تکمیل را در `/etc/bash_completion.d` منبع‌سازی می‌کند.
{{< /note >}}

هر دو روش معادل هستند. پس از بازنشانی شل خود، تکمیل خودکار kubectl باید کار کند.
برای فعال‌سازی تکمیل خودکار bash در جلسه جاری شل، فایل `~/.bashrc` را منبع‌سازی کنید:

```bash
source ~/.bashrc
```
