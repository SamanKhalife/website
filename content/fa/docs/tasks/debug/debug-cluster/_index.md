---
title: "رفع مشکلات کلاسترها"
description: رفع مشکلات متداول کلاسترها.
weight: 20
reviewers:
- davidopp
no_list: true
---

<!-- overview -->

این مستند درباره رفع مشکلات کلاسترهاست؛ ما فرض می‌کنیم که شما قبلاً برنامه خود را به عنوان عامل اصلی مشکل که تجربه می‌کنید، نپذیرفته‌اید. برای راهنمایی در اشکال‌زدایی برنامه، به [راهنمای اشکال‌زدایی برنامه](/docs/tasks/debug/debug-application/) مراجعه کنید. همچنین می‌توانید به [سند مروری راه‌اندازی](/docs/tasks/debug/) برای اطلاعات بیشتر مراجعه کنید.

برای اشکال‌زدایی {{<glossary_tooltip text="kubectl" term_id="kubectl">}}، به [رفع اشکال kubectl](/docs/tasks/debug/debug-cluster/troubleshoot-kubectl/) مراجعه کنید.

<!-- body -->

## فهرست کردن کلاستر شما

اولین چیزی که باید در اشکال‌زدایی کلاستر خود بررسی کنید، ثبت صحیح تمامی نودهایتان است.

دستور زیر را اجرا کنید:

```shell
kubectl get nodes
```

و بررسی کنید که همه نودهایی که انتظار دارید حاضر هستند و همه آن‌ها در وضعیت `Ready` هستند.

برای دریافت اطلاعات دقیق‌تر درباره وضعیت کلی کلاستر خود، می‌توانید اجرا کنید:

```shell
kubectl cluster-info dump
```

### مثال: اشکال‌زدایی یک نود خاموش/ناموجود

گاهی وقت‌ها در اشکال‌زدایی مفید است که وضعیت یک نود را بررسی کنید - به عنوان مثال، به دلیل رفتار عجیب یک Pod که روی نود اجرا می‌شود، یا برای پیدا کردن دلیلی که یک Pod روی نود برنامه‌ریزی نمی‌شود. مانند Pods، می‌توانید از `kubectl describe node` و `kubectl get node -o yaml` برای دریافت اطلاعات دقیق درباره نودها استفاده کنید. به عنوان مثال، اگر یک نود خاموش است (قطع از شبکه، یا kubelet متوقف شده و دیگر راه‌اندازی نمی‌شود، و غیره)، رویدادهای نشان می‌دهند که نود NotReady است و همچنین توجه داشته باشید که پس از پنج دقیقه وضعیت NotReady، Pods دیگری اجرا نمی‌شود.

```shell
kubectl get nodes
```

```none
NAME                     STATUS       ROLES     AGE     VERSION
kube-worker-1            NotReady     <none>    1h      v1.23.3
kubernetes-node-bols     Ready        <none>    1h      v1.23.3
kubernetes-node-st6x     Ready        <none>    1h      v1.23.3
kubernetes-node-unaj     Ready        <none>    1h      v1.23.3
```

```shell
kubectl describe node kube-worker-1
```

