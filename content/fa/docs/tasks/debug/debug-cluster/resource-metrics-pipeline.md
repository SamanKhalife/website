```markdown
---
reviewers:
- fgrzadkowski
- piosz
title: Pipeline Metrics Resource
content_type: concept
weight: 15
---

<!-- overview -->

برای Kubernetes، _Metrics API_ مجموعه‌ای از متریک‌های پایه را برای پشتیبانی از اسکیلینگ خودکار و موارد مشابه فراهم می‌کند. این API اطلاعاتی را دربارهٔ استفاده از منابع برای نود و پاد ارائه می‌دهد، شامل متریک‌های CPU و حافظه است. اگر Metrics API را در خوشه خود نصب کنید، مشتریان API Kubernetes می‌توانند این اطلاعات را درخواست کنند و شما می‌توانید با استفاده از مکانیزم‌های کنترل دسترسی Kubernetes اجازه دسترسی به آن را مدیریت کنید.

[HorizontalPodAutoscaler](/docs/tasks/run-application/horizontal-pod-autoscale/) (HPA) و [VerticalPodAutoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler#readme) (VPA) از داده‌های ارائه شده توسط Metrics API برای تنظیم تعداد نمونه‌های کاری و منابع برای برآورده کردن تقاضای مشتری استفاده می‌کنند.

همچنین می‌توانید متریک‌های منابع را با استفاده از دستور [`kubectl top`](/docs/reference/generated/kubectl/kubectl-commands#top) مشاهده کنید.

{{< note >}}
Metrics API و پایپ‌لاین متریک که فراهم می‌کند، فقط متریک‌های حداقلی CPU و حافظه را برای اسکیلینگ خودکار با استفاده از HPA و/یا VPA فراهم می‌کند. اگر می‌خواهید مجموعه‌ای کامل‌تر از متریک‌ها را فراهم کنید، می‌توانید Metrics API ساده را با نصب یک دومین پایپ‌لاین [metrics pipeline](/docs/tasks/debug/debug-cluster/resource-usage-monitoring/#full-metrics-pipeline) که از _Custom Metrics API_ استفاده می‌کند، تکمیل کنید.
{{< /note >}}

Figure 1 نشان‌دهندهٔ معماری پایپ‌لاین متریک منابع است.

{{< mermaid >}}
flowchart RL
subgraph cluster[Cluster]
direction RL
S[ <br><br> ]
A[Metrics-<br>Server]
subgraph B[Nodes]
direction TB
D[cAdvisor] --> C[kubelet]
E[Container<br>runtime] --> D
E1[Container<br>runtime] --> D
P[pod data] -.- C
end
L[API<br>server]
W[HPA]
C ---->|node level<br>resource metrics| A -->|metrics<br>API| L --> W
end
L ---> K[kubectl<br>top]
classDef box fill:#fff,stroke:#000,stroke-width:1px,color:#000;
class W,B,P,K,cluster,D,E,E1 box
classDef spacewhite fill:#ffffff,stroke:#fff,stroke-width:0px,color:#000
class S spacewhite
classDef k8s fill:#326ce5,stroke:#fff,stroke-width:1px,color:#fff;
class A,L,C k8s
{{< /mermaid >}}

Figure 1. Pipeline Metrics Resource

اجزای معماری، از راست به چپ در شکل، شامل موارد زیر می‌شود:

* [cAdvisor](https://github.com/google/cadvisor): دیمونی برای جمع‌آوری، تجمیع و ارائه متریک‌های کانتینرهای مدیریت شده در Kubelet.
* [kubelet](/docs/concepts/overview/components/#kubelet): عامل نود برای مدیریت منابع کانتینرها. متریک‌های منابع از طریق نقاط پایانی `/metrics/resource` و `/stats` kubelet قابل دسترس هستند.
* [node level resource metrics](/docs/reference/instrumentation/node-metrics): API ارائه شده توسط kubelet برای کشف و بازیابی آمار خلاصه شده منابع موجود از طریق نقطه پایانی `/metrics/resource`.
* [metrics-server](#metrics-server): جزء یک افزونه خوشه که متریک‌های منابع جمع‌آوری و تجمیع می‌کند و به ازای هر kubelet ارائه می‌دهد. سرور API متریک را برای استفاده توسط HPA، VPA و دستور `kubectl top` ارائه می‌دهد. Metrics Server پیاده‌سازی مرجع Metrics API است.
* [Metrics API](#metrics-api): API Kubernetes که پشتیبانی از دسترسی به مصرف CPU و حافظه برای اسکیلینگ کاری دارد. برای این کار در خوشه خود نیاز به یک سرور توسعه API دارید که Metrics API را ارائه می‌دهد.

  {{< note >}}
  cAdvisor از خواندن متریک‌ها از cgroups پشتیبانی می‌کند که با مکانیسم‌های جداسازی منابع کانتینر معمول در Linux کار می‌کند. اگر از یک ران‌تایم کانتینر استفاده می‌کنید که از مکانیزم جداسازی منابع دیگری مانند مجازی‌سازی استفاده می‌کند، آن ران‌تایم کانتینر باید پشتیبانی از
  [CRI Container Metrics](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/cri-container-stats.md)
  را داشته باشد تا متریک‌ها برای kubelet قابل دسترس باشند.
  {{< /note >}}

<!-- body -->

## Metrics API
{{< feature-state for_k8s_version="1.8" state="beta" >}}

metrics-server Metrics API را پیاده‌سازی می‌کند. این API به شما اجازه می‌دهد تا مصرف CPU و حافظه برای نود‌ها و پاد‌ها در خوشه‌ی خود را دسترسی داشته باشید. نقش اصلی آن ارائه متریک‌های مصرف منابع به اجزای اتواسکیلر K8s است.

اینجا مثالی از درخواست Metrics API برای یک نود `minikube` است که از طریق `jq` برای خوانایی آسان‌تر نمایش داده شده است:

```shell
kubectl get --raw "/apis/metrics.k8s.io/v1beta1/n

