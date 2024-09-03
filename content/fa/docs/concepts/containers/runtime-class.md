---
reviewers:
  - tallclair
  - dchen1107
title: Runtime Class
content_type: concept
weight: 30
hide_summary: true # Listed separately in section index
---

<!-- overview -->

{{< feature-state for_k8s_version="v1.20" state="stable" >}}

این صفحه منبع RuntimeClass و مکانیزم انتخاب Runtime را شرح می‌دهد.

RuntimeClass یک ویژگی برای انتخاب پیکربندی اجرای کانتینر است. پیکربندی اجرای کانتینر برای اجرای کانتینرهای یک Pod استفاده می‌شود.

<!-- body -->

## دلایل

می‌توانید RuntimeClass متفاوتی را بین Podهای مختلف تنظیم کنید تا تعادلی بین عملکرد و امنیت فراهم کنید. به عنوان مثال، اگر بخشی از بارکاری شما نیاز به اطمینان بالای اطلاعاتی دارد، ممکن است انتخاب کنید تا این Podها را در یک زمان اجرای کانتینر اجرای کانتینرهای سخت‌افزاری کنید. سپس از جداسازی اضافی این زمان اجرای انتخاب شده استفاده کنید، با هزینه برخی از هزینه‌های اضافی.

همچنین می‌توانید از RuntimeClass برای اجرای Podهای مختلف با همان زمان اجرای کانتینر، اما با تنظیمات متفاوت استفاده کنید.

## راه‌اندازی

1. پیاده‌سازی CRI بر روی گره‌ها (وابسته به زمان اجرای پیاده‌سازی)
2. ایجاد منابع RuntimeClass متناظر

### 1. پیاده‌سازی CRI بر روی گره‌ها

