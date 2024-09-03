```markdown
---
title: دسترسی به خدمات در حال اجرا در خوشه‌ها
content_type: task
weight: 140
---

<!-- مرور -->
این صفحه نحوه‌ی اتصال به خدماتی که در خوشه Kubernetes در حال اجرا هستند را نشان می‌دهد.

## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!-- مراحل -->

## دسترسی به خدمات در حال اجرا در خوشه

در Kubernetes، [گره‌ها](/docs/concepts/architecture/nodes/)، [پادها](/docs/concepts/workloads/pods/) و [خدمات](/docs/concepts/services-networking/service/) هر کدام آی‌پی‌های خود را دارند. در بسیاری از موارد، آی‌پی‌های گره، آی‌پی‌های پاد و برخی آی‌پی‌های خدمات در یک خوشه قابل دسترسی نیستند، بنابراین از بیرون خوشه، مانند دسکتاپ شما، قابل دسترسی نیستند.

### روش‌های اتصال

شما چندین گزینه برای اتصال به گره‌ها، پادها و خدمات از بیرون خوشه دارید:

- دسترسی به خدمات از طریق آی‌پی‌های عمومی.
  - استفاده از خدمات با نوع `NodePort` یا `LoadBalancer` برای دسترسی به خدمات بیرون خوشه. برای اطلاعات بیشتر به [خدمات](/docs/concepts/services-networking/service/) و مستندات [kubectl expose](/docs/reference/generated/kubectl/kubectl-commands/#expose) مراجعه کنید.
  - بسته به محیط خوشه شما، این کار ممکن است تنها خدمت را برای شبکه شرکتی شما فعال کند یا ممکن است آن را به اینترنت ارائه دهد. در نظر داشته باشید که آیا خدمتی که در حال ارائه است ایمن است یا خیر؟ آیا احراز هویت خودش را انجام می‌دهد؟
  - قرار دادن پادها در پشت خدمات. برای دسترسی به یک پاد خاص از مجموعه‌ای از نمونه‌ها، مانند برای اشکال‌زدایی، برچسب منحصر به فردی بر روی پاد قرار دهید و خدمت جدیدی ایجاد کنید که این برچسب را انتخاب کند.
  - در اغلب موارد، برای توسعه‌دهنده برنامه، دسترسی مستقیم به گره‌ها از طریق آی‌پی‌های گره خود لازم نیست.
- دسترسی به خدمات، گره‌ها یا پادها با استفاده از فعل Proxy.
  - قبل از دسترسی به خدمات از راه دور، احراز هویت و اجازه دسترسی به خدمات را از apiserver انجام دهید. از این کار استفاده کنید اگر خدمات برای ارائه به اینترنت کافی ایمن نباشند، یا برای دسترسی به پورت‌های آی‌پی گره یا برای اشکال‌زدایی.
  - پروکسی‌ها ممکن است برای برخی برنامه‌های وب مشکلاتی ایجاد کنند.
  - فقط برای HTTP/HTTPS کار می‌کند.
  - [اینجا](#manually-constructing-apiserver-proxy-urls) توصیف شده است.
- دسترسی از یک گره یا پاد در خوشه.
  - یک پاد را اجرا کنید و سپس به آن از طریق [kubectl exec](/docs/reference/generated/kubectl/kubectl-commands/#exec) وصل شوید. از آن شل، به سایر گره‌ها، پادها و خدمات متصل شوید.
  - برخی از خوشه‌ها ممکن است به شما اجازه دهند که به یک گره در خوشه ssh کنید. از آنجا ممکن است به خدمات خوشه دسترسی پیدا کنید. این یک روش غیر استاندارد است و برخی خوشه‌ها از این روش پشتیبانی می‌کنند و برخی دیگر نه. ممکن است مرورگرها و ابزارهای دیگر نصب شده نباشد. DNS خوشه ممکن است کار نکند.

### کشف خدمات به طور داخلی

معمولاً، چندین خدمت که توسط kube-system روی یک خوشه راه‌اندازی می‌شوند وجود دارد. لیست این‌ها را با دستور `kubectl cluster-info` دریافت کنید:

```shell
kubectl cluster-info
```

خروجی شبیه به این است:

```
Kubernetes master is running at https://192.0.2.1
elasticsearch-logging is running at https://192.0.2.1/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy
kibana-logging is running at https://192.0.2.1/api/v1/namespaces/kube-system/services/kibana-logging/proxy
kube-dns is running at https://192.0.2.1/api/v1/namespaces/kube-system/services/kube-dns/proxy
grafana is running at https://192.0.2.1/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
heapster is running at https://192.0.2.1/api/v1/namespaces/kube-system/services/monitoring-heapster/proxy
```

این نشان می‌دهد که URL پروکسی برای دسترسی به هر خدمت چیست.
برای مثال، این خوشه از نمایه‌گره مربوط به گرید (از Elasticsearch استفاده می‌کند) به آدرس
`https://192.0.2.1/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy/`
می‌رسد اگر مدارک معتبری رد شود، یا از پروکسی kubectl استفاده می‌کند

