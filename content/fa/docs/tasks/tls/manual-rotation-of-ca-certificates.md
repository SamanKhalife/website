---
title: چرخش دستی گواهی‌نامه‌های CA
content_type: task
---

<!-- overview -->

این صفحه نحوه چرخش دستی گواهی‌نامه‌های مرکز اعتبار (CA) را نشان می‌دهد.

## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}}

- برای اطلاعات بیشتر درباره احراز هویت در Kubernetes، به [Authenticating](/docs/reference/access-authn-authz/authentication) مراجعه کنید.
- برای اطلاعات بیشتر درباره بهترین روش‌های گواهی‌نامه‌های CA، به [Single root CA](/docs/setup/best-practices/certificates/#single-root-ca) مراجعه کنید.

<!-- steps -->

## چرخش دستی گواهی‌نامه‌های CA

{{< caution >}}
مطمئن شوید که پوشه گواهی‌نامه خود را همراه با فایل‌های پیکربندی و هر فایل مورد نیاز دیگری پشتیبان‌گیری کرده‌اید.

این رویکرد فرض می‌شود که کنترل پلن Kubernetes در یک تنظیم HA با چندین سرور API عمل می‌کند. قطعی مهربانانه از سرور API نیز فرض شده است تا مشتریان بتوانند به طور کامل از یک سرور API قطعی شده جدا شوند و به سرور دیگری متصل شوند.

پیکربندی‌هایی با یک سرور API تجربه در دسترس‌نبودن در حالی که سرور API دوباره راه‌اندازی می‌شود را تجربه خواهند کرد.
{{< /caution >}}

1. گواهی‌نامه‌های جدید CA و کلید‌های خصوصی (برای مثال: `ca.crt`، `ca.key`، `front-proxy-ca.crt` و `front-proxy-ca.key`) را به تمام نودهای کنترل پلن خود در پوشه گواهی‌نامه‌های Kubernetes توزیع کنید.

1. پرچم `--root-ca-file` را برای {{< glossary_tooltip term_id="kube-controller-manager" >}} بروزرسانی کنید تا شامل CA‌های قدیمی و جدید شود، سپس kube-controller-manager را دوباره راه‌اندازی کنید.

    هر {{< glossary_tooltip text="ServiceAccount" term_id="service-account" >}} که پس از این نقطه ایجاد می‌شود، سری‌هایی شامل هر دو CA قدیمی و جدید دریافت می‌کند.

    {{< note >}}
    فایل‌های مشخص شده توسط پرچم‌های kube-controller-manager `--client-ca-file` و `--cluster-signing-cert-file` نمی‌توانند بسته‌های CA باشند. اگر این پرچم‌ها و `--root-ca-file` به همان فایل `ca.crt` اشاره می‌کنند که در حال حاضر یک بسته است (شامل هر دو CA قدیمی و جدید) شما با یک خطا روبرو خواهید شد. برای حل این مشکل می‌توانید CA جدید را به یک فایل جداگانه کپی کرده و پرچم‌های `--client-ca-file` و `--cluster-signing-cert-file` را به این کپی اشاره دهید. هنگامی که `ca.crt` دیگر یک بسته نیست، می‌توانید پرچم‌های مشکل را به `ca.crt` اشاره دهید و کپی را حذف کنید.

    مشکل 1350 برای kubeadm یک مشکل با kube-controller-manager است که نمی‌تواند یک بسته CA را پذیرفته است.
    {{< /note >}}

1. منتظر بمانید تا مدیر کنترل‌گر گواهی‌نامه `ca.crt` را در Secrets حساب‌های خدمات به‌روز کند تا شامل گواهی‌نامه‌های CA قدیمی و جدید شود.

    اگر هرگونه پادها قبل از استفاده از CA جدید توسط سرورهای API شروع شوند، پادهای جدید این به‌روزرسانی را دریافت می‌کنند و به هر دو CA قدیمی و جدید اعتماد می‌کنند.

1. تمامی پادهای استفاده‌کننده از پیکربندی‌های داخلی خود راه‌اندازی مجدد کنید (برای مثال: kube-proxy، CoreDNS و غیره) تا بتوانند از اطلاعات اعتبار مرکز اعتبار به‌روز شده از Secrets استفاده کنند که به حساب‌های خدمات متصل می‌شوند.

   * مطمئن شوید که CoreDNS، kube-proxy و سایر پادهایی که از پیکربندی‌های داخلی استفاده می‌کنند، به‌طور موردنیاز کار می‌کنند.

1. CA قدیمی و جدید را به فایل برابر `--client-ca-file` و `--kubelet-certificate-authority` در پیکربندی `kube-apiserver` اضافه کنید.

1. CA قدیمی و جدید را به فایل برابر `--client-ca-file` در پیکربندی `kube-scheduler` اضافه کنید.

1. گواهی‌نامه‌ها را برای حساب‌های کاربری با جایگزینی محتوای `client-certificate-data` و `client-key-data` به‌روز کنید.

   برای اطلاعات بیشتر درباره ایجاد گواهی‌نامه برای حساب‌های کاربری، به [Configure certificates for user accounts](/docs/setup/best-practices/certificates/#configure-certificates-for-user-accounts) مراجعه کنید.

   همچنین، بخش `certificate-authority-data` را در فایل‌های kubeconfig به‌طور مجزا با داده‌های Base64-کد شده گواهی‌نامه قدیمی و جدید به‌روز کنید.

1. پرچم `--root-ca-file` را برای {{< glossary_tooltip term_id="cloud-controller

-manager" >}} بروزرسانی کنید تا شامل هر دو CA قدیمی و جدید شود، سپس cloud-controller-manager را دوباره راه‌اندازی کنید.

   {{< note >}}
   اگر خوشه شما دارای یک cloud-controller-manager نیست، می‌توانید این مرحله را نادیده بگیرید.
   {{< /note >}}

1. به روش زیر به صورت پیش‌رونده اقدام کنید.

   1. سایر سرورهای API مجتمع یا کنترل کننده‌های webhook را دوباره راه‌اندازی کنید تا به گواهی‌نامه‌های CA جدید اعتماد کنند.

   1. kubelet را با به‌روز کردن فایل برابر `clientCAFile` در پیکربندی kubelet و `certificate-authority-data` در `kubelet.conf` برای استفاده از هر دو CA قدیمی و جدید در تمام نودها دوباره راه‌اندازی کنید.

      اگر kubelet شما از چرخش گواهی‌نامه مشتری استفاده نمی‌کند، فایل‌های `client-certificate-data` و `client-key-data` را در `kubelet.conf` در تمام نودها به‌روز کنید همچنین فایل گواهی‌نامه kubelet که معمولاً در `/var/lib/kubelet/pki` پیدا می‌شود.

   1. سرورهای API را با استفاده از گواهی‌نامه‌های (`apiserver.crt`، `apiserver-kubelet-client.crt` و `front-proxy-client.crt`) به‌روز کنید که توسط CA جدید امضا شده‌اند.
      می‌توانید از کلید‌های خصوصی موجود یا کلید‌های خصوصی جدید استفاده کنید.
      اگر کلید‌های خصوصی را تغییر داده‌اید، آن‌ها را هم در پوشه گواهی‌نامه‌های Kubernetes به‌روز کنید.

      از آنجا که پادهای خوشه شما از هر دو CA قدیمی و جدید اعتماد می‌کنند، پس اتصال موقتی وجود خواهد داشت که مشتریان Kubernetes پادهای جدید به سرور API جدید متصل می‌شوند.
      سرور API جدید از گواهی‌نامه‌ای استفاده می‌کند که توسط CA جدید امضا شده است.

      * kube-scheduler را برای استفاده و اعتماد به CA جدید دوباره راه‌اندازی کنید.
      * مطمئن شوید که مولفه‌های کنترل پلن هیچ خطاهای TLSی گزارش نمی‌دهند.

      {{< note >}}
      برای تولید گواهی‌نامه‌ها و کلید‌های خصوصی برای خوشه خود با استفاده از ابزار خط فرمان `openssl`، به [Certificates (`openssl`)](/docs/tasks/administer-cluster/certificates/#openssl) مراجعه کنید.
      همچنین می‌توانید از [`cfssl`](/docs/tasks/administer-cluster/certificates/#cfssl) استفاده کنید.
      {{< /note >}}

    1. برای انجام تعمیر به شیوه پیشروی اقدام کنید.

      ```shell
      for namespace in $(kubectl get namespace -o jsonpath='{.items[*].metadata.name}'); do
          for name in $(kubectl get deployments -n $namespace -o jsonpath='{.items[*].metadata.name}'); do
              kubectl patch deployment -n ${namespace} ${name} -p '{"spec":{"template":{"metadata":{"annotations":{"ca-rotation": "1"}}}}}';
          done
          for name in $(kubectl get daemonset -n $namespace -o jsonpath='{.items[*].metadata.name}'); do
              kubectl patch daemonset -n ${namespace} ${name} -p '{"spec":{"template":{"metadata":{"annotations":{"ca-rotation": "1"}}}}}';
          done
      done
      ```

      {{< note >}}
      برای محدود کردن تعداد اختلالات هم‌زمان که برنامه شما تجربه می‌کند، به [configure pod disruption budget](/docs/tasks/run-application/configure-pdb/) مراجعه کنید.
      {{< /note >}}

1. اگر خوشه خود از token‌های bootstrap برای پیوستن نودها استفاده می‌کند، ConfigMap `cluster-info` را در فضای‌نامه `kube-public` با CA جدید به‌روز کنید.

   ```shell
   base64_encoded_ca="$(base64 -w0 /etc/kubernetes/pki/ca.crt)"

   kubectl get cm/cluster-info --namespace kube-public -o yaml | \
       /bin/sed "s/\(certificate-authority-data:\).*/\1 ${base64_encoded_ca}/" | \
       kubectl apply -f -
   ```

1. عملکرد خوشه را تأیید کنید.

    1. لاگ‌ها از مولفه‌های کنترل پلن، به همراه kubelet و kube-proxy را بررسی کنید.
       مطمئن شوید که این مولفه‌ها هیچ خطای TLSی گزارش نمی‌دهند؛ برای جزئیات بیشتر به [looking at the logs](/docs/tasks/debug/debug-cluster/#looking-at-logs) مراجعه کنید.

    1. لاگ‌ها از هرگونه سرور API مجتمع و پادهای استفاده‌کننده از پیکربندی‌های داخلی را تأیید کنید.

1. بعد از تأیید موفقیت‌آمیز عملکرد خوشه:

   1. تمام token‌های حساب‌های خدمات را به‌روزرسانی کنید تا فقط شامل گواهی‌نامه CA جدید باشد.

      * همه پادهایی که از kubeconfig داخلی استفاده می‌کنند در نهایت باید بازآغاز شوند تا Secret جدید را بگیرند، به‌طوری که هیچ پادی به گواهی‌نامه CA قدیمی اعتماد نکند.

   1. مولفه‌های کنترل پلن را با حذف گواهی‌نامه CA قدیمی از فایل‌های kubeconfig و فایل‌های مقابل پرچم‌های `--client-ca-file`، `--root-ca-file` به‌روز کنید.

   1. بر روی هر نود، kubelet را با حذف گواهی

‌نامه CA قدیمی از فایل مقابل `clientCAFile` و از فایل kubeconfig kubelet دوباره راه‌اندازی کنید. این را به عنوان یک به‌روزرسانی پیمایشی انجام دهید.

      اگر خوشه شما به شما اجازه می‌دهد این تغییر را انجام دهید، همچنین می‌توانید آن را با جایگزینی نودها به جای تنظیم مجدد آنها بیشتر ارزیابی کنید.

