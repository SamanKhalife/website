---
title: اجرای برنامه Stateful تک‌نمونه
content_type: آموزش
weight: 20
---

<!-- مرور -->

در این صفحه نحوه اجرای یک برنامه stateful تک‌نمونه در Kubernetes با استفاده از PersistentVolume و Deployment نمایش داده می‌شود. برنامه MySQL در نظر گرفته شده است.

## {{% heading "اهداف" %}}

- ایجاد PersistentVolume که به یک دیسک در محیط شما مراجعه می‌کند.
- ایجاد Deployment MySQL.
- افشای MySQL به دیگر پادها در خوشه با یک نام DNS معروف.

## {{% heading "پیش‌نیازها" %}}

- {{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}
- {{< include "default-storage-class-prereqs.md" >}}

<!-- محتوای درس -->

## اجرای MySQL

می‌توانید یک برنامه stateful را با ایجاد یک Deployment Kubernetes و اتصال آن به یک PersistentVolume موجود با استفاده از PersistentVolumeClaim اجرا کنید. به عنوان مثال، این فایل YAML یک Deployment را که MySQL را اجرا می‌کند و به PersistentVolumeClaim مراجعه می‌کند، توصیف می‌کند. این فایل یک mount volume برای /var/lib/mysql تعریف می‌کند و سپس یک PersistentVolumeClaim ایجاد می‌کند که به دنبال یک حجم 20G می‌گردد. این ادعا توسط هر حجم موجود که الزامات را برآورده می‌کند یا توسط یک provisioner پویا، برآورده می‌شود.

توجه: رمز عبور در YAML تعریف شده است و این ناامن است. برای یک راه حل امن، به [Secrets Kubernetes](/docs/concepts/configuration/secret/) مراجعه کنید.

{{% code_sample file="application/mysql/mysql-deployment.yaml" %}}
{{% code_sample file="application/mysql/mysql-pv.yaml" %}}

1. پیاده‌سازی PV و PVC فایل YAML:

   ```shell
   kubectl apply -f https://k8s.io/examples/application/mysql/mysql-pv.yaml
   ```

1. پیاده‌سازی محتویات فایل YAML:

   ```shell
   kubectl apply -f https://k8s.io/examples/application/mysql/mysql-deployment.yaml
   ```

1. نمایش اطلاعات درباره Deployment:

   ```shell
   kubectl describe deployment mysql
   ```

   خروجی مشابه این است:

   ```
   Name:                 mysql
   Namespace:            default
   CreationTimestamp:    Tue, 01 Nov 2016 11:18:45 -0700
   Labels:               app=mysql
   Annotations:          deployment.kubernetes.io/revision=1
   Selector:             app=mysql
   Replicas:             1 desired | 1 updated | 1 total | 0 available | 1 unavailable
   StrategyType:         Recreate
   MinReadySeconds:      0
   Pod Template:
     Labels:       app=mysql
     Containers:
       mysql:
       Image:      mysql:5.6
       Port:       3306/TCP
       Environment:
         MYSQL_ROOT_PASSWORD:      password
       Mounts:
         /var/lib/mysql from mysql-persistent-storage (rw)
     Volumes:
       mysql-persistent-storage:
       Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
       ClaimName:  mysql-pv-claim
       ReadOnly:   false
   Conditions:
     Type          Status  Reason
     ----          ------  ------
     Available     False   MinimumReplicasUnavailable
     Progressing   True    ReplicaSetUpdated
   OldReplicaSets:       <none>
   NewReplicaSet:        mysql-63082529 (1/1 replicas created)
   Events:
     FirstSeen    LastSeen    Count    From                SubobjectPath    Type        Reason            Message
     ---------    --------    -----    ----                -------------    --------    ------            -------
     33s          33s         1        {deployment-controller }             Normal      ScalingReplicaSet Scaled up replica set mysql-63082529 to 1
   ```

1. لیست کردن پادهای ایجاد شده توسط Deployment:

   ```shell
   kubectl get pods -l app=mysql
   ```

   خروجی مشابه این است:

   ```
   NAME                   READY     STATUS    RESTARTS   AGE
   mysql-63082529-2z3ki   1/1       Running   0          3m
   ```

1. بررسی PersistentVolumeClaim:

   ```shell
   kubectl describe pvc mysql-pv-claim
   ```

   خروجی مشابه این است:

   ```
   Name:         mysql-pv-claim
   Namespace:    default
   StorageClass:
   Status:       Bound
   Volume:       mysql-pv-volume
   Labels:       <none>
   Annotations:    pv.kubernetes.io/bind-completed=yes
                   pv.kubernetes.io/bound-by-controller=yes
   Capacity:     20Gi
   Access Modes: RWO
   Events:       <none>
   ```

## دسترسی به نمونه MySQL

فایل YAML پیشین یک سرویس ایجاد می‌کند که به دیگر پادها در خوشه امکان دسترسی به پایگاه داده را می‌دهد. گزینه `clusterIP: None` در سرویس، اسم DNS سرویس را به طور مستقیم به آدرس IP پاد تبدیل می‌کند. این بهینه است زمانی که فقط یک پاد پشت یک سرویس وجود دارد و شما قصد افزایش تعداد پادها را ندارید.

برای اتصال به سرور، یک مشتری MySQL اجرا کنید:

```shell
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -ppassword
```

این دستور یک پاد جدید در خوشه ایجاد می‌کند که یک مشتری MySQL را اجرا کرده و از طریق سرویس به سرور متصل می‌شود. اگر اتصال برقرار شد، به این معنی است که پایگاه داده MySQL شما در حال اجرا است.

```
Waiting for pod default/mysql-client-274442439-zyp6i to be running, status is Pending, pod ready: false
If you don't see a command prompt, try pressing enter.

mysql>
```

## به‌روزرسانی

می‌توانید تصویر یا هر بخش دیگر از Deployment را به طور معمول با دستور `kubectl apply` به‌روزرسانی کنید. در اینجا توجهاتی که به برنامه‌های stateful اختصاص داده می‌شود، می‌تواند مشاهده شود:

- اسکیل کردن برنامه مجاز نیست. این تنظیم برای برنامه‌های تک‌نمونه مناسب است. PersistentVolume زیرین فقط به یک پاد قابل نصب است. برای برنامه‌های stateful خوشه‌ای، به مستندات

 [StatefulSet](/docs/concepts/workloads/controllers/statefulset/) مراجعه کنید.
- استفاده از استراتژی `strategy: Recreate` در فایل تنظیمات Deployment YAML. این دستور به Kubernetes دستور می‌دهد که به‌روزرسانی‌های جریانی را استفاده نکند. به دلیل اینکه نمی‌توان بیشتر از یک پاد به طور همزمان در حال اجرا داشت، استراتژی `Recreate` پیشنهاد می‌شود که پاد اول را قبل از ایجاد یک پاد جدید با پیکربندی به‌روزرسانی شده متوقف کند.

## حذف Deployment

اشیاء ایجاد شده را به نام حذف کنید:

```shell
kubectl delete deployment,svc mysql
kubectl delete pvc mysql-pv-claim
kubectl delete pv mysql-pv-volume
```

اگر یک PersistentVolume را به طور دستی فراهم کردید، نیاز است که آن را به طور دستی حذف کنید و منبع زیرین را آزاد کنید. اگر از یک provisioner پویا استفاده کرده‌اید، به طور خودکار PersistentVolume را حذف می‌کند هنگامی که مشاهده می‌کند که PersistentVolumeClaim حذف شده است. برخی از provisioner‌های پویا (مانند EBS و PD) همچنین منبع زیرین را پس از حذف PersistentVolume رها می‌کنند.

## {{% heading "چی بعد" %}}

- بیشتر در مورد [اشیاء Deployment](/docs/concepts/workloads/controllers/deployment/) بیاموزید.
- بیشتر در مورد [اجرای برنامه‌ها](/docs/tasks/run-application/run-stateless-application-deployment/) بیاموزید.
- [مستندات kubectl run](/docs/reference/generated/kubectl/kubectl-commands/#run)
- [Volumes](/docs/concepts/storage/volumes/) و [Persistent Volumes](/docs/concepts/storage/persistent-volumes/)
