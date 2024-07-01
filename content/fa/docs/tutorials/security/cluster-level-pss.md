---
title: اعمال استانداردهای امنیتی Pod در سطح خوشه
content_type: آموزشی
weight: 10
---

{{% alert title="توجه" %}}
این آموزش فقط برای خوشه‌های جدید اعمال می‌شود.
{{% /alert %}}

کنترل کننده ورودی Pod Security چک‌هایی را در برابر [استانداردهای امنیتی Pod Kubernetes](/docs/concepts/security/pod-security-standards/) انجام می‌دهد هنگامی که پادهای جدید ایجاد می‌شوند. این ویژگی از نسخه v1.25 به بعد GA شده است.
این آموزش به شما نشان می‌دهد که چگونه استاندارد `baseline` امنیتی Pod را در سطح خوشه اعمال کنید که یک تنظیم استاندارد را به تمام فضاهای نام در یک خوشه اعمال می‌کند.

برای اعمال استانداردهای امنیتی Pod به فضاهای نام خاص، به [اعمال استانداردهای امنیتی Pod در سطح فضای نام](/docs/tutorials/security/ns-level-pss) مراجعه کنید.

اگر از نسخه Kubernetes دیگری به جز v{{< skew currentVersion >}} استفاده می‌کنید، مستندات مربوط به آن نسخه را بررسی کنید.

## {{% heading "پیش‌نیازها" %}}

نصب موارد زیر را در کامپیوتر خود انجام دهید:

