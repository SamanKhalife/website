```markdown
---
title: به‌روزرسانی پیکربندی از طریق ConfigMap
content_type: آموزش
weight: 20
---

<!-- overview -->
این صفحه یک مثال گام به گام از به‌روزرسانی پیکربندی در یک Pod از طریق ConfigMap را ارائه می‌دهد و بر پایه وظیفه [پیکربندی یک Pod برای استفاده از ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/) ساخته شده است. در انتهای این آموزش، شما قادر خواهید بود تا چگونگی تغییر پیکربندی برای یک برنامه در حال اجرا را درک کنید. این آموزش از تصاویر `alpine` و `nginx` به عنوان مثال استفاده می‌کند.

## {{% heading "پیش‌نیازها" %}}
{{< include "task-tutorial-prereqs.md" >}}

شما باید ابزار خط فرمان [curl](https://curl.se/) را برای ارسال درخواست‌های HTTP از ترمینال یا پنجره دستوری داشته باشید. اگر `curl` در دسترس نیست، می‌توانید آن را نصب کنید. برای جزئیات بیشتر مستندات سیستم عامل محلی خود را بررسی کنید.

## {{% heading "اهداف" %}}
* به‌روزرسانی پیکربندی از طریق یک ConfigMap که به عنوان یک Volume مونت شده است
* به‌روزرسانی متغیرهای محیطی یک Pod از طریق یک ConfigMap
* به‌روزرسانی پیکربندی از طریق یک ConfigMap در یک Pod چند-کانتینری
* به‌روزرسانی پیکربندی از طریق یک ConfigMap در یک Pod دارای کانتینر Sidecar

<!-- lessoncontent -->

## به‌روزرسانی پیکربندی از طریق یک ConfigMap که به عنوان یک Volume مونت شده است {#rollout-configmap-volume}

از دستور `kubectl create configmap` برای ساختن یک ConfigMap از [مقادیر لیترال](/docs/tasks/configure-pod-container/configure-pod-configmap/#create-configmaps-from-literal-values) استفاده کنید:

```shell
kubectl create configmap sport --from-literal=sport=football
```

در زیر، یک نمونه از یک Deployment manifest با ConfigMap `sport` به عنوان یک {{< glossary_tooltip text="volume" term_id="volume" >}} به کانتینر واحد Pod مونت شده است.
{{% code_sample file="deployments/deployment-with-configmap-as-volume.yaml" %}}

ساختن Deployment:
```shell
kubectl apply -f https://k8s.io/examples/deployments/deployment-with-configmap-as-volume.yaml
```

برای اطمینان از آمادگی Pod‌های این Deployment (براساس
{{< glossary_tooltip text="selector" term_id="selector" >}}):
```shell
kubectl get pods --selector=app.kubernetes.io/name=configmap-volume
```

باید خروجی مشابه زیر را ببینید:
```
NAME                                READY   STATUS    RESTARTS   AGE
configmap-volume-6b976dfdcf-qxvbm   1/1     Running   0          72s
configmap-volume-6b976dfdcf-skpvm   1/1     Running   0          72s
configmap-volume-6b976dfdcf-tbc6r   1/1     Running   0          72s
```

در هر یک از نودهایی که یکی از این Podها در حال اجرا است، kubelet داده‌های ConfigMap را برای آن بازیابی می‌کند و آن‌ها را به فایل‌هایی در یک Volume محلی ترجمه می‌کند. سپس kubelet آن Volume را به کانتینر مونت می‌کند، همانطور که در الگوی Pod مشخص شده است. کدی که در این کانتینر اجرا می‌شود، اطلاعات را از فایل بارگذاری می‌کند و آن را برای چاپ گزارش به stdout استفاده می‌کند. شما می‌توانید این گزارش را با مشاهده لاگ‌های یکی از Podهای این Deployment بررسی کنید:

```shell
# یک Pod را که به Deployment تعلق دارد انتخاب کنید و لاگ‌های آن را مشاهده کنید
kubectl logs deployments/configmap-volume
```

باید خروجی مشابه زیر را ببینید:
```
Thu Jan  4 14:06:46 UTC 2024 My preferred sport is football
Thu Jan  4 14:06:56 UTC 2024 My preferred sport is football
Thu Jan  4 14:07:06 UTC 2024 My preferred sport is football
Thu Jan  4 14:07:16 UTC 2024 My preferred sport is football
Thu Jan  4 14:07:26 UTC 2024 My preferred sport is football
```

ویرایش ConfigMap:
```shell
kubectl edit configmap sport
```

در ویرایشگری که ظاهر می‌شود، مقدار کلید `sport` را از `football` به `cricket` تغییر دهید و تغییرات خود را ذخیره کنید. ابزار kubectl مطابق با این تغییرات ConfigMap را به‌روز می‌کند (در صورت دیدن خطایی، دوباره امتحان کنید).

در ادامه نمونه‌ای از اینکه چگونه این منظره می‌تواند بعد از ویرایش به نمایش درآید:

```yaml
apiVersion: v1
data:
  sport: cricket
