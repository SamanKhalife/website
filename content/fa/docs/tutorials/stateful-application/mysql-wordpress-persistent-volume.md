---
title: "مثال: استقرار WordPress و MySQL با حجم‌های دائمی"
reviewers:
- ahmetb
content_type: tutorial
weight: 20
card: 
  name: tutorials
  weight: 40
  title: "مثال Stateful: WordPress با حجم‌های دائمی"
---

<!-- خلاصه -->
این آموزش به شما نحوه استقرار یک وب‌سایت WordPress و یک پایگاه داده MySQL با استفاده از Minikube را نشان می‌دهد. هر دو برنامه از PersistentVolumes و PersistentVolumeClaims برای ذخیره داده‌ها استفاده می‌کنند.

یک [PersistentVolume](/docs/concepts/storage/persistent-volumes/) (PV) یک قطعه ذخیره‌سازی در خوشه است که به صورت دستی توسط مدیر تامین شده است، یا به طور پویا توسط Kubernetes با استفاده از [StorageClass](/docs/concepts/storage/storage-classes) تامین می‌شود. یک [PersistentVolumeClaim](/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) (PVC) یک درخواست برای ذخیره‌سازی توسط کاربر است که می‌تواند توسط یک PV برآورده شود. PersistentVolumes و PersistentVolumeClaims مستقل از چرخه عمر Pods هستند و داده‌ها را از طریق بازنشانی، مجدد‌برنامه‌ریزی، و حتی حذف Pods حفظ می‌کنند.

