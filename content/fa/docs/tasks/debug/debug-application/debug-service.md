---
reviewers:
- thockin
- bowei
content_type: task
title: اشکال‌زدایی سرویس‌ها
weight: 20
---

<!-- overview -->
یکی از مسائلی که برای نصب‌های جدید Kubernetes به طور متداول پیش می‌آید، عدم کارکرد درست یک سرویس است. شما پادهای خود را از طریق یک Deployment (یا کنترل‌گر بارگیری کار) اجرا کرده و یک سرویس ایجاد کرده‌اید، اما وقتی تلاش می‌کنید به آن دسترسی پیدا کنید، پاسخی دریافت نمی‌کنید. این سند به شما کمک می‌کند تا بفهمید مشکل از کجاست.

<!-- body -->

## اجرای دستورات در یک پاد

برای بسیاری از مراحل اینجا، شما می‌خواهید ببینید که یک پاد در خوشه چه می‌بیند. ساده‌ترین راه برای این کار اجرای یک پاد تعاملی busybox است:

```none
kubectl run -it --rm --restart=Never busybox --image=gcr.io/google-containers/busybox sh
```

{{< note >}}
اگر پرامتر دستوری را نمی‌بینید، سعی کنید کلید Enter را فشار دهید.
{{< /note >}}

اگر پادی در حال اجرا دارید که ترجیح می‌دهید استفاده کنید، می‌توانید یک دستور در آن اجرا کنید با استفاده از:

```shell
kubectl exec <نام-پاد> -c <نام-کانتینر> -- <دستور>
```

## راه‌اندازی

برای اهداف این راهنما، بیایید چند پاد اجرا کنیم. اگرچه شما در حال اشکال‌زدایی سرویس خود هستید می‌توانید جزئیات خود را جایگزین کنید، یا می‌توانید همراه شوید و نقطه داده دوم را بگیرید.

```shell
kubectl create deployment hostnames --image=registry.k8s.io/serve_hostname
```
```none
deployment.apps/hostnames created
```

دستورات `kubectl` نوع و نام منبع ایجاد شده یا تغییر یافته را چاپ می‌کنند که می‌تواند در دستورات بعدی استفاده شود.

حالا ما می‌خواهیم این Deployment را به ۳ کپی افزایش دهیم.

```shell
kubectl scale deployment hostnames --replicas=3
```
```none
deployment.apps/hostnames scaled
```

توجه داشته باشید که این مانند آن است که شما با استفاده از YAML زیر Deployment را آغاز کرده‌اید:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hostnames
  name: hostnames
spec:
  selector:
    matchLabels:
      app: hostnames
  replicas: 3
  template:
    metadata:
      labels:
        app: hostnames
    spec:
      containers:
      - name: hostnames
        image: registry.k8s.io/serve_hostname
```

برچسب "app" به طور خودکار توسط `kubectl create deployment` به نام Deployment تنظیم می‌شود.

شما می‌توانید تأیید کنید که پادهای شما در حال اجرا هستند:

```shell
kubectl get pods -l app=hostnames
```
```none
NAME                        READY     STATUS    RESTARTS   AGE
hostnames-632524106-bbpiw   1/1       Running   0          2m
hostnames-632524106-ly40y   1/1       Running   0          2m
hostnames-632524106-tlaok   1/1       Running   0          2m
```

شما همچنین می‌توانید تأیید کنید که پادهای شما در حال خدمت‌رسانی هستند. شما می‌توانید لیست آدرس‌های IP پادها را بگیرید و آن‌ها را مستقیماً تست کنید.

```shell
kubectl get pods -l app=hostnames \
    -o go-template='{{range .items}}{{.status.podIP}}{{"\n"}}{{end}}'
```
```none
10.244.0.5
10.244.0.6
10.244.0.7
```

کانتینر مثال استفاده شده برای این راهنما نام خود را از طریق HTTP در پورت 9376 ارائه می‌دهد، اما اگر شما در حال اشکال‌زدایی برنامه‌ی خود هستید، باید از شماره پورتی که پادهای شما در آن گوش می‌دهند، استفاده کنید.

از داخل یک پاد:

```shell
for ep in 10.244.0.5:9376 10.244.0.6:9376 10.244.0.7:9376; do
    wget -qO- $ep
