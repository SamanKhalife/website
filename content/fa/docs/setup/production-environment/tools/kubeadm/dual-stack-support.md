---
title: پشتیبانی از دو استک با kubeadm
content_type: task
weight: 100
min-kubernetes-server-version: 1.21
---

<!-- overview -->

{{< feature-state for_k8s_version="v1.23" state="stable" >}}

خوشه Kubernetes شما شامل شبکه‌بندی [دو استک](/docs/concepts/services-networking/dual-stack/)
است، به این معنی که شبکه خوشه به شما اجازه می‌دهد از هر دو خانواده آدرس استفاده کنید.
در یک خوشه، صفحه کنترل می‌تواند یک آدرس IPv4 و یک آدرس IPv6 را به یک {{< glossary_tooltip text="Pod" term_id="pod" >}} یا یک {{< glossary_tooltip text="Service" term_id="service" >}} اختصاص دهد.

<!-- body -->

## {{% heading "پیش‌نیازها" %}}

شما باید ابزار {{< glossary_tooltip text="kubeadm" term_id="kubeadm" >}} را نصب کرده باشید،
طبق مراحل [نصب kubeadm](/docs/setup/production-environment/tools/kubeadm/install-kubeadm/).

برای هر سروری که می‌خواهید به عنوان یک {{< glossary_tooltip text="node" term_id="node" >}} استفاده کنید،
مطمئن شوید که اجازه ارسال IPv6 را دارد. در Linux، می‌توانید این کار را با اجرای دستور
`sysctl -w net.ipv6.conf.all.forwarding=1` با حساب کاربری root بر روی هر سرور انجام دهید.

شما باید برای استفاده از آنها یک محدوده آدرس IPv4 و IPv6 داشته باشید. اپراتورهای خوشه معمولاً
از محدوده‌های آدرس خصوصی برای IPv4 استفاده می‌کنند. برای IPv6، یک اپراتور خوشه معمولاً
یک بلوک آدرس یکانی گلوبال از درون `2000::/3` انتخاب می‌کند، با استفاده از یک محدوده که به او اختصاص داده شده است.
شما نیازی ندارید که محدوده آدرس IP خوشه را به اینترنت عمومی مسیردهی کنید.

اندازه تخصیص آدرس IP باید برای تعداد Podها و Servicesی که قصد اجرای آنها را دارید، مناسب باشد.

{{< note >}}
اگر شما یک خوشه موجود را با دستور `kubeadm upgrade` ارتقا می‌دهید،
`kubeadm` امکان انجام تغییرات در محدوده آدرس IP پاد (CIDR خوشه) و یا محدوده آدرس سرویس خوشه (CIDR سرویس) را پشتیبانی نمی‌کند.
{{< /note >}}

### ایجاد یک خوشه دو استک

برای ایجاد یک خوشه دو استک با `kubeadm init` می‌توانید آرگومان‌های خط فرمان را
مشابه مثال زیر ارسال کنید:

```shell
# این محدوده‌های آدرس مثال هستند
kubeadm init --pod-network-cidr=10.244.0.0/16,2001:db8:42:0::/56 --service-cidr=10.96.0.0/16,2001:db8:42:1::/112
```

برای شفاف‌سازی موارد، یک فایل پیکربندی kubeadm
[مثال](/docs/reference/config-api/kubeadm-config.v1beta3/)
`kubeadm-config.yaml` برای یک نود کنترل اصلی دو استک ارائه شده است.

```yaml
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
networking:
  podSubnet: 10.244.0.0/16,2001:db8:42:0::/56
  serviceSubnet: 10.96.0.0/16,2001:db8:42:1::/112
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: "10.100.0.1"
  bindPort: 6443
nodeRegistration:
  kubeletExtraArgs:
    node-ip: 10.100.0.2,fd00:1:2:3::2
```

در InitConfiguration، `advertiseAddress` آدرس IP را مشخص می‌کند که سرور API آن را برای شنوایی تبلیغ می‌کند.
مقدار `advertiseAddress` معادل پرچم `--apiserver-advertise-address` در `kubeadm init` است.

