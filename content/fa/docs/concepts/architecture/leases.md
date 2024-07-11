---
title: Leases
api_metadata:
- apiVersion: "coordination.k8s.io/v1"
  kind: "Lease"
content_type: concept
weight: 30
---

<!-- overview -->

سیستم‌های توزیع شده اغلب نیاز به _لیز_ دارند که مکانیزمی برای قفل کردن منابع مشترک و هماهنگی فعالیت بین اعضای یک مجموعه فراهم می‌کند.
در Kubernetes، مفهوم لیز توسط اشیاء [Lease](/docs/reference/kubernetes-api/cluster-resources/lease-v1/) در {{< glossary_tooltip text="گروه API" term_id="api-group" >}} `coordination.k8s.io` نمایان می‌شود،
که برای قابلیت‌های حیاتی سیستم مانند ضربان قلب نود و انتخاب رهبر در سطح کامپوننت‌ها استفاده می‌شود.

<!-- body -->

## ضربان قلب نودها {#node-heart-beats}

Kubernetes از API لیز برای انتقال ضربان قلب نود kubelet به سرور API Kubernetes استفاده می‌کند.
برای هر `Node`، یک شیء `Lease` با نام منطبق در فضای نام `kube-node-lease` وجود دارد.
در پشت صحنه، هر ضربان قلب kubelet یک درخواست **به‌روزرسانی** به این شیء `Lease` است که فیلد `spec.renewTime` را برای لیز به‌روزرسانی می‌کند.
پلان کنترل Kubernetes از زمان ثبت شده در این فیلد برای تعیین دسترسی این `Node` استفاده می‌کند.

برای جزئیات بیشتر به [اشیاء لیز نود](/docs/concepts/architecture/nodes/#node-heartbeats) مراجعه کنید.

## انتخاب رهبر

Kubernetes همچنین از لیزها برای اطمینان از اجرای تنها یک نسخه از یک کامپوننت در هر لحظه استفاده می‌کند.
این مورد توسط کامپوننت‌های پلان کنترل مانند `kube-controller-manager` و `kube-scheduler` در تنظیمات HA استفاده می‌شود،
جایی که تنها یک نسخه از کامپوننت باید فعال باشد در حالی که نسخه‌های دیگر در حالت آماده‌باش هستند.

## هویت سرور API

{{< feature-state feature_gate_name="APIServerIdentity" >}}

از Kubernetes نسخه 1.26، هر `kube-apiserver` از API لیز برای انتشار هویت خود به سایر سیستم استفاده می‌کند.
اگرچه به تنهایی خیلی مفید نیست، این مکانیزمی برای کشف تعداد نسخه‌های `kube-apiserver` که پلان کنترل Kubernetes را اجرا می‌کنند فراهم می‌کند.
وجود لیزهای kube-apiserver قابلیت‌های آینده‌ای که ممکن است به هماهنگی بین هر kube-apiserver نیاز داشته باشند را فعال می‌کند.

می‌توانید لیزهای مالکیت‌یافته توسط هر kube-apiserver را با بررسی اشیاء لیز در فضای نام `kube-system` با نام `kube-apiserver-<sha256-hash>` بررسی کنید.
یا می‌توانید از انتخاب‌گر برچسب `apiserver.kubernetes.io/identity=kube-apiserver` استفاده کنید:

```shell
kubectl -n kube-system get lease -l apiserver.kubernetes.io/identity=kube-apiserver
```
```
NAME                                        HOLDER                                                                           AGE
apiserver-07a5ea9b9b072c4a5f3d1c3702        apiserver-07a5ea9b9b072c4a5f3d1c3702_0c8914f7-0f35-440e-8676-7844977d3a05        5m33s
apiserver-7be9e061c59d368b3ddaf1376e        apiserver-7be9e061c59d368b3ddaf1376e_84f2a85d-37c1-4b14-b6b9-603e62e4896f        4m23s
apiserver-1dfef752bcb36637d2763d1868        apiserver-1dfef752bcb36637d2763d1868_c5ffa286-8a9a-45d4-91e7-61118ed58d2e        4m43s

```

هش SHA256 استفاده شده در نام لیز بر اساس نام میزبان OS که توسط آن سرور API دیده می‌شود است.
هر kube-apiserver باید به گونه‌ای پیکربندی شود که از نام میزبان منحصر به فرد در کلاستر استفاده کند.
نسخه‌های جدید kube-apiserver که از همان نام میزبان استفاده می‌کنند لیزهای موجود را با استفاده از هویت جدید مالکیت می‌گیرند،
به جای ایجاد اشیاء لیز جدید.
می‌توانید نام میزبان استفاده شده توسط kube-apisever را با بررسی مقدار برچسب `kubernetes.io/hostname` بررسی کنید:

```shell
kubectl -n kube-system get lease apiserver-07a5ea9b9b072c4a5f3d1c3702 -o yaml
```
```yaml
apiVersion: coordination.k8s.io/v1
kind: Lease
metadata:
  creationTimestamp: "2023-07-02T13:16:48Z"
  labels:
    apiserver.kubernetes.io/identity: kube-apiserver
    kubernetes.io/hostname: master-1
  name: apiserver-07a5ea9b9b072c4a5f3d1c3702
  namespace: kube-system
  resourceVersion: "334899"
  uid: 90870ab5-1ba9-4523-b215-e4d4e662acb1
spec:
  holderIdentity: apiserver-07a5ea9b9b072c4a5f3d1c3702_0c8914f7-0f35-440e-8676-7844977d3a05
  leaseDurationSeconds: 3600
  renewTime: "2023-07-04T21:58:48.065888Z"
```

لیزهای منقضی شده از kube-apiserverهایی که دیگر وجود ندارند توسط kube-apiserverهای جدید بعد از 1 ساعت جمع‌آوری زباله می‌شوند.

می‌توانید لیزهای هویت سرور API را با غیرفعال کردن [feature gate](/docs/reference/command-line-tools-reference/feature-gates/) `APIServerIdentity` غیرفعال کنید.

## بارهای کاری {#custom-workload}

بار کاری خودتان می‌تواند استفاده خاص خود از لیزها را تعریف کند.
برای مثال، شما ممکن است یک {{< glossary_tooltip term_id="controller" text="کنترلر" >}} سفارشی اجرا کنید که یک عضو اولیه یا رهبر عملیات‌هایی را که همتایانش انجام نمی‌دهند انجام دهد.
شما یک لیز تعریف می‌کنید تا نسخه‌های کنترلر بتوانند یک رهبر را انتخاب کنند یا انتخاب کنند، با استفاده از API Kubernetes برای هماهنگی.
اگر از لیز استفاده می‌کنید، بهترین کار این است که نامی برای لیز تعریف کنید که به وضوح به محصول یا کامپوننت مرتبط باشد.
برای مثال، اگر یک کامپوننت به نام Example Foo دارید، از یک لیز به نام `example-foo` استفاده کنید.

اگر یک اپراتور کلاستر یا کاربر نهایی دیگری می‌تواند چندین نسخه از یک کامپوننت را استقرار دهد، یک پیشوند نام انتخاب کنید و از یک مکانیزم (مانند هش نام استقرار) برای جلوگیری از تداخل نام‌ها برای لیزها استفاده کنید.

می‌توانید از رویکرد دیگری استفاده کنید تا زمانی که به همان نتیجه برسد: محصولات نرم‌افزاری مختلف با یکدیگر تداخل نداشته باشند.
