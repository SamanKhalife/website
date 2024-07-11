---
title: آمادگی برنامه‌ریزی پادها
content_type: مفهوم
weight: 40
---

<!-- خلاصه -->

{{< feature-state for_k8s_version="v1.30" state="stable" >}}

پادها بعد از ایجاد آماده برای برنامه‌ریزی در نظر گرفته می‌شوند. برنامه‌ریز Kubernetes وظیفه دارد که برای قرار دادن پاد‌های در انتظار، به دنبال گره‌های مناسب باشد. با این حال، در شرایط واقعی، برخی از پادها ممکن است به مدت طولانی در وضعیت "miss-essential-resources" باقی بمانند که این موجب می‌شود برنامه‌ریز (و ابزارهای ادغامی مانند Cluster AutoScaler) به طور ناچار در حالی که نیازی نیست، فعالیت کنند.

با تعیین/حذف فیلد `.spec.schedulingGates` یک پاد، شما می‌توانید کنترل کنید که زمانی که یک پاد برای برنامه‌ریزی در نظر گرفته می‌شود، چه زمانی آماده باشد.

<!-- محتوا -->

## تنظیم دروازه‌های برنامه‌ریزی پاد

فیلد `schedulingGates` حاوی یک لیست رشته‌ای است و هر رشته به عنوان یک معیار در نظر گرفته می‌شود که پاد باید قبل از اینکه قابل برنامه‌ریزی در نظر گرفته شود، برآورده کند. این فیلد فقط زمانی می‌تواند در زمان ایجاد پاد مقداردهی اولیه شود (توسط مشتری یا به طور تغییریافته در زمان تأیید). پس از ایجاد، هر دروازه برنامه‌ریزی به ترتیب دلخواه حذف می‌شود، اما اضافه کردن دروازه برنامه‌ریزی جدید ممنوع است.

{{< figure src="/docs/images/podSchedulingGates.svg" alt="نمودار دروازه‌های برنامه‌ریزی پاد" caption="شکل. دروازه‌های برنامه‌ریزی پاد" class="diagram-large" link="https://mermaid.live/edit#pako:eNplkktTwyAUhf8KgzuHWpukaYszutGlK3caFxQuCVMCGSDVTKf_XfKyPlhxz4HDB9wT5lYAptgHFuBRsdKxenFMClMYFIdfUdRYgbiD6ItJTEbR8wpEq5UpUfnDTf-5cbPoJjcbXdcaE61RVJIiqJvQ_Y30D-OCt-t3tFjcR5wZayiVnIGmkv4NiEfX9jijKTmmRH5jf0sRugOP0HyHUc1m6KGMFP27cM28fwSJDluPpNKaXqVJzmFNfHD2APRKSjnNFx9KhIpmzSfhVls3eHdTRrwG8QnxKfEZUUNeYTDBNbiaKRF_5dSfX-BQQQ0FpnEqQLJWhwIX5hyXsjbYl85wTINrgeC2EZd_xFQy7b_VJ6GCdd-itkxALE84dE3fAqXyIUZya6Qqe711OspVCI2ny2Vv35QqVO3-htt66ZWomAvVcZcv8yTfsiSFfJOydZoKvl_ttjLJVlJsblcJw-czwQ0zr9ZeqGDgeR77b2jD8xdtjtDn" >}}

## مثال استفاده

برای نشان دادن یک پاد غیرآماده برای برنامه‌ریزی، می‌توانید آن را با یک یا چند دروازه برنامه‌ریزی مشابه زیر ایجاد کنید:

{{% code_sample file="pods/pod-with-scheduling-gates.yaml" %}}

بعد از ایجاد پاد، می‌توانید وضعیت آن را با استفاده از دستور زیر بررسی کنید:

```bash
kubectl get pod test-pod
```

خروجی نشان می‌دهد که در وضعیت `SchedulingGated` قرار دارد:

```none
NAME       READY   STATUS            RESTARTS   AGE
test-pod   0/1     SchedulingGated   0          7s
```

همچنین می‌توانید فیلد `schedulingGates` آن را با اجرای دستور زیر بررسی کنید:

