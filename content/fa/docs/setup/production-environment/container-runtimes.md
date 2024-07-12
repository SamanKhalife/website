---
reviewers:
- vincepri
- bart0sh
title: ران‌تایم‌های کانتینری
content_type: concept
weight: 20
---
<!-- overview -->

{{% dockershim-removal %}}

برای اجرای پادها در هر نود کلاستر، نیاز به نصب یک
{{< glossary_tooltip text="ران‌تایم کانتینری" term_id="container-runtime" >}}
دارید. این صفحه جزئیات مربوطه و وظایف مرتبط برای راه‌اندازی نودها را توضیح می‌دهد.

Kubernetes {{< skew currentVersion >}} نیاز دارد که از ران‌تایمی استفاده کنید که با
{{< glossary_tooltip term_id="cri" text="رابط ران‌تایم کانتینر">}} (CRI) سازگار باشد.

برای اطلاعات بیشتر، بخش [پشتیبانی از نسخه‌های CRI](#cri-versions) را ببینید.

این صفحه یک مرور کلی از چگونگی استفاده از چند ران‌تایم کانتینری رایج با Kubernetes فراهم می‌کند.

- [containerd](#containerd)
- [CRI-O](#cri-o)
- [Docker Engine](#docker)
- [Mirantis Container Runtime](#mcr)

{{< note >}}
نسخه‌های Kubernetes قبل از v1.24 شامل یک ادغام مستقیم با Docker Engine بودند که از یک مؤلفه به نام _dockershim_ استفاده می‌کردند. این ادغام ویژه دیگر بخشی از Kubernetes نیست (این حذف در
[اعلامیه](/blog/2020/12/08/kubernetes-1-20-release-announcement/#dockershim-deprecation)
در نسخه v1.20 اعلام شد).
می‌توانید
[بررسی کنید که آیا حذف Dockershim بر شما تأثیر می‌گذارد](/docs/tasks/administer-cluster/migrating-from-dockershim/check-if-dockershim-removal-affects-you/)
تا بفهمید که این حذف چگونه ممکن است بر شما تأثیر بگذارد. برای آموختن در مورد مهاجرت از استفاده از dockershim، به
[مهاجرت از dockershim](/docs/tasks/administer-cluster/migrating-from-dockershim/)
مراجعه کنید.

اگر از نسخه‌ای از Kubernetes غیر از v{{< skew currentVersion >}} استفاده می‌کنید،
مستندات آن نسخه را بررسی کنید.
{{< /note >}}

<!-- body -->
## نصب و پیکربندی پیش‌نیازها

### پیکربندی شبکه

به طور پیش‌فرض، کرنل لینوکس اجازه نمی‌دهد که بسته‌های IPv4 بین رابط‌ها مسیریابی شوند.
بیشتر پیاده‌سازی‌های شبکه کلاستر Kubernetes این تنظیم را (در صورت نیاز) تغییر می‌دهند، اما برخی ممکن است انتظار داشته باشند که مدیر سیستم این کار را برای آن‌ها انجام دهد. (برخی ممکن است انتظار داشته باشند که دیگر پارامترهای sysctl تنظیم شوند، ماژول‌های کرنل بارگذاری شوند و غیره؛ مستندات پیاده‌سازی شبکه خاص خود را مشورت کنید.)

### فعال‌سازی مسیریابی بسته‌های IPv4 {#prerequisite-ipv4-forwarding-optional}

برای فعال‌سازی دستی مسیریابی بسته‌های IPv4:

```bash
# پارامترهای sysctl مورد نیاز توسط راه‌اندازی، پارامترها پس از ریبوت باقی می‌مانند
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# اعمال پارامترهای sysctl بدون ریبوت
sudo sysctl --system
```

تأیید کنید که `net.ipv4.ip_forward` روی 1 تنظیم شده است:

```bash
sysctl net.ipv4.ip_forward
```

## درایورهای cgroup

در لینوکس، {{< glossary_tooltip text="گروه‌های کنترلی" term_id="cgroup" >}}
برای محدود کردن منابع اختصاص داده شده به فرآیندها استفاده می‌شوند.

هر دو {{< glossary_tooltip text="kubelet" term_id="kubelet" >}} و
ران‌تایم کانتینری زیرین نیاز به ارتباط با گروه‌های کنترلی برای اجرای
[مدیریت منابع برای پادها و کانتینرها](/docs/concepts/configuration/manage-resources-containers/)
و تنظیم منابعی مانند درخواست‌ها و محدودیت‌های CPU/Memory دارند. برای ارتباط با گروه‌های کنترلی، kubelet و ران‌تایم کانتینری نیاز به استفاده از یک *درایور cgroup* دارند.
بسیار مهم است که kubelet و ران‌تایم کانتینری از همان درایور cgroup استفاده کنند و به همان صورت پیکربندی شوند.

دو درایور cgroup موجود هستند:

* [`cgroupfs`](#cgroupfs-cgroup-driver)
* [`systemd`](#systemd-cgroup-driver)

### درایور cgroupfs {#cgroupfs-cgroup-driver}

درایور `cgroupfs` [درایور پیش‌فرض cgroup در kubelet](/docs/reference/config-api/kubelet-config.v1beta1) است.
وقتی که از درایور `cgroupfs` استفاده می‌شود، kubelet و ران‌تایم کانتینری به طور مستقیم با
سیستم فایل cgroup برای پیکربندی cgroupها ارتباط برقرار می‌کنند.

درایور `cgroupfs` **توصیه نمی‌شود** زمانی که
[systemd](https://www.freedesktop.org/wiki/Software/systemd/) سیستم init است زیرا systemd یک مدیر cgroup واحد در
سیستم را انتظار دارد. علاوه بر این، اگر از [cgroup v2](/docs/concepts/architecture/cgroups) استفاده می‌کنید، از درایور cgroup `systemd` به جای `cgroupfs` استفاده کنید.

### درایور cgroup systemd {#systemd-cgroup-driver}

وقتی [systemd](https://www.freedesktop.org/wiki/Software/systemd/) به عنوان سیستم init
برای یک توزیع لینوکس انتخاب می‌شود، فرآیند init یک گروه کنترلی (cgroup) ریشه تولید و مصرف می‌کند
و به عنوان مدیر cgroup عمل می‌کند.

systemd یک یکپارچگی تنگاتنگ با cgroup‌ها دارد و یک cgroup برای هر واحد systemd اختصاص می‌دهد.
در نتیجه، اگر از systemd به عنوان سیستم init با درایور `cgroupfs`
استفاده کنید، سیستم دو مدیر cgroup مختلف دریافت می‌کند.

دو مدیر cgroup منجر به دو دیدگاه از منابع موجود و استفاده شده در
سیستم می‌شود. در برخی موارد، نودهایی که برای استفاده از `cgroupfs` برای
kubelet و ران‌تایم کانتینری پیکربندی شده‌اند، اما از systemd برای بقیه فرآیندها استفاده می‌کنند، تحت فشار منابع ناپایدار می‌شوند.

راهکار برای کاهش این ناپایداری این است که systemd به عنوان درایور cgroup برای
kubelet و ران‌تایم کانتینری وقتی systemd سیستم init منتخب است، استفاده شود.

برای تنظیم systemd به عنوان درایور cgroup، گزینه
[`KubeletConfiguration`](/docs/tasks/administer-cluster/kubelet-config-file/)
از `cgroupDriver` را ویرایش کرده و به `systemd` تنظیم کنید. برای مثال:

```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
...
cgroupDriver: systemd
```

{{< note >}}
از نسخه v1.22 به بعد، هنگام ایجاد یک کلاستر با kubeadm، اگر کاربر فیلد
`cgroupDriver` را تحت `KubeletConfiguration` تنظیم نکند، kubeadm به طور پیش‌فرض آن را به `systemd` تنظیم می‌کند.
{{< /note >}}

در Kubernetes v1.28، با فعال بودن دروازه ویژگی `KubeletCgroupDriverFromCRI`
[feature gate](/docs/reference/command-line-tools-reference/feature-gates/)
و یک ران‌تایم کانتینری که از `RuntimeConfig` CRI RPC پشتیبانی می‌کند،
kubelet به طور خودکار درایور cgroup مناسب را از ران‌تایم تشخیص می‌دهد و تنظیم `cgroupDriver` در پیکربندی kubelet را نادیده می‌گیرد.

اگر systemd را به عنوان درایور cgroup برای kubelet پیکربندی کنید، باید
systemd را به عنوان درایور cgroup برای ران‌تایم کانتینری نیز پیکربندی کنید. به
مستندات ران‌تایم کانتینری خود برای دستورالعمل‌ها مراجعه کنید. برای مثال:

*  [containerd](#containerd-systemd)
*  [CRI-O](#cri-o)

{{< caution >}}
تغییر درایور cgroup یک نود که به کلاستر پیوسته است، یک عملیات حساس است.
اگر kubelet پادهایی را با استفاده از معناشناسی‌های یک درایور cgroup ایجاد کرده باشد، تغییر ران‌تایم
کانتینری به یک درایور cgroup دیگر می‌تواند باعث بروز خطاها در هنگام تلاش برای بازسازی سندباکس پاد برای چنین پادهای موجود شود. ری‌استارت کردن kubelet ممکن است این خطاها را حل نکند.

اگر شما خودکاری دارید که این کار را ممکن می‌سازد، نود را با استفاده از پیکربندی به‌روزرسانی شده جایگزین کنید، یا آن را با استفاده از خودکارسازی مجدداً نصب کنید.
{{< /caution >}}


### مهاجرت به درایور `systemd` در کلاسترهای مدیریت شده توسط kubeadm

اگر می‌خواهید به درایور cgroup `systemd` در کلاسترهای مدیریت شده توسط kubeadm موجود مهاجرت کنید،
[پیکربندی درایور cgroup](/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/) را دنبال کنید.

## پشتیبانی از نسخه‌های CRI {#cri-versions}

ران‌تایم کانتینری شما باید حداقل از v1alpha2 رابط ران‌تایم کانتینری پشتیبانی کند.

Kubernetes [از نسخه v1.26](/blog/2022/11/18/upcoming-changes-in-kubernetes-1-26/#cri-api-removal)
_فقط با_ نسخه v1 API CRI کار می‌کند. نسخه‌های قبلی به طور پیش‌فرض به نسخه v1 می‌روند، اما اگر یک ران‌تایم کانتینری از API v1 پشتیبانی نکند، kubelet به
استفاده از API v1alpha2 (که منسوخ شده است) به عنوان جایگزین برمی‌گردد.

## ران‌تایم‌های کانتینری

{{% thirdparty-content %}}

### containerd

این بخش مراحل لازم برای استفاده از containerd به عنوان ران‌تایم CRI را توضیح می‌دهد.

برای نصب containerd روی سیستم خود، دستورالعمل‌های
[شروع به کار با containerd](https://github.com/containerd/containerd/blob/main/docs/getting-started.md) را دنبال کنید.
پس از ایجاد یک فایل پیکربندی `config.toml` معتبر به این مرحله برگردید.

{{< tabs name="یافتن فایل config.toml خود" >}}
{{% tab name="لینوکس" %}}
می‌توانید این فایل را در مسیر `/etc/containerd/config.toml` پیدا کنید.
{{% /tab %}}
{{% tab name="ویندوز" %}}
می‌توانید این فایل را در مسیر `C:\Program Files\containerd\config.toml` پیدا کنید.
{{% /tab %}}
{{< /tabs >}}

در لینوکس، سوکت پیش‌فرض CRI برای containerd `/run/containerd/containerd.sock` است.
در ویندوز، نقطه پایانی پیش‌فرض CRI `npipe://./pipe/containerd-containerd` است.

#### پیکربندی درایور cgroup `systemd` {#containerd-systemd}

برای استفاده از درایور cgroup `systemd` در `/etc/containerd/config.toml` با `runc`، تنظیم کنید

```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

درایور cgroup `systemd` توصیه می‌شود اگر از [cgroup v2](/docs/concepts/architecture/cgroups) استفاده می‌کنید.

{{< note >}}
اگر containerd را از یک بسته نصب کردید (برای مثال، RPM یا `.deb`)، ممکن است
که پلاگین یکپارچه‌سازی CRI به طور پیش‌فرض غیرفعال باشد.

برای استفاده از containerd با Kubernetes نیاز به پشتیبانی CRI دارید. مطمئن شوید که `cri`
در لیست `disabled_plugins` در فایل `/etc/containerd/config.toml` قرار نگرفته است؛
اگر تغییری در آن فایل ایجاد کردید، همچنین containerd را ری‌استارت کنید.

اگر پس از نصب اولیه کلاستر یا پس از نصب یک CNI با حلقه‌های کرشی کانتینر مواجه شدید،
پیکربندی containerd که با بسته ارائه شده ممکن است پارامترهای ناسازگاری را شامل شود. در نظر بگیرید که پیکربندی containerd را با `containerd config default > /etc/containerd/config.toml` همانطور که در
[getting-started.md](https://github.com/containerd/containerd/blob/main/docs/getting-started.md#advanced-topics)
ذکر شده است، بازنشانی کرده و سپس پارامترهای پیکربندی ذکر شده در بالا را به‌طور مناسب تنظیم کنید.
{{< /note >}}

اگر این تغییر را اعمال کردید، مطمئن شوید که containerd را ری‌استارت کنید:

```shell
sudo systemctl restart containerd
```

هنگام استفاده از kubeadm، به صورت دستی
[پیکربندی درایور cgroup برای kubelet](/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/#configuring-the-kubelet-cgroup-driver) را تنظیم کنید.

در Kubernetes v1.28، می‌توانید شناسایی خودکار درایور
cgroup را به عنوان یک ویژگی الفا فعال کنید. برای جزئیات بیشتر به [درایور cgroup systemd](#systemd-cgroup-driver)
مراجعه کنید.

#### بازنویسی تصویر سندباکس (pause) {#override-pause-image-containerd}

در [پیکربندی containerd](https://github.com/containerd/containerd/blob/main/docs/cri/config.md) خود می‌توانید
تصویر سندباکس را با تنظیم پیکربندی زیر بازنویسی کنید:

```toml
[plugins."io.containerd.grpc.v1.cri"]
  sandbox_image = "registry.k8s.io/pause:3.2"
```

ممکن است پس از به‌روزرسانی فایل پیکربندی نیاز به ری‌استارت `containerd` داشته باشید: `systemctl restart containerd`.

لطفاً توجه داشته باشید که به‌عنوان یک روش برتر، بهتر است kubelet تصویری که مطابقت دارد را به‌عنوان `pod-infra-container-image` اعلام کند.
اگر پیکربندی نشود، kubelet ممکن است تلاش کند تصویر `pause` را جمع‌آوری کند.
کار بر روی [پین کردن تصویر pause در containerd](https://github.com/containerd/containerd/issues/6352) ادامه دارد
و دیگر نیازی به این تنظیمات در kubelet نیست.

### CRI-O

این بخش شامل مراحل لازم برای نصب CRI-O به‌عنوان ران‌تایم کانتینری است.

برای نصب CRI-O، دستورالعمل‌های [نصب CRI-O](https://github.com/cri-o/cri-o/blob/main/install.md#readme) را دنبال کنید.

#### درایور cgroup

CRI-O به‌طور پیش‌فرض از درایور cgroup systemd استفاده می‌کند، که احتمالاً برای شما مناسب خواهد بود. برای تغییر به درایور cgroup `cgroupfs`، یا فایل
`/etc/crio/crio.conf` را ویرایش کنید یا یک پیکربندی drop-in در
`/etc/crio/crio.conf.d/02-cgroup-manager.conf` قرار دهید، برای مثال:

```toml
[crio.runtime]
conmon_cgroup = "pod"
cgroup_manager = "cgroupfs"
```

باید به تغییر `conmon_cgroup` که باید به مقدار
`pod` تنظیم شود نیز توجه کنید وقتی که از CRI-O با `cgroupfs` استفاده می‌کنید. به‌طور کلی، نگه داشتن
پیکربندی درایور cgroup برای kubelet (معمولاً با kubeadm انجام می‌شود) و CRI-O هم‌زمان ضروری است.

در Kubernetes v1.28، می‌توانید شناسایی خودکار درایور
cgroup را به‌عنوان یک ویژگی الفا فعال کنید. برای جزئیات بیشتر به [درایور cgroup systemd](#systemd-cgroup-driver)
مراجعه کنید.

برای CRI-O، سوکت CRI به‌طور پیش‌فرض `/var/run/crio/crio.sock` است.

#### بازنویسی تصویر سندباکس (pause) {#override-pause-image-cri-o}

در [پیکربندی CRI-O](https://github.com/cri-o/cri-o/blob/main/docs/crio.conf.5.md) خود می‌توانید مقدار
پیکربندی زیر را تنظیم کنید:

```toml
[crio.image]
pause_image="registry.k8s.io/pause:3.6"
```

این گزینه پیکربندی از بارگیری مجدد زنده پیکربندی برای اعمال این تغییر پشتیبانی می‌کند: `systemctl reload crio` یا با ارسال
`SIGHUP` به فرآیند `crio`.

### Docker Engine {#docker}

{{< note >}}
این دستورالعمل‌ها فرض می‌کنند که شما از
[cri-dockerd](https://mirantis.github.io/cri-dockerd/) برای ادغام
Docker Engine با Kubernetes استفاده می‌کنید.
{{< /note >}}

1. روی هر یک از نودهای خود، Docker را برای توزیع لینوکس خود طبق
  [نصب Docker Engine](https://docs.docker.com/engine/install/#server) نصب کنید.

2. [`cri-dockerd`](https://mirantis.github.io/cri-dockerd/usage/install) را نصب کنید، با دنبال کردن دستورالعمل‌ها در بخش نصب مستندات.

برای `cri-dockerd`، سوکت CRI به‌طور پیش‌فرض `/run/cri-dockerd.sock` است.

### Mirantis Container Runtime {#mcr}

[Mirantis Container Runtime](https://docs.mirantis.com/mcr/20.10/overview.html) (MCR) یک ران‌تایم کانتینری تجاری است که قبلاً با نام Docker Enterprise Edition شناخته می‌شد.

می‌توانید از Mirantis Container Runtime با Kubernetes با استفاده از کامپوننت
[`cri-dockerd`](https://mirantis.github.io/cri-dockerd/) متن‌باز، که با MCR همراه است، استفاده کنید.

برای یادگیری بیشتر در مورد نحوه نصب Mirantis Container Runtime،
به [راهنمای استقرار MCR](https://docs.mirantis.com/mcr/20.10/install.html) مراجعه کنید.

واحد systemd به نام `cri-docker.socket` را بررسی کنید تا مسیر سوکت CRI را پیدا کنید.

#### بازنویسی تصویر سندباکس (pause) {#override-pause-image-cri-dockerd-mcr}

آداپتور `cri-dockerd` یک آرگومان خط فرمان برای
مشخص کردن کدام تصویر کانتینری به‌عنوان کانتینر زیرساخت پاد ("تصویر pause") استفاده شود، قبول می‌کند.
آرگومان خط فرمان مورد استفاده `--pod-infra-container-image` است.

## {{% heading "whatsnext" %}}

علاوه بر ران‌تایم کانتینری، کلاستر شما به یک
[پلاگین شبکه](docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-network-model) کاری نیاز خواهد داشت.