kind: ConfigMap
# شما می‌توانید اطلاعات موجودیت‌ها را به عنوان یکی از آن‌ها نگذارید.
# مقادیر که شما باید مشابه نباشد.
metadata:
  creationTimestamp: "2024-01-04T14:05:06Z"
  name: sport
  namespace: default
  resourceVersion: "1743935"
  uid: 024ee001-fe72-487e-872e-34d6464a8a23
```

باید خروجی مشابه زیر را ببینید:
```
configmap/sport edited
```

دنبال کردن لاگ‌های یکی از Podهایی که به این Deployment تعلق دارد:

```shell
kubectl logs deployments/configmap-volume --follow
```

پس از چن

د ثانیه، باید تغییرات خروجی لاگ‌ها به صورت زیر باشد:
```
Thu Jan  4 14:11:36 UTC 2024 My preferred sport is football
Thu Jan  4 14:11:46 UTC 2024 My preferred sport is football
Thu Jan  4 14:11:56 UTC 2024 My preferred sport is football
Thu Jan  4 14:12:06 UTC 2024 My preferred sport is cricket
Thu Jan  4 14:12:16 UTC 2024 My preferred sport is cricket
```

وقتی که یک ConfigMap که به‌روزرسانی شده است و به یک Pod در حال اجرا مپ شده است (با استفاده از یک volume `configMap` یا یک volume `projected`)، Pod در حال اجرا تغییر را تقریباً فوراً مشاهده می‌کند. با این حال، برنامه شما فقط در صورتی که برای مشاهده تغییرات نوشته شود یا تغییرات فایل را بررسی کند، تغییرات را مشاهده می‌کند. یک برنامه که پیکربندی خود را یکبار در زمان راه‌اندازی بارگذاری می‌کند، تغییری را که در ConfigMap اعمال می‌شود متوجه نمی‌شود.

{{< note >}}
تأخیر کلی از لحظه‌ای که ConfigMap به‌روزرسانی می‌شود تا زمانی که کلیدهای جدید به Pod پروژه‌نگاری شده انتقال داده می‌شوند، ممکن است تا مدت زمان همگام‌سازی kubelet باشد.
همچنین بررسی کنید [ConfigMaps مونت شده به طور خودکار به‌روزرسانی می‌شوند](/docs/tasks/configure-pod-container/configure-pod-configmap/#mounted-configmaps-are-updated-automatically).
{{< /note >}}

## به‌روزرسانی متغیرهای محیطی یک Pod از طریق یک ConfigMap {#rollout-configmap-env}
از دستور `kubectl create configmap` برای ساختن یک ConfigMap از [مقادیر لیترال](/docs/tasks/configure-pod-container/configure-pod-configmap/#create-configmaps-from-literal-values) استفاده کنید:

```shell
kubectl create configmap fruits --from-literal=fruits=apples
```

در زیر، یک نمونه از یک Deployment manifest با یک متغیر محیطی از طریق ConfigMap `fruits` تنظیم شده است.
{{% code_sample file="deployments/deployment-with-configmap-as-envvar.yaml" %}}

ساختن Deployment:
```shell
kubectl apply -f https://k8s.io/examples/deployments/deployment-with-configmap-as-envvar.yaml
```

برای اطمینان از آمادگی Pod‌های این Deployment (براساس
{{< glossary_tooltip text="selector" term_id="selector" >}}):
```shell
kubectl get pods --selector=app.kubernetes.io/name=configmap-env-var
```


You should see an output similar to:
```
NAME                                 READY   STATUS    RESTARTS   AGE
configmap-env-var-59cfc64f7d-74d7z   1/1     Running   0          46s
configmap-env-var-59cfc64f7d-c4wmj   1/1     Running   0          46s
configmap-env-var-59cfc64f7d-dpr98   1/1     Running   0          46s
```

The key-value pair in the ConfigMap is configured as an environment variable in the container of the Pod.
Check this by viewing the logs of one Pod that belongs to the Deployment.
```shell
kubectl logs deployment/configmap-env-var
```

You should see an output similar to:
```
Found 3 pods, using pod/configmap-env-var-7c994f7769-l74nq
Thu Jan  4 16:07:06 UTC 2024 The basket is full of apples
Thu Jan  4 16:07:16 UTC 2024 The basket is full of apples
Thu Jan  4 16:07:26 UTC 2024 The basket is full of apples
```

Edit the ConfigMap:
```shell
kubectl edit configmap fruits
```

In the editor that appears, change the value of key `fruits` from `apples` to `mangoes`. Save your changes.
The kubectl tool updates the ConfigMap accordingly (if you see an error, try again).

Here's an example of how that manifest could look after you edit it:
```yaml
apiVersion: v1
data:
  fruits: mangoes
kind: ConfigMap
# You can leave the existing metadata as they are.
# The values you'll see won't exactly match these.
metadata:
  creationTimestamp: "2024-01-04T16:04:19Z"
  name: fruits
  namespace: default
  resourceVersion: "1749472"

```

You should see the following output:
```
configmap/fruits edited
```

Tail the logs of the Deployment and observe the output for few seconds:
```shell
# As the text explains, the output does NOT change
kubectl logs deployments/configmap-env-var --follow
```

Notice that the output remains **unchanged**, even though you edited the ConfigMap:
```
Thu Jan  4 16:12:56 UTC 2024 The basket is full of apples
Thu Jan  4 16:13:06 UTC 2024 The basket is full of apples
Thu Jan  4 16:13:16 UTC 2024 The basket is full of apples
Thu Jan  4 16:13:26 UTC 2024 The basket is full of apples
```

{{< note >}}
Although the value of the key inside the ConfigMap has changed, the environment variable in the Pod still shows the earlier value. This is because environment variables for a process running inside a Pod are **not** updated when the source data changes; if you wanted to force an update, you would need to have Kubernetes replace your existing Pods. The new Pods would then run with the updated information.
{{< /note >}}

You can trigger that replacement. Perform a rollout for the Deployment, using
[`kubectl rollout`](/docs/reference/kubectl/generated/kubectl_rollout/):
```shell
# Trigger the rollout
kubectl rollout restart deployment configmap-env-var

# Wait for the rollout to complete
kubectl rollout status deployment configmap-env-var --watch=true
```

Next, check the Deployment:
```shell
kubectl get deployment configmap-env-var
```

You should see an output similar to:
```
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
configmap-env-var   3/3     3            3           12m
```

Check the Pods:
```shell
kubectl get pods --selector=app.kubernetes.io/name=configmap-env-var
```

The rollout causes Kubernetes to make a new {{< glossary_tooltip term_id="replica-set" text="ReplicaSet" >}} for the Deployment; that means the existing Pods eventually terminate, and new ones are created. After few seconds, you should see an output similar to:
```
NAME                                 READY   STATUS        RESTARTS   AGE
configmap-env-var-6d94d89bf5-2ph2l   1/1     Running       0          13s
configmap-env-var-6d94d89bf5-74twx   1/1     Running       0          8s
configmap-env-var-6d94d89bf5-d5vx8   1/1     Running       0          11s
```

{{< note >}}
Please wait for the older Pods to fully terminate before proceeding with the next steps.
{{< /note >}}

View the logs for a Pod in this Deployment:
```shell
# Pick one Pod that belongs to the Deployment, and view its logs
kubectl logs deployment/configmap-env-var
```

You should see an output similar to the below:
```
Found 3 pods, using pod/configmap-env-var-6d9ff89fb6-bzcf6
Thu Jan  4 16:30:35 UTC 2024 The basket is full of mangoes
Thu Jan  4 16:30:45 UTC 2024 The basket is full of mangoes
Thu Jan  4 16:30:55 UTC 2024 The basket is full of mangoes
```

This demonstrates the scenario of updating environment variables in a Pod that are derived
from a ConfigMap. Changes to the ConfigMap values are applied to the Pod during the subsequent
rollout. If Pods get created for another reason, such as scaling up the Deployment, then the new Pods
also use the latest configuration values; if you don't trigger a rollout, then you might find that your
app is running with a mix of old and new environment variable values.


## Update configuration via a ConfigMap in a multi-container Pod {#rollout-configmap-multiple-containers}

Use the `kubectl create configmap` command to create a ConfigMap from [literal values](/docs/tasks/configure-pod-container/configure-pod-configmap/#create-configmaps-from-literal-values):

```shell
kubectl create configmap color --from-literal=color=red
```
Below is an example manifest for a Deployment that manages a set of Pods, each with two containers. The two containers share an `emptyDir` volume that they use to communicate.
The first container runs a web server (`nginx`). The mount path for the shared volume in the web server container is `/usr/share/nginx/html`. The second helper container is based on `alpine`, and for this container the `emptyDir` volume is mounted at `/pod-data`. The helper container writes a file in HTML that has its content based on a ConfigMap. The web server container serves the HTML via HTTP.

{{% code_sample file="deployments/deployment-with-configmap-two-containers.yaml" %}}

Create the Deployment:
```shell
kubectl apply -f https://k8s.io/examples/deployments/deployment-with-configmap-two-containers.yaml
```

Check the pods for this Deployment to ensure they are ready (matching by
{{< glossary_tooltip text="selector" term_id="selector" >}}):
```shell
kubectl get pods --selector=app.kubernetes.io/name=configmap-two-containers
```
You should see an output similar to:
```
NAME                                        READY   STATUS    RESTARTS   AGE
configmap-two-containers-565fb6d4f4-2xhxf   2/2     Running   0          20s
configmap-two-containers-565fb6d4f4-g5v4j   2/2     Running   0          20s
configmap-two-containers-565fb6d4f4-mzsmf   2/2     Running   0          20s
````

Expose the Deployment (the `kubectl` tool creates a
{{<glossary_tooltip text="Service" term_id="service">}}  for you):
```shell
kubectl expose deployment configmap-two-containers --name=configmap-service --port=8080 --target-port=80
```

Use `kubectl` to forward the port:
```shell
kubectl port-forward service/configmap-service 8080:8080 & # this stays running in the background
```

Access the service.
```shell
curl http://localhost:8080
```

You should see an output similar to:
```
Fri Jan  5 08:08:22 UTC 2024 My preferred color is red
```

Edit the ConfigMap:
```shell
kubectl edit configmap color
```

In the editor that appears, change the value of key `color` from `red` to `blue`. Save your changes.
The kubectl tool updates the ConfigMap accordingly (if you see an error, try again).

Here's an example of how that manifest could look after you edit it:

```yaml
apiVersion: v1
data:
  color: blue
kind: ConfigMap
# You can leave the existing metadata as they are.
# The values you'll see won't exactly match these.
metadata:
  creationTimestamp: "2024-01-05T08:12:05Z"
  name: color
  namespace: configmap
  resourceVersion: "1801272"
  uid: 80d33e4a-cbb4-4bc9-ba8c-544c68e425d6
```

Loop over the service URL for few seconds.
```shell
# Cancel this when you're happy with it (Ctrl-C)
while true; do curl --connect-timeout 7.5 http://localhost:8080; sleep 10; done
```