پیکربندی‌های موجود از طریق RuntimeClass وابسته به پیاده‌سازی Container Runtime Interface (CRI) است. برای مشاهده مستندات متناظر ([پایین](#config-cri)) برای پیاده‌سازی CRI خود را بررسی کنید.

{{< note >}}
RuntimeClass به طور پیش‌فرض فرض می‌کند که پیکربندی یکنواخت گره در سراسر خوشه است (که به این معنی است که همه گره‌ها به همان روشی با توجه به اجرای کانتینرها پیکربندی شده‌اند). برای پشتیبانی از پیکربندی‌های گره مختلف، [زمان بندی](#scheduling) را بررسی کنید.
{{< /note >}}

پیکربندی‌ها نام `handler` متناظر دارند، که توسط RuntimeClass ارجاع می‌شود. `handler` باید نام معتبری باشد [برچسب DNS](/docs/concepts/overview/working-with-objects/names/#dns-label-names).

### 2. ایجاد منابع RuntimeClass متناظر

پیکربندی‌های ایجاد شده در مرحله 1 هر کدام باید نام `handler` متناظری داشته باشند که پیکربندی را مشخص می‌کند. برای هر `handler`، یک شی RuntimeClass متناظر ایجاد کنید.

منبع RuntimeClass فعلی فقط دو فیلد معنی‌دار دارد: نام RuntimeClass (`metadata.name`) و `handler` (`handler`). تعریف شی مانند این است:

```yaml
# RuntimeClass در گروه API node.k8s.io تعریف شده است
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  # نامی که توسط RuntimeClass ارجاع می‌شود.
  # RuntimeClass یک منبع خارج از فضای نام است.
  name: myclass 
# نام پیکربندی CRI متناظر
handler: myconfiguration 
```

نام یک شی RuntimeClass باید یک [نام زیر دامنه DNS](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names) معتبر باشد.

{{< note >}}
توصیه می‌شود که عملیات نوشتن RuntimeClass (ایجاد/به‌روزرسانی/رفع) توسط مدیر خوشه محدود شود. این به طور معمول پیش‌فرض است. برای جزئیات بیشتر، [مرور اختیارات](/docs/reference/access-authn-authz/authorization/) را ببینید.
{{< /note >}}

## استفاده

هنگامی که RuntimeClasses برای خوشه پیکربندی می‌شود، می‌توانید در مشخصه Pod یک `runtimeClassName` مشخص کنید تا از آن استفاده کنید. به عنوان مثال:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  runtimeClassName: myclass
  # ...
```

این دستور به kubelet این دستور می‌دهد که از RuntimeClass معین شده برای اجرای این Pod استفاده کند. اگر RuntimeClass معین شده وجود نداشته باشد یا CRI نتواند handler متناظر را اجرا کند، پاد به مرحله `Failed` ورودی [phase](/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase) وارد می‌شود. جستجو برای یک [event](/docs/tasks/debug/debug-application/debug-running-pod/) متناظر برای یک پیام خطا.

اگر `runtimeClassName` مشخص نشود، RuntimeHandler پیش‌فرض استفاده می‌شود که معادل رفتار زمانی است که ویژگی RuntimeClass غیرفعال است.

### پیکربندی CRI

برای جزئیات بیشتر در مورد تنظیمات runtime CRI، به [نصب CRI](/docs/setup/production-environment/container-runtimes/) مراجعه کنید.

#### {{< glossary_tooltip term_id="containerd" >}}

هندلر runtime از طریق تنظیم containerd در `/etc/containerd/config.toml` پیکربندی می‌شود. هندلر‌های معتبر در زیر بخش‌های runtimes پیکربندی می‌شوند:

```
[plugins."io.containerd.grpc.v1.cri".containerd.runt

imes.${HANDLER_NAME}]
```

برای جزئیات بیشتر به [مستندات پیکربندی containerd](https://github.com/containerd/containerd/blob/main/docs/cri/config.md) مراجعه کنید:

#### {{< glossary_tooltip term_id="cri-o" >}}

هندلر runtime از طریق پیکربندی CRI-O در `/etc/crio/crio.conf` پیکربندی می‌شود. هندلر‌های معتبر در زیر
[table crio.runtime](https://github.com/cri-o/cri-o/blob/master/docs/crio.conf.5.md#crioruntime-table):

```
[crio.runtime.runtimes.${HANDLER_NAME}]
  runtime_path = "${PATH_TO_BINARY}"
```

برای جزئیات بیشتر به [مستندات پیکربندی CRI-O](https://github.com/cri-o/cri-o/blob/master/docs/crio.conf.5.md) مراجعه کنید.

## زمان‌بندی

{{< feature-state for_k8s_version="v1.16" state="beta" >}}

با تعیین فیلد `scheduling` برای یک RuntimeClass، می‌توانید محدودیت‌ها را تنظیم کنید تا اطمینان حاصل شود که Podهایی که با این RuntimeClass اجرا می‌شوند، بر روی گره‌هایی که از آن پشتیبانی می‌کنند، برنامه‌ریزی می‌شوند. اگر `scheduling` تنظیم نشده باشد، این RuntimeClass فرض می‌شود که توسط تمام گره‌ها پشتیبانی می‌شود.

برای اطمینان از این که Podها بر روی گره‌هایی که از یک RuntimeClass خاص پشتیبانی می‌کنند، نشسته‌اند، می‌توانید یک برچسب مشترک برای این مجموعه گره تنظیم کنید که سپس توسط `runtimeclass.scheduling.nodeSelector` RuntimeClass انتخاب شده است. NodeSelector RuntimeClass با NodeSelector پاد ترکیب می‌شود در تأیید، عملیات انتخاب نودهایی که توسط هرکدام انتخاب شده‌اند، موثر است. اگر یک تضاد وجود داشته باشد، پاد رد خواهد شد.

اگر گره‌های پشتیبانی‌کننده به منظور جلوگیری از اجرای Podهای RuntimeClass دیگر روی گره توسعه داده شوند، می‌توانید `tolerations` را به RuntimeClass اضافه کنید. همچنان با `nodeSelector`، tolerations RuntimeClass با tolerations پاد در تأیید، ائتلاف نودهایی که توسط هر کدام تحمل می‌شوند، موثر است.

برای اطلاعات بیشتر در مورد پیکربندی انتخاب گره و tolerations، به [اختصاص Podها به گره‌ها](/docs/concepts/scheduling-eviction/assign-pod-node/) مراجعه کنید.

### هزینه Pod

{{< feature-state for_k8s_version="v1.24" state="stable" >}}

می‌توانید منابع _هزینه_ را که با اجرای یک Pod مرتبط هستند مشخص کنید. اظهار هزینه امکان دارد خوشه (شامل برنامه‌ریز) این هنگام که تصمیمات در مورد Podها و منابع گرفته می‌شود را محاسبه کند.

هزینه Pod از طریق فیلد `overhead` در RuntimeClass تعریف می‌شود. با استفاده از این فیلد، می‌توانید هزینه اجرای Podهایی که از این RuntimeClass استفاده می‌کنند را مشخص کنید و اطمینان حاصل کنید که این هزینه‌ها در Kubernetes محاسبه شده‌اند.

## {{% heading "whatsnext" %}}

- [طراحی RuntimeClass](https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/585-runtime-class/README.md)
- [طراحی زمان‌بندی RuntimeClass](https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/585-runtime-class/README.md#runtimeclass-scheduling)
- درباره مفهوم [هزینه Pod](/docs/concepts/scheduling-eviction/pod-overhead/) بیشتر بخوانید
- [طراحی ویژگی PodOverhead](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/688-pod-overhead)
