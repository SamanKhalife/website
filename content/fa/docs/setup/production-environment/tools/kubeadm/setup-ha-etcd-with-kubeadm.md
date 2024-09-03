### راه اندازی خوشه بازدید بالا از اتس

{{< note >}}
در حالی که kubeadm به عنوان ابزار مدیریتی برای گره‌های etcd خارجی در این راهنما استفاده می‌شود، لطفاً توجه داشته باشید که kubeadm نقشی در چرخه مدیریتی گواهی‌ها یا ارتقاء‌های این گره‌ها را پشتیبانی نمی‌کند. طرح بلند مدت برای ایجاد امکان این است که ابزار
[etcdadm](https://github.com/kubernetes-sigs/etcdadm)
را برای مدیریت این جنبه‌ها تقویت کند.
{{< /note >}}

به طور پیش‌فرض، kubeadm یک نمونه محلی از etcd را در هر گره کنترل اجرا می‌کند. همچنین امکان دارد خوشه etcd را به عنوان خارجی در نظر بگیرد و نمونه‌های etcd را در میزبان‌های جداگانه فراهم کند. تفاوت‌های این دو رویکرد در
[گزینه‌های برای توپولوژی بازدید بالا](/docs/setup/production-environment/tools/kubeadm/ha-topology)
پوشش داده شده است.

این وظیفه از فرایند ایجاد یک خوشه بازدید بالای خارجی etcd که شامل سه عضو است که توسط kubeadm در زمان ایجاد خوشه استفاده می‌شود، عبور می‌کند.

## {{% heading "پیش‌نیازها" %}}

- سه میزبان که بتوانند از طریق گذرگاه‌های TCP 2379 و 2380 با یکدیگر صحبت کنند. این مستند فرض می‌کند که این درگاه‌های پیش‌فرض هستند. با این حال، آنها قابل تنظیم از طریق فایل پیکربندی kubeadm هستند.
- هر میزبان باید دارای systemd و یک پوسته سازگار با bash باشد.
- هر میزبان باید [دریافتنی](/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
container runtime، kubelet، و kubeadm نصب شده است.
- هر میزبان باید دسترسی به رجیستری تصویر container Kubernetes (`registry.k8s.io`) یا لیست/بلعکس تصویر etcd مورد نیاز به کمک
`kubeadm config images list/pull`. این راهنما نمونه‌های etcd را به صورت
[pods ثابت](/docs/tasks/configure-pod-container/static-pod/)
که توسط یک kubelet مدیریت می‌شود تنظیم می‌کند.
- زیرساختی برای کپی کردن فایل‌ها بین میزبان‌ها. به عنوان مثال `ssh` و `scp`
می‌توانند این نیاز را برآورده کنند.

## راه اندازی خوشه

رویکرد کلی این است که تمامی گواهی‌ها را در یک میزبان ایجاد کنید و فقط فایل‌های _ضروری_ را به سایر میزبان‌ها توزیع کنید.

{{< note >}}
kubeadm تمام ماشین‌های رمزنگاری لازم برای تولید
گواهی‌هایی که در زیر توضیح داده شده‌اند؛ برای این مثال ابزار رمزنگاری دیگری مورد نیاز نیست.
{{< /note >}}

{{< note >}}
مثال‌های زیر از آدرس‌های IPv4 استفاده می‌کنند، اما شما همچنین می‌توانید kubeadm، kubelet و etcd را برای استفاده از آدرس‌های IPv6 پیکربندی کنید. پشتیبانی از ارتباطات دوگانه توسط برخی گزینه‌های Kubernetes پشتیبانی می‌شود، اما توسط etcd پشتیبانی نمی‌شود. برای اطلاعات بیشتر
درباره پشتیبانی
از پشتیبانی بیشتر با kubeadm
(/docs/setup/production-environment/tools/kubeadm/dual-stack-support/).
{{< /note >}}

1. پیکربندی kubelet برای مدیریت خدمات etcd.

   {{< note >}}شما باید این کار را در هر میزبانی که etcd باید اجرا شود، انجام دهید.{{< /note >}}
   از آنجایی که اولین بار etcd ایجاد شد، شما باید اولویت خدمت را با ایجاد یک فایل واحد جدید
   که از اولویت بالاتری نسبت به فایل واحد kubelet ارائه شده توسط kubeadm استفاده می‌کند.

   ```sh
   cat << EOF > /etc/systemd/system/kubelet.service.d/kubelet.conf
   # جایگزینی "systemd" با درایور cgroup محرک runtime شما. در مقدار پیش‌فرض در kubelet "cgroupfs".
   # جایگزین کردن مقدار "containerRuntimeEndpoint" برای یک runtime مختلف اگر نیاز باشد.
   #
   apiVersion: kubelet.config.k8s.io/v1beta1
   kind: KubeletConfiguration
   authentication:
     anonymous:
       enabled: false
     webhook:
       enabled: false
   authorization:
     mode: AlwaysAllow
   cgroupDriver: systemd
   address: 127.0.0.1
   containerRuntimeEndpoint: unix:///var/run/containerd/containerd.sock
   staticPodPath: /etc/kubernetes/manifests
   EOF

   cat << EOF > /etc/systemd/system/kubelet.service.d/20-etcd-service-manager.conf
   [Service]
   ExecStart=
   ExecStart=/usr/bin/kubelet --config=/etc/systemd/system/kubelet.service.d/kubelet.conf
   Restart=always
   EOF

   systemctl daemon-reload
   systemctl restart kubelet
   ```

   برای اطمینان از اجرای kubelet وضعیت آن را بررسی کنید.

   ```sh
   systemctl status kubelet
   ```

1. ای

جاد فایل‌های پیکربندی برای kubeadm.

   یک فایل پیکربندی kubeadm برای هر میزبانی که یک عضو etcd روی آن اجرا خواهد شد با استفاده از اسکریپت زیر ایجاد کنید.

   ```sh
   # HOST0, HOST1 و HOST2 را با آدرس‌های IP میزبان‌های خود به‌روز کنید
   export HOST0=10.0.0.6
   export HOST1=10.0.0.7
   export HOST2=10.0.0.8

   # NAME0, NAME1 و NAME2 را با نام‌های میزبان خود به‌روز کنید
   export NAME0="infra0"
   export NAME1="infra1"
   export NAME2="infra2"

   # دایرکتوری‌های موقتی برای ذخیره فایل‌ها که در نهایت بر روی میزبان‌های دیگر قرار می‌گیرند ایجاد کنید
   mkdir -p /tmp/${HOST0}/ /tmp/${HOST1}/ /tmp/${HOST2}/

   HOSTS=(${HOST0} ${HOST1} ${HOST2})
   NAMES=(${NAME0} ${NAME1} ${NAME2})

   for i in "${!HOSTS[@]}"; do
   HOST=${HOSTS[$i]}
   NAME=${NAMES[$i]}
   cat << EOF > /tmp/${HOST}/kubeadmcfg.yaml
   ---
   apiVersion: "kubeadm.k8s.io/v1beta3"
   kind: InitConfiguration
   nodeRegistration:
       name: ${NAME}
   localAPIEndpoint:
       advertiseAddress: ${HOST}
   ---
   apiVersion: "kubeadm.k8s.io/v1beta3"
   kind: ClusterConfiguration
   etcd:
       local:
           serverCertSANs:
           - "${HOST}"
           peerCertSANs:
           - "${HOST}"
           extraArgs:
               initial-cluster: ${NAMES[0]}=https://${HOSTS[0]}:2380,${NAMES[1]}=https://${HOSTS[1]}:2380,${NAMES[2]}=https://${HOSTS[2]}:2380
               initial-cluster-state: new
               name: ${NAME}
               listen-peer-urls: https://${HOST}:2380
               listen-client-urls: https://${HOST}:2379
               advertise-client-urls: https://${HOST}:2379
               initial-advertise-peer-urls: https://${HOST}:2380
   EOF
   done
   ```
Sure, here's the markdown formatted text in Persian:


1. تولید معیارگذاری‌گر.

   اگر یک CA دارید، تنها کاری که لازم است انجام دهید کپی کردن `crt` و `key` فایل CA به `/etc/kubernetes/pki/etcd/ca.crt` و `/etc/kubernetes/pki/etcd/ca.key` است. بعد از کپی این فایل‌ها، به مرحله بعدی، "ایجاد گواهی‌نامه‌ها برای هر عضو" بروید.

   اگر شما هنوز یک CA ندارید، این دستور را روی $HOST0 (جایی که فایل‌های پیکربندی برای kubeadm ایجاد کرده‌اید) اجرا کنید.

   ```
   kubeadm init phase certs etcd-ca
   ```

   این دو فایل را ایجاد می‌کند:

   - `/etc/kubernetes/pki/etcd/ca.crt`
   - `/etc/kubernetes/pki/etcd/ca.key`

1. ایجاد گواهی‌نامه‌ها برای هر عضو.

   ```sh
   kubeadm init phase certs etcd-server --config=/tmp/${HOST2}/kubeadmcfg.yaml
   kubeadm init phase certs etcd-peer --config=/tmp/${HOST2}/kubeadmcfg.yaml
   kubeadm init phase certs etcd-healthcheck-client --config=/tmp/${HOST2}/kubeadmcfg.yaml
   kubeadm init phase certs apiserver-etcd-client --config=/tmp/${HOST2}/kubeadmcfg.yaml
   cp -R /etc/kubernetes/pki /tmp/${HOST2}/
   # پاکسازی گواهی‌نامه‌های غیرقابل استفاده مجدد
   find /etc/kubernetes/pki -not -name ca.crt -not -name ca.key -type f -delete

   kubeadm init phase certs etcd-server --config=/tmp/${HOST1}/kubeadmcfg.yaml
   kubeadm init phase certs etcd-peer --config=/tmp/${HOST1}/kubeadmcfg.yaml
   kubeadm init phase certs etcd-healthcheck-client --config=/tmp/${HOST1}/kubeadmcfg.yaml
   kubeadm init phase certs apiserver-etcd-client --config=/tmp/${HOST1}/kubeadmcfg.yaml
   cp -R /etc/kubernetes/pki /tmp/${HOST1}/
   find /etc/kubernetes/pki -not -name ca.crt -not -name ca.key -type f -delete

   kubeadm init phase certs etcd-server --config=/tmp/${HOST0}/kubeadmcfg.yaml
   kubeadm init phase certs etcd-peer --config=/tmp/${HOST0}/kubeadmcfg.yaml
   kubeadm init phase certs etcd-healthcheck-client --config=/tmp/${HOST0}/kubeadmcfg.yaml
   kubeadm init phase certs apiserver-etcd-client --config=/tmp/${HOST0}/kubeadmcfg.yaml
   # نیازی به جابجایی گواهی‌نامه‌ها نیست زیرا برای HOST0 هستند

   # پاکسازی گواهی‌نامه‌هایی که نباید از این میزبان به بیرون منتقل شود
   find /tmp/${HOST2} -name ca.key -type f -delete
   find /tmp/${HOST1} -name ca.key -type f -delete
   ```

1. کپی کردن گواهی‌نامه‌ها و تنظیمات kubeadm.

   گواهی‌نامه‌ها ایجاد شده و حالا باید به میزبان‌های مربوطه منتقل شوند.

   ```sh
   USER=ubuntu
   HOST=${HOST1}
   scp -r /tmp/${HOST}/* ${USER}@${HOST}:
   ssh ${USER}@${HOST}
   USER@HOST $ sudo -Es
   root@HOST $ chown -R root:root pki
   root@HOST $ mv pki /etc/kubernetes/
   ```

1. اطمینان از وجود تمام فایل‌های مورد نیاز.

   لیست کامل فایل‌های مورد نیاز در $HOST0:

   ```
   /tmp/${HOST0}
   └── kubeadmcfg.yaml
   ---
   /etc/kubernetes/pki
   ├── apiserver-etcd-client.crt
   ├── apiserver-etcd-client.key
   └── etcd
       ├── ca.crt
       ├── ca.key
       ├── healthcheck-client.crt
       ├── healthcheck-client.key
       ├── peer.crt
       ├── peer.key
       ├── server.crt
       └── server.key
   ```

   در $HOST1:

   ```
   $HOME
   └── kubeadmcfg.yaml
   ---
   /etc/kubernetes/pki
   ├── apiserver-etcd-client.crt
   ├── apiserver-etcd-client.key
   └── etcd
       ├── ca.crt
       ├── healthcheck-client.crt
       ├── healthcheck-client.key
       ├── peer.crt
       ├── peer.key
       ├── server.crt
       └── server.key
   ```

   در $HOST2:

   ```
   $HOME
   └── kubeadmcfg.yaml
   ---
   /etc/kubernetes/pki
   ├── apiserver-etcd-client.crt
   ├── apiserver-etcd-client.key
   └── etcd
       ├── ca.crt
       ├── healthcheck-client.crt
       ├── healthcheck-client.key
       ├── peer.crt
       ├── peer.key
       ├── server.crt
       └── server.key
   ```

1. ایجاد manifest‌های پاد استاتیک.

   حالا که گواهی‌نامه‌ها و پیکربندی‌ها در محل قرار گرفته‌اند، زمان آن رسیده است که manifest‌ها را ایجاد کنید. روی هر میزبان دستور kubeadm را برای تولید یک manifest استاتیک برای etcd اجرا کنید.

   ```sh
   root@HOST0 $ kubeadm init phase etcd local --config=/tmp/${HOST0}/kubeadmcfg.yaml
   root@HOST1 $ kubeadm init phase etcd local --config=$HOME/kubeadmcfg.yaml
   root@HOST2 $ kubeadm init phase etcd local --config=$HOME/kubeadmcfg.yaml
   ```

1. اختیاری: بررسی سلامت خوشه.

   اگر `etcdctl` در دسترس نیست، می‌توانید این ابزار را درون یک تصویر container اجرا کنید. شما می‌توانید این کار را مستقیماً با container runtime خود با استفاده از ابزاری مانند `crictl run` انجام دهید و نه از طریق Kubernetes.

   ```sh
   ETCDCTL_API=3 etcdctl \
   --cert /etc/kubernetes/pki/etcd/peer.crt \
   --key /etc/kubernetes/pki/etcd/peer.key \
   --cacert /etc/kubernetes/pki/etcd/ca.crt \
   --endpoints https://${HOST0}:2379 endpoint health
   ...
   https://[HOST0 IP]:2379 is healthy: successfully committed proposal: took = 16.283339ms
   https://[HOST1 IP]:2379 is healthy: successfully committed proposal: took = 19.44402ms
   https://[HOST2 IP]:2379 is healthy: successfully committed proposal: took = 35.926451ms
   ```

   - `${HOST0}` را به آدرس IP میزبانی که در حال آزمایش هستید تنظیم کنید.

## {{% heading "whatsnext" %}}

با داشتن یک خوشه etcd با 3 عضو

 کارآمد، می‌توانید به راحتی ادامه دهید به تنظیم یک سیستم کنترل پلن برتر با استفاده از [روش etcd خارجی با kubeadm](/docs/setup/production-environment/tools/kubeadm/high-availability/).