{{< warning >}}
این استقرار مناسب برای موارد استفاده تولیدی نیست، زیرا از Pods تک‌نسخه‌ای WordPress و MySQL استفاده می‌کند. در نظر داشته باشید که از [نمودار Helm برنامه WordPress](https://github.com/bitnami/charts/tree/master/bitnami/wordpress) برای استقرار WordPress در محیط تولید استفاده کنید.
{{< /warning >}}

{{< note >}}
فایل‌های ارائه شده در این آموزش از API‌های استقرار GA استفاده می‌کنند و به خصوص برای نسخه‌های Kubernetes 1.9 به بعد است. اگر می‌خواهید از این آموزش با نسخه‌های قدیمی‌تر Kubernetes استفاده کنید، لطفا API version را به‌طور مناسب به‌روزرسانی کنید یا به نسخه‌های قدیمی‌تر از این آموزش مراجعه کنید.
{{< /note >}}

## {{% heading "اهداف" %}}

* ایجاد PersistentVolumeClaims و PersistentVolumes
* ایجاد یک `kustomization.yaml` با
  * یک تولیدکننده Secret
  * پیکربندی‌های منبع MySQL
  * پیکربندی‌های منبع WordPress
* اعمال دایرکتوری kustomization با `kubectl apply -k ./`
* پاکسازی

## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

مثال نشان داده شده در این صفحه با `kubectl` نسخه 1.27 و بالاتر کار می‌کند.

فایل‌های پیکربندی زیر را دانلود کنید:

1. [mysql-deployment.yaml](/examples/application/wordpress/mysql-deployment.yaml)

1. [wordpress-deployment.yaml](/examples/application/wordpress/wordpress-deployment.yaml)

<!-- محتوای درس -->

## ایجاد PersistentVolumeClaims و PersistentVolumes

هرکدام از MySQL و Wordpress نیازمند یک PersistentVolume برای ذخیره داده‌ها هستند. PersistentVolumeClaims آن‌ها در مرحله استقرار ایجاد می‌شود.

بسیاری از محیط‌های خوشه دارای یک StorageClass پیش‌فرض هستند. زمانی که یک StorageClass در PersistentVolumeClaim مشخص نشود، StorageClass پیش‌فرض خوشه استفاده می‌شود. زمانی که یک PersistentVolumeClaim ایجاد می‌شود، یک PersistentVolume به طور پویا بر اساس پیکربندی StorageClass تأمین می‌شود.

{{< warning >}}
در خوشه‌های محلی، StorageClass پیش‌فرض از provisioner `hostPath` استفاده می‌کند. حجم‌های `hostPath` فقط برای توسعه و آزمایش مناسب هستند. با حجم‌های `hostPath`، داده‌های شما در `/tmp` روی نودی که Pod روی آن برنامه‌ریزی شده است قرار دارد و بین نودها جابجا نمی‌شود. اگر یک Pod بمیرد و بر روی نود دیگری در خوشه برنامه‌ریزی شود یا نود دوباره راه‌اندازی شود، داده‌ها از بین می‌روند.
{{< /warning >}}

{{< note >}}
اگر شما یک خوشه Kubernetes دارای provisioner `hostPath` راه‌اندازی می‌کنید، باید پرچم `--enable-hostpath-provisioner` را در مولفه `controller-manager` تنظیم کنید.
{{< /note >}}

{{< note >}}
اگر یک خوشه Kubernetes در محیط Google Kubernetes Engine دارید، لطفا [این راهنما](https://cloud.google.com/kubernetes-engine/docs/tutorials/persistent-disk) را دنبال کنید.
{{< /note >}}

## ایجاد kustomization.yaml

### افزودن یک Secret generator

یک [Secret](/docs/concepts/configuration/secret/) یک شیء است که یک قطعه اطلاعات حساس مانند رمزعبور یا کلید را ذخیره می‌کند. از نسخه 1.14 به بعد، `kubectl` از مدیریت شیء Kubernetes با استفاده از یک فایل kustomization پشتیبانی می‌کند. می‌توانید با استفاده از generators، یک Secret را ایجاد کنید.

یک Secret generator را در `kustomization.yaml` با استفاده از دستور زیر اضافه کنید. شما باید `YOUR_PASSWORD` را با رمزعبور مورد نظر خود جایگزین کنید.

```shell
cat <<EOF >./kustomization.yaml
secretGenerator:
- name: mysql-pass
  literals:
  - password=YOUR_PASSWORD
EOF
```
```markdown
## اضافه کردن پیکربندی منابع برای MySQL و WordPress

مانیفست زیر، استقرار MySQL با یک نمونه واحد را توصیف می‌کند. کانتینر MySQL حجم PersistentVolume را در `/var/lib/mysql` می‌مونت می‌کند. متغیر محیطی `MYSQL_ROOT_PASSWORD` رمز عبور پایگاه داده را از Secret تنظیم می‌کند.

{{% code_sample file="application/wordpress/mysql-deployment.yaml" %}}

مانیفست زیر، استقرار WordPress با یک نمونه واحد را توصیف می‌کند. کانتینر WordPress حجم PersistentVolume را در `/var/www/html` برای فایل‌های داده‌های وبسایت می‌مونت می‌کند. متغیر محیطی `WORDPRESS_DB_HOST` نام سرویس MySQL را که در بالا تعریف شده است تنظیم می‌کند، و WordPress به دیتابیس از طریق سرویس دسترسی خواهد داشت. متغیر محیطی `WORDPRESS_DB_PASSWORD` رمز عبور پایگاه داده را از Secretی که تولید شده است تنظیم می‌کند.

{{% code_sample file="application/wordpress/wordpress-deployment.yaml" %}}

1. فایل پیکربندی استقرار MySQL را دانلود کنید.

   ```shell
   curl -LO https://k8s.io/examples/application/wordpress/mysql-deployment.yaml
   ```

2. فایل پیکربندی استقرار WordPress را دانلود کنید.

   ```shell
   curl -LO https://k8s.io/examples/application/wordpress/wordpress-deployment.yaml
   ```

3. آن‌ها را به فایل `kustomization.yaml` اضافه کنید.

   ```shell
   cat <<EOF >>./kustomization.yaml
   resources:
     - mysql-deployment.yaml
     - wordpress-deployment.yaml
   EOF
   ```

## اعمال و بررسی

فایل `kustomization.yaml` شامل تمام منابع برای استقرار یک سایت WordPress و یک پایگاه داده MySQL است. می‌توانید با اجرای دستور زیر این دایرکتوری را اعمال کنید.

```shell
kubectl apply -k ./
```

حالا می‌توانید از وجود تمام اشیاء اطمینان حاصل کنید.

1. اطمینان حاصل شود که Secret وجود دارد با اجرای دستور زیر:

   ```shell
   kubectl get secrets
   ```

   پاسخ باید به این صورت باشد:

   ```
   NAME                    TYPE                                  DATA   AGE
   mysql-pass-c57bb4t7mf   Opaque                                1      9s
   ```

2. اطمینان حاصل شود که یک PersistentVolume به طور پویا تأمین شده است.

   ```shell
   kubectl get pvc
   ```

   {{< note >}}
   ممکن است تا چند دقیقه طول بکشد تا PVs تأمین شود و بایند شود.
   {{< /note >}}

   پاسخ باید به این صورت باشد:

   ```
   NAME             STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
   mysql-pv-claim   Bound     pvc-8cbd7b2e-4044-11e9-b2bb-42010a800002   20Gi       RWO            standard           77s
   wp-pv-claim      Bound     pvc-8cd0df54-4044-11e9-b2bb-42010a800002   20Gi       RWO            standard           77s
   ```

3. اطمینان حاصل شود که Pod در حال اجرا است با اجرای دستور زیر:

   ```shell
   kubectl get pods
   ```

   {{< note >}}
   ممکن است تا چند دقیقه طول بکشد تا وضعیت Pods به `RUNNING` برسد.
   {{< /note >}}

   پاسخ باید به این صورت باشد:

   ```
   NAME                               READY     STATUS    RESTARTS   AGE
   wordpress-mysql-1894417608-x5dzt   1/1       Running   0          40s
   ```

4. اطمینان حاصل شود که سرویس در حال اجرا است با اجرای دستور زیر:

   ```shell
   kubectl get services wordpress
   ```

   پاسخ باید به این صورت باشد:

   ```
   NAME        TYPE            CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
   wordpress   LoadBalancer    10.0.0.89    <pending>     80:32406/TCP   4m
   ```

   {{< note >}}
   Minikube فقط می‌تواند سرویس‌ها را از طریق `NodePort` ارائه دهد. EXTERNAL-IP همیشه در حال انتظار است.
   {{< /note >}}

5. برای دریافت آدرس IP برای سرویس WordPress دستور زیر را اجرا کنید:

   ```shell
   minikube service wordpress --url
   ```

   پاسخ باید به این صورت باشد:

   ```
   http://1.2.3.4:32406
   ```

6. آدرس IP را کپی کرده و صفحه را در مرورگر خود بارگذاری کنید تا وب‌سایت خود را مشاهده کنید.

   باید صفحه راه‌اندازی WordPress را مشابه تصویر زیر ببینید.

   ![wordpress-init](https://raw.githubusercontent.com/kubernetes/examples/master/mysql-wordpress-pd/WordPress.png)

   {{< warning >}}
   این صفحه راه‌اندازی WordPress را ترک نکنید. اگر فرد دیگری آن را پیدا کند، می‌تواند یک وب‌سایت را بر روی نمونه شما نصب کند و از آن برای ارائه محتوای مخرب استفاده کند.<br/><br/>
   یا WordPress را با ایجاد یک نام کاربری و رمز عبور نصب کنید یا نمونه خود را حذف کنید.
   {{< /warning >}}

## {{% heading "پاکسازی" %}}

1. دستور زیر را برای حذف Secret، Deployments، Services و PersistentVolumeClaims اجرا کنید:

   ```shell
   kubectl delete -k ./
   ```

## {{% heading "مرحله‌بعدی" %}}

* بیشتر در مورد [معاینه و اشکال‌زدایی](/docs/tasks/debug/debug-application/debug-running-pod/) بیاموزید
* بیشتر در مورد [کارها](/docs/concepts/workloads/controllers/job/) بیاموزید
* بیشتر در مورد [انتقال پورت](/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) بیاموزید
* یاد بگیرید که چگونه [ به یک کانتینر دسترسی shell دریافت کنید](/docs/tasks/debug/debug-application/get-shell-running-container/)