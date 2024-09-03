---
reviewers:
- aravindhp
- jayunit100
- jsturtevant
- marosset
title: Windows debugging tips
content_type: concept
---

<!-- overview -->

این مستند شامل نکاتی جهت رفع مشکلات مربوط به ویندوز در Kubernetes است.

<!-- body -->

## اشکال‌زدایی سطح نود {#troubleshooting-node}

1. پادهای من در وضعیت "Container Creating" گیر کرده‌اند یا به صورت تکراری دوباره شروع به راه‌اندازی می‌کنند

   مطمئن شوید که تصویر pause شما با نسخه ویندوز OS شما سازگار است. 
   برای اطلاعات بیشتر، [تصویر Pause](/docs/concepts/windows/intro/#pause-container) را بررسی کنید.

   {{< note >}}
   اگر containerd به عنوان runtime کانتینر شما استفاده می‌شود، تصویر pause در فیلد `plugins.plugins.cri.sandbox_image` فایل پیکربندی config.toml مشخص می‌شود.
   {{< /note >}}

1. وضعیت پادهای من به عنوان `ErrImgPull` یا `ImagePullBackOff` نمایش داده می‌شود

   مطمئن شوید که پاد شما به یک نود ویندوز [سازگار](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility) جدول‌بندی شده است.

   برای اطلاعات بیشتر در مورد اینکه چگونه نود مناسب را برای پاد خود مشخص کنید، در [این راهنما](/docs/concepts/windows/user-guide/#ensuring-os-specific-workloads-land-on-the-appropriate-container-host) بیشتر بخوانید.

## اشکال‌زدایی شبکه {#troubleshooting-network}

1. پادهای ویندوز من دسترسی به شبکه ندارند

   اگر از ماشین‌های مجازی استفاده می‌کنید، اطمینان حاصل کنید که MAC spoofing در همه آداپتورهای شبکه VM فعال شده است.

1. پادهای ویندوز من نمی‌توانند به منابع خارجی ping دهند

   پادهای ویندوز دارای قوانین خروجی برای پروتکل ICMP نیستند. با این حال، TCP/UDP پشتیبانی می‌شود. در صورت تلاش برای نمایش ارتباط با منابع خارج از خوشه، دستورهای `curl <IP>` را به جای `ping <IP>` معادل استفاده کنید.

   اگر هنوز با مشکلات روبرو هستید، احتمالاً پیکربندی شبکه شما در [cni.conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) نیاز به توجه اضافی دارد. همیشه می‌توانید این فایل استاتیک را ویرایش کنید. به ویژگی‌های مورد نیاز شبکه Kubernetes برای ارتباط خوشه بدون NAT داخلی مراجعه کنید.

1. نود ویندوز من نمی‌تواند به خدمات نوع `NodePort` دسترسی داشته باشد

   دسترسی NodePort محلی از خود نود ناموفق است. این یک محدودیت شناخته‌شده است. دسترسی NodePort از سایر نودها یا مشتریان خارجی کار می‌کند.

1. vNICها و HNS endpoints کانتینرها حذف می‌شوند

   این مشکل ممکن است زمانی رخ دهد که پارامتر `hostname-override` به [kube-proxy](/docs/reference/command-line-tools-reference/kube-proxy/) منتقل نشود. برای رفع این مشکل، کاربران باید نام میزبان را به kube-proxy منتقل کنند به صورت زیر:

   ```powershell
   C:\k\kube-proxy.exe --hostname-override=$(hostname)
   ```

1. نود ویندوز من نمی‌تواند به خدمات خود با استفاده از IP سرویس دسترسی داشته باشد

   این یک محدودیت شناخته‌شده در پشته شبکه ویندوز است. با این حال، پادهای ویندوز می‌توانند به IP سرویس دسترسی داشته باشند.

1. آداپتور شبکه موجود نیست هنگامی که kubelet راه‌اندازی می‌کند

   پشته شبکه ویندوز نیاز به آداپتور مجازی برای کارکرد شبکه Kubernetes دارد. اگر دستورات زیر (در یک پوشه admin shell) نتیجه‌ای نداشته باشند، ایجاد شبکه مجازی — یک پیش‌نیاز ضروری برای کار kubelet — ناموفق بوده است:

   ```powershell
   Get-HnsNetwork | ? Name -ieq "cbr0"
   Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
   ```

   اغلب ارزشمند است که پارامتر [InterfaceName](https://github.com/microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1#L7) اسکریپت start.ps1 را ویرایش کنید، در مواردی که آداپتور شبکه میزبان "Ethernet" نباشد. در غیر این صورت، خروجی اسکریپت start-kubelet.ps1 را مشاهده کنید تا مشکلات ایجاد شبکه مجازی را بررسی کنید.

1. رزولوشن DNS به درستی کار نمی‌کند

   محدودیت‌های DNS برای ویندوز را در این [بخش](/docs/concepts/services-networking/dns-pod-service/#dns-windows) بررسی کنید.

1. `kubectl port-forward` با "unable to do port forwarding: wincat not found" ناموفق می‌ماند

   این ویژگی در Kubernetes 1.15 به وجود آمده است با اضافه کردن `wincat.exe` به کانتینر زیرساخت استراحتی `mcr.microsoft.com/oss/kubernetes/pause:3.6`.
   مطمئن شوید از نسخه پشتیبانی شده Kubernetes استفاده کنید.
   اگر می‌خواهید ک

انتینر زیرساخت استراحتی خود را بسازید، حتماً [wincat](https://github.com/kubernetes/kubernetes/tree/master/build/pause/windows/wincat) را در آن اضافه کنید.

1. نصب Kubernetes من به دلیل آنکه نود سرور ویندوز من پشت پروکسی است، ناموفق است

   اگر پشت پروکسی هستید، متغیرهای محیطی PowerShell زیر باید تعریف شوند:

   ```PowerShell
   [Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://proxy.example.com:80/", [EnvironmentVariableTarget]::Machine)
   [Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://proxy.example.com:443/", [EnvironmentVariableTarget]::Machine)
   ```

### اشکال‌زدایی Flannel

1. با Flannel، نودهای من پس از پیوستن مجدد به خوشه مشکل دارند

   هرگاه یک نود از پیش حذف شده به خوشه پیوسته می‌شود، flannelD سعی می‌کند یک زیرشبکه پاد جدید به نود اختصاص دهد. کاربران باید فایل‌های پیکربندی قبلی زیرشبکه پاد را در مسیرهای زیر حذف کنند:

   ```powershell
   Remove-Item C:\k\SourceVip.json
   Remove-Item C:\k\SourceVipRequest.json
   ```

1. Flanneld در حالت "Waiting for the Network to be created" گیر کرده است

   گزارش‌های متعددی از این [مشکل](https://github.com/coreos/flannel/issues/1066) وجود دارد؛ احتمالاً یک مسئله زمان‌بندی برای زمانی است که IP مدیریتی شبکه flannel تنظیم می‌شود. یک راه‌حل موقت است که `start.ps1` را مجدداً راه‌اندازی کنید یا به صورت دستی به صورت زیر:

   ```powershell
   [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
   C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
   ```

1. پادهای ویندوز من نمی‌توانند به دلیل عدم وجود `/run/flannel/subnet.env` راه‌اندازی شوند

   این نشان می‌دهد که Flannel به درستی راه‌اندازی نشده است. شما می‌توانید یا سعی کنید `flanneld.exe` را دوباره راه‌اندازی کنید یا می‌توانید فایل‌ها را به صورت دستی از `/run/flannel/subnet.env` روی مستر Kubernetes به `C:\run\flannel\subnet.env` روی نود کارگر ویندوز کپی کنید و `FLANNEL_SUBNET` را به یک شماره مختلف تغییر دهید. به عنوان مثال، اگر زیرشبکه 10.244.4.1/24 نود مورد نظر است:

   ```env
   FLANNEL_NETWORK=10.244.0.0/16
   FLANNEL_SUBNET=10.244.4.1/24
   FLANNEL_MTU=1500
   FLANNEL_IPMASQ=true
   ```

### تحقیقات بیشتر

اگر این مراحل مشکل شما را حل نکرد، می‌توانید از طریق موارد زیر کمک بگیرید:

* [تاپیک Windows Server Container در StackOverflow](https://stackoverflow.com/questions/tagged/windows-server-container)
* [انجمن رسمی Kubernetes](https://discuss.kubernetes.io/)
* [کانال SIG-Windows در Kubernetes Slack](https://kubernetes.slack.com/messages/sig-windows)