```markdown
reviewers:
- verb
- soltysh
title: اشکال‌زدایی پادهای در حال اجرا
content_type: task
---

<!-- مرور -->

این صفحه شرح می‌دهد که چگونه می‌توان پادهای در حال اجرا (یا در حال خرابی) روی یک Node را اشکال‌زدایی کرد.


## {{% heading "prerequisites" %}}


* {{< glossary_tooltip text="Pod" term_id="pod" >}} شما باید قبلاً زمان‌بندی شده و در حال اجرا باشد. اگر Pod شما هنوز اجرا نشده است، از [اشکال‌زدایی پادها](/docs/tasks/debug/debug-application/) شروع کنید.
* برای برخی از مراحل اشکال‌زدایی پیشرفته، نیاز به دانستن این دارید که پاد شما روی کدام Node اجرا می‌شود و دسترسی به shell برای اجرای دستورات در آن Node دارید. شما نیازی به دسترسی به این دستورات برای اجرای مراحل اشکال‌زدایی استاندارد با استفاده از `kubectl` ندارید.

## استفاده از `kubectl describe pod` برای دریافت جزئیات درباره پادها

برای این مثال، ما از یک Deployment برای ایجاد دو پاد استفاده می‌کنیم، مشابه مثال‌های قبلی.

{{% code_sample file="application/nginx-with-request.yaml" %}}

ایجاد Deployment را با اجرای دستور زیر انجام دهید:

```shell
kubectl apply -f https://k8s.io/examples/application/nginx-with-request.yaml
```

```none
deployment.apps/nginx-deployment created
```

وضعیت پاد را با دستور زیر بررسی کنید:

```shell
kubectl get pods
```

```none
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-67d4bdd6f5-cx2nz   1/1     Running   0          13s
nginx-deployment-67d4bdd6f5-w6kd7   1/1     Running   0          13s
```

می‌توانیم با استفاده از `kubectl describe pod` اطلاعات بیشتری راجع به هر یک از این پادها دریافت کنیم. به عنوان مثال:

```shell
kubectl describe pod nginx-deployment-67d4bdd6f5-w6kd7
```

```none
Name:         nginx-deployment-67d4bdd6f5-w6kd7
Namespace:    default
Priority:     0
Node:         kube-worker-1/192.168.0.113
Start Time:   Thu, 17 Feb 2022 16:51:01 -0500
Labels:       app=nginx
              pod-template-hash=67d4bdd6f5
Annotations:  <none>
Status:       Running
IP:           10.88.0.3
IPs:
  IP:           10.88.0.3
  IP:           2001:db8::1