done
```

این باید مانند این باشد:

```
hostnames-632524106-bbpiw
hostnames-632524106-ly40y
hostnames-632524106-tlaok
```

اگر در این نقطه پاسخ‌هایی که انتظار داشتید دریافت نمی‌کنید، پادهای شما ممکن است سالم نباشند یا ممکن است بر روی پورتی که فکر می‌کنید گوش می‌دهند، گوش نکنند. شما ممکن است ببینید که استفاده از `kubectl logs` برای دیدن اتفاقات مفید است، یا شاید نیاز به `kubectl exec` برای ورود مستقیم به پادهای خود داشته باشید و از آنجا اشکال‌زدایی کنید.

با فرض این که تا الان همه چیز به نقطه مطلوب رسیده است، می‌توانید شروع به بررسی دلیل عدم کارکرد سرویس خود کنید.

## آیا سرویس وجود دارد؟

قارئ دقیق متوجه می‌شود که شما واقعاً یک سرویس ایجاد نکرده‌اید - این امر قصدی است. این یک مرحله است که گاهی اوقات فراموش می‌شود و اولین چیزی است که باید بررسی شود.

اگر سعی کنید به یک سرویس غیر وجودی دسترسی پیدا کنید، چه اتفاقی می‌افتد؟ اگر یک پاد دیگر وجود داشته باشد که از این سرویس با نام استفاده می‌کند، شما چیزی شبیه به زیر دریافت خواهید کرد:

```shell
wget -O- hostnames
```
```none
Resolving hostnames (hostnames)... failed: Name or service not known.
wget: unable to resolve host address 'hostnames'
```

اولین چیزی که باید بررسی کنید این است که آیا آن سرویس واقعاً وجود دارد یا خیر:

```shell
kubectl get svc hostnames
```
```none
No resources found.
Error from server (NotFound): services "hostnames" not found
```

بیایید سرویس را ایجاد کنیم. همانند قبل، این برای راهنمایی است - می‌توانید جزئیات سرویس خود را اینجا استفاده کنید.

```shell
kubectl expose deployment hostnames --port=80 --target-port=9376
```
```none
service/hostnames exposed
```

و بخوانید آن را دوباره:

```shell
kubectl get svc hostnames
```
```none
NAME        TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
hostnames   ClusterIP   10.0.1.175   <none>        80/TCP    5s
```

حالا می‌دانید که سرویس وجود دارد.

مانند قبل، این همانند آن است که شما سرویس را با YAML زیر آغاز کرده‌اید:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hostnames
  name: hostnames
spec:
  selector:
    app: hostnames
  ports:
  - name: default
    protocol: TCP
    port: 80
    targetPort: 9376
```

برای برجسته کردن تمام محدوده پیکربندی، سرویسی که در اینجا ایجاد می‌کنید، از یک شماره پورت متفاوت نسبت به پادها استفاده می‌کند. برای بسیاری از سرویس‌های واقعی، این مقادیر ممکن است یکسان باشند.

## آیا قوانین ورود شبکه Network Policy بر روی پادهای هدف تأثیر می‌گذارد؟

اگر قوانین ورود شبکه Network Policy را برای ترافیک ورودی به پادهای "hostnames-*" پیاده‌سازی کرده‌اید، باید آن‌ها را مرور کنید.

لطفاً برای اطلاعات بیشتر به [Network Policies](/docs/concepts/services-networking/network-policies/) مراجعه کنید.

## آیا سرویس از طریق نام DNS کار می‌کند؟

یکی از راه‌های متداول که مشتریان از یک سرویس استفاده می‌کنند، از طریق یک نام DNS است.

از یک پاد در همان فضای نام استفاده می‌کنیم:

```shell
nslookup hostnames
```
```none
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local

Name:      hostnames
Address 1: 10.0.1.175 hostnames.default.svc.cluster.local
```

اگر این عملیات ناموفق باشد، شاید پاد و سرویس شما در فضای نام‌های مختلف باشند، امتحان کنید نام کامل فضای نام را استفاده کنید (دوباره، از داخل یک پاد):

