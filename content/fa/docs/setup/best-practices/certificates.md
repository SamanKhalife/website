---
title: PKI certificates and requirements
reviewers:
- sig-cluster-lifecycle
content_type: concept
weight: 50
---

<!-- overview -->

برای احراز هویت از طریق TLS، Kubernetes نیازمند گواهی‌نامه‌های PKI می‌باشد. اگر شما Kubernetes را با استفاده از [kubeadm](/docs/reference/setup-tools/kubeadm/) نصب می‌کنید، گواهی‌نامه‌هایی که خوشه شما نیاز دارد به طور خودکار تولید می‌شوند. همچنین می‌توانید گواهی‌نامه‌های خود را ایجاد کنید، به عنوان مثال برای افزایش امنیت کلیدهای خصوصی خود با ذخیره آن‌ها در سرور API.

<!-- body -->

## نحوه استفاده از گواهی‌نامه‌ها توسط خوشه شما

Kubernetes برای عملیات‌های زیر به گواهی‌نامه‌های PKI نیاز دارد:

* گواهی‌نامه‌های کلاینت برای kubelet برای احراز هویت در برابر سرور API
* گواهی‌نامه‌های سرور kubelet برای API server برای ارتباط با kubelets
* گواهی‌نامه‌ی سرور برای نقطه‌ی پایان API
* گواهی‌نامه‌های کلاینت برای مدیران خوشه برای احراز هویت در برابر سرور API
* گواهی‌نامه‌های کلاینت برای API server برای ارتباط با kubelets
* گواهی‌نامه‌های کلاینت برای API server برای ارتباط با etcd
* گواهی‌نامه‌ی کلاینت/kubeconfig برای مدیر کنترلر برای ارتباط با سرور API
* گواهی‌نامه‌ی کلاینت/kubeconfig برای اسکجولر برای ارتباط با سرور API
* گواهی‌نامه‌های کلاینت و سرور برای [پروکسی جلویی](/docs/tasks/extend-kubernetes/configure-aggregation-layer/)

{{< note >}}
گواهی‌نامه‌های `front-proxy` تنها در صورت استفاده از kube-proxy برای پشتیبانی از [سرور API توسعه](/docs/tasks/extend-kubernetes/setup-extension-api-server/) لازم است.
{{< /note >}}

etcd نیز اجرای TLS متقابل را برای احراز هویت مشتریان و همتایان پیاده‌سازی می‌کند.

## محل ذخیره گواهی‌نامه‌ها

اگر شما Kubernetes را با kubeadm نصب می‌کنید، بیشتر گواهی‌نامه‌ها در مسیر `/etc/kubernetes/pki` ذخیره می‌شوند. تمام مسیرهای مستندات در این مسیر نسبت به آن است، به جز گواهی‌نامه‌های حساب‌های کاربری که kubeadm آن‌ها را در `/etc/kubernetes` قرار می‌دهد.

## پیکربندی دستی گواهی‌نامه‌ها

اگر نمی‌خواهید kubeadm گواهی‌نامه‌های لازم را تولید کند، می‌توانید آن‌ها را با استفاده از یک CA ریشه یا با ارائه تمام گواهی‌نامه‌ها ایجاد کنید. برای جزئیات بیشتر در مورد ایجاد گواهی‌نامه‌های خود، به [Certificates](/docs/tasks/administer-cluster/certificates/) مراجعه کنید. برای مدیریت گواهی‌نامه‌ها، به [Certificate Management with kubeadm](/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/) مراجعه کنید.

### CA ریشه تک

می‌توانید یک CA ریشه تک ایجاد کنید که توسط یک مدیر کنترل می‌شود. این CA ریشه سپس می‌تواند چندین CA میانی ایجاد کند و همه ایجادات بعدی را به Kubernetes اختصاص دهد.

CA‌های مورد نیاز:

| مسیر                   | نام پیش فرض          | توضیحات                      |
|------------------------|-----------------------|------------------------------|
| ca.crt,key             | kubernetes-ca         | CA کلی Kubernetes             |
| etcd/ca.crt,key        | etcd-ca               | برای تمام عملیات مربوط به etcd  |
| front-proxy-ca.crt,key | kubernetes-front-proxy-ca | برای [پروکسی جلویی](/docs/tasks/extend-kubernetes/configure-aggregation-layer/) |

علاوه بر CA‌های فوق، نیاز به جفت کلید عمومی/خصوصی برای مدیریت حساب‌های سرویس `sa.key` و `sa.pub` دارید.
مثال زیر شامل فایل‌های کلید و گواهی CA در جدول قبلی نشان داده شده است:

```
/etc/kubernetes/pki/ca.crt
/etc/kubernetes/pki/ca.key
/etc/kubernetes/pki/etcd/ca.crt
/etc/kubernetes/pki/etcd/ca.key
/etc/kubernetes/pki/front-proxy-ca.crt
/etc/kubernetes/pki/front-proxy-ca.key
```

### تمام گواهی‌نامه‌ها

اگر نمی‌خواهید کلید‌های خصوصی CA را به خوشه خود کپی کنید، می‌توانید تمام گواهی‌نامه‌ها را خود تولید کنید.

گواهی‌نامه‌های مورد نیاز:

| نام پیش فرض                   | CA مادر                | O (در Subject) | نوع              | hosts (SAN)                                         |
|-------------------------------|---------------------------|----------------|------------------|-----------------------------------------------------|
| kube-etcd                     | etcd-ca                   |                | server, client   | `<hostname>`, `<Host_IP>`, `localhost`, `127.0.0.1` |
| kube-etcd-peer                | etcd-ca                   |                | server, client   | `<hostname>`, `<Host_IP>`, `localhost`, `127.0.0.1` |
| kube-etcd-healthcheck-client  | etcd-ca                   |                | client           |                                                     |
| kube-apiserver-etcd-client    | etcd-ca                   |                | client           |                                                     |
| kube-apiserver                | kubernetes-ca             |                | server           | `<hostname>`, `<Host_IP>`, `<advertise_IP>`, `[1]`  |
| kube-apiserver-k

ubelet-client | kubernetes-ca             | system:masters | client           |                                                     |
| front-proxy-client            | kubernetes-front-proxy-ca |                | client           |                                                     |

{{< note >}}
به جای استفاده از گروه کاربری ابر-کاربر `system:masters` برای `kube-apiserver-kubelet-client` می‌توانید از یک گروه با دسترسی کمتر استفاده کنید. kubeadm برای این منظور از گروه `kubeadm:cluster-admins` استفاده می‌کند.
{{< /note >}}

[1]: هر IP دیگر یا نام DNS که شما با خوشه خود تماس می‌گیرید (همانطور که توسط [kubeadm](/docs/reference/setup-tools/kubeadm/)
IP و/یا نام DNS پایدار متوازن، `kubernetes`، `kubernetes.default`، `kubernetes.default.svc`,
`kubernetes.default.svc.cluster`، `kubernetes.default.svc.cluster.local`)

