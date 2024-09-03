---
title: راه‌اندازی خدمات Konnectivity
content_type: task
weight: 70
---

<!-- overview -->

سرویس Konnectivity یک پروکسی سطح TCP برای ارتباط پلیر کنترل با خوشه فراهم می‌کند.

## {{% heading "پیش‌نیازها" %}}

شما باید یک خوشه Kubernetes داشته باشید و ابزار خط فرمان kubectl باید پیکربندی شده باشد تا با خوشه شما ارتباط برقرار کند. توصیه می‌شود این آموزش را در یک خوشه با حداقل دو نود که به عنوان میزبان‌های پلیر کنترل عمل نمی‌کنند، اجرا کنید. اگر خوشه‌ای ندارید، می‌توانید یکی با استفاده از [minikube](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/) ایجاد کنید.

<!-- steps -->

## پیکربندی خدمات Konnectivity

مراحل زیر نیازمند یک پیکربندی خروجی هستند، به عنوان مثال:

{{% code_sample file="admin/konnectivity/egress-selector-configuration.yaml" %}}

شما باید API Server را برای استفاده از سرویس Konnectivity پیکربندی کنید و ترافیک شبکه را به نود‌های خوشه هدایت کنید:

1. اطمینان حاصل کنید که ویژگی [Service Account Token Volume Projection](/docs/tasks/configure-pod-container/configure-service-account/#serviceaccount-token-volume-projection) در خوشه شما فعال شده است. این از نسخه Kubernetes v1.20 به بعد به طور پیش‌فرض فعال است.
1. یک فایل پیکربندی خروجی مانند `admin/konnectivity/egress-selector-configuration.yaml` ایجاد کنید.
1. پرچم `--egress-selector-config-file` API Server را به مسیر فایل پیکربندی خروجی API Server خود تنظیم کنید.
1. اگر از اتصال UDS استفاده می‌کنید، پیکربندی volumes را به kube-apiserver اضافه کنید:
   ```yaml
   spec:
     containers:
       volumeMounts:
       - name: konnectivity-uds
         mountPath: /etc/kubernetes/konnectivity-server
         readOnly: false
     volumes:
     - name: konnectivity-uds
       hostPath:
         path: /etc/kubernetes/konnectivity-server
         type: DirectoryOrCreate
   ```

یک گواهی و kubeconfig برای konnectivity-server تولید یا به دست آورید. به عنوان مثال، می‌توانید از ابزار خط فرمان OpenSSL برای صدور گواهی X.509 استفاده کنید، با استفاده از گواهی CA خوشه `/etc/kubernetes/pki/ca.crt` از میزبان پلیر کنترل.

```bash
openssl req -subj "/CN=system:konnectivity-server" -new -newkey rsa:2048 -nodes -out konnectivity.csr -keyout konnectivity.key
openssl x509 -req -in konnectivity.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out konnectivity.crt -days 375 -sha256
SERVER=$(kubectl config view -o jsonpath='{.clusters..server}')
kubectl --kubeconfig /etc/kubernetes/konnectivity-server.conf config set-credentials system:konnectivity-server --client-certificate konnectivity.crt --client-key konnectivity.key --embed-certs=true
kubectl --kubeconfig /etc/kubernetes/konnectivity-server.conf config set-cluster kubernetes --server "$SERVER" --certificate-authority /etc/kubernetes/pki/ca.crt --embed-certs=true
kubectl --kubeconfig /etc/kubernetes/konnectivity-server.conf config set-context system:konnectivity-server@kubernetes --cluster kubernetes --user system:konnectivity-server
kubectl --kubeconfig /etc/kubernetes/konnectivity-server.conf config use-context system:konnectivity-server@kubernetes
rm -f konnectivity.crt konnectivity.key konnectivity.csr
```

سپس، باید سرور Konnectivity و عوامل آن را استقرار دهید. [kubernetes-sigs/apiserver-network-proxy](https://github.com/kubernetes-sigs/apiserver-network-proxy) یک پیاده‌سازی مرجع است.

سرور Konnectivity را در گره پلیر کنترل خود استقرار دهید. منظور از `konnectivity-server.yaml` منیفست فرض می‌شود که مؤلفه‌های Kubernetes به عنوان یک {{< glossary_tooltip text="static Pod" term_id="static-pod" >}} در خوشه شما استقرار یافته است. اگر اینطور نیست، می‌توانید سرور Konnectivity را به عنوان یک DaemonSet استقرار دهید.

{{% code_sample file="admin/konnectivity/konnectivity-server.yaml" %}}

سپس عوامل Konnectivity را در خوشه خود استقرار دهید:

{{% code_sample file="admin/konnectivity/konnectivity-agent.yaml" %}}

در نهایت، اگر RBAC در خوشه شما فعال است، قوانین RBAC مربوطه را ایجاد کنید:

{{% code_sample file="admin/konnectivity/konnectivity-rbac.yaml" %}}