- [kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- [kubectl](/docs/tasks/tools/)

این آموزش نشان می‌دهد که چگونه می‌توانید یک خوشه Kubernetes را که کاملاً کنترل می‌کنید، پیکربندی کنید. اگر در حال یادگیری استانداردهای ورودی Pod Security برای یک خوشه مدیریت شده هستید که نمی‌توانید صفحه کنترل را پیکربندی کنید، [اعمال استانداردهای امنیتی Pod در سطح فضای نام](/docs/tutorials/security/ns-level-pss) را بخوانید.

## انتخاب استاندارد مناسب امنیتی Pod

ورودی Pod Security به شما اجازه می‌دهد تا با حالت‌های `enforce`، `audit` و `warn` استانداردهای امنیتی Pod داخلی را اعمال کنید.

برای جمع‌آوری اطلاعات که به شما کمک می‌کند تا استانداردهای امنیتی Pod مناسب برای پیکربندی خود را انتخاب کنید، اقدامات زیر را انجام دهید:

1. یک خوشه بدون استانداردهای امنیتی Pod ایجاد کنید:

   ```shell
   kind create cluster --name psa-wo-cluster-pss
   ```

   خروجی مشابه زیر است:
   ```
   Creating cluster "psa-wo-cluster-pss" ...
   ✓ Ensuring node image (kindest/node:v{{< skew currentPatchVersion >}}) 🖼
   ✓ Preparing nodes 📦
   ✓ Writing configuration 📜
   ✓ Starting control-plane 🕹️
   ✓ Installing CNI 🔌
   ✓ Installing StorageClass 💾
   Set kubectl context to "kind-psa-wo-cluster-pss"
   You can now use your cluster with:

   kubectl cluster-info --context kind-psa-wo-cluster-pss

   Thanks for using kind! 😊
   ```

1. کانتکست kubectl را به خوشه جدید تنظیم کنید:

   ```shell
   kubectl cluster-info --context kind-psa-wo-cluster-pss
   ```

   خروجی مشابه زیر است:
   ```
   Kubernetes control plane is running at https://127.0.0.1:61350

   CoreDNS is running at https://127.0.0.1:61350/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

   To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
   ```

1. لیست فضاهای نام در خوشه را دریافت کنید:

   ```shell
   kubectl get ns
   ```

   خروجی مشابه زیر است:
   ```
   NAME                 STATUS   AGE
   default              Active   9m30s
   kube-node-lease      Active   9m32s
   kube-public          Active   9m32s
   kube-system          Active   9m32s
   local-path-storage   Active   9m26s
   ```

1. از `--dry-run=server` برای درک اینکه وقتی استانداردهای امنیتی Pod مختلف اعمال می‌شود، چه اتفاقی می‌افتد استفاده کنید:

   1. Privileged
      ```shell
      kubectl label --dry-run=server --overwrite ns --all \
      pod-security.kubernetes.io/enforce=privileged
      ```

      خروجی مشابه زیر است:
      ```
      namespace/default labeled
      namespace/kube-node-lease labeled
      namespace/kube-public labeled
      namespace/kube-system labeled
      namespace/local-path-storage labeled
      ```
   2. Baseline
      ```shell
      kubectl label --dry-run=server --overwrite ns --all \
      pod-security.kubernetes.io/enforce=baseline
      ```

      خروجی مشابه زیر است:
      ```
      namespace/default labeled
      namespace/kube-node-lease labeled
      namespace/kube-public labeled
      Warning: existing pods in namespace "kube-system" violate the new PodSecurity enforce level "baseline:latest"
      Warning: etcd-psa-wo-cluster-pss-control-plane (and 3 other pods): host namespaces, hostPath volumes
      Warning: kindnet-vzj42: non-default capabilities, host namespaces, hostPath volumes
      Warning: kube-proxy-m6hwf: host namespaces, hostPath volumes, privileged
      namespace/kube-system labeled
      namespace/local-path-storage labeled
      ```

   3. Restricted
      ```shell
      kubectl label --dry-run=server --overwrite ns --all \
      pod-security.kubernetes.io/enforce=restricted
      ```

      خروجی مشابه زیر است:
      ```
      namespace/default labeled
      namespace/kube-node-lease labeled
      namespace/kube-public labeled
      Warning: existing pods in namespace "kube-system" violate the new PodSecurity enforce level "restricted:latest"
      Warning: coredns-7bb9c7b568-hsptc (and 1 other pod): unrestricted capabilities, runAsNonRoot != true, seccompProfile
      Warning: etcd-psa-wo-cluster-pss-control-plane (and 3 other pods): host namespaces, hostPath volumes, allowPrivilegeEscalation != false, unrestricted capabilities, restricted volume types, runAsNonRoot != true
      Warning: kindnet-vzj42: non-default capabilities, host namespaces, hostPath volumes, allowPrivilegeEscalation != false, unrestricted capabilities, restricted volume types, runAsNonRoot != true, seccompProfile
      Warning: kube-proxy-m6hwf: host namespaces, hostPath volumes, privileged, allowPrivilegeEscalation != false, unrestricted capabilities, restricted volume types, runAsNonRoot != true, seccompProfile
     

 namespace/kube-system labeled
      Warning: existing pods in namespace "local-path-storage" violate the new PodSecurity enforce level "restricted:latest"
      Warning: local-path-provisioner-d6d9f7ffc-lw9lh: allowPrivilegeEscalation != false, unrestricted capabilities, runAsNonRoot != true, seccompProfile
      namespace/local-path-storage labeled
      ```

از خروجی‌های قبلی متوجه می‌شوید که اعمال استاندارد امنیتی `privileged` هیچ هشداری برای هیچ یک از فضاهای نام ندارد. با این حال، استانداردهای `baseline` و `restricted` هر دو هشدارهایی دارند، به خصوص در فضای نام `kube-system`.

## تنظیم حالت‌ها، نسخه‌ها و استانداردها امنیتی Pod

در این بخش، شما استانداردهای امنیتی Pod زیر را با نسخه `latest` اعمال می‌کنید:

* استاندارد `baseline` با حالت `enforce`.
* استاندارد `restricted` با حالت‌های `warn` و `audit`.

استاندارد امنیتی Pod `baseline` یک میانه مناسب است که به شما امکان می‌دهد لیست مستثنیات را کوتاه نگه دارید و جلوگیری از افزایش دسترسی‌های خودکار شناخته شده را ممکن می‌سازد.

به علاوه، برای جلوگیری از شکست پادها در `kube-system`، شما فضای نام را از اعمال استانداردهای امنیتی Pod مستثنی می‌کنید.

هنگامی که شما اقدام به پیاده‌سازی ورودی Pod Security در محیط خود می‌کنید، در نظر داشته باشید:

1. براساس موقعیت خطری که برای یک خوشه اعمال می‌شود، استاندارد امنیتی Pod سخت‌گیرانه مانند `restricted` ممکن است گزینه بهتری باشد.
1. مستثنی کردن فضای نام `kube-system` به پادهای اجازه می‌دهد که به عنوان `privileged` در این فضای نام اجرا شوند. برای استفاده واقعی، پروژه Kubernetes به شدت توصیه می‌کند که شما سیاست‌های RBAC سخت‌گیرانه را که دسترسی به `kube-system` را محدود می‌کند، اعمال کنید.

برای پیاده‌سازی استانداردهای گذشته، اقدامات زیر را انجام دهید:

1. یک فایل پیکربندی ایجاد کنید که توسط کنترل کننده ورودی Pod Security مصرف شود تا این استانداردهای امنیتی Pod را پیاده‌سازی کند:

   ```
   mkdir -p /tmp/pss
   cat <<EOF > /tmp/pss/cluster-level-pss.yaml
   apiVersion: apiserver.config.k8s.io/v1
   kind: AdmissionConfiguration
   plugins:
   - name: PodSecurity
     configuration:
       apiVersion: pod-security.admission.config.k8s.io/v1
       kind: PodSecurityConfiguration
       defaults:
         enforce: "baseline"
         enforce-version: "latest"
         audit: "restricted"
         audit-version: "latest"
         warn: "restricted"
         warn-version: "latest"
       exemptions:
         usernames: []
         runtimeClasses: []
         namespaces: [kube-system]
   EOF
   ```

   {{< note >}}
   پیکربندی `pod-security.admission.config.k8s.io/v1` نیازمند v1.25+ است.
   برای v1.23 و v1.24، از [v1beta1](https://v1-24.docs.kubernetes.io/docs/tasks/configure-pod-container/enforce-standards-admission-controller/) استفاده کنید.
   برای v1.22، از [v1alpha1](https://v1-22.docs.kubernetes.io/docs/tasks/configure-pod-container/enforce-standards-admission-controller/) استفاده کنید.
   {{< /note >}}

1. API سرور را پیکربندی کنید تا این فایل را هنگام ایجاد خوشه مصرف کند:

   ```
   cat <<EOF > /tmp/pss/cluster-config.yaml
   kind: Cluster
   apiVersion: kind.x-k8s.io/v1alpha4
   nodes:
   - role: control-plane
     kubeadmConfigPatches:
     - |
       kind: ClusterConfiguration
       apiServer:
           extraArgs:
             admission-control-config-file: /etc/config/cluster-level-pss.yaml
           extraVolumes:
             - name: accf
               hostPath: /etc/config
               mountPath: /etc/config
               readOnly: false
               pathType: "DirectoryOrCreate"
     extraMounts:
     - hostPath: /tmp/pss
       containerPath: /etc/config
       # اختیاری: اگر تنظیم شود، مانت فقط خواندنی است.
       # پیش فرض نادرست است
       readOnly: false
       # اختیاری: اگر تنظیم شود، مانت نیاز به برچسب آزادی SELinux است.
       # پیش فرض نادرست است
       selinuxRelabel: false
       # اختیاری: حالت انتقال را تنظیم می‌کند (هیچ، HostToContainer یا Bidirectional)
       # مشاهده https://kubernetes.io/docs/concepts/storage/volumes/#mount-propagation
       # پیش فرض نادرست است
       propagation: None
   EOF
   ```

   {{<note>}}
   اگر از Docker Desktop با *kind* در macOS استفاده می‌کنید، می‌توانید `/tmp` را به عنوان یک دایرکتوری مشترک در زیر منو **Preferences > Resources > File Sharing** اضافه کنید.
   {{</note>}}

1. خوشه‌ای ایجاد کنید که از ورودی Pod Security برای اعمال این استانداردهای امنیتی Pod استفاده می‌کند:

   ```shell
   kind create cluster --name psa-with-cluster-pss --config /tmp/pss/cluster-config.yaml
   ```

   خروجی مشابه زیر است:
   ```
   Creating cluster "psa-with-cluster-pss" ...
    ✓ Ensuring node image (kindest/node:v{{< skew currentPatchVersion >}}) 🖼
    ✓ Preparing nodes 📦
    ✓ Writing configuration 📜
    ✓ Starting control-plane 🕹️
    ✓ Installing CNI 🔌
    ✓ Installing StorageClass 💾
   Set kubectl context to "kind-psa-with-cluster-pss"
   You can now use your cluster with:

   kubectl cluster-info --context kind-psa-with-cluster-pss

   Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
   ```

1. kubectl را به خوشه متصل کنید:
   ```shell
   kubectl cluster-info --context kind-psa-with-cluster-pss
   ```

   خروجی مشابه زیر

 است:
   ```
   Kubernetes control plane is running at https://127.0.0.1:63855
   CoreDNS is running at https://127.0.0.1:63855/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

   To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
   ```

1. یک Pod در فضای نام پیش فرض ایجاد کنید:

    {{% code_sample file="security/example-baseline-pod.yaml" %}}

   ```shell
   kubectl apply -f https://k8s.io/examples/security/example-baseline-pod.yaml
   ```

   پاد به طور معمول شروع می‌شود، اما خروجی شامل یک هشدار است:
   ```
   Warning: would violate PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "nginx" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "nginx" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "nginx" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "nginx" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
   pod/nginx created
   ```

## پاک کردن

حالا خوشه‌هایی را که ایجاد کرده‌اید را با اجرای دستور زیر پاک کنید:

```shell
kind delete cluster --name psa-with-cluster-pss
```
```shell
kind delete cluster --name psa-wo-cluster-pss
```

## {{% heading "whatsnext" %}}

- اجرای
  [shell script](/examples/security/kind-with-cluster-level-baseline-pod-security.sh)
  برای انجام همه مراحل گذشته به یکباره:
  1. ایجاد یک پیکربندی خوشه بر اساس استانداردهای امنیتی Pod
  2. ایجاد یک فایل برای اجازه‌دهی API سرور به این پیکربندی
  3. ایجاد یک خوشه که API سرور با این پیکربندی ایجاد می‌شود
  4. تنظیم کانتکست kubectl به این خوشه جدید
  5. ایجاد یک فایل yaml pod حداقلی
  6. اعمال این فایل برای ایجاد یک Pod در خوشه جدید
- [ورودی Pod Security Admission](/docs/concepts/security/pod-security-admission/)
- [استانداردهای امنیتی Pod](/docs/concepts/security/pod-security-standards/)
- [اعمال استانداردهای امنیتی Pod در سطح فضای نام](/docs/tutorials/security/ns-level-pss/)

