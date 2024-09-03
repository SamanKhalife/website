---
reviewers:
- lachie83
- khenidak
- bridgetkromhout
min-kubernetes-server-version: v1.23
title: "اعتبارسنجی دوگانه IPv4/IPv6"
content_type: task
---

<!-- overview -->
این سند نحوه اعتبارسنجی خوشه‌های Kubernetes فعال شده برای دوگانه IPv4/IPv6 را به اشتراک می‌گذارد.


## {{% heading "پیش‌نیازها" %}}

* پشتیبانی از شبکه دوگانه توسط ارائه‌دهنده خدمات (ارائه‌دهنده ابر یا دیگر) برای رابط‌های شبکه‌ای IPv4/IPv6 مسیریابی شود.
* افزونه شبکه که از شبکه دوگانه پشتیبانی می‌کند.
* خوشه فعال شده برای دوگانه (Dual-stack).

{{< version-check >}}

{{< note >}}
اگرچه می‌توانید با نسخه‌های قدیمی‌تر از Kubernetes اعتبارسنجی کنید، این ویژگی تنها از ورژن v1.23 به بعد GA شده و به طور رسمی پشتیبانی می‌شود.
{{< /note >}}


<!-- steps -->

## اعتبارسنجی آدرس‌دهی

### اعتبارسنجی آدرس‌دهی نودها

هر نود دوگانه با یک بلوک IPv4 و یک بلوک IPv6 تخصیص داده شده باید داشته باشد. برای اعتبارسنجی برنامه‌ریزی بلوک‌های آدرس IPv4/IPv6 پادها، دستور زیر را اجرا کنید. نام نمونه نود را با یک نود دوگانه معتبر از خوشه خود جایگزین کنید. در این مثال، نام نود `k8s-linuxpool1-34450317-0` است:

```shell
kubectl get nodes k8s-linuxpool1-34450317-0 -o go-template --template='{{range .spec.podCIDRs}}{{printf "%s\n" .}}{{end}}'
```
```
10.244.1.0/24
2001:db8::/64
```
باید یک بلوک IPv4 و یک بلوک IPv6 تخصیص داده شده باشد.

اعتبارسنجی کنید که نود دارای یک رابط IPv4 و IPv6 تشخیص داده شده است. نام نمونه نود را با یک نود معتبر از خوشه خود جایگزین کنید. در این مثال، نام نود `k8s-linuxpool1-34450317-0` است:

```shell
kubectl get nodes k8s-linuxpool1-34450317-0 -o go-template --template='{{range .status.addresses}}{{printf "%s: %s\n" .type .address}}{{end}}'
```
```
Hostname: k8s-linuxpool1-34450317-0
InternalIP: 10.0.0.5
InternalIP: 2001:db8:10::5
```

### اعتبارسنجی آدرس‌دهی پادها

اعتبارسنجی کنید که یک پاد دارای آدرس IPv4 و IPv6 تخصیص داده شده است. نام پاد معتبر خود را در خوشه خود با یک پاد جایگزین کنید. در این مثال، نام پاد `pod01` است:

```shell
kubectl get pods pod01 -o go-template --template='{{range .status.podIPs}}{{printf "%s\n" .ip}}{{end}}'
```
```
10.244.1.4
2001:db8::4
```

شما همچنین می‌توانید از API Downward با فیلد `status.podIPs` برای اعتبارسنجی آدرس‌های IP پاد استفاده کنید. شیوه زیر نمایش می‌دهد چگونه می‌توانید آدرس‌های IP پاد را از طریق متغیر محیطی با نام `MY_POD_IPS` در یک ظرفیت اجرا کننده نمایش دهید.

```
        env:
        - name: MY_POD_IPS
          valueFrom:
            fieldRef:
              fieldPath: status.podIPs
```

دستور زیر مقدار متغیر محیطی `MY_POD_IPS` را از داخل یک ظرفیت اجرا کننده نمایش می‌دهد. ارزش یک لیست جداگانه که با آدرس‌های IPv4 و IPv6 پاد مطابقت دارد.

```shell
kubectl exec -it pod01 -- set | grep MY_POD_IPS
```
```
MY_POD_IPS=10.244.1.4,2001:db8::4
```

آدرس‌های IP پاد نیز در `/etc/hosts` در داخل یک ظرفیت اجرا کننده نمایش داده می‌شوند. دستور زیر یک cat را بر روی `/etc/hosts` در یک پاد دوگانه اجرا می‌کند. از خروجی می‌توانید آدرس‌های IP IPv4 و IPv6 برای پاد را بررسی کنید.

```shell
kubectl exec -it pod01 -- cat /etc/hosts
```
```
# Kubernetes-managed hosts file.
127.0.0.1    localhost
::1    localhost ip6-localhost ip6-loopback
fe00::0    ip6-localnet
fe00::0    ip6-mcastprefix
fe00::1    ip6-allnodes
fe00::2    ip6-allrouters
10.244.1.4    pod01
2001:db8::4    pod01
```

## اعتبارسنجی خدمات