، برای مثال:
`http://localhost:8080/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy/`.

{{< note >}}
برای کسب اطلاعات بیشتر درباره انتقال به خوشه‌ها با استفاده از API Kubernetes به [Access Clusters Using the Kubernetes API](/docs/tasks/administer-cluster/access-cluster-api/#accessing-the-kubernetes-api) مراجعه کنید.
{{< /note >}}

#### ساخت دستی آدرس‌های پروکسی apiserver

همانطور که در بالا ذکر شد، از دستور `kubectl cluster-info` برای دریافت URL پروکسی خدمت استفاده می‌کنید. برای ایجاد URL‌های پروکسی که شامل نقاط اتصال خدمت، پسوندها و پارامترها هستند، به URL پروکسی خدمت اضافه می‌کنید:
`http://`*`kubernetes_master_address`*`/api/v1/namespaces/`*`namespace_name`*`/services/`*`[https:]service_name[:port_name]`*`/proxy`

اگر نام پورت خود را مشخص نکرده‌اید، نیازی به مشخص کردن *port_name* در URL ندارید. همچنین می‌توانید شماره پورت را به جای *port_name* برای پورت‌های نامی و نام‌ندار استفاده کنید.

به طور پیش‌فرض، سرور API از طریق HTTP به خدمت شما پروکسی می‌کند. برای استفاده از HTTPS، نام خدمت را با `https:` پیش‌فرض کنید:
`http://<kubernetes_master_address>/api/v1/namespaces/<namespace_name>/services/<service_name>/proxy`

فرمت‌های پشتیبانی شده برای بخش `<service_name>` از URL عبارتند از:

* `<service_name>` - پروکسی به پورت پیش‌فرض یا نام‌نامی با استفاده از http
* `<service_name>:<port_name>` - پروکسی به نام پورت مشخص یا شماره پورت مشخص با استفاده از http
* `https:<service_name>:` - پروکسی به پورت پیش‌فرض یا نام‌نامی با استفاده از https (توجه به دونیم سرانجامی)
* `https:<service_name>:<port_name>` - پروکسی به نام پورت مشخص یا شماره پورت مشخص با استفاده از https

##### مثال‌ها

* برای دسترسی به نقطه پایانی خدمت Elasticsearch `_search?q=user:kimchy`، از این استفاده می‌شود:

  ```
  http://192.0.2.1/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy/_search?q=user:kimchy
  ```

* برای دسترسی به اطلاعات سلامتی خوشه Elasticsearch `_cluster/health?pretty=true`، از این استفاده می‌شود:

  ```
  https://192.0.2.1/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy/_cluster/health?pretty=true
  ```

  اطلاعات سلامتی مانند این است:

  ```json
  {
    "cluster_name" : "kubernetes_logging",
    "status" : "yellow",
    "timed_out" : false,
    "number_of_nodes" : 1,
    "number_of_data_nodes" : 1,
    "active_primary_shards" : 5,
    "active_shards" : 5,
    "relocating_shards" : 0,
    "initializing_shards" : 0,
    "unassigned_shards" : 5
  }
  ```

* برای دسترسی به اطلاعات سلامتی خدمت Elasticsearch با استفاده از HTTPS `_cluster/health?pretty=true`، از این استفاده می‌شود:

  ```
  https://192.0.2.1/api/v1/namespaces/kube-system/services/https:elasticsearch-logging:/proxy/_cluster/health?pretty=true
  ```

#### استفاده از مرورگرهای وب برای دسترسی به خدمات در حال اجرا در خوشه

ممکن است بتوانید URL پروکسی apiserver را در نوار آدرس مرورگر قرار دهید. اما:

- مرورگرهای وب معمولاً نمی‌توانند اغلب توکن‌ها را منتقل کنند، بنابراین شاید نیاز به استفاده از احراز هویت اساسی (رمز عبور) داشته باشید.
  apiserver می‌تواند به تنظیمات احراز هویت اساسی پذیرفته شود، اما خوشه شما ممکن است به احراز هویت اساسی پذیرفته نباشد.
- برخی برنامه‌های وب ممکن است کار نکنند، به ویژه آن‌هایی که دارای جاوااسکریپت سمت کلاینت هستند که URL‌ها را به یک روش ساختار داده می‌کنند که از پیشوند مسیر پروکسی آگاه نیستند.