```none
Name:               kube-worker-1
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=kube-worker-1
                    kubernetes.io/os=linux
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Thu, 17 Feb 2022 16:46:30 -0500
Taints:             node.kubernetes.io/unreachable:NoExecute
                    node.kubernetes.io/unreachable:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  kube-worker-1
  AcquireTime:     <unset>
  RenewTime:       Thu, 17 Feb 2022 17:13:09 -0500
Conditions:
  Type                 Status    LastHeartbeatTime                 LastTransitionTime                Reason              Message
  ----                 ------    -----------------                 ------------------                ------              -------
  NetworkUnavailable   False     Thu, 17 Feb 2022 17:09:13 -0500   Thu, 17 Feb 2022 17:09:13 -0500   WeaveIsUp           Weave pod has set this
  MemoryPressure       Unknown   Thu, 17 Feb 2022 17:12:40 -0500   Thu, 17 Feb 2022 17:13:52 -0500   NodeStatusUnknown   Kubelet stopped posting node status.
  DiskPressure         Unknown   Thu, 17 Feb 2022 17:12:40 -0500   Thu, 17 Feb 2022 17:13:52 -0500   NodeStatusUnknown   Kubelet stopped posting node status.
  PIDPressure          Unknown   Thu, 17 Feb 2022 17:12:40 -0500   Thu, 17 Feb 2022 17:13:52 -0500   NodeStatusUnknown   Kubelet stopped posting node status.
  Ready                Unknown   Thu, 17 Feb 2022 17:12:40 -0500   Thu, 17 Feb 2022 17:13:52 -0500   NodeStatusUnknown   Kubelet stopped posting node status.
Addresses:
  InternalIP:  192.168.0.113
  Hostname:    kube-worker-1
Capacity:
  cpu:                2
  ephemeral-storage:  15372232Ki
  hugepages-2Mi:      0
  memory:             2025188Ki
  pods:               110
Allocatable:
  cpu:                2
  ephemeral-storage:  14167048988
  hugepages-2Mi:      0
  memory:             1922788Ki
  pods:               110
System Info:
  Machine ID:                 9384e2927f544209b5d7b67474bbf92b
  System UUID:                aa829ca9-73d7-064d-9019-df07404ad448
  Boot ID:                    5a295a03-aaca-4340-af20-1327fa5dab5c
  Kernel Version:             5.13.0-28-generic
  OS Image:                   Ubuntu 21.10
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.5.9
  Kubelet Version:            v1.23.3
  Kube-Proxy Version:         v1.23.3
Non-terminated Pods:          (4 in total)
  Namespace                   Name                                 CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                 ------------  ----------  ---------------  -------------  ---
  default                     nginx-deployment-67d4bdd6f5-cx2nz    500m (25%)    500m (25%)  128Mi (6%)       128Mi (6%)     23m
  default                     nginx-deployment-67d4bdd6f5-w6kd7    500m (25%)    500m (25%)  128Mi (6%)       128Mi (6%)

     23m
  kube-system                 kube-proxy-dnxbz                     0 (0%)        0 (0%)      0 (0%)           0 (0%)         28m
  kube-system                 weave-net-gjxxp                      100m (5%)     0 (0%)      200Mi (10%)      0 (0%)         28m
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests     Limits
  --------           --------     ------
  cpu                1100m (55%)  1 (50%)
  memory             456Mi (24%)  256Mi (13%)
  ephemeral-storage  0 (0%)       0 (0%)
  hugepages-2Mi      0 (0%)       0 (0%)
Events:
...
```
```markdown
---
title: "رفع مشکلات کلاسترها"
description: راهنمای رفع مشکلات متداول کلاسترها.
weight: 20
reviewers:
- davidopp
no_list: true
---

## مشاهده اطلاعات نود

برای دریافت اطلاعات کامل در مورد یک نود خاص، می‌توانید دستور زیر را اجرا کنید:

```shell
kubectl get node kube-worker-1 -o yaml
```

```yaml
apiVersion: v1
kind: Node
metadata:
  annotations:
    kubeadm.alpha.kubernetes.io/cri-socket: /run/containerd/containerd.sock
    node.alpha.kubernetes.io/ttl: "0"
    volumes.kubernetes.io/controller-managed-attach-detach: "true"
  creationTimestamp: "2022-02-17T21:46:30Z"
  labels:
    beta.kubernetes.io/arch: amd64
    beta.kubernetes.io/os: linux
    kubernetes.io/arch: amd64
    kubernetes.io/hostname: kube-worker-1
    kubernetes.io/os: linux
  name: kube-worker-1
  resourceVersion: "4026"
  uid: 98efe7cb-2978-4a0b-842a-1a7bf12c05f8