odes/minikube" | jq '.'
```

این همان درخواست API با استفاده از `curl`:

```shell
curl http://localhost:8080/apis/metrics.k8s.io/v1beta1/nodes/minikube
```

پاسخ نمونه:

```json
{
  "kind": "NodeMetrics",
  "apiVersion": "metrics.k8s.io/v1beta1",
  "metadata": {
    "name": "minikube",
    "selfLink": "/apis/metrics.k8s.io/v1beta1/nodes/minikube",
    "creationTimestamp": "2022-01-27T18:48:43Z"
  },
  "timestamp": "2022-01-27T18:48:33Z",
  "window": "30s",
  "usage": {
    "cpu": "487558164n",
    "memory": "732212Ki"
  }
}
```

اینجا یک مثال از درخواست Metrics API برای پاد `kube-scheduler-minikube` که در فضای نام `kube-system` قرار دارد و از طریق `jq` برای خوانایی آسان‌تر نمایش داده شده است:

```shell
kubectl get --raw "/apis/metrics.k8s.io/v1beta1/namespaces/kube-system/pods/kube-scheduler-minikube" | jq '.'
```

همین درخواست API با استفاده از `curl`:

```shell
curl http://localhost:8080/apis/metrics.k8s.io/v1beta1/namespaces/kube-system/pods/kube-scheduler-minikube
```

پاسخ نمونه:

```json
{
  "kind": "PodMetrics",
  "apiVersion": "metrics.k8s.io/v1beta1",
  "metadata": {
    "name": "kube-scheduler-minikube",
    "namespace": "kube-system",
    "selfLink": "/apis/metrics.k8s.io/v1beta1/namespaces/kube-system/pods/kube-scheduler-minikube",
    "creationTimestamp": "2022-01-27T19:25:00Z"
  },
  "timestamp": "2022-01-27T19:24:31Z",
  "window": "30s",
  "containers": [
    {
      "name": "kube-scheduler",
      "usage": {
        "cpu": "9559630n",
        "memory": "22244Ki"
      }
    }
  ]
}
```

Metrics API در مخزن [k8s.io/metrics](https://github.com/kubernetes/metrics) تعریف شده است. برای استفاده از آن، شما باید لایه تجمیع API را فعال کرده و یک [APIService](/docs/reference/kubernetes-api/cluster-resources/api-service-v1/) را برای API `metrics.k8s.io` ثبت کنید.

برای دریافت اطلاعات بیشتر در مورد Metrics API، به [resource metrics API design](https://git.k8s.io/design-proposals-archive/instrumentation/resource-metrics-api.md)، [مخزن metrics-server](https://github.com/kubernetes-sigs/metrics-server) و [resource metrics API](https://github.com/kubernetes/metrics#resource-metrics-api) مراجعه کنید.

{{< note >}}
شما باید metrics-server یا آداپتور جایگزین مناسبی که Metrics API را ارائه می‌دهد را نصب کنید تا بتوانید به آن دسترسی پیدا کنید.
{{< /note >}}

## اندازه‌گیری مصرف منابع

### CPU

مصرف CPU به عنوان میانگین استفاده از هسته‌ها برحسب واحد CPU گزارش می‌شود. یک واحد CPU، در Kubernetes، معادل 1 vCPU/Core برای ارائه دهندگان ابری و 1 hyper-thread در پردازنده‌های Intel bare-metal است.

این مقدار از طریق گرفتن نرخ روی یک شمارنده CPU تجمیعی ارائه شده توسط هسته سیستم عامل (در هسته‌های Linux و Windows) به دست می‌آید. پنجره زمانی مورد استفاده برای محاسبه CPU در فیلد window در Metrics API نشان داده شده است.

برای دریافت اطلاعات بیشتر در مورد نحوهٔ اختصاص و اندازه‌گیری منابع CPU توسط Kubernetes، به [معنای CPU](/docs/concepts/configuration/manage-resources-containers/#meaning-of-cpu) مراجعه کنید.

### حافظه

مصرف حافظه به عنوان مجموعه کاری، به بایت، در لحظه‌ای که متریک جمع‌آوری شده است، گزارش می‌شود.

در یک جهان ایده‌آل، "مجموعه کاری" مقدار حافظه استفاده شده است که در زیر فشار حافظه نمی‌تواند آزاد شود. با این حال، محاسبه مجموعه کاری بستگی به سیستم عامل میزبان دارد و معمولاً از مصرف زیاد الگوریتم‌های ساخته شده استفاده می‌کند تا یک برآورد تولید کند.

مدل Kubernetes برای مجموعه کاری یک کانتینر انتظار دارد که ران‌تایم کانتینر حافظه بی‌نام مربوط به کانتینر مورد نظر را شمارش کند. معمولاً متریک مجموعه کاری شامل همچنین برخی حافظه‌های ذخیره شده (با پشتیبانی از فایل) است، زیرا سیستم عامل میزبان نمی‌تواند همیشه صفحات را بازیابی کند.

برای دریافت اطلاعات بیشتر در مورد نحوهٔ اختصاص و اندازه‌گیری منابع حافظه توسط Kubernetes، به [معنای حافظه](/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory) مراجعه کنید.

## Metrics Server

metrics-server متریک‌های منابع را از kubelet جمع‌آوری کرده و آن‌ها را در Kubernetes API server از طریق Metrics API برای استفاده از HPA و VPA ارائه می‌دهد. همچنین می‌توانید این متریک‌ها را با استفاده از دستور `kubectl top` مشاهده کنید.

metrics-server از API Kubernetes برای پیگیری نودها و پادها در خوشه‌ی شما استفاده می‌کند. metrics-server هر نود را از طریق HTTP برای جمع‌آوری متریک‌ها پرس‌وجو می‌کند. همچنین metrics-server یک نمای داخلی از اطلاعات متاپاد را ایجاد می‌کند و یک حافظه پنهان از سلامتی پاد را نگه می‌دارد. اطلاعات سلامتی پاد کش شده از طریق API توسعه‌ای است که metrics-server آن را در دسترس قرار می‌دهد.

به عنوان مثال با یک درخواست HPA، metrics-server باید شناسایی کند که کدام پادها برچسب‌های انتخاب‌گر در رویه را انجام می‌دهند.

metrics-server با استفاده از [kubelet](/docs/reference/command-line-tools-reference/kubelet/) API برای جمع‌آوری متریک‌ها از هر نود تماس می‌گیرد. بسته به نسخه metrics

-server که استفاده می‌کند:

* نقطه پایان منابع متریک `/metrics/resource` در نسخه v0.6.0+ یا
* نقطه پایان API خلاصه `/stats/summary` در نسخه‌های قدیمی‌تر

## {{% heading "whatsnext" %}}

برای دریافت اطلاعات بیشتر در مورد metrics-server، به [مخزن metrics-server](https://github.com/kubernetes-sigs/metrics-server) مراجعه کنید.

همچنین می‌توانید به موارد زیر مراجعه کنید:

* [طراحی metrics-server](https://git.k8s.io/design-proposals-archive/instrumentation/metrics-server.md)
* [پرسش‌های متداول metrics-server](https://github.com/kubernetes-sigs/metrics-server/blob/master/FAQ.md)
* [مشکلات شناخته‌شده metrics-server](https://github.com/kubernetes-sigs/metrics-server/blob/master/KNOWN_ISSUES.md)
* [انتشارهای metrics-server](https://github.com/kubernetes-sigs/metrics-server/releases)
* [اتوماسیون پرتو پایین پاد هافایز](/docs/tasks/run-application/horizontal-pod-autoscale/)

برای دریافت اطلاعات در مورد این که چگونه kubelet متریک‌های نود را سرویس می‌دهد و چگونه می‌توانید از آنها از طریق API Kubernetes دسترسی پیدا کنید، [داده‌های متریک‌های نود](/docs/reference/instrumentation/node-metrics) را بخوانید.