که `kind` به یک یا چندین استفاده از کلید x509 mapping می‌کند که همچنین در `.spec.usages` یک [CertificateSigningRequest](/docs/reference/kubernetes-api/authentication-resources/certificate-signing-request-v1#CertificateSigningRequest)
نوع:

| kind   | استفاده از کلید                                                                       |
|--------|---------------------------------------------------------------------------------|
| server | امضای دیجیتال، رمزگذاری کلید، احراز هویت سرور                                |
| client | امضای دیجیتال، رمزگذاری کلید، احراز هویت کلاینت                                |

{{< note >}}
هاست‌های/SAN‌های فوق برای یک خوشه کار می‌کنند؛ اگر توسط یک راه‌اندازی خاص مورد نیاز باشد، امکان اضافه کردن SAN‌های اضافی در تمام گواهی‌نامه‌های سرور وجود دارد.
{{< /note >}}

{{< note >}}
برای کاربران kubeadm فقط:

* سناریویی که شما گواهی‌نامه‌های CA خود را بدون کلید‌های خصوصی به خوشه خود کپی می‌کنید به عنوان CA خارجی در مستندات kubeadm مشخص شده است.
* اگر شما لیست بالا را با یک PKI تولید شده توسط kubeadm مقایسه می‌کنید، لطفاً به یاد داشته باشید که گواهی‌نامه‌های `kube-etcd`، `kube-etcd-peer` و `kube-etcd-healthcheck-client` در صورت etcd خارجی تولید نمی‌شوند.
{{< /note >}}

### مسیرهای گواهی‌نامه‌ها

گواهی‌نامه‌ها باید در یک مسیر پیشنهادی قرار گیرند (همانطور که توسط [kubeadm](/docs/reference/setup-tools/kubeadm/) استفاده می‌شود).
مسیرها باید با استفاده از آرگومان‌های داده شده مشخص شوند، بدون توجه به مکان.

| CN پیش‌فرض                   | مسیر پیشنهادی کلید         | مسیر پیشنهادی گواهی       | دستور                 | آرگومان کلیدی               | آرگومان گواهی‌نامه                        |
|------------------------------|------------------------------|-----------------------------|-------------------------|------------------------------|-------------------------------------------|
| etcd-ca                      | etcd/ca.key                  | etcd/ca.crt                 | kube-apiserver          |                              | --etcd-cafile                             |
| kube-apiserver-etcd-client   | apiserver-etcd-client.key    | apiserver-etcd-client.crt   | kube-apiserver          | --etcd-keyfile               | --etcd-certfile                           |
| kubernetes-ca                | ca.key                       | ca.crt                      | kube-apiserver          |                              | --client-ca-file                          |
| kubernetes-ca                | ca.key                       | ca.crt                      | kube-controller-manager | --cluster-signing-key-file   | --client-ca-file, --root-ca-file, --cluster-signing-cert-file |
| kube-apiserver               | apiserver.key                | apiserver.crt               | kube-apiserver          | --tls-private-key-file       | --tls-cert-file                           |
| kube-apiserver-kubelet-client| apiserver-kubelet-client.key | apiserver-kubelet-client.crt| kube-apiserver          | --kubelet-client-key         | --kubelet-client-certificate              |
| front-proxy-ca               | front-proxy-ca.key           | front-proxy-ca.crt          | kube-apiserver          |                              | --requestheader-client-ca-file            |
| front-proxy-ca               | front-proxy-ca.key           | front-proxy-ca.crt          | kube-controller-manager |                              | --requestheader-client-ca-file            |
| front-proxy-client           | front-proxy-client.key       | front-proxy-client.crt      | kube-apiserver          | --proxy-client-key-file      | --proxy-client-cert-file                  |
| etcd-ca                      | etcd/ca.key                  | etcd/ca.crt                 | etcd                    |                              | --trusted-ca-file, --peer-trusted-ca-file |
| kube-etcd                    | etcd/server.key              | etcd/server.crt             | etcd                    | --key-file                   | --cert-file                               |
| kube-etcd-peer               | etcd/peer.key                | etcd/peer.crt               | etcd                    | --peer-key-file              | --peer-cert-file                          |
| etcd-ca                      |                              | etcd/ca.crt                 | etcdctl                 |                              | --cacert                                  |
| kube-etcd-healthcheck-client | etcd/healthcheck-client.key  | etcd/healthcheck-client.crt | etcdctl                 | --key                        | --cert                                    |

همان ملاحظات برای جفت‌های کلید حساب‌های خدماتی نیز اعمال می‌شود:

| مسیر کلید خصوصی  | مسیر کلید عمومی  | دستور                 | آرگومان                             |
|-------------------|------------------|-------------------------|--------------------------------------|
|  sa.key           |                  | kube-controller-manager | --service-account-private-key-file   |
|                   | sa.pub           | kube-apiserver          | --service-account-key-file           |

مثال زیر مسیرهای فایل‌ها را نشان می‌دهد که [در جداول قبلی](#certificate-paths) نیاز دارید اگر همه کلیدها و گواهی‌نامه‌های خود را ایجاد می‌کنید:

```
/etc/kubernetes/pki/etcd/ca.key
/etc/kubernetes/pki/etcd/ca.crt
/etc/kubernetes/pki/apiserver-etcd-client.key
/etc/kubernetes/pki/apiserver-etcd-client.crt
/etc/kubernetes/pki/ca.key
/etc/kubernetes/pki/ca.crt
/etc/kubernetes/pki/apiserver.key
/etc/kubernetes/pki/apiserver.crt
/etc/kubernetes/pki/apiserver-kubelet-client.key
/etc/kubernetes/pki/apiserver-kubelet-client.crt
/etc/kubernetes/pki/front-proxy-ca.key
/etc/kubernetes/pki/front-proxy-ca.crt
/etc/kubernetes/pki/front-proxy-client.key
/etc/kubernetes/pki/front-proxy-client.crt
/etc/kubernetes/pki/etcd/server.key
/etc/kubernetes/pki/etcd/server.crt
/etc/kubernetes/pki/etcd/peer.key
/etc/kubernetes/pki/etcd/peer.crt
/etc/kubernetes/pki/etcd/healthcheck-client.key
/etc/kubernetes/pki/etcd/healthcheck-client.crt
/etc/kubernetes/pki/sa.key
/etc/kubernetes/pki/sa.pub
```

## پیکربندی گواهی‌نامه‌ها برای حساب‌های کاربری

شما باید به صورت دستی حساب‌های مدیر و حساب‌های خدماتی زیر را پیکربندی کنید:

| نام فایل                | نام اعتباری               | CN پیش‌فرض                          | O (در Subject)         |
|---------------------------|----------------------------|---------------------------------------|------------------------|
| admin.conf                | default-admin              | kubernetes-admin                      | `<admin-group>`        |
| super-admin.conf          | default-super-admin        | kubernetes-super-admin                | system:masters         |
| kubelet.conf              | default-auth               | system:node:`<nodeName>` (نگاه داشته باشید) | system:nodes           |
| controller-manager.conf   | default-controller-manager | system:kube-controller-manager        |                        |
| scheduler.conf            | default-scheduler          | system:kube-scheduler                 |                        |

{{< note >}}
مقدار `<nodeName>` برای `kubelet.conf` **باید** دقیقاً با مقدار نام گره که توسط kubelet به apiserver اعلام می‌شود، هم‌خوانی داشته باشد.
برای جزئیات بیشتر، [مجوز گره](/docs/reference/access-authn-authz/node/) را بخوانید.
{{< /note >}}

{{< note >}}
در مثال بالا `<admin-group>` از پیاده‌سازی استفاده شده است. برخی ابزارها گواهی‌نامه را در `admin.conf` به عنوان گروه `system:masters` امضا می‌کنند.
`system:masters` یک گروه کاربری فوق برای برش شکسته‌شده، کاربر ابری می‌تواند از لایه احراز هویت Kubernetes، مانند RBAC گذر کند. همچنین برخی ابزار با گواهی‌نامه جداگانه `super-admin.conf` با گواهی‌نامه متصل به این گروه کاربری فوقی ساخته نمی‌شوند.
`kubeadm` دو گواهی‌نامه‌ی مدیر جداگانه را در فایل‌های kubeconfig تولید می‌کند.
یکی در `admin.conf` است که دارای `Subject: O = kubeadm:cluster-admins, CN = kubernetes-admin` است.
`kubeadm:cluster-admins` یک گروه سفارشی است که به `cluster-admin` ClusterRole متصل شده است.
این فایل بر روی تمام ماشین‌های کنترل صنعتی kubeadm تولید می‌شود.
دیگری در `super-admin.conf` که `Subject: O = system:masters, CN = kubernetes-super-admin` دارد.
این فایل تنها در گر

ه‌ای که `kubeadm init` را فراخوانی کرده است تولید می‌شود.
{{< /note >}}

1. برای هر پیکربندی، جفت x509 cert/key با CN و O داده شده را ایجاد کنید.

1. `kubectl` را برای هر پیکربندی به صورت زیر اجرا کنید:

```
KUBECONFIG=<filename> kubectl config set-cluster default-cluster --server=https://<host ip>:6443 --certificate-authority <path-to-kubernetes-ca> --embed-certs
KUBECONFIG=<filename> kubectl config set-credentials <credential-name> --client-key <path-to-key>.pem --client-certificate <path-to-cert>.pem --embed-certs
KUBECONFIG=<filename> kubectl config set-context default-system --cluster default-cluster --user <credential-name>
KUBECONFIG=<filename> kubectl config use-context default-system
```

این فایل‌ها به صورت زیر استفاده می‌شوند:

| نام فایل                | دستور                 | توضیحات                                                              |
|---------------------------|-------------------------|-----------------------------------------------------------------------|
| admin.conf                | kubectl                 | پیکربندی کاربر مدیر برای خوشه                                      |
| super-admin.conf          | kubectl                 | پیکربندی کاربر فوق‌العاده مدیر برای خوشه                           |
| kubelet.conf              | kubelet                 | یکی برای هر گره در خوشه مورد نیاز است.                             |
| controller-manager.conf   | kube-controller-manager | باید به منظوری در `manifests/kube-controller-manager.yaml` اضافه شود |
| scheduler.conf            | kube-scheduler          | باید به منظوری در `manifests/kube-scheduler.yaml` اضافه شود        |

فایل‌های زیر مسیرهای کاملی از فایل‌های لیست شده در جداول قبلی را نشان می‌دهند:

```
/etc/kubernetes/admin.conf
/etc/kubernetes/super-admin.conf
/etc/kubernetes/kubelet.conf
/etc/kubernetes/controller-manager.conf
/etc/kubernetes/scheduler.conf
```
