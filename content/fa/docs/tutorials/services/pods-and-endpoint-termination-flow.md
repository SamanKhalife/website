---
title: بررسی رفتار پایانی پاد‌ها و انتهاگرهای آن‌ها
content_type: آموزش
weight: 60
---


<!-- مقدمه -->

با اتصال برنامه‌ی خود به سرویس با مراحلی مانند [اتصال برنامه‌ها به سرویس‌ها](/docs/tutorials/services/connect-applications-service/)،
یک برنامه بازتابی، مکرر و در حال اجرا دارید که در شبکه قرار داده شده است.
این آموزش به شما کمک می‌کند که جریان پایانی پاد‌ها را بررسی کنید و راه‌هایی را برای پیاده‌سازی تخلیه اتصالات مناسب بررسی کنید.

<!-- متن -->

## فرآیند پایان دادن به پاد‌ها و انتهاگرهای آن‌ها

غالباً در مواردی نیاز است که یک پاد را پایان دهید - برای ارتقا یا کاهش مقیاس. به منظور بهبود دسترس‌پذیری برنامه، امکاناتی برای تخلیه مناسب اتصالات فعال ممکن است حیاتی باشند.

این آموزش جریان پایانی پاد را در ارتباط با حالت و حذف انتهاگرهای متناظر با استفاده از یک سرور وب nginx ساده به عنوان نمونه شرح می‌دهد.

<!-- متن -->

## نمونه جریان با پایان دادن به انتهاگر

در زیر مثالی از جریانی که در سند [پایان دادن به پاد‌ها](/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination) توصیف شده است، آمده است.

فرض کنید یک توزیع (Deployment) دارای یک تکنولوژی `nginx` (فقط به عنوان مثال) و یک سرویس دارید:

{{% code_sample file="service/pod-with-graceful-termination.yaml" %}}

{{% code_sample file="service/explore-graceful-termination-nginx.yaml" %}}

حالا پاد توزیع و سرویس را با استفاده از فایل‌های فوق ایجاد کنید:

```shell
kubectl apply -f pod-with-graceful-termination.yaml
kubectl apply -f explore-graceful-termination-nginx.yaml
```

وقتی که پاد و سرویس در حال اجرا هستند، می‌توانید نام هر انتهاگر (EndpointSlice) مرتبط را بگیرید:

```shell
kubectl get endpointslice
```

خروجی مشابه زیر خواهد بود:

```none
NAME                  ADDRESSTYPE   PORTS   ENDPOINTS                 AGE
nginx-service-6tjbr   IPv4          80      10.12.1.199,10.12.1.201   22m
```

می‌توانید وضعیت آن را ببینید و اعتبار سنجی کنید که یک انتهاگر ثبت شده است:

```shell
kubectl get endpointslices -o json -l kubernetes.io/service-name=nginx-service
```

خروجی مشابه زیر خواهد بود:

```none
{
    "addressType": "IPv4",
    "apiVersion": "discovery.k8s.io/v1",
    "endpoints": [
        {
            "addresses": [
                "10.12.1.201"
            ],
            "conditions": {
                "ready": true,
                "serving": true,
                "terminating": false
```

حالا بیایید پاد را پایان دهیم و اعتبار سنجی کنیم که پاد با رعایت دوره پایان دهی مناسب در حال حذف است:

```shell
kubectl delete pod nginx-deployment-7768647bf9-b4b9s
```

همه پاد‌ها:

```shell
kubectl get pods
```

خروجی مشابه زیر خواهد بود:

```none
NAME                                READY   STATUS        RESTARTS      AGE
nginx-deployment-7768647bf9-b4b9s   1/1     Terminating   0             4m1s
nginx-deployment-7768647bf9-rkxlw   1/1     Running       0             8s
```

می‌بینید که پاد جدید زمانبندی شده است.

در حالی که انتهاگر جدید برای پاد جدید ایجاد می‌شود، انتهاگر قدیمی هنوز در حالت حذف است:

```shell
kubectl get endpointslice -o json nginx-service-6tjbr
```

خروجی مشابه زیر خواهد بود:

```none
{
    "addressType": "IPv4",
    "apiVersion": "discovery.k8s.io/v1",
    "endpoints": [
        {
            "addresses": [
                "10.12.1.201"
            ],
            "conditions": {
                "ready": false,
                "serving": true,
                "terminating": true
            },
            "nodeName": "gke-main-default-pool-dca1511c-d17b",
            "targetRef": {
                "kind": "Pod",
                "name": "nginx-deployment-7768647bf9-b4b9s",
                "namespace": "default",
                "uid": "66fa831c-7eb2-407f-bd2c-f96dfe841478"
            },
            "zone": "us-central1-c"
        },
        {
            "addresses": [
                "10.12.1.202"
            ],
            "conditions": {
                "ready": true,
                "serving": true,
                "terminating": false
            },
            "nodeName": "gke-main-default-pool-dca1511c-d17b",
            "targetRef": {
                "kind": "Pod",
                "name": "nginx-deployment-7768647bf9-rkxlw",
                "namespace": "default",
                "uid": "722b1cbe-dcd7-4ed4-8928-4a4d0e2bbe35"
            },
            "zone": "us-central1-c"
```

این امکان را به برنامه‌ها می‌دهد تا در حین پایان دادن، وضعیت خود را اعلام کنند و مشتریان (مانند توزیع‌کننده‌های بار) برای پیاده‌سازی قابلیت تخلیه اتصالات اقدام کنند.
این مشتریان ممکن است انتهاگرهای در حال حذف را شناسایی کرده و برای آن‌ها منطق خاصی را پیاده‌سازی کنند.

در Kubernetes، انتهاگرهای در حال حذف همیشه وضعیت `ready` خود را به عنوان `false` تنظیم می‌کنند. این برای سازگاری بازگشتی است که بارگذارهای موجود از آن برای ترافیک عادی استفاده نکنند

.
اگر نیاز به تخلیه ترافیک در پادی در حال حذف باشد، می‌توان به عنوان شرط `serving` واقعیت را بررسی کرد.

زمانی که پاد حذف می‌شود، انتهاگر قدیمی هم حذف خواهد شد.


## {{% heading "whatsnext" %}}


* یادگیری نحوه [اتصال برنامه‌ها به سرویس‌ها](/docs/tutorials/services/connect-applications-service/)
* یادگیری بیشتر در مورد [استفاده از سرویس برای دسترسی به یک برنامه در یک خوشه](/docs/tasks/access-application-cluster/service-access-application-cluster/)
* یادگیری بیشتر در مورد [اتصال یک بخش جلویی به یک بخش پشتیبان با استفاده از سرویس](/docs/tasks/access-application-cluster/connecting-frontend-backend/)
* یادگیری بیشتر در مورد [ایجاد یک بارگذار بار خارجی](/docs/tasks/access-application-cluster/create-external-load-balancer/)
```