برای شروع kubeadm به اجرای نود کنترل اصلی دو استک:

```shell
kubeadm init --config=kubeadm-config.yaml
```

پرچم‌های kube-controller-manager `--node-cidr-mask-size-ipv4|--node-cidr-mask-size-ipv6`
با مقادیر پیش‌فرض تنظیم شده‌اند. برای مشاهده [پیکربندی استک دو آدرس IPv4/IPv6](/docs/concepts/services-networking/dual-stack#configure-ipv4-ipv6-dual-stack) را ببینید.

{{< note >}}
پرچم `--apiserver-advertise-address` از پشتیبانی از دو استک پشتیبانی نمی‌کند.
{{< /note >}}

### اضافه شدن یک نود به خوشه دو استک

پیش از پیوستن به یک نود، مطمئن شوید که نود دارای یک رابط شبکه IPv6 است و اجازه ارسال IPv6 را دارد.

در ادامه، یک فایل پیکربندی kubeadm [مثال](/docs/reference/config-api/kubeadm-config.v1beta3/)
`kubeadm-config.yaml` برای پیوستن یک نود کارگر به خوشه ارائه شده است.

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: JoinConfiguration
discovery:
  bootstrapToken:
    apiServerEndpoint: 10.100.0.1:6443
    token: "clvldh.vjjwg16ucnhp94qr"
    caCertHashes:
    - "sha256:a4863cde706cfc580a439f842cc65d5ef112b7b2be31628513a9881cf0d9fe0e"
    # تغییر دادن اطلاعات احراز هویت بالا برای همخ

وانی با توکن و گواهی CA واقعی خوشه شما
nodeRegistration:
  kubeletExtraArgs:
    node-ip: 10.100.0.3,fd00:1:2:3::3
```

همچنین، یک فایل پیکربندی kubeadm [مثال](/docs/reference/config-api/kubeadm-config.v1beta3/)
`kubeadm-config.yaml` برای پیوستن یک نود کنترل اصلی دیگر به خوشه ارائه شده است.

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: JoinConfiguration
controlPlane:
  localAPIEndpoint:
    advertiseAddress: "10.100.0.2"
    bindPort: 6443
discovery:
  bootstrapToken:
    apiServerEndpoint: 10.100.0.1:6443
    token: "clvldh.vjjwg16ucnhp94qr"
    caCertHashes:
    - "sha256:a4863cde706cfc580a439f842cc65d5ef112b7b2be31628513a9881cf0d9fe0e"
    # تغییر دادن اطلاعات احراز هویت بالا برای همخوانی با توکن و گواهی CA واقعی خوشه شما
nodeRegistration:
  kubeletExtraArgs:
    node-ip: 10.100.0.4,fd00:1:2:3::4

```

در JoinConfiguration.controlPlane، `advertiseAddress` آدرس IP را مشخص می‌کند که سرور API آن را برای شنوایی تبلیغ می‌کند.
مقدار `advertiseAddress` معادل پرچم `--apiserver-advertise-address` در `kubeadm join` است.

```shell
kubeadm join --config=kubeadm-config.yaml
```

### ایجاد یک خوشه تک استک

{{< note >}}
پشتیبانی از دو استک به این معنی نیست که شما باید از آدرس‌دهی دو استک استفاده کنید.
می‌توانید یک خوشه تک استک را که دارای ویژگی شبکه دو استک است، پیاده‌سازی کنید.
{{< /note >}}

برای شفاف‌سازی موارد، یک فایل پیکربندی kubeadm
[مثال](/docs/reference/config-api/kubeadm-config.v1beta3/)
`kubeadm-config.yaml` برای نود کنترل تک استک ارائه شده است.

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
networking:
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/16
```

## {{% heading "چه کاری بعدا" %}}

* [اعتبارسنجی شبکه دو استک IPv4/IPv6](/docs/tasks/network/validate-dual-stack)
* درباره [شبکه دو استک](/docs/concepts/services-networking/dual-stack/) در خوشه بیشتر بخوانید
* بیشتر در مورد [فرمت پیکربندی kubeadm](/docs/reference/config-api/kubeadm-config.v1beta3/) بیاموزید
