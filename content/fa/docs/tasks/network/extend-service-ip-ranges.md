---
reviewers:
- thockin
- dwinship
min-kubernetes-server-version: v1.29
title: "گسترش برد IP‌های سرویس"
content_type: task
---

<!-- overview -->
{{< feature-state feature_gate_name="MultiCIDRServiceAllocator" >}}

این سند نحوه گسترش برد IP موجود اختصاص داده شده به یک خوشه را به اشتراک می‌گذارد.


## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}}

{{< version-check >}}

<!-- steps -->

## API

خوشه‌های Kubernetes با kube-apiservers که ویژگی `MultiCIDRServiceAllocator`
[دروازه ویژگی‌ها](/docs/reference/command-line-tools-reference/feature-gates/) را فعال کرده‌اند و API `networking.k8s.io/v1alpha1`،
یک شی ServiceCIDR جدید ایجاد می‌کنند که نام مشهور `kubernetes` را دارد و از طیف آدرس IP استفاده می‌کند
بر اساس مقدار آرگومان خط فرمان `--service-cluster-ip-range` به kube-apiserver.

```sh
kubectl get servicecidr
```
```
NAME         CIDRS          AGE
kubernetes   10.96.0.0/28   17d
```

سرویس مشهور `kubernetes`، که نقطه‌ی انتهایی kube-apiserver را به پادها ارائه می‌دهد،
آدرس IP اول از طیف پیش‌فرض ServiceCIDR را محاسبه می‌کند و این آدرس IP را به عنوان آدرس IP خوشه خود استفاده می‌کند.

```sh
kubectl get service kubernetes
```
```
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   17d
```

سرویس پیش‌فرض، در این مورد، از ClusterIP 10.96.0.1 استفاده می‌کند، که دارای شی IPAddress متناظر است.

```sh
kubectl get ipaddress 10.96.0.1
```
```
NAME        PARENTREF
10.96.0.1   services/default/kubernetes
```

ServiceCIDR ها با استفاده از {{<glossary_tooltip text="finalizers" term_id="finalizer">}} محافظت می‌شوند تا از ایجاد IP های ClusterIP یا یتیم جلوگیری کنند.
فاینالایزر تنها زمانی حذف می‌شود که یک زیرشبکه دیگر وجود داشته باشد که شامل IPAddress های موجود باشد یا
IPAddress های زیرشبکه مورد نظری وجود نداشته باشد.

## گسترش تعداد آدرس‌های IP موجود برای سرویس‌ها

مواردی وجود دارد که کاربران نیاز دارند تا تعداد آدرس‌های موجود برای سرویس‌ها را افزایش دهند، قبلاً افزایش برد سرویس یک عملیات مخرب بود که ممکن بود از دست دادن اطلاعات را نیز ایجاد کند. با این ویژگی جدید، کاربران فقط نیاز دارند که یک ServiceCIDR جدید را اضافه کنند تا تعداد آدرس‌های موجود را افزایش دهند.

### افزودن یک ServiceCIDR جدید

در یک خوشه با محدوده 10.96.0.0/28 برای سرویس‌ها، فقط 2^(32-28) - 2 = 14 آدرس IP در دسترس است. سرویس `kubernetes.default` همیشه ایجاد می‌شود؛ برای این مثال، این به شما 13 سرویس ممکن را باقی می‌گذارد.

```sh
for i in $(seq 1 13); do kubectl create service clusterip "test-$i" --tcp 80 -o json | jq -r .spec.clusterIP; done
```
```
10.96.0.11
10.96.0.5
10.96.0.12
10.96.0.13
10.96.0.14
10.96.0.2
10.96.0.3
10.96.0.4
10.96.0.6
10.96.0.7
10.96.0.8
10.96.0.9
error: failed to create ClusterIP service: Internal error occurred: failed to allocate a serviceIP: range is full
```

می‌توانید تعداد آدرس‌های IP موجود برای سرویس‌ها را با ایجاد یک ServiceCIDR جدید که طیف آدرس IP جدید را گسترش می‌دهد یا اضافه می‌کند، افزایش دهید.

```sh
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1alpha1
kind: ServiceCIDR
metadata:
  name: newcidr1
spec:
  cidrs:
  - 10.96.0.0/24
EOF
```
```
servicecidr.networking.k8s.io/newcidr1 created
```

این به شما اجازه می‌دهد که سرویس‌های جدیدی با ClusterIP هایی که از این محدوده جدید انتخاب می‌شوند، ایجاد کنید.

```sh
for i in $(seq 13 16); do kubectl create service clusterip "test-$i" --tcp 80 -o json | jq -r .spec.clusterIP; done
```
```
10.96.0.48
10.96.0.200
10.96.0.121
10.96.0.144
```

### حذف یک ServiceCIDR

نمی‌توانید یک ServiceCIDR را حذف کنید اگر IPAddress هایی وابسته به آن وجود داشته باشد.

```sh
kubectl delete servicecidr newcidr1
```
```
servicecidr.networking.k8s.io "newcidr1" deleted
```

Kubernetes از finalizer بر روی ServiceCIDR استفاده می‌کند تا این رابطه وابسته را پیگیری کند.

```sh
kubectl get servicecidr newcidr1 -o yaml
```
```
apiVersion: networking.k8s.io/v1alpha1
kind: ServiceCIDR
metadata:
  creationTimestamp: "2023-10-12T15:11:07Z"
  deletionGracePeriodSeconds: 0
  deletionTimestamp: "2023-10-12T15:12:45Z"
  finalizers:
  - networking.k8s.io/service-cidr-finalizer
  name: newcidr1
  resourceVersion: "1133"
  uid: 5ffd8afe-c78f-4e60-ae76-cec448a8af40
spec:
  cidrs:
  - 10.96

.0.0/24
status:
  conditions:
  - lastTransitionTime: "2023-10-12T15:12:45Z"
    message: There are still IPAddresses referencing the ServiceCIDR, please remove
      them or create a new ServiceCIDR
    reason: OrphanIPAddress
    status: "False"
    type: Ready
```

با حذف سرویس‌های حاوی آدرس‌های IP که مانع حذف ServiceCIDR هستند

```sh
for i in $(seq 13 16); do kubectl delete service "test-$i" ; done
```
```
service "test-13" deleted
service "test-14" deleted
service "test-15" deleted
service "test-16" deleted
```

صفحه کنترل حذف finalizer را تشخیص می‌دهد. سپس صفحه کنترل finalizer خود را حذف می‌کند،
تا ServiceCIDR در حال حاضر در حال حذف برداشته شود.

```sh
kubectl get servicecidr newcidr1
```
```
Error from server (NotFound): servicecidrs.networking.k8s.io "newcidr1" not found
```
