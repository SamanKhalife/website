---
reviewers:
- bprashanth
- erictune
- foxish
- janetkuo
- smarterclayton
title: حذف یک StatefulSet
content_type: task
weight: 60
---

<!-- overview -->

این تسک به شما نشان می‌دهد که چگونه یک {{< glossary_tooltip term_id="StatefulSet" >}} را حذف کنید.

## {{% heading "prerequisites" %}}

- این تسک فرض می‌کند که شما یک برنامه را در کلاستر خود دارید که با یک StatefulSet نشان داده می‌شود.

<!-- steps -->

## حذف یک StatefulSet

می‌توانید یک StatefulSet را به همان روشی که سایر منابع در کوبرنتیز حذف می‌کنید، حذف کنید:
از دستور `kubectl delete` استفاده کنید و StatefulSet را یا با فایل یا با نام مشخص کنید.

```shell
kubectl delete -f <file.yaml>
```

```shell
kubectl delete statefulsets <statefulset-name>
```

ممکن است نیاز داشته باشید که سرویس headless مرتبط را به صورت جداگانه بعد از حذف خود StatefulSet حذف کنید.

```shell
kubectl delete service <service-name>
```

هنگام حذف یک StatefulSet از طریق `kubectl`، StatefulSet به 0 مقیاس می‌شود.
همه پادهایی که بخشی از این کار هستند نیز حذف می‌شوند. اگر می‌خواهید فقط
StatefulSet را حذف کنید و نه پادها را، از `--cascade=orphan` استفاده کنید. برای مثال:

```shell
kubectl delete -f <file.yaml> --cascade=orphan
```

با استفاده از `--cascade=orphan` به `kubectl delete`، پادهای مدیریت شده توسط StatefulSet
حتی پس از حذف شیء StatefulSet باقی می‌مانند. اگر پادها دارای لیبل `app.kubernetes.io/name=MyApp` باشند، می‌توانید سپس آن‌ها را به این صورت حذف کنید:

```shell
kubectl delete pods -l app.kubernetes.io/name=MyApp
```

### دیسک‌های پایدار (Persistent Volumes)

حذف پادها در یک StatefulSet دیسک‌های مرتبط را حذف نمی‌کند.
این برای اطمینان از این است که شما فرصت دارید داده‌ها را از دیسک کپی کنید قبل از
حذف آن. حذف PVC بعد از اتمام پادها ممکن است باعث حذف دیسک‌های پایدار پشتیبان شود بسته به کلاس ذخیره‌سازی
و سیاست بازپس‌گیری. شما هرگز نباید توانایی دسترسی به یک دیسک بعد از حذف claim را فرض کنید.

{{< note >}}
هنگام حذف یک PVC احتیاط کنید، زیرا ممکن است منجر به از دست رفتن داده‌ها شود.
{{< /note >}}

### حذف کامل یک StatefulSet

برای حذف کامل همه چیز در یک StatefulSet، از جمله پادهای مرتبط،
می‌توانید یک سری از دستورات مشابه زیر را اجرا کنید:

```shell
grace=$(kubectl get pods <stateful-set-pod> --template '{{.spec.terminationGracePeriodSeconds}}')
kubectl delete statefulset -l app.kubernetes.io/name=MyApp
sleep $grace
kubectl delete pvc -l app.kubernetes.io/name=MyApp
```

در مثال بالا، پادها دارای لیبل `app.kubernetes.io/name=MyApp` هستند؛
برچسب خود را به صورت مناسب جایگزین کنید.

### حذف اجباری پادهای StatefulSet

اگر متوجه شدید که برخی از پادهای StatefulSet شما در حالت 'Terminating'
یا 'Unknown' برای مدت زمان طولانی گیر کرده‌اند، ممکن است نیاز داشته باشید به صورت دستی
برای حذف اجباری پادها از apiserver مداخله کنید.
این یک کار بالقوه خطرناک است. به
[حذف اجباری پادهای StatefulSet](/docs/tasks/run-application/force-delete-stateful-set-pod/)
برای جزئیات بیشتر مراجعه کنید.

## {{% heading "whatsnext" %}}

بیشتر در مورد [حذف اجباری پادهای StatefulSet](/docs/tasks/run-application/force-delete-stateful-set-pod/) یاد بگیرید.