```bash
kubectl get pod test-pod -o jsonpath='{.spec.schedulingGates}'
```

خروجی:

```none
[{"name":"example.com/foo"},{"name":"example.com/bar"}]
```

برای اطلاع دادن به برنامه‌ریز که این پاد آماده برای برنامه‌ریزی است، می‌توانید دروازه‌های برنامه‌ریزی آن را به طور کامل حذف کنید با اعمال یک مندرجه اصلاح شده:

{{% code_sample file="pods/pod-without-scheduling-gates.yaml" %}}

می‌توانید با اجرای دستور زیر بررسی کنید که آیا `schedulingGates` پاک شده است یا خیر:

```bash
kubectl get pod test-pod -o jsonpath='{.spec.schedulingGates}'
```

خروجی مورد انتظار خالی است. و می‌توانید وضعیت جدید آن را با اجرای دستور زیر بررسی کنید:

```bash
kubectl get pod test-pod -o wide
```

از آنجا که پاد آزمایشی هیچ منبع CPU/حافظه‌ای درخواست نکرده است، منتظر است که وضعیت این پاد از `SchedulingGated` به `Running` منتقل شود:

```none
NAME       READY   STATUS    RESTARTS   AGE   IP         NODE
test-pod   1/1     Running   0          15s   10.0.0.4   node-2
```

## قابلیت مشاهده

متریک `scheduler_pending_pods` با برچسب جدید `"gated"` ارائه می‌شود تا تفاوت بین اینکه آیا یک پاد سعی شده است برنامه‌ریزی شود و امکان نداشت

ه است یا به طور صریح بعنوان آماده برای برنامه‌ریزی مشخص شود. می‌توانید از `scheduler_pending_pods{queue="gated"}` برای بررسی نتایج متریک استفاده کنید.

## دستورالعمل‌های برنامه‌ریزی قابل تغییر پاد

می‌توانید دستورالعمل‌های برنامه‌ریزی پاد را در حالتی که دروازه‌های برنامه‌ریزی دارند، تغییر دهید با محدودیت‌های خاص. به طور کلی، شما تنها می‌توانید دستورالعمل‌های برنامه‌ریزی یک پاد را تنگ‌تر کنید. به عبارت دیگر، دستورالعمل‌های به روزرسانی شده باعث می‌شود که پادها فقط بتوانند بر روی زیرمجموعه‌ای از گره‌ها که قبلاً مطابقت داشتند، برنامه‌ریزی شوند. به طور مشخص‌تر، قوانین به‌روزرسانی دستورالعمل‌های برنامه‌ریزی یک پاد به شرح زیر است:

1. برای `.spec.nodeSelector`، تنها افزودن مجاز است. اگر وجود نداشته باشد، اجازه تنظیم آن داده می‌شود.

2. برای `spec.affinity.nodeAffinity`، اگر nil باشد، آنگاه هر تنظیمی مجاز است.

3. اگر `NodeSelectorTerms` خالی بود، اجازه تنظیم آن داده می‌شود. اگر خالی نبود، تنها افزودن `NodeSelectorRequirements` به `matchExpressions` یا `fieldExpressions` مجاز است و هیچ تغییری در `matchExpressions` و `fieldExpressions` موجود مجاز نیست. این به این دلیل است که شرایط در `.requiredDuringSchedulingIgnoredDuringExecution.NodeSelectorTerms`، OR شده‌اند در حالی که عبارات در `nodeSelectorTerms[].matchExpressions` و `nodeSelectorTerms[].fieldExpressions` AND شده‌اند.

4. برای `.preferredDuringSchedulingIgnoredDuringExecution`، همه به‌روزرسانی‌ها مجاز هستند. این به این دلیل است که شرایط ترجیحی اختیاری هستند و بنابراین کنترل‌گرهای سیاستی آنها را تأیید نمی‌کنند.

## {{% heading "whatsnext" %}}

* مطالعه [KEP آمادگی برنامه‌ریزی پاد](https://github.com/kubernetes/enhancements/blob/master/keps/sig-scheduling/3521-pod-scheduling-readiness) برای اطلاعات بیشتر
