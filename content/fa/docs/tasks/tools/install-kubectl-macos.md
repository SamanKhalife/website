```markdown
## پیش‌نیازها

شما باید از نسخه‌ای از kubectl استفاده کنید که یک نسخهٔ کوچک‌تر یا بزرگ‌تر از نسخهٔ خوشهٔ شما باشد. به عنوان مثال، یک مشتری v{{< skew currentVersion >}} می‌تواند با تخته‌های کنترل v{{< skew currentVersionAddMinor -1 >}}، v{{< skew currentVersionAddMinor 0 >}} و v{{< skew currentVersionAddMinor 1 >}} ارتباط برقرار کند. استفاده از آخرین نسخه سازگار با kubectl کمک می‌کند تا مشکلات غیرمنتظره را از بین ببرید.

## نصب kubectl در macOS

روش‌های زیر برای نصب kubectl در macOS موجود است:

- [نصب kubectl در macOS](#install-kubectl-on-macos)
  - [نصب باینری kubectl با curl در macOS](#install-kubectl-binary-with-curl-on-macos)
  - [نصب با استفاده از Homebrew در macOS](#install-with-homebrew-on-macos)
  - [نصب با استفاده از Macports در macOS](#install-with-macports-on-macos)
- [تأیید پیکربندی kubectl](#verify-kubectl-configuration)
- [پیکربندی‌ها و افزونه‌های اختیاری kubectl](#optional-kubectl-configurations-and-plugins)
  - [فعالسازی تکمیل خودکار پوسته](#enable-shell-autocompletion)
  - [نصب افزونه `kubectl convert`](#install-kubectl-convert-plugin)

### نصب باینری kubectl با curl در macOS

1. دانلود آخرین نسخه:

   {{< tabs name="download_binary_macos" >}}
   {{< tab name="Intel" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
   {{< /tab >}}
   {{< tab name="Apple Silicon" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl"
   {{< /tab >}}
   {{< /tabs >}}

   {{< note >}}
   برای دانلود یک نسخه خاص، بخش `$(curl -L -s https://dl.k8s.io/release/stable.txt)` از دستور را با نسخه خاص جایگزین کنید.

   به عنوان مثال، برای دانلود نسخه {{< skew currentPatchVersion >}} بر روی macOS Intel، دستور زیر را تایپ کنید:

   ```bash
   curl -LO "https://dl.k8s.io/release/v{{< skew currentPatchVersion >}}/bin/darwin/amd64/kubectl"
   ```

   و برای macOS روی Apple Silicon، دستور زیر را تایپ کنید:

   ```bash
   curl -LO "https://dl.k8s.io/release/v{{< skew currentPatchVersion >}}/bin/darwin/arm64/kubectl"
   ```

   {{< /note >}}

2. اعتبارسنجی باینری (اختیاری)

   دانلود فایل checksum برای kubectl:

   {{< tabs name="download_checksum_macos" >}}
   {{< tab name="Intel" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl.sha256"
   {{< /tab >}}
   {{< tab name="Apple Silicon" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl.sha256"
   {{< /tab >}}
   {{< /tabs >}}
  
   اعتبارسنجی باینری kubectl نسبت به فایل checksum:

   ```bash
   echo "$(cat kubectl.sha256)  kubectl" | shasum -a 256 --check
   ```

   اگر صحیح باشد، خروجی مشابه زیر خواهد بود:

   ```console
   kubectl: OK
   ```

   اگر بررسی ناموفق باشد، `shasum` با خروجی مشابه زیر از وضعیت غیرصحیح خارج می‌شود:

   ```console
   kubectl: FAILED
   shasum: WARNING: 1 computed checksum did NOT match
   ```

   {{< note >}}
   دانلود همان نسخه از باینری و فایل checksum را انجام دهید.
   {{< /note >}}

3. تنظیم اجرایی باینری kubectl.

   ```bash
   chmod +x ./kubectl
   ```

4. انتقال باینری kubectl به مسیر فایلی در محیط `PATH` سیستم شما.

   ```bash
   sudo mv ./kubectl /usr/local/bin/kubectl
   sudo chown root: /usr/local/bin/kubectl
   ```

   {{< note >}}
   اطمینان حاصل کنید که `/usr/local/bin` در متغیر محیطی PATH شما قرار دارد.
   {{< /note >}}

5. آزمایش برای اطمینان از اینکه نسخه نصب شده به‌روز است:

   ```bash
   kubectl version --client
   ```

6. پس از نصب و اعتبارسنجی kubectl، فایل checksum را حذف کنید:

   ```bash
   rm kubectl.sha256
   ```

### نصب با استفاده از Homebrew در macOS

اگر شما در macOS هستید و از مدیریت بستهٔ Homebrew استفاده می‌کنید، می‌توانید kubectl را با Homebrew نصب کنید.

1. اجرای دستور نصب:

   ```bash
   brew install kubectl
   ```

   یا

   ```bash
   brew install kubernetes-cli
   ```

2. آزمایش برای اطمینان از اینکه نسخه نصب شده به‌روز است:

   ```bash
   kubectl version --client
   ```

### نصب با استفاده از Macports در macOS

اگر شما در macOS هستید و از مدیریت بستهٔ [Macports](https://macports.org/) استفاده می‌کنید، می‌توانید kubectl را با Macports نصب کنید.

1. اجرای دستور نصب:

   ```bash
   sudo port selfupdate
   sudo port install kubectl
   ```

2. آزمایش برای اطمینان از اینکه نسخه نصب شده به‌روز است:

   ```bash
   kubectl version --client
   ```

## تأیید پیکربندی kubectl

{{< include "included/verify-kubectl.md" >}}

## پیکربندی‌ها و افزونه‌های اختیاری kubectl

### فعالسازی تکمیل خودکار پوسته

kubectl پشتیبانی از تکمیل خودکار را برای Bash، Zsh، Fish و PowerShell فراهم می‌کند که می‌تواند در ذخیره‌سازی تایپ‌های شما کمک کند.

در زیر روش‌های تنظیم تکمیل خودکار برای Bash، Fish و Zsh آمده است.

{{< tabs name="kubectl_autocompletion" >}}
{{< tab name="Bash" include="included/optional-kubectl-configs-bash-mac.md" />}}
{{< tab name="Fish" include="included/optional-kubectl-configs-fish.md" />}}
{{< tab name="Zsh" include="included/optional-kubectl-configs-zsh.md" />}}
{{< /tabs >}}

### نصب افزونه `kubectl convert`

{{< include "included/kubectl-convert-overview.md" >}}

1. دانلود آخرین نسخه با دستور زیر:

   {{< tabs name="download_convert_binary_macos" >}}
   {{< tab name="Intel" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl-convert"
   {{< /tab >}}
   {{< tab name="Apple Silicon" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl-convert"
   {{< /tab >}}
   {{< /tabs >}}

2. اعتبارسنجی باینری (اختیاری)

   دانلود فایل checksum برای kubectl-convert:

   {{< tabs name="download_convert_checksum_macos" >}}
   {{< tab name="Intel" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl-convert.sha256"
   {{< /tab >}}
   {{< tab name="Apple Silicon" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl-convert.sha256"
   {{< /tab >}}
   {{< /tabs >}}

   اعتبارسنجی باینری kubectl-convert نسبت به فایل checksum:

   ```bash
   echo "$(cat kubectl-convert.sha256)  kubectl-convert" | shasum -a 256 --check
   ```

   اگر صحیح باشد، خروجی مشابه زیر خواهد بود:

   ```console
   kubectl-convert: OK
   ```

   اگر بررسی ناموفق باشد، `shasum` با خروجی مشابه زیر از وضعیت غیرصحیح خارج می‌شود:

   ```console
   kubectl-convert: FAILED
   shasum: WARNING: 1 computed checksum did NOT match
   ```

   {{< note >}}
   دانلود همان نسخه از باینری و فایل checksum را انجام دهید.
   {{< /note >}}

3. تنظیم اجرایی باینری kubectl-convert

   ```bash
   chmod +x ./kubectl-convert
   ```

4. انتقال باینری kubectl-convert به مسیر فایلی در محیط `PATH` سیستم شما.

   ```bash
   sudo mv ./kubectl-convert /usr/local/bin/kubectl-convert
   sudo chown root: /usr/local/bin/kubectl-convert
   ```

   {{< note >}}
   اطمینان حاصل کنید که `/usr/local/bin` در متغیر محیطی PATH شما قرار دارد.
   {{< /note >}}

5. تأیید نصب موفق افزونه

   ```shell
   kubectl convert --help
   ```

   اگر خطایی ندید، به این معنی است که افزونه با موفقیت نصب شده است.

6. پس از نصب افزونه، فایل‌های نصب را پاک کنید:

   ```bash
   rm kubectl-convert kubectl-convert.sha256
   ```

### حذف kubectl از macOS

بر اساس اینکه چگونه kubectl را نصب کرده‌اید، از یکی از روش‌های زیر استفاده کنید.

### حذف kubectl با استفاده از خط فرمان

1. مکان باینری kubectl را در سیستم خود پیدا کنید:

   ```bash
   which kubectl
   ```

2. باینری kubectl را حذف کنید:

   ```bash
   sudo rm <path>
   ```
   `<path>` را با مسیر باینری kubectl از مرحله قبل جایگزین کنید. به عنوان مثال، `sudo rm /usr/local/bin/kubectl`.

### حذف kubectl با استفاده از Homebrew

اگر شما از Homebrew برای نصب kubectl استفاده کرده‌اید، دستور زیر را اجرا کنید:

```bash
brew remove kubectl
```

## {{% heading "whatsnext" %}}

{{< include "included/kubectl-whats-next.md" >}}
