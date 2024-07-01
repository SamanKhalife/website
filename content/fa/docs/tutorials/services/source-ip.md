---
title: استفاده از آدرس IP منبع
content_type: آموزش
min-kubernetes-server-version: v1.5
weight: 40
---

<!-- مقدمه -->

برنامه‌هایی که در یک خوشه Kubernetes اجرا می‌شوند، با استفاده از انتزاع سرویس یکدیگر و جهان بیرون، با یکدیگر ارتباط برقرار می‌کنند. این سند توضیح می‌دهد که چه اتفاقی برای آدرس IP منبع بسته‌های ارسالی به انواع مختلف سرویس‌ها می‌افتد و چگونه می‌توانید این رفتار را بر اساس نیازهای خود فعال یا غیرفعال کنید.


## {{% heading "پیش‌نیازها" %}}

### اصطلاحات

این سند از اصطلاحات زیر استفاده می‌کند:

[NAT](https://fa.wikipedia.org/wiki/Network_address_translation)
: ترجمه آدرس شبکه

[Source NAT](https://fa.wikipedia.org/wiki/Network_address_translation#SNAT)
: جایگزین کردن آدرس IP منبع در یک بسته؛ در این صفحه، این به طور معمول به معنای جایگزین کردن با آدرس IP یک گره است.

[Destination NAT](https://fa.wikipedia.org/wiki/Network_address_translation#DNAT)
: جایگزین کردن آدرس مقصد در یک بسته؛ در این صفحه، این به طور معمول به معنای جایگزین کردن با آدرس IP یک {{< glossary_tooltip term_id="pod" >}}

[VIP](/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies)
: یک آدرس IP مجازی، مانند آن که به هر {{< glossary_tooltip text="Service" term_id="service" >}} در Kubernetes اختصاص می‌یابد

[kube-proxy](/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies)
: یک دیمون شبکه که مدیریت VIP سرویس را بر روی هر گره انجام می‌دهد

### پیش‌نیازها

{{< include "task-tutorial-prereqs.md" >}}

در مثال‌ها از یک وب‌سرور کوچک nginx استفاده می‌شود که آدرس IP منبع درخواست‌هایی که از طریق هدر HTTP دریافت می‌کند، را برمی‌گرداند. شما می‌توانید آن را به شکل زیر ایجاد کنید:

{{< note >}}
تصویر در دستور زیر تنها بر روی معماری‌های AMD64 اجرا می‌شود.
{{< /note >}}

```shell
kubectl create deployment source-ip-app --image=registry.k8s.io/echoserver:1.4
```
خروجی:

```
deployment.apps/source-ip-app created
```

## {{% heading "اهداف" %}}

* ارائه یک برنامه ساده از طریق انواع مختلف سرویس‌ها
* درک اینکه هر نوع سرویس چگونه با NAT آدرس IP منبع رفتار می‌کند
* درک تضادهای مرتبط با حفظ آدرس IP منبع


<!-- محتوای درس -->

## آدرس IP منبع برای سرویس‌های با `Type=ClusterIP`

بسته‌هایی که از داخل خوشه به ClusterIP ارسال می‌شوند، هرگز NAT آدرس منبع نمی‌شوند اگر که kube-proxy را در حالت
[iptables](/docs/reference/networking/virtual-ips/#proxy-mode-iptables)
(حالت پیش‌فرض) اجرا می‌کنید. شما می‌توانید حالت kube-proxy را با دریافت
`http://localhost:10249/proxyMode`
بر روی گره‌ای که kube-proxy در آن اجرا می‌شود، بررسی کنید.

```console
kubectl get nodes
```
خروجی مشابه این است:
```
NAME                           STATUS     ROLES    AGE     VERSION
kubernetes-node-6jst   Ready      <none>   2h      v1.13.0
kubernetes-node-cx31   Ready      <none>   2h      v1.13.0
kubernetes-node-jj1t   Ready      <none>   2h      v1.13.0
```

برای دریافت حالت پروکسی در یکی از گره‌ها (kube-proxy بر روی پورت 10249 گوش می‌دهد) از دستور زیر استفاده کنید:

```shell
# این دستور را در یک پوسته بر روی گره مورد نظر اجرا کنید.
curl http://localhost:10249/proxyMode
```
خروجی:

```
iptables
```

می‌توانید با ایجاد یک سرویس بر روی برنامه آدرس IP منبع را آزمایش کنید:

```shell
kubectl expose deployment source-ip-app --name=clusterip --port=80 --target-port=8080
```
خروجی:

```
service/clusterip exposed
```

```shell
kubectl get svc clusterip
```
خروجی مشابه این است:
```
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
clusterip    ClusterIP   10.0.170.92   <none>        80/TCP    51s
```

و به `ClusterIP` از داخل یک پاد در همان خوشه دسترسی دارید:

```shell
kubectl run busybox -it --image=busybox:1.28 --restart=Never --rm
```
خروجی مشابه این است:
```
در حال انتظار برای اجرای پاد پیش‌فرض/busybox، وضعیت در حال انتظار است، پاد آماده است: نادرست
اگر یک خط دستور را ندیدید، سعی کنید کلید وارد را فشار دهید.
```

سپس یک دستور درون این پاد اجرا کنید:

```shell
# این را درون ترمینال از "kubectl run" اجرا کنید
ip addr
```
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue
    پیوند/حلقه 00:00:00:00:00:00 brd 00:00:00

:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
3: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc noqueue
    پیوند/از طریق 0a:58:0a:f4:03:08 brd ff:ff:ff:ff:ff:ff
    inet 10.244.3.8/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::188a:84ff:feb0:26a5/64 scope link
       valid_lft forever preferred_lft forever
```

…سپس از `wget` برای درخواست وب سرور محلی استفاده کنید
```shell
# "10.0.170.92" را با آدرس IPv4 سرویس به نام "clusterip" جایگزین کنید
wget -qO - 10.0.170.92
```
```
مقادیر مشتری:
آدرس_مشتری=10.244.3.8
دستور=GET
...
```
آدرس `آدرس_مشتری` همیشه آدرس IP پاد مشتری است، بدون در نظر گرفتن اینکه پاد مشتری و پاد سرور در یک گره یا گره‌های مختلف قرار دارند.

## آدرس IP منبع برای سرویس‌های با `Type=NodePort`

بسته‌های ارسال شده به سرویس‌های با `Type=NodePort` به طور پیش‌فرض NAT آدرس منبع را انجام می‌دهند. شما می‌توانید این را با ایجاد یک سرویس `NodePort` آزمایش کنید:

```shell
kubectl expose deployment source-ip-app --name=nodeport --port=80 --target-port=8080 --type=NodePort
```
خروجی:

```
service/nodeport exposed
```

```shell
NODEPORT=$(kubectl get -o jsonpath="{.spec.ports[0].nodePort}" services nodeport)
NODES=$(kubectl get nodes -o jsonpath='{ $.items[*].status.addresses[?(@.type=="InternalIP")].address }')
```

اگر بر روی یک ارائه‌دهنده ابری اجرا می‌کنید، ممکن است نیاز به باز کردن یک قانون جداره‌ای برای `nodes:nodeport` داشته باشید که در بالا گزارش شده است. حال می‌توانید سرویس را از بیرون خوشه از طریق پورت گره موجود آزمایش کنید.

```shell
for node in $NODES; do curl -s $node:$NODEPORT | grep -i client_address; done
```
خروجی مشابه این است:

```
client_address=10.180.1.1
client_address=10.240.0.5
client_address=10.240.0.3
```

توجه داشته باشید که این آدرس‌ها، آدرس‌های داخلی خوشه هستند نه آدرس‌های صحیح مشتری. این اتفاق می‌افتد:

* مشتری بسته را به `node2:nodePort` ارسال می‌کند
* `node2` آدرس IP منبع (SNAT) در بسته را با آدرس IP خود جایگزین می‌کند
* `node2` آدرس مقصد بسته را با IP پاد جایگزین می‌کند
* بسته به گره 1 ارسال می‌شود، و سپس به نقطه پایانی
* پاسخ پاد به node2 برگشت داده می‌شود
* پاسخ پاد به مشتری ارسال می‌شود

تصویری از این فرآیند:

{{< figure src="/docs/images/tutor-service-nodePort-fig01.svg" alt="source IP nodeport figure 01" class="diagram-large" caption="شکل. آدرس IP منبع با استفاده از SNAT" link="https://mermaid.live/edit#pako:eNqNkV9rwyAUxb-K3LysYEqS_WFYKAzat9GHdW9zDxKvi9RoMIZtlH732ZjSbE970cu5v3s86hFqJxEYfHjRNeT5ZcUtIbXRaMNN2hZ5vrYRqt52cSXV-4iMSuwkZiYtyX739EqWaahMQ-V1qPxDVLNOvkYrO6fj2dupWMR2iiT6foOKdEZoS5Q2hmVSStoH7w7IMqXUVOefWoaG3XVftHbGeZYVRbH6ZXJ47CeL2-qhxvt_ucTe1SUlpuMN6CX12XeGpLdJiaMMFFr0rdAyvvfxjHEIDbbIgcVSohKDCRy4PUV06KQIuJU6OA9MCdMjBTEEt_-2NbDgB7xAGy3i97VJPP0ABRmcqg" >}}


برای جلوگیری از این اتفاق، Kubernetes یک ویژگی برای
[حفظ آدرس IP منبع مشتری](/docs/tasks/access-application-cluster/create-external-load-balancer/#preserving-the-client-source-ip)
دارد. اگر `service.spec.externalTrafficPolicy` را به مقدار `Local` تنظیم کنید، kube-proxy تنها درخواست‌های proxy را به نقاط پایانی محلی منتقل می‌کند و ترافیک را به سایر گره‌ها ارسال نمی‌کند. این رویکرد آدرس IP اصلی منبع را حفظ می‌کند. اگر نقاط پایانی محلی وجود نداشته باشند، بسته‌های ارسالی به گره‌ها رد می‌شوند، بنابراین می‌توانید بر روی آدرس IP صحیح در هر قانون پردازش بسته که از طریق آن ممکن است با یک بسته پردازش شود، اعتماد کنید.

برای انجام این کار، فیلد `service.spec.externalTrafficPolicy` را به صورت زیر تنظیم کنید:

```shell
kubectl patch svc nodeport -p '{"spec":{"externalTrafficPolicy":"Local"}}'
```
خروجی:

```
service/nodeport patched
```

حالا، تست را دوباره انجام دهید:

```shell
for node in $NODES; do curl --connect-timeout 1 -s $node:$NODEPORT | grep -i client_address; done
```
خروجی مشابه این است:

```
client_address=198.51.100.79
```

توجه داشته باشید که فقط یک پاسخ با آدرس IP صحیح مشتری از یک گره در که پاد نقطه پایانی در آن اجرا می‌شود، دریافت کردید.

این اتفاق می‌افتد:

* مشتری بسته را به `node2:nodePort` ارسال می‌کند، که هیچ نقطه پایانی ندارد
* بسته رد می‌شود
* مشتری بسته را به `node1:nodePort` ارسال می‌کند، که نقطه پایانی دارد
* گره 1 بسته را به نقطه پایانی با آدرس IP صحیح منتقل می‌کند

تصویری از این فرآیند:

{{< figure src="/docs/images/tutor-service-nodePort-fig02.svg" alt="source IP nodeport figure 02" class="diagram-large" caption="شکل. Type=NodePort حفظ آدرس IP منبع مشتری" link="" >}}
```

این ترجمه به شما کمک می‌کند تا بهتر فهمیده و استفاده مناسب از سرویس‌های `NodePort` در Kubernetes داشته باشید. در صورت نیاز به تغییرات بیشتر یا اصلاحات، من در دسترس هستم.

## آدرس IP منبع برای سرویس‌های با `Type=LoadBalancer`

بسته‌های ارسال شده به سرویس‌های با
[`Type=LoadBalancer`](/docs/concepts/services-networking/service/#loadbalancer)
به طور پیش‌فرض NAT آدرس منبع را انجام می‌دهند، زیرا همه گره‌های Kubernetes قابل زمان‌بندی در حالت `Ready` برای ترافیک توازن بار مجاز هستند. بنابراین، اگر بسته‌ها به یک گره برسند که هیچ نقطه پایانی ندارد، سیستم آن را به یک گره با نقطه پایانی منتقل می‌کند و آدرس IP منبع در بسته را با آدرس IP گره جایگزین می‌کند (همانطور که در بخش قبل توضیح داده شد).

می‌توانید این را با ایجاد یک بارگذاری منابع `source-ip-app` از طریق یک بارگذاری منابع آزمایش کنید:

```shell
kubectl expose deployment source-ip-app --name=loadbalancer --port=80 --target-port=8080 --type=LoadBalancer
```
خروجی:

```
service/loadbalancer exposed
```

گرفتن آدرس‌های IP سرویس:

```console
kubectl get svc loadbalancer
```
خروجی مشابه این است:

```
NAME           TYPE           CLUSTER-IP    EXTERNAL-IP       PORT(S)   AGE
loadbalancer   LoadBalancer   10.0.65.118   203.0.113.140     80/TCP    5m
```

سپس، یک درخواست به IP خارجی این سرویس ارسال کنید:

```shell
curl 203.0.113.140
```
خروجی مشابه این است:

```
CLIENT VALUES:
client_address=10.240.0.5
...
```

با این حال، اگر بر روی موتور گوگل Kubernetes Engine/GCE اجرا می‌کنید، تنظیم مقدار `service.spec.externalTrafficPolicy` به `Local` باعث می‌شود که گره‌ها بدون نقطه پایانی خود را از لیست گره‌های مجاز برای ترافیک توازن باری برطرف کنند و این کار با شکست ارزیابی سلامت انجام می‌شود.

بصری:

![Source IP with externalTrafficPolicy](/images/docs/sourceip-externaltrafficpolicy.svg)

می‌توانید این را با تنظیم آناتاسیون زیر آزمایش کنید:

```shell
kubectl patch svc loadbalancer -p '{"spec":{"externalTrafficPolicy":"Local"}}'
```

شما باید به طور فوری مشاهده کنید که فیلد `service.spec.healthCheckNodePort` توسط Kubernetes اختصاص داده شده است:

```shell
kubectl get svc loadbalancer -o yaml | grep -i healthCheckNodePort
```
خروجی مشابه این است:

```yaml
  healthCheckNodePort: 32122
```

فیلد `service.spec.healthCheckNodePort` به یک پورت روی هر گره اشاره دارد که سرویس سلامت را در `/healthz` ارائه می‌دهد. می‌توانید این را آزمایش کنید:

```shell
kubectl get pod -o wide -l app=source-ip-app
```
خروجی مشابه این است:

```
NAME                            READY     STATUS    RESTARTS   AGE       IP             NODE
source-ip-app-826191075-qehz4   1/1       Running   0          20h       10.180.1.136   kubernetes-node-6jst
```

از `curl` برای دریافت اطلاعات `/healthz` روی گره‌های مختلف استفاده کنید:

```shell
# این را به صورت محلی روی یک گره که انتخاب می‌کنید اجرا کنید
curl localhost:32122/healthz
```
```
1 Service Endpoints found
```

در یک گره دیگر، ممکن است نتیجه‌ای متفاوت دریافت کنید:

```shell
# این را به صورت محلی روی یک گره که انتخاب می‌کنید اجرا کنید
curl localhost:32122/healthz
```
```
No Service Endpoints Found
```

یک کنترل‌گر اجرا شده در
{{< glossary_tooltip text="control plane" term_id="control-plane" >}} مسئول تخصیص بارگذار بار ابر است. همچنین
بارگذار آناتاسیونهای HTTP را بر روی هر گره بنظر می‌آورد که روشن است برای یک گره تحقیق های بهره ی شما.

```shell
curl 203.0.113.140
```
The output is similar to this:
```
CLIENT VALUES:
client_address=198.51.100.79
...
```

Here is the Persian translation for the remaining sections:

## پشتیبانی از چندپلتفرم

فقط برخی از ارائه‌دهندگان ابری پشتیبانی از حفظ آدرس IP منبع از طریق سرویس‌های با `Type=LoadBalancer` را ارائه می‌دهند. ارائه‌دهنده ابری که بر روی آن اجرا می‌کنید ممکن است درخواست برای یک بارگذار بار را به چندین روش مختلف انجام دهد:

1. با یک پروکسی که اتصال مشتری را پایان می‌دهد و اتصال جدیدی به گره‌ها/نقاط پایانی شما باز می‌کند. در چنین مواردی، آدرس IP منبع همیشه آدرس IP بارگذار ابری خواهد بود، نه آدرس IP مشتری.

2. با یک فرستنده بسته، به طوری که درخواست‌هایی از مشتری ارسال شده به VIP بارگذار به گره با آدرس IP مشتری ختم می‌شود، نه یک پروکسی میانی.

بارگذارهای بارگذاری در دسته‌بندی اول باید از طریق پروتکلی که بین بارگذار و پشتیبان به منظور ارتباط با آدرس IP واقعی مشتری مانند هدرهای HTTP [Forwarded](https://tools.ietf.org/html/rfc7239#section-5.2)
یا [X-FORWARDED-FOR](https://en.wikipedia.org/wiki/X-Forwarded-For)
یا پروتکل [proxy](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt)
. بارگذارهای در دسته‌بندی دوم می‌توانند از ویژگی مورد نظر فوق با ایجاد یک برچسب هدایت HTTP در پورت ذخیره شده در
میدان  `service.spec.healthCheckNodePort`  بر روی سرویس استفاده کنند.

## {{% heading "cleanup" %}}

حذف سرویس‌ها:

```shell
kubectl delete svc -l app=source-ip-app
```

حذف کردن استقرار، تکثیر و کپی:

```shell
kubectl delete deployment source-ip-app
```


## {{% heading "whatsnext" %}}

* بیشتر در مورد [اتصال برنامه‌ها از طریق سرویس‌ها](/docs/tutorials/services/connect-applications-service/) بیاموزید.
* بخوانید که چگونه [یک بارگذار خارجی ایجاد کنید](/docs/tasks/access-application-cluster/create-external-load-balancer/)
```