Controlled By:  ReplicaSet/nginx-deployment-67d4bdd6f5
Containers:
  nginx:
    Container ID:   containerd://5403af59a2b46ee5a23fb0ae4b1e077f7ca5c5fb7af16e1ab21c00e0e616462a
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:2834dc507516af02784808c5f48b7cbe38b8ed5d0f4837f16e78d00deb7e7767
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 17 Feb 2022 16:51:05 -0500
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  128Mi
    Requests:
      cpu:        500m
      memory:     128Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-bgsgp (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-bgsgp:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Guaranteed
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  34s   default-scheduler  Successfully assigned default/nginx-deployment-67d4bdd6f5-w6kd7 to kube-worker-1
  Normal  Pulling    31s   kubelet            Pulling image "nginx"
  Normal  Pulled     30s   kubelet            Successfully pulled image "nginx" in 1.146417389s
  Normal  Created    30s   kubelet            Created container nginx
  Normal  Started    30s   kubelet            Started container nginx
```

در اینجا می‌توانید اطلاعات پیکربندی درباره کانتینر(ها) و پاد (برچسب‌ها، نیازهای منابع و غیره) را مشاهده کنید، همچنین اطلاعات وضعیت درباره کانتینر(ها) و پاد (وضعیت، آمادگی، تعداد بازنشانی، رویدادها و غیره) را مشاهده کنید.

وضعیت کانتینر یکی از وضعیت‌های Waiting، Running یا Terminated است. بسته به وضعیت، اطلاعات اضافی ارائه می‌شود - در اینجا می‌توانید ببینید که برای یک کانتینر در وضعیت Running، سیستم به شما اطلاع می‌دهد که کانتینر کی شروع شده است.

آماده شما اطلاع می‌دهد که آیا کانتینر پاسخگوی آخرین امتحان آمادگی خود را گذرانده است یا نه. (در این مورد، اگر آزمون آمادگی برای کانتینر پیکربندی نشود، فرض می‌شود که کانتینر آماده است اگر آزمون آمادگی پیکربندی نشده باشد.)

تعداد بازنشانی به شما می‌گوید چند بار کانتینر دوباره راه‌اندازی شده است؛ این اطلاعات می‌تواند برای شناسایی حلقه‌های خرابی در کانتینرهایی که با سیاست بازنشانی "همیشه" پیکربندی شده‌اند، مفید باشد.

در حال حاض

ر تنها شرایط مرتبط با یک پاد شرط آمادگی دودویی است، که نشان می‌دهد که پاد قادر به سرویس‌دهی درخواست‌ها است و باید به استخرهای توازن بار تمام خدمات مطابقت دهد.

در نهایت، شما لاگ از رویدادهای اخیر مرتبط با پاد خود را مشاهده می‌کنید. "از" نشان می‌دهد که کدام جزء دارد رویداد را لاگ می‌کند. "دلیل" و "پیام" به شما می‌گوید چه اتفاقی افتاده است.

## مثال: اشکال‌زدایی پادهای در انتظار

یک سناریو رایج که می‌توانید با استفاده از رویدادها شناسایی کنید، زمانی است که یک پاد ایجاد کرده‌اید که نمی‌تواند بر روی هیچ Nodeی جا شود. به عنوان مثال، ممکن است پاد بیشتری منابع را درخواست کند که روی هیچ Nodeی آزاد نیست یا ممکن است یک انتخاب‌گر برچسب مشخص کنید که با هیچ Nodeی مطابقت ندارد. بگذارید بگوییم که ما Deployment قبلی را با 5 نمونه (به جای 2) ایجاد کرده‌ایم و درخواست 600 میلی‌کور همچنین کرده‌ایم، در یک خوشه چهار Nodeی که هر (ماشین مجازی) دارای 1 CPU است. در این صورت، یکی از پادها قادر به زمان‌بندی نمی‌شود. (توجه داشته باشید که به دلیل پودهای افزونه خوشه مانند fluentd، skydns، و غیره که روی هر Node اجرا می‌شوند، اگر ما 1000 میلی‌کور درخواست کنیم، هیچکدام از پادها قادر به زمان‌بندی نمی‌شوند.)

```shell
kubectl get pods
```

```none
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-1006230814-6winp   1/1       Running   0          7m
nginx-deployment-1006230814-fmgu3   1/1       Running   0          7m
nginx-deployment-1370807587-6ekbw   1/1       Running   0          1m
nginx-deployment-1370807587-fg172   0/1       Pending   0          1m
nginx-deployment-1370807587-fz9sd   0/1       Pending   0          1m
```

برای بررسی علت اینکه پاد nginx-deployment-1370807587-fz9sd در حالت Pending است، می‌توانیم از `kubectl describe pod` روی پاد در انتظار استفاده کنیم و رویدادهای آن را بررسی کنیم:

```shell
kubectl describe pod nginx-deployment-1370807587-fz9sd
```

```none
  Name:		nginx-deployment-1370807587-fz9sd
  Namespace:	default
  Node:		/
  Labels:		app=nginx,pod-template-hash=1370807587
  Status:		Pending
  IP:
  Controllers:	ReplicaSet/nginx-deployment-1370807587
  Containers:
    nginx:
      Image:	nginx
      Port:	80/TCP
      QoS Tier:
        memory:	Guaranteed
        cpu:	Guaranteed
      Limits:
        cpu:	1
        memory:	128Mi
      Requests:
        cpu:	1
        memory:	128Mi
      Environment Variables:
  Volumes:
    default-token-4bcbi:
      Type:	Secret (a volume populated by a Secret)
      SecretName:	default-token-4bcbi
  Events:
    FirstSeen	LastSeen	Count	From			        SubobjectPath	Type		Reason			    Message
    ---------	--------	-----	----			        -------------	--------	------			    -------
    1m		    48s		    7	    {default-scheduler }			        Warning		FailedScheduling	pod (nginx-deployment-1370807587-fz9sd) failed to fit in any node
  fit failure on node (kubernetes-node-6ta5): Node didn't have enough resource: CPU, requested: 1000, used: 1420, capacity: 2000
  fit failure on node (kubernetes-node-wul5): Node didn't have enough resource: CPU, requested: 1000, used: 1100, capacity: 2000
```

در اینجا می‌توانید رویدادی که توسط برنامه‌زمان‌دهنده تولید شده است که نشان می‌دهد پاد نمی‌تواند بر روی هیچ Node‌ای زمان‌بندی شود به دلیل `FailedScheduling` (و شاید دیگر دلایل) را مشاهده کنید. پیام به ما می‌گوید که بر روی هیچ Node‌ای منابع کافی برای پاد وجود ندارد.

برای اصلاح این موقعیت، می‌توانید از `kubectl scale` برای به‌روزرسانی Deployment خود برای مشخص کردن چهار نمونه یا کمتر استفاده کنید. (یا می‌توانید یک پاد را در حالت Pending بگذارید که ضرر ندارد.)

رویدادهایی مانند آن‌هایی که در انتهای `kubectl describe pod` دیدید، در etcd ذخیره می‌شوند و اطلاعات سطح بالایی را در مورد چه اتفاقی در خوشه رخ می‌دهد، ارائه می‌دهند. برای لیست کردن تمام رویدادها می‌توانید از

```shell
kubectl get events
```

استفاده کنید، اما باید به یاد داشته باشید که رویدادها نیم‌فضایی هستند. این بدان معناست که اگر به رویدادهایی برای برخی از اشیاء نیم‌فضایی (مانند چه اتفاقی با پادها در فضای نام `my-namespace` رخ داده است) علاقه دارید، باید به صراحت نام فضای نام را به دستور بدهید:

```shell
kubectl get events --namespace=my-namespace
```

برای دیدن رویدادها از تمام فضاهای نام می‌توانید از آرگومان `--all-namespaces` استفاده کنید.

به علاوه به `kubectl describe pod`، راه دیگری برای گرفتن اطلاعات اضافی درباره یک پاد (فراتر از آنچه که `kubectl get pod` ارائه می‌دهد)، استفاده از پرچم فرمت خروجی `-o yaml` به `kubectl get pod` است. این به شما در قالب YAML، اطلاعات بیشتری از `kubectl describe pod` می‌دهد - اساساً تمام اطلاعاتی که سیستم در مورد پاد دارد را مشاهده می‌کنید. در اینجا چیزهایی مانند حاشیه نوشته‌ها (که اطلاعات کلیدی-مقدار بدون محدودیت برچسب را که توس

ط جزئیات نشان داده شده‌اند که توسط اجزاء سیستم Kubernetes به طور داخلی استفاده می‌شود)، سیاست دوباره‌شروعی، پورت‌ها و حجم‌ها را خواهید دید.

```shell
kubectl get pod nginx-deployment-1006230814-6winp -o yaml
```


```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2022-02-17T21:51:01Z"
  generateName: nginx-deployment-67d4bdd6f5-
  labels:
    app: nginx
    pod-template-hash: 67d4bdd6f5
  name: nginx-deployment-67d4bdd6f5-w6kd7
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: nginx-deployment-67d4bdd6f5
    uid: 7d41dfd4-84c0-4be4-88ab-cedbe626ad82
  resourceVersion: "1364"
  uid: a6501da1-0447-4262-98eb-c03d4002222e
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: nginx
    ports:
    - containerPort: 80
      protocol: TCP
    resources:
      limits:
        cpu: 500m
        memory: 128Mi
      requests:
        cpu: 500m
        memory: 128Mi
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-bgsgp
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: kube-worker-1
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-bgsgp
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2022-02-17T21:51:01Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2022-02-17T21:51:06Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2022-02-17T21:51:06Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2022-02-17T21:51:01Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://5403af59a2b46ee5a23fb0ae4b1e077f7ca5c5fb7af16e1ab21c00e0e616462a
    image: docker.io/library/nginx:latest
    imageID: docker.io/library/nginx@sha256:2834dc507516af02784808c5f48b7cbe38b8ed5d0f4837f16e78d00deb7e7767
    lastState: {}
    name: nginx
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2022-02-17T21:51:05Z"
  hostIP: 192.168.0.113
  phase: Running
  podIP: 10.88.0.3
  podIPs:
  - ip: 10.88.0.3
  - ip: 2001:db8::1
  qosClass: Guaranteed
  startTime: "2022-02-17T21:51:01Z"
```

## بررسی لاگ‌های پاد {#examine-pod-logs}

ابتدا به لاگ‌های کانتینر مورد نظر نگاهی بیندازید:

```shell
kubectl logs ${POD_NAME} ${CONTAINER_NAME}
```

اگر کانتینر شما قبلاً خراب شده است، می‌توانید به لاگ‌های قبلی خرابی کانتینر دسترسی پیدا کنید با:

```shell
kubectl logs --previous ${POD_NAME} ${CONTAINER_NAME}
```

## اشکال‌زدایی با اجرای دستورات در کانتینر {#container-exec}

اگر {{< glossary_tooltip text="تصویر کانتینر" term_id="image" >}} شامل ابزارهای اشکال‌زدایی باشد، همانطور که در تصاویر ساخته شده از تصاویر پایه سیستم‌عامل لینوکس و ویندوز ممکن است، شما می‌توانید دستورات را درون یک کانتینر خاص اجرا کنید با استفاده از `kubectl exec`:

```shell
kubectl exec ${POD_NAME} -c ${CONTAINER_NAME} -- ${CMD} ${ARG1} ${ARG2} ... ${ARGN}
```

{{< note >}}
`-c ${CONTAINER_NAME}` اختیاری است. می‌توانید آن را برای پادهایی که فقط شامل یک کانتینر هستند، حذف کنید.
{{< /note >}}

به عنوان مثال، برای مشاهده لاگ‌ها از یک پاد Cassandra در حال اجرا، ممکن است این دستور را اجرا کنید:

```shell
kubectl exec cassandra -- cat /var/log/cassandra/system.log
```

می‌توانید یک شل که به ترمینال شما متصل است، با استفاده از آرگومان‌های `-i` و `-t` به `kubectl exec` اجرا کنید، به عنوان مثال:

```shell
kubectl exec -it cassandra -- sh
```

برای جزئیات بیشتر، [Get a Shell to a Running Container](
/docs/tasks/debug/debug-application/get-shell-running-container/) را مشاهده کنید.

## اشکال‌زدایی با استفاده از کانتینر اشکال‌زدایی موقت {#ephemeral-container}

{{< feature-state state="stable" for_k8s_version="v1.25" >}}

{{< glossary_tooltip text="کانتینرهای اشکال‌زدایی موقت" term_id="ephemeral-container" >}}
مفید برای رفع اشکال تعاملی هستند زمانی که `kubectl exec` کافی نیست چرا که یک کانتینر خراب شده است یا یک تصویر کانتینر شامل ابزارهای اشکال‌زدایی نیست، مانند [تصاویر distroless](
https://github.com/GoogleContainerTools/distroless).

### مثال اشکال‌زدایی با استفاده از کانتینرهای اشکال‌زدایی موقت {#ephemeral-container-example}

می‌توانید از دستور `kubectl debug` برای اضافه کردن کانتینرهای اشکال‌زدایی موقت به یک پاد در حال اجرا استفاده کنید. ابتدا، یک پاد برای مثال ایجاد کنید:

```shell
kubectl run ephemeral-demo --image=registry.k8s.io/pause:3.1 --restart=Never
```

مثال‌های این بخش از تصویر کانتینر `pause` استفاده می‌کند زیرا این تصویر شامل ابزارهای اشکال‌زدایی نیست، اما این روش با تمام تصاویر کانتینر کار می‌کند.

اگر سعی کنید با `kubectl exec` یک شل ایجاد کنید، با خطای زیر مواجه می‌شوید زیرا در این تصویر کانتینر شل وجود ندارد.

```shell
kubectl exec -it ephemeral-demo -- sh
```

```
OCI runtime exec failed: exec failed: container_linux.go:346: starting container process caused "exec: \"sh\": executable file not found in $PATH": unknown
```

می‌توانید به جای آن از `kubectl debug` برای اضافه کردن یک کانتینر اشکال‌زدایی استفاده کنید. اگر پارامتر `-i`/`--interactive` را مشخص کنید، `kubectl` به طور خودکار به کنسول کانتینر اشکال‌زدایی موقت متصل می‌شود.

```shell
kubectl debug -it ephemeral-demo --image=busybox:1.28 --target=ephemeral-demo
```

```
Defaulting debug container name to debugger-8xzrl.
If you don't see a command prompt, try pressing enter.
/ #
```

این دستور یک کانتینر جدید busybox اضافه می‌کند و به آن متصل می‌شود. پارامتر `--target` به فضای فرآیند یک کانتینر دیگر هدف می‌شود. این در اینجا ضروری است زیرا `kubectl run` فضای فرآیند را در پادی که ایجاد می‌کند فعال نمی‌کند.

{{< note >}}
پارامتر `--target` باید توسط {{< glossary_tooltip
text="موتور اجرای کانتینر" term_id="container-runtime" >}} پشتیبانی شود. هنگامی که پشتیبانی نمی‌شود، کانتینر اشکال‌زدایی موقت شروع نمی‌شود، یا ممکن است با فضای فرآیند منزوی شروع شود به طوری که `ps` فرآیندهای در کانتینرهای دیگر را نشان نمی‌دهد.
{{< /note >}}

برای مشاهده وضعیت کانتینر اشکال‌زدایی موقت جدید از `kubectl describe` استفاده کنید:

```shell
kubectl describe pod ephemeral-demo
```

```
...
Ephemeral Containers:
  debugger-8xzrl:
    Container ID:   docker://b888f9adfd15bd5739fefaa39e1df4dd3c617b9902082b1cfdc29c4028ffb2eb
    Image:          busybox
    Image ID:       docker-pullable://busybox@sha256:1828edd60c5efd34b2bf5dd3282ec0cc04d47b2ff9caa0b6d4f07a21d1c08084
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 12 Feb 2020 14:25:42 +0100
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:         <none>
...


```

هنگامی که کار خود را به پایان رساندید، از `kubectl delete` برای حذف پاد استفاده کنید:

```shell
kubectl delete pod ephemeral-demo
```

## اشکال‌زدایی با استفاده از یک کپی از پاد

گاهی اوقات گزینه‌های پیکربندی پاد، در برخی موقعیت‌ها سختی اشکال‌زدایی را افزایش می‌دهند. به عنوان مثال، شما نمی‌توانید از `kubectl exec` برای اشکال‌زدایی کانتینر خود استفاده کنید اگر تصویر کانتینر شما شامل یک شل نباشد یا اگر برنامه‌تان در هنگام راه‌اندازی خراب شود. در این موارد می‌توانید از `kubectl debug` استفاده کنید تا یک کپی از پاد با تغییر مقادیر پیکربندی برای کمک به اشکال‌زدایی ایجاد کنید.

### ایجاد یک کپی از پاد با اضافه کردن یک کانتینر جدید

اضافه کردن یک کانتینر جدید مفید است زمانی که برنامه‌ی شما در حال اجرا است اما با نحوه عملکرد خود راضی نیستید و می‌خواهید ابزارهای اشکال‌زدایی اضافی را به پاد اضافه کنید.

به عنوان مثال، شاید تصاویر کانتینر برنامه‌ی شما بر روی `busybox` ساخته شده باشد اما شما نیاز به ابزارهای اشکال‌زدایی ندارید که در `busybox` وجود ندارند. می‌توانید این سناریو را با استفاده از `kubectl run` شبیه‌سازی کنید:

```shell
kubectl run myapp --image=busybox:1.28 --restart=Never -- sleep 1d
```

برای ایجاد یک کپی از `myapp` به نام `myapp-debug` که یک کانتینر جدید اوبونتو برای اشکال‌زدایی اضافه می‌کند، این دستور را اجرا کنید:

```shell
kubectl debug myapp -it --image=ubuntu --share-processes --copy-to=myapp-debug
```

```
Defaulting debug container name to debugger-w7xmf.
If you don't see a command prompt, try pressing enter.
root@myapp-debug:/#
```

{{< note >}}
* `kubectl debug` به طور خودکار یک نام کانتینر تولید می‌کند اگر شما با استفاده از پرچم `--container` یکی را انتخاب نکنید.
* پرچم `-i` باعث می‌شود که `kubectl debug` به طور خودکار به کانتینر جدید متصل شود. شما می‌توانید این را با مشخص کردن `--attach=false` جلوگیری کنید. اگر جلسه شما قطع شود، می‌توانید با استفاده از `kubectl attach` دوباره وصل شوید.
* پرچم `--share-processes` به کانتینرها این امکان را می‌دهد که فرآیندهای موجود در دیگر کانتینرهای پاد را ببینند. برای اطلاعات بیشتر در مورد نحوه عملکرد آن، به [Share Process Namespace between Containers in a Pod](
/docs/tasks/configure-pod-container/share-process-namespace/) مراجعه کنید.
{{< /note >}}

حتماً هنگام پایان کار با آن، پاد اشکال‌زدایی را پاک کنید:

```shell
kubectl delete pod myapp myapp-debug
```

### ایجاد یک کپی از پاد با تغییر دستور آن

گاهی اوقات مفید است دستور یک کانتینر را تغییر داد، به عنوان مثال برای اضافه کردن یک پرچم اشکال‌زدایی یا زمانی که برنامه‌ی شما خراب می‌شود.

برای شبیه‌سازی یک برنامه که در حالت CrashLoopBackOff است، از `kubectl run` برای ایجاد یک کانتینر استفاده کنید که به طور فوری خارج شود:

```shell
kubectl run --image=busybox:1.28 myapp -- false
```

می‌توانید با استفاده از `kubectl describe pod myapp` ببینید که این کانتینر در حالت خرابی است:

```
Containers:
  myapp:
    Image:         busybox
    ...
    Args:
      false
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
```

می‌توانید از `kubectl debug` استفاده کنید تا یک کپی از این پاد با دستور تغییر یافته به یک شل تعاملی ایجاد کنید:

```
kubectl debug myapp -it --copy-to=myapp-debug --container=myapp -- sh
```

```
If you don't see a command prompt, try pressing enter.
/ #
```

حالا یک شل تعاملی دارید که می‌توانید از آن برای انجام وظایفی مانند بررسی مسیرهای فایل سیستم یا اجرای دستور کانتینر به صورت دستی استفاده کنید.

{{< note >}}
* برای تغییر دستور یک کانتینر خاص، شما باید نام آن را با استفاده از `--container` مشخص کنید، در غیر این صورت `kubectl debug` به جای آن یک کانتینر جدید برای اجرای دستور مشخص شده ایجاد می‌کند.
* پرچم `-i` باعث می‌شود که `kubectl debug` به طور خودکار به کانتینر متصل شود. می‌توانید این را با مشخص کردن `--attach=false` جلوگیری کنید. اگر جلسه شما قطع شود، می‌توانید با استفاده از `kubectl attach` دوباره وصل شوید.
{{< /note >}}

حتماً هنگام پایان کار با آن، پاد اشکال‌زدایی را پاک کنید:

```shell
kubectl delete pod myapp myapp-debug
```

### ایجاد یک کپی از پاد با تغییر تصاویر کانتینر

در برخی موارد ممکن است که نیاز داشته باشید که یک پاد

 ناخوش‌کار را از تصاویر استاندارد تولیدی به تصویر حاوی ساخت اشکال‌زدایی یا ابزارهای اضافی تغییر دهید.

به عنوان مثال، یک پاد با استفاده از `kubectl run` ایجاد کنید:

```shell
kubectl run myapp --image=busybox:1.28 --restart=Never -- sleep 1d
```

حالا از `kubectl debug` برای ایجاد یک کپی و تغییر تصویر کانتینر آن به `ubuntu` استفاده کنید:

```shell
kubectl debug myapp --copy-to=myapp-debug --set-image=*=ubuntu
```

سینتکس `--set-image` همان سینتکس `container_name=image` استفاده شده در `kubectl set image` است. `*=ubuntu` بدان معناست که تصویر تمام کانتینرها را به `ubuntu` تغییر دهید.

حتماً هنگام پایان کار با آن، پاد اشکال‌زدایی را پاک کنید:

```shell
kubectl delete pod myapp myapp-debug
```

## اشکال‌زدایی از طریق شل بر روی نود {#node-shell-session}

اگر هیچکدام از این روش‌ها کار نکرد، می‌توانید نودی که پاد در آن در حال اجرا است را پیدا کنید و یک پاد بر روی نود اجرا کنید. برای ایجاد یک جلسه شل تعاملی بر روی یک نود با استفاده از `kubectl debug`، این دستور را اجرا کنید:

```shell
kubectl debug node/mynode -it --image=ubuntu
```

```
Creating debugging pod node-debugger-mynode-pdx84 with container debugger on node mynode.
If you don't see a command prompt, try pressing enter.
root@ek8s:/#
```

هنگام ایجاد جلسه اشکال‌زدایی بر روی یک نود، به خاطر داشته باشید که:

* `kubectl debug` به طور خودکار نام جدید پاد را بر اساس نام نود تولید می‌کند.
* فایل سیستم ریشه نود به `/host` متصل خواهد شد.
* کانتینر در فضای نام‌های IPC، شبکه و PID میزبان اجرا می‌شود، اگرچه پاد دسترسی افزوده ندارد، بنابراین خواندن برخی اطلاعات فرآیند ممکن است شکست بخورد و `chroot /host` ممکن است شکست بخورد.
* اگر نیاز به پادی با دسترسی افزوده دارید، آن را به صورت دستی ایجاد کنید.

حتماً هنگام پایان کار با آن، پاد اشکال‌زدایی را پاک کنید:

```shell
kubectl delete pod node-debugger-mynode-pdx84
```