```shell
nslookup hostnames.default
```
```none
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local

Name:      hostnames.default
Address 1: 10.0.1.175 hostnames.default.svc.cluster.local
```

اگر این کار موفق باشد، باید برنامه‌ی خود را برای استفاده از نام متقاطع فضای نام تنظیم کنید یا برنامه و سرویس خود را در یک فضای نام یکسان اجرا کنید. اگر این همچنان ناموفق باشد، امتحان کنید نام کامل (fully-qualified) را استفاده کنید:

```shell
nslookup hostnames.default.svc.cluster.local
```
```none
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local

Name:      hostnames.default.svc.cluster.local
Address 1: 10.0.1.175 hostnames.default.svc.cluster.local
```

در اینجا پسوند "default.svc.cluster.local" توجه کنید. "default" فضای نامی است که در آن عمل می‌کنید. "svc" نشان می‌دهد که این یک سرویس است. "cluster.local" دامنه خوشه شما است که ممکن است در خوشه خود متفاوت باشد.

همچنین می‌توانید این کار را از یک Node در خوشه انجام دهید:

{{< note >}}
10.0.0.10 آدرس IP سرویس DNS خوشه شماست، ممکن است برای شما متفاوت باشد.
{{< /note >}}

```shell
nslookup hostnames.default.svc.cluster.local 10.0.0.10
```
```none
Server:         10.0.0.10
Address:        10.0.0.10#53

Name:   hostnames.default.svc.cluster.local
Address: 10.0.1.175
```

اگر شما موفق به انجام جستجوی نام کامل شده اما نتوانستید با نام نسبی انجام دهید، باید بررسی کنید که فایل `/etc/resolv.conf` در پاد شما صحیح باشد. از داخل یک پاد:

```shell
cat /etc/resolv.conf
```

باید چیزی شبیه به این ببینید:

```
nameserver 10.0.0.10
search default.svc.cluster.local svc.cluster.local cluster.local example.com
options ndots:5
```



خط `nameserver` باید IP سرویس DNS خوشه شما را نشان دهد. این با پارامتر `--cluster-dns` به `kubelet` منتقل می‌شود.

خط `search` باید شامل پسوند مناسبی باشد تا شما بتوانید نام سرویس را پیدا کنید. در این حالت، به دنبال سرویس‌ها در فضای نام محلی ("default.svc.cluster.local")، سرویس‌ها در تمام فضای نام‌ها ("svc.cluster.local") و در آخر برای نام‌های خوشه ("cluster.local") می‌گردد. بسته به نصب خود ممکن است شما دارای رکوردهای اضافی بعد از آن باشید (حداکثر 6 رکورد). پسوند خوشه به `kubelet` با پارامتر `--cluster-domain` منتقل می‌شود. در این سند، فرض بر این است که پسوند خوشه "cluster.local" است. خوشه خود ممکن است به طور متفاوت پیکربندی شده باشد، در این صورت باید آن را در تمام دستورات قبلی تغییر دهید.

خط `options` باید `ndots` را به اندازه کافی بالا تنظیم کند که کتابخانه مشتری DNS شما مسیرهای جستجو را در نظر بگیرد. Kubernetes این را به طور پیش‌فرض به 5 تنظیم می‌کند که به اندازه کافی بالاست تا تمام نام‌های DNS تولید شده توسط آن را پوشش دهد.

### آیا هر سرویسی با نام DNS کار می‌کند؟ {#does-any-service-exist-in-dns}

اگر عملیات بالا همچنان ناموفق باشد، به نظر می‌رسد جستجوی DNS برای سرویس شما کار نمی‌کند. شما می‌توانید یک قدم عقب برگردید و ببینید که چه چیز دیگری کار نمی‌کند. سرویس اصلی Kubernetes باید همیشه کار کند. از داخل یک پاد:

```shell
nslookup kubernetes.default
```
```none
Server:    10.0.0.10
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes.default
Address 1: 10.0.0.1 kubernetes.default.svc.cluster.local
```

