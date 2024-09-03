```markdown
---
reviewers:
- caesarxuchao
- lavalamp
- thockin
title: اتصال برنامه‌ها با خدمات در Kubernetes
content_type: آموزش
weight: 20
---

<!-- overview -->

## مدل Kubernetes برای اتصال کانتینرها

حالا که یک برنامه مستمر و با تکرار دارید، می‌توانید آن را در شبکه قرار دهید.

Kubernetes فرض می‌کند که پادها می‌توانند با پادهای دیگر ارتباط برقرار کنند، بدون اینکه اهمیت داشته باشد که روی کدام میزبانی قرار گرفته‌اند.
هر پاد در Kubernetes یک آدرس IP خصوصی داخلی خویش را دارد، بنابراین نیازی به ایجاد لینک‌های مستقیم بین پادها یا نقشه‌برداری پورت‌های کانتینر به پورت‌های میزبان نیست.
این به معنای آن است که کانتینرهای داخل یک پاد می‌توانند به پورت‌های یکدیگر در localhost دسترسی داشته باشند و تمام پادها در یک خوشه می‌توانند یکدیگر را بدون NAT ببینند. این سند توضیح می‌دهد که چگونه می‌توانید خدمات قابل اعتماد را در چنین مدل شبکه‌ای اجرا کنید.

این آموزش از یک سرور وب ساده nginx برای نمایش این مفهوم‌ها استفاده می‌کند.

<!-- body -->

## ارائه پادها به خوشه

ما این کار را قبلاً انجام داده‌ایم، اما دوباره آن را از منظر شبکه بررسی می‌کنیم.
یک پاد nginx با مشخصات پورت کانتینر ایجاد کنید:

{{% code_sample file="service/networking/run-my-nginx.yaml" %}}

این امکان را به پاد ارائه می‌دهد که از هر یک از گره‌های خوشه به دسترس باشد. گره‌هایی که پاد در آنها اجرا می‌شود را بررسی کنید:

```shell
kubectl apply -f ./run-my-nginx.yaml
kubectl get pods -l run=my-nginx -o wide
```
```
NAME                        READY     STATUS    RESTARTS   AGE       IP            NODE
my-nginx-3800858182-jr4a2   1/1       Running   0          13s       10.244.3.4    kubernetes-minion-905m
my-nginx-3800858182-kna2y   1/1       Running   0          13s       10.244.2.5    kubernetes-minion-ljyd
```

آدرس‌های پادهای خود را تأیید کنید:

```shell
kubectl get pods -l run=my-nginx -o custom-columns=POD_IP:.status.podIPs
    POD_IP
    [map[ip:10.244.3.4]]
    [map[ip:10.244.2.5]]
