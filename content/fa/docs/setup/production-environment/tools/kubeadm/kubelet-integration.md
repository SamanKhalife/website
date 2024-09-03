---
reviewers:
- sig-cluster-lifecycle
title: پیکربندی هر kubelet در خوشه خود با استفاده از kubeadm
content_type: concept
weight: 80
---

<!-- overview -->

{{% dockershim-removal %}}

{{< feature-state for_k8s_version="v1.11" state="stable" >}}

عمر ابزار CLI kubeadm از [kubelet](/docs/reference/command-line-tools-reference/kubelet) جدا شده است، که یک دیمون است که بر روی هر گره در داخل خوشه Kubernetes اجرا می‌شود. ابزار CLI kubeadm توسط کاربر اجرا می‌شود هنگامی که Kubernetes مقدماتی یا ارتقاء می‌یابد، در حالی که kubelet همیشه در پس زمینه در حال اجرا است.

زیرا kubelet یک دیمون است، نیاز به نگهداری توسط یک سیستم init یا مدیر سرویسی دارد. هنگامی که kubelet با استفاده از DEB یا RPM نصب می‌شود، systemd پیکربندی می‌شود تا kubelet را مدیریت کند. می‌توانید از یک مدیر سرویس دیگر استفاده کنید، اما باید آن را به صورت دستی پیکربندی کنید.

