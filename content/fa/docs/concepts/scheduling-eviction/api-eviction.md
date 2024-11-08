---
title: "خروج از API شروع شده"
content_type: concept
weight: 110
---

{{< glossary_definition term_id="api-eviction" length="short" >}} </br>

می‌توانید با تماس به صورت مستقیم با API خروج را درخواست کنید یا به صورت برنامه‌ریزی‌شده
از طریق مشتری {{<glossary_tooltip term_id="kube-apiserver" text="سرور API">}} مانند دستور `kubectl drain`. این عمل یک شی `Eviction` ایجاد می‌کند
که باعث می‌شود سرور API پاد را خاتمه دهد.

خروج از API شروع شده احترام می‌گذارد به پیکربندی‌های شما [`بودجه‌های اختلال در پاد`](/docs/tasks/run-application/configure-pdb/)
و [`ثانیه‌های گریس خاتمه`](/docs/concepts/workloads/pods/pod-lifecycle#pod-termination).

برای ایجاد یک شی Eviction برای یک پاد، از API می‌توانید به شیوه زیر عمل کنید:

{{< tabs name="مثال Eviction" >}}
{{% tab name="policy/v1" %}}
{{< note >}}
`policy/v1` Eviction در نسخه v1.22+ قابل دسترسی است. از `policy/v1beta1` در نسخه‌های قبلی استفاده کنید.
{{< /note >}}

```json
{
  "apiVersion": "policy/v1",
  "kind": "Eviction",
  "metadata": {
    "name": "quux",
    "namespace": "default"
  }
}
```
{{% /tab %}}
{{% tab name="policy/v1beta1" %}}
{{< note >}}
در v1.22، قدیمی شده است و ترجیحاً از `policy/v1` استفاده کنید.
{{< /note >}}

```json
{
  "apiVersion": "policy/v1beta1",
  "kind": "Eviction",
  "metadata": {
    "name": "quux",
    "namespace": "default"
  }
}
```
{{% /tab %}}
{{< /tabs >}}

همچنین، می‌توانید با استفاده از `curl` یا `wget` به API دسترسی پیدا کنید و عملیات خروج را امتحان کنید، مشابه مثال زیر:

```bash
curl -v -H 'Content-type: application/json' https://your-cluster-api-endpoint.example/api/v1/namespaces/default/pods/quux/eviction -d @eviction.json
```

## چگونگی کار خروج از API شروع شده

زمانی که خروج را با استفاده از API درخواست می‌دهید، سرور API عملیات تأیید واگذاری را انجام می‌دهد و به یکی از روش‌های زیر پاسخ می‌دهد:

* `200 OK`: خروج مجاز است، زیرمجموعه `Eviction` ایجاد می‌شود و پاد حذف می‌شود، مشابه ارسال یک درخواست `DELETE` به URL پاد.
* `429 Too Many Requests`: خروج در حال حاضر مجاز نیست به دلیل {{<glossary_tooltip term_id="pod-disruption-budget" text="بودجه‌های اختلال در پاد">}} پیکربندی شده. ممکن است بتوانید بعداً دوباره خروج را امتحان کنید. همچنین ممکن است به دلیل محدودیت نرخ API این پاسخ را ببینید.
* `500 Internal Server Error`: خروج مجاز نیست زیرا پیکربندی اشتباه است، مانند اینکه بیش از یک بودجه‌ی اختلال در پاد به همان پاد اشاره کنند.

اگر پادی که می‌خواهید خارج شود جزء بارکاری نبود که دارای بودجه‌ای اختلال در پاد باشد، سرور API همیشه `200 OK` را برمی‌گرداند و خروج را اجازه می‌دهد.

اگر سرور API اجازه خروج را دهد، پاد به صورت زیر حذف می‌شود:

1. منبع `Pod` در سرور API با یک برچسب حذف به روزرسانی می‌شود، که پس از آن سرور API منبع `Pod` را به عنوان خاتمه در نظر می‌گیرد. همچنین منبع `Pod` با دوره‌ی گریس تعیین‌شده نشان داده می‌شود.
1. Kubelet در گره‌ای که پاد محلی در آن اجرا می‌شود متوجه می‌شود که منبع `Pod` برای خاتمه نشان داده شده است و آغاز به خوشه خوشه‌های کراسی داشته باشید. به علاوه، کنترل‌ها دیگر دیگر پاد را به عنوان یک شی معتبر در نظر نمی‌گیرد.
1. بعد از انقضای دوره گریس برای پاد، kubelet به صورت اجباری پاد محلی را خاتمه می‌دهد.
1. kubelet به سرور API می‌گوید که منبع `Pod` را حذف کند.
1. سرور API منبع `Pod` را حذف می‌کند.

## رفع مشکلات خروجی متعلقه به دام

در برخی موارد، برنامه‌های شما ممکن است وارد یک حالت خرابی شوند، جایی که API خروج فقط پاسخ 429 یا 500 را بازگردانده و نهایتاً تا زمانی که مداخله کنید می‌توانید آن‌ها را درمان کنید. این ممکن است در مواردی اتفاق بیافتد که به عنوان مثال ReplicaSet پاد‌هایی برای برنامه شما ایجاد می‌کند اما پاد‌های جدید به حالت `Ready` نمی‌رسند. این رفتار را ممکن است در مواردی مشاهده کنید که آخرین پادی که اخراج شد دارای دوره گریس خاتمه طولانی بود.

اگر توقف خروجی را مشاهده می‌کنید، یکی از راه‌حل‌های زیر را امتحان کنید:

* عملیات خودکار که موجب ایج

اد مشکل شده است را متوقف یا متوقف کنید. قبل از راه‌اندازی مجدد عملیات، مشکلات برنامه خود را بررسی کنید.
* یک مدتی صبر کنید، سپس پاد را مستقیماً از صفحه کنترل کلاستر خود به جای استفاده از API خروج حذف کنید.

## {{% heading "whatsnext" %}}

* یاد بگیرید که چگونه با [بودجه‌های اختلال در پاد](/docs/tasks/run-application/configure-pdb/) اپلیکیشن‌های خود را محافظت کنید.
* درباره [خروج از فشار نود](/docs/concepts/scheduling-eviction/node-pressure-eviction/) بیشتر بدانید.
* درباره [اولویت و پیش‌خوانی پاد](/docs/concepts/scheduling-eviction/pod-priority-preemption/) بیشتر بدانید.