```

شما باید بتوانید به هر یک از گره‌های خوشه SSH کنید و از ابزاری مانند `curl` برای درخواست به هر دو آدرس استفاده کنید. توجه داشته باشید که کانتینرها از پورت 80 روی گره استفاده نمی‌کنند و هیچ قانون NAT ویژه‌ای برای هدایت ترافیک به پاد وجود ندارد. این بدان معناست که می‌توانید چندین پاد nginx را روی همان گره اجرا کنید، همه از همان `containerPort` استفاده کنند و به آنها از هر پاد یا گره دیگری در خوشه خود دسترسی داشته باشید، با استفاده از IP پاد اختصاصی آن.

برای اطلاعات بیشتر، می‌توانید درباره
[مدل شبکه Kubernetes](/docs/concepts/cluster-administration/networking/#the-kubernetes-network-model)
مطالعه کنید.

## ایجاد یک خدمات

ما پادهای nginx را در یک فضای آدرس مسطح وسیع خوشه داریم. به طور تئوری می‌توانید به این پادها مستقیماً صحبت کنید، اما چه اتفاقی می‌افتد وقتی یک گره می‌میرد؟ پادهای مرتبط همچنین می‌میرند و ReplicaSet داخل Deployment آنها را با آی‌پی‌های مختلف بازسازی می‌کند. این مسئله است که خدمات آن را حل می‌کند.

یک خدمات Kubernetes یک انتزاع است که مجموعه‌ای منطقی از پادهای در حال اجرا در جایی در خوشه شما را تعریف می‌کند که همه توانایی یکسانی را ارائه می‌دهند. هرگاه ایجاد شود، به هر خدمات یک آدرس IP یکتا (معروف به clusterIP) اختصاص داده می‌شود. این آدرس به عمر خدمات مربوط می‌شود و تا زمانی که خدمات زنده است تغییر نخواهد کرد. می‌توان پادها را پیکربندی کرد که با خدمات صحبت کنند و مطمئن باشند که ارتباط به خدمات به طور خودکار بین پادهای عضو خدمات بارگذاری خواهد شد.

شما می‌توانید یک خدمات برای 2 کپی nginx خود با استفاده از `kubectl expose` ایجاد کنید:

```shell
kubectl expose deployment/my-nginx
```
```
service/my-nginx exposed
```

همچنین می‌توانید yaml زیر را با استفاده از `kubectl apply -f` اعمال کنید:

{{% code_sample file="service/networking/nginx-svc.yaml" %}}

این مشخصات یک خدمات ایجاد می‌کند که TCP پ

ورت 80 را بر روی هر پاد با برچسب `run: my-nginx` هدف قرار می‌دهد و آن را بر روی یک پورت خدمات انتزاعی (targetPort) که دیگر پادها برای دسترسی به خدمات استفاده می‌کنند، عرضه می‌کند.
برای مشاهده [خدمات](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#service-v1-core)
شی API شامل فهرستی از فیلدهای پشتیبانی شده در تعریف خدمات مشاهده کنید.
خدمات خود را بررسی کنید:

```shell
kubectl get svc my-nginx
```
```
NAME       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
my-nginx   ClusterIP   10.0.162.149   <none>        80/TCP    21s
```

همانطور که قبلاً ذکر شد، خدمات توسط یک گروه از پادها پشتیبانی می‌شوند که از طریق
{{<glossary_tooltip term_id="endpoint-slice" text="EndpointSlices">}} 
برخوردار هستند. انتخاب گزینه خدمات به طور مداوم ارزیابی می‌شود و نتایج به
EndpointSlice ای که با خدمات از طریق
{{< glossary_tooltip text="labels" term_id="label" >}} 
متصل شده است POST خواهد شد. وقتی یک پاد می‌میرد، به طور خودکار از EndpointSlices که شامل آن به عنوان یک انتهای مسیر وجود دارد، حذف می‌شود. پادهای جدید که با انتخاب کننده خدمات همخوانی دارند به طور خودکار به EndpointSlice برای آن خدمات اضافه می‌شوند.
موقعیت پایانی را بررسی کنید، و توجه داشته باشید که آدرس‌ها با آدرس‌های پادهایی که در مرحله اول ایجاد شده‌اند، همخوانی دارند:

```shell
kubectl describe svc my-nginx
```
```
Name:                my-nginx
Namespace:           default
Labels:              run=my-nginx
Annotations:         <none>
Selector:            run=my-nginx
Type:                ClusterIP
IP Family Policy:    SingleStack
IP Families:         IPv4
IP:                  10.0.162.149
IPs:                 10.0.162.149
Port:                <unset> 80/TCP
TargetPort:          80/TCP
Endpoints:           10.244.2.5:80,10.244.3.4:80
Session Affinity:    None
Events:              <none>
```
```shell
kubectl get endpointslices -l kubernetes.io/service-name=my-nginx
```
```
NAME             ADDRESSTYPE   PORTS   ENDPOINTS               AGE
my-nginx-7vzhx   IPv4          80      10.244.2.5,10.244.3.4   21s
```


حالا شما می‌توانید از هر گره‌ای در خوشه خود به خدمات nginx با استفاده از `<CLUSTER-IP>:<PORT>` curl کنید. توجه داشته باشید که آدرس IP خدمات کاملاً مجازی است و هیچ‌وقت به سیم نمی‌خورد. اگر کنجکاوید که این چگونه کار می‌کند می‌توانید بیشتر درباره [پروکسی خدمات](/docs/reference/networking/virtual-ips/) مطالعه کنید.

## دسترسی به خدمات

Kubernetes پشتیبانی از 2 حالت اصلی برای پیدا کردن یک خدمت دارد - متغیرهای محیطی و DNS. حالت اول به طور خودکار کار می‌کند در حالی که حالت دوم نیاز به افزونه خوشه [CoreDNS](https://releases.k8s.io/v{{< skew currentPatchVersion >}}/cluster/addons/dns/coredns) را دارد.

{{< note >}}
اگر متغیرهای محیطی خدمات (به عنوان مثال به دلیل تداخل با متغیرهای برنامه مورد انتظار، تعداد زیادی متغیر برای پردازش، استفاده تنها از DNS و غیره) نیاز نیست، می‌توانید این حالت را با تنظیم پرچم `enableServiceLinks` به `false` در [مشخصات پاد](/docs/reference/generated/kubernetes-api/v{{< skew latestVersion >}}/#pod-v1-core) غیرفعال کنید.
{{< /note >}}

### متغیرهای محیطی

زمانی که یک پاد روی یک گره اجرا می‌شود، kubelet مجموعه‌ای از متغیرهای محیطی برای هر خدمت فعال اضافه می‌کند. این مسئله مسئله‌ی چینش معرفی می‌کند. برای دیدن دلیل آن، محیط اجرایی پاد nginx فعال خود را بررسی کنید (نام پاد شما ممکن است متفاوت باشد):

```shell
kubectl exec my-nginx-3800858182-jr4a2 -- printenv | grep SERVICE
```
```
KUBERNETES_SERVICE_HOST=10.0.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
```

توجه داشته باشید که هیچ اشاره‌ای به خدمت شما نیست. این به این دلیل است که شما رپلیکا‌ها را قبل از خدمت ایجاد کرده‌اید. یک معایب دیگر این است که برنامه زمانبند ممکن است هر دو پاد را روی همان ماشین قرار دهد که اگر مرده باشد، تمام خدمت شما را برمی‌دارد. ما می‌توانیم این کار را به درستی با از بین بردن 2 پاد و انتظار برای Deployment برای بازسازی آنها انجام دهیم. این بار خدمت قبل از رپلیکا‌ها وجود دارد. این به شما انتشار خدمات بر روی برنامه زمانبند پادهای شما (با توجه به اینکه تمام گره‌های شما ظرفیت مساوی دارند) و همچنین متغیرهای محیطی مناسب خواهد داد:

```shell
kubectl scale deployment my-nginx --replicas=0; kubectl scale deployment my-nginx --replicas=2;