spec: {}
status:
  addresses:
  - address: 192.168.0.113
    type: InternalIP
  - address: kube-worker-1
    type: Hostname
  allocatable:
    cpu: "2"
    ephemeral-storage: "14167048988"
    hugepages-2Mi: "0"
    memory: 1922788Ki
    pods: "110"
  capacity:
    cpu: "2"
    ephemeral-storage: 15372232Ki
    hugepages-2Mi: "0"
    memory: 2025188Ki
    pods: "110"
  conditions:
  - lastHeartbeatTime: "2022-02-17T22:20:32Z"
    lastTransitionTime: "2022-02-17T22:20:32Z"
    message: Weave pod has set this
    reason: WeaveIsUp
    status: "False"
    type: NetworkUnavailable
  - lastHeartbeatTime: "2022-02-17T22:20:15Z"
    lastTransitionTime: "2022-02-17T22:13:25Z"
    message: kubelet has sufficient memory available
    reason: KubeletHasSufficientMemory
    status: "False"
    type: MemoryPressure
  - lastHeartbeatTime: "2022-02-17T22:20:15Z"
    lastTransitionTime: "2022-02-17T22:13:25Z"
    message: kubelet has no disk pressure
    reason: KubeletHasNoDiskPressure
    status: "False"
    type: DiskPressure
  - lastHeartbeatTime: "2022-02-17T22:20:15Z"
    lastTransitionTime: "2022-02-17T22:13:25Z"
    message: kubelet has sufficient PID available
    reason: KubeletHasSufficientPID
    status: "False"
    type: PIDPressure
  - lastHeartbeatTime: "2022-02-17T22:20:15Z"
    lastTransitionTime: "2022-02-17T22:15:15Z"
    message: kubelet is posting ready status
    reason: KubeletReady
    status: "True"
    type: Ready
  daemonEndpoints:
    kubeletEndpoint:
      Port: 10250
  nodeInfo:
    architecture: amd64
    bootID: 22333234-7a6b-44d4-9ce1-67e31dc7e369
    containerRuntimeVersion: containerd://1.5.9
    kernelVersion: 5.13.0-28-generic
    kubeProxyVersion: v1.23.3
    kubeletVersion: v1.23.3
    machineID: 9384e2927f544209b5d7b67474bbf92b
    operatingSystem: linux
    osImage: Ubuntu 21.10
    systemUUID: aa829ca9-73d7-064d-9019-df07404ad448
