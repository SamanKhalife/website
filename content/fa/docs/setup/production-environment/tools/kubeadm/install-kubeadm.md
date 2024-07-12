---
title: نصب kubeadm
content_type: task
weight: 10
card:
  name: setup
  weight: 40
  title: نصب ابزار kubeadm
---

<!-- overview -->

<img src="/images/kubeadm-stacked-color.png" align="right" width="150px"></img>
این صفحه نحوه نصب ابزار `kubeadm` را نشان می‌دهد.
برای اطلاعات بیشتر در مورد ایجاد خوشه با استفاده از kubeadm بعد از انجام فرایند نصب،
صفحه [ایجاد خوشه با kubeadm](/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) را ببینید.

{{< doc-versions-list "راهنمای نصب" >}}

## {{% heading "پیش‌نیازها" %}}

* میزبان Linux سازگار. پروژه Kubernetes دستورالعمل‌های عمومی برای توزیع‌های Debian و Red Hat را فراهم می‌کند و توزیع‌های بدون مدیر بسته.
* حداقل ۲ گیگابایت RAM برای هر دستگاه (کمتر از این مقدار باقی می‌ماند).
* حداقل ۲ CPU.
* اتصال شبکه کامل بین تمام دستگاه‌ها در خوشه (شبکه عمومی یا خصوصی امکان‌پذیر است).
* نام میزبان، آدرس MAC و product_uuid یکتا برای هر نود. برای جزئیات بیشتر [اینجا](#verify-mac-address) را ببینید.
* برخی پورت‌های مشخص بر روی دستگاه‌های شما باز شده باشد. برای جزئیات بیشتر [اینجا](#check-required-ports) را ببینید.
* پیکربندی Swap. رفتار پیش‌فرض kubelet با شناسایی حافظه Swap چیزی است. برای جزئیات بیشتر [مدیریت حافظه Swap](/docs/concepts/architecture/nodes/#swap-memory) را ببینید.
  * شما **باید** Swap را غیرفعال کنید اگر kubelet به درستی پیکربندی نشده است تا از swap استفاده کنید. به عنوان مثال، `sudo swapoff -a`
    معکوس از معمولی اجرا خواهد شد. برای ایجاد این تغییر بر روی فایل‌های پیکربندی مانند `/etc/fstab`, `systemd.swap`, بسته به اینکه چگونه در سیستم شما پیکربندی شده است، مطمئن شوید که swap غیرفعال شده است.

{{< note >}}
نصب kubeadm از طریق فایل‌های باینری است که از پیوندهای پویایی استفاده می‌کنند و فرض می‌شود که سیستم مقصد شما `glibc` را فراهم می‌کند.
این فرضیه قابل قبول است بر روی بسیاری از توزیع‌های Linux (شامل Debian، Ubuntu، Fedora، CentOS و غیره)
اما این حالت همیشه در توزیع‌های سفارشی و سبک که به طور پیش‌فرض `glibc` را شامل نمی‌شوند، مثل Alpine Linux، درست نیست.
پیش‌فرض این است که توزیع امکان استفاده از `glibc` یا [لایه سازگاری](https://wiki.alpinelinux.org/wiki/Running_glibc_programs)
را فراهم می‌کند که نمادهای مورد انتظار را فراهم می‌کند.
{{< /note >}}

<!-- steps -->

## اعتبار MAC address و product_uuid برای هر نود را چک کنید {#verify-mac-address}

* شما می‌توانید آدرس MAC رابط‌های شبکه را با دستور `ip link` یا `ifconfig -a` بگیرید.
* product_uuid می‌تواند با استفاده از دستور `sudo cat /sys/class/dmi/id/product_uuid` بررسی شود.

احتمال زیادی وجود دارد که دستگاه‌های سخت‌افزاری آدرس‌های منحصربه‌فردی داشته باشند، هرچند برخی از ماشین‌های مجازی ممکن است دارای مقادیر یکسان باشند.
Kubernetes از این مقادیر برای شناسایی یکتای نودها در خوشه استفاده می‌کند.
اگر این مقادیر برای هر نود یکتا نباشند، فرآیند نصب ممکن است به [شکست](https://github.com/kubernetes/kubeadm/issues/31) بخورد.

## بررسی آداپتورهای شبکه

اگر بیش از یک آداپتور شبکه دارید و مؤلفه‌های Kubernetes در مسیر پیش‌فرض قابل دسترسی نیستند، توصیه می‌کنیم که مسیر IP (های) را اضافه کنید، تا آدرس‌های خوشه Kubernetes از طریق آداپتور مناسب روند رود.

## بررسی پورت‌های مورد نیاز {#check-required-ports}

این [پورت‌های مورد نیاز](/docs/reference/networking/ports-and-protocols/)
باید برای ارتباط مؤلفه‌های Kubernetes با یکدیگر باز باشند.
شما می‌توانید از ابزارهایی مانند [netcat](https://netcat.sourceforge.net) برای بررسی وجود یک پورت باز استفاده کنید. برای مثال:

```shell
nc 127.0.0.1 6443 -v
```

افزونه شبکه پودی که استفاده می‌کنید، ممکن است نیاز به پورت‌های خاصی داشته باشد.
از آنجا که این با هر افزونه شبکه پود متفاوت است، لطفاً به [مستندات](/docs/setup/production-environment/container-runtimes/) برای اطلاعات بیشتر مراجعه کنید.

## نصب یک موتور اجرایی ظرفیت {#installing-runtime}



برای اجرای کانتینرها در پادها، Kubernetes از یک
{{< glossary_tooltip term_id="container-runtime" text="container runtime" >}}
استفاده می‌کند.

به طور پیش‌فرض، Kubernetes از
{{< glossary_tooltip term_id="cri" text="Container Runtime Interface">}} (CRI)
برای ارتباط با موتور اجرایی کانتینر انتخابی شما استفاده می‌کند.

اگر شما یک موتور اجرایی مشخص نکنید، kubeadm به طور خودکار تلاش می‌کند تا با اسکن لیستی از نقاط پایانی شناخته شده، موتور اجرایی کانتینر نصب شده را شناسایی کند.

اگر چندین یا هیچ موتور اجرایی کانتینری شناسایی نشود، kubeadm خطا می‌دهد و درخواست می‌کند که شما مشخص کنید کدام یک را می‌خواهید استفاده کنید.

برای اطلاعات بیشتر به [موتورهای اجرایی کانتینر](/docs/setup/production-environment/container-runtimes/)
مراجعه کنید.

{{< note >}}
موتور Docker Engine اجرای [CRI](/docs/concepts/architecture/cri/)
را پیاده‌سازی نمی‌کند که یک نیاز برای کار کردن موتور اجرایی کانتینر با Kubernetes است.
به همین دلیل، باید یک خدمات اضافی [cri-dockerd](https://mirantis.github.io/cri-dockerd/)
نصب شود. cri-dockerd یک پروژه بر اساس پشتیبانی از Docker Engine مدیریت شده داخلی بود که [حذف شد](/dockershim) از kubelet در نسخه 1.24.
{{< /note >}}

جداول زیر شامل نقاط پایانی شناخته شده برای سیستم‌عامل‌های پشتیبانی شده است:

{{< tabs name="container_runtime" >}}
{{% tab name="Linux" %}}

{{< table caption="موتورهای اجرایی کانتینر Linux" >}}
| موتور                             | مسیر سوکت دامنه Unix                   |
|------------------------------------|------------------------------------------|
| containerd                         | `unix:///var/run/containerd/containerd.sock` |
| CRI-O                              | `unix:///var/run/crio/crio.sock`         |
| Docker Engine (استفاده از cri-dockerd)  | `unix:///var/run/cri-dockerd.sock`   |
{{< /table >}}

{{% /tab %}}

{{% tab name="Windows" %}}

{{< table caption="موتورهای اجرایی کانتینر Windows" >}}
| موتور                             | مسیر لوله نام‌گذاری شده Windows                   |
|------------------------------------|------------------------------------------------------|
| containerd                         | `npipe:////./pipe/containerd-containerd`             |
| Docker Engine (استفاده از cri-dockerd)  | `npipe:////./pipe/cri-dockerd`                 |
{{< /table >}}

{{% /tab %}}
{{< /tabs >}}

## نصب kubeadm، kubelet و kubectl

شما باید این بسته‌ها را بر روی تمام دستگاه‌های خود نصب کنید:

* `kubeadm`: دستوری برای ایجاد خوشه.
* `kubelet`: اجزایی که بر روی تمام دستگاه‌های خوشه شما اجرا می‌شود و کارهایی مانند شروع پادها و کانتینرها را انجام می‌دهد.
* `kubectl`: ابزار خط فرمان برای ارتباط با خوشه شما.

kubeadm **نصب یا مدیریت** `kubelet` یا `kubectl` را برای شما انجام نمی‌دهد، بنابراین باید اطمینان حاصل کنید که آن‌ها با نسخه پلتفرم کنترل Kubernetes که می‌خواهید kubeadm برای شما نصب کند، همخوانی داشته باشند. در غیر این صورت، خطای نسخه‌پیچی ممکن است اتفاق بیافتد که منجر به رفتارهای غیرمنتظره و اشکال‌زا می‌شود. با این حال، _یک_ نسخه‌پیچی کوچک بین kubelet و کنترل پلتفرم پشتیبانی می‌شود، اما نسخه kubelet هرگز نباید از نسخه سرور API بیشتر شود. به عنوان مثال، kubelet با نسخه 1.7.0 کاملاً سازگار با سرور API نسخه 1.8.0 است، اما واگرا نه.

برای اطلاعات بیشتر در مورد نسخه‌پیچی‌ها، به موارد زیر مراجعه کنید:

* سیاست [نسخه و نسخه‌پیچی Kubernetes](/docs/setup/release/version-skew-policy/)
* سیاست نسخه‌پیچی خاص [kubeadm](/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#version-skew-policy)

{{< warning >}}
این دستورالعمل‌ها تمام بسته‌های Kubernetes را از هر ارتقاء سیستمی حذف می‌کند.
این به این دلیل است که kubeadm و Kubernetes نیاز به [توجه خاص برای ارتقاء دارند](/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/).
{{</ warning >}}

برای اطلاعات بیشتر در مورد نسخه‌پیچی، به موارد زیر مراجعه کنید:

* [سیاست نسخه و نسخه‌پیچی Kubernetes](/docs/setup/release/version-skew-policy/)
* [سیاست نسخه‌پیچی خاص kubeadm](/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#version-skew-policy)

{{% legacy-repos-deprecation %}}

{{< note >}}
برای هر نسخه کوچک Kubernetes، یک مخزن بسته اختصاصی وجود دارد. اگر می‌خواهید یک نسخه کوچک دیگر به جز v{{< skew currentVersion >}} را نصب کنید، لطفاً راهنمای نصب را برای نسخه کوچک مورد نظر خود ببینید.
{{< /note >}}

{{< tabs name="k8s_install" >}}
{{% tab name="توزیع‌های مبتنی بر Debian" %}}

این دستورالعمل‌ها برای Kubernetes v{{< skew currentVersion >}} هستند.

1. به‌روزرسانی شاخص بسته `apt` و نصب بسته‌های مورد نیاز برای استفاده از مخزن `apt` Kubernetes:

   ```shell
   sudo apt-get update
   # بسته apt-transport-https ممکن است یک بسته پوچ باشد؛ در این صورت می‌توانید آن بسته را بپرشینید
   sudo apt-get install -y apt-transport-https ca-certificates curl gpg
   ```

2. دانلود کلید امضای عمومی برای مخازن بسته Kubernetes.
   همان کلید امضایی که برای همه مخازن استفاده می‌شود، بنابراین می‌توانید نسخه را در URL نادیده بگیرید:

   ```shell
   # اگر دایرکتوری `/etc/apt/keyrings` وجود ندارد ، قبل از دستور curl ایجاد شود. اطلاعات بیشتر را در زیر بخوانید.
   # sudo mkdir -p -m 755 /etc/apt/keyrings
   curl -fsSL https://pkgs.k8s.io/core:/stable:/{{< param "version" >}}/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
   ```

{{< note >}}
در نسخه‌های قدیمی‌تر از Debian 12 و Ubuntu 22.04، دایرکتوری `/etc/apt/keyrings` به طور پیش‌فرض وجود ندارد، و باید قبل از دستور curl ایجاد شود.
{{< /note >}}

3. اضافه کردن مخزن `apt` Kubernetes مناسب. لطفاً توجه داشته باشید که این مخزن فقط برای Kubernetes {{< skew currentVersion >}} بسته‌ها دارد؛ برای نسخه‌های کوچک‌تر Kubernetes، نیاز است نسخه کوچک Kubernetes را در URL تغییر دهید تا با نسخه کوچک مورد نظر خود همخوانی داشته باشد
   (همچنین باید بررسی کنید که شما در حال خواندن مستندات نسخه Kubernetes هستید که قصد نصب آن را دارید).

   ```shell
   # این تنظیمات هر تنظیم موجود در /etc/apt/sources.list.d/kubernetes.list را بازنویسی می‌کند
   echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/{{< param "version" >}}/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
   ```

4. به‌روزرسانی شاخص بسته `apt`، نصب kubelet، kubeadm و kubectl و نصب نسخه آن‌ها:

   ```shell
   sudo apt-get update
   sudo apt-get install -y kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
   ```

5. (اختیاری) فعال

 کردن خدمات kubelet قبل از اجرای kubeadm:

   ```shell
   sudo systemctl enable --now kubelet
   ```

{{% /tab %}}
{{% tab name="توزیع‌های مبتنی بر Red Hat" %}}

1. SELinux را به حالت `permissive` تنظیم کنید:

   این دستورالعمل‌ها برای Kubernetes {{< skew currentVersion >}} هستند.

   ```shell
   # SELinux را به حالت permissive (که به طور موثر آن را غیرفعال می‌کند) تنظیم کنید
   sudo setenforce 0
   sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
   ```

{{< caution >}}
- تنظیم SELinux به حالت permissive با اجرای `setenforce 0` و `sed ...`
  به طور موثر آن را غیرفعال می‌کند. این اقدام برای اجازه دسترسی کانتینرها به فایل سیستم میزبان لازم است؛ به عنوان مثال، برخی از پلاگین‌های شبکه خوشه نیاز به آن دارند. شما باید این کار را انجام دهید تا پشتیبانی از SELinux در kubelet بهبود یابد.
- می‌توانید SELinux را فعال بگذارید اگر بدانید چگونه آن را پیکربندی کنید، اما ممکن است نیاز به تنظیماتی داشته باشد که توسط kubeadm پشتیبانی نمی‌شوند.
{{< /caution >}}

2. مخزن `yum` Kubernetes را اضافه کنید. پارامتر `exclude` در تعریف مخزن، مطمئن می‌شود که بسته‌های مربوط به Kubernetes بر اساس اجرای `yum update` بروزرسانی نمی‌شوند زیرا رویه‌ای خاص وجود دارد که باید برای بروزرسانی Kubernetes دنبال شود. لطفاً توجه داشته باشید که این مخزن فقط برای بسته‌های Kubernetes {{< skew currentVersion >}} دارد؛ برای نسخه‌های کوچک‌تر Kubernetes، نیاز است نسخه کوچک Kubernetes را در URL تغییر دهید تا با نسخه کوچک مورد نظر خود همخوانی داشته باشد
   (همچنین باید بررسی کنید که شما در حال خواندن مستندات نسخه Kubernetes هستید که قصد نصب آن را دارید).


   ```shell
   # This overwrites any existing configuration in /etc/yum.repos.d/kubernetes.repo
   cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
   [kubernetes]
   name=Kubernetes
   baseurl=https://pkgs.k8s.io/core:/stable:/{{< param "version" >}}/rpm/
   enabled=1
   gpgcheck=1
   gpgkey=https://pkgs.k8s.io/core:/stable:/{{< param "version" >}}/rpm/repodata/repomd.xml.key
   exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
   EOF
   ```

3. نصب kubelet، kubeadm و kubectl:

   ```shell
   sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
   ```

4. (اختیاری) فعال کردن خدمت kubelet قبل از اجرای kubeadm:

   ```shell
   sudo systemctl enable --now kubelet
   ```

{{% /tab %}}
{{% tab name="بدون یک مدیر بسته" %}}
نصب افزونه‌های CNI (برای بیشتر شبکه‌های pod ضروری است):

```bash
CNI_PLUGINS_VERSION="v1.3.0"
ARCH="amd64"
DEST="/opt/cni/bin"
sudo mkdir -p "$DEST"
curl -L "https://github.com/containernetworking/plugins/releases/download/${CNI_PLUGINS_VERSION}/cni-plugins-linux-${ARCH}-${CNI_PLUGINS_VERSION}.tgz" | sudo tar -C "$DEST" -xz
```

تعریف دایرکتوری برای دانلود فایل‌های دستوری:

{{< note >}}
متغیر `DOWNLOAD_DIR` باید به یک دایرکتوری قابل نوشتن تنظیم شود.
اگر شما از Flatcar Container Linux استفاده می‌کنید، `DOWNLOAD_DIR="/opt/bin"` را تنظیم کنید.
{{< /note >}}

```bash
DOWNLOAD_DIR="/usr/local/bin"
sudo mkdir -p "$DOWNLOAD_DIR"
```

نصب crictl (برای kubeadm / Kubelet Container Runtime Interface (CRI) ضروری است):

```bash
CRICTL_VERSION="v1.30.0"
ARCH="amd64"
curl -L "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-${ARCH}.tar.gz" | sudo tar -C $DOWNLOAD_DIR -xz
```

نصب `kubeadm`، `kubelet` و افزودن خدمت `kubelet` به systemd:

```bash
RELEASE="$(curl -sSL https://dl.k8s.io/release/stable.txt)"
ARCH="amd64"
cd $DOWNLOAD_DIR
sudo curl -L --remote-name-all https://dl.k8s.io/release/${RELEASE}/bin/linux/${ARCH}/{kubeadm,kubelet}
sudo chmod +x {kubeadm,kubelet}

RELEASE_VERSION="v0.16.2"
curl -sSL "https://raw.githubusercontent.com/kubernetes/release/${RELEASE_VERSION}/cmd/krel/templates/latest/kubelet/kubelet.service" | sed "s:/usr/bin:${DOWNLOAD_DIR}:g" | sudo tee /usr/lib/systemd/system/kubelet.service
sudo mkdir -p /usr/lib/systemd/system/kubelet.service.d
curl -sSL "https://raw.githubusercontent.com/kubernetes/release/${RELEASE_VERSION}/cmd/krel/templates/latest/kubeadm/10-kubeadm.conf" | sed "s:/usr/bin:${DOWNLOAD_DIR}:g" | sudo tee /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
```

{{< note >}}
لطفاً به یاد داشته باشید که در بخش [قبل از شروع](#before-you-begin) برای توزیع‌های Linux که شامل `glibc` نیستند، یک یادداشت را بخوانید.
{{< /note >}}

نصب `kubectl` با دنبال کردن دستورالعمل‌ها در [صفحه نصب ابزارها](/docs/tasks/tools/#kubectl).

اختیاری، فعال کردن خدمت kubelet قبل از اجرای kubeadm:

```bash
sudo systemctl enable --now kubelet
```

{{< note >}}
توزیع Flatcar Container Linux، دایرکتوری `/usr` را به عنوان یک فایل سیستم خواندنی تنظیم می‌کند.
قبل از راه‌اندازی خوشه خود، شما باید مراحل اضافی برای پیکربندی یک دایرکتوری قابل نوشتن را انجام دهید.
برای یادگیری نحوه تنظیم یک دایرکتوری قابل نوشتن، [راهنمای رفع اشکال Kubeadm](/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/#usr-mounted-read-only) را ببینید.
{{< /note >}}
{{% /tab %}}
{{< /tabs >}}

kubelet در حال حاضر هر چند ثانیه یک‌بار راه‌اندازی می‌شود، زیرا در یک حلقه خرابی منتظر است kubeadm به آن بگوید چه باید انجام دهد.

## پیکربندی درایور cgroup

هر دو runtime container و kubelet دارای خاصیتی به نام
["درایور cgroup"](/docs/setup/production-environment/container-runtimes/#cgroup-drivers) هستند که برای مدیریت cgroups در ماشین‌های Linux اهمیت دارد.

{{< warning >}}
همخوانی درایور cgroup بین runtime container و kubelet الزامی است یا در غیر این صورت فرآیند kubelet شکست خواهد خورد.

برای اطلاعات بیشتر در مورد پیکربندی درایور cgroup، [صفحه پیکربندی درایور cgroup](/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/) را مشاهده کنید.
{{< /warning >}}

## رفع اشکال

اگر با مشکلاتی در kubeadm روبرو شدید، لطفاً [مستندات رفع اشکال](/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/) ما را مشاهده کنید.

## {{% heading "whatsnext" %}}

* [استفاده از kubeadm برای ایجاد خوشه](/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
