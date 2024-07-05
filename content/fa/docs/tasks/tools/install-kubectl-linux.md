---
reviewers:
- mikedanese
title: نصب و راه‌اندازی kubectl در Linux
content_type: task
weight: 10
---

## {{% heading "پیش‌نیازها" %}}

شما باید از نسخه‌ای از kubectl استفاده کنید که در یک نسخه کوچک‌تر یا بزرگ‌تر از خوشه شما قرار دارد. به عنوان مثال، یک مشترک v{{< skew currentVersion >}} می‌تواند با سرورهای کنترل v{{< skew currentVersionAddMinor -1 >}}، v{{< skew currentVersionAddMinor 0 >}}، و v{{< skew currentVersionAddMinor 1 >}} ارتباط برقرار کند. استفاده از آخرین نسخه سازگار با kubectl ممکنه مشکلات غیرمنتظره را جلوگیری کند.

## نصب kubectl در Linux

روش‌های زیر برای نصب kubectl در Linux وجود دارد:

- [نصب باینری kubectl با curl بر روی Linux](#install-kubectl-binary-with-curl-on-linux)
- [نصب با استفاده از مدیریت بسته‌های نیتیو](#install-using-native-package-management)
- [نصب با استفاده از مدیریت بسته‌های دیگر](#install-using-other-package-management)

### نصب باینری kubectl با curl بر روی Linux

1. دانلود آخرین نسخه با دستور زیر:

   {{< tabs name="download_binary_linux" >}}
   {{< tab name="x86-64" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   {{< /tab >}}
   {{< tab name="ARM64" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/arm64/kubectl"
   {{< /tab >}}
   {{< /tabs >}}

   {{< note >}}
   برای دانلود یک نسخه خاص، بخش `$(curl -L -s https://dl.k8s.io/release/stable.txt)` را با نسخه خاص جایگزین کنید.

   به عنوان مثال، برای دانلود نسخه {{< skew currentPatchVersion >}} بر روی Linux x86-64، دستور زیر را وارد کنید:

   ```bash
   curl -LO https://dl.k8s.io/release/v{{< skew currentPatchVersion >}}/bin/linux/amd64/kubectl
   ```

   و برای Linux ARM64، دستور زیر را وارد کنید:

   ```bash
   curl -LO https://dl.k8s.io/release/v{{< skew currentPatchVersion >}}/bin/linux/arm64/kubectl
   ```

   {{< /note >}}

1. اعتبارسنجی باینری (اختیاری)

   فایل چک‌سام kubectl را دانلود کنید:

   {{< tabs name="download_checksum_linux" >}}
   {{< tab name="x86-64" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
   {{< /tab >}}
   {{< tab name="ARM64" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/arm64/kubectl.sha256"
   {{< /tab >}}
   {{< /tabs >}}

   باینری kubectl را در مقایسه با فایل چک‌سام اعتبارسنجی کنید:

   ```bash
   echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
   ```

   اگر معتبر باشد، خروجی به شکل زیر خواهد بود:

   ```console
   kubectl: OK
   ```

   اگر بررسی ناموفق باشد، `sha256` با خطایی غیرصحیح خروجی می‌دهد و پیامی مانند زیر چاپ می‌شود:

   ```console
   kubectl: FAILED
   sha256sum: WARNING: 1 computed checksum did NOT match
   ```

   {{< note >}}
   فایل و باینری نسخه مشابه را دانلود کنید و اعتبارسنجی کنید.
   {{< /note >}}

1. نصب kubectl

   ```bash
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   ```

   {{< note >}}
   اگر دسترسی root را در سیستم هدف ندارید، همچنان می‌توانید kubectl را در دایرکتوری `~/.local/bin` نصب کنید:

   ```bash
   chmod +x kubectl
   mkdir -p ~/.local/bin
   mv ./kubectl ~/.local/bin/kubectl
   # و سپس ~/.local/bin را به $PATH اضافه کنید (یا ابتدا یا انتها)
   ```

   {{< /note >}}

1. تست برای اطمینان از اینکه نسخه نصب شده به‌روز است:

   ```bash
   kubectl version --client
   ```

   یا برای مشاهده دقیق تر نسخه:

   ```cmd
   kubectl version --client --output=yaml
   ```

### نصب با استفاده از مدیریت بسته‌های نیتیو

{{< tabs name="kubectl_install" >}}
{{% tab name="توزیع‌های مبتنی بر Debian" %}}

1. به‌روزرسانی شاخص بسته‌های `apt` و نصب بسته‌های مورد نیاز برای استفاده از مخازن `apt` Kubernetes:

   ```shell
   sudo apt-get update
   # apt-transport-https ممکن است یک بسته خالی باشد؛ اگر چنین است، می‌توانید این بسته را از ردیابی کنید
   sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
   ```

2. دانلود کلید امضای عمومی برای مخازن بسته‌های Kubernetes. این همان کلید امضای استفاده شده برای تمام مخازن است، بنابراین می‌توانید نسخه را در URL نادیده بگیرید:

   ```shell
   # در نسخه‌های قدیمی‌تر از Debian 12 و Ubuntu 22.04، پوشه `/etc/apt/keyrings` به صورت پیش‌فرض وجود ندارد و باید قبل از دستور curl آن را ایجاد کنید، نکته زیر را بخوانید.
   # sudo mkdir -p -m 755 /etc/apt/keyrings
   curl -fsSL https://pkgs.k8s.io/core:/stable:/{{< param "version" >}}/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
   sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # اجازه می‌دهد به برنامه‌های غیر امتیازی APT این keyring را بخوانند
   ```

{{< note >}}
در نسخه‌های قدیمی‌تر از Debian 12 و Ubuntu 22.04، پوشه `/etc/apt/keyrings` به صورت پیش‌فرض وجود ندارد و باید قبل از دستور curl آن را ایجاد کنید.
{{< /note >}}

3. اضافه کردن مخزن مناسب `apt` Kubernetes. اگر می‌خواهید از نسخه Kubernetes متفاوتی نسبت به {{< param "version" >}} استفاده کنید،
   نسخه کوچک‌تر را با مورد مورد نظر خود جایگزین کنید:

   ```shell
   # این همه تنظیمات موجود در /etc/apt/sources.list.d/kubernetes.list را بازنویسی می‌کند
   echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/{{< param "version" >}}/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
   sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list   # به ابزارهایی مانند command-not-found کمک می‌کند که به درستی کار کنند
   ```

{{< note >}}
برای ارتقاء kubectl به یک نسخه کوچکتر، باید نسخه را در `/etc/apt/sources.list.d/kubernetes.list` قبل از اجرای `apt-get update` و `apt-get upgrade` بالا ببرید. این روش به تفصیل در [تغییر مخزن بسته Kubernetes](/docs/tasks/administer-cluster/kubeadm/change-package-repository/) توضیح داده شده است.
{{< /note >}}

4. به‌روزرسانی شاخص بسته‌های `apt`، سپس نصب kubectl:

   ```shell
   sudo apt-get update
   sudo apt-get install -y kubectl
   ```

{{% /tab %}}

{{% tab name="توزیع‌های بر پایه Red Hat" %}}

1. افزودن مخزن `yum` Kubernetes. اگر می‌خواهید از نسخه Kubernetes
   متفاوتی نسبت به {{< param "version" >}} استفاده کنید، {{< param "version" >}} را با
   نسخه مورد نظر خود جایگزین کنید.

   ```bash
   # این همه تنظیمات موجود در /etc/yum.repos.d/kubernetes.repo را بازنویسی می‌کند
   cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
   [kubernetes]
   name=Kubernetes
   baseurl=https://pkgs.k8s.io/core:/stable:/{{< param "version" >}}/rpm/
   enabled=1
   gpgcheck=1
   gpgkey=https://pkgs.k8s.io/core:/stable:/{{< param "version" >}}/rpm/repodata/repomd.xml.key
   EOF
   ```

{{< note >}}
برای ارتقاء kubectl به یک نسخه کوچکتر، باید نسخه را در `/etc/yum.repos.d/kubernetes.repo` قبل از اجرای `yum update` بالا ببرید. این روش به تفصیل در [تغییر مخزن بسته Kubernetes](/docs/tasks/administer-cluster/kubeadm/change-package-repository/) توضیح داده شده است.
{{< /note >}}

2. نصب kubectl با استفاده از `yum`:

   ```bash
   sudo yum install -y kubectl
   ```

{{% /tab %}}

{{% tab name="توزیع‌های بر پایه SUSE" %}}

1. افزودن مخزن `zypper` Kubernetes. اگر می‌خواهید از نسخه Kubernetes
   متفاوتی نسبت به {{< param "version" >}} استفاده کنید، {{< param "version" >}} را با
   نسخه مورد نظر خود جایگزین کنید.

   ```bash
   # این همه تنظیمات موجود در /etc/zypp/repos.d/kubernetes.repo را بازنویسی می‌کند
   cat <<EOF | sudo tee /etc/zypp/repos.d/kubernetes.repo
   [kubernetes]
   name=Kubernetes
   baseurl=https://pkgs.k8s.io/core:/stable:/{{< param "version" >}}/rpm/
   enabled=1
   gpgcheck=1
   gpgkey=https://pkgs.k8s.io/core:/stable:/{{< param "version" >}}/rpm/repodata/repomd.xml.key
   EOF
   ```

{{< note >}}
برای ارتقاء kubectl به یک نسخه کوچکتر، باید نسخه را در `/etc/zypp/repos.d/kubernetes.repo` قبل از اجرای `zypper update` بالا ببرید. این روش به تفصیل در [تغییر مخزن بسته Kubernetes](/docs/tasks/administer-cluster/kubeadm/change-package-repository/) توضیح داده شده است.
{{< /note >}}

2. نصب kubectl با استفاده از `zypper`:

   ```bash
   sudo zypper install -y kubectl
   ```

{{% /tab %}}
{{< /tabs >}}

### نصب با استفاده از مدیریت بسته‌های دیگر

{{< tabs name="other_kubectl_install"

 >}}
{{% tab name="Snap" %}}
اگر شما در اوبونتو یا یک توزیع لینوکس دیگر که از مدیریت بسته‌های [snap](https://snapcraft.io/docs/core/install) پشتیبانی می‌کند، هستید، kubectl به عنوان یک برنامه [snap](https://snapcraft.io/) در دسترس است.

```shell
snap install kubectl --classic
kubectl version --client
```

{{% /tab %}}

{{% tab name="Homebrew" %}}
اگر شما در لینوکس هستید و از مدیریت بسته‌های [Homebrew](https://docs.brew.sh/Homebrew-on-Linux) استفاده می‌کنید، kubectl برای [نصب](https://docs.brew.sh/Homebrew-on-Linux#install) در دسترس است.

```shell
brew install kubectl
kubectl version --client
```

{{% /tab %}}

{{< /tabs >}}

## تأیید تنظیمات kubectl

{{< include "included/verify-kubectl.md" >}}

## تنظیمات اختیاری kubectl و افزونه‌ها

### فعال کردن تکمیل خودکار پوسته

kubectl پشتیبانی از تکمیل خودکار برای Bash، Zsh، Fish، و PowerShell را فراهم می‌کند که می‌تواند به شما در ذخیره کردن تایپ زیاد کمک کند.

در زیر روش‌های راه‌اندازی تکمیل خودکار برای Bash، Fish، و Zsh آمده است.

{{< tabs name="kubectl_autocompletion" >}}
{{< tab name="Bash" include="included/optional-kubectl-configs-bash-linux.md" />}}
{{< tab name="Fish" include="included/optional-kubectl-configs-fish.md" />}}
{{< tab name="Zsh" include="included/optional-kubectl-configs-zsh.md" />}}
{{< /tabs >}}

### نصب افزونه `kubectl convert`

{{< include "included/kubectl-convert-overview.md" >}}

1. دانلود آخرین نسخه با دستور زیر:

   {{< tabs name="download_convert_binary_linux" >}}
   {{< tab name="x86-64" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert"
   {{< /tab >}}
   {{< tab name="ARM64" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/arm64/kubectl-convert"
   {{< /tab >}}
   {{< /tabs >}}

1. اعتبارسنجی فایل دودویی (اختیاری)

   فایل checksum kubectl-convert را دانلود کنید:

   {{< tabs name="download_convert_checksum_linux" >}}
   {{< tab name="x86-64" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert.sha256"
   {{< /tab >}}
   {{< tab name="ARM64" codelang="bash" >}}
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/arm64/kubectl-convert.sha256"
   {{< /tab >}}
   {{< /tabs >}}

   اعتبارسنجی فایل دودویی kubectl-convert در مقابل فایل checksum:

   ```bash
   echo "$(cat kubectl-convert.sha256) kubectl-convert" | sha256sum --check
   ```

   اگر معتبر باشد، خروجی به شکل زیر است:

   ```console
   kubectl-convert: OK
   ```

   اگر بررسی ناموفق باشد، `sha256` با وضعیت نامناسب خروجی می‌دهد و خروجی مشابه زیر را چاپ می‌کند:

   ```console
   kubectl-convert: FAILED
   sha256sum: WARNING: 1 computed checksum did NOT match
   ```

   {{< note >}}
   نسخه مشابه فایل دودویی و checksum را دانلود کنید.
   {{< /note >}}

1. نصب kubectl-convert

   ```bash
   sudo install -o root -g root -m 0755 kubectl-convert /usr/local/bin/kubectl-convert
   ```

1. تأیید نصب موفقیت‌آمیز افزونه

   ```shell
   kubectl convert --help
   ```

   اگر خطا ندیدید، بدین معنی است که افزونه با موفقیت نصب شده است.

1. پس از نصب افزونه، پاک کردن فایل‌های نصب:

   ```bash
   rm kubectl-convert kubectl-convert.sha256
   ```

## {{% heading "whatsnext" %}}

{{< include "included/kubectl-whats-next.md" >}}
