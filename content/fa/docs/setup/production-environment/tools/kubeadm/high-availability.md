---
reviewers:
- sig-cluster-lifecycle
title: ایجاد خوشه‌های با قابلیت Highly Available با استفاده از kubeadm
content_type: task
weight: 60
---

<!-- overview -->

این صفحه دو رویکرد مختلف برای راه‌اندازی یک خوشه Kubernetes با قابلیت Highly Available با استفاده از kubeadm را توضیح می‌دهد:

- با نودهای کنترلی پشته‌ای. این رویکرد نیاز کمتری به زیرساخت دارد. اعضای etcd و نودهای کنترلی در یک مکان قرار دارند.
- با یک خوشه etcd خارجی. این رویکرد نیاز بیشتری به زیرساخت دارد. نودهای کنترلی و اعضای etcd جدا می‌شوند.

قبل از ادامه، باید به دقت بررسی کنید که کدام رویکرد بهترین پاسخ را برای نیازهای برنامه‌ها و محیط شما ارائه می‌دهد. [گزینه‌های توپولوژی با قابلیت Highly Available](/docs/setup/production-environment/tools/kubeadm/ha-topology/) مزایا و معایب هر یک را برجسته می‌کند.

در صورت برخورد با مشکلات در راه‌اندازی خوشه HA، لطفاً این موارد را در [پیگیری مشکلات kubeadm](https://github.com/kubernetes/kubeadm/issues/new) گزارش دهید.

همچنین به [مستندات ارتقاء](/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/) مراجعه کنید.

{{< caution >}}
این صفحه به موضوع اجرای خوشه شما در یک ارائه‌دهنده ابری نمی‌پردازد. در محیط ابری، هیچ‌یک از رویکردهای مستند شده در اینجا با اشیاء خدمت از نوع LoadBalancer یا با حجم‌های PersistentVolumes پویا کار نمی‌کند.
{{< /caution >}}

## {{% heading "پیش‌نیازها" %}}

پیش‌نیازها بسته به توپولوژی انتخابی برای نودهای کنترلی خوشه شما است:

{{< tabs name="prerequisite_tabs" >}}
{{% tab name="Stacked etcd" %}}
<!--
    نکته به بازبینان: این پیش‌نیازها باید با شروع tab از تبادل اطلاعات جهانی باشد
-->

شما نیاز دارید به:

- سه یا بیشتر دستگاه که شرایط حداقلی [مورد نیاز kubeadm](/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin) برای نودهای کنترلی را دارا باشند. داشتن یک تعداد فردی از نودهای کنترلی می‌تواند در انتخاب رهبر در صورت شکست دستگاه یا منطقه کمک کند.
  - از جمله یک {{< glossary_tooltip text="container runtime" term_id="container-runtime" >}}، که قبلاً راه‌اندازی شده و کار می‌کند
- سه یا بیشتر دستگاه که شرایط حداقلی [مورد نیاز kubeadm](/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin) برای کارگران را دارا باشند
  - از جمله یک container runtime که قبلاً راه‌اندازی شده و کار می‌کند
- اتصال شبکه کامل بین تمام دستگاه‌ها در خوشه (شبکه عمومی یا خصوصی)
- دسترسی مدیریتی بر روی تمام دستگاه‌ها با استفاده از `sudo`
  - می‌توانید از ابزار مختلفی استفاده کنید؛ این راهنما از `sudo` در مثال‌ها استفاده می‌کند.
- دسترسی SSH از یک دستگاه به تمام نودها در سیستم
- نصب شده بودن `kubeadm` و `kubelet` بر روی تمام دستگاه‌ها.

_برای اطلاعات بیشتر، به [توپولوژی etcd پشته‌ای](/docs/setup/production-environment/tools/kubeadm/ha-topology/#stacked-etcd-topology) مراجعه کنید._

{{% /tab %}}
{{% tab name="External etcd" %}}
<!--
    نکته به بازبینان: این پیش‌نیازها باید با شروع tab از تبادل اطلاعات جهانی باشد
-->
شما نیاز دارید به:

- سه یا بیشتر دستگاه که شرایط حداقلی [مورد نیاز kubeadm](/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin) برای نودهای کنترلی را دارا باشند. داشتن یک تعداد فردی از نودهای کنترلی می‌تواند در انتخاب رهبر در صورت شکست دستگاه یا منطقه کمک کند.
  - از جمله یک {{< glossary_tooltip text="container runtime" term_id="container-runtime" >}}، که قبلاً راه‌اندازی شده و کار می‌کند
- سه یا بیشتر دستگاه که شرایط حداقلی [مورد نیاز kubeadm](/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin) برای کارگران را دارا باشند
  - از جمله یک container runtime که قبلاً راه‌اندازی شده و کار می‌کند
- اتصال شبکه کامل بین تمام دستگاه‌ها در خوشه (شبکه عمومی یا خصوصی)
- دسترسی مدیریتی بر روی تمام دستگاه‌ها با استفاده از `sudo`
  - می‌توانید از ابزار مختلفی استفاده کنید؛ این راهنما از `sudo` در مثال‌ها استفاده می‌کند.


- دسترسی SSH از یک دستگاه به تمام نودها در سیستم
- نصب شده بودن `kubeadm` و `kubelet` بر روی تمام دستگاه‌ها.

<!-- پایان پیش‌نیازهای مشترک -->

و همچنین نیاز به:

- سه یا بیشتر دستگاه اضافی، که عضوهای خوشه etcd می‌شوند. داشتن یک تعداد فردی از اعضای خوشه etcd برای دستیابی به آرایه‌ای بهینه از تحقق بهتر است.
  - این دستگاه‌ها باید دوباره `kubeadm` و `kubelet` نصب شده باشند.
  - این دستگاه‌ها همچنین به یک container runtime نیاز دارند که قبلاً راه‌اندازی شده و کار می‌کند.

_برای اطلاعات بیشتر، به [توپولوژی etcd خارجی](/docs/setup/production-environment/tools/kubeadm/ha-topology/#external-etcd-topology) مراجعه کنید._

{{< /tab >}}
{{< /tabs >}}

### تصاویر ظروف

هر میزبان باید دسترسی به کتابخانه ظروف ظرف‌های محیطی Kubernetes، `registry.k8s.io`، را داشته باشد.
اگر می‌خواهید یک خوشه با قابلیت Highly Available اجرا کنید که میزبان‌ها دسترسی به کشیدن تصاویر را ندارند، این امکان وجود دارد. شما باید از طریق متوسل به ابزار دیگر مطمئن شوید که تصاویر ظروف صحیح از قبل بر روی میزبان‌های مربوطه در دسترس هستند.

### رابط خط فرمان {#kubectl}

برای مدیریت Kubernetes پس از راه‌اندازی خوشه خود، شما باید [kubectl را نصب کنید](/docs/tasks/tools/#kubectl) روی رایانه شخصی خود. همچنین نصب ابزار `kubectl` بر روی هر نود کنترلی می‌تواند برای رفع مشکل مشکلات مفید باشد.

<!-- steps -->

## اقدامات اولیه برای هر دو روش

### ایجاد load balancer برای kube-apiserver

{{< note >}}
برای توزیع بار، چندین پیکربندی وجود دارد. مثال زیر فقط یک گزینه است. نیازمندی‌های خوشه شما ممکن است نیاز به پیکربندی مختلفی داشته باشد.
{{< /note >}}

1. یک load balancer kube-apiserver با یک نام ایجاد کنید که به DNS حل می‌شود.

   - در یک محیط ابری، شما باید نودهای نودهای کنترلی را پشت یک load balancer تکانه TCP قرار دهید. این توزیع ترافیک را به تمام نودهای کنترلی سالم در لیست هدف خود توزیع می‌کند. بررسی سلامت برای یک سرور API یک بررسی TCP روی پورتی است که kube-apiserver بر روی آن گوش می‌دهد (مقدار پیش‌فرض `: 6443` است).

   - توصیه نمی‌شود که مستقیماً از یک آدرس IP در یک محیط ابری استفاده کنید.

   - load balancer باید قادر به ارتباط با تمام نودهای کنترلی بر روی پورت apiserver باشد. همچنین باید ترافیک ورودی را در پورت گوش دادن خود مجاز کند.

   - اطمینان حاصل کنید که آدرس load balancer همیشه با آدرس `ControlPlaneEndpoint` kubeadm مطابقت دارد.

   - راهنمای [گزینه‌های Load Balancing Software](https://git.k8s.io/kubeadm/docs/ha-considerations.md#options-for-software-load-balancing) را برای جزئیات بیشتر مطالعه کنید.

1. نود اول کنترل پلان را به گروه هدف load balancer اضافه کنید و اتصال را تست کنید:

   ```shell
   nc -v <LOAD_BALANCER_IP> <PORT>
   ```

   انتظار می‌رود خطای اتصال را به دلیل اینکه سرویس API هنوز اجرا نشده است. اگر یک مهلت توقف اتفاق بیفتد، یعنی نمی‌تواند load balancer با نود کنترل ارتباط برقرار کند. اگر یک timeout اتفاق بیفتد، load balancer را برای ارتباط با نود کنترل تنظیم مجدد کنید.

## نودهای کنترلی پشته‌ای و اعضای etcd

### مراحل برای نود اول کنترل پلان

1. شروع نود کنترلی:

   ```sh
   sudo kubeadm init --control-plane-endpoint "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT" --upload-certs
   ```

   - می‌توانید از پرچم `--kubernetes-version` برای تنظیم نسخه Kubernetes استفاده کنید. توصیه می‌شود که نسخه‌های kubeadm، kubelet، kubectl و Kubernetes با هم مطابقت داشته باشند.
   - پرچم `--control-plane-endpoint` باید به آدرس یا DNS و پورت load balancer تنظیم شود.

   - پرچم `--upload-certs` برای آپلود گواهینامه‌هایی که باید در تمام نمونه‌های نود کنترلی به اشتراک گذاشته شود، استفاده می‌شود. اگر به جای این کار ترجیح می‌دهید که گواهینامه‌ها را به صورت دستی یا با استفاده از ابزارهای اتوماسیونی کپی کنید، لطفاً این پرچم را حذف کنید و به بخش [توزیع دستی گواهینامه‌ها](#manual-certs) مراجعه کنید.

   {{< note >}}
   پرچم‌های `kubeadm init` `--config` و `--certificate-key` نمی‌توانند مخلوط شوند، بنابراین اگر می‌خواهید از [پیکربندی kubeadm](/docs/reference/config-api/kubeadm-config.v1beta3/) استفاده کنید باید فیلد `certificateKey` را در مکان‌های پیکربندی مناسب (زیر `InitConfiguration` و `JoinConfiguration: controlPlane`) اضافه کنید.
   {{< /note >}}

   {{< note >}}
   برخی از افزونه‌های شبکه CNI نیازمند پیکربندی‌های اضافی هستند، مانند تعیین CIDR IP پاد. برای جزئیات بیشتر به [مستندات شبکه CNI](/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network) مراجعه کنید.
   برای اضافه کردن یک CIDR پاد، پرچم `--pod-network-cidr` را منتقل کنید، یا اگر از یک فایل پیکربندی kubeadm استفاده می‌کنید، فیلد `podSubnet` را زیر شی networking تنظیمات ClusterConfiguration قرار دهید.
   {{< /note >}}

   خروجی مانند زیر خواهد بود:

   ```sh
   ...
   شما می‌توانید هر تعداد نود کنترلی جدید را با اجرای دستور زیر به عنوان ریشه به خوشه بپیوندید:
       kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866 --control-plane --certificate-key f8902e114ef118304e561c3ecd4d0b543adc226b7a07f675f56564185ffe0c07

   لطفاً توجه داشته باشید که certificate-key به داده‌های حساس خوشه دسترسی می‌دهد، آن را محرمانه نگه دارید!
   به عنوان یک ایمنی، گواهینامه‌های آپلود شده بعد از دو ساعت حذف می‌شوند؛ در صورت لزوم می‌توانید از مرحله آپلود گواهینامه‌ها برای بارگذاری مجدد گواهینامه‌ها استفاده کنید.

   سپس می‌توانید هر تعداد نود کارگر را با اجرای زیر روی هر یک به عنوان ریشه به خوشه بپیوندید:
       kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866
   ```

   - این خروجی را به یک فایل متنی کپی کنید. بعداً برای پیوستن نودهای کنترل پلان و کارگر به خوشه نیاز خواهید داشت.
   - هنگام استفاده از `--upload-certs` با `kubeadm init`، گواهینامه‌های نود اصلی کنترل پلان رمزگذاری شده و در Secret `kubeadm-certs` آپلود می‌شوند.
   - برای مجدداً آپلود گواهینامه‌ها و تولید یک کلید رمزگذاری جدید، از دستور زیر بر روی نود کنترلی استفاده کنید که در حال حاضر به خوشه پیوسته است:

     ```sh
     sudo kubeadm init phase upload-certs --upload-certs
     ```

   - همچنین می‌توانید در طول `init` یک `--certificate-key` دلخواه مشخص کنید که بعداً توسط `join` استفاده شود. برای تولید چنین کلیدی، می‌توانید از دستور زیر استفاده کنید:

     ```sh
     kubeadm certs certificate-key
     ```

   کلید گواهینامه یک رشته hex encoded است که یک AES key با اندازه 32 بایت است.

   {{< note >}}
   Secret `kubeadm-certs` و کلید رمزگذاری بعد از دو ساعت منقضی می‌شوند.
   {{< /note >}}

   {{< caution >}}
   همانطور که در خروجی دستور ذکر شده است، certificate key دسترسی به داده‌های حساس خوشه را فراهم می‌کند، آن را محرمانه نگه دارید!
   {{< /caution >}}

1. اعمال افزونه شبکه CNI انتخابی خود:
   [دست

ورالعمل‌های مربوطه](/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network)
   را برای نصب ارائه‌دهنده CNI دنبال کنید. اطمینان حاصل کنید که پیکربندی با CIDR پاد مطابقت دارد که در فایل پیکربندی kubeadm تعیین شده است (در صورت لزوم).

   {{< note >}}
   شما باید یک افزونه شبکه را انتخاب کنید که با مورد استفاده شما سازگار باشد و آن را قبل از ادامه مرحله بعدی نصب کنید.
   اگر این کار را انجام ندهید، نمی‌توانید خوشه خود را به درستی راه‌اندازی کنید.
   {{< /note >}}

1. دستور زیر را تایپ کنید و پادهای مولفه‌های نود کنترلی را آغاز کنید:

   ```sh
   kubectl get pod -n kube-system -w
   ```

### مراحل برای بقیه نودهای کنترلی

برای هر نود کنترلی اضافی باید:

1. دستور پیوستنی را که پیشتر توسط خروجی `kubeadm init` در نود اول داده شده است، اجرا کنید. باید مانند زیر به نظر برسد:

   ```sh
   sudo kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866 --control-plane --certificate-key f8902e114ef118304e561c3ecd4d0b543adc226b7a07f675f56564185ffe0c07
   ```

   - پرچم `--control-plane` به `kubeadm join` می‌گوید که یک کنترل پلان جدید ایجاد کند.
   - پرچم `--certificate-key ...` باعث می‌شود گواهینامه‌های نود کنترلی از Secret `kubeadm-certs` در خوشه دانلود شوند و با استفاده از کلید داده شده رمزگشایی شوند.

می‌توانید چندین نود کنترلی را به صورت همزمان به خوشه پیوست کنید.

## نودهای خارجی etcd

راه‌اندازی یک خوشه با نودهای etcd خارجی شبیه روش استفاده شده برای اجرای پشته‌ای با استثنای آنکه باید ابتدا etcd راه‌اندازی کنید و باید اطلاعات etcd را در فایل پیکربندی kubeadm گذرانده شود.

### راه‌اندازی خوشه etcd

1. دستورالعمل‌های [زیر](/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/) را دنبال کنید تا خوشه etcd راه‌اندازی شود.

1. SSH را به عنوان [اینجا](#manual-certs) شرح داده شده راه‌اندازی کنید.

1. فایل‌های زیر را از هر نود etcd در خوشه به نود کنترلی اول کپی کنید:

   ```sh
   export CONTROL_PLANE="ubuntu@10.0.0.7"
   scp /etc/kubernetes/pki/etcd/ca.crt "${CONTROL_PLANE}":
   scp /etc/kubernetes/pki/apiserver-etcd-client.crt "${CONTROL_PLANE}":
   scp /etc/kubernetes/pki/apiserver-etcd-client.key "${CONTROL_PLANE}":
   ```

   - مقدار CONTROL_PLANE را با کاربر@میزبان اول نود کنترلی جایگزین کنید.

### راه‌اندازی اولین نود کنترلی پلان

1. یک فایل به نام `kubeadm-config.yaml` با محتوای زیر ایجاد کنید:

   ```yaml
   ---
   apiVersion: kubeadm.k8s.io/v1beta3
   kind: ClusterConfiguration
   kubernetesVersion: stable
   controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT" # این را تغییر دهید (جهت مشاهده پایین)
   etcd:
     external:
       endpoints:
         - https://ETCD_0_IP:2379 # ETCD_0_IP را به‌مناسبت تغییر دهید
         - https://ETCD_1_IP:2379 # ETCD_1_IP را به‌مناسبت تغییر دهید
         - https://ETCD_2_IP:2379 # ETCD_2_IP را به‌مناسبت تغییر دهید
       caFile: /etc/kubernetes/pki/etcd/ca.crt
       certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
       keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
   ```

   {{< note >}}
   تفاوت بین etcd پشته‌ای و etcd خارجی در اینجا این است که راه‌اندازی etcd خارجی نیاز به یک فایل پیکربندی با نقاط پایانی etcd در زیر شی `external` دارد برای `etcd`.
   در صورت استفاده از توپولوژی etcd پشته‌ای، اینکار به صورت خودکار مدیریت می‌شود.
   {{< /note >}}

   - مقادیر زیر را در قالب پیکربندی با مقادیر مناسب برای خوشه‌تان جایگزین کنید:

     - `LOAD_BALANCER_DNS`
     - `LOAD_BALANCER_PORT`
     - `ETCD_0_IP`
     - `ETCD_1_IP`
     - `ETCD_2_IP`

مراحل بعدی شبیه به راه‌اندازی etcd پشته‌ای هستند:

1. دستور `sudo kubeadm init --config kubeadm-config.yaml --upload-certs` را در این نود اجرا کنید.

1. دستورهای پیوسته از خروجی `join` که برای استفاده بعدی برگردانده می‌شوند، را در یک فایل متنی ذخیره کنید.

1. افزونه CNI مورد نظر خود را اعمال کنید.

   {{< note >}}
   شما باید یک افزونه شبکه را انتخاب کنید که با مورد استفاده‌ی شما سازگار باشد و قبل از ادامه به مرحله بعدی آن را نصب کنید.
   اگر این کار را انجام ندهید، احتمالاً نخواهید توانست خوشه‌تان را به درستی راه‌اندازی کنید.
   {{< /note >}}

### مراحل برای سایر نودهای کنترلی پلان

مراحل مشابه راه‌اندازی etcd پشته‌ای هستند:

- مطمئن شوید که اولین نود کنترلی پلان به طور کامل راه‌اندازی شده است.
- هر نود کنترلی پلان را با دستور `join` که در یک فایل متنی ذخیره کرده‌اید به خوشه متصل کنید. توصیه می‌شود که نودهای کنترلی پلان را یکی‌یکی به خوشه متصل کنید.
- فراموش نکنید که کلید رمزگشایی از `--certificate-key` بعد از دو ساعت به صورت پیش‌فرض منقضی می‌شود.

## وظایف مشترک پس از راه‌اندازی اولیه نود کنترلی

### نصب نودهای کارگر

نودهای کارگر می‌توانند با دستوری که قبلاً به عنوان خروجی از دستور `kubeadm init` ذخیره کرده‌اید، به خوشه متصل شوند:

```sh
sudo kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866
```

## توزیع دستی گواهینامه‌ها {#manual-certs}

اگر تصمیم دارید که از `kubeadm init` با پرچم `--upload-certs` استفاده نکنید، این به معنی است که شما باید به صورت دستی گواهینامه‌ها را از نود کنترلی اصلی به نودهای کنترلی مشترک بیاورید.

برای انجام این کار روش‌های مختلفی وجود دارد. مثال زیر از `ssh` و `scp` استفاده می‌کند:

SSH اجباری است اگر می‌خواهید از یک دستگاه کنترلی همه نودها را کنترل کنید.

1. بر روی دستگاه اصلی که دسترسی به تمام نودها را دارد، ssh-agent را فعال کنید:

   ```
   eval $(ssh-agent)
   ```

1. هویت SSH خود را به جلسه اضافه کنید:

   ```
   ssh-add ~/.ssh/path_to_private_key
   ```

1. SSH بین نودها برای بررسی صحت ارتباط کار می‌کند.

   - وقتی به هر نود SSH می‌کنید، پرچم `-A` را اضافه کنید. این پرچم به نودی که از طریق SSH وارد شده‌اید، اجازه می‌دهد به SSH agent در PC شما دسترسی داشته باشد. اگر به اعتماد کامل به امنیت جلسه کاربری خود در نود، نمی‌اندازید، روش‌های جایگزین را در نظر بگیرید.

     ```
     ssh -A 10.0.0.7
     ```

   - زمان استفاده sudo روی هر نود، مطمئن شوید که محیط را حفظ می‌کنید تا SSH فورواردینگ کار کند:

     ```
     sudo -E -s
     ```

1. بعد از پیکربندی SSH بر روی همه‌ی نوده

ا، شما باید اسکریپت زیر را بر روی اولین نود کنترلی پس از اجرای `kubeadm init` اجرا کنید. این اسکریپت گواهینامه‌ها را از نود کنترلی اولیه به سایر نودهای کنترلی منتقل می‌کند:

   در مثال زیر، `CONTROL_PLANE_IPS` را با آدرس‌های IP نودهای کنترلی دیگر جایگزین کنید.

   ```sh
   USER=ubuntu # قابل شخصی‌سازی
   CONTROL_PLANE_IPS="10.0.0.7 10.0.0.8"
   for host in ${CONTROL_PLANE_IPS}; do
       scp /etc/kubernetes/pki/ca.crt "${USER}"@$host:
       scp /etc/kubernetes/pki/ca.key "${USER}"@$host:
       scp /etc/kubernetes/pki/sa.key "${USER}"@$host:
       scp /etc/kubernetes/pki/sa.pub "${USER}"@$host:
       scp /etc/kubernetes/pki/front-proxy-ca.crt "${USER}"@$host:
       scp /etc/kubernetes/pki/front-proxy-ca.key "${USER}"@$host:
       scp /etc/kubernetes/pki/etcd/ca.crt "${USER}"@$host:etcd-ca.crt
       # اگر از etcd خارجی استفاده می‌کنید، خط بعدی را از دستور حذف کنید
       scp /etc/kubernetes/pki/etcd/ca.key "${USER}"@$host:etcd-ca.key
   done
   ```

   {{< caution >}}
   فقط گواهینامه‌های مذکور را کپی کنید. kubeadm به تولید بقیه‌ی گواهینامه‌ها با SAN مورد نیاز برای نمونه‌های نود کنترلی جدید می‌پردازد. اگر به‌طور اشتباهی همه‌ی گواهینامه‌ها را کپی کنید، ایجاد نودهای اضافی ممکن است به دلیل عدم وجود SAN‌های مورد نیاز، شکست بخورد.
   {{< /caution >}}

1. سپس بر روی هر نود کنترلی پیوسته باید اسکریپت زیر را قبل از اجرای `kubeadm join` اجرا کنید. این اسکریپت گواهینامه‌هایی را که پیش‌تر از دستگاه اصلی نودها کپی کرده‌اید، از دایرکتوری خانه به `/etc/kubernetes/pki` منتقل می‌کند:

   ```sh
   USER=ubuntu # قابل شخصی‌سازی
   mkdir -p /etc/kubernetes/pki/etcd
   mv /home/${USER}/ca.crt /etc/kubernetes/pki/
   mv /home/${USER}/ca.key /etc/kubernetes/pki/
   mv /home/${USER}/sa.pub /etc/kubernetes/pki/
   mv /home/${USER}/sa.key /etc/kubernetes/pki/
   mv /home/${USER}/front-proxy-ca.crt /etc/kubernetes/pki/
   mv /home/${USER}/front-proxy-ca.key /etc/kubernetes/pki/
   mv /home/${USER}/etcd-ca.crt /etc/kubernetes/pki/etcd/ca.crt
   # اگر از etcd خارجی استفاده می‌کنید، خط بعدی را از دستور حذف کنید
   mv /home/${USER}/etcd-ca.key /etc/kubernetes/pki/etcd/ca.key
   ```
