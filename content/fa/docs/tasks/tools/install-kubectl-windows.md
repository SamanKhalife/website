---
reviewers:
- mikedanese
title: نصب و تنظیم kubectl در ویندوز
content_type: task
weight: 10
---

## {{% heading "پیش‌نیازها" %}}

شما باید از نسخه‌ای از kubectl استفاده کنید که در یک نسخه کوچک‌تر یا بزرگ‌تر نسخه کنترلی خود قرار دارد. به عنوان مثال، یک مشتری v{{< skew currentVersion >}} می‌تواند با صفحه کنترل‌های v{{< skew currentVersionAddMinor -1 >}}, v{{< skew currentVersionAddMinor 0 >}} و v{{< skew currentVersionAddMinor 1 >}} ارتباط برقرار کند.
استفاده از آخرین نسخه سازگار kubectl به جلوگیری از مشکلات غیرمنتظره کمک می‌کند.

## نصب kubectl در ویندوز

روش‌های زیر برای نصب kubectl در ویندوز وجود دارد:

- [نصب فایل اجرایی kubectl با curl در ویندوز](#install-kubectl-binary-with-curl-on-windows)
- [نصب با استفاده از Chocolatey، Scoop یا winget در ویندوز](#install-nonstandard-package-tools)

### نصب فایل اجرایی kubectl با curl در ویندوز

1. دانلود آخرین نسخه پچ {{< skew currentVersion >}}:
   [kubectl {{< skew currentPatchVersion >}}](https://dl.k8s.io/release/v{{< skew currentPatchVersion >}}/bin/windows/amd64/kubectl.exe).

   یا اگر `curl` را نصب کرده‌اید، از این دستور استفاده کنید:

   ```powershell
   curl.exe -LO "https://dl.k8s.io/release/v{{< skew currentPatchVersion >}}/bin/windows/amd64/kubectl.exe"
   ```

   {{< note >}}
   برای پیدا کردن آخرین نسخه پایدار (مثلاً برای اسکریپت‌نویسی) به [https://dl.k8s.io/release/stable.txt](https://dl.k8s.io/release/stable.txt) مراجعه کنید.
   {{< /note >}}

1. اعتبارسنجی فایل اجرایی (اختیاری)

   فایل چک‌سام کubectl را دانلود کنید:

   ```powershell
   curl.exe -LO "https://dl.k8s.io/v{{< skew currentPatchVersion >}}/bin/windows/amd64/kubectl.exe.sha256"
   ```

   اعتبارسنجی فایل باینری kubectl با فایل چک‌سام:

   - استفاده از Command Prompt برای مقایسه دستی خروجی CertUtil با فایل چک‌سام دانلود شده:

     ```cmd
     CertUtil -hashfile kubectl.exe SHA256
     type kubectl.exe.sha256
     ```

   - استفاده از PowerShell برای اتوماسیون اعتبارسنجی با استفاده از عملگر `-eq` برای دریافت نتیجه `True` یا `False`:

     ```powershell
     $(Get-FileHash -Algorithm SHA256 .\kubectl.exe).Hash -eq $(Get-Content .\kubectl.exe.sha256)
     ```

1. اضافه کردن یا پیش‌پیوستن پوشه باینری kubectl به متغیر محیطی `PATH`.

1. آزمایش برای اطمینان از اینکه نسخه kubectl با آنچه دانلود شده همخوانی دارد:

   ```cmd
   kubectl version --client
   ```

   یا برای مشاهده جزییات بیشتر از نسخه:

   ```cmd
   kubectl version --client --output=yaml
   ```

{{< note >}}
[Docker Desktop برای ویندوز](https://docs.docker.com/docker-for-windows/#kubernetes)
نسخه خود را به `PATH` اضافه می‌کند. اگر پیش از این Docker Desktop را نصب کرده‌اید، ممکن است نیاز به قرار دادن ورودی `PATH` خود قبل از ورودی اضافه شده توسط نصب‌کننده Docker Desktop داشته باشید یا kubectl Docker Desktop را حذف کنید.
{{< /note >}}

### نصب در ویندوز با استفاده از Chocolatey، Scoop یا winget {#install-nonstandard-package-tools}

1. برای نصب kubectl در ویندوز می‌توانید از مدیر بسته Chocolatey](https://chocolatey.org)، نصب کننده خط فرمان Scoop](https://scoop.sh) یا [مدیر بسته winget](https://learn.microsoft.com/en-us/windows/package-manager/winget/) استفاده کنید.

   {{< tabs name="kubectl_win_install" >}}
   {{% tab name="choco" %}}
   ```powershell
   choco install kubernetes-cli
   ```
   {{% /tab %}}
   {{% tab name="scoop" %}}
   ```powershell
   scoop install kubectl
   ```
   {{% /tab %}}
   {{% tab name="winget" %}}
   ```powershell
   winget install -e --id Kubernetes.kubectl
   ```
   {{% /tab %}}
   {{< /tabs >}}

1. برای اطمینان از اینکه نسخه‌ای که نصب کرده‌اید به‌روز است، تست کنید:

   ```powershell
   kubectl version --client
   ```

1. به دایرکتوری خانه‌ی خود بروید:

   ```powershell
   # اگر از cmd.exe استفاده می‌کنید، این دستور را اجرا کنید: cd %USERPROFILE%
   cd ~
   ```

1. دایرکتوری `.kube` را ایجاد کنید:

   ```powershell
   mkdir .kube
   ```

1. به دایرکتوری `.kube` که به تازگی ایجاد کرده‌اید بروید:

   ```powershell
   cd .kube
   ```

1. kubectl را پیکربندی کنید تا از یک خوشه Kubernetes از راه دور استفاده کند:

   ```powershell
   New-Item config -type file
   ```

{{< note >}}
فایل پیکربندی را با ویرایشگر متنی دلخواه خود ویرایش کنید، مانند Notepad.
{{< /note >}}

## تایید پیکربندی kubectl

{{< include "included/verify-kubectl.md" >}}

## پیکربندی‌ها و پلاگین‌های اختیاری kubectl

### فعال‌سازی تکمیل خودکار پوسته

kubectl پشتیبانی از تکمیل خودکار برای Bash، Zsh، Fish و PowerShell را فراهم می‌کند که می‌تواند زمان زیادی را برای شما صرفه‌جویی کند.

در زیر روش‌های تنظیم تکمیل خودکار برای PowerShell آمده است.

{{< include "included/optional-kubectl-configs-pwsh.md" >}}

### نصب پلاگین `kubectl convert`

{{< include "included/kubectl-convert-overview.md" >}}

1. آخرین نسخه را با این دستور دانلود کنید:

   ```powershell
   curl.exe -LO "https://dl.k8s.io/release/v{{< skew currentPatchVersion >}}/bin/windows/amd64/kubectl-convert.exe"
   ```

1. اعتبارسنجی فایل اجرایی (اختیاری).

   فایل چک‌سام kubectl-convert را دانلود کنید:

   ```powershell
   curl.exe -LO "https://dl.k8s.io/v{{< skew currentPatchVersion >}}/bin/windows/amd64/kubectl-convert.exe.sha256"
   ```

   اعتبارسنجی فایل باینری kubectl-convert با فایل چک‌سام:

   - استفاده از Command Prompt برای مقایسه دستی خروجی CertUtil با فایل چک‌سام دانلود شده:

     ```cmd
     CertUtil -hashfile kubectl-convert.exe SHA256
     type kubectl-convert.exe.sha256
     ```

   - استفاده از PowerShell برای اتوماسیون اعتبارسنجی با استفاده از عملگر `-eq` برای دریافت نتیجه `True` یا `False`:

     ```powershell
     $($(CertUtil -hashfile .\kubectl-convert.exe SHA256)[1] -replace " ", "") -eq $(type .\kubectl-convert.exe.sha256)
     ```

1. پوشه باینری kubectl-convert را به متغیر محیطی `PATH` اضافه یا پیش‌پیوست کنید.

1. تایید کنید که پلاگین با موفقیت نصب شده است.

   ```shell
   kubectl convert --help
   ```

   اگر خطایی ندیدید، این به معنی نصب موفق پلاگین است.

1. پس از نصب پلاگین، فایل‌های نصب را پاک کنید:

   ```powershell
   del kubectl-convert.exe
   del kubectl-convert.exe.sha256
   ```

## {{% heading "what's next" %}}

{{< include "included/kubectl-whats-next.md" >}}
