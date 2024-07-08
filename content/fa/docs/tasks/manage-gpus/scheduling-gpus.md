---
reviewers:
- vishh
content_type: concept
title: برنامه‌ریزی GPU‌ها
description: پیکربندی و برنامه‌ریزی GPU‌ها برای استفاده به عنوان یک منبع توسط نودها در یک خوشه.
---

<!-- مرور -->

{{< feature-state state="stable" for_k8s_version="v1.26" >}}

Kubernetes پشتیبانی **ثابت** از مدیریت GPU‌های AMD و NVIDIA را در سراسر نودهای خوشه شما با استفاده از
{{< glossary_tooltip text="device plugins" term_id="device-plugin" >}}
شامل می‌شود.

این صفحه توضیح می‌دهد که کاربران چگونه می‌توانند GPU‌ها را مصرف کنند و محدودیت‌های موجود در پیاده‌سازی را شرح می‌دهد.

<!-- محتوا -->

## استفاده از پلاگین‌های دستگاه

Kubernetes پلاگین‌های دستگاه را پیاده‌سازی می‌کند تا به پاد‌ها اجازه دسترسی به ویژگی‌های سخت‌افزاری ویژه مانند GPU بدهد.

{{% thirdparty-content %}}

به عنوان یک مدیر، شما باید در نودها درایورهای GPU را از تولیدکننده‌ی سخت‌افزار مربوطه نصب کرده و پلاگین دستگاه مربوطه را از سمت تولیدکننده GPU اجرا کنید. در ادامه لینک‌های دستورالعمل تولیدکنندگان را می‌توانید ببینید:

* [AMD](https://github.com/RadeonOpenCompute/k8s-device-plugin#deployment)
* [Intel](https://intel.github.io/intel-device-plugins-for-kubernetes/cmd/gpu_plugin/README.html)
* [NVIDIA](https://github.com/NVIDIA/k8s-device-plugin#quick-start)

بعد از نصب پلاگین، خوشه‌ی شما یک منبع برنامه‌ریزی شده اختصاصی مانند `amd.com/gpu` یا `nvidia.com/gpu` را فراهم می‌کند.

می‌توانید این GPU‌ها را از طریق کانتینرهای خود مصرف کنید با درخواست منبع GPU سفارشی، به همان روشی که برای `cpu` یا `memory` درخواست می‌کنید. با این حال، تعدادی محدودیت در نحوه‌ی مشخص‌کردن الزامات منابع برای دستگاه‌های سفارشی وجود دارد.

GPU‌ها فقط باید در بخش `limits` مشخص شوند، یعنی:
* می‌توانید `limits` GPU را بدون مشخص‌کردن `requests` مشخص کنید، زیرا Kubernetes به‌طور پیش‌فرض از مقدار `limit` به عنوان مقدار `request` استفاده می‌کند.
* می‌توانید GPU را هم در `limits` و هم در `requests` مشخص کنید اما این دو مقدار باید برابر باشند.
* نمی‌توانید `requests` GPU را بدون مشخص‌کردن `limits` مشخص کنید.

در ادامه مانیفست نمونه‌ای برای یک Pod که یک GPU درخواست می‌کند آمده است:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-vector-add
spec:
  restartPolicy: OnFailure
  containers:
    - name: example-vector-add
      image: "registry.example/example-vector-add:v42"
      resources:
        limits:
          gpu-vendor.example/example-gpu: 1 # درخواست 1 GPU
```

## مدیریت خوشه با انواع مختلف GPU‌ها

اگر نودهای مختلف در خوشه شما دارای انواع مختلف GPU باشند، می‌توانید از [برچسب‌های Node و Selector‌های Node](/docs/tasks/configure-pod-container/assign-pods-nodes/) برای برنامه‌ریزی پادها به نودهای مناسب استفاده کنید.

به عنوان مثال:

```shell
# نودهای خود را با نوع شتاب‌دهنده‌ی موجود در آن برچسب‌گذاری کنید.
kubectl label nodes node1 accelerator=example-gpu-x100
kubectl label nodes node2 accelerator=other-gpu-k915
```

کلید برچسب `accelerator` فقط یک مثال است؛ می‌توانید از یک کلید برچسب دیگر استفاده کنید اگر ترجیح می‌دهید.

## تغییر خودکار برچسب نود {#node-labeller}

به عنوان یک مدیر، می‌توانید به طور خودکار همه‌ی نودهای خود را کشف و برچسب‌گذاری کنید که دارای GPU هستند با اجرای [Node Feature Discovery](https://github.com/kubernetes-sigs/node-feature-discovery) (NFD) Kubernetes.
NFD ویژگی‌های سخت‌افزاری که بر روی هر نود در یک خوشه Kubernetes موجود هستند را شناسایی می‌کند.
به طور معمول NFD به گونه‌ای تنظیم می‌شود که این ویژگی‌ها را به عنوان برچسب‌های نود تبلیغ می‌کند، اما NFD می‌تواند منابع توسعه یافته، حاشیه‌نشانی‌ها و انوتیشن‌ها را هم اضافه کند.
NFD با همه‌ی [نسخه‌های پشتیبانی شده](/releases/version-skew-policy/#supported-versions) Kubernetes سازگار است.
به طور پیش‌فرض NFD برچسب‌های [ویژگی](https://kubernetes-sigs.github.io/node-feature-discovery/master/usage/features.html) برای ویژگی‌های شناسایی شده ایجاد می‌کند.
مدیران می‌توانند از NFD برای تینت‌گذاری نودها با ویژگی‌های خاص استفاده کنند، به‌طوری که فقط پادهایی که این ویژگی‌ها را درخواست کنند بر روی این نودها برنامه‌ریزی شوند.

همچنین نیا

ز به پلاگین برای NFD است که برچسب‌های مناسب را به نودهای شما اضافه کند؛ این برچسب‌ها ممکن است برچسب‌های عمومی یا ممکن است از طرف تولیدکننده‌ی GPU ارائه شود. برای اطلاعات بیشتر جزئیات مربوط به پلاگین NFD ارائه دهنده‌ی GPU خود را بررسی کنید.

{{< highlight yaml "linenos=false,hl_lines=7-18" >}}
apiVersion: v1
kind: Pod
metadata:
  name: example-vector-add
spec:
  restartPolicy: OnFailure
  # می‌توانید بازیابی ترجیحات نود Kubernetes را برای برنامه‌ریزی این Pod بر روی یک نود استفاده کنید
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "gpu.gpu-vendor.example/installed-memory"
            operator: Gt # (greater than)
            values: ["40535"]
          - key: "feature.node.kubernetes.io/pci-10.present" # برچسب ویژگی NFD
            values: ["true"] # (اختیاری) فقط برنامه‌ریزی بر روی نودهایی با دستگاه PCI 10
  containers:
    - name: example-vector-add
      image: "registry.example/example-vector-add:v42"
      resources:
        limits:
          gpu-vendor.example/example-gpu: 1 # درخواست 1 GPU
{{< /highlight >}}

#### پیاده‌سازی‌های تولیدکنندگان GPU

- [Intel](https://intel.github.io/intel-device-plugins-for-kubernetes/cmd/gpu_plugin/README.html)
- [NVIDIA](https://github.com/NVIDIA/gpu-feature-discovery/#readme)