kubectl get pods -l run=my-nginx -o wide
```
```
NAME                        READY     STATUS    RESTARTS   AGE     IP            NODE
my-nginx-3800858182-e9ihh   1/1       Running   0          5s      10.244.2.7    kubernetes-minion-ljyd
my-nginx-3800858182-j4rm4   1/1       Running   0          5s      10.244.3.8    kubernetes-minion-905m
```

ممکن است توجه کنید که پادها نام‌های متفاوتی دارند، زیرا که کشته و بازسازی شده‌اند.

```shell
kubectl exec my-nginx-3800858182-e9ihh -- printenv | grep SERVICE
```
```
KUBERNETES_SERVICE_PORT=443
MY_NGINX_SERVICE_HOST=10.0.162.149
KUBERNETES_SERVICE_HOST=10.0.0.1
MY_NGINX_SERVICE_PORT=80
KUBERNETES_SERVICE_PORT_HTTPS=443
```

### DNS

Kubernetes یک افزونه خوشه DNS Service ارائه می‌دهد که به طور خودکار نام‌های dns را به سایر خدمات اختصاص می‌دهد. شما می‌توانید بررسی کنید که آیا آن در خوشه شما در حال اجرا است یا خیر:

```shell
kubectl get services kube-dns --namespace=kube-system
```
```
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
kube-dns   ClusterIP   10.0.0.10    <none>        53/UDP,53/TCP   8m
```

قسمت دیگر این بخش فرض می‌کند که یک خدمت با IP طولانی مدت (my-nginx) و یک سرور DNS دارید که نامی به آن IP اختصاص داده است. در اینجا ما از افزونه خوشه CoreDNS استفاده می‌کنیم (نام برنامه `kube-dns`، بنابراین می‌توانید از روش‌های استاندارد مانند `gethostbyname()` برای صحبت کردن با خدمات از هر پاد در خوشه خود استفاده کنید. اگر CoreDNS در حال اجرا نیست، می‌توانید آن را فعال کنید که به [README CoreDNS](https://github.com/coredns/deployment/tree/master/kubernetes) یا [نصب CoreDNS](/docs/tasks/administer-cluster/coredns/#installing-coredns) مراجعه کنید. بیایید برنامه curl دیگری را اجرا کنیم تا این را تست کنیم:

```shell
kubectl run curl --image=radial/busyboxplus:curl -i --tty --rm
```
```
Waiting for pod default/curl-131556218-9fnch to be running, status is Pending, pod ready: false


Hit enter for command prompt
```

سپس، enter را فشار دهید و `nslookup my-nginx` را اجرا کنید:

```shell
[ root@curl-131556218-9fnch:/ ]$ nslookup my-nginx
Server:    10.0.0.10
Address 1: 10.0.0.10

Name:      my-nginx
Address 1: 10.0.162.149
```

## امن کردن خدمات

تا به حال فقط از سرور nginx درون خوشه دسترسی داشته‌ایم. قبل از اینکه خدمات را به اینترنت بیرونی منتشر کنید، می‌خواهید اطمینان حاصل کنید که کانال ارتباطی امن است. برای این کار، شما به نیاز خواهید داشت:

- گواهی‌نامه‌های امنیتی (SSL) از نوع خودامضا (مگر اینکه یک گواهی‌نامه ایدنتیت دیگری داشته باشید)
- سرور nginx تنظیم شده برای استفاده از این گواهی‌نامه‌ها
- یک [راز (Secret)](/docs/concepts/configuration/secret/) که گواهی‌نامه‌ها را به پادها ارائه می‌دهد

شما می‌توانید همه این‌ها را از [مثال https nginx](https://github.com/kubernetes/examples/tree/master/staging/https-nginx/) بدست آورید. این نیازمند داشتن ابزارهای go و make بر روی سیستم شما است. اگر نمی‌خواهید این ابزارها را نصب کنید، می‌توانید مراحل دستی را دنبال کنید. به طور خلاصه:

```shell
make keys KEY=/tmp/nginx.key CERT=/tmp/nginx.crt
kubectl create secret tls nginxsecret --key /tmp/nginx.key --cert /tmp/nginx.crt
```
```
secret/nginxsecret created
```
```shell
kubectl get secrets
```
```
NAME                  TYPE                                  DATA      AGE
nginxsecret           kubernetes.io/tls                     2         1m
```
همچنین configmap:
```shell
kubectl create configmap nginxconfigmap --from-file=default.conf
```

شما می‌توانید نمونه‌ای از `default.conf` را در [مخزن پروژه‌های Kubernetes](https://github.com/kubernetes/examples/tree/bc9ca4ca32bb28762ef216386934bef20f1f9930/staging/https-nginx/) پیدا کنید.

```shell
configmap/nginxconfigmap created
```
```shell
kubectl get configmaps
```
```
NAME             DATA   AGE
nginxconfigmap   1      114s
```

شما می‌توانید جزئیات configmap `nginxconfigmap` را با استفاده از دستور زیر مشاهده کنید:

```shell
kubectl describe configmap nginxconfigmap
```

خطوط دستی را به دنبال این مراحل انجام دهید اگر مشکلی با اجرای make (مثلاً در ویندوز) داشته باشید:

```shell
# ایجاد یک جفت کلید خصوصی عمومی
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /d/tmp/nginx.key -out /d/tmp/nginx.crt -subj "/CN=my-nginx/O=my-nginx"
# تبدیل کلیدها به رمزگذاری base64
cat /d/tmp/nginx.crt | base64
cat /d/tmp/nginx.key | base64
```

از خروجی دستورات قبلی برای ایجاد یک فایل yaml به شکل زیر استفاده کنید. مقادیر رمزگذاری شده base64 باید در یک خط تکی باشند.

```yaml
apiVersion: "v1"
kind: "Secret"
metadata:
  name: "nginxsecret"
  namespace: "default"
