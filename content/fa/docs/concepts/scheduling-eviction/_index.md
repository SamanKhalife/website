---
title: "برنامه‌ریزی، پیش‌خوانی و خروج از میزبان"
weight: 95
content_type: concept
description: >
  در Kubernetes، برنامه‌ریزی به معنای اطمینان از این است که پاد‌ها با گره‌ها هماهنگ شوند
  تا kubelet بتواند آن‌ها را اجرا کند. پیش‌خوانی فرایندی است که در آن پادهای با اولویت پایین‌تر
  متوقف می‌شوند تا پادهای با اولویت بالاتر بتوانند بر روی گره‌ها برنامه‌ریزی شوند.
  خروج از میزبان همان فرایندی است که در آن یک یا چند پاد به صورت پیش‌فرض بر روی گره‌های
  محدود منابع خاتمه می‌یابند.
no_list: true
---

در Kubernetes، برنامه‌ریزی به معنای اطمینان از این است که {{<glossary_tooltip text="پاد" term_id="pod">}}
ها با {{<glossary_tooltip text="گره" term_id="node">}} ها هماهنگ شوند تا
{{<glossary_tooltip text="kubelet" term_id="kubelet">}} بتواند آن‌ها را اجرا کند. پیش‌خوانی
فرایندی است که در آن پادهای با اولویت پایین‌تر را متوقف می‌کند تا پادهای با اولویت بالاتر
بتوانند بر روی گره‌ها برنامه‌ریزی شوند. خروج از میزبان همان فرایندی است که در آن یک یا
چند پاد به صورت پیش‌فرض بر روی گره‌های منابع کمبودی خاتمه می‌یابند.

## برنامه‌ریزی

* [برنامه‌ریز Kubernetes](/docs/concepts/scheduling-eviction/kube-scheduler/)
* [اختصاص پاد به گره‌ها](/docs/concepts/scheduling-eviction/assign-pod-node/)
* [بیش‌برداشت پاد](/docs/concepts/scheduling-eviction/pod-overhead/)
* [محدودیت‌های توپولوژی گسترش پاد](/docs/concepts/scheduling-eviction/topology-spread-constraints/)
* [توزیع Taints و Tolerations](/docs/concepts/scheduling-eviction/taint-and-toleration/)
* [چارچوب برنامه‌ریزی](/docs/concepts/scheduling-eviction/scheduling-framework)
* [تخصیص پویای منابع](/docs/concepts/scheduling-eviction/dynamic-resource-allocation)
* [تنظیم عملکرد برنامه‌ریز](/docs/concepts/scheduling-eviction/scheduler-perf-tuning/)
* [بسته‌بندی منابع برای منابع گسترده](/docs/concepts/scheduling-eviction/resource-bin-packing/)
* [آمادگی برنامه‌ریزی پاد](/docs/concepts/scheduling-eviction/pod-scheduling-readiness/)
* [Descheduler](https://github.com/kubernetes-sigs/descheduler#descheduler-for-kubernetes)

## قطعیت پاد

{{<glossary_definition term_id="pod-disruption" length="all">}}

* [اولویت و پیش‌خوانی پاد](/docs/concepts/scheduling-eviction/pod-priority-preemption/)
* [خروج از میزبان فشار نود](/docs/concepts/scheduling-eviction/node-pressure-eviction/)
* [خروج از API شروع شده](/docs/concepts/scheduling-eviction/api-eviction/)
