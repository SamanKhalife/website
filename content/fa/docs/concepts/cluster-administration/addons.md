---
title: نصب افزونه‌ها
content_type: concept
weight: 150
---

<!-- overview -->

{{% thirdparty-content %}}

افزونه‌ها قابلیت‌های Kubernetes را گسترش می‌دهند.

این صفحه فهرستی از افزونه‌های موجود را نشان می‌دهد و به دستورالعمل‌های نصب آن‌ها مراجعه می‌کند. این فهرست تلاشی برای جامع بودن ندارد.

<!-- body -->

## شبکه‌سازی و سیاست شبکه

* [ACI](https://www.github.com/noironetworks/aci-containers) شبکه‌سازی و امنیت شبکه یکپارچه با Cisco ACI را فراهم می‌کند.
* [Antrea](https://antrea.io/) در لایه ۳/۴ عمل می‌کند تا خدمات شبکه و امنیت را برای Kubernetes فراهم آورد، با استفاده از Open vSwitch به عنوان دیتا پلن شبکه. Antrea یک [پروژه CNCF در سطح Sandbox است](https://www.cncf.io/projects/antrea/).
* [Calico](https://www.tigera.io/project-calico/) ارائه دهنده شبکه و سیاست شبکه است. Calico از مجموعه‌ای انعطاف‌پذیر از گزینه‌های شبکه پشتیبانی می‌کند تا بتوانید گزینه‌ی موثرتر را برای وضعیت خود انتخاب کنید، شامل شبکه‌های بدون پوشش و پوششی، با یا بدون BGP. Calico از همان موتور برای اجرای سیاست شبکه برای میزبان‌ها، پادها و (اگر از Istio & Envoy استفاده می‌کنید) برنامه‌ها در لایه شبکه مشبکه استفاده می‌کند.
* [Canal](https://projectcalico.docs.tigera.io/getting-started/kubernetes/flannel/flannel) Flannel و Calico را ترکیب می‌کند و شبکه‌سازی و سیاست شبکه را فراهم می‌کند.
* [Cilium](https://github.com/cilium/cilium) یک راه‌حل شبکه‌سازی، مشاهدات و امنیت با یک دیتا پلن مبنی بر eBPF است. Cilium یک شبکه لایه ۳ ساده فلت با توانایی پوشش دادن به چندین کلاستر در حالت‌های مسیریابی یا پوشش/برگه‌بندی، و می‌تواند سیاست‌های شبکه را در L3-L7 با استفاده از یک مدل امنیتی مبتنی بر هویت که از آدرسدهی شبکه جدا شده است، اجرا کند. Cilium می‌تواند به عنوان یک جایگزین برای kube-proxy عمل کند؛ همچنین ویژگی‌های بیشتری را برای مشاهده و امنیت انتخابی ارائه می‌دهد. Cilium یک [پروژه CNCF در سطح Graduated است](https://www.cncf.io/projects/cilium/).
* [CNI-Genie](https://github.com/cni-genie/CNI-Genie) امکانات Kubernetes را فراهم می‌کند تا به طور شفاف به یکی از پلاگین‌های CNI مانند Calico، Canal، Flannel یا Weave متصل شود. CNI-Genie یک [پروژه CNCF در سطح Sandbox است](https://www.cncf.io/projects/cni-genie/).
* [Contiv](https://contivpp.io/) شبکه‌سازی قابل تنظیم (L3 اصلی با استفاده از BGP، پوشش با استفاده از vxlan، L2 کلاسیک و چارچوب سیاست غنی) را برای موارد استفاده مختلف و چارچوب سیاست غنی فراهم می‌کند. پروژه Contiv کاملاً [منبع باز است](https://github.com/contiv). [نصب‌کننده](https://github.com/contiv/install) گزینه‌های نصب بر پایه kubeadm و non-kubeadm را ارائه می‌دهد.
* [Contrail](https://www.juniper.net/us/en/products-services/sdn/contrail/contrail-networking/) بر پایه [Tungsten Fabric](https://tungsten.io) یک پلتفرم مجازی‌سازی شبکه و مدیریت سیاست چندابرگی است. Contrail و Tungsten Fabric با سیستم‌های ارکستراسیون مانند Kubernetes، OpenShift، OpenStack و Mesos یکپارچه شده‌اند و حالت‌های جداکنندگی برای ماشین‌های مجازی، کانتینرها/پادها و بارهای کاری فلزی خالی فراهم می‌کنند.
* [Flannel](https://github.com/flannel-io/flannel#deploying-flannel-manually) یک ارائه‌دهنده شبکه‌ی پوششی است که می‌توان با Kubernetes استفاده کرد.
* [Gateway API](/docs/concepts/services-networking/gateway/) یک پروژه متن باز است که توسط جامعه [SIG Network](https://github.com/kubernetes/community/tree/master/sig-network) مدیریت می‌شود و یک API بیانی، قابل توسعه و مبنی بر نقش برای مدل‌سازی شبکه‌دهی سرویس فراهم می‌کند.
* [Knitter](https://github.com/ZTE/Knitter/) یک افزونه برای پشتیبانی از چندین رابط شبکه در یک pod Kubernetes است.
* [Multus](https://github.com/k8snetworkplumbingwg/multus-cni) یک افزونه چندگانه برای پشتیبانی از شبکه‌های متعدد در Kubernetes برای پشتیبانی از همه پلاگین‌های CNI (مانند Calico، Cilium، Contiv، Flannel)، به علاوه SRIOV، DPDK، OVS-DPDK و بارهای کاری مبتنی بر VPP است.
* [OVN-Kubernetes](https://github.com/ovn-org/ovn-kubernetes/) یک ارائه‌دهنده شبکه برای Kubernetes بر پایه [OVN (Open Virtual Network)](https://github.com/ovn-org/ovn/) است، یک اجرایی از شبکه مجازی‌سازی که از پروژه Open vSwitch (OVS) برگرفته است. OVN-Kubernetes یک اجرایی بر مبنای پوششی از پیاده‌سازی شبکه برای Kubernetes فراهم می‌کند، شامل یک پیاده‌سازی مبتنی بر OVS از بار‌تقسیم و سیاست شبکه.
* [Nodus](https://github.com/akraino-edge-stack/icn-nodus) یک افزونه کنترل کننده CNI مبتنی بر OVN است که برای تأمین چینش سرویس بر اساس سرویس نوآورانه ابری ارائه می‌دهد(SFC).
* [NSX-T](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/index.html) Container Plug-in (NCP)
  ارائه ادغام بین VMware NSX-T و ارکستراتورهای کانتینری مانند Kubernetes، و همچنین ادغام بین NSX-T و پلتفرم‌های CaaS/PaaS مبتنی بر کانتینر مانند Pivotal Container Service (PKS) و OpenShift.
* [Nuage](https://github.com/nuagenetworks/nuage-kubernetes/blob/v5.1.1-1/docs/kubernetes-1-installation.rst)
  یک پلتفرم SDN است که شبکه‌سازی مبنی بر سیاست بین Pods Kubernetes و محیط‌های غیر Kubernetes با دیداری و نظارت امنیتی فراهم می‌کند.
* [Romana](https://github.com/romana) یک راه‌حل شبکه‌سازی لایه ۳ برای شبکه‌های pod است که همچنین پشتیبانی از [NetworkPolicy](/docs/concepts/services-networking/network-policies/) API را دارد.
* [Spiderpool](https://github.com/spidernet-io/spiderpool) یک راه‌حل شبکه‌سازی underlay و RDMA برای Kubernetes است. Spiderpool روی فلز خام، ماشین‌های مجازی و محیط‌های ابر عمومی پشتیبانی می‌شود.
* [Weave Net](https://github.com/rajch/weave#using-weave-on-kubernetes)
  شبکه‌سازی و سیاست شبکه را فراهم می‌کند، همچنین در هر دو طرف یک جداشدگی شبکه کار می‌کند و نیازی به پایگاه داده خارجی ندارد.

## کشف خدمات

* [CoreDNS](https://coredns.io) یک سرور DNS انعطاف‌پذیر و قابل گسترش است که می‌توان به عنوان DNS داخلی برای پادها نصب کرد.

## مصورسازی و کنترل

* [Dashboard](https://github.com/kubernetes/dashboard#kubernetes-dashboard)
  یک رابط وب داشبورد برای Kubernetes است.
* [Weave Scope](https://www.weave.works/documentation/scope-latest-installing/#k8s) یک ابزار برای مصورسازی کانتینرها، پادها، خدمات و موارد دیگر است.

## زیرساخت

* [KubeVirt](https://kubevirt.io/user-guide/#/installation/installation) یک افزونه برای اجرای ماشین‌های مجازی بر روی Kubernetes است. معمولاً در خوشه‌های فلزی خام اجرا می‌شود.
* [node problem detector](https://github.com/kubernetes/node-problem-detector)
  روی گره‌های Linux اجرا می‌شود و مشکلات سیستم را به عنوان یکی از [رویدادها](/docs/reference/kubernetes-api/cluster-resources/event-v1/) یا [شرایط گره](/docs/concepts/architecture/nodes/#condition) گزارش می‌دهد.

## ابزارهای نصب

* [kube-state-metrics](/docs/concepts/cluster-administration/kube-state-metrics)

## افزونه‌های قدیمی

چندین افزونه دیگر در دایرکتوری منسوخ [cluster/addons](https://git.k8s.io/kubernetes/cluster/addons) مستند شده‌اند.

افزونه‌های نگهداری شده خوب باید به اینجا متصل شوند. PRs خوش‌آمدید!
