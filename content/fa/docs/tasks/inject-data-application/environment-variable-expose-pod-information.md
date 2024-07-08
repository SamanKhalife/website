## افشای اطلاعات Pod به کانتینرها از طریق متغیرهای محیطی

این صفحه نشان می‌دهد چگونه یک Pod می‌تواند از متغیرهای محیطی استفاده کند تا اطلاعاتی درباره خود را به کانتینرهایی که در آن Pod اجرا می‌شوند ارائه دهد، با استفاده از _downward API_.
شما می‌توانید از متغیرهای محیطی برای افشای فیلدهای Pod، فیلدهای کانتینر، یا هر دو استفاده کنید.

در Kubernetes، دو روش برای افشای فیلدهای Pod و کانتینر به یک کانتینر در حال اجرا وجود دارد:

* _متغیرهای محیطی_، که در این تمرین توضیح داده شده است
* [فایل‌های ولوم](/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/)

این دو روش افشای فیلدهای Pod و کانتینر به downward API نامیده می‌شوند.

زیر مجموعه‌های اولیه از ارتباطات از بین برنامه‌های کانتینری به این وسیله هستند و به این وسیله است که برای این نوع ارتباطات استفاده می‌شود.

بیشتر اطلاعات درباره دسترسی به این سرویس [اینجا](/docs/tutorials/services/connect-applications-service/#accessing-the-service) می‌توانید بخوانید.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}}

<!-- steps -->

## استفاده از فیلدهای Pod به عنوان مقدار متغیرهای محیطی

در این بخش از تمرین، شما یک Pod ایجاد می‌کنید که شامل یک کانتینر است، و فیلدهای سطح Pod را به عنوان متغیرهای محیطی به کانتینر در حال اجرا انتقال می‌دهید.

{{% code_sample file="pods/inject/dapi-envars-pod.yaml" %}}

در این مانیفست، پنج متغیر محیطی را می‌بینید. فیلد `env`
یک آرایه از تعریف‌های
متغیرهای محیطی است.
عنصر اول در آرایه مشخص می‌کند که متغیر محیطی `MY_NODE_NAME` مقدار خود را از فیلد `spec.nodeName` Pod دریافت می‌کند. به طور مشابه، سایر متغیرهای محیطی نیز نام خود را از فیلدهای Pod دریافت می‌کنند.

{{< note >}}
فیلدهای در این مثال فیلدهای Pod هستند. آنها فیلدهای کانتینر در Pod نیستند.
{{< /note >}}

ایجاد Pod:

```shell
kubectl apply -f https://k8s.io/examples/pods/inject/dapi-envars-pod.yaml
```

بررسی اجرای کانتینر در Pod:

```shell
# اگر Pod جدید هنوز سالم نیست، چند بار دیگر این دستور را اجرا کنید.
kubectl get pods
```

مشاهده لاگ‌های کانتینر:

```shell
kubectl logs dapi-envars-fieldref
```

خروجی نشان می‌دهد مقادیر متغیرهای محیطی انتخاب شده:

```
minikube
dapi-envars-fieldref
default
172.17.0.4
default
```

برای دیدن اینکه چرا این مقادیر در لاگ هستند، به فیلدهای `command` و `args` در فایل پیکربندی نگاه کنید. وقتی که کانتینر شروع به کار می‌کند، مقادیر پنج متغیر محیطی را به stdout می‌نویسد. این عمل هر ده ثانیه تکرار می‌شود.

بعد، وارد شل کانتینری که در Pod شما در حال اجرا است شوید:

```shell
kubectl exec -it dapi-envars-fieldref -- sh
```

در شل خود، مشاهده متغیرهای محیطی:

```shell
# این دستور را در یک شل داخل کانتینر اجرا کنید
printenv
```

خروجی نشان می‌دهد که برخی از متغیرهای محیطی مقادیر فیلدهای Pod را دارند:

```
MY_POD_SERVICE_ACCOUNT=default
...
MY_POD_NAMESPACE=default
MY_POD_IP=172.17.0.4
...
MY_NODE_NAME=minikube
...
MY_POD_NAME=dapi-envars-fieldref
```

## استفاده از فیلدهای کانتینر به عنوان مقدار متغیرهای محیطی

در تمرین قبلی، شما از اطلاعات از فیلدهای سطح Pod به عنوان مقادیر
برای متغیرهای محیطی استفاده کردید.
در این تمرین بعدی، شما قصد دارید فیلدهایی را که بخشی از تعریف Pod هستند، اما از کانتینر خاص گرفته شده‌اند را به آن متغیرهای محیطی انتقال دهید.

اینجا یک مانیفست برای یک Pod دیگر است که دوباره فقط یک کانتینر دارد:

{{% code_sample file="pods/inject/dapi-envars-container.yaml" %}}

در این مانیفست، چهار متغیر محیطی را مشاهده می‌کنید. فیلد `env`
یک آرایه از تعریف‌های
متغیرهای محیطی است.
عنصر اول در آرایه مشخص می‌کند که متغیر محیطی `MY_CPU_REQUEST` مقدار خود را از فیلد `requests.cpu` یک کانتینر با نام `test-container` دریافت می‌کند. به طور مشابه، متغیرهای محیطی دیگر نیز از فیلدهایی که ویژه این کانتینر هستند، مقادیر خود را دریافت می‌کنند.

ایجاد Pod:

```shell
kubectl apply -f https://k8s.io/examples/pods/inject/dapi-envars-container.yaml
```

بررسی اجرای کانتینر در Pod:



```shell
# اگر Pod جدید هنوز سالم نیست، چند بار دیگر این دستور را اجرا کنید.
kubectl get pods
```

مشاهده لاگ‌های کانتینر:

```shell
kubectl logs dapi-envars-resourcefieldref
```

خروجی نشان می‌دهد مقادیر متغیرهای محیطی انتخاب شده:

```
1
1
33554432
67108864
```

## {{% heading "whatsnext" %}}

* مطالعه [تعریف متغیرهای محیطی برای یک کانتینر](/docs/tasks/inject-data-application/define-environment-variable-container/)
* مطالعه مشخصات API [`spec`](/docs/reference/kubernetes-api/workload-resources/pod-v1/#PodSpec) برای Pod. این شامل تعریف Container (قسمتی از Pod) است.
* مطالعه لیست [فیلدهای موجود](/docs/concepts/workloads/pods/downward-api/#available-fields) که می‌توانید از طریق downward API ارائه دهید.

در مرجع API قدیمی، درباره Pods، containers و environment variables بیشتر بخوانید:

* [PodSpec](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podspec-v1-core)
* [Container](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#container-v1-core)
* [EnvVar](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#envvar-v1-core)
* [EnvVarSource](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#envvarsource-v1-core)
* [ObjectFieldSelector](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#objectfieldselector-v1-core)
* [ResourceFieldSelector](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#resourcefieldselector-v1-core)