type: kubernetes.io/tls
data:
  tls.crt: "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURIekNDQWdlZ0F3SUJBZ0lKQUp5M3lQK0pzMlpJTUEwR0NTcUdTSWIzRFFFQkJRVUFNQ1l4RVRBUEJnTlYKQkFNVENHNW5hVzU0YzNaak1SRXdEd1lEVlFRS0V3aHVaMmx1ZUhOMll6QWVGdzB4TnpFd01qWXdOekEzTVRKYQpGdzB4T0RFd01qWXdOekEzTVRKYU1DWXhFVEFQQmdOVkJBTVRDRzVuYVc1NGMzWmpNUkV3RHdZRFZRUUtFd2h1CloybHVlSE4yWXpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBSjFxSU1SOVdWM0IKMlZIQlRMRmtobDRONXljMEJxYUhIQktMSnJMcy8vdzZhU3hRS29GbHlJSU94NGUrMlN5ajBFcndCLzlYTnBwbQppeW1CL3JkRldkOXg5UWhBQUxCZkVaTmNiV3NsTVFVcnhBZW50VWt1dk1vLzgvMHRpbGhjc3paenJEYVJ4NEo5Ci82UVRtVVI3a0ZTWUpOWTVQZkR3cGc3dlVvaDZmZ1Voam92VG42eHNVR0M2QURVODBpNXFlZWhNeVI1N2lmU2YKNHZpaXdIY3hnL3lZR1JBRS9mRTRqakxCdmdONjc2SU90S01rZXV3R0ljNDFhd05tNnNTSzRqYUNGeGpYSnZaZQp2by9kTlEybHhHWCtKT2l3SEhXbXNhdGp4WTRaNVk3R1ZoK0QrWnYvcW1mMFgvbVY0Rmo1NzV3ajFMWVBocWtsCmdhSXZYRyt4U1FVQ0F3RUFBYU5RTUU0d0hRWURWUjBPQkJZRUZPNG9OWkI3YXc1OUlsYkROMzhIYkduYnhFVjcKTUI4R0ExVWRJd1FZTUJhQUZPNG9OWkI3YXc1OUlsYkROMzhIYkduYnhFVjdNQXdHQTFVZEV3UUZNQU1CQWY4dwpEUVlKS29aSWh2Y05BUUVGQlFBRGdnRUJBRVhTMW9FU0lFaXdyMDhWcVA0K2NwTHI3TW5FMTducDBvMm14alFvCjRGb0RvRjdRZnZqeE04Tzd2TjB0clcxb2pGSW0vWDE4ZnZaL3k4ZzVaWG40Vm8zc3hKVmRBcStNZC9jTStzUGEKNmJjTkNUekZqeFpUV0UrKzE5NS9zb2dmOUZ3VDVDK3U2Q3B5N0M3MTZvUXRUakViV05VdEt4cXI0Nk1OZWNCMApwRFhWZmdWQTRadkR4NFo3S2RiZDY5eXM3OVFHYmg5ZW1PZ05NZFlsSUswSGt0ejF5WU4vbVpmK3FqTkJqbWZjCkNnMnlwbGQ0Wi8rUUNQZjl3SkoybFIrY2FnT0R4elBWcGxNSEcybzgvTHFDdnh6elZPUDUxeXdLZEtxaUMwSVEKQ0I5T2wwWW5scE9UNEh1b2hSUzBPOStlMm9KdFZsNUIyczRpbDlhZ3RTVXFxUlU9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
  tls.key: "LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2UUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktjd2dnU2pBZ0VBQW9JQkFRQ2RhaURFZlZsZHdkbFIKd1V5eFpJWmVEZWNuTkFhbWh4d1NpeWF5N1AvOE9ta3NVQ3FCWmNpQ0RzZUh2dGtzbzlCSzhBZi9WemFhWm9zcApnZjYzUlZuZmNmVUlRQUN3WHhHVFhHMXJKVEVGSzhRSHA3VkpMcnpLUC9QOUxZcFlYTE0yYzZ3MmtjZUNmZitrCkU1bEVlNUJVbUNUV09UM3c4S1lPNzFLSWVuNEZJWTZMMDUrc2JGQmd1Z0ExUE5JdWFubm9UTWtlZTRuMG4rTDQKb3NCM01ZUDhtQmtRQlAzeE9JNHl3YjREZXUraURyU2pKSHJzQmlIT05Xc0RadXJFaXVJMmdoY1kxeWIyWHI2UAozVFVOcGNSbC9pVG9zQngxcHJHclk4V09HZVdPeGxZZmcvbWIvNnBuOUYvNWxlQlkrZStjSTlTMkQ0YXBKWUdpCkwxeHZzVWtGQWdNQkFBRUNnZ0VBZFhCK0xkbk8ySElOTGo5bWRsb25IUGlHWWVzZ294RGQwci9hQ1Zkank4dlEKTjIwL3FQWkUxek1yall6Ry9kVGhTMmMwc0QxaTBXSjdwR1lGb0xtdXlWTjltY0FXUTM5SjM0VHZaU2FFSWZWNgo5TE1jUHhNTmFsNjRLMFRVbUFQZytGam9QSFlhUUxLOERLOUtnNXNrSE5pOWNzMlY5ckd6VWlVZWtBL0RBUlBTClI3L2ZjUFBacDRuRWVBZmI3WTk1R1llb1p5V21SU3VKdlNyblBESGtUdW1vVlVWdkxMRHRzaG9reUxiTWVtN3oKMmJzVmpwSW1GTHJqbGtmQXlpNHg0WjJrV3YyMFRrdWtsZU1jaVlMbjk4QWxiRi9DSmRLM3QraTRoMTVlR2ZQegpoTnh3bk9QdlVTaDR2Q0o3c2Q5TmtEUGJvS2JneVVHOXBYamZhRGR2UVFLQmdRRFFLM01nUkhkQ1pKNVFqZWFKClFGdXF4cHdnNzhZTjQyL1NwenlUYmtGcVFoQWtyczJxWGx1MDZBRzhrZzIzQkswaHkzaE9zSGgxcXRVK3NHZVAKOWRERHBsUWV0ODZsY2FlR3hoc0V0L1R6cEdtNGFKSm5oNzVVaTVGZk9QTDhPTm1FZ3MxMVRhUldhNzZxelRyMgphRlpjQ2pWV1g0YnRSTHVwSkgrMjZnY0FhUUtCZ1FEQmxVSUUzTnNVOFBBZEYvL25sQVB5VWs1T3lDdWc3dmVyClUycXlrdXFzYnBkSi9hODViT1JhM05IVmpVM25uRGpHVHBWaE9JeXg5TEFrc2RwZEFjVmxvcG9HODhXYk9lMTAKMUdqbnkySmdDK3JVWUZiRGtpUGx1K09IYnRnOXFYcGJMSHBzUVpsMGhucDBYSFNYVm9CMUliQndnMGEyOFVadApCbFBtWmc2d1BRS0JnRHVIUVV2SDZHYTNDVUsxNFdmOFhIcFFnMU16M2VvWTBPQm5iSDRvZUZKZmcraEppSXlnCm9RN3hqWldVR3BIc3AyblRtcHErQWlSNzdyRVhsdlhtOElVU2FsbkNiRGlKY01Pc29RdFBZNS9NczJMRm5LQTQKaENmL0pWb2FtZm1nZEN0ZGtFMXNINE9MR2lJVHdEbTRpb0dWZGIwMllnbzFyb2htNUpLMUI3MkpBb0dBUW01UQpHNDhXOTVhL0w1eSt5dCsyZ3YvUHM2VnBvMjZlTzRNQ3lJazJVem9ZWE9IYnNkODJkaC8xT2sybGdHZlI2K3VuCnc1YytZUXRSTHlhQmd3MUtpbGhFZDBKTWU3cGpUSVpnQWJ0LzVPbnlDak9OVXN2aDJjS2lrQ1Z2dTZsZlBjNkQKckliT2ZIaHhxV0RZK2Q1TGN1YSt2NzJ0RkxhenJsSlBsRzlOZHhrQ2dZRUF5elIzT3UyMDNRVVV6bUlCRkwzZAp4Wm5XZ0JLSEo3TnNxcGFWb2RjL0d5aGVycjFDZzE2MmJaSjJDV2RsZkI0VEdtUjZZdmxTZEFOOFRwUWhFbUtKCnFBLzVzdHdxNWd0WGVLOVJmMWxXK29xNThRNTBxMmk1NVdUTThoSDZhTjlaMTltZ0FGdE5VdGNqQUx2dFYxdEYKWSs4WFJkSHJaRnBIWll2NWkwVW1VbGc9Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K"
```

برای ایجاد اسرارات از فایل زیر استفاده کنید:

```shell
kubectl apply -f nginxsecrets.yaml
kubectl get secrets
```
```
NAME                  TYPE                                  DATA      AGE
nginxsecret           kubernetes.io/tls                     2         1m
```

حالا تعداد نمونه‌های nginx خود را برای شروع یک سرور HTTPS با استفاده از گواهی‌نامه در اسرار تغییر دهید و سرویس را طوری تنظیم کنید که هر دو پورت (80 و 443) را ارائه دهد:

{{% code_sample file="service/networking/nginx-secure-app.yaml" %}}

نکات قابل توجه در مانیفست nginx-secure-app:

- شامل مشخصات هم Deployment و هم Service در یک فایل است.
- [سرور nginx](https://github.com/kubernetes/examples/tree/master/staging/https-nginx/default.conf) ترافیک HTTP را در پورت 80 و ترافیک HTTPS را در پورت 443 ارائه می‌دهد و سرویس nginx هر دو پورت را بیرون می‌آورد.
- هر کانتینر از طریق یک حجم که در `/etc/nginx/ssl` مانت شده است، به کلیدها دسترسی دارد. این تنظیم *قبل از* شروع سرور nginx انجام می‌شود.

```shell
kubectl delete deployments,svc my-nginx; kubectl create -f ./nginx-secure-app.yaml
```

در این مرحله، می‌توانید به سرور nginx از هر گره‌ای دسترسی پیدا کنید.

```shell
kubectl get pods -l run=my-nginx -o custom-columns=POD_IP:.status.podIPs
    POD_IP
    [map[ip:10.244.3.5]]
