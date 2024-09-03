---
title: "Example: اجرای برنامه PHP Guestbook با Redis"
reviewers:
- ahmetb
- jimangel
content_type: tutorial
weight: 20
card:
  name: tutorials
  weight: 30
  title: "Example: اجرای برنامه PHP Guestbook با Redis"
min-kubernetes-server-version: v1.14
source: https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook
---

<!-- مرور کلی -->
این آموزش به شما نحوه‌ی ساخت و اجرای یک برنامه وب چند لایه ساده (نه آماده برای محیط تولید) با استفاده از Kubernetes و [Docker](https://www.docker.com/) را نشان می‌دهد. این مثال شامل اجزای زیر است:

* یک نمونه تک‌نود [Redis](https://www.redis.io/) برای ذخیره‌سازی ورودی‌های guestbook
* چند نمونه frontend وب

## {{% heading "اهداف" %}}

* راه‌اندازی یک Redis leader
* راه‌اندازی دو Redis follower
* راه‌اندازی frontend برنامه guestbook
* ارائه و مشاهده سرویس Frontend
* پاکسازی

## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}}

{{< version-check >}}

<!-- محتوای درس -->

## راه‌اندازی پایگاه داده Redis

برنامه guestbook برای ذخیره‌سازی داده‌های خود از Redis استفاده می‌کند.

### ایجاد Deployment Redis

فایل manifest زیر یک کنترل‌گر Deployment را مشخص می‌کند که یک Replica از Pod Redis را اجرا می‌کند.

{{% code_sample file="application/guestbook/redis-leader-deployment.yaml" %}}

1. یک پنجره ترمینال در دایرکتوری ایجاد شده برای فایل‌های manifest باز کنید.
2. Deployment Redis را از فایل `redis-leader-deployment.yaml` اجرا کنید:

   ```shell
   kubectl apply -f https://k8s.io/examples/application/guestbook/redis-leader-deployment.yaml
   ```

3. لیست Pod‌ها را برای تأیید اجرای Pod Redis بررسی کنید:

   ```shell
   kubectl get pods
   ```

   پاسخ باید مشابه زیر باشد:

   ```
   NAME                           READY   STATUS    RESTARTS   AGE
   redis-leader-fb76b4755-xjr2n   1/1     Running   0          13s
   ```

4. برای مشاهده لاگ‌های Pod Redis leader دستور زیر را اجرا کنید:

   ```shell
   kubectl logs -f deployment/redis-leader
   ```

### ایجاد سرویس Redis leader

برنامه guestbook برای نوشتن داده‌های خود به Redis نیاز دارد. برای این منظور باید یک [Service](/docs/concepts/services-networking/service/) اعمال کنید تا ترافیک را به Pod Redis پروکسی کند. یک Service یک سیاست برای دسترسی به Pod‌ها تعریف می‌کند.

{{% code_sample file="application/guestbook/redis-leader-service.yaml" %}}

1. سرویس Redis را از فایل `redis-leader-service.yaml` زیر اعمال کنید:

   ```shell
   kubectl apply -f https://k8s.io/examples/application/guestbook/redis-leader-service.yaml
   ```

2. لیست سرویس‌ها را برای تأیید اجرای سرویس Redis بررسی کنید:

   ```shell
   kubectl get service
   ```

   پاسخ باید مشابه زیر باشد:

   ```
   NAME           TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
   kubernetes     ClusterIP   10.0.0.1     <none>        443/TCP    1m
   redis-leader   ClusterIP   10.103.78.24 <none>        6379/TCP   16s
   ```

{{< note >}}
این فایل manifest یک Service به نام `redis-leader` با مجموعه‌ای از برچسب‌ها ایجاد می‌کند که با برچسب‌های تعریف شده قبلاً همخوانی دارد، بنابراین ترافیک شبکه را به Pod Redis هدایت می‌کند.
{{< /note >}}

### راه‌اندازی Redis followers

اگرچه Redis leader یک Pod تکی است، اما با اضافه کردن چند Replica از Redis follower می‌توانید آن را در دسترس بالا قرار دهید و نیازهای ترافیک را برآورده کنید.

{{% code_sample file="application/guestbook/redis-follower-deployment.yaml" %}}

1. Deployment Redis را از فایل `redis-follower-deployment.yaml` زیر اعمال کنید:

   ```shell
   kubectl apply -f https://k8s.io/examples/application/guestbook/redis-follower-deployment.yaml
   ```

2. با بررسی لیست Pod‌ها مطمئن شوید که دو Replica از Redis follower در حال اجرا هستند:

   ```shell
   kubectl get pods
   ```

   پاسخ باید مشابه زیر باشد:

   ```
   NAME                             READY   STATUS    RESTARTS   AGE
   redis-follower-dddfbdcc9-82sfr   1/1     Running   0          37s
   redis-follower-dddfbdcc9-qrt5k   1/1     Running   0          38s
   redis-leader-fb76b4755-xjr2n     1/1     Running   0          11m
   ```

### ایجاد سرویس Redis follower

برنامه guestbook برای خواندن داده‌های خود از Redis followers نیاز دارد. برای این منظور باید یک [Service](/docs/concepts/services-networking/service/) دیگر را ایجاد کنید.

{{% code_sample file="application/guestbook/redis-follower-service.yaml" %}}

1. سرویس Redis را از فایل `redis-follower-service.yaml` زیر اعمال کنید:

   ```shell
   kubectl apply -f https://k8s.io/examples/application/guestbook/redis-follower-service.yaml
   ```

2. لیست سرویس‌ها را برای تأیید اجرای سرویس Redis بررسی کنید:

   ```shell
   kubectl get service
   ```

   پاسخ باید مشابه زیر باشد:

   ```
   NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
   kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP    3d19h
   redis-follower   ClusterIP   10.110.162.42   <none>        6379/TCP   9s
   redis-leader     ClusterIP   10.103.78.24    <none>        6379/TCP

   6m10s
   ```

{{< note >}}
این فایل manifest یک Service به نام `redis-follower` با مجموعه‌ای از برچسب‌ها ایجاد می‌کند که با برچسب‌های تعریف شده قبلاً همخوانی دارد، بنابراین ترافیک شبکه را به Pod Redis هدایت می‌کند.
{{< /note >}}

```markdown
## راه‌اندازی و افشای رابط کاربری اصلی Guestbook

حالا که ذخیره‌سازی Redis برای guestbook شما در حال اجرا است، وب‌سرورهای guestbook راه‌اندازی می‌کنیم. همانطور که در زمان اجرای Redis، فرانت‌اند نیز با استفاده از Deployment Kubernetes راه‌اندازی می‌شود.

اپلیکیشن guestbook از یک فرانت‌اند PHP استفاده می‌کند. این فرانت‌اند به گونه‌ای پیکربندی شده است که با سرویس‌های Redis follower یا leader ارتباط برقرار می‌کند، بسته به اینکه درخواست یک خواندن یا نوشتن باشد. فرانت‌اند یک رابط JSON را ارائه می‌دهد و یک رابط کاربری بر پایه jQuery-Ajax را ارائه می‌کند.

### ایجاد Deployment فرانت‌اند Guestbook

{{% code_sample file="application/guestbook/frontend-deployment.yaml" %}}

1. Deployment فرانت‌اند را از فایل `frontend-deployment.yaml` اعمال کنید:

   <!---
   برای آزمایش محلی محتوا از طریق مسیر فایل نسبی
   kubectl apply -f ./content/en/examples/application/guestbook/frontend-deployment.yaml
   -->

   ```shell
   kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-deployment.yaml
   ```

1. لیست Pods را برای تأیید اجرای سه نمونه از فرانت‌اند بررسی کنید:

   ```shell
   kubectl get pods -l app=guestbook -l tier=frontend
   ```

   پاسخ باید به شکل زیر باشد:

   ```
   NAME                        READY   STATUS    RESTARTS   AGE
   frontend-85595f5bf9-5tqhb   1/1     Running   0          47s
   frontend-85595f5bf9-qbzwm   1/1     Running   0          47s
   frontend-85595f5bf9-zchwc   1/1     Running   0          47s
   ```

### ایجاد سرویس فرانت‌اند

سرویس Redis که شما اعمال کردید، فقط در داخل خوشه Kubernetes قابل دسترسی است زیرا نوع پیش‌فرض یک سرویس ClusterIP است که یک آدرس IP تک برای مجموعه Pods است که سرویس به آن اشاره دارد فراهم می‌کند. این آدرس IP فقط در داخل خوشه قابل دسترسی است.

اگر می‌خواهید مهمانان بتوانند به guestbook شما دسترسی داشته باشند، باید سرویس فرانت‌اند را به صورت خارجی قابل مشاهده کنید تا یک مشتری بتواند از خارج از خوشه Kubernetes درخواست سرویس را ارسال کند. با این حال، یک کاربر Kubernetes می‌تواند با استفاده از `kubectl port-forward` به سرویس دسترسی داشته باشد، هر چند که از نوع ClusterIP استفاده می‌کند.

{{< note >}}
بعضی از ارائه‌دهندگان ابر، مانند Google Compute Engine یا Google Kubernetes Engine، از بارگذارهای بار تماس پشتیبانی می‌کنند. اگر ارائه‌دهنده ابر شما از بارگذارهای بار پشتیبانی می‌کند و می‌خواهید از آن استفاده کنید، `type: LoadBalancer` را uncomment کنید.
{{< /note >}}

{{% code_sample file="application/guestbook/frontend-service.yaml" %}}

1. سرویس فرانت‌اند را از فایل `frontend-service.yaml` اعمال کنید:

   <!---
   برای آزمایش محلی محتوا از طریق مسیر فایل نسبی
   kubectl apply -f ./content/en/examples/application/guestbook/frontend-service.yaml
   -->

   ```shell
   kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-service.yaml
   ```

1. لیست سرویس‌ها را برای تأیید اجرای سرویس فرانت‌اند بررسی کنید:

   ```shell
   kubectl get services
   ```

   پاسخ باید به شکل زیر باشد:

   ```
   NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
   frontend         ClusterIP   10.97.28.230    <none>        80/TCP     19s
   kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP    3d19h
   redis-follower   ClusterIP   10.110.162.42   <none>        6379/TCP   5m48s
   redis-leader     ClusterIP   10.103.78.24    <none>        6379/TCP   11m
   ```

### مشاهده سرویس فرانت‌اند از طریق `kubectl port-forward`

1. دستور زیر را اجرا کنید تا پورت `8080` در دستگاه محلی خود را به پورت `80` در سرویس فرانت‌اند فوروارد کنید.

   ```shell
   kubectl port-forward svc/frontend 8080:80
   ```

   پاسخ باید به شکل زیر باشد:

   ```
   Forwarding from 127.0.0.1:8080 -> 80
   Forwarding from [::1]:8080 -> 80
   ```

1. صفحه را در مرورگر خود بارگذاری کنید: [http://localhost:8080](http://localhost:8080) تا guestbook خود را مشاهده کنید.

### مشاهده سرویس فرانت‌اند از طریق `LoadBalancer`

اگر شما manifest `frontend-service.yaml` را با نوع `LoadBalancer` اعمال کردید، برای مشاهده Guestbook خود نیاز به یافتن آدرس IP است.

1. دستور زیر را اجرا کنید تا آدرس IP برای سرویس فرانت‌اند را بدست آورید.

   ```shell
   kubectl get service frontend
   ```

   پاسخ باید به شکل زیر باشد:

   ```
   NAME       TYPE           CLUSTER-IP      EXTERNAL-IP        PORT(S)        AGE
   frontend   LoadBalancer   10.51.242.136   109.197.92.229     80:32372/TCP   1m
   ```

1. آدرس IP خارجی را کپی کرده و صفحه را در مرور

گر خود بارگذاری کنید تا guestbook خود را مشاهده کنید.

{{< note >}}
با افزودن چند ورودی guestbook با وارد کردن یک پیام و کلیک بر روی ارسال، سعی کنید تا اطمینان حاصل کنید که داده‌ها با موفقیت به Redis اضافه شده اند که از طریق سرویس‌هایی که از پیش ایجاد کرده‌اید ممکن است.
{{< /note >}}

## تغییر مقیاس فرانت‌اند وب

شما می‌توانید به میزان نیاز خود را افزایش یا کاهش دهید زیرا سرورهای شما به عنوان یک سرویس تعریف شده است که از کنترل‌گر Deployment استفاده می‌کند.

1. دستور زیر را اجرا کنید تا تعداد Pods فرانت‌اند را افزایش دهید:

   ```shell
   kubectl scale deployment frontend --replicas=5
   ```

1. لیست Pods را برای تأیید تعداد Pods فرانت‌اند در حال اجرا بررسی کنید:

   ```shell
   kubectl get pods
   ```

   پاسخ باید به شکل زیر باشد:

   ```
   NAME                             READY   STATUS    RESTARTS   AGE
   frontend-85595f5bf9-5df5m        1/1     Running   0          83s
   frontend-85595f5bf9-7zmg5        1/1     Running   0          83s
   frontend-85595f5bf9-cpskg        1/1     Running   0          15m
   frontend-85595f5bf9-l2l54        1/1     Running   0          14m
   frontend-85595f5bf9-l9c8z        1/1     Running   0          14m
   redis-follower-dddfbdcc9-82sfr   1/1     Running   0          97m
   redis-follower-dddfbdcc9-qrt5k   1/1     Running   0          97m
   redis-leader-fb76b4755-xjr2n     1/1     Running   0          108m
   ```

1. دستور زیر را اجرا کنید تا تعداد Pods فرانت‌اند را کاهش دهید:

   ```shell
   kubectl scale deployment frontend --replicas=2
   ```

1. لیست Pods را برای تأیید تعداد Pods فرانت‌اند در حال اجرا بررسی کنید:

   ```shell
   kubectl get pods
   ```

   پاسخ باید به شکل زیر باشد:

   ```
   NAME                             READY   STATUS    RESTARTS   AGE
   frontend-85595f5bf9-cpskg        1/1     Running   0          16m
   frontend-85595f5bf9-l9c8z        1/1     Running   0          15m
   redis-follower-dddfbdcc9-82sfr   1/1     Running   0          98m
   redis-follower-dddfbdcc9-qrt5k   1/1     Running   0          98m
   redis-leader-fb76b4755-xjr2n     1/1     Running   0          109m
   ```

## {{% heading "cleanup" %}}

حذف Deployments و Services باعث حذف هر Pods در حال اجرا می‌شود. از برچسب‌ها برای حذف منابع چندگانه با یک دستور استفاده کنید.

1. دستورات زیر را اجرا کنید تا تمام Pods، Deployments، و Services را حذف کنید.

   ```shell
   kubectl delete deployment -l app=redis
   kubectl delete service -l app=redis
   kubectl delete deployment frontend
   kubectl delete service frontend
   ```

   پاسخ باید به شکل زیر باشد:

   ```
   deployment.apps "redis-follower" deleted
   deployment.apps "redis-leader" deleted
   deployment.apps "frontend" deleted
   service "frontend" deleted
   ```

1. لیست Pods را برای تأیید عدم وجود Pods در حال اجرا بررسی کنید:

   ```shell
   kubectl get pods
   ```

   پاسخ باید به شکل زیر باشد:

   ```
   No resources found in default namespace.
   ```

## {{% heading "whatsnext" %}}

* تکمیل آموزش‌های تعاملی [Kubernetes Basics](/docs/tutorials/kubernetes-basics/)
* استفاده از Kubernetes برای ایجاد یک وبلاگ با استفاده از [Persistent Volumes for MySQL and Wordpress](/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/#visit-your-new-wordpress-blog)
* بیشتر بخوانید در مورد [اتصال برنامه‌ها با سرویس‌ها](/docs/tutorials/services/connect-applications-service/)
* بیشتر بخوانید در مورد [استفاده از برچسب‌ها به صورت موثر](/docs/concepts/overview/working-with-objects/labels/#using-labels-effectively)
