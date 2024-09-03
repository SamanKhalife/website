---
reviewers:
- bprashanth
- enisoc
- erictune
- foxish
- janetkuo
- kow3ns
- smarterclayton
title: Running ZooKeeper, A Distributed System Coordinator
content_type: tutorial
weight: 40
---


### معرفی کلی

این آموزش نحوه اجرای [Apache Zookeeper](https://zookeeper.apache.org) را بر روی Kubernetes با استفاده از [StatefulSets](/docs/concepts/workloads/controllers/statefulset/), [PodDisruptionBudgets](/docs/concepts/workloads/pods/disruptions/#pod-disruption-budget), و [PodAntiAffinity](/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity) نمایش می‌دهد.

## {{% heading "پیش‌نیازها" %}}

قبل از شروع این آموزش، باید با مفاهیم Kubernetes زیر آشنا باشید:

- [Pods](/docs/concepts/workloads/pods/)
- [Cluster DNS](/docs/concepts/services-networking/dns-pod-service/)
- [Headless Services](/docs/concepts/services-networking/service/#headless-services)
- [PersistentVolumes](/docs/concepts/storage/persistent-volumes/) 
- [PersistentVolume Provisioning](https://github.com/kubernetes/examples/tree/master/staging/persistent-volume-provisioning/)
- [StatefulSets](/docs/concepts/workloads/controllers/statefulset/)
- [PodDisruptionBudgets](/docs/concepts/workloads/pods/disruptions/#pod-disruption-budget)
- [PodAntiAffinity](/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)
- [kubectl CLI](/docs/reference/kubectl/kubectl/)

شما باید یک کلاستر با حداقل چهار نود داشته باشید، و هر نود نیاز به حداقل 2 واحد پردازنده و 4 گیگابایت حافظه دارد. در این آموزش شما باید نودهای کلاستر خود را مسدود و تخلیه کنید. **این بدان معنی است که کلاستر تمام پادها را در نودهای خود خاتمه داده و به طور موقت غیرقابل برنامه‌ریزی شود.** شما باید از یک کلاستر اختصاصی برای این آموزش استفاده کنید، یا باید اطمینان حاصل کنید که اختلالی که ایجاد می‌کنید تداخلی با سایر اشتراک‌گذاران نداشته باشد.

این آموزش فرض می‌کند که کلاستر شما به صورت پویش دهنده حجم‌های دائمی پیکربندی شده است. اگر کلاستر شما برای این کار پیکربندی نشده است، شما باید قبل از شروع این آموزش سه حجم 20 گیگابایتی را به صورت دستی پیکربندی کنید.

## {{% heading "اهداف" %}}

بعد از اتمام این آموزش، شما موارد زیر را خواهید دانست:

- چگونه یک مجموعه ZooKeeper با استفاده از StatefulSet ایجاد کنید.
- چگونه به طور پیوسته مجموعه را پیکربندی کنید.
- چگونه اجرای سرورهای ZooKeeper را در مجموعه پخش کنید.
- چگونه از PodDisruptionBudget برای اطمینان از در دسترس بودن سرویس در طول نگهداری برنامه‌ریزی شده استفاده کنید.

<!-- lessoncontent -->

### ZooKeeper

[Apache ZooKeeper](https://zookeeper.apache.org/doc/current/) یک خدمت هماهنگ سازی توزیع‌شده و منبع باز برای برنامه‌های توزیع‌شده است. ZooKeeper به شما امکان می‌دهد تا به اطلاعات بخوانید، بنویسید و به‌روزرسانی‌ها را مشاهده کنید. داده‌ها به صورت یک سلسله مراتبی سازماندهی شده هستند و به تمام سرورهای ZooKeeper در مجموعه (یک مجموعه از سرورهای ZooKeeper) کپی می‌شوند. تمام عملیات بر داده‌ها اتمیک و به صورت متوالی است. ZooKeeper این اطمینان را از طریق استفاده از پروتکل کنسانس Zab برای تکثیر یک ماشین حالت را در تمام سرورهای مجموعه اطمینان می‌دهد.

این مجموعه از پروتکل Zab برای انتخاب یک رهبر استفاده می‌کند، و مجموعه نمی‌تواند داده را تا انتخاب کامل کامل نکند. یکبار کامل شدن، مجموعه Zab برای اطمینان از این که تمام نوشته‌ها به یک حجم قبل از اینکه مخاطب به آنها پذیرفته و نمایش دهد را تکثیر می‌کند. بدون احترام به تکثیرهای گروهی، یک تکیه‌گاه مجموعه به عنوان ترکیبی از تمام حاکم و یک تکیه‌گاه دیگر استفاده می‌کند. اگر مجموعه نتواند به یک کاملی برسد، نمی‌تواند داده را بنویسد.

سرورهای ZooKeeper ماشین حالت کامل خود را در حافظه نگهداری می‌کنند، و هر تغییر در یک WAL دائمی (Write Ahead Log) در رسانه ذخیره سازی نوشته می‌شود. هنگامی که یک سرور خراب می‌شود، می‌تواند وضعیت قبلی خود را با بازیابی WAL به بازیابی برگرداند. برای جلوگیری از افزایش WAL بدون محدودیت، سرورهای ZooKeeper این عکس‌العمل‌ها را به طور دوره‌ای به حالت حافظه در رسانه ذخیره می‌کنند. این تصاویر می‌توانند به طور مستقیم در حافظه بارگذاری شوند، و همه ورودی‌های WAL که پیش از تصویر می‌توانند دور شوند.

## ایجاد مجموعه ZooKeeper

مانیفست زیر شام

ل یک
[Headless Service](/docs/concepts/services-networking/service/#headless-services),
یک [Service](/docs/concepts/services-networking/service/),
یک [PodDisruptionBudget](/docs/concepts/workloads/pods/disruptions/#pod-disruption-budgets),
و یک [StatefulSet](/docs/concepts/workloads/controllers/statefulset/) می‌باشد.

{{% code_sample file="application/zookeeper/zookeeper.yaml" %}}

یک ترمینال باز کرده، و از دستور
[`kubectl apply`](/docs/reference/generated/kubectl/kubectl-commands/#apply) برای ایجاد مانیفست استفاده کنید.

```shell
kubectl apply -f https://k8s.io/examples/application/zookeeper/zookeeper.yaml
```

این دستور `zk-hs` Headless Service، `zk-cs` Service، `zk-pdb` PodDisruptionBudget، و `zk` StatefulSet را ایجاد می‌کند.

```
service/zk-hs created
service/zk-cs created
poddisruptionbudget.policy/zk-pdb created
statefulset.apps/zk created
```

از [`kubectl get`](/docs/reference/generated/kubectl/kubectl-commands/#get) برای تماشای کنترلر StatefulSet و ایجاد کردن پادهای StatefulSet استفاده کنید.

```shell
kubectl get pods -w -l app=zk
```

هنگامی که پاد `zk-2` در حالت Running و Ready است، از `CTRL-C` برای خاتمه دادن به kubectl استفاده کنید.

```
NAME      READY     STATUS    RESTARTS   AGE
zk-0      0/1       Pending   0          0s
zk-0      0/1       Pending   0         0s
zk-0      0/1       ContainerCreating   0         0s
zk-0      0/1       Running   0         19s
zk-0      1/1       Running   0         40s
zk-1      0/1       Pending   0         0s
zk-1      0/1       Pending   0         0s
zk-1      0/1       ContainerCreating   0         0s
zk-1      0/1       Running   0         18s
zk-1      1/1       Running   0         40s
zk-2      0/1       Pending   0         0s
zk-2      0/1       Pending   0         0s
zk-2      0/1       ContainerCreating   0         0s
zk-2      0/1       Running   0         19s
zk-2      1/1       Running   0         40s
```

کنترلر StatefulSet سه پاد ایجاد می‌کند، و هر پاد دارای یک کانتینر با سرور ZooKeeper است.

### یاری در انتخاب رهبر

زیرا در یک شبکه ناشناس الگوریتم پایانی برای انتخاب یک رهبر ندارد، Zab نیاز به پیکربندی عضویت صریح برای انجام انتخاب رهبر دارد. هر سرور در مجموعه نیاز به شناسه یکتا است، همه سرورها باید مجموعه جهانی شناسه‌ها را بدانند، و هر شناسه باید با یک آدرس شبکه مرتبط باشد.

از [`kubectl exec`](/docs/reference/generated/kubectl/kubectl-commands/#exec) برای گرفتن نام‌های میزبان پادها در StatefulSet `zk` استفاده کنید.

```shell
for i in 0 1 2; do kubectl exec zk-$i -- hostname; done
```

کنترلر StatefulSet هر پاد را با یک نام میزبان یکتا بر اساس فهرست شماره‌ای آن ایجاد می‌کند. نام‌های میزبان به شکل `<statefulset name>-<ordinal index>` است. اگر فیلد `replicas` از StatefulSet `zk` به `3` تنظیم شود، کنترلر Set سه پاد با نام‌های میزبان `zk-0`، `zk-1`، و `zk-2` ایجاد می‌کند.

```
zk-0
zk-1
zk-2
```

سرورهای در یک مجموعه ZooKeeper از اعداد طبیعی به عنوان شناسه‌های یکتا استفاده می‌کنند، و هر شناسه را در یک پرونده به نام `myid` در دایرکتوری داده سرور ذخیره می‌کنند.

برای بررسی محتوای پرونده `myid` برای هر سرور از دستور زیر استفاده کنید.

```shell
for i in 0 1 2; do echo "myid zk-$i";kubectl exec zk-$i -- cat /var/lib/zookeeper/data/myid; done
```

زیرا شناسه‌ها اعداد طبیعی هستند و شاخص‌های غیر منفی اعداد صحیح هستند، می‌توانید یک شناسه را با افزودن 1 به شاخص تولید کنید.

```
myid zk-0
1
myid zk-1
2
myid zk-2
3
```

برای دریافت Fully Qualified Domain Name (FQDN) هر پاد در StatefulSet `zk` از دستور زیر استفاده کنید.

```shell
for i in 0 1 2; do kubectl exec zk-$i -- hostname -f; done
```

سرویس `zk-hs` یک دامنه برای همه پادها ایجاد می‌کند،
`zk-hs.default.svc.cluster.local`.

```
zk-0.zk-hs.default.svc.cluster.local
zk-1.zk-hs.default.svc.cluster.local
zk-2.zk-hs.default.svc.cluster.local
```

رکوردهای A در [DNS Kubernetes](/docs/concepts/services-networking/dns-pod-service/) FQDNs را به آدرس‌های IP پادها حل می‌کنند. اگر Kubernetes پادها را برنامه‌ریزی مجدد کند، رکوردهای A را با آدرس‌های IP جدید پادها به‌روزرسانی خواهد کرد، اما نام‌های رکوردهای A تغییر نخواهند کرد.

ZooKeeper پیکربندی برنامه‌ریزی خود را در یک پرونده به نام `zoo.cfg` ذخیره می‌کند. از `kubectl exec` برای مشاهده محتوای فایل `zoo.cfg` در پاد `zk-0` استفاده کنید.

```shell
kubectl exec zk-0 -- cat /opt/zookeeper/conf/zoo.cfg
```

در خطوط `server.1`، `server.2`، و `server.3` در پایین فایل، `1`، `2`، و `3` با شناسه‌ها در پرونده‌های `myid` سرورهای ZooKeeper متناظر هستند. آنها برای FQDNs پادها در StatefulSet

 `zk` تنظیم شده‌اند.

```
clientPort=2181
dataDir=/var/lib/zookeeper/data
dataLogDir=/var/lib/zookeeper/log
tickTime=2000
initLimit=10
syncLimit=2000
maxClientCnxns=60
minSessionTimeout= 4000
maxSessionTimeout= 40000
autopurge.snapRetainCount=3
autopurge.purgeInterval=0
server.1=zk-0.zk-hs.default.svc.cluster.local:2888:3888
server.2=zk-1.zk-hs.default.svc.cluster.local:2888:3888
server.3=zk-2.zk-hs.default.svc.cluster.local:2888:3888
```

### تحقق اجماع

پروتکل‌های توافقی نیاز دارند که شناسه‌های هر شرکت‌کننده منحصر به فرد باشند. در پروتکل Zab اجازه نمی‌دهد که دو پاد همردیف با شماره همسان را به عنوان همسان تشخیص دهد. اگر دو پاد با شماره همردیف به وجود آیند، دو سرور ZooKeeper هر دو به عنوان همان سرور شناسایی خود اعلام می‌کنند.

```shell
kubectl get pods -w -l app=zk
```

```
NAME      READY     STATUS    RESTARTS   AGE
zk-0      0/1       Pending   0          0s
zk-0      0/1       Pending   0         0s
zk-0      0/1       ContainerCreating   0         0s
zk-0      0/1       Running   0         19s
zk-0      1/1       Running   0         40s
zk-1      0/1       Pending   0         0s
zk-1      0/1       Pending   0         0s
zk-1      0/1       ContainerCreating   0         0s
zk-1      0/1       Running   0         18s
zk-1      1/1       Running   0         40s
zk-2      0/1       Pending   0         0s
zk-2      0/1       Pending   0         0s
zk-2      0/1       ContainerCreating   0         0s
zk-2      0/1       Running   0         19s
zk-2      1/1       Running   0         40s
```

رکوردهای A برای هر پاد وارد می‌شود هنگامی که پاد آماده می‌شود. بنابراین، FQDNs سرورهای ZooKeeper به یک نقطه پایانی واحد حل می‌شود، و آن نقطه پایانی سرور ZooKeeper یکتا را مدعی می‌شود که هویت تنظیم شده در پرونده `myid` خود را.

```
zk-0.zk-hs.default.svc.cluster.local
zk-1.zk-hs.default.svc.cluster.local
zk-2.zk-hs.default.svc.cluster.local
```

این اطمینان حاصل می‌کند که خصوصیات `servers` در فایل‌های `zoo.cfg` سرورهای ZooKeeper، یک مجموعه به درستی پیکربندی شده را نمایان می‌دهد.

```
server.1=zk-0.zk-hs.default.svc.cluster.local:2888:3888
server.2=zk-1.zk-hs.default.svc.cluster.local:2888:3888
server.3=zk-2.zk-hs.default.svc.cluster.local:2888:3888
```

زمانی که سرورها از پروتکل Zab استفاده می‌کنند تا سعی کنند یک مقدار را تایید کنند، آنها یا موفق به توافق و تایید مقدار می‌شوند (اگر انتخاب رهبر انجام شده باشد و حداقل دو پاد Running و Ready باشد)، یا شکست خواهند خورد (اگر هر یک از شرایط برقرار نباشد). هیچ وضعیتی پیش نمی‌آید که یک سرور به جای یکی دیگری نویسی را به عنوان نماینده پذیرفته باشد.

### تست صحت مجموعه

ساده‌ترین تست صحت، نوشتن داده به یک سرور ZooKeeper و خواندن داده از یک دیگر است.

دستور زیر اسکریپت `zkCli.sh` را اجرا می‌کند تا `world` را به مسیر `/hello` در پاد `zk-0` در مجموعه نویسد.

```shell
kubectl exec zk-0 -- zkCli.sh create /hello world
```

```
WATCHER::

WatchedEvent state:SyncConnected type:None path:null
Created /hello
```

برای دریافت داده از پاد `zk-1` از دستور زیر استفاده کنید.

```shell
kubectl exec zk-1 -- zkCli.sh get /hello
```

داده‌ای که در `zk-0` ایجاد کردید، در تمام سرورهای مجموعه در دسترس است.

```
WATCHER::

WatchedEvent state:SyncConnected type:None path:null
world
cZxid = 0x100000002
ctime = Thu Dec 08 15:13:30 UTC 2016
mZxid = 0x100000002
mtime = Thu Dec 08 15:13:30 UTC 2016
pZxid = 0x100000002
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
```

### ارائه ذخیره‌سازی دائمی

همانطور که در بخش [مبانی ZooKeeper](#zookeeper) ذکر شد، ZooKeeper تمام ورودی‌ها را در یک WAL دائمی تعهد می‌کند و به طور دوره‌ای عکس‌العمل‌های حافظه‌ای را به رسانه‌های ذخیره‌سازی می‌نویسد. استفاده از WAL برای ارائه دوام یک تکنیک متداول برای برنامه‌هایی است که از پروتکل‌های توافق برای دستیابی به یک ماشین وضعیت مکرر استفاده می‌کنند.

از دستور [`kubectl delete`](/docs/reference/generated/kubectl/kubectl-commands/#delete) برای حذف `StatefulSet` zk استفاده کنید.

```shell
kubectl delete statefulset zk
```

```
statefulset.apps "zk" deleted
```

نگاهی به پایان پذیرفتن Pods در `StatefulSet` بندازید.

```shell
kubectl get pods -w -l app=zk
```

وقتی `zk-0` کاملاً خاتمه یافت، از `CTRL-C` برای خاتمه دادن به `kubectl` استفاده کنید.

```
zk-2      1/1       Terminating   0         9m
zk-0      1/1       Terminating   0         11m
zk-1      1/1       Terminating   0         10m
zk-2      0/1       Terminating   0         9m
zk-2      0/1       Terminating   0         9m
zk-2      0/1       Terminating   0         9m
zk-1      0/1       Terminating   0         10m
zk-1      0/1       Terminating   0         10m
zk-1      0/1       Terminating   0         10m
zk-0      0/1       Terminating   0         11m
zk-0      0/1       Terminating   0         11m
zk-0      0/1       Terminating   0         11m
```

مانیفست `zookeeper.yaml` را مجدداً اعمال کنید.

```shell
kubectl apply -f https://k8s.io/examples/application/zookeeper/zookeeper.yaml
```

این کار، شیء `StatefulSet` zk را ایجاد می‌کند، اما سایر اشیاء API در مانیفست تغییری نخواهند کرد زیرا که از قبل وجود دارند.

نظارت بر کنترل‌کننده `StatefulSet` و بازسازی پاد‌های `StatefulSet`.

```shell
kubectl get pods -w -l app=zk
```

وقتی که پاد `zk-2` در وضعیت Running و Ready قرار گرفت، از `CTRL-C` برای خاتمه دادن به `kubectl` استفاده کنید.

```
NAME      READY     STATUS    RESTARTS   AGE
zk-0      0/1       Pending   0          0s
zk-0      0/1       Pending   0         0s
zk-0      0/1       ContainerCreating   0         0s
zk-0      0/1       Running   0         19s
zk-0      1/1       Running   0         40s
zk-1      0/1       Pending   0         0s
zk-1      0/1       Pending   0         0s
zk-1      0/1       ContainerCreating   0         0s
zk-1      0/1       Running   0         18s
zk-1      1/1       Running   0         40s
zk-2      0/1       Pending   0         0s
zk-2      0/1       Pending   0         0s
zk-2      0/1       ContainerCreating   0         0s
zk-2      0/1       Running   0         19s
zk-2      1/1       Running   0         40s
```

از دستور زیر برای دریافت مقداری که در [تست صحت](#sanity-testing-the-ensemble) وارد کرده‌اید، از پاد `zk-2` استفاده کنید.

```shell
kubectl exec zk-2 zkCli.sh get /hello
```

حتی اگر شما تمام پاد‌های `StatefulSet` zk را خاتمه داده و مجدداً ایجاد کرده باشید، مجموعه همچنان مقدار اصلی را ارائه می‌دهد.

```
WATCHER::

WatchedEvent state:SyncConnected type:None path:null
world
cZxid = 0x100000002
ctime = Thu Dec 08 15:13:30 UTC 2016
mZxid = 0x100000002
mtime = Thu Dec 08 15:13:30 UTC 2016
pZxid = 0x100000002
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
```

بخش `volumeClaimTemplates` از `spec` `StatefulSet` zk یک `PersistentVolume` را برای هر پاد مشخص می‌کند.

```yaml
volumeClaimTemplates:
  - metadata:
      name: datadir
      annotations:
        volume.alpha.kubernetes.io/storage-class: anything
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 20Gi
```

کنترل‌کننده `StatefulSet` برای هر پاد، یک `PersistentVolumeClaim` تولید می‌کند.

از دستور زیر برای دریافت `PersistentVolumeClaims` `StatefulSet` استفاده کنید.

```shell
kubectl get pvc -l app=zk
```

زمانی که `StatefulSet` پاد‌های خود را بازسازی می‌کند، `PersistentVolumes` پاد‌ها را دوباره می‌نهد.

```
NAME           STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
datadir-zk-0   Bound     pvc-bed742cd-bcb1-11e6-994f-42010a800002   20Gi       RWO           1h
datadir-zk-1   Bound     pvc-bedd27d2-bcb1-11e6-994f-42010a800002   20Gi       RWO           1h
datadir-zk-2   Bound     pvc-bee0817e-bcb1-11e6-994f-42010a800002   20Gi       RWO           1h
```

بخش `volumeMounts` از `template` ظرفینه کانتینر `StatefulSet` مانت `PersistentVolumes` را در دایرکتوری‌های داده سرورهای ZooKeeper می‌کند.

```yaml
volumeMounts:
- name: datadir
  mountPath: /var/lib/zookeeper
```
هنگامی که یک پاد در `zk` `StatefulSet` مجدداً برنامه‌ریزی (re-scheduled) می‌شود، همیشه همان `PersistentVolume` به دایرکتوری داده‌های سرور ZooKeeper متصل می‌شود. حتی زمانی که پادها مجدداً برنامه‌ریزی می‌شوند، تمامی نوشتارهایی که به WAL سرورهای ZooKeeper انجام شده و تمامی عکس‌های فوری (snapshots) آنها همچنان پایدار می‌مانند.

## اطمینان از پیکربندی یکپارچه

همانطور که در بخش‌های [تسهیل انتخابات رهبر](#facilitating-leader-election) و [دستیابی به توافق](#achieving-consensus) ذکر شد، سرورهای یک مجموعه ZooKeeper نیاز به پیکربندی یکپارچه برای انتخاب یک رهبر و تشکیل یک گروه (quorum) دارند. آنها همچنین نیاز به پیکربندی یکپارچه پروتکل Zab دارند تا این پروتکل بتواند به درستی روی شبکه کار کند. در مثال ما، پیکربندی یکپارچه با جاسازی پیکربندی مستقیماً در مانیفست حاصل می‌شود.

دریافت `zk` StatefulSet.

```shell
kubectl get sts zk -o yaml
```

```
…
command:
      - sh
      - -c
      - "start-zookeeper \
        --servers=3 \
        --data_dir=/var/lib/zookeeper/data \
        --data_log_dir=/var/lib/zookeeper/data/log \
        --conf_dir=/opt/zookeeper/conf \
        --client_port=2181 \
        --election_port=3888 \
        --server_port=2888 \
        --tick_time=2000 \
        --init_limit=10 \
        --sync_limit=5 \
        --heap=512M \
        --max_client_cnxns=60 \
        --snap_retain_count=3 \
        --purge_interval=12 \
        --max_session_timeout=40000 \
        --min_session_timeout=4000 \
        --log_level=INFO"
…
```

دستور مورد استفاده برای راه‌اندازی سرورهای ZooKeeper پیکربندی را به عنوان پارامتر خط فرمان منتقل می‌کند. همچنین می‌توانید از متغیرهای محیطی برای انتقال پیکربندی به مجموعه استفاده کنید.

### پیکربندی لاگ‌ها

یکی از فایل‌هایی که توسط اسکریپت `zkGenConfig.sh` تولید می‌شود، لاگ‌های ZooKeeper را کنترل می‌کند. ZooKeeper از [Log4j](https://logging.apache.org/log4j/2.x/) استفاده می‌کند و به طور پیش‌فرض، از یک appender فایل رولینگ بر اساس زمان و اندازه برای پیکربندی لاگ‌های خود استفاده می‌کند.

از دستور زیر برای دریافت پیکربندی لاگ از یکی از پادها در `zk` StatefulSet استفاده کنید.

```shell
kubectl exec zk-0 cat /usr/etc/zookeeper/log4j.properties
```

پیکربندی لاگ زیر باعث می‌شود که فرآیند ZooKeeper تمام لاگ‌های خود را به جریان خروجی استاندارد بنویسد.

```
zookeeper.root.logger=CONSOLE
zookeeper.console.threshold=INFO
log4j.rootLogger=${zookeeper.root.logger}
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.Threshold=${zookeeper.console.threshold}
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} [myid:%X{myid}] - %-5p [%t:%C{1}@%L] - %m%n
```

این ساده‌ترین روش ممکن برای لاگ‌گیری امن داخل کانتینر است. به دلیل اینکه برنامه‌ها لاگ‌ها را به خروجی استاندارد می‌نویسند، Kubernetes مدیریت چرخش لاگ را برای شما انجام می‌دهد. Kubernetes همچنین یک سیاست حفظ منطقی را اجرا می‌کند که اطمینان می‌دهد لاگ‌های برنامه که به خروجی استاندارد و خطای استاندارد نوشته می‌شوند، فضای ذخیره‌سازی محلی را تمام نمی‌کنند.

از [`kubectl logs`](/docs/reference/generated/kubectl/kubectl-commands/#logs) برای بازیابی آخرین ۲۰ خط لاگ از یکی از پادها استفاده کنید.

```shell
kubectl logs zk-0 --tail 20
```

می‌توانید لاگ‌های برنامه‌ای که به خروجی استاندارد یا خطای استاندارد نوشته شده‌اند را با استفاده از `kubectl logs` و از طریق داشبورد Kubernetes مشاهده کنید.

```
2016-12-06 19:34:16,236 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@827] - Processing ruok command from /127.0.0.1:52740
2016-12-06 19:34:16,237 [myid:1] - INFO  [Thread-1136:NIOServerCnxn@1008] - Closed socket connection for client /127.0.0.1:52740 (no session established for client)
2016-12-06 19:34:26,155 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxnFactory@192] - Accepted socket connection from /127.0.0.1:52749
2016-12-06 19:34:26,155 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@827] - Processing ruok command from /127.0.0.1:52749
2016-12-06 19:34:26,156 [myid:1] - INFO  [Thread-1137:NIOServerCnxn@1008] - Closed socket connection for client /127.0.0.1:52749 (no session established for client)
2016-12-06 19:34:26,222 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@827] - Accepted socket connection from /127.0.0.1:52750
2016-12-06 19:34:26,222 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@827] - Processing ruok command from /127.0.0.1:52750
2016-12-06 19:34:26,226 [myid:1] - INFO  [Thread-1138:NIOServerCnxn@1008] - Closed socket connection for client /127.0.0.1:52750 (no session established for client)
2016-12-06 19:34:36,151 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxnFactory@192] - Accepted socket connection from /127.0.0.1:52760
2016-12-06 19:34:36,152 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@827] - Processing ruok command from /127.0.0.1:52760
2016-12-06 19:34:36,152 [myid:1] - INFO  [Thread-1139:NIOServerCnxn@1008] - Closed socket connection for client /127.0.0.1:52760 (no session established for client)
2016-12-06 19:34:36,230 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxnFactory@192] - Accepted socket connection from /127.0.0.1:52761
2016-12-06 19:34:36,231 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@827] - Processing ruok command from /127.0.0.1:52761
2016-12-06 19:34:36,231 [myid:1] - INFO  [Thread-1140:NIOServerCnxn@1008] - Closed socket connection for client /127.0.0.1:52761 (no session established for client)
2016-12-06 19:34:46,149 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerC

nxnFactory@192] - Accepted socket connection from /127.0.0.1:52767
2016-12-06 19:34:46,149 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@827] - Processing ruok command from /127.0.0.1:52767
2016-12-06 19:34:46,149 [myid:1] - INFO  [Thread-1141:NIOServerCnxn@1008] - Closed socket connection for client /127.0.0.1:52767 (no session established for client)
2016-12-06 19:34:46,230 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxnFactory@192] - Accepted socket connection from /127.0.0.1:52768
2016-12-06 19:34:46,230 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@827] - Processing ruok command from /127.0.0.1:52768
2016-12-06 19:34:46,230 [myid:1] - INFO  [Thread-1142:NIOServerCnxn@1008] - Closed socket connection for client /127.0.0.1:52768 (no session established for client)
```

Kubernetes با بسیاری از راه‌حل‌های لاگ‌گیری یکپارچه می‌شود. می‌توانید یک راه‌حل لاگ‌گیری را انتخاب کنید که به بهترین نحو به خوشه و برنامه‌های شما متناسب باشد. برای لاگ‌گیری و تجمیع در سطح خوشه، در نظر بگیرید یک [کانتینر جانبی](/docs/concepts/cluster-administration/logging#sidecar-container-with-logging-agent) برای چرخش و ارسال لاگ‌های خود مستقر کنید.

### پیکربندی کاربر غیرمجاز

بهترین روش‌ها برای اجازه دادن به یک برنامه برای اجرا به عنوان یک کاربر مجاز داخل یک کانتینر موضوع بحث است. اگر سازمان شما نیاز دارد که برنامه‌ها به عنوان یک کاربر غیرمجاز اجرا شوند، می‌توانید از [SecurityContext](/docs/tasks/configure-pod-container/security-context/) استفاده کنید تا کنترل کنید که نقطه ورودی به عنوان کدام کاربر اجرا شود.

`StatefulSet` پاد `zk` شامل یک `SecurityContext` است.

```yaml
securityContext:
  runAsUser: 1000
  fsGroup: 1000
```

در کانتینرهای پاد، UID 1000 به کاربر zookeeper و GID 1000 به گروه zookeeper مربوط است.

دریافت اطلاعات فرآیند ZooKeeper از پاد `zk-0`.

```shell
kubectl exec zk-0 -- ps -elf
```

از آنجا که فیلد `runAsUser` از شیء `securityContext` به 1000 تنظیم شده است، به جای اجرای به عنوان root، فرآیند ZooKeeper به عنوان کاربر zookeeper اجرا می‌شود.

```
F S UID        PID  PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
4 S zookeep+     1     0  0  80   0 -  1127 -      20:46 ?        00:00:00 sh -c zkGenConfig.sh && zkServer.sh start-foreground
0 S zookeep+    27     1  0  80   0 - 1155556 -    20:46 ?        00:00:19 /usr/lib/jvm/java-8-openjdk-amd64/bin/java -Dzookeeper.log.dir=/var/log/zookeeper -Dzookeeper.root.logger=INFO,CONSOLE -cp /usr/bin/../build/classes:/usr/bin/../build/lib/*.jar:/usr/bin/../share/zookeeper/zookeeper-3.4.9.jar:/usr/bin/../share/zookeeper/slf4j-log4j12-1.6.1.jar:/usr/bin/../share/zookeeper/slf4j-api-1.6.1.jar:/usr/bin/../share/zookeeper/netty-3.10.5.Final.jar:/usr/bin/../share/zookeeper/log4j-1.2.16.jar:/usr/bin/../share/zookeeper/jline-0.9.94.jar:/usr/bin/../src/java/lib/*.jar:/usr/bin/../etc/zookeeper: -Xmx2G -Xms2G -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /usr/bin/../etc/zookeeper/zoo.cfg
```

به طور پیش‌فرض، هنگامی که PersistentVolumes پاد به دایرکتوری داده‌های سرور ZooKeeper متصل می‌شود، فقط توسط کاربر root قابل دسترسی است. این پیکربندی مانع از نوشتن فرآیند ZooKeeper به WAL و ذخیره عکس‌های فوری آن می‌شود.

از دستور زیر برای دریافت مجوزهای فایل دایرکتوری داده‌های ZooKeeper در پاد `zk-0` استفاده کنید.

```shell
kubectl exec -ti zk-0 -- ls -ld /var/lib/zookeeper/data
```

به دلیل اینکه فیلد `fsGroup` از شیء `securityContext` به 1000 تنظیم شده است، مالکیت PersistentVolumes پادها به گروه zookeeper تنظیم می‌شود و فرآیند ZooKeeper قادر به خواندن و نوشتن داده‌های خود است.

```
drwxr-sr-x 3 zookeeper zookeeper 4096 Dec  5 20:45 /var/lib/zookeeper/data
```## مدیریت فرآیند ZooKeeper

[مستندات ZooKeeper](https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_supervision) ذکر می‌کند که "شما نیاز دارید که یک فرآیند نظارتی داشته باشید که هر یک از فرآیندهای سرور ZooKeeper شما (JVM) را مدیریت کند." استفاده از یک فرآیند نظارتی برای راه‌اندازی مجدد فرآیندهای شکست‌خورده در یک سیستم توزیع‌شده یک الگوی رایج است. هنگام استقرار یک برنامه در Kubernetes، به جای استفاده از یک ابزار خارجی به عنوان فرآیند نظارتی، باید از Kubernetes به عنوان ناظر برای برنامه خود استفاده کنید.

### به‌روزرسانی ensemble

`StatefulSet` مربوط به `zk` به گونه‌ای پیکربندی شده است که از استراتژی به‌روزرسانی `RollingUpdate` استفاده کند.

شما می‌توانید با استفاده از `kubectl patch` تعداد `cpu` های تخصیص داده شده به سرورها را به‌روزرسانی کنید.

```shell
kubectl patch sts zk --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/resources/requests/cpu", "value":"0.3"}]'
```

```
statefulset.apps/zk patched
```

از `kubectl rollout status` برای مشاهده وضعیت به‌روزرسانی استفاده کنید.

```shell
kubectl rollout status sts/zk
```

```
waiting for statefulset rolling update to complete 0 pods at revision zk-5db4499664...
Waiting for 1 pods to be ready...
Waiting for 1 pods to be ready...
waiting for statefulset rolling update to complete 1 pods at revision zk-5db4499664...
Waiting for 1 pods to be ready...
Waiting for 1 pods to be ready...
waiting for statefulset rolling update to complete 2 pods at revision zk-5db4499664...
Waiting for 1 pods to be ready...
Waiting for 1 pods to be ready...
statefulset rolling update complete 3 pods at revision zk-5db4499664...
```

این کار به ترتیب یکی یکی پادها را خاتمه می‌دهد و آنها را با پیکربندی جدید ایجاد می‌کند. این کار اطمینان می‌دهد که اکثریت در حین به‌روزرسانی حفظ می‌شود.

از دستور `kubectl rollout history` برای مشاهده تاریخچه یا پیکربندی‌های قبلی استفاده کنید.

```shell
kubectl rollout history sts/zk
```

خروجی مشابه این است:

```
statefulsets "zk"
REVISION
1
2
```

از دستور `kubectl rollout undo` برای بازگرداندن تغییر استفاده کنید.

```shell
kubectl rollout undo sts/zk
```

خروجی مشابه این است:

```
statefulset.apps/zk rolled back
```

### مدیریت خطاهای فرآیند

[سیاست‌های راه‌اندازی مجدد](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) کنترل می‌کنند که چگونه Kubernetes خطاهای فرآیند را برای نقطه ورود به کانتینر در یک پاد مدیریت می‌کند. برای پادها در `StatefulSet`، تنها `RestartPolicy` مناسب `Always` است و این مقدار پیش‌فرض است. برای برنامه‌های حالت‌مند، هرگز نباید سیاست پیش‌فرض را تغییر دهید.

از دستور زیر برای بررسی درخت فرآیند برای سرور ZooKeeper که در پاد `zk-0` اجرا می‌شود، استفاده کنید.

```shell
kubectl exec zk-0 -- ps -ef
```

دستور استفاده‌شده به عنوان نقطه ورود به کانتینر دارای PID 1 است و فرآیند ZooKeeper که یک فرآیند فرزند نقطه ورود است، دارای PID 27 است.

```
UID        PID  PPID  C STIME TTY          TIME CMD
zookeep+     1     0  0 15:03 ?        00:00:00 sh -c zkGenConfig.sh && zkServer.sh start-foreground
zookeep+    27     1  0 15:03 ?        00:00:03 /usr/lib/jvm/java-8-openjdk-amd64/bin/java -Dzookeeper.log.dir=/var/log/zookeeper -Dzookeeper.root.logger=INFO,CONSOLE -cp /usr/bin/../build/classes:/usr/bin/../build/lib/*.jar:/usr/bin/../share/zookeeper/zookeeper-3.4.9.jar:/usr/bin/../share/zookeeper/slf4j-log4j12-1.6.1.jar:/usr/bin/../share/zookeeper/slf4j-api-1.6.1.jar:/usr/bin/../share/zookeeper/netty-3.10.5.Final.jar:/usr/bin/../share/zookeeper/log4j-1.2.16.jar:/usr/bin/../share/zookeeper/jline-0.9.94.jar:/usr/bin/../src/java/lib/*.jar:/usr/bin/../etc/zookeeper: -Xmx2G -Xms2G -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /usr/bin/../etc/zookeeper/zoo.cfg
```

در یک ترمینال دیگر، پادهای `StatefulSet` را با دستور زیر مشاهده کنید.

```shell
kubectl get pod -w -l app=zk
```

در یک ترمینال دیگر، فرآیند ZooKeeper را در پاد `zk-0` با دستور زیر خاتمه دهید.

```shell
kubectl exec zk-0 -- pkill java
```

خاتمه فرآیند ZooKeeper باعث خاتمه فرآیند والد آن شد. از آنجا که `RestartPolicy` کانتینر `Always` است، فرآیند والد مجدداً راه‌اندازی شد.

```
NAME      READY     STATUS    RESTARTS   AGE
zk-0      1/1       Running   0          21m
zk-1      1/1       Running   0          20m
zk-2      1/1       Running   0          19m
NAME      READY     STATUS    RESTARTS   AGE
zk-0      0/1       Error     0          29m
zk-0      0/1       Running   1         29m
zk-0      1/1       Running   1         29m
```

اگر برنامه شما از یک اسکریپت (مانند `zkServer.sh`) برای راه‌اندازی فرآیندی که منطق کسب و کار برنامه را پیاده‌سازی می‌کند استفاده می‌کند، اسکریپت باید با فرآیند فرزند خاتمه یابد. این اطمینان می‌دهد که Kubernetes کانتینر برنامه را زمانی که فرآیند پیاده‌سازی منطق کسب و کار برنامه شکست می‌خورد، مجدداً راه‌اندازی می‌کند.

### آزمایش برای زندگی

پیکربندی برنامه شما برای راه‌اندازی مجدد فرآیندهای شکست‌خورده کافی نیست تا یک سیستم توزیع‌شده سالم نگه داشته شود. سناریوهایی وجود دارد که فرآیندهای سیستم می‌توانند هم زنده و هم غیرپاسخگو یا به هر نحو دیگر ناسالم باشند. شما باید از پروب‌های زنده بودن استفاده کنید تا به Kubernetes اطلاع دهید که فرآیندهای برنامه شما ناسالم هستند و باید آنها را مجدداً راه‌اندازی کند.

قالب پاد برای `StatefulSet` مشخص‌کننده یک پروب زنده بودن است.

```yaml
  livenessProbe:
    exec:
      command:
      - sh
      - -c
      - "zookeeper-ready 2181"
    initialDelaySeconds: 15
    timeoutSeconds: 5
```

پروب یک اسکریپت bash را فراخوانی می‌کند که از کلمه چهار حرفی `ruok` ZooKeeper برای تست سلامتی سرور استفاده می‌کند.

```
OK=$(echo ruok | nc 127.0.0.1 $1)
if [ "$OK" == "imok" ]; then
    exit 0
else
    exit 1
fi
```

در یک پنجره ترمینال، از دستور زیر برای مشاهده پادهای `StatefulSet` استفاده کنید.

```shell
kubectl get pod -w -l app=zk
```

در یک پنجره دیگر، از دستور زیر برای حذف اسکریپت `zookeeper-ready` از سیستم فایل پاد `zk-0` استفاده کنید.

```shell
kubectl exec zk-0 -- rm /opt/zookeeper/bin/zookeeper-ready
```

هنگامی که پروب زنده بودن برای فرآیند ZooKeeper شکست می‌خورد، Kubernetes به طور خودکار فرآیند را برای شما مجدداً راه‌اندازی می‌کند، اطمینان حاصل می‌کند که فرآیندهای ناسالم در ensemble مجدداً راه‌اندازی می‌شوند.

```shell
kubectl get pod -w -l app=zk
```

```
NAME      READY     STATUS    RESTARTS   AGE
zk-0      1/1       Running   0          1h
zk-1      1/1       Running   0          1h
zk-2      1/1       Running   0          1h
NAME      READY     STATUS

    RESTARTS   AGE
zk-0      0/1       Running   0          1h
zk-0      0/1       Running   1         1h
zk-0      1/1       Running   1         1h
```
### آزمایش آمادگی

آمادگی با زنده بودن یکسان نیست. اگر یک فرآیند زنده است، به معنای آن است که برنامه‌ریزی شده و سالم است. اگر یک فرآیند آماده است، به این معناست که قادر به پردازش ورودی است. زنده بودن یک شرط لازم است، اما کافی برای آمادگی نیست. مواردی وجود دارد، به ویژه در طول مراحل اولیه و خاتمه، که یک فرآیند می‌تواند زنده باشد اما آماده نباشد.

اگر یک پروب آمادگی تعیین کنید، Kubernetes اطمینان حاصل خواهد کرد که فرآیندهای برنامه شما تا زمانی که بررسی‌های آمادگی آنها عبور نکرده‌اند، ترافیک شبکه را دریافت نخواهند کرد.

برای سرور ZooKeeper، زنده بودن به معنای آمادگی است. بنابراین، پروب آمادگی در فایل `zookeeper.yaml` مشابه پروب زنده بودن است.

```yaml
  readinessProbe:
    exec:
      command:
      - sh
      - -c
      - "zookeeper-ready 2181"
    initialDelaySeconds: 15
    timeoutSeconds: 5
```

با اینکه پروب‌های زنده بودن و آمادگی یکسان هستند، مهم است که هر دو را تعیین کنید. این اطمینان می‌دهد که تنها سرورهای سالم در مجموعه ZooKeeper ترافیک شبکه را دریافت می‌کنند.

## تحمل خطای نود

ZooKeeper نیاز به یک اکثریت از سرورها دارد تا به طور موفقیت‌آمیزی تغییرات داده را ثبت کند. برای یک مجموعه سه سروره، دو سرور باید سالم باشند تا نوشتن‌ها موفق شوند. در سیستم‌های مبتنی بر اکثریت، اعضا در حوزه‌های خرابی مختلف مستقر می‌شوند تا اطمینان حاصل شود که سیستم همیشه در دسترس است. برای جلوگیری از بروز خطا به دلیل از دست دادن یک ماشین خاص، بهترین شیوه‌ها قرار دادن چندین نمونه از برنامه در یک ماشین واحد را ممنوع می‌کنند.

به طور پیش‌فرض، Kubernetes ممکن است پادهای یک `StatefulSet` را در یک نود مشترک قرار دهد. برای مجموعه سه سروره‌ای که ایجاد کرده‌اید، اگر دو سرور در یک نود قرار گیرند و آن نود دچار خرابی شود، مشتریان سرویس ZooKeeper شما تا زمانی که حداقل یکی از پادها مجدداً برنامه‌ریزی نشود، با اختلال مواجه خواهند شد.

شما باید همیشه ظرفیت اضافی فراهم کنید تا اجازه دهید فرآیندهای سیستم‌های حیاتی در صورت خرابی نودها مجدداً برنامه‌ریزی شوند. اگر این کار را انجام دهید، اختلال تنها تا زمانی طول خواهد کشید که برنامه‌ریز Kubernetes یکی از سرورهای ZooKeeper را مجدداً برنامه‌ریزی کند. با این حال، اگر می‌خواهید سرویس شما در صورت خرابی نودها بدون هیچ وقفه‌ای ادامه یابد، باید `podAntiAffinity` را تنظیم کنید.

از دستور زیر برای گرفتن نودها برای پادهای `StatefulSet` در `zk` استفاده کنید.

```shell
for i in 0 1 2; do kubectl get pod zk-$i --template {{.spec.nodeName}}; echo ""; done
```

تمام پادهای `StatefulSet` در `zk` در نودهای مختلف مستقر هستند.

```
kubernetes-node-cxpk
kubernetes-node-a5aq
kubernetes-node-2g2d
```

این به این دلیل است که پادهای `StatefulSet` در `zk` دارای `PodAntiAffinity` مشخص شده‌اند.

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: "app"
              operator: In
              values:
                - zk
        topologyKey: "kubernetes.io/hostname"
```

فیلد `requiredDuringSchedulingIgnoredDuringExecution` به برنامه‌ریز Kubernetes می‌گوید که هرگز نباید دو پاد با label `app` به عنوان `zk` را در دامنه تعریف‌شده توسط `topologyKey` هم‌مکان کند. `topologyKey` `kubernetes.io/hostname` نشان می‌دهد که دامنه یک نود جداگانه است. با استفاده از قوانین، labelها و selectorهای مختلف، می‌توانید از این تکنیک برای پخش مجموعه خود در دامنه‌های خرابی فیزیکی، شبکه و برق استفاده کنید.
## زنده ماندن در هنگام نگهداری

در این بخش، نودها را قرنطینه و تخلیه خواهید کرد. اگر از این آموزش در یک خوشه مشترک استفاده می‌کنید، مطمئن شوید که این کار بر روی دیگر کاربران تأثیر منفی نداشته باشد.

بخش قبلی نشان داد که چگونه پادهای خود را در نودها پخش کنید تا از خرابی‌های برنامه‌ریزی نشده نودها جان سالم به در ببرید، اما همچنین باید برای خرابی‌های موقت نودها که به دلیل نگهداری برنامه‌ریزی شده رخ می‌دهد نیز برنامه‌ریزی کنید.

از این دستور برای دریافت نودهای خوشه خود استفاده کنید.

```shell
kubectl get nodes
```

این آموزش یک خوشه با حداقل چهار نود را فرض می‌کند. اگر خوشه بیش از چهار نود دارد، از دستور [`kubectl cordon`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands/#cordon) برای قرنطینه کردن همه نودها به جز چهار نود استفاده کنید. محدود کردن به چهار نود باعث می‌شود که Kubernetes هنگام برنامه‌ریزی پادهای zookeeper با محدودیت‌های affinity و PodDisruptionBudget مواجه شود.

```shell
kubectl cordon <node-name>
```

از این دستور برای دریافت `PodDisruptionBudget` با نام `zk-pdb` استفاده کنید.

```shell
kubectl get pdb zk-pdb
```

فیلد `max-unavailable` به Kubernetes می‌گوید که حداکثر یک پاد از `StatefulSet` `zk` می‌تواند در هر زمان غیرفعال باشد.

```
NAME      MIN-AVAILABLE   MAX-UNAVAILABLE   ALLOWED-DISRUPTIONS   AGE
zk-pdb    N/A             1                 1
```

در یک ترمینال، از این دستور برای مشاهده پادهای `StatefulSet` در `zk` استفاده کنید.

```shell
kubectl get pods -w -l app=zk
```

در یک ترمینال دیگر، از این دستور برای دریافت نودهایی که پادها در حال حاضر بر روی آنها برنامه‌ریزی شده‌اند استفاده کنید.

```shell
for i in 0 1 2; do kubectl get pod zk-$i --template {{.spec.nodeName}}; echo ""; done
```

خروجی مشابه این خواهد بود:

```
kubernetes-node-pb41
kubernetes-node-ixsl
kubernetes-node-i4c4
```

از دستور [`kubectl drain`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands/#drain) برای قرنطینه و تخلیه نودی که پاد `zk-0` بر روی آن برنامه‌ریزی شده است، استفاده کنید.

```shell
kubectl drain $(kubectl get pod zk-0 --template {{.spec.nodeName}}) --ignore-daemonsets --force --delete-emptydir-data
```

خروجی مشابه این خواهد بود:

```
node "kubernetes-node-pb41" cordoned

WARNING: Deleting pods not managed by ReplicationController, ReplicaSet, Job, or DaemonSet: fluentd-cloud-logging-kubernetes-node-pb41, kube-proxy-kubernetes-node-pb41; Ignoring DaemonSet-managed pods: node-problem-detector-v0.1-o5elz
pod "zk-0" deleted
node "kubernetes-node-pb41" drained
```

با توجه به اینکه در خوشه شما چهار نود وجود دارد، دستور `kubectl drain` موفقیت‌آمیز خواهد بود و `zk-0` در نود دیگری مجدداً برنامه‌ریزی می‌شود.

```
NAME      READY     STATUS    RESTARTS   AGE
zk-0      1/1       Running   2          1h
zk-1      1/1       Running   0          1h
zk-2      1/1       Running   0          1h
NAME      READY     STATUS        RESTARTS   AGE
zk-0      1/1       Terminating   2          2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Pending   0         0s
zk-0      0/1       Pending   0         0s
zk-0      0/1       ContainerCreating   0         0s
zk-0      0/1       Running   0         51s
zk-0      1/1       Running   0         1m
```

مشاهده پادهای `StatefulSet` را در ترمینال اول ادامه دهید و نودی که `zk-1` بر روی آن برنامه‌ریزی شده است را تخلیه کنید.

```shell
kubectl drain $(kubectl get pod zk-1 --template {{.spec.nodeName}}) --ignore-daemonsets --force --delete-emptydir-data
```

خروجی مشابه این خواهد بود:

```
"kubernetes-node-ixsl" cordoned
WARNING: Deleting pods not managed by ReplicationController, ReplicaSet, Job, or DaemonSet: fluentd-cloud-logging-kubernetes-node-ixsl, kube-proxy-kubernetes-node-ixsl; Ignoring DaemonSet-managed pods: node-problem-detector-v0.1-voc74
pod "zk-1" deleted
node "kubernetes-node-ixsl" drained
```

پاد `zk-1` نمی‌تواند برنامه‌ریزی شود زیرا `StatefulSet` `zk` دارای یک قانون `PodAntiAffinity` است که هم‌مکانی پادها را ممنوع می‌کند، و چون تنها دو نود قابل برنامه‌ریزی هستند، پاد در حالت Pending باقی می‌ماند.

```shell
kubectl get pods -w -l app=zk
```

خروجی مشابه این خواهد بود:

```
NAME      READY     STATUS    RESTARTS   AGE
zk-0      1/1       Running   2          1h
zk-1      1/1       Running   0          1h
zk-2      1/1       Running   0          1h
NAME      READY     STATUS        RESTARTS   AGE
zk-0      1/1       Terminating   2          2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Pending   0         0s
zk-0      0/1       Pending   0         0s
zk-0      0/1       ContainerCreating   0         0s
zk-0      0/1       Running   0         51s
zk-0      1/1       Running   0         1m
zk-1      1/1       Terminating   0         2h
zk-1      0/1       Terminating   0         2h
zk-1      0/1       Terminating   0         2h
zk-1      0/1       Terminating   0         2h
zk-1      0/1       Pending   0         0s
zk-1      0/1       Pending   0         0s
```

مشاهده پادهای `StatefulSet` را ادامه دهید و نودی که `zk-2` بر روی آن برنامه‌ریزی شده است را تخلیه کنید.


```shell
kubectl drain $(kubectl get pod zk-2 --template {{.spec.nodeName}}) --ignore-daemonsets --force --delete-emptydir-data
```

خروجی مشابه این خواهد بود:

```
node "kubernetes-node-i4c4" cordoned

WARNING: Deleting pods not managed by ReplicationController, ReplicaSet, Job, or DaemonSet: fluentd-cloud-logging-kubernetes-node-i4c4, kube-proxy-kubernetes-node-i4c4; Ignoring DaemonSet-managed pods: node-problem-detector-v0.1-dyrog
WARNING: Ignoring DaemonSet-managed pods: node-problem-detector-v0.1-dyrog; Deleting pods not managed by ReplicationController, ReplicaSet, Job, or DaemonSet: fluentd-cloud-logging-kubernetes-node-i4c4, kube-proxy-kubernetes-node-i4c4
There are pending pods when an error occurred: Cannot evict pod as it would violate the pod's disruption budget.
pod/zk-2
```

از `CTRL-C` برای خاتمه دادن به kubectl استفاده کنید.

شما نمی‌توانید نود سوم را تخلیه کنید زیرا خروج کردن `zk-2` ممکن است به بودجه اختلال `zk` تخلل وارد کند. با این حال، نود قرنطینه خواهد ماند.

از `zkCli.sh` برای بازیابی مقداری که در آزمون سلامتی از `zk-0` وارد کرده‌اید، استفاده کنید.

```shell
kubectl exec zk-0 zkCli.sh get /hello
```

سرویس هنوز در دسترس است زیرا بودجه اختلال پاد آن رعایت می‌شود.

```
WatchedEvent state:SyncConnected type:None path:null
world
cZxid = 0x200000002
ctime = Wed Dec 07 00:08:59 UTC 2016
mZxid = 0x200000002
mtime = Wed Dec 07 00:08:59 UTC 2016
pZxid = 0x200000002
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
```

از [`kubectl uncordon`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands/#uncordon) برای رفع قرنطینه نود اول استفاده کنید.

```shell
kubectl uncordon kubernetes-node-pb41
```

خروجی مشابه این خواهد بود:

```
node "kubernetes-node-pb41" uncordoned
```

`zk-1` بر روی این نود مجدداً برنامه‌ریزی می‌شود. منتظر بمانید تا `zk-1` در وضعیت Running و Ready باشد.

```shell
kubectl get pods -w -l app=zk
```

خروجی مشابه این خواهد بود:

```
NAME      READY     STATUS    RESTARTS   AGE
zk-0      1/1       Running   2          1h
zk-1      1/1       Running   0          1h
zk-2      1/1       Running   0          1h
NAME      READY     STATUS        RESTARTS   AGE
zk-0      1/1       Terminating   2          2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Pending   0         0s
zk-0      0/1       Pending   0         0s
zk-0      0/1       ContainerCreating   0         0s
zk-0      0/1       Running   0         51s
zk-0      1/1       Running   0         1m
zk-1      1/1       Terminating   0         2h
zk-1      0/1       Terminating   0         2h
zk-1      0/1       Terminating   0         2h
zk-1      0/1       Terminating   0         2h
zk-1      0/1       Pending   0         0s
zk-1      0/1       Pending   0         0s
zk-1      0/1       Pending   0         12m
zk-1      0/1       ContainerCreating   0         12m
zk-1      0/1       Running   0         13m
zk-1      1/1       Running   0         13m
```

سعی کنید نودی را تخلیه کنید که `zk-2` بر روی آن برنامه‌ریزی شده است.

```shell
kubectl drain $(kubectl get pod zk-2 --template {{.spec.nodeName}}) --ignore-daemonsets --force --delete-emptydir-data
```

خروجی مشابه این خواهد بود:

```
node "kubernetes-node-i4c4" already cordoned
WARNING: Deleting pods not managed by ReplicationController, ReplicaSet, Job, or DaemonSet: fluentd-cloud-logging-kubernetes-node-i4c4, kube-proxy-kubernetes-node-i4c4; Ignoring DaemonSet-managed pods: node-problem-detector-v0.1-dyrog
pod "heapster-v1.2.0-2604621511-wht1r" deleted
pod "zk-2" deleted
node "kubernetes-node-i4c4" drained
```

این بار `kubectl drain` موفق می‌شود.

نود دوم را برای اجازه به برنامه‌ریزی `zk-2` رفع قرنطینه کنید.

```shell
kubectl uncordon kubernetes-node-ixsl
```

خروجی مشابه این خواهد بود:

```
node "kubernetes-node-ixsl" uncordoned
```

می‌توانید از `kubectl drain` به همراه `PodDisruptionBudgets` استفاده کنید تا اطمینان حاصل کنید که خدمات شما در طول نگهداری در دسترس خواهند بود.
اگر `drain` برای قرنطینه کردن نودها و برنامه‌ریزی مجدد پادها قبل از خاموش کردن نود برای نگهداری استفاده شود،
خدماتی که بودجه اختلال را بیان می‌کنند، این بودجه را رعایت خواهند کرد.
همیشه باید ظرفیت اضافی برای خدمات اساسی اختصاص دهید تا پادهای آنها بتوانند به سرعت مجدداً برنامه‌ریزی شوند.

## {{% heading "cleanup" %}}

- از `kubectl uncordon` برای برداشتن قرنطینه از تمام نودهای خوشه خود استفاده کنید.
- باید رسانه‌های ذخیره‌سازی دائمی برای PersistentVolumes استفاده شده در این آموزش را حذف کنید.
  مراحل لازم را بر اساس محیط، پیکربندی ذخیره‌سازی و روش ارائه، برای اطمینان از بازیابی کامل ذخیره‌سازی انجام دهید.