جزئیات پیکربندی برخی جزئیات kubelet باید در تمام kubelet‌های درگیر در خوشه یکسان باشد، در حالی که جنبه‌های دیگر پیکربندی باید بر اساس هر kubelet به طور مجزا تنظیم شود تا ویژگی‌های مختلف یک ماشین خاص (مانند سیستم عامل، ذخیره‌سازی و شبکه) را در بر بگیرد. می‌توانید پیکربندی kubelet‌های خود را به صورت دستی مدیریت کنید، اما kubeadm در حال حاضر یک نوع API به نام `KubeletConfiguration` را برای [مدیریت مرکزی پیکربندی‌های kubelet](#configure-kubelets-using-kubeadm) فراهم می‌کند.

<!-- body -->

## الگوهای پیکربندی kubelet

بخش‌های زیر الگوهای پیکربندی kubelet را که با استفاده از kubeadm ساده شده‌اند، توضیح می‌دهند به جای مدیریت پیکربندی kubelet برای هر گره به صورت دستی.

### انتقال پیکربندی سطح خوشه به هر kubelet

می‌توانید مقادیر پیش‌فرضی را برای دستورهای `kubeadm init` و `kubeadm join` به kubelet ارائه دهید. مثال‌های جالب شامل استفاده از یک runtime container متفاوت یا تنظیم زیرشبکه پیش‌فرض استفاده شده توسط خدمات است.

اگر می‌خواهید خدمات خود را برای استفاده از زیرشبکه `10.96.0.0/12` به عنوان پیش‌فرض تنظیم کنید، می‌توانید پارامتر `--service-cidr` را به kubeadm منتقل کنید:

```bash
kubeadm init --service-cidr 10.96.0.0/12
```

آی‌پی‌های مجازی برای خدمات از این زیرشبکه تخصیص می‌یابند. همچنین باید آدرس DNS استفاده شده توسط kubelet را با استفاده از پرچم `--cluster-dns` تنظیم کنید. این تنظیم باید برای هر kubelet بر روی هر مدیر و گره در خوشه یکسان باشد. kubelet یک شیء API ساختاری و نسخه‌بندی شده را فراهم می‌کند که می‌تواند اکثر پارامترها را در kubelet پیکربندی کرده و این پیکربندی را به هر kubelet در حال اجرا در خوشه ارسال کند. این شیء با نام
[`KubeletConfiguration`](/docs/reference/config-api/kubelet-config.v1beta1/)
نامیده می‌شود.

برای جزئیات بیشتر در مورد `KubeletConfiguration` به [این بخش](#configure-kubelets-using-kubeadm) مراجعه کنید.

### ارائه جزئیات پیکربندی مخصوص موارد خاص

بعضی از میزبان‌ها به دلیل تفاوت‌های سخت‌افزاری، سیستم عامل، شبکه یا سایر پارامترهای خاص می‌توانند نیازمند پیکربندی‌های خاص kubelet باشند. لیست زیر چند مثال را ارائه می‌دهد.

- مسیر فایل DNS resolution، مشخص شده توسط پرچم پیکربندی `--resolv-conf` kubelet، ممکن است بین سیستم‌عامل‌ها متفاوت باشد یا بسته به اینکه از `systemd-resolved` استفاده می‌کنید، متفاوت باشد. اگر این مسیر اشتباه باشد، DNS resolution بر روی گره که kubelet آن به طور نادرست پیکربندی شده است، شکست می‌خورد.

- شی‌ء API گره `.metadata.name` به صورت پیش‌فرض به نام میزبان ماشین تنظیم شده است، مگر اینکه از یک ارائه دهنده ابر استفاده کنید. می‌توانید از پرچم `--hostname-override` استفاده کنید تا رفتار پیش‌فرض را بازنویسی کنید اگر نیاز به مشخص کردن نام گره متفاوت از نام ماشین داشته باشید.

- در حال حاضر، kubelet نمی‌تواند به طور خودکار درایور cgroup استفاده شده توسط runtime container را شناسایی کند، اما مقدار `--cgroup-driver`

 باید با درایور cgroup استفاده شده توسط runtime container مطابقت داشته باشد تا از سلامت kubelet اطمینان حاصل شود.

- برای تعیین runtime container باید به انتهای `--container-runtime-endpoint=<path>` فلگ تنظیم شده باشد.

روش توصیه شده برای اعمال پیکربندی خاص مورد استفاده با استفاده از [پچ‌های KubeletConfiguration](/docs/setup/production-environment/tools/kubeadm/control-plane-flags#patches) است.

## پیکربندی kubelet با استفاده از kubeadm

می‌توانید kubelet را که kubeadm اجرا خواهد کرد اگر یک شیء API `KubeletConfiguration` سفارشی با یک فایل پیکربندی مانند این `kubeadm ... --config some-config-file.yaml` منتقل شود.

با فراخوانی `kubeadm config print init-defaults --component-configs KubeletConfiguration` می‌توانید تمام مقادیر پیش‌فرض برای این ساختار را ببینید.

همچنین می‌توانید پچ‌های خاص مربوط به نمونه را بر روی `KubeletConfiguration` پایه اعمال کنید. برای جزئیات بیشتر به [شخصی سازی kubelet](/docs/setup/production-environment/tools/kubeadm/control-plane-flags#customizing-the-kubelet) مراجعه کنید.

### جریان کار هنگام استفاده از `kubeadm init`

وقتی `kubeadm init` را صدا می‌زنید، پیکربندی kubelet به دیسک در مسیر `/var/lib/kubelet/config.yaml` مارشال می‌شود و همچنین به ConfigMap `kubelet-config` در فضای نام `kube-system` خوشه بارگذاری می‌شود. یک فایل پیکربندی kubelet همچنین در `/etc/kubernetes/kubelet.conf` برای پیکربندی اصلی خوشه‌وسیع برای تمام kubelet‌ها در خوشه نوشته می‌شود. این فایل پیکربندی به گواهی‌های مشتری که به kubelet اجازه ارتباط با سرور API را می‌دهد، اشاره دارد. این موضوع به
[انتقال پیکربندی سطح خوشه به هر kubelet](#propagating-cluster-level-configuration-to-each-kubelet)
پرداخته است.

برای پرداختن به الگوی دوم
[ارائه جزئیات پیکربندی مخصوص موارد خاص](#providing-instance-specific-configuration-details)،
kubeadm یک فایل محیطی را به `/var/lib/kubelet/kubeadm-flags.env` می‌نویسد که حاوی لیستی از پرچم‌هایی است که باید به kubelet در زمان شروع منتقل شود. این پرچم‌ها به این صورت در فایل ارائه می‌شوند:

```bash
KUBELET_KUBEADM_ARGS="--flag1=value1 --flag2=value2 ..."
```

علاوه بر پرچم‌های استفاده شده در زمان شروع kubelet، این فایل شامل پارامترهای پویا مانند درایور cgroup و اینکه آیا باید از سوکت runtime container متفاوت استفاده شود (`--cri-socket`) می‌شود.

بعد از مارشال کردن این دو فایل به دیسک، kubeadm سعی می‌کند که دو دستور زیر را اجرا کند، اگر از systemd استفاده می‌کنید:

```bash
systemctl daemon-reload && systemctl restart kubelet
```

اگر بازنشانی و راه‌اندازی موفق باشد، جریان کار عادی `kubeadm init` ادامه می‌یابد.

### جریان کار هنگام استفاده از `kubeadm join`

وقتی `kubeadm join` را اجرا می‌کنید، kubeadm از اعتبارنامه Bootstrap Token برای انجام یک راه اندازی TLS استفاده می‌کند که اعتبار لازم برای دانلود ConfigMap `kubelet-config` را به دست می‌آورد و آن را به `/var/lib/kubelet/config.yaml` می‌نویسد. فایل محیطی پویا به همان روش `kubeadm init` تولید می‌شود.

سپس، `kubeadm` دو دستور زیر را برای بارگذاری پیکربندی جدید به kubelet اجرا می‌کند:

```bash
systemctl daemon-reload && systemctl restart kubelet
```

پس از اینکه kubelet پیکربندی جدید را بارگیری می‌کند، kubeadm فایل KubeConfig `/etc/kubernetes/bootstrap-kubelet.conf` را می‌نویسد که شامل یک گواهی CA و Bootstrap Token است. این‌ها توسط kubelet برای انجام TLS Bootstrap و به دست آوردن یک اعتبار منحصر به فرد استفاده می‌شوند که در `/etc/kubernetes/kubelet.conf` ذخیره می‌شود.

وقتی فایل `/etc/kubernetes/kubelet.conf` نوشته شود، kubelet به اتمام رسیدن راه‌اندازی TLS Bootstrap می‌رسد. Kubeadm پس از اتمام TLS Bootstrap، فایل `/etc/kubernetes/bootstrap-kubelet.conf` را حذف می‌کند.

## فایل drop-in kubelet برای systemd

`kubeadm` با پیکربندی هایی برای اجرای kubelet با systemd ارائه شده است. توجه داشته باشید که دستور CLI kubeadm هیچ وقت این فایل drop-in را لمس نمی‌کند.

این فایل پیکربندی که توسط بسته `kubeadm` نصب شده است، به `/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf` نوشته می‌شود و توسط systemd استفاده می‌شود. این فایل به `kubelet.service` اصلی
[`kubelet.service`](https://github.com/kubernetes/release/blob/cd53840/cmd/krel/templates/latest/kubelet/kubelet.service)
اضافه می‌شود.

اگر می‌خواهید برای ارائه پیکربندی بیشتر، می‌توانید یک دایرکتوری به نام `/etc/systemd/system/kubelet.service.d/` (نه `/usr/lib/systemd/system/kubelet.service.d/`) ایجاد کنید و سفارشی‌سازی‌های خود را به یک فایل آنجا بیاورید. به عنوان مثال، ممکن است یک فایل محلی جدید به نام `/etc/systemd/system/kubelet.service.d/local-overrides.conf` را برای بازنویسی تنظیمات واحد پیکربندی شده توسط `kubeadm` اضافه کنید.

اینجا آن چیزی است که احتمالاً در `/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf` پیدا می‌کنید:

{{< note >}}
محتوای زیر فقط یک مثال است. اگر نمی‌خواهید از یک مدیر بسته استفاده کنید، دنباله راهنمایی مشخص شده است ([بدون مدیر بسته](/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#k8s-install-2)).
{{< /note >}}

```none
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# این یک فایل است که "kubeadm init" و "kubeadm join" زمان اجرا تولید می‌کنند، پر کردن
# متغیر KUBELET_KUBE

ADM_ARGS به طور پویا
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# این یک فایل است که کاربر می‌تواند برای بازنویسی پرچم‌های kubelet به عنوان آخرین اقدام استفاده کند. بهتر است که کاربر از شیء .NodeRegistration.KubeletExtraArgs در فایل‌های پیکربندی به جای این فایل استفاده کند.
# KUBELET_EXTRA_ARGS باید از این فایل منبع گرفته شود.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```

این فایل مکان‌های پیش‌فرض برای تمامی فایل‌های مدیریت شده توسط kubeadm برای kubelet را مشخص می‌کند.

- فایل KubeConfig برای استفاده در TLS Bootstrap `/etc/kubernetes/bootstrap-kubelet.conf` است، اما فقط در صورت عدم وجود `/etc/kubernetes/kubelet.conf` استفاده می‌شود.
- فایل KubeConfig با هویت منحصر به فرد kubelet `/etc/kubernetes/kubelet.conf` است.
- فایلی که حاوی تنظیمات ComponentConfig kubelet است `/var/lib/kubelet/config.yaml` است.
- فایل محیطی پویا که حاوی `KUBELET_KUBEADM_ARGS` است از `/var/lib/kubelet/kubeadm-flags.env` منبع‌گرفته می‌شود.
- فایلی که می‌تواند شامل ابطال‌های پرچم توسط کاربر با `KUBELET_EXTRA_ARGS` باشد از `/etc/default/kubelet` (برای DEB) یا `/etc/sysconfig/kubelet` (برای RPM) منبع‌گرفته می‌شود. `KUBELET_EXTRA_ARGS` آخرین در زنجیره پرچم است و در مواجهه با تنظیمات منافع متضاد، بالاترین اولویت را دارد.

## باینری‌ها و محتویات بسته Kubernetes

بسته‌های DEB و RPM که با انتشارات Kubernetes ارائه می‌شود عبارتند از:

| نام بسته | توضیحات |
|--------------|-------------|
| `kubeadm`    | نصب ابزار CLI `/usr/bin/kubeadm` و [فایل drop-in kubelet](#the-kubelet-drop-in-file-for-systemd) برای kubelet. |
| `kubelet`    | نصب باینری `/usr/bin/kubelet`. |
| `kubectl`    | نصب باینری `/usr/bin/kubectl`. |
| `cri-tools` | نصب باینری `/usr/bin/crictl` از [مخزن git cri-tools](https://github.com/kubernetes-sigs/cri-tools). |
| `kubernetes-cni` | نصب باینری‌های `/opt/cni/bin` از [مخزن git plugins](https://github.com/containernetworking/plugins). |
