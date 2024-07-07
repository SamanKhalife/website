---
reviewers:
- bprashanth
- enisoc
- erictune
- foxish
- janetkuo
- kow3ns
- smarterclayton
title: مقیاس‌پذیر کردن یک StatefulSet
content_type: وظیفه
weight: 50
---

<!-- مرور -->

این وظیفه نحوه مقیاس‌پذیر کردن یک StatefulSet را نمایش می‌دهد. مقیاس‌پذیر کردن یک StatefulSet به افزایش یا کاهش تعداد نمونه‌ها اشاره دارد.

## {{% heading "پیش‌نیازها" %}}

- StatefulSets فقط در نسخه‌های Kubernetes 1.5 به بالا موجود هستند. برای بررسی نسخه Kubernetes خود، دستور `kubectl version` را اجرا کنید.

- همه برنامه‌های stateful به خوبی مقیاس‌پذیر نیستند. اگر مطمئن نیستید که آیا باید StatefulSet خود را مقیاس‌پذیر کنید یا خیر، به [مفاهیم StatefulSet](/docs/concepts/workloads/controllers/statefulset/) یا [آموزش StatefulSet](/docs/tutorials/stateful-application/basic-stateful-set/) مراجعه کنید.

- شما باید مقیاس‌پذیری را فقط زمانی انجام دهید که اطمینان حاصل کنید که خوشه برنامه stateful شما کاملاً سالم است.

<!-- مراحل -->

## مقیاس‌پذیر کردن StatefulSets

### استفاده از kubectl برای مقیاس‌پذیر کردن StatefulSets

ابتدا StatefulSetی که می‌خواهید مقیاس‌پذیر کنید را پیدا کنید.

```shell
kubectl get statefulsets <نام-stateful-set>
```

تعداد نمونه‌های StatefulSet خود را تغییر دهید:

```shell
kubectl scale statefulsets <نام-stateful-set> --replicas=<تعداد-جدید-نمونه‌ها>
```

### انجام به‌روزرسانی‌های در محل بر روی StatefulSets شما

در صورت لزوم، می‌توانید به [به‌روزرسانی‌های در محل](/docs/concepts/cluster-administration/manage-deployment/#in-place-updates-of-resources) بر روی StatefulSets خود انجام دهید.

اگر StatefulSet شما ابتدا با `kubectl apply` ایجاد شده بود، `.spec.replicas` از اظهارنامه‌های StatefulSet را به‌روز کنید، سپس `kubectl apply` را اجرا کنید:

```shell
kubectl apply -f <فایل-stateful-set-به‌روزرسانی-شده>
```

در غیر این صورت، آن فیلد را با `kubectl edit` ویرایش کنید:

```shell
kubectl edit statefulsets <نام-stateful-set>
```

یا از `kubectl patch` استفاده کنید:

```shell
kubectl patch statefulsets <نام-stateful-set> -p '{"spec":{"replicas":<تعداد-جدید-نمونه‌ها>}}'
```

## رفع مشکلات

### مقیاس‌پذیر کردن به درستی کار نمی‌کند

شما نمی‌توانید یک StatefulSet را کاهش دهید زمانی که هر یک از پادهای stateful که آن را مدیریت می‌کند، ناسالم است. مقیاس‌پذیر کردن تنها پس از آن انجام می‌شود که این پادهای stateful به حالت Running و Ready برسند.

اگر spec.replicas > 1 باشد، Kubernetes نمی‌تواند دلیل عدم سالم بودن یک پاد را تشخیص دهد. این ممکن است نتیجه یک خطای دائمی یا خطای عابر باشد. خطای عابر ممکن است ناشی از یک راه‌اندازی مجددی باشد که برای ارتقا یا نگهداری لازم است.

اگر پاد به دلیل خطای دائمی ناسالم است، مقیاس‌پذیر کردن بدون اصلاح خطا ممکن است منجر به وضعیتی شود که عضویت StatefulSet زیر حداقلی از تعداد نمونه‌های لازم برای عملکرد صحیح شود. این می‌تواند منجر به عدم دسترسی به StatefulSet شما شود.

اگر پاد به دلیل یک خطای عابر ناسالم است و احتمال دارد که پاد مجدداً قابل دسترس شود، خطای گذرا ممکن است با عملیات مقیاس‌پذیر کردن شما مداخله کند. برخی از پایگاه‌داده‌های توزیع‌شده مشکلاتی دارند زمانی که گره‌ها همزمان به عنوان عضویت پیوسته و ترک می‌کنند. در این موارد، بهتر است که عملیات مقیاس‌پذیر کردن را در سطح برنامه بررسی کنید و فقط زمانی انجام دهید که اطمینان حاصل کنید که خوشه برنامه stateful شما کاملاً سالم است.

## {{% heading "چی بعد" %}}

- بیشتر در مورد [حذف یک StatefulSet](/docs/tasks/run-application/delete-stateful-set/) بیاموزید.
