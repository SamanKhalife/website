---
title: دسترسی به API کوبرنتیز از داخل یک پاد
content_type: task
weight: 120
---

<!-- overview -->

این راهنما نحوه دسترسی به API کوبرنتیز از داخل یک پاد را نشان می‌دهد.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}}

<!-- steps -->

## دسترسی به API از داخل یک پاد

هنگام دسترسی به API از داخل یک پاد، مکان‌یابی و احراز هویت به سرور API کمی متفاوت از حالت کلاینت خارجی است.

آسان‌ترین راه برای استفاده از API کوبرنتیز از داخل یک پاد استفاده از یکی از [کتابخانه‌های کلاینت رسمی](/docs/reference/using-api/client-libraries/) است. این کتابخانه‌ها می‌توانند به صورت خودکار سرور API را کشف و احراز هویت کنند.

### استفاده از کتابخانه‌های کلاینت رسمی

از داخل یک پاد، روش‌های توصیه شده برای اتصال به API کوبرنتیز عبارتند از:

- برای کلاینت Go، از
  [کتابخانه کلاینت رسمی Go](https://github.com/kubernetes/client-go/)
  استفاده کنید.
  تابع `rest.InClusterConfig()` به طور خودکار کشف سرور API و احراز هویت را انجام می‌دهد.
  [یک مثال را اینجا ببینید](https://git.k8s.io/client-go/examples/in-cluster-client-configuration/main.go).

- برای کلاینت Python، از
  [کتابخانه کلاینت رسمی Python](https://github.com/kubernetes-client/python/)
  استفاده کنید.
  تابع `config.load_incluster_config()` به طور خودکار کشف سرور API و احراز هویت را انجام می‌دهد.
  [یک مثال را اینجا ببینید](https://github.com/kubernetes-client/python/blob/master/examples/in_cluster_config.py).

- کتابخانه‌های دیگری نیز موجود هستند، لطفاً به صفحه
  [کتابخانه‌های کلاینت](/docs/reference/using-api/client-libraries/)
  مراجعه کنید.

در هر صورت، از اطلاعات حساب سرویس پاد برای ارتباط امن با سرور API استفاده می‌شود.

### دسترسی مستقیم به API REST

در حالی که در یک پاد اجرا می‌شوید، کانتینر شما می‌تواند یک URL HTTPS برای سرور API کوبرنتیز ایجاد کند با بازیابی متغیرهای محیطی `KUBERNETES_SERVICE_HOST` و `KUBERNETES_SERVICE_PORT_HTTPS`. آدرس درون خوشه سرور API نیز به یک سرویس به نام `kubernetes` در فضای نام `default` منتشر شده است تا پادها بتوانند `kubernetes.default.svc` را به عنوان یک نام DNS برای سرور API محلی مرجع کنند.

{{< note >}}
کوبرنتیز تضمین نمی‌کند که سرور API یک گواهی معتبر برای نام میزبان `kubernetes.default.svc` داشته باشد؛
اما، انتظار می‌رود که پلن کنترل یک گواهی معتبر برای نام میزبان یا آدرسی که `$KUBERNETES_SERVICE_HOST` نشان می‌دهد ارائه دهد.
{{< /note >}}

راه توصیه شده برای احراز هویت به سرور API استفاده از
[اطلاعات حساب سرویس](/docs/tasks/configure-pod-container/configure-service-account/)
است. به طور پیش فرض، یک پاد با یک حساب سرویس مرتبط است، و یک اعتبارنامه (توکن) برای آن حساب سرویس در سیستم فایل هر کانتینر در آن پاد در مسیر `/var/run/secrets/kubernetes.io/serviceaccount/token` قرار داده شده است.

در صورت موجود بودن، یک بسته گواهی در سیستم فایل هر کانتینر در مسیر `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt` قرار داده شده است و باید برای اعتبارسنجی گواهی سرور API استفاده شود.

در نهایت، فضای نام پیش فرض برای عملیات API نام‌دار در فایلی در مسیر `/var/run/secrets/kubernetes.io/serviceaccount/namespace` در هر کانتینر قرار داده شده است.

### استفاده از kubectl proxy

اگر می‌خواهید API را بدون استفاده از یک کتابخانه کلاینت رسمی جستجو کنید، می‌توانید `kubectl proxy` را به عنوان [فرمان](/docs/tasks/inject-data-application/define-command-argument-container/) یک کانتینر سایدار جدید در پاد اجرا کنید. به این ترتیب، `kubectl proxy` به API احراز هویت می‌کند و آن را در رابط `localhost` پاد اکسپوز می‌کند، بنابراین سایر کانتینرهای پاد می‌توانند مستقیماً از آن استفاده کنند.

### بدون استفاده از پراکسی

می‌توان از استفاده از پراکسی kubectl با ارسال مستقیم توکن احراز هویت به سرور API خودداری کرد. گواهی داخلی اتصال را امن می‌کند.

```shell
# اشاره به نام میزبان سرور API داخلی
APISERVER=https://kubernetes.default.svc

# مسیر به توکن حساب سرویس
SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount

# خواندن فضای نام این پاد
NAMESPACE=$(cat ${SERVICEACCOUNT}/namespace)

# خواندن توکن حساب سرویس
TOKEN=$(cat ${SERVICEACCOUNT}/token)

# مرجع به مرجع گواهی داخلی (CA)
CACERT=${SERVICEACCOUNT}/ca.crt

# کاوش API با TOKEN
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api
```

خروجی مشابه این خواهد بود:

```json
{
  "kind": "APIVersions",
  "versions": ["v1"],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "10.0.1.149:443"
    }
  ]
}
```