اگر این عملیات ناموفق باشد، لطفاً به بخش [kube-proxy](#is-the-kube-proxy-working) این سند مراجعه کنید، یا حتی به ابتدای این سند برگردید و از ابتدا شروع کنید، اما به جای اشکال‌زدایی سرویس خود، DNS سرویس را اشکال‌زدایی کنید.

## آیا سرویس با آدرس IP کار می‌کند؟

فرض کنید که تایید کرده‌اید که DNS کار می‌کند، چیز بعدی که باید تست شود این است که آیا سرویس شما با آدرس IP خود کار می‌کند. از یک پاد در خوشه خود، به IP سرویس دسترسی پیدا کنید (از دستور `kubectl get` بالا).

```shell
for i in $(seq 1 3); do
    wget -qO- 10.0.1.175:80
done
```

باید چیزی شبیه به این ببینید:

```
hostnames-632524106-bbpiw
hostnames-632524106-ly40y
hostnames-632524106-tlaok
```

اگر سرویس شما کار می‌کند، باید پاسخ‌های صحیح دریافت کنید. اگر نه، تعدادی از مسائل ممکن است در حال وقوع باشند. ادامه بخوانید.

## آیا سرویس به درستی تعریف شده است؟

ممکن است احساس کودکانه باشد، اما شما واقعاً باید بررسی کنید که سرویس شما درست است و با پورت پاد شما مطابقت دارد. سرویس خود را بررسی کنید و تأیید کنید:

```shell
kubectl get service hostnames -o json
```
```json
{
    "kind": "Service",
    "apiVersion": "v1",
    "metadata": {
        "name": "hostnames",
        "namespace": "default",
        "uid": "428c8b6c-24bc-11e5-936d-42010af0a9bc",
        "resourceVersion": "347189",
        "creationTimestamp": "2015-07-07T15:24:29Z",
        "labels": {
            "app": "hostnames"
        }
    },
    "spec": {
        "ports": [
            {
                "name": "default",
                "protocol": "TCP",
                "port": 80,
                "targetPort": 9376,
                "nodePort": 0
            }
        ],
        "selector": {
            "app": "hostnames"
        },
        "clusterIP": "10.0.1.175",
        "type": "ClusterIP",
        "sessionAffinity": "None"
    },
    "status": {
        "loadBalancer": {}
    }
}
```

* آیا پورت سرویسی که سعی می‌کنید به آن دسترسی پیدا کنید در `spec.ports[]` لیست شده است؟
* آیا `targetPort` برای پادهای شما صحیح است (برخی از پادها از پورت متفاوتی نسبت به سرویس استفاده می‌کنند؟)
* اگر قصد دارید از یک پورت عددی استفاده کنید، آیا این عدد عدد (9376) است یا رشته "9376"؟
* اگر قصد داشتید از یک پورت نامگذاری شده استفاده کنید، آیا پادهای شما یک پورت با همان نام را ارائه می‌دهند؟
* آیا پروتکل پورت برای پادهای شما صحیح است؟

## آیا سرویس هرگونه Endpoints دارد؟

اگر تا اینجا رسیده‌اید، تأیید کرده‌اید که سرویس شما به درستی تعریف شده است و توسط DNS حل می‌شود. حالا بیایید بررسی کنیم که آیا پادهایی که اجرا کرده‌اید واقعاً توسط سرویس انتخاب می‌شوند.

قبلاً دیدید که پادها در حال اجرا بودند. می‌توانید دوباره بررسی کنید:

```shell
kubectl get pods -l app=hostnames
```
```none
NAME                        READY     STATUS    RESTARTS   AGE
hostnames-632524106-bbpiw   1/1       Running   0          1h
hostnames-632524106-ly40y   1/1       Running   0          1h
hostnames-632524106-tlaok   1/1       Running   0          1h
```

آرگومان `-l app=hostnames` یک انتخاب‌گر برچسب روی سرویس تنظیم شده است.

ستون "AGE" نشان می‌دهد که این پادها حدود یک ساعت است که در حال اجرا هستند، که نشان می‌دهد که آنها به درستی اجرا می‌شوند و نمی‌افتند.

ستون "RESTARTS" نشان می‌دهد که این پادها به طور متداول یا پرتکرار برگشت می‌کنند. بازگشت‌های مکرر می‌تواند منجر به مشکلات اتصال موقت شود. اگر شمارش بازنشسته‌ها بالا باشد، بیشتر درباره [اشکال‌زدایی پادها](/docs/tasks/debug/debug-application/debug-pods) بخوانید.

درون یک حلقه کنترلی در سیستم Kubernetes، انتخاب‌کننده هر سرویس را ارزیابی م

ی‌کند و نتایج را در یک شی Endpoints متناظر ذخیره می‌کند.

```shell
kubectl get endpoints hostnames

NAME        ENDPOINTS
hostnames   10.244.0.5:9376,10.244.0.6:9376,10.244.0.7:9376
```

این تأیید می‌کند که کنترل‌کننده Endpoints پادهای صحیح را برای سرویس شما پیدا کرده است. اگر ستون `ENDPOINTS` `<none>` باشد، باید بررسی کنید که فیلد `spec.selector` سرویس شما در واقع برای مقادیر `metadata.labels` در پادهای شما انتخاب می‌کند. یک اشتباه معمول این است که تایپو یا خطای دیگر، مانند انتخاب سرویس برای `app=hostnames`، اما انتشار `run=hostnames` در نسخه‌های قبل از 1.18، که دستور `kubectl run` می‌تواند برای ایجاد یک اجرا استفاده شود.

## آیا پادها کار می‌کنند؟

در این نقطه، شما می‌دانید که سرویس شما وجود دارد و پادهای شما را انتخاب کرده است. در ابتدای این راهنما، شما پیش از این پادهای خود را تأیید کرده‌اید. بیایید دوباره بررسی کنیم که آیا پادها واقعاً کار می‌کنند - می‌توانید از مکانیزم سرویس عبور کنید و مستقیماً به پادها بروید، همانطور که در Endpoints ذکر شده است.

{{< note >}}
این دستورات از پورت پاد (9376) استفاده می‌کنند، به جای پورت سرویس (80).
{{< /note >}}

از داخل یک پاد:

```shell
for ep in 10.244.0.5:9376 10.244.0.6:9376 10.244.0.7:9376; do
    wget -qO- $ep
done
```

باید چیزی شبیه به این ببینید:

```
hostnames-632524106-bbpiw
hostnames-632524106-ly40y
hostnames-632524106-tlaok
```

شما انتظار دارید که هر پاد در لیست Endpoints پاسخ متناظر خود را برگرداند. اگر این اتفاق نمی‌افتد (یا هر رفتار صحیح دیگری برای پادهای شما)، باید بررسی کنید که چه اتفاقی می‌افتد.
```markdown
## آیا kube-proxy کار می‌کند؟

اگر به اینجا رسیده‌اید، به این معناست که سرویس شما در حال اجرا است، Endpoints دارد و پادهای شما در واقع خدمت می‌دهند. در این نقطه، مکانیزم کلی پروکسی سرویس مورد شک است. بیایید قطعه به قطعه آن را تأیید کنیم.

پیاده‌سازی پیش‌فرض سرویس‌ها، و آنی که بر روی بیشتر خوشه‌ها استفاده می‌شود، kube-proxy است. این یک برنامه است که بر روی هر گره اجرا می‌شود و یکی از مکانیزم‌های کوچک برای فراهم کردن انتزاع سرویس را پیکربندی می‌کند. اگر خوشه شما از kube-proxy استفاده نمی‌کند، بخش‌های زیر برقرار نمی‌شود و شما باید تحقیقات خود را درباره پیاده‌سازی سرویس‌هایی که استفاده می‌کنید، انجام دهید.

### آیا kube-proxy در حال اجرا است؟

تأیید کنید که `kube-proxy` بر روی گره‌های شما در حال اجرا است. با اجرای دستی بر روی یک گره، باید نتایجی مشابه زیر را ببینید:

```shell
ps auxw | grep kube-proxy
```
```none
root  4194  0.4  0.1 101864 17696 ?    Sl Jul04  25:43 /usr/local/bin/kube-proxy --master=https://kubernetes-master --kubeconfig=/var/lib/kube-proxy/kubeconfig --v=2
```

سپس، تأیید کنید که به اشتباهی چیزی مثل عدم تماس با مستر رخ نمی‌دهد. برای این کار، شما باید به لاگ‌ها نگاه کنید. دسترسی به لاگ‌ها بستگی به سیستم‌عامل گره شما دارد. در برخی سیستم‌عامل‌ها، این یک فایل مانند /var/log/kube-proxy.log است، در حالی که سیستم‌عامل‌های دیگر از `journalctl` برای دسترسی به لاگ‌ها استفاده می‌کنند. باید چیزی شبیه به زیر را ببینید:

```none
I1027 22:14:53.995134    5063 server.go:200] Running in resource-only container "/kube-proxy"
I1027 22:14:53.998163    5063 server.go:247] Using iptables Proxier.
I1027 22:14:54.038140    5063 proxier.go:352] Setting endpoints for "kube-system/kube-dns:dns-tcp" to [10.244.1.3:53]
I1027 22:14:54.038164    5063 proxier.go:352] Setting endpoints for "kube-system/kube-dns:dns" to [10.244.1.3:53]
I1027 22:14:54.038209    5063 proxier.go:352] Setting endpoints for "default/kubernetes:https" to [10.240.0.2:443]
I1027 22:14:54.038238    5063 proxier.go:429] Not syncing iptables until Services and Endpoints have been received from master
I1027 22:14:54.040048    5063 proxier.go:294] Adding new service "default/kubernetes:https" at 10.0.0.1:443/TCP
I1027 22:14:54.040154    5063 proxier.go:294] Adding new service "kube-system/kube-dns:dns" at 10.0.0.10:53/UDP
I1027 22:14:54.040223    5063 proxier.go:294] Adding new service "kube-system/kube-dns:dns-tcp" at 10.0.0.10:53/TCP
```

اگر پیام‌های خطا درباره عدم توانایی برقراری تماس با مستر را ببینید، باید پیکربندی و مراحل نصب گره خود را دوباره بررسی کنید.

یکی از دلایل ممکن که kube-proxy نمی‌تواند به درستی اجرا شود این است که باینری `conntrack` مورد نیاز پیدا نمی‌شود. این ممکن است بر روی برخی سیستم‌عامل‌های لینوکس اتفاق بیفتد، به نحوی که از طریق آن گونه که شما از پشته گردانی کلانه یا نصب مجدد کابرنتیس را به اشتراک می‌گذارید. اگر این اتفاق افتاده باشد، باید بسته conntrack را به طور دستی نصب کنید (مانند `sudo apt install conntrack` در اوبونتو) و سپس دوباره امتحان کنید.

kube-proxy می‌تواند در یکی از چند حالت اجرا شود. در لاگ گفته شده در بالا، خط `Using iptables Proxier` نشان می‌دهد که kube-proxy در حال اجرا در حالت "iptables" است. حالت متداول دیگر "ipvs" است.

#### حالت iptables

در حالت "iptables"، باید چیزی شبیه به زیر را ببینید:

```shell
iptables-save | grep hostnames
```
```none
-A KUBE-SEP-57KPRZ3JQVENLNBR -s 10.244.3.6/32 -m comment --comment "default/hostnames:" -j MARK --set-xmark 0x00004000/0x00004000
-A KUBE-SEP-57KPRZ3JQVENLNBR -p tcp -m comment --comment "default/hostnames:" -m tcp -j DNAT --to-destination 10.244.3.6:9376
-A KUBE-SEP-WNBA2IHDGP2BOBGZ -s 10.244.1.7/32 -m comment --comment "default/hostnames:" -j MARK --set-xmark 0x00004000/0x00004000
-A KUBE-SEP-WNBA2IHDGP2BOBGZ -p tcp -m comment --comment "default/hostnames:" -m tcp -j DNAT --to-destination 10.244.1.7:9376
-A KUBE-SEP-X3P2623AGDH6CDF3 -s 10.244.2.3/32 -m comment --comment "default/hostnames:" -j MARK --set-xmark 0x00004000/0x00004000
-A KUBE-SEP-X3P2623AGDH6CDF3 -p tcp -m comment --comment "default/hostnames:" -m tcp -j DNAT --to-destination 10.244.2.3:9376
-A KUBE-S