You should see the output change as follows:
```
Fri Jan  5 08:14:00 UTC 2024 My preferred color is red
Fri Jan  5 08:14:02 UTC 2024 My preferred color is red
Fri Jan  5 08:14:20 UTC 2024 My preferred color is red
Fri Jan  5 08:14:22 UTC 2024 My preferred color is red
Fri Jan  5 08:14:32 UTC 2024 My preferred color is blue
Fri Jan  5 08:14:43 UTC 2024 My preferred color is blue
Fri Jan  5 08:15:00 UTC 2024 My preferred color is blue
```

## Update configuration via a ConfigMap in a Pod possessing a sidecar container {#rollout-configmap-sidecar}

The above scenario can be replicated by using a [Sidecar Container](/docs/concepts/workloads/pods/sidecar-containers/) as a helper container to write the HTML file.  
As a Sidecar Container is conceptually an Init Container, it is guaranteed to start before the main web server container.  
This ensures that the HTML file is always available when the web server is ready to serve it.  
Please see [Enabling sidecar containers](/docs/concepts/workloads/pods/sidecar-containers/#enabling-sidecar-containers) to utilize this feature.

If you are continuing from the previous scenario, you can reuse the ConfigMap named `color` for this scenario.  
If you are executing this scenario independently, use the `kubectl create configmap` command to create a ConfigMap from [literal values](/docs/tasks/configure-pod-container/configure-pod-configmap/#create-configmaps-from-literal-values):

```shell
kubectl create configmap color --from-literal=color=blue
```

Below is an example manifest for a Deployment that manages a set of Pods, each with a main container and a sidecar container. The two containers share an `emptyDir` volume that they use to communicate.
The main container runs a web server (NGINX). The mount path for the shared volume in the web server container is `/usr/share/nginx/html`. The second container is a Sidecar Container based on Alpine Linux which acts as a helper container. For this container the `emptyDir` volume is mounted at `/pod-data`. The Sidecar Container writes a file in HTML that has its content based on a ConfigMap. The web server container serves the HTML via HTTP.

{{% code_sample file="deployments/deployment-with-configmap-and-sidecar-container.yaml" %}}

Create the Deployment:
```shell
kubectl apply -f https://k8s.io/examples/deployments/deployment-with-configmap-and-sidecar-container.yaml
```

Check the pods for this Deployment to ensure they are ready (matching by
{{< glossary_tooltip text="selector" term_id="selector" >}}):
```shell
kubectl get pods --selector=app.kubernetes.io/name=configmap-sidecar-container
```
You should see an output similar to:
```
NAME                                           READY   STATUS    RESTARTS   AGE
configmap-sidecar-container-5fb59f558b-87rp7   2/2     Running   0          94s
configmap-sidecar-container-5fb59f558b-ccs7s   2/2     Running   0          94s
configmap-sidecar-container-5fb59f558b-wnmgk   2/2     Running   0          94s
````

Expose the Deployment (the `kubectl` tool creates a
{{<glossary_tooltip text="Service" term_id="service">}}  for you):
```shell
kubectl expose deployment configmap-sidecar-container --name=configmap-sidecar-service --port=8081 --target-port=80
```

Use ```kubectl``` to forward the port:
```shell
kubectl port-forward service/configmap-sidecar-service 8081:8081 & # this stays running in the background
```

Access the service.
```shell
curl http://localhost:8081
```

You should see an output similar to:
```
Sat Feb 17 13:09:05 UTC 2024 My preferred color is blue
```

Edit the ConfigMap:
```shell
kubectl edit configmap color
```

In the editor that appears, change the value of key `color` from `blue` to `green`. Save your changes.
The kubectl tool updates the ConfigMap accordingly (if you see an error, try again).

Here's an example of how that manifest could look after you edit it:

```yaml
apiVersion: v1
data:
  color: green
kind: ConfigMap
# You can leave the existing metadata as they are.
# The values you'll see won't exactly match these.
metadata:
  creationTimestamp: "2024-02-17T12:20:30Z"
  name: color
  namespace: default
  resourceVersion: "1054"
  uid: e40bb34c-58df-4280-8bea-6ed16edccfaa
```

Loop over the service URL for few seconds.
```shell
# Cancel this when you're happy with it (Ctrl-C)
while true; do curl --connect-timeout 7.5 http://localhost:8081; sleep 10; done
```

You should see the output change as follows:
```
Sat Feb 17 13:12:35 UTC 2024 My preferred color is blue
Sat Feb 17 13:12:45 UTC 2024 My preferred color is blue
Sat Feb 17 13:12:55 UTC 2024 My preferred color is blue
Sat Feb 17 13:13:05 UTC 2024 My preferred color is blue
Sat Feb 17 13:13:15 UTC 2024 My preferred color is green
Sat Feb 17 13:13:25 UTC 2024 My preferred color is green
Sat Feb 17 13:13:35 UTC 2024 My preferred color is green
```

## Update configuration via an immutable ConfigMap that is mounted as a volume {#rollout-configmap-immutable-volume}

{{< note >}}
Immutable ConfigMaps are especially used for configuration that is constant and is **not** expected to change over time. Marking a ConfigMap as immutable allows a performance improvement where the kubelet does not watch for changes.

If you do need to make a change, you should plan to either:
- change the name of the ConfigMap, and switch to running Pods that reference the new name
- replace all the nodes in your cluster that have previously run a Pod that used the old value
- restart the kubelet on any node where the kubelet previously loaded the old ConfigMap
{{< /note >}}

An example manifest for an [Immutable ConfigMap](/docs/concepts/configuration/configmap/#configmap-immutable) is shown below.
{{% code_sample file="configmap/immutable-configmap.yaml" %}}

Create the Immutable ConfigMap:
```shell
kubectl apply -f https://k8s.io/examples/configmap/immutable-configmap.yaml
```

Below is an example of a Deployment manifest with the Immutable ConfigMap `company-name-20150801` mounted as a
{{< glossary_tooltip text="volume" term_id="volume" >}} into the Pod's only container.
{{% code_sample file="deployments/deployment-with-immutable-configmap-as-volume.yaml" %}}

Create the Deployment:
```shell
kubectl apply -f https://k8s.io/examples/deployments/deployment-with-immutable-configmap-as-volume.yaml
```

Check the pods for this Deployment to ensure they are ready (matching by
{{< glossary_tooltip text="selector" term_id="selector" >}}):
```shell
kubectl get pods --selector=app.kubernetes.io/name=immutable-configmap-volume
```

You should see an output similar to:
```
NAME                                          READY   STATUS    RESTARTS   AGE
immutable-configmap-volume-78b6fbff95-5gsfh   1/1     Running   0          62s
immutable-configmap-volume-78b6fbff95-7vcj4   1/1     Running   0          62s
immutable-configmap-volume-78b6fbff95-vdslm   1/1     Running   0          62s
```

The Pod's container refers to the data defined in the ConfigMap and uses it to print a report to stdout. You can check this report by viewing the logs for one of the Pods in that Deployment:
```shell
# Pick one Pod that belongs to the Deployment, and view its logs
kubectl logs deployments/immutable-configmap-volume
```

You should see an output similar to:
```
Found 3 pods, using pod/immutable-configmap-volume-78b6fbff95-5gsfh
Wed Mar 20 03:52:34 UTC 2024 The name of the company is ACME, Inc.
Wed Mar 20 03:52:44 UTC 2024 The name of the company is ACME, Inc.
Wed Mar 20 03:52:54 UTC 2024 The name of the company is ACME, Inc.
```

{{< note >}}
Once a ConfigMap is marked as immutable, it is not possible to revert this change 
nor to mutate the contents of the data or the binaryData field.  
In order to modify the behavior of the Pods that use this configuration, 
you will create a new immutable ConfigMap and edit the Deployment 
to define a slightly different pod template, referencing the new ConfigMap.
{{< /note >}}

Create a new immutable ConfigMap by using the manifest shown below:
{{% code_sample file="configmap/new-immutable-configmap.yaml" %}}

```shell
kubectl apply -f https://k8s.io/examples/configmap/new-immutable-configmap.yaml
```

You should see an output similar to:
```
configmap/company-name-20240312 created
```

Check the newly created ConfigMap:

```shell
kubectl get configmap
```

You should see an output displaying both the old and new ConfigMaps:
```
NAME                    DATA   AGE
company-name-20150801   1      22m
company-name-20240312   1      24s
```

Modify the Deployment to reference the new ConfigMap.  

Edit the Deployment:
```shell
kubectl edit deployment immutable-configmap-volume
```

In the editor that appears, update the existing volume definition to use the new ConfigMap.

```yaml
volumes:
- configMap:
    defaultMode: 420
    name: company-name-20240312 # Update this field
  name: config-volume
```
You should see the following output:
```
deployment.apps/immutable-configmap-volume edited
```

This will trigger a rollout. Wait for all the previous Pods to terminate and the new Pods to be in a ready state.

Monitor the status of the Pods:
```shell
kubectl get pods --selector=app.kubernetes.io/name=immutable-configmap-volume
```

```
NAME                                          READY   STATUS        RESTARTS   AGE
immutable-configmap-volume-5fdb88fcc8-29v8n   1/1     Running       0          13s
immutable-configmap-volume-5fdb88fcc8-52ddd   1/1     Running       0          14s
immutable-configmap-volume-5fdb88fcc8-n5jx4   1/1     Running       0          15s
immutable-configmap-volume-78b6fbff95-5gsfh   1/1     Terminating   0          32m
immutable-configmap-volume-78b6fbff95-7vcj4   1/1     Terminating   0          32m
immutable-configmap-volume-78b6fbff95-vdslm   1/1     Terminating   0          32m

```

You should eventually see an output similar to:
```
NAME                                          READY   STATUS    RESTARTS   AGE
immutable-configmap-volume-5fdb88fcc8-29v8n   1/1     Running   0          43s
immutable-configmap-volume-5fdb88fcc8-52ddd   1/1     Running   0          44s
immutable-configmap-volume-5fdb88fcc8-n5jx4   1/1     Running   0          45s
```

View the logs for a Pod in this Deployment:
```shell
# Pick one Pod that belongs to the Deployment, and view its logs
kubectl logs deployment/immutable-configmap-volume
```

You should see an output similar to the below:
```
Found 3 pods, using pod/immutable-configmap-volume-5fdb88fcc8-n5jx4
Wed Mar 20 04:24:17 UTC 2024 The name of the company is Fiktivesunternehmen GmbH
Wed Mar 20 04:24:27 UTC 2024 The name of the company is Fiktivesunternehmen GmbH
Wed Mar 20 04:24:37 UTC 2024 The name of the company is Fiktivesunternehmen GmbH
```

Once all the deployments have migrated to use the new immutable ConfigMap, it is advised to delete the old one.

```shell
kubectl delete configmap company-name-20150801 
```

## Summary

Changes to a ConfigMap mounted as a Volume on a Pod are available seamlessly after the subsequent kubelet sync.

Changes to a ConfigMap that configures environment variables for a Pod are available after the subsequent rollout for the Pod.

Once a ConfigMap is marked as immutable, it is not possible to revert this change (you cannot make an immutable ConfigMap mutable), and you also cannot make any change to the contents of the `data` or the `binaryData` field. You can delete and recreate the ConfigMap, or you can make a new different ConfigMap. When you delete a ConfigMap, running containers
and their Pods maintain a mount point to any volume that referenced that existing ConfigMap.


## {{% heading "cleanup" %}}
Terminate the `kubectl port-forward` commands in case they are running.

Delete the resources created during the tutorial:
```shell
kubectl delete deployment configmap-volume configmap-env-var configmap-two-containers configmap-sidecar-container immutable-configmap-volume
kubectl delete service configmap-service configmap-sidecar-service
kubectl delete configmap sport fruits color company-name-20240312

kubectl delete configmap company-name-20150801 # In case it was not handled during the task execution
```