```

```shell
node $ curl -k https://10.244.3.5
...
<h1>Welcome to nginx!</h1>
```

لطفاً توجه داشته باشید که در مرحله‌ی آخر، پارامتر `-k` را به curl دادیم، این به این دلیل است که در زمان تولید گواهی‌نامه، هیچ اطلاعاتی در مورد پادهایی که nginx را اجرا می‌کنند نداریم، بنابراین باید به curl بگوییم که از عدم تطابق CName چشم پوشی کند. با ایجاد یک سرویس، اسم CName مورد استفاده در گواهی‌نامه را با نام DNS واقعی که پادها آن را در جستجوی سرویس استفاده می‌کنند، مرتبط کرده‌ایم. بیایید این را از یک پاد تست کنیم (برای سادگی، از همان اسرار استفاده می‌کنیم که تکرار می‌شود، پاد فقط به nginx.crt نیاز دارد تا به سرویس دسترسی پیدا کند):

{{% code_sample file="service/networking/curlpod.yaml" %}}

```shell
kubectl apply -f ./curlpod.yaml
kubectl get pods -l app=curlpod
```
```
NAME                               READY     STATUS    RESTARTS   AGE
curl-deployment-1515033274-1410r   1/1       Running   0          1m
```
```shell
kubectl exec curl-deployment-1515033274-1410r -- curl https://my-nginx --cacert /etc/nginx/ssl/tls.crt
...
<title>Welcome to nginx!</title>
...
```
```