ایجاد خدمات زیر را که `.spec.ipFamilyPolicy` را به طور صریح تعریف نکرده است. Kubernetes برای خدمات یک آدرس IP خوشه را از اولین `service-cluster-ip-range` پیکربندی می‌کند و `.spec.ipFamilyPolicy` را به `SingleStack` تنظیم می‌کند.

{{% code_sample file="service/networking/dual-stack-default-svc.yaml" %}}

از `kubectl` برای مشاهده YAML خدمات استفاده کنید.

```shell
kubectl get svc my-service -o yaml
```

در خدمات `.spec.ipFamilyPolicy` به `SingleStack` تنظیم شده و `.spec.clusterIP` به آدرس IPv4 از اولین محدوده تنظیم شده از طریق پرچم `--service-cluster-ip-range` در kube-controller-manager.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: default
spec:
  clusterIP: 10.0.217.164
  clusterIPs:
  - 10.0.217.164
  ipF

amilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 80
    protocol: TCP
    targetPort: 9376
  selector:
    app.kubernetes.io/name: MyApp
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

ایجاد خدمات زیر را که `IPv6` را به عنوان عنصر اول در `.spec.ipFamilies` به طور صریح تعریف می‌کند. Kubernetes برای خدمات یک آدرس IP خوشه را از محدوده IPv6 تنظیم شده `--service-cluster-ip-range` پیکربندی می‌کند و `.spec.ipFamilyPolicy` را به `SingleStack` تنظیم می‌کند.

{{% code_sample file="service/networking/dual-stack-ipfamilies-ipv6.yaml" %}}

از `kubectl` برای مشاهده YAML خدمات استفاده کنید.

```shell
kubectl get svc my-service -o yaml
```

در خدمات `.spec.ipFamilyPolicy` به `SingleStack` تنظیم شده و `.spec.clusterIP` به آدرس IPv6 از محدوده IPv6 تنظیم شده از طریق پرچم `--service-cluster-ip-range` در kube-controller-manager.

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: MyApp
  name: my-service
spec:
  clusterIP: 2001:db8:fd00::5118
  clusterIPs:
  - 2001:db8:fd00::5118
  ipFamilies:
  - IPv6
  ipFamilyPolicy: SingleStack
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app.kubernetes.io/name: MyApp
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

ایجاد خدمات زیر را که `.spec.ipFamilyPolicy` را به `PreferDualStack` تعریف می‌کند. Kubernetes هر دو آدرس IPv4 و IPv6 را (زیرا این خوشه دارای دوگانه است) اختیار دارد و `.spec.ClusterIP` را از لیست `.spec.ClusterIPs` بر اساس خانواده آدرس اولیه در آرایه `.spec.ipFamilies` انتخاب می‌کند.

{{% code_sample file="service/networking/dual-stack-preferred-svc.yaml" %}}

{{< note >}}
دستور `kubectl get svc` فقط IP اصلی را در فیلد `CLUSTER-IP` نشان می‌دهد.

```shell
kubectl get svc -l app.kubernetes.io/name=MyApp

NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
my-service   ClusterIP   10.0.216.242   <none>        80/TCP    5s
```
{{< /note >}}

اعتبارسنجی کنید که خدمات آدرس‌های خوشه را از بلوک‌های آدرس IPv4 و IPv6 با استفاده از `kubectl describe` دریافت می‌کنند. سپس می‌توانید دسترسی به خدمات را از طریق آدرس‌ها و پورت‌ها اعتبارسنجی کنید.

```shell
kubectl describe svc -l app.kubernetes.io/name=MyApp
```

```
Name:              my-service
Namespace:         default
Labels:            app.kubernetes.io/name=MyApp
Annotations:       <none>
Selector:          app.kubernetes.io/name=MyApp
Type:              ClusterIP
IP Family Policy:  PreferDualStack
IP Families:       IPv4,IPv6
IP:                10.0.216.242
IPs:               10.0.216.242,2001:db8:fd00::af55
Port:              <unset>  80/TCP
TargetPort:        9376/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>
```

### ایجاد یک خدمات بارگذاری دوگانه

اگر ارائه‌دهنده ابر از ایجاد بارگذاری خارجی دارای قابلیت IPv6 پشتیبانی کند، خدمات زیر را با `PreferDualStack` در `.spec.ipFamilyPolicy`، `IPv6` به عنوان عنصر اول آرایه `.spec.ipFamilies` و میدان `type` تنظیم شده به `LoadBalancer` ایجاد کنید.

{{% code_sample file="service/networking/dual-stack-prefer-ipv6-lb-svc.yaml" %}}

بررسی خدمات:

```shell
kubectl get svc -l app.kubernetes.io/name=MyApp
```

اعتبارسنجی کنید که خدمات یک آدرس `CLUSTER-IP` را از بلوک آدرس IPv6 دریافت می‌کنند همراه با یک `EXTERNAL-IP`. سپس می‌توانید دسترسی به خدمات را از طریق IP و پورت اعتبارسنجی کنید.

```shell
NAME         TYPE           CLUSTER-IP            EXTERNAL-IP        PORT(S)        AGE
my-service   LoadBalancer   2001:db8:fd00::7ebc   2603:1030:805::5   80:30790/TCP   35s
```