ERVICES -d 10.0.1.175/32 -p tcp -m comment --comment "default/hostnames: cluster IP" -m tcp --dport 80 -j KUBE-SVC-NWV5X2332I4OT4T3
-A KUBE-SVC-NWV5X2332I4OT4T3 -m comment --comment "default/hostnames:" -m statistic --mode random --probability 0.33332999982 -j KUBE-SEP-WNBA2IHDGP2BOBGZ
-A KUBE-SVC-NWV5X2332I4OT4T3 -m comment --comment "default/hostnames:" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-X3P2623AGDH6CDF3
-A KUBE-SVC-NWV5X2332I4OT4T3 -m comment --comment "default/hostnames:" -j KUBE-SEP-57KPRZ3JQVENLNBR
```

برای هر پورت هر سرویس، باید 1 قاعده در `KUBE-SERVICES` و یک زنجیره `KUBE-SVC-<hash>` وجود داشته باشد. برای هر Endpoint پاد، باید تعداد کمی قوانین در آن `KUBE-SVC-<hash>` و یک `KUBE-SEP-<hash>` با تعداد کمی قوانین در آن وجود داشته باشد. قوانین دقیق بر اساس پیکربندی دقیق شما (شامل NodePorts و load-balancerها) متغیر خواهد بود.

#### حالت ipvs

در حالت "ipvs"، باید چیزی شبیه به زیر را ببینید:

```shell
ipvsadm -ln
```
```none
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
...
TCP  10.0.1.175:80 rr
  -> 10.244.0.5:9376               Masq    1      0          0
  -> 10.244.0.6:9376               Masq    1      0          0
  -> 10.244.0.7:9376               Masq    1      0          0
