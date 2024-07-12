---
reviewers:
- sig-cluster-lifecycle
title: سفارشی‌سازی اجزا با استفاده از API kubeadm
content_type: concept
weight: 40
---

<!-- overview -->

این صفحه به بررسی نحوه سفارشی‌سازی اجزایی که kubeadm نصب می‌کند می‌پردازد. برای اجزای کنترل پلن می‌توانید از پرچم‌ها در ساختار `ClusterConfiguration` یا پچ‌های مربوط به هر نود استفاده کنید. برای kubelet و kube-proxy نیز می‌توانید به ترتیب از `KubeletConfiguration` و `KubeProxyConfiguration` استفاده نمایید.

همه این گزینه‌ها از طریق API پیکربندی kubeadm امکان‌پذیر است.
برای جزئیات بیشتر در مورد هر فیلد در پیکربندی، به صفحات [مرجع API](/docs/reference/config-api/kubeadm-config.v1beta3/) مراجعه نمایید.

{{< note >}}
در حال حاضر، پشتیبانی از سفارشی‌سازی استقرار CoreDNS توسط kubeadm وجود ندارد. شما باید به صورت دستی `kube-system/coredns` را پچ کرده و پس از آن پادهای CoreDNS را بازسازی کنید. به علاوه، می‌توانید استقرار پیش‌فرض CoreDNS را از رده خود بگذرانید و نسخه‌ای سفارشی از آن را ارائه دهید. برای جزئیات بیشتر به [استفاده از مراحل شروع با kubeadm](/docs/reference/setup-tools/kubeadm/kubeadm-init/#init-phases) مراجعه کنید.
{{< /note >}}

{{< note >}}
برای بازپیکربندی یک خوشه که قبلاً ایجاد شده است، به [بازپیکربندی خوشه kubeadm](/docs/tasks/administer-cluster/kubeadm/kubeadm-reconfigure) مراجعه کنید.
{{< /note >}}

<!-- body -->

## سفارشی‌سازی پلن کنترل با استفاده از پرچم‌ها در `ClusterConfiguration`

اشیاء `ClusterConfiguration` در kubeadm یک راه برای کاربران فراهم می‌آورد تا پرچم‌های پیش‌فرضی که به اجزای پلن کنترل مانند APIServer، ControllerManager، Scheduler و Etcd منتقل می‌شوند را نادیده بگیرند. اجزا به کمک ساختارهای زیر تعریف می‌شوند:

- `apiServer`
- `controllerManager`
- `scheduler`
- `etcd`

این ساختارها شامل یک فیلد مشترک `extraArgs` هستند که شامل جفت `key: value` می‌باشد. برای نادیده گرفتن یک پرچم برای یک اجزای کنترل پلن:

1.  پرچم‌های مناسب را به پیکربندی خود اضافه کنید.
2.  پرچم‌ها را به فیلد `extraArgs` اضافه کنید.
3.  دستور `kubeadm init` را با `--config <YOUR CONFIG YAML>` اجرا کنید.

{{< note >}}
می‌توانید یک شیء `ClusterConfiguration` با مقادیر پیش‌فرض را با دستور `kubeadm config print init-defaults` تولید کنید و خروجی آن را در یک فایل به انتخاب خود ذخیره کنید.
{{< /note >}}

{{< note >}}
شیء `ClusterConfiguration` در حال حاضر به صورت جهانی در خوشه‌های kubeadm است. این بدان معنی است که هر پرچمی که شما اضافه می‌کنید، برای تمام نمونه‌های همان اجزا در نودهای مختلف اعمال می‌شود. برای اعمال پیکربندی فردی برای هر اجزا بر روی نودهای مختلف، می‌توانید از [پچ‌ها](#patches) استفاده کنید.
{{< /note >}}

{{< note >}}
پرچم‌های تکراری (کلیدها)، یا انتقال چند باره همان پرچم `--foo` در حال حاضر پشتیبانی نمی‌شود. برای اینکار باید از [پچ‌ها](#patches) استفاده نمایید.
{{< /note >}}

### پرچم‌های APIServer

برای جزئیات بیشتر به [مستندات مرجع برای kube-apiserver](/docs/reference/command-line-tools-reference/kube-apiserver/) مراجعه نمایید.

مثال استفاده:

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.16.0
apiServer:
  extraArgs:
    anonymous-auth: "false"
    enable-admission-plugins: AlwaysPullImages,DefaultStorageClass
    audit-log-path: /home/johndoe/audit.log
```

### پرچم‌های ControllerManager

برای جزئیات بیشتر به [مستندات مرجع برای kube-controller-manager](/docs/reference/command-line-tools-reference/kube-controller-manager/) مراجعه نمایید.

مثال استفاده:

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.16.0
controllerManager:
  extraArgs:
    cluster-signing-key-file: /home/johndoe/keys/ca.key
    deployment-controller-sync-period: "50"
```


### پرچم‌های Scheduler

برای جزئیات بیشتر به [مستندات مرجع برای kube-scheduler](/docs/reference/command-line-tools-reference/kube-scheduler/) مراجعه نمایید.

مثال استفاده:

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.16.0
scheduler:
  extraArgs:
    config: /etc/kubernetes/scheduler-config.yaml
  extraVolumes:
    - name: schedulerconfig
      hostPath: /home/johndoe/schedconfig.yaml
      mountPath: /etc/kubernetes/scheduler-config.yaml
      readOnly: true
      pathType: "File"
```

### پرچم‌های Etcd

برای جزئیات بیشتر به [مستندات سرور etcd](https://etcd.io/docs/) مراجعه نمایید.

مثال استفاده:

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
etcd:
  local:
    extraArgs:
      election-timeout: 1000
```

## سفارشی‌سازی با استفاده از پچ‌ها {#patches}

{{< feature-state for_k8s_version="v1.22" state="beta" >}}

Kubeadm به شما امکان می‌دهد که یک دایرکتوری حاوی فایل‌های پچ را به `InitConfiguration` و `JoinConfiguration`
در نودهای فردی منتقل کنید. این پچ‌ها می‌توانند به عنوان آخرین مرحله سفارشی‌سازی قبل از تنظیمات اجزا بر روی دیسک استفاده شوند.

می‌توانید این فایل را با `kubeadm init` با `--config <YOUR CONFIG YAML>` منتقل کنید:

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
patches:
  directory: /home/user/somedir
```

{{< note >}}
برای `kubeadm init` می‌توانید یک فایل شامل `ClusterConfiguration` و `InitConfiguration`
که توسط `---` جدا شده‌اند، منتقل کنید.
{{< /note >}}

می‌توانید این فایل را با `kubeadm join` با `--config <YOUR CONFIG YAML>` منتقل کنید:

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: JoinConfiguration
patches:
  directory: /home/user/somedir
```

دایرکتوری باید فایل‌هایی با نام‌های `target[suffix][+patchtype].extension` را شامل شود.
برای مثال، `kube-apiserver0+merge.yaml` یا فقط `etcd.json`.

- `target` می‌تواند یکی از `kube-apiserver`، `kube-controller-manager`، `kube-scheduler`، `etcd`
و `kubeletconfiguration` باشد.
- `suffix` یک رشته اختیاری است که می‌تواند برای تعیین این که کدام پچ‌ها به صورت الفبایی-عددی اعمال شوند، استفاده شود.
- `patchtype` می‌تواند یکی از `strategic`، `merge` یا `json` باشد و باید با فرمت‌های پچینگ
[پشتیبانی شده توسط kubectl](/docs/tasks/manage-kubernetes-objects/update-api-object-kubectl-patch) مطابقت داشته باشد.
پیش‌فرض `patchtype` `strategic` است.
- `extension` باید یکی از `json` یا `yaml` باشد.

{{< note >}}
اگر از `kubeadm upgrade` برای ارتقای نودهای kubeadm خود استفاده می‌کنید، باید دوباره همان
پچ‌ها را فراهم کنید تا سفارشی‌سازی پس از ارتقا حفظ شود. برای انجام این کار می‌توانید از پرچم `--patches`
استفاده کنید که باید به همان دایرکتوری اشاره کند. `kubeadm upgrade` در حال حاضر از یک ساختار API پیکربندی پشتیبانی نمی‌کند که برای همین منظور استفاده شود.
{{< /note >}}

## سفارشی‌سازی kubelet {#kubelet}

برای سفارشی‌سازی kubelet می‌توانید یک [`KubeletConfiguration`](/docs/reference/config-api/kubelet-config.v1beta1/)
را کنار `ClusterConfiguration` یا `InitConfiguration` که با `---` جدا شده‌اند، به فایل پیکربندی اضافه کنید.
این فایل سپس می‌تواند به `kubeadm init` منتقل شود و kubeadm همان پایه `KubeletConfiguration` را برای تمام نودها در خوشه اعمال می‌کند.

برای اعمال پیکربندی خاص نمونه به برگه `KubeletConfiguration` پایه، می‌توانید از
[`kubeletconfiguration` patch target](#patches) استفاده کنید.

به طور جایگزین، می‌توانید از پرچم‌های kubelet به عنوان نادیده گرفتن با گذراندن آنها در
`nodeRegistration.kubeletExtraArgs` که توسط `InitConfiguration` و `JoinConfiguration` پشتیبانی می‌شود، استفاده کنید.
بعضی از پرچم‌های kubelet قدیمی شده‌اند، بنابراین قبل از استفاده از آنها وضعیت آنها را در
[مستندات مرجع kubelet](/docs/reference/command-line-tools-reference/kubelet) بررسی کنید.

برای جزئیات بیشتر به [پیکربندی هر kubelet در خوشه خود با استفاده از kubeadm](/docs/setup/production-environment/tools/kubeadm/kubelet-integration) مراجعه نمایید.

## سفارشی‌سازی kube-proxy

برای سفارشی‌سازی kube-proxy می‌توانید یک `KubeProxyConfiguration` را به `ClusterConfiguration` یا
`InitConfiguration` خود اضافه کنید و با `---` جدا شده به `kubeadm init` منتقل کنید.

برای جزئیات بیشتر می‌توانید به صفحات [مرجع API ما](/docs/reference/config-api/kubeadm-config.v1beta3/) مراجعه نمایید.

{{< note >}}
kubeadm kube-proxy را به عنوان یک {{< glossary_tooltip text="DaemonSet" term_id="daemonset" >}} استقرار می‌دهد، که به این معنی است که `KubeProxyConfiguration` به همه نمونه‌های kube-proxy در خ

وشه اعمال خواهد شد.
{{< /note >}}
