---
reviewers:
- rickypai
- thockin
title: "افزودن موارد به فایل /etc/hosts با استفاده از HostAliases"
content_type: task
weight: 60
min-kubernetes-server-version: 1.7
---

<!-- overview -->

افزودن موارد به فایل `/etc/hosts` یک پاد، امکان ایجاد اولویت برای رزولوشن نام‌ها در سطح پاد را بدون استفاده از DNS و گزینه‌های دیگر فراهم می‌کند. شما می‌توانید این ورودی‌های سفارشی را با فیلد HostAliases در مشخصه PodSpec اضافه کنید.

توصیه می‌شود که تغییرات انجام شده بدون استفاده از HostAliases انجام نشود، زیرا فایل توسط kubelet مدیریت می‌شود و ممکن است در طول ایجاد / راه‌اندازی مجدد پاد بازنویسی شود.

<!-- steps -->

## محتوای پیش‌فرض فایل hosts

یک پاد Nginx را که به یک IP پاد اختصاص داده شده است شروع کنید:

```shell
kubectl run nginx --image nginx
```

```
pod/nginx created
```

آدرس IP پاد را بررسی کنید:

```shell
kubectl get pods --output=wide
```

```
NAME     READY     STATUS    RESTARTS   AGE    IP           NODE
nginx    1/1       Running   0          13s    10.200.0.4   worker0
```

محتوای فایل hosts به این شکل خواهد بود:

```shell
kubectl exec nginx -- cat /etc/hosts
```

```
# فایل hosts توسط Kubernetes مدیریت می‌شود.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.200.0.4	nginx
```

به طور پیش‌فرض، فایل hosts شامل بویلرپلیت‌های IPv4 و IPv6 مانند localhost و نام میزبان خود است.

## اضافه کردن موارد اضافی با استفاده از HostAliases

علاوه بر بویلرپلیت پیش‌فرض، می‌توانید ورودی‌های اضافی را به فایل hosts اضافه کنید.
به عنوان مثال: برای حل نام‌های foo.local و bar.local به 127.0.0.1 و foo.remote و bar.remote به 10.1.2.3، می‌توانید HostAliases را برای یک پاد در زیر `.spec.hostAliases` پیکربندی کنید:

{{% code_sample file="service/networking/hostaliases-pod.yaml" %}}

می‌توانید یک پاد با این پیکربندی را اجرا کنید با اجرای دستور زیر:

```shell
kubectl apply -f https://k8s.io/examples/service/networking/hostaliases-pod.yaml
```

```
pod/hostaliases-pod created
```

جزئیات یک پاد را برای دیدن آدرس IPv4 و وضعیت آن بررسی کنید:

```shell
kubectl get pod --output=wide
```

```
NAME                           READY     STATUS      RESTARTS   AGE       IP              NODE
hostaliases-pod                0/1       Completed   0          6s        10.200.0.5      worker0
```

محتوای فایل hosts به این شکل خواهد بود:

```shell
kubectl logs hostaliases-pod
```

```
# فایل hosts توسط Kubernetes مدیریت می‌شود.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.200.0.5	hostaliases-pod

# ورودی‌های اضافه شده توسط HostAliases.
127.0.0.1	foo.local	bar.local
10.1.2.3	foo.remote	bar.remote
```

با ورودی‌های اضافی که در پایین مشخص شده است.

## چرا kubelet فایل hosts را مدیریت می‌کند؟ {#why-does-kubelet-manage-the-hosts-file}

kubelet فایل hosts را برای هر کانتینر پاد مدیریت می‌کند تا از این جلوگیری شود که رانتایم کانتینر فایل را بعد از شروع کانتینر‌ها تغییر دهد.
تاریخچه‌ای که به تازگی به استفاده از Docker Engine به عنوان رانتایم کانتینر Kubernetes تبدیل شده است، Docker Engine پس از هر کانتینر آغاز شده، فایل `/etc/hosts` را تغییر داده است.

در حال حاضر Kubernetes می‌تواند از انواع مختلف رانتایم کانتینر استفاده کند؛ با این حال، kubelet فایل hosts را در هر کانتینر مدیریت می‌کند تا نتیجه به عنوان انتظار شده باشد، بدون توجه به اینکه از کدام رانتایم کانتینر استفاده می‌کنید.

{{< caution >}}
از ایجاد تغییرات دستی در فایل hosts داخل یک کانتینر خودداری کنید.

اگر تغییرات دستی در فایل hosts را انجام دهید، این تغییرات هنگام خروج کانتینر از دست می‌رود.
{{< /caution >}}