...
```

برای هر پورت هر سرویس، به علاوه هر NodePort، آی‌پی‌های خارجی و آی‌پی‌های load-balancer، kube-proxy یک سرور مجازی ایجاد می‌کند. برای هر Endpoint پاد، سرورهای واقعی متناظر را ایجاد می‌کند. در این مثال، هاست نام‌ها (`10.0.1.175:80`) 3 نقطه پایان (`10.244.0.5:9376`، `10.244.0.6:9376`، `10.244.0.7:9376`) دارد.

### آیا kube-proxy پروکسی می‌کند؟

اگر فقط یکی از موارد فوق را ببینید، دوباره تلاش کنید تا به سرویس خود از طریق آی‌پی از یکی از گره‌های خود دسترسی پیدا کنید:

```shell
curl 10.0.1.175:80
```
```none
hostnames-632524106-bbpiw
```

اگر هنوز هم این کار انجام نمی‌شود، لاگ‌های `kube-proxy` را برای خطوط خاصی مانند زیر بررسی کنید:

```none
Setting endpoints for default/hostnames:default to [10.244.0.5:9376 10.244.0.6:9376 10.244.0.7:9376]
```

اگر این خطوط را ندیدید، دوباره `kube-proxy` را با تنظیم پرچم `-v` به 4 راه‌اندازی کنید و سپس دوباره به لاگ‌ها نگاه کنید.

### مورد ویژه: پادی از طریق آی‌پی سرویس به خودش دسترسی ندارد

این اتفاق ممکن است نادر به نظر برسد، اما رخ می‌دهد و باید کار کند.

این ممکن است در حالتی رخ دهد که ترافیک شبکه برای تراکم "موهای موجود"، معمولاً هنگامی که `kube-proxy` در حال اجرا در حالت `iptables` است و پادها با شبکه پل متصل شده‌اند، تنظیم نشده باشد. `Kubelet` یک `hairpin-mode` را به اشتراک می‌گذارد [flag](/docs/reference/command-line-tools-reference/kubelet/) که به انتهایهای سرویس اجازه می‌دهد که مجدداً به خودشان برگردند اگر سعی کنند که به VIP سرویس خود دسترسی پیدا کنند. `hairpin-mode` flag باید یا به `hairpin-veth` یا به `promiscuous-bridge` تنظیم شود.

گام‌های رایج برای رفع این مشکل به شرح زیر است:

* تأیید کنید که `hairpin-mode` به `hairpin-veth` یا `promiscuous-bridge` تنظیم شده باشد. باید چیزی شبیه به زیر را ببینید. `hairpin-mode` به `promiscuous-bridge` در مثال زیر تنظیم شده است.

```shell
ps auxw | grep kubelet
```
```none
root      3392  1.1  0.8 186804 65208 ?        Sl   00:51  11:11 /usr/local/bin/kubelet --enable-debugging-handlers=true --config=/etc/kubernetes/manifests --allow-privileged=True --v=4 --cluster-dns=10.0.0.10 --cluster-domain=cluster.local --configure-cbr0=true --cgroup-root=/ --system-cgroups=/system --hairpin-mode=promiscuous-bridge --runtime-cgroups=/docker-daemon --kubelet-cgroups=/kubelet --babysit-daemons=true --max-pods=110 --serialize-image-pulls=false --outofdisk-transition-frequency=0
```

* تأیید کنید که `hairpin-mode` اثربخش باشد. برای انجام این کار، باید به لاگ kubelet نگاه کنید. دسترسی به لاگ‌ها بستگی به سیستم‌عامل گره شما دارد. در برخی سیستم‌عامل‌ها، این یک فایل مانند /var/log/kubelet.log است، در حالی که سیستم‌عامل‌های دیگر از `journalctl` برای دسترسی به لاگ‌ها استفاده می‌کنند. لطفاً توجه داشته باشید که حالت hairpin اثربخش ممکن است با پرچم `--hairpin-mode` به دلیل سازگاری مطابقت نداشته باشد. بررسی کنید که آیا خطوط

 لاگ‌هایی با کلمه کلیدی `hairpin` در kubelet.log وجود دارد. باید خطوط لاگ با نمادگذاری حالت hairpin اثربخش را مشخص کند، مانند چیزی زیر.

```none
I0629 00:51:43.648698    3252 kubelet.go:380] Hairpin mode set to "promiscuous-bridge"
```

* اگر حالت hairpin موثر `hairpin-veth` باشد، اطمینان حاصل کنید که `Kubelet` دارای اجازه مورد نیاز برای عملکرد در `/sys` در گره است. اگر همه چیز به درستی کار کند، باید چیزی شبیه به زیر را ببینید.

```shell
for intf in /sys/devices/virtual/net/cbr0/brif/*; do cat $intf/hairpin_mode; done
```
```none
1
1
1
1
```

* اگر حالت hairpin موثر `promiscuous-bridge` باشد، اطمینان حاصل کنید که `Kubelet` دارای اجازه مورد نیاز برای مدیریت پل linux در گره است. اگر پل `cbr0` استفاده شده و به درستی پیکربندی شده باشد، باید چیزی شبیه به زیر را ببینید.

```shell
ifconfig cbr0 |grep PROMISC
```
```none
UP BROADCAST RUNNING PROMISC MULTICAST  MTU:1460  Metric:1
```

* اگر هیچ‌کدام از موارد فوق به نتیجه نرسید، به دنبال کمک بگردید.

## به دنبال کمک بگردید

اگر تا به اینجا رسیدید، چیزی بسیار عجیب در حال رخ دادن است. سرویس شما در حال اجرا است، پایانه‌هایی دارد و پادهای شما واقعاً خدمت می‌کنند. شما DNS را دارید و `kube-proxy` به نظر نمی‌رسد به شیوه ناهماهنگ کار می‌کند. اما سرویس شما کار نمی‌کند. لطفاً به ما اطلاع دهید که چه اتفاقی می‌افتد تا بتوانیم در بررسی کمک کنیم!

برای ارتباط با ما، به
[Slack](https://slack.k8s.io/) یا
[Forum](https://discuss.kubernetes.io) یا
[GitHub](https://github.com/kubernetes/kubernetes) مراجعه کنید.

## {{% heading "whatsnext" %}}

برای کسب اطلاعات بیشتر، به [سند مروری اشکال‌زدایی](/docs/tasks/debug/) مراجعه کنید.