## افشای خدمات

برخی از بخش‌های برنامه‌های شما ممکن است نیاز به افشای یک خدمات بر روی یک IP آدرس خارجی داشته باشند. Kubernetes دو روش برای انجام این کار پشتیبانی می‌کند: NodePorts و LoadBalancers. خدماتی که در بخش قبلی ایجاد شدند از `NodePort` استفاده کردند، بنابراین نمونه HTTPS nginx شما آماده است تا ترافیک را بر روی اینترنت پذیرفته کند، اگر نود شما دارای یک IP عمومی باشد.

```shell
kubectl get svc my-nginx -o yaml | grep nodePort -C 5
  uid: 07191fb3-f61a-11e5-8ae5-42010af00002
spec:
  clusterIP: 10.0.162.149
  ports:
  - name: http
    nodePort: 31704
    port: 8080
    protocol: TCP
    targetPort: 80
  - name: https
    nodePort: 32453
    port: 443
    protocol: TCP
    targetPort: 443
  selector:
    run: my-nginx
```
```shell
kubectl get nodes -o yaml | grep ExternalIP -C 1
    - address: 104.197.41.11
      type: ExternalIP
    allocatable:
--
    - address: 23.251.152.56
      type: ExternalIP
    allocatable:
...

$ curl https://<EXTERNAL-IP>:<NODE-PORT> -k
...
<h1>Welcome to nginx!</h1>
```

حالا بیایید خدمت را با استفاده از یک توزیع کننده بار ابری (Load Balancer) مجدداً ایجاد کنیم.
نوع `Type` خدمت `my-nginx` را از `NodePort` به `LoadBalancer` تغییر دهید:

```shell
kubectl edit svc my-nginx
kubectl get svc my-nginx
```
```
NAME       TYPE           CLUSTER-IP     EXTERNAL-IP        PORT(S)               AGE
my-nginx   LoadBalancer   10.0.162.149   xx.xxx.xxx.xxx     8080:30163/TCP        21s
```
```
curl https://<EXTERNAL-IP> -k
...
<title>Welcome to nginx!</title>
```

آدرس IP در ستون `EXTERNAL-IP` آن است که در اینترنت عمومی در دسترس است.
`CLUSTER-IP` فقط درون شبکه خصوصی یا شبکه خوشه شما در دسترس است.

توجه داشته باشید که در AWS، نوع `LoadBalancer` یک ELB (Elastic Load Balancer) ایجاد می‌کند که از یک نام میزبان (hostname) بلند استفاده می‌کند، نه یک IP. این نام بلند برای نمایش در خروجی استاندارد `kubectl get svc` خیلی بلند است، بنابراین شما باید `kubectl describe service my-nginx` را اجرا کنید تا آن را ببینید. شما ممکن است چنین چیزی را ببینید:

```shell
kubectl describe service my-nginx
...
LoadBalancer Ingress:   a320587ffd19711e5a37606cf4a74574-1142138393.us-east-1.elb.amazonaws.com
...
```

## {{% heading "whatsnext" %}}

* بیشتر درباره [استفاده از یک خدمت برای دسترسی به برنامه در یک خوشه](/docs/tasks/access-application-cluster/service-access-application-cluster/) بیاموزید.
* بیشتر درباره [اتصال یک قسمت جلویی به قسمت پشتیبانی با استفاده از یک خدمت](/docs/tasks/access-application-cluster/connecting-frontend-backend/) بیاموزید.
* بیشتر درباره [ایجاد یک توزیع‌کننده بار خارجی](/docs/tasks/access-application-cluster/create-external-load-balancer/) بیاموزید.
```