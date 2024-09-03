---
title: استفاده از Port Forwarding برای دسترسی به برنامه‌ها در یک خوشه
content_type: وظیفه
weight: 40
min-kubernetes-server-version: v1.10
---

<!-- overview -->

این صفحه نحوه استفاده از `kubectl port-forward` را برای اتصال به یک سرور MongoDB که در یک خوشه Kubernetes در حال اجرا است نشان می‌دهد. این نوع اتصال می‌تواند برای اشکال‌زدایی دیتابیس مفید باشد.

## {{% heading "پیش‌نیازها" %}}

* {{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}
* نصب [MongoDB Shell](https://www.mongodb.com/try/download/shell).

<!-- steps -->

## ایجاد Deployment و Service برای MongoDB

1. ایجاد یک Deployment برای اجرای MongoDB:

   ```shell
   kubectl apply -f https://k8s.io/examples/application/mongodb/mongo-deployment.yaml
   ```

   خروجی دستور موفق نشان می‌دهد که Deployment ایجاد شده است:

   ```
   deployment.apps/mongo created
   ```

   برای بررسی آماده‌بودن پاد، وضعیت آن را بررسی کنید:

   ```shell
   kubectl get pods
   ```

   خروجی نشان می‌دهد که پاد ایجاد شده است:

   ```
   NAME                     READY   STATUS    RESTARTS   AGE
   mongo-75f59d57f4-4nd6q   1/1     Running   0          2m4s
   ```

   وضعیت Deployment را بررسی کنید:

   ```shell
   kubectl get deployment
   ```

   خروجی نشان می‌دهد که Deployment ایجاد شده است:

   ```
   NAME    READY   UP-TO-DATE   AVAILABLE   AGE
   mongo   1/1     1            1           2m21s
   ```

   Deployment به صورت خودکار یک ReplicaSet را مدیریت می‌کند.
   برای بررسی وضعیت ReplicaSet از دستور زیر استفاده کنید:

   ```shell
   kubectl get replicaset
   ```

   خروجی نشان می‌دهد که ReplicaSet ایجاد شده است:

   ```
   NAME               DESIRED   CURRENT   READY   AGE
   mongo-75f59d57f4   1         1         1       3m12s
   ```

2. ایجاد یک Service برای افشای MongoDB در شبکه:

   ```shell
   kubectl apply -f https://k8s.io/examples/application/mongodb/mongo-service.yaml
   ```

   خروجی دستور موفق نشان می‌دهد که Service ایجاد شده است:

   ```
   service/mongo created
   ```

   بررسی Service ایجاد شده:

   ```shell
   kubectl get service mongo
   ```

   خروجی نشان می‌دهد که Service ایجاد شده است:

   ```
   NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
   mongo   ClusterIP   10.96.41.183   <none>        27017/TCP   11s
   ```

3. تأیید کنید که سرور MongoDB در Pod اجرا می‌شود و به پورت 27017 گوش می‌دهد:

   ```shell
   # تغییر دهید mongo-75f59d57f4-4nd6q به نام Pod مربوطه
   kubectl get pod mongo-75f59d57f4-4nd6q --template='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'
   ```

   خروجی پورت MongoDB در آن Pod را نشان می‌دهد:

   ```
   27017
   ```

   27017 پورت TCP رسمی برای MongoDB است.

## Forward کردن یک پورت محلی به یک پورت در Pod

1. `kubectl port-forward` اجازه استفاده از نام منبع، مانند نام Pod، برای انتخاب یک Pod مطابقتی برای Forward به آن را می‌دهد.


   ```shell
   # تغییر دهید mongo-75f59d57f4-4nd6q به نام Pod مربوطه
   kubectl port-forward mongo-75f59d57f4-4nd6q 28015:27017
   ```

   که همانند

   ```shell
   kubectl port-forward pods/mongo-75f59d57f4-4nd6q 28015:27017
   ```

   یا

   ```shell
   kubectl port-forward deployment/mongo 28015:27017
   ```

   یا

   ```shell
   kubectl port-forward replicaset/mongo-75f59d57f4 28015:27017
   ```

   یا

   ```shell
   kubectl port-forward service/mongo 28015:27017
   ```

   هر یک از دستورات فوق کار می‌کند. خروجی مشابه زیر است:

   ```
   Forwarding from 127.0.0.1:28015 -> 27017
   Forwarding from [::1]:28015 -> 27017
   ```

   {{< note >}}
   `kubectl port-forward` بازگشت نمی‌دهد. برای ادامه تمرینات، باید یک ترمینال دیگر باز کنید.
   {{< /note >}}

2. شروع رابط خط فرمان MongoDB:

   ```shell
   mongosh --port 28015
   ```

3. در خط فرمان MongoDB، دستور `ping` را وارد کنید:

   ```
   db.runCommand( { ping: 1 } )
   ```

   درخواست ping موفقیت‌آمیز به این صورت باز می‌گردد:

   ```
   { ok: 1 }
   ```

### اختیاری: اجازه دادن به kubectl برای انتخاب پورت محلی {#let-kubectl-choose-local-port}

اگر نیازی به پورت محلی خاص ندارید، می‌توانید به kubectl اجازه دهید پورت محلی را انتخاب کند و به این ترتیب شما را از مدیریت تضاد‌های پورت محلی رها کند:

```shell
kubectl port-forward deployment/mongo :27017
```

ابزار `kubectl` یک شماره پورت محلی پیدا می‌کند که در حال حاضر استفاده نمی‌شود (که از شماره‌های پایین جلوگیری می‌کند، زیرا اینها ممکن است توسط برنامه‌های دیگر استفاده شوند). خروجی مشابه است:

```
Forwarding from 127.0.0.1:63753 -> 27017
Forwarding from [::1]:63753 -> 27017
```

<!-- بحث -->

## بحث

اتصال‌های انجام شده به پورت محلی 28015 به پورت 27017 Pod که MongoDB را اجرا می‌کند، Forward می‌شوند. با این اتصال، می‌توانید از ایستگاه کاری محلی خود برای اشکال‌زدایی دیتابیس استفاده کنید.

{{< note >}}
`kubectl port-forward` فقط برای پورت‌های TCP پیاده‌سازی شده است.
پشتیبانی از پروتکل UDP در [issue 47862](https://github.com/kubernetes/kubernetes/issues/47862) پیگیری می‌شود.
{{< /note >}}

## {{% heading "whatsnext" %}}

برای یادگیری بیشتر درباره [kubectl port-forward](/docs/reference/generated/kubectl/kubectl-commands/#port-forward) مطالعه کنید.