```

## مشاهده لاگ‌ها

برای مشاهده لاگ‌های کلاستر، به موارد زیر مراجعه کنید:

### نودهای کنترلی

- `/var/log/kube-apiserver.log`: لاگ API Server
- `/var/log/kube-scheduler.log`: لاگ Scheduler
- `/var/log/kube-controller-manager.log`: لاگ Controller Manager

### نودهای کارگر

- `/var/log/kubelet.log`: لاگ Kubelet
- `/var/log/kube-proxy.log`: لاگ kube-proxy

## حالت‌های شکست کلاستر

### عوامل مشارکت‌کننده

- خاموش شدن VM(ها)
- پارتیشن شبکه درون کلاستر، و یا بین کلاستر و کاربران
- کرش در نرم افزار کوبرنیتیس
- از دست رفتن یا در دسترس نبودن ذخیره سازی مداوم (مانند حجم GCE PD یا AWS EBS)
- خطای عامل، به عنوان مثال، نرم‌افزار کوبرنیتیس یا نرم‌افزار برنامه بدرستی پیکربندی نشده

### سناریوهای خاص

- خاموش شدن VM apiserver یا چکش
- نتیجه‌ها
  - ناتوانی در توقف، به‌روز رسانی یا شروع پاد، خدمات، کنترل کننده تکثیر
  - پادهای موجود و خدمات باید به طور عادی کار کنند مگر آنکه بسته به API کوبرنیتیس وابسته باشند
- از دست رفتن ذخیره سازی پشتیبان
  - نتیجه
    - مولفه kube-apiserver به درستی نمی‌تواند استارت زده و به حالت سالم برسد
    - kubelets نمی‌تواند به آن دسترسی داشته باشد اما همچنان پادها را اجرا کرده و خدمات یکسان را ارائه می‌دهد
    - بازیابی دستی یا بازسازی حالت apiserver قبل از استارت apiserver لازم است
- خدمات پشتیبانی‌کننده (کنترل کننده نود، کنترل کننده تکثیر مدیر، برنامه‌ریز) VM خاموش یا انفجار
  - در حال حاضر این‌ها با apiserver مشارکت می‌کنند و ناموجودی آن‌ها مشکلات مشابهی را به عنوان apiserver می‌آورد
  - در آینده، این‌ها نیز با هم تکثیر می‌شوند و ممکن است مکان‌های زندگی نداشته باشند
  - آن‌ها حالت دائمی خود را ندارند
- متوقف کردن نود فردی (VM یا دستگاه فیزیکی)
  - نتایج
    - پادها در آن نود متوقف می‌شوند
- بخشی کردن شبکه
  - نتایج
    - بخش A فکر می‌کند که

 نودهای بخش B خاموش هستند؛ بخش B فکر می‌کند که apiserver خاموش است. (اگر VM اصلی در بخش A قرار داشته باشد.)
- خطای نرم افزار Kubelet
  - نتایج
    - kubelet از کار افتاده نمی‌تواند پادهای جدید را در نود شروع کند
    - kubelet ممکن است پادها را حذف کند یا نکند
    - نود به عنوان ناسالم مشخص می‌شود
    - کنترل کننده تکثیر در مکان‌های دیگر پادها را شروع می‌کند
- خطای عامل کلاستر
  - نتایج
    - از دست رفتن پادها، خدمات، و غیره
    - از دست رفتن ذخیره سازی پشتیبان apiserver
    - کاربران نمی‌توانند API را بخوانند
    - غیره

### ضد اندازه‌گیری

- عمل: استفاده از ویژگی بازنشانی خودکار VM ارائه‌دهنده IaaS برای VM‌های IaaS
  - کاهش می‌دهد: خاموش شدن VM apiserver یا چکش
  - کاهش می‌دهد: خاموش شدن خدمات پشتیبانی‌کننده یا خراب‌شدن آنها

- عمل: استفاده از ذخیره‌سازی قابل اعتماد ارائه‌دهنده‌های IaaS (مانند حجم GCE PD یا AWS EBS) برای VM‌ها با apiserver + etcd
  - کاهش می‌دهد: از دست رفتن ذخیره‌سازی پشتیبان

- عمل: استفاده از پیکربندی [پایداری بالا](/docs/setup/production-environment/tools/kubeadm/high-availability/)
  - کاهش می‌دهد: خاموش شدن نود کنترلی یا خراب شدن کامپوننت‌های کنترل کننده (برنامه‌ریز، apiserver، مدیر کنترل کننده)
    - مقاومت در برابر یک یا چند شکست هم‌زمان نود یا کامپوننت

  - کاهش می‌دهد: ذخیره سازی پشتیبان API (به عنوان مثال، دایرکتوری داده etcd) از دست رفته
    - فرض می‌شود پیکربندی HA (بالایی) etcd

- عمل: گرفتن عکس از PDs / EBS-volumes apiserver به طور دوره‌ای
  - کاهش می‌دهد: از دست رفتن ذخیره‌سازی پشتیبان
  - کاهش می‌دهد: برخی از موارد خطای عامل
  - کاهش می‌دهد: برخی از خطاهای نرم افزار کوبرنیتیس

- عمل: استفاده از کنترل کننده تکثیر و خدمات در جلوی پادها
  - کاهش می‌دهد: خاموش شدن نود
  - کاهش می‌دهد: خطای نرم افزار kubelet

- عمل: برنامه‌ها (کانتینرها) طراحی شده برای تحمل بارهای غیرمنتظره
  - کاهش می‌دهد: خاموش شدن نود
  - کاهش می‌دهد: خطای نرم افزار kubelet

## {{% heading "whatsnext" %}}

* آشنایی با معیارهای موجود در [Resource Metrics Pipeline](/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/)
* کشف ابزارهای اضافی برای [مانیتورینگ مصرف منابع](/docs/tasks/debug/debug-cluster/resource-usage-monitoring/)
* استفاده از Node Problem Detector برای [مانیتورینگ سلامت نود](/docs/tasks/debug/debug-cluster/monitor-node-health/)
* استفاده از `kubectl debug node` برای [دیباگ نودهای کوبرنیتیس](/docs/tasks/debug/debug-cluster/kubectl-node-debug)
* استفاده از `crictl` برای [دیباگ نودهای کوبرنیتیس](/docs/tasks/debug/debug-cluster/crictl/)
* به دست آوردن اطلاعات بیشتر در مورد [بایگانی کوبرنیتیس](/docs/tasks/debug/debug-cluster/audit/)
* استفاده از `telepresence` برای [توسعه و دیباگ خدمات به صورت محلی](/docs/tasks/debug/debug-cluster/local-debugging/)
