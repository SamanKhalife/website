---
reviewers:
- dcbw
- freehan
- thockin
title: پلاگین‌های شبکه
content_type: concept
weight: 10
---

<!-- overview -->

Kubernetes {{< skew currentVersion >}} پشتیبانی می‌کند [از رابط شبکه ظرف‌های کانتینر](https://github.com/containernetworking/cni)
(CNI) برای شبکه کلاستر. شما باید از یک پلاگین CNI استفاده کنید که با کلاستر شما سازگار و نیاز‌های شما را برطرف کند. پلاگین‌های مختلف در اکوسیستم گسترده Kubernetes (هم متن‌باز و بسته) در دسترس هستند.

برای پیاده‌سازی [مدل شبکه Kubernetes](/docs/concepts/services-networking/#the-kubernetes-network-model)، نیاز به یک پلاگین CNI دارید.

شما باید از یک پلاگین CNI استفاده کنید که با ورژن [v0.4.0](https://github.com/containernetworking/cni/blob/spec-v0.4.0/SPEC.md) یا بعد از آن از مشخصات CNI سازگار باشد. پروژه Kubernetes توصیه می‌کند که از پلاگینی استفاده شود که با ورژن [v1.0.0](https://github.com/containernetworking/cni/blob/spec-v1.0.0/SPEC.md) از مشخصات CNI نیز سازگار باشد (پلاگین‌ها ممکن است با چندین ورژن مشخصات سازگار باشند).

<!-- body -->

## نصب

در متن شبکه، یک ظرف اجرا، دیمونی در یک گره است که برای ارائه خدمات CRI به kubelet پیکربندی شده است. به ویژه، باید دیمون ظرف اجرا برای بارگذاری پلاگین‌های CNI مورد نیاز برای پیاده‌سازی مدل شبکه Kubernetes پیکربندی شود.

{{< note >}}
پیش از Kubernetes 1.24، پلاگین‌های CNI نیز می‌توانستند توسط kubelet با استفاده از پارامترهای خط فرمان `cni-bin-dir` و `network-plugin` مدیریت شوند. این پارامترهای خط فرمان در Kubernetes 1.24 حذف شدند و مدیریت CNI دیگر در محدوده kubelet نیست.

برای مشاهده [رفع مشکلات مربوط به پلاگین CNI](/docs/tasks/administer-cluster/migrating-from-dockershim/troubleshooting-cni-plugin-related-errors/)
اگر با حذف dockershim مواجه هستید.
{{< /note >}}

برای اطلاعات خاص درباره چگونگی یک ظرف اجرای شبکه پلاگین‌های CNI را مدیریت می‌کند، به مستندات آن ظرف اجرا مراجعه کنید، به عنوان مثال:

- [containerd](https://github.com/containerd/containerd/blob/main/script/setup/install-cni)
- [CRI-O](https://github.com/cri-o/cri-o/blob/main/contrib/cni/README.md)

برای اطلاعات خاص درباره نصب و مدیریت یک پلاگین CNI، به مستندات آن پلاگین یا [ارائه‌دهنده شبکه](/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-network-model) مراجعه کنید.

## الزامات پلاگین شبکه

### پلاگین CNI Loopback

به علاوه، به جز پلاگین CNI نصب شده در گره‌ها برای پیاده‌سازی مدل شبکه Kubernetes، Kubernetes نیازمند ارائه رابط loopback `lo` در هر sandbox (sandbox‌های پاد، sandbox‌های vm، ...) است. پیاده‌سازی رابط loopback می‌تواند با استفاده از [پلاگین loopback CNI](https://github.com/containernetworking/plugins/blob/master/plugins/main/loopback/loopback.go)
یا با توسعه کد خود انجام شود (برای مثال، [این مثال از CRI-O](https://github.com/cri-o/ocicni/blob/release-1.24/pkg/ocicni/util_linux.go#L91)).

### پشتیبانی از hostPort

پلاگین شبکه CNI از `hostPort` پشتیبانی می‌کند. می‌توانید از پلاگین رسمی [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap)
از تیم پلاگین CNI استفاده کنید یا از پلاگین خود با قابلیت portMapping استفاده کنید.

اگر می‌خواهید پشتیبانی از `hostPort` را فعال کنید، باید قابلیت `portMappings capability` را در `cni-conf-dir` خود مشخص کنید. به عنوان مثال:

```json
{
  "name": "k8s-pod-network",
  "cniVersion": "0.4.0",
  "plugins": [
    {
      "type": "calico",
      "log_level": "info",
      "datastore_type": "kubernetes",
      "nodename": "127.0.0.1",
      "ipam": {
        "type": "host-local",
        "subnet": "usePodCidr"
      },
      "policy": {
        "type": "k8s"
      },
      "kubernetes": {
        "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
      }
    },
    {
      "type": "portmap",
      "capabilities": {"portMappings": true},
      "externalSetMarkChain": "KUBE-MARK-MASQ"
    }
  ]
}
```

### پشتیبانی از تنظیم ترافیک

**ویژگی آزمایشی**

پلاگین شبکه CNI همچنین از تنظیم ترافیک ورودی و خروجی pod پشتیبانی می‌کند. می‌توانید از پلاگین رسمی [bandwidth](https://github.com/containernetworking/plugins/tree/master/plugins/meta/bandwidth)
از تیم پلاگین CNI استفاده کنید یا از پلاگین خود با قابلیت کنترل bandwidth استفاده کنید.

اگر می‌خواهید پشتیبانی از تنظیم ترافیک را فعال کنید، باید پلاگین `bandwidth` را به فایل پیکربندی CNI خود (پیش‌فرض `/etc/cni/net.d`) اضافه کنید و اطمینان حاصل کنید که باینری آن در CNI bin dir (پیش‌ف

رض `/opt/cni/bin`) درج شده است.

```json
{
  "name": "k8s-pod-network",
  "cniVersion": "0.4.0",
  "plugins": [
    {
      "type": "calico",
      "log_level": "info",
      "datastore_type": "kubernetes",
      "nodename": "127.0.0.1",
      "ipam": {
        "type": "host-local",
        "subnet": "usePodCidr"
      },
      "policy": {
        "type": "k8s"
      },
      "kubernetes": {
        "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
      }
    },
    {
      "type": "bandwidth",
      "capabilities": {"bandwidth": true}
    }
  ]
}
```

اکنون می‌توانید یادداشت‌های `kubernetes.io/ingress-bandwidth` و `kubernetes.io/egress-bandwidth` را به Pod خود اضافه کنید. به عنوان مثال:

```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubernetes.io/ingress-bandwidth: 1M
    kubernetes.io/egress-bandwidth: 1M
...
```

## {{% heading "whatsnext" %}}

- بیشتر درباره [شبکه کلاستر](/docs/concepts/cluster-administration/networking/) بیاموزید
- بیشتر درباره [سیاست‌های شبکه](/docs/concepts/services-networking/network-policies/) بیاموزید
- درباره [رفع مشکلات مربوط به پلاگین CNI](/docs/tasks/administer-cluster/migrating-from-dockershim/troubleshooting-cni-plugin-related-errors/) بیشتر بیاموزید
