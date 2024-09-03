---
title: رفع مشکلات kubeadm
content_type: concept
weight: 20
---

<!-- overview -->

مانند هر برنامه‌ای، ممکن است هنگام نصب یا اجرای kubeadm با خطاهایی مواجه شوید. در این صفحه، برخی از سناریوهای شایع خطا را لیست کرده‌ایم و مراحلی را ارائه داده‌ایم که می‌توانند به شما کمک کنند مشکل را درک کنید و آن را برطرف کنید.

اگر مشکل شما در زیر لیست نشده باشد، لطفاً مراحل زیر را دنبال کنید:

- اگر فکر می‌کنید مشکل شما یک باگ با kubeadm است:
  - به [github.com/kubernetes/kubeadm](https://github.com/kubernetes/kubeadm/issues) بروید و برای موارد موجود جستجو کنید.
  - اگر هیچ موردی وجود نداشت، لطفاً [یک مورد جدید باز کنید](https://github.com/kubernetes/kubeadm/issues/new) و الگوی موضوع را دنبال کنید.

- اگر درباره نحوه کار kubeadm مطمئن نیستید، می‌توانید در [Slack](https://slack.k8s.io/) در `#kubeadm` سوال کنید،
  یا یک سوال در [StackOverflow](https://stackoverflow.com/questions/tagged/kubernetes) باز کنید. لطفاً برچسب‌های مربوط مانند `#kubernetes` و `#kubeadm` را اضافه کنید تا افراد بتوانند به شما کمک کنند.

<!-- body -->

## عدم امکان پیوستن یک Node v1.18 به خوشه v1.17 به دلیل عدم وجود RBAC

در v1.18، kubeadm امکان پیوستن یک Node به خوشه را ممنوع کرد اگر یک Node با همان نام از قبل وجود داشته باشد.
این اقدام نیاز به افزودن RBAC برای کاربر bootstrap-token دارد تا بتواند یک شیء Node را GET کند.

اما این موضوع باعث مشکلاتی می‌شود که `kubeadm join` از v1.18 نمی‌تواند به یک خوشه اضافه شود که توسط kubeadm v1.17 ایجاد شده است.

برای حل مشکل، دو گزینه دارید:

اجرای `kubeadm init phase bootstrap-token` روی یک Node کنترل‌پلن با استفاده از kubeadm v1.18.
توجه داشته باشید که این اجازه می‌دهد سایر مجوزهای bootstrap-token را نیز فعال کنید.

یا

اعمال RBAC زیر به صورت دستی با استفاده از `kubectl apply -f ...`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kubeadm:get-nodes
rules:
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubeadm:get-nodes
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kubeadm:get-nodes
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: Group
    name: system:bootstrappers:kubeadm:default-node-token
```

## `ebtables` یا مشابه آن در هنگام نصب یافت نشد

اگر هنگام اجرای `kubeadm init` اخطارهای زیر را مشاهده می‌کنید:

```console
[preflight] WARNING: ebtables not found in system path
[preflight] WARNING: ethtool not found in system path
```

این ممکن است به دلیل عدم وجود `ebtables`، `ethtool` یا یک اجرای‌پذیر مشابه بر روی نود شما باشد.
شما می‌توانید آن‌ها را با دستورات زیر نصب کنید:

- برای کاربران Ubuntu/Debian، دستور `apt install ebtables ethtool` را اجرا کنید.
- برای کاربران CentOS/Fedora، دستور `yum install ebtables ethtool` را اجرا کنید.

## kubeadm در هنگام نصب در حالت بلوکه شدن منتظر control plane

اگر مشاهده کنید که پس از چاپ خط زیر، `kubeadm init` در حالت بلوکه شدن است:

```console
[apiclient] Created API client, waiting for the control plane to become ready
```

این ممکن است به دلیل چند مشکل باشد. معمول‌ترین آنها عبارتند از:

- مشکلات اتصال شبکه. اطمینان حاصل کنید که ماشین شما قبل از ادامه دارای اتصال شبکه کامل است.
- درایور cgroup از container runtime با کبلت متفاوت است. برای درک این که چگونه آن را به درستی پیکربندی کنید، به [پیکربندی درایور cgroup](/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/) مراجعه کنید.
- کانتینرهای control plane در حال crashlooping یا hanging هستند. می‌توانید این را با اجرای `docker ps` بررسی کنید و با اجرای `docker logs` هر کانتینر را بررسی کنید. برای runtime کانتینر دیگر، به [Debugging Kubernetes nodes with crictl](/docs/tasks/debug/debug-cluster/crictl/) مراجعه کنید.

## kubeadm در هنگام حذف کانتینرهای مدیریت شده مسدود شده است

اگر container runtime متوقف شود و هیچ کانتینری مدیریت شده توسط Kubernetes را حذف نکند، این مشکل ممکن است رخ دهد:

```shell
sudo kubeadm reset
```

```console
[preflight] Running pre-flight checks
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Removing kubernetes-managed containers
(block)
```

راه‌حل ممکن این است که container runtime را دوباره راه‌اندازی کرده و سپس `kubeadm reset` را مجدداً اجرا کنید. همچنین می‌توانید از `crictl` برای اشکال‌زدایی وضعیت runtime کانتینر استفاده کنید. برای اطلاعات بیشتر به [Debugging Kubernetes nodes with crictl](/docs/tasks/debug/debug-cluster/crictl/) مراجعه کنید.

## Pods in `RunContainerError`, `CrashLoopBackOff` or `Error` state

به‌طور مستقیم پس از اجرای `kubeadm init` نباید هیچ پاد در این وضعیت‌ها باشد.

- اگر پس از اجرای `kubeadm init` پادهایی در یکی از این وضعیت‌ها وجود داشته باشد، لطفاً یک مشکل در مخزن kubeadm باز کنید. `coredns` (یا `kube-dns`) باید در وضعیت `Pending` باشد تا زمانی که شما افزونه شبکه را ارائه داده باشید.
- اگر پس از نصب افزونه شبکه پادها در وضعیت `RunContainerError`، `CrashLoopBackOff` یا `Error` باقی بمانند و هیچ اقدامی در مورد `coredns` (یا `kube-dns`) انجام نشود، احتمالاً افزونه شبکه پاد شما به‌طور ناگهانی شکسته است. شما ممکن است نیاز داشته باشید مجوزهای RBAC بیشتری را اعطا کنید یا از نسخه جدیدتری استفاده کنید. لطفاً یک مشکل در ردیاب مشکلات افزونه‌های شبکه پاد باز کنید و مشکل را بررسی کنید.

## `coredns` در وضعیت `Pending` مسدود شده است

این امر **منتظره** و بخشی از طراحی است. kubeadm تامین‌کننده شبکه ناوابسته به استرس‌گذار است، بنابراین مدیر باید [افزونه شبکه پاد را](/docs/concepts/cluster-administration/addons/) برگزیند. شما باید پیش از این که CoreDNS به طور کامل نصب شود یک شبکه پاد را نصب کنید. بنابراین وضعیت `Pending` قبل از راه‌اندازی شبکه امر معقول است.

## خدمات `HostPort` کار نمی‌کنند

قابلیت `HostPort` و `HostIP` بسته به تامین‌کننده شبکه پاد شما در دسترس است. لطفاً با نویسنده افزونه شبکه پاد تماس بگیرید تا ببینید آیا قابلیت‌های `HostPort` و `HostIP` در دسترس هستند.

تأیید شده است که تامین‌کنندگان CNI مانند Calico، Canal و Flannel از `HostPort` پشتیبانی می‌کنند.

برای اطلاعات بیشتر، به [مستندات CNI portmap](https://github.com/containernetworking/plugins/blob/master/plugins/meta/portmap/README.md) مراجعه کنید.

اگر تامین‌کننده شبکه شما از افزونه CNI portmap پشتیبانی نمی‌کند، ممکن است نیاز داشته باشید از ویژگی NodePort خدمات استفاده کنید یا `HostNetwork=true` را استفاده کنید.

## به وسیله آدرس IP خدمات به Pods دسترسی ندارند

- بسیاری از افزونه‌های شبکه هنوز حالت hairpin را فعال نمی‌کنند که این اجازه را به پادها می‌دهد تا از طریق آدرس IP خدمت به خود دسترسی پیدا کنند. این یک مسئله مرتبط با [CNI](https://github.com/containernetworking/cni/issues/476) است. لطفاً با تامین‌کننده افزونه شبکه تماس بگیرید تا وضعیت جدیدترین پشتیبانی از حالت hairpin را بدست آورید.

- اگر از VirtualBox (مستقیماً یا از طریق Vagrant) استفاده می‌کنید، باید اطمینان حاصل کنید که `hostname -i` آدرس IP routable بازمی‌گرداند. به طور پیش‌فرض، اولین رابط به شبکه محلی host-only non-routable متصل است. یک راه‌حل این است که فایل `/etc/hosts` را تغییر دهید، به این [Vagrantfile](https://github.com/errordeveloper/k8s-playground/blob/22dd39dfc06111235620e6c4404a96ae146f26fd/Vagrantfile#L11) مراجعه کنید.

## خطاهای گواهی TLS

خطای زیر نشان می‌دهد که احتمالاً یک عدم تطابق گواهی رخ داده است.

```none
# kubectl get pods
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
```

- تأیید کنید که فایل `$HOME/.kube/config` شامل یک گواهی معتبر است و در صورت لزوم گواهی را بازسازی کنید. گواهی‌ها در یک فایل kubeconfig به صورت base64 encoded هستند. دستور `base64 --decode` برای رمزگشایی گواهی و `openssl x509 -text -noout` برای مشاهده اط

لاعات گواهی استفاده می‌شود.

- متغیر `KUBECONFIG` را با استفاده از دستور زیر نا‌منظم کنید:

  ```sh
  unset KUBECONFIG
  ```

  یا آن را به محل پیش‌فرض `KUBECONFIG` تنظیم کنید:

  ```sh
  export KUBECONFIG=/etc/kubernetes/admin.conf
  ```

- یک راه‌حل دیگر این است که `kubeconfig` موجود را برای کاربر "admin" بازنویسی کنید:

  ```sh
  mv $HOME/.kube $HOME/.kube.bak
  mkdir $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```

## عدم موفقیت در چرخه گواهی نمونه‌ی کلاینت kubelet {#kubelet-client-cert}

به‌طور پیش‌فرض، kubeadm یک kubelet را با چرخه‌ی اتوماتیک گواهی‌های مشتری با استفاده از symlink `/var/lib/kubelet/pki/kubelet-client-current.pem` در `/etc/kubernetes/kubelet.conf` پیکربندی می‌کند. اگر این فرایند چرخه ناموفق باشد، ممکن است خطاهایی مانند `x509: certificate has expired or is not yet valid` در logهای kube-apiserver مشاهده شود. برای رفع این مشکل، باید این مراحل را دنبال کنید:

1. فایل‌های `/etc/kubernetes/kubelet.conf` و `/var/lib/kubelet/pki/kubelet-client*` را در نودی که شکست خورده است، پشتیبان‌گیری و حذف کنید.
1. از یک نود کنترل پلان کاری کنید که دارای `/etc/kubernetes/pki/ca.key` است و دستور `kubeadm kubeconfig user --org system:nodes --client-name system:node:$NODE > kubelet.conf` را اجرا کنید. `$NODE` باید به نام نود شکست خورده موجود در خوشه تنظیم شود. خروجی `kubelet.conf` را به‌طور دستی ویرایش کنید تا نام خوشه و نقطه پایانی سرور را تنظیم کنید، یا پارامتر `kubeconfig user --config` را منتقل کنید (مراجعه به [Generating kubeconfig files for additional users](/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/#kubeconfig-additional-users)). اگر خوشه شما برای `ca.key` دارای نه داشت، شما باید گواهی‌های جاسازی شده در `kubelet.conf` را خارجی امضا کنید.
1. این `kubelet.conf` حاصل را به `/etc/kubernetes/kubelet.conf` روی نود شکست خورده کپی کنید.
1. kubelet را (`systemctl restart kubelet`) روی نود شکست خورده راه‌اندازی مجدد کنید و منتظر شوید تا `/var/lib/kubelet/pki/kubelet-client-current.pem` بازسازی شود.
1. `kubelet.conf` را به‌طور دستی ویرایش کنید تا به گواهی‌های مشتری kubelet دوره‌ای اشاره کنید، با جایگزینی `client-certificate-data` و `client-key-data` با:

   ```yaml
   client-certificate: /var/lib/kubelet/pki/kubelet-client-current.pem
   client-key: /var/lib/kubelet/pki/kubelet-client-current.pem
   ```

1. kubelet را مجدداً راه‌اندازی کنید.
1. اطمینان حاصل کنید که نود به وضعیت `Ready` می‌رسد.

## NIC پیش‌فرض هنگام استفاده از flannel به عنوان شبکه‌ی pod در Vagrant

اگر خطای زیر را مشاهده می‌کنید، احتمالاً مشکلی در شبکه‌ی pod وجود دارد:

```sh
Error from server (NotFound): the server could not find the requested resource
```

- اگر از flannel به عنوان شبکه‌ی pod در داخل Vagrant استفاده می‌کنید، باید نام رابط پیش‌فرض برای flannel را مشخص کنید.

  Vagrant به طور معمول دو رابط را به تمام ماشین‌های مجازی اختصاص می‌دهد. اولین رابط که به همه میزبان‌ها آدرس IP `10.0.2.15` را اختصاص می‌دهد، برای ترافیک خارجی است که NAT می‌شود.

  این ممکن است باعث مشکلاتی برای flannel شود که به رابط اول بر روی میزبان پیش‌فرض می‌کند. این باعث می‌شود تمام میزبان‌ها فکر کنند که آدرس IP عمومی یکسانی دارند. برای جلوگیری از این مشکل، باید پرچم `--iface eth1` را به flannel منتقل کنید تا رابط دوم انتخاب شود.

## استفاده از آدرس آی‌پی غیرعمومی برای کانتینرها

در برخی شرایط، دستورات `kubectl logs` و `kubectl run` ممکن است با خطاهای زیر در یک خوشه‌ی عملکردی دیگر:

```console
Error from server: Get https://10.19.0.41:10250/containerLogs/default/mysql-ddc65b868-glc5m/mysql: dial tcp 10.19.0.41:10250: getsockopt: no route to host
```

- این ممکن است به دلیل استفاده از Kubernetes از یک آدرس آی‌پی که نمی‌تواند با سایر آدرس‌ها در زیرشبکه‌ی مشابه ارتباط برقرار کند باشد، احتمالاً به دلیل سیاست ارائه‌دهنده ماشین.
- DigitalOcean یک آدرس آی‌پی عمومی به `eth0` اختصاص می‌دهد و همچنین یک آدرس خصوصی برای استفاده داخلی برای ویژگی floating IP خود می‌دهد، اما `kubelet` آخرین را به عنوان `InternalIP` گره انتخاب خواهد کرد به جای اولیه.

  از `ip addr show` برای بررسی این حالت استفاده کنید تا `ifconfig`، زیرا `ifconfig` آدرس IP فرعی مشکل‌زا را نشان نمی‌دهد. به علاوه، یک انتها‌نقطه API خاص به DigitalOcean اجازه می‌دهد تا برای آدرس IP anchor از droplet پرس‌وجو کنید:

  ```sh
  curl http://169.254.169.254/metadata/v1/interfaces/public/0/anchor_ipv4/address
  ```

  راه‌حل این است که به `kubelet` بگویید کدام آی‌پی را با استفاده از `--node-ip` استفاده کند.
  هنگام استفاده از DigitalOcean، می‌تواند آی‌پی عمومی (که به `eth0` اختصاص داده شده است) یا آی‌پی خصوصی (که به `eth1` اختصاص داده شده است) باشد اگر بخواهید از شبکه‌ی خصوصی اختیاری استفاده کنید. بخش `kubeletExtraArgs` از ساختار `NodeRegistrationOptions` در kubeadm
  می‌تواند برای این مورد استفاده شود.

  سپس `kubelet` را دوباره راه‌اندازی کنید:

  ```sh
  systemctl daemon-reload
  systemctl restart kubelet
  ```

## پاد‌های coredns در وضعیت `CrashLoopBackOff` یا `Error`

اگر نودهایی دارید که با SELinux با نسخه قدیمی Docker اجرا می‌شوند، ممکن است به مشکلی برخورده‌اید که پاد‌های `coredns` شروع نمی‌شوند. برای حل این مشکل، می‌توانید یکی از گزینه‌های زیر را امتحان کنید:

- ارتقاء به [نسخه جدید‌تر Docker](/docs/setup/production-environment/container-runtimes/#docker).

- [غیرفعال‌سازی SELinux](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security-enhanced_linux/sect-security-enhanced_linux-enabling_and_disabling_selinux-disabling_selinux).

- تغییر پیکربندی راه‌اندازی `coredns` برای تنظیم `allowPrivilegeEscalation` به `true`:

```bash
kubectl -n kube-system get deployment coredns -o yaml | \
  sed 's/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | \
  kubectl apply -f -
```

دلیل دیگری برای داشتن `CrashLoopBackOff` در CoreDNS این است که یک پاد CoreDNS که در Kubernetes استقرار می‌یابد، حلقه را تشخیص دهد. [تعدادی راه‌حل](https://github.com/coredns/coredns/tree/master/plugin/loop#troubleshooting-loops-in-kubernetes-clusters)
برای جلوگیری از این مشکل وجود دارد تا Kubernetes هر بار CoreDNS حلقه را تشخیص دهد و خارج شود.

{{< warning >}}
غیرفعال‌سازی SELinux یا تنظیم `allowPrivilegeEscalation` به `true` می‌تواند امنیت خوشه‌ی شما را تهدید کند.
{{< /warning >}}

## پادهای etcd به طور مداوم دوباره راه‌اندازی می‌شوند

اگر با خطای زیر مواجه شدید:

```
rpc error: code = 2 desc = oci runtime error: exec failed: container_linux.go:247: starting container process caused "process_linux.go:110: decoding init error from pipe caused \"read parent: connection reset by peer\""
```

این مشکل ظاهر می‌شود اگر CentOS 7 را با Docker نسخه 1.13.1.84

 اجرا کنید.
این نسخه از Docker می‌تواند kubelet را از اجرای فرآیند به کانتینر etcd مانع کند.

برای حل مشکل، یکی از این گزینه‌ها را انتخاب کنید:

- بازگشت به نسخه‌ای قدیمی‌تر از Docker مانند 1.13.1-75

  ```
  yum downgrade docker-1.13.1-75.git8633870.el7.centos.x86_64 docker-client-1.13.1-75.git8633870.el7.centos.x86_64 docker-common-1.13.1-75.git8633870.el7.centos.x86_64
  ```

- نصب یکی از نسخه‌های پیشنهادی به تازگی مانند 18.06:

  ```bash
  sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
  yum install docker-ce-18.06.1.ce-3.el7.x86_64
  ```

## امکان ارسال یک لیست از مقادیر جدا شده توسط کاما به آرگومان‌ها داخل پرچم `--component-extra-args`

پرچم‌های `kubeadm init` مانند `--component-extra-args` به شما اجازه می‌دهند تا آرگومان‌های سفارشی را به یک مولفه کنترل‌پلن مانند kube-apiserver منتقل کنید. با این حال، این مکانیسم به دلیل نوع زیرین استفاده شده برای تجزیه‌وتحلیل مقادیر، محدود است (`mapStringString`).

اگر تصمیم دارید که آرگومانی را منتقل کنید که از چند مقدار جدا شده توسط کاما پشتیبانی می‌کند مانند
`--apiserver-extra-args "enable-admission-plugins=LimitRanger,NamespaceExists"` این پرچم با خطا به خاطر
`flag: malformed pair, expect string=string` شکسته می‌شود. این اتفاق افتاده است چون لیست آرگومان‌ها برای
`--apiserver-extra-args` انتظار `key=value` را دارد و در این حالت `NamespaceExists` به عنوان یک کلید در نظر گرفته
می‌شود که مقدار ندارد.

در غیر این صورت، می‌توانید سعی کنید `key=value` را جدا کنید:
`--apiserver-extra-args "enable-admission-plugins=LimitRanger,enable-admission-plugins=NamespaceExists"`
اما این باعث می‌شود که کلید `enable-admission-plugins` فقط مقدار `NamespaceExists` را داشته باشد.

یک راه‌حل شناخته‌شده استفاده از [فایل پیکربندی kubeadm](/docs/reference/config-api/kubeadm-config.v1beta3/).

## kube-proxy scheduled before node is initialized by cloud-controller-manager

In cloud provider scenarios, kube-proxy can end up being scheduled on new worker nodes before
the cloud-controller-manager has initialized the node addresses. This causes kube-proxy to fail
to pick up the node's IP address properly and has knock-on effects to the proxy function managing
load balancers.

The following error can be seen in kube-proxy Pods:

```
server.go:610] Failed to retrieve node IP: host IP unknown; known addresses: []
proxier.go:340] invalid nodeIP, initializing kube-proxy with 127.0.0.1 as nodeIP
```

A known solution is to patch the kube-proxy DaemonSet to allow scheduling it on control-plane
nodes regardless of their conditions, keeping it off of other nodes until their initial guarding
conditions abate:

```
kubectl -n kube-system patch ds kube-proxy -p='{
  "spec": {
    "template": {
      "spec": {
        "tolerations": [
          {
            "key": "CriticalAddonsOnly",
            "operator": "Exists"
          },
          {
            "effect": "NoSchedule",
            "key": "node-role.kubernetes.io/control-plane"
          }
        ]
      }
    }
  }
}'
```

ردیابی مشکل برای این موضوع [اینجا](https://github.com/kubernetes/kubeadm/issues/1027) قرار دارد.

## `/usr` به صورت فقط خواندنی روی نود‌ها متصل می‌شود {#usr-mounted-read-only}

در توزیع‌های لینوکسی مانند Fedora CoreOS یا Flatcar Container Linux، دایرکتوری `/usr` به صورت یک فایل‌سیستم فقط خواندنی متصل می‌شود. برای [پشتیبانی از فلکس-والوم](https://github.com/kubernetes/community/blob/ab55d85/contributors/devel/sig-storage/flexvolume.md)،
اجزای Kubernetes مانند kubelet و kube-controller-manager از مسیر پیش‌فرض
`/usr/libexec/kubernetes/kubelet-plugins/volume/exec/` استفاده می‌کنند، اما دایرکتوری فلکس-والوم باید _قابل نوشتن باشد_
تا این ویژگی کار کند.

{{< note >}}
FlexVolume در نسخه v1.23 Kubernetes منسوخ شده است.
{{< /note >}}

برای رفع این مشکل، می‌توانید دایرکتوری فلکس-والوم را با استفاده از فایل پیکربندی
[کابرد](/docs/reference/config-api/kubeadm-config.v1beta3/)
پیکربندی کنید.

بر روی نود کنترل اصلی (ساخته شده با `kubeadm init`)، فایل زیر را با استفاده از `--config` ارسال کنید:

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
nodeRegistration:
  kubeletExtraArgs:
    volume-plugin-dir: "/opt/libexec/kubernetes/kubelet-plugins/volume/exec/"
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
controllerManager:
  extraArgs:
    flex-volume-plugin-dir: "/opt/libexec/kubernetes/kubelet-plugins/volume/exec/"
```

در نودهای پیوستنده:

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: JoinConfiguration
nodeRegistration:
  kubeletExtraArgs:
    volume-plugin-dir: "/opt/libexec/kubernetes/kubelet-plugins/volume/exec/"
```

در غیر این صورت، می‌توانید `/etc/fstab` را تغییر دهید تا نصب `/usr` را قابل نوشتن کنید، اما لطفاً توجه داشته باشید که این یک اصل طراحی توزیع لینوکسی را اصلاح می‌کند.

## `kubeadm upgrade plan` پیام خطای `context deadline exceeded` را چاپ می‌کند

این پیام خطایی است که هنگام ارتقاء یک خوشه Kubernetes با `kubeadm` در
صورت اجرای یک etcd خارجی نمایش داده می‌شود. این یک باگ اساسی نیست و به این دلیل اتفاق می‌افتد که نسخه‌های قدیمی kubeadm بررسی نسخه را بر روی خوشه etcd خارجی انجام می‌دهد.
می‌توانید با `kubeadm upgrade apply ...` ادامه دهید.

این مشکل تا نسخه 1.19 برطرف شده است.

## `kubeadm reset` `/var/lib/kubelet` را unmout می‌کند

اگر `/var/lib/kubelet` در حال مونت شدن است، انجام `kubeadm reset` آن را به طور موثر unmout خواهد کرد.

برای رفع این مشکل، دایرکتوری `/var/lib/kubelet` را بعد از انجام عملیات `kubeadm reset` مجدداً مونت کنید.

این یک افت رگرسیونی است که در kubeadm 1.15 معرفی شده است. این مشکل در نسخه 1.20 برطرف شده است.

## نمی‌توانید متریک‌سرو را به صورت امن در یک خوشه kubeadm استفاده کنید

در یک خوشه kubeadm، [metrics-server](https://github.com/kubernetes-sigs/metrics-server)
می‌تواند به صورت ناامن با ارسال `--kubelet-insecure-tls` به آن استفاده شود. این برای خوشه‌های تولیدی توصیه نمی‌شود.

اگر می‌خواهید از TLS بین metrics-server و kubelet استفاده کنید، مشکلی وجود دارد،
زیرا kubeadm یک گواهی خدمت‌دهنده خودنامه را برای kubelet ارائه می‌دهد. این می‌تواند باعث خطاهای زیر در سمت metrics-server شود:

```
x509: certificate signed by unknown authority
x509: certificate is valid for IP-foo not IP-bar
```

مشاهده کنید [Enabling signed kubelet serving certificates](/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/#kubelet-serving-certs)
برای درک اینکه چگونه می‌توانید kubelet‌ها را در یک خوشه kubeadm به دارای گواهی‌نامه‌های خدمت‌دهنده به‌صورت صحیح امضا شده پیکربندی کنید.

همچنین مشاهده کنید [How to run the metrics-server securely](https://github.com/kubernetes-sigs/metrics-server/blob/master/FAQ.md#how-to-run-metrics-server-securely).


## ارتقاء ناموفق به دلیل عدم تغییر هش etcd

فقط برای ارتقاء یک نود پلن کنترل با یک دودویی kubeadm v1.28.3 یا بعدی قابل اجرا است
جایی که نود در حال حاضر توسط نسخه‌های kubeadm v1.28.0، v1.28.1 یا v1.28.2 مدیریت می‌شود.

در اینجا پیام خطایی که ممکن است برخورد کنید، نمایش داده شده است:

```
[upgrade/etcd] Failed to upgrade etcd: couldn't upgrade control plane. kubeadm has tried to recover everything into the earlier state. Errors faced: static Pod hash for component etcd on Node kinder-upgrade-control-plane-1 did not change after 5m0s: timed out waiting for the condition
[upgrade/etcd] Waiting for previous etcd to become available
I0907 10:10:09.109104    3704 etcd.go:588] [etcd] attempting to see if all cluster endpoints ([https://172.17.0.6:2379/ https://172.17.0.4:2379/ https://172.17.0.3:2379/]) are available 1/10
[upgrade/etcd] Etcd was rolled back and is now available
static Pod hash for component etcd on Node kinder-upgrade-control-plane-1 did not change after 5m0s: timed out waiting for the condition
couldn't upgrade control plane. kubeadm has tried to recover everything into the earlier state. Errors faced
k8s.io/kubernetes/cmd/kubeadm/app/phases/upgrade.rollbackOldManifests
	cmd/kubeadm/app/phases/upgrade/staticpods.go:525
k8s.io/kubernetes/cmd/kubeadm/app/phases/upgrade.upgradeComponent
	cmd/kubeadm/app/phases/upgrade/staticpods.go:254
k8s.io/kubernetes/cmd/kubeadm/app/phases/upgrade.performEtcdStaticPodUpgrade
	cmd/kubeadm/app/phases/upgrade/staticpods.go:338
...
```

دلیل این شکست این است که نسخه‌های مورد تاثیر یک فایل مناسب etcd را با تنظیمات پیش‌فرض نامطلوب در PodSpec تولید می‌کنند. این موجب تفاوتی از مقایسه مانیفست می‌شود و kubeadm انتظار دارد که یک تغییر در هش Pod را مشاهده کند، اما kubelet هیچ‌گاه هش را به‌روز نخواهد کرد.

دو راه برای حل این مشکل در خوشه خود وجود دارد:

- ارتقاء etcd را می‌توانید بین نسخه‌های متاثر و v1.28.3 (یا بالاتر) رد کنید با استفاده از:

  ```shell
  kubeadm upgrade {apply|node} [version] --etcd-upgrade=false
  ```

  این توصیه نمی‌شود اگر نسخه جدیدی از etcd توسط یک نسخه بعدی v1.28 معرفی شود.

- قبل از ارتقاء، مانیفست برای پاد استاتیک etcd را برای حذف ویژگی‌های پیش‌فرض مشکل‌زا پچ کنید:

  ```patch
  diff --git a/etc/kubernetes/manifests/etcd_defaults.yaml b/etc/kubernetes/manifests/etcd_origin.yaml
  index d807ccbe0aa..46b35f00e15 100644
  --- a/etc/kubernetes/manifests/etcd_defaults.yaml
  +++ b/etc/kubernetes/manifests/etcd_origin.yaml
  @@ -43,7 +43,6 @@ spec:
          scheme: HTTP
        initialDelaySeconds: 10
        periodSeconds: 10
  -      successThreshold: 1
        timeoutSeconds: 15
      name: etcd
      resources:
  @@ -59,26 +58,18 @@ spec:
          scheme: HTTP
        initialDelaySeconds: 10
        periodSeconds: 10
  -      successThreshold: 1
        timeoutSeconds: 15
  -    terminationMessagePath: /dev/termination-log
  -    terminationMessagePolicy: File
      volumeMounts:
      - mountPath: /var/lib/etcd
        name: etcd-data
      - mountPath: /etc/kubernetes/pki/etcd
        name: etcd-certs
  -  dnsPolicy: ClusterFirst
  -  enableServiceLinks: true
    hostNetwork: true
    priority: 2000001000
    priorityClassName: system-node-critical
  -  restartPolicy: Always
  -  schedulerName: default-scheduler
    securityContext:
      seccompProfile:
        type: RuntimeDefault
  -  terminationGracePeriodSeconds: 30
    volumes:
    - hostPath:
        path: /etc/kubernetes/pki/etcd
  ```

اطلاعات بیشتر در [ردیابی مشکل](https://github.com/kubernetes/kubeadm/issues/2927) برای این اشکال موجود است.
