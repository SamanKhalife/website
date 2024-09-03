---
title: آموزش Hello Minikube
content_type: tutorial
weight: 5
card:
  name: tutorials
  weight: 10
---

<!-- overview -->

این آموزش به شما نحوه اجرای یک برنامه نمونه در Kubernetes با استفاده از Minikube را نشان می‌دهد.
این آموزش یک تصویر کانتینر ارائه می‌دهد که از NGINX برای پاسخگویی به درخواست‌ها استفاده می‌کند.

## {{% heading "objectives" %}}

* اجرای یک برنامه نمونه در Minikube.
* اجرای برنامه.
* مشاهده لاگ‌های برنامه.

## {{% heading "prerequisites" %}}


این آموزش فرض می‌کند که شما Minikube را قبلاً نصب کرده‌اید.
برای دستورات نصب، به __مرحله 1__ در [شروع Minikube](https://minikube.sigs.k8s.io/docs/start/) مراجعه کنید.
{{< note >}}
تنها دستورات در __مرحله 1، نصب__ را اجرا کنید. بقیه مراحل در این صفحه پوشش داده شده‌اند.  
{{< /note >}}

شما همچنین نیاز به نصب `kubectl` دارید.
برای دستورات نصب، به [نصب ابزارها](/docs/tasks/tools/#kubectl) مراجعه کنید.


<!-- lessoncontent -->

## ایجاد یک کلاستر Minikube

```shell
minikube start
```

## باز کردن داشبورد

داشبورد Kubernetes را باز کنید. این کار را می‌توانید به دو روش متفاوت انجام دهید:

{{< tabs name="dashboard" >}}
{{% tab name="راه‌اندازی مرورگر" %}}
یک ترمینال **جدید** باز کنید و اجرا کنید:
```shell
# یک ترمینال جدید باز کنید و این را اجرا کنید.
minikube dashboard
```

حالا به ترمینالی که Minikube را اجرا کرده‌اید برگردید.

{{< note >}}
دستور `dashboard` افزونه داشبورد را فعال می‌کند و پروکسی را در مرورگر وب پیش‌فرض باز می‌کند.
شما می‌توانید منابع Kubernetes مانند Deployment و Service را در داشبورد ایجاد کنید.

برای آگاهی از اینکه چگونه از فراخوانی مستقیم مرورگر از ترمینال جلوگیری کنید و URL برای داشبورد وب را دریافت کنید، به تب "کپی و paste URL" مراجعه کنید.

دستور `dashboard` یک پروکسی موقت ایجاد می‌کند تا داشبورد از خارج از شبکه مجازی Kubernetes دسترسی‌پذیر شود.

برای متوقف کردن پروکسی، دستور `Ctrl+C` را برای خروج از فرآیند اجرا کنید.
پس از خروج از دستور، داشبورد در خوشه Kubernetes فعال می‌ماند.
می‌توانید دستور `dashboard` را دوباره اجرا کنید تا یک پروکسی دیگر برای دسترسی به داشبورد ایجاد کنید.
{{< /note >}}

{{% /tab %}}
{{% tab name="کپی و paste URL" %}}

اگر نمی‌خواهید Minikube برای شما یک مرورگر وب باز کند، زیر دستور `dashboard` را با زیر دستور `--url` اجرا کنید. `minikube` یک URL را خروجی می‌دهد که می‌توانید آن را در مرورگر خود باز کنید.

یک ترمینال **جدید** باز کنید و اجرا کنید:
```shell
# یک ترمینال جدید باز کنید و این را اجرا کنید.
minikube dashboard --url
```

حالا می‌توانید از این URL استفاده کنید و به ترمینالی که Minikube را اجرا کرده‌اید برگردید.

{{% /tab %}}
{{< /tabs >}}

## ایجاد یک Deployment

یک [*Pod* Kubernetes](/docs/concepts/workloads/pods/) یک گروه از یک یا چند کانتینر است که برای اداره و شبکه‌سازی به هم متصل شده‌اند. Pod در این آموزش فقط دارای یک کانتینر است.
یک [*Deployment Kubernetes*](/docs/concepts/workloads/controllers/deployment/) وضعیت پاد شما را بررسی کرده و اگر پاد کانتینر خود را متوقف کند، دوباره آن را شروع می‌کند. Deployments را به عنوان روش توصیه می‌شود برای مدیریت ایجاد و مقیاس‌پذیری پاد.

1. از دستور `kubectl create` برای ایجاد یک Deployment استفاده کنید که یک Pod را مدیریت می‌کند. این Pod یک کانتینر بر اساس تصویر Docker ارائه شده اجرا می‌کند.

    ```shell
    # یک تصویر آزمایشی اجرا کنید که شامل یک وب‌سرور است
    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.39 -- /agnhost netexec --http-port=8080
    ```

1. Deployment را مشاهده کنید:

    ```shell
    kubectl get deployments
    ```

    خروجی مشابه زیر خواهد بود:

    ```
    NAME         READY   UP-TO-DATE   AVAILABLE   AGE
    hello-node   1/1     1            1           1m
    ```

    (ممکن است کمی زمان برود تا پاد در دسترس قرار گیرد. اگر "0/1" را مشاهده می‌کنید، چند ثانیه دیگر دوباره امتحان کنید.)

1. Pod را مشاهده کنید:

    ```shell
    kubectl get pods
    ```

    خروجی مشابه زیر خواهد بود:

    ```
    NAME                          READY     STATUS    RESTARTS   AGE
    hello-node-5f76cf6ccf-br9b5   1/1       Running   0          1m
    ```

1. رویدادهای خوشه را مشاهده کنید:

    ```shell
    kubectl get events
    ```

1. پیکربندی `kubectl` را مشاهده کنید:

    ```shell
    kubectl config view
    ```

1. لاگ‌های برنامه را برای یک کانتینر در یک پاد مشاهده کنید (نام پاد را با نامی که از دستور `kubectl get pods` دریافت کردید جایگزین کنید).

   {{< note >}}
   نام `hello-node-5f76cf6ccf-br9b5` را در دستور `kubectl logs` با نام پاد از خروجی دستور `kubectl get pods` جایگزین کنید.
   {{< /note >}}

   ```shell
   kubectl logs hello-node-5f76cf6ccf-br9b5
   ```

   خروجی مشابه زیر خواهد بود:

   ```
   I0911 09:19:26.677397       1 log.go:195] Started HTTP server on port 8080
   I0911 09:19:26.677586       1 log.go:195] Started UDP server on port  8081
   ```


{{< note >}}
برای اطلاعات بیشتر درباره دستورات `kubectl` به [بررسی اجمالی kubectl](/docs/reference/kubectl/) مراجعه کنید.
{{< /note >}}

## ایجاد یک سرویس

به طور پیش‌فرض، Pod تنها از طریق آدرس IP داخلی خود در داخل کلاستر Kubernetes قابل دسترسی است. برای اینکه کانتینر `hello-node` از خارج از شبکه مجازی Kubernetes قابل دسترسی باشد، باید Pod را به عنوان یک [*Service* Kubernetes](/docs/concepts/services-networking/service/) منتشر کنید.

1. Pod را با استفاده از دستور `kubectl expose` به اینترنت عمومی منتشر کنید:

    ```shell
    kubectl expose deployment hello-node --type=LoadBalancer --port=8080
    ```

    پرچم `--type=LoadBalancer` نشان می‌دهد که می‌خواهید سرویس خود را خارج از کلاستر منتشر کنید.

    کد برنامه درون تصویر آزمایشی تنها به پورت TCP 8080 گوش می‌دهد. اگر از `kubectl expose` برای انتشار پورت دیگری استفاده کنید، کلاینت‌ها نمی‌توانند به آن پورت دیگر متصل شوند.

2. سرویسی که ایجاد کردید را مشاهده کنید:

    ```shell
    kubectl get services
    ```

    خروجی مشابه زیر خواهد بود:

    ```
    NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    hello-node   LoadBalancer   10.108.144.78   <pending>     8080:30369/TCP   21s
    kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          23m
    ```

    در ارائه‌دهندگان ابری که از بارگذاری بار پشتیبانی می‌کنند، یک آدرس IP خارجی برای دسترسی به سرویس تخصیص داده می‌شود. در Minikube، نوع `LoadBalancer` سرویس را از طریق دستور `minikube service` قابل دسترسی می‌کند.

3. دستور زیر را اجرا کنید:

    ```shell
    minikube service hello-node
    ```

    این یک پنجره مرورگر را باز می‌کند که برنامه شما را سرو می‌کند و پاسخ برنامه را نشان می‌دهد.

## فعال‌سازی افزونه‌ها

ابزار Minikube شامل مجموعه‌ای از {{< glossary_tooltip text="افزونه‌ها" term_id="addons" >}} داخلی است که می‌توانند در محیط Kubernetes محلی فعال، غیرفعال و باز شوند.

1. افزونه‌های پشتیبانی شده فعلی را فهرست کنید:

    ```shell
    minikube addons list
    ```

    خروجی مشابه زیر خواهد بود:

    ```
    addon-manager: enabled
    dashboard: enabled
    default-storageclass: enabled
    efk: disabled
    freshpod: disabled
    gvisor: disabled
    helm-tiller: disabled
    ingress: disabled
    ingress-dns: disabled
    logviewer: disabled
    metrics-server: disabled
    nvidia-driver-installer: disabled
    nvidia-gpu-device-plugin: disabled
    registry: disabled
    registry-creds: disabled
    storage-provisioner: enabled
    storage-provisioner-gluster: disabled
    ```

1. یک افزونه را فعال کنید، به عنوان مثال، `metrics-server`:

    ```shell
    minikube addons enable metrics-server
    ```

    خروجی مشابه زیر خواهد بود:

    ```
    The 'metrics-server' addon is enabled
    ```

1. Pod و سرویس ایجاد شده توسط نصب آن افزونه را مشاهده کنید:

    ```shell
    kubectl get pod,svc -n kube-system
    ```

    خروجی مشابه زیر خواهد بود:

    ```
    NAME                                        READY     STATUS    RESTARTS   AGE
    pod/coredns-5644d7b6d9-mh9ll                1/1       Running   0          34m
    pod/coredns-5644d7b6d9-pqd2t                1/1       Running   0          34m
    pod/metrics-server-67fb648c5                1/1       Running   0          26s
    pod/etcd-minikube                           1/1       Running   0          34m
    pod/influxdb-grafana-b29w8                  2/2       Running   0          26s
    pod/kube-addon-manager-minikube             1/1       Running   0          34m
    pod/kube-apiserver-minikube                 1/1       Running   0          34m
    pod/kube-controller-manager-minikube        1/1       Running   0          34m
    pod/kube-proxy-rnlps                        1/1       Running   0          34m
    pod/kube-scheduler-minikube                 1/1       Running   0          34m
    pod/storage-provisioner                     1/1       Running   0          34m

    NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
    service/metrics-server         ClusterIP   10.96.241.45    <none>        80/TCP              26s
    service/kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP       34m
    service/monitoring-grafana     NodePort    10.99.24.54     <none>        80:30002/TCP        26s
    service/monitoring-influxdb    ClusterIP   10.111.169.94   <none>        8083/TCP,8086/TCP   26s
    ```

```markdown
1. خروجی `metrics-server` را بررسی کنید:

    ```shell
    kubectl top pods
    ```

    خروجی مشابه به این است:

    ```
    NAME                         CPU(cores)   MEMORY(bytes)
    hello-node-ccf4b9788-4jn97   1m           6Mi
    ```

    اگر پیام زیر را دیدید، صبر کنید و دوباره تلاش کنید:

    ```
    error: Metrics API not available
    ```

1. `metrics-server` را غیرفعال کنید:

    ```shell
    minikube addons disable metrics-server
    ```

    خروجی مشابه به این است:

    ```
    metrics-server was successfully disabled
    ```

## پاکسازی

اکنون می‌توانید منابعی که در کلاستر خود ایجاد کرده‌اید را پاکسازی کنید:

```shell
kubectl delete service hello-node
kubectl delete deployment hello-node
```

کلاستر Minikube را متوقف کنید:

```shell
minikube stop
```

اختیاری، ماشین مجازی Minikube را حذف کنید:

```shell
# اختیاری
minikube delete
```

اگر می‌خواهید دوباره از minikube برای یادگیری بیشتر در مورد Kubernetes استفاده کنید، نیازی به حذف آن نیست.

## نتیجه‌گیری

این صفحه جنبه‌های اساسی برای راه‌اندازی یک کلاستر minikube را پوشش داد. اکنون آماده‌ی استقرار برنامه‌ها هستید.

## {{% heading "whatsnext" %}}

* آموزش برای _[استقرار اولین برنامه خود در Kubernetes با استفاده از kubectl](/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/)_.
* بیشتر در مورد [اشیاء استقرار](/docs/concepts/workloads/controllers/deployment/) یاد بگیرید.
* بیشتر در مورد [استقرار برنامه‌ها](/docs/tasks/run-application/run-stateless-application-deployment/) یاد بگیرید.
* بیشتر در مورد [اشیاء سرویس](/docs/concepts/services-networking/service/) یاد بگیرید.
