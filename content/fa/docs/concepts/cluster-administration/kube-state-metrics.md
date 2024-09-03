---
title: معیارهای وضعیت اشیاء Kubernetes
content_type: مفهوم
weight: 75
description: >-
   kube-state-metrics، یک نماینده افزودنی برای تولید و ارائه معیارهای سطح خوشه.
---

وضعیت اشیاء Kubernetes در API Kubernetes می‌تواند به عنوان معیارها ارائه شود. یک نماینده افزودنی به نام [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics) می‌تواند به سرور API Kubernetes متصل شود و یک نقطه پایان HTTP را با معیارهای تولید شده از وضعیت اشیاء فردی در خوشه ارائه دهد. این نماینده اطلاعات مختلفی را درباره وضعیت اشیاء مانند برچسب‌ها و حاشیه‌نویسی‌ها، زمان‌های شروع و پایان، وضعیت یا مرحله که اشیاء در حال حاضر در آن هستند، ارائه می‌دهد. به عنوان مثال، ظروفی که در پاد‌ها اجرا می‌شوند، معیار `kube_pod_container_info` را ایجاد می‌کنند. این شامل نام ظرف، نام پادی که قسمت آن است، {{< glossary_tooltip term_id="namespace" text="فضای‌نام" >}} که پاد در آن اجرا می‌شود، نام تصویر ظرف، شناسه تصویر، نام تصویر از مشخصه ظرف، شناسه ظرف اجرایی و شناسه پاد به عنوان برچسب‌ها است.

{{% thirdparty-content single="true" %}}

یک عنصر خارجی که قادر به جمع‌آوری اطلاعات نقطه پایان kube-state-metrics است (به عنوان مثال از طریق Prometheus) می‌تواند برای فعال کردن موارد استفاده زیر استفاده شود.

## مثال: استفاده از معیارهای kube-state-metrics برای پرس و جوی وضعیت خوشه {#example-kube-state-metrics-query-1}

سری‌های معیاری توسط kube-state-metrics مفید هستند تا در فهم بهتری از وضعیت خوشه کمک کنند، زیرا می‌توانند برای پرس و جو استفاده شوند.

اگر از Prometheus یا ابزار دیگری استفاده کنید که از همان زبان پرس و جو استفاده می‌کند، پرس و جوی PromQL زیر تعداد پادهایی را که آماده نیستند برمی‌گرداند:

```
count(kube_pod_status_ready{condition="false"}) by (namespace, pod)
```

## مثال: هشداردهی براساس معیارهای kube-state-metrics {#example-kube-state-metrics-alert-1}

معیارهای تولید شده از kube-state-metrics همچنین اجازه هشداردهی در مورد مشکلات در خوشه را می‌دهند.

اگر از Prometheus یا ابزار مشابهی استفاده کنید که از همان زبان قانون هشدار استفاده می‌کند، هشدار زیر در صورت وجود پادهایی که برای بیش از 5 دقیقه در وضعیت `Terminating` بوده‌اند، فعال می‌شود:

```yaml
groups:
- name: وضعیت پاد
  rules:
  - alert: PodsBlockedInTerminatingState
    expr: count(kube_pod_deletion_timestamp) by (namespace, pod) * count(kube_pod_status_reason{reason="NodeLost"} == 0) by (namespace, pod) > 0
    for: 5m
    labels:
      severity: page
    annotations:
      summary: پاد {{$labels.namespace}}/{{$labels.pod}} در وضعیت Terminating مسدود شده است.
```
