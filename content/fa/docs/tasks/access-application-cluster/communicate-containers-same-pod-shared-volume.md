---
title: ارتباط بین کانتینرها در یک پاد با استفاده از یک حجم مشترک
content_type: task
weight: 120
---

<!-- overview -->

این صفحه نشان می‌دهد که چگونه می‌توان از یک حجم (Volume) برای ارتباط بین دو کانتینری که در یک پاد (Pod) در حال اجرا هستند استفاده کرد. همچنین نحوه اجازه دادن به فرآیندها برای ارتباط از طریق [اشتراک‌گذاری فضای نام فرآیند](/docs/tasks/configure-pod-container/share-process-namespace/) بین کانتینرها را مشاهده کنید.

## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!-- steps -->

## ایجاد پادی که دو کانتینر را اجرا می‌کند

در این تمرین، شما یک پاد ایجاد می‌کنید که دو کانتینر را اجرا می‌کند. این دو کانتینر یک حجم مشترک دارند که می‌توانند از آن برای ارتباط استفاده کنند. در اینجا فایل پیکربندی برای پاد آورده شده است:

{{% code_sample file="pods/two-container-pod.yaml" %}}

در فایل پیکربندی، می‌توانید ببینید که پاد یک حجم به نام `shared-data` دارد.

اولین کانتینری که در فایل پیکربندی ذکر شده است، یک سرور nginx اجرا می‌کند. مسیر نصب برای حجم مشترک `/usr/share/nginx/html` است.
کانتینر دوم بر اساس تصویر debian است و مسیر نصب آن `/pod-data` می‌باشد. کانتینر دوم دستور زیر را اجرا می‌کند و سپس خاتمه می‌یابد:

    echo Hello from the debian container > /pod-data/index.html

توجه داشته باشید که کانتینر دوم فایل `index.html` را در دایرکتوری ریشه سرور nginx می‌نویسد.

پاد و دو کانتینر را ایجاد کنید:

    kubectl apply -f https://k8s.io/examples/pods/two-container-pod.yaml

اطلاعات مربوط به پاد و کانتینرها را مشاهده کنید:

    kubectl get pod two-containers --output=yaml

در اینجا بخشی از خروجی آورده شده است:

    apiVersion: v1
    kind: Pod
    metadata:
      ...
      name: two-containers
      namespace: default
      ...
    spec:
      ...
      containerStatuses:

      - containerID: docker://c1d8abd1 ...
        image: debian
        ...
        lastState:
          terminated:
            ...
        name: debian-container
        ...

      - containerID: docker://96c1ff2c5bb ...
        image: nginx
        ...
        name: nginx-container
        ...
        state:
          running:
        ...

می‌توانید ببینید که کانتینر debian خاتمه یافته و کانتینر nginx همچنان در حال اجرا است.

یک شل به کانتینر nginx بگیرید:

    kubectl exec -it two-containers -c nginx-container -- /bin/bash

در شل خود، مطمئن شوید که nginx در حال اجرا است:

    root@two-containers:/# apt-get update
    root@two-containers:/# apt-get install curl procps
    root@two-containers:/# ps aux

خروجی مشابه این است:

    USER       PID  ...  STAT START   TIME COMMAND
    root         1  ...  Ss   21:12   0:00 nginx: master process nginx -g daemon off;

به یاد داشته باشید که کانتینر debian فایل `index.html` را در دایرکتوری ریشه nginx ایجاد کرده است. از `curl` برای ارسال یک درخواست GET به سرور nginx استفاده کنید:

```
root@two-containers:/# curl localhost
```

خروجی نشان می‌دهد که nginx یک صفحه وب نوشته شده توسط کانتینر debian را سرو می‌کند:

```
Hello from the debian container
```

<!-- discussion -->

## بحث

دلیل اصلی اینکه پادها می‌توانند چندین کانتینر داشته باشند، پشتیبانی از برنامه‌های کمکی است که به برنامه اصلی کمک می‌کنند. مثال‌های معمولی از برنامه‌های کمکی، جمع‌آوری‌کنندگان داده، ارسال‌کنندگان داده و پروکسی‌ها هستند.
برنامه‌های کمکی و اصلی اغلب نیاز به ارتباط با یکدیگر دارند. معمولاً این کار از طریق یک سیستم فایل مشترک انجام می‌شود، همانطور که در این تمرین نشان داده شده است، یا از طریق واسط شبکه حلقه‌دار، localhost. یک مثال از این الگو، یک سرور وب همراه با یک برنامه کمکی است که یک مخزن Git را برای به‌روزرسانی‌های جدید بررسی می‌کند.

حجم در این تمرین روشی برای ارتباط کانتینرها در طول عمر پاد فراهم می‌کند. اگر پاد حذف و دوباره ایجاد شود، هر داده‌ای که در حجم مشترک ذخیره شده است، از بین می‌رود.

## {{% heading "مرحله بعدی" %}}

* بیشتر درباره [الگوهای کانتینرهای ترکیبی](/blog/2015/06/the-distributed-system-toolkit-patterns/) بیاموزید.

* درباره [کانتینرهای ترکیبی برای معماری مدولار](https://www.slideshare.net/Docker/slideshare-burns) مطالعه کنید.

* [پیکربندی یک پاد برای استفاده از یک حجم برای ذخیره‌سازی](/docs/tasks/configure-pod-container/configure-volume-storage/) را مشاهده کنید.

* [پیکربندی یک پاد برای اشتراک‌گذاری فضای نام فرآیند بین کانتینرها در یک پاد](/docs/tasks/configure-pod-container/share-process-namespace/) را مشاهده کنید.

* [حجم](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#volume-v1-core) را ببینید.

* [پاد](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#pod-v1-core) را ببینید.
