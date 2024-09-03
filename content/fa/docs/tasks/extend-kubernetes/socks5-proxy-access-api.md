---
title: استفاده از پروکسی SOCKS5 برای دسترسی به API Kubernetes
content_type: task
weight: 42
min-kubernetes-server-version: v1.24
---

<!-- overview -->

{{< feature-state for_k8s_version="v1.24" state="stable" >}}

این صفحه نحوه استفاده از یک پروکسی SOCKS5 برای دسترسی به API یک خوشه Kubernetes از راه دور را نشان می‌دهد. این کاربردی است زمانی که خوشه مورد نظر شما API خود را مستقیماً در اینترنت عمومی نمایش نمی‌دهد.

## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

شما نیاز به نرم‌افزار مشتری SSH (ابزار `ssh`) و یک سرویس SSH در سرور از راه دور دارید. شما باید بتوانید وارد سرویس SSH در سرور از راه دور شوید.

<!-- steps -->

## سند context

{{< note >}}
این مثال ترافیک را با استفاده از SSH و پروکسی SOCKS5 انتقال می‌دهد. شما می‌توانید به جای این روش از هر نوع دیگری از پروکسی [SOCKS5](https://en.wikipedia.org/wiki/SOCKS#SOCKS5) استفاده کنید.
{{</ note >}}

شکل ۱ نمایش می‌دهد که چه چیزی را در این وظیفه دنبال می‌کنید:

* شما یک کامپیوتر مشتری دارید که به عنوان محلی در مراحل زیر معرفی شده است، از جایی که شما به API Kubernetes متصل خواهید شد.
* سرور / API Kubernetes در یک سرور از راه دور میزبانی می‌شود.
* شما از نرم‌افزار مشتری و سرور SSH استفاده می‌کنید تا یک تونل امن SOCKS5 بین محلی و سرور از راه دور ایجاد کنید. ترافیک HTTPS بین مشتری و API Kubernetes از طریق تونل SOCKS5 جاری خواهد شد، که خود در تونل SSH است.

{{< mermaid >}}
graph LR;

  subgraph محلی[ماشین مشتری محلی]
  client([مشتری])-. ترافیک محلی .->  local_ssh[پروکسی SOCKS5 محلی SSH];
  end
  local_ssh[پروکسی SOCKS5 SSH]-- تونل SSH -->sshd

  subgraph سرور از راه دور[سرور از راه دور]
  sshd[سرور SSH]-- ترافیک محلی -->service1;
  end
  client([مشتری])-. ترافیک HTTPs از طریق پروکسی .->service1[API Kubernetes];

  classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
  classDef k8s fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
  classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
  class ingress,service1,service2,pod1,pod2,pod3,pod4 k8s;
  class client plain;
  class cluster cluster;
{{</ mermaid >}}
شکل ۱. مؤلفه‌های آموزش SOCKS5

## استفاده از ssh برای ایجاد پروکسی SOCKS5

دستور زیر یک پروکسی SOCKS5 بین کامپیوتر مشتری شما و سرور SOCKS5 از راه دور را شروع می‌کند:

```shell
# تونل SSH پس از اجرای این دستور به صورت foreground ادامه می‌یابد
ssh -D 1080 -q -N username@kubernetes-remote-server.example
```

پروکسی SOCKS5 به شما اجازه می‌دهد تا به سرور API خوشه خود متصل شوید بر اساس پیکربندی زیر:
* `-D 1080`: یک پروکسی SOCKS را در پورت محلی :1080 باز می‌کند.
* `-q`: حالت ساکت. باعث سکوت اکثر پیام‌های هشدار و تشخیصی می‌شود.
* `-N`: اجرای دستورات راه دور نکنید. برای فقط انتقال پورتها مفید است.
* `username@kubernetes-remote-server.example`: سرور SSH از راه دور که خوشه Kubernetes در پشت آن اجرا می‌شود (مثلاً یک میزبان bastion).

## پیکربندی مشتری

برای دسترسی به سرور API Kubernetes از طریق پروکسی باید `kubectl` را به ارسال درخواست‌ها از طریق پروکسی `SOCKS` ایجاد شده ابزاری به آنها اسم شما. این را با تنظیم متغیر محیطی مناسب از طریق یا از طریق ویژگی `proxy-url` در پرونده kubeconfig انجام دهید. استفاده از یک متغیر محیطی:

```shell
export HTTPS_PROXY=socks5://localhost:1080
```

برای همیشه استفاده از این تنظیم در یک context خاص `kubectl`، `proxy-url` را در ورودی مربوطه خود ،ه. مثلاً:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LRMEMMW2 # shortened for readability
    server: https://<API_SERVER_IP_ADDRESS>:6443  # the "Kubernetes API" server, in other words the IP address of kubernetes-remote-server.example
    proxy-url: socks5://localhost:1080   # the "SSH SOCKS5 proxy" in the diagram above
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    client-certificate-data: LS0tLS1CR== # shortened for readability
    client-key-data: LS0tLS1CRUdJT=      # shortened for readability
```

با ایجاد تونل از طریق دستور ssh مطرح شده و تعریف متغیر محیطی یا ویژگی `proxy-url`، می‌توانید از این پروکسی به عنوان مرجع خود با خوشه تعامل داشته باشید. برای مثال:

```shell
kubectl get pods
```

```console
NAMESPACE     NAME                                     READY   STATUS      RESTARTS   AGE
kube-system   coredns-85cb69466-klwq8

 1/1     Running     0          5m46s
```

{{< note >}}
- قبل از `kubectl` 1.24، بیشتر دستورات `kubectl` هنگام استفاده از پروکسی socks کار می‌کردند، به جز `kubectl exec`.
- `kubectl` از هر دو متغیر محیطی `HTTPS_PROXY` و `https_proxy` پشتیبانی می‌کند. این‌ها توسط برنامه‌های دیگری که پشتیبانی از SOCKS دارند مانند `curl` استفاده می‌شود. بنابراین در برخی موارد بهتر است که متغیر محیطی را در خط فرمان تعریف کنید:
  ```shell
  HTTPS_PROXY=socks5://localhost:1080 kubectl get pods
  ```
- هنگام استفاده از `proxy-url`، پروکسی فقط برای context مربوط به `kubectl` استفاده می‌شود، در حالی که متغیر محیطی تمام contextها را تحت تأثیر قرار می‌دهد.
- نام میزبان سرور API k8s می‌تواند با استفاده از نام سرویس `socks5h` از نشت DNS حفاظت بیشتری یابد. در این حالت، `kubectl` سرور API k8s می‌پرسد تا دامنه نام را حل کند، به جای حل آن در سیستم اجرای `kubectl`. توجه کنید که با `socks5h`، یک URL سرور API k8s مانند `https://localhost:6443/api` به کامپیوتر مشتری محلی شما ارجاع نمی‌دهد. به جای آن، به `localhost` به عنوان معروف در سرور پروکسی (مانند bastion ssh) ارجاع دارد.
{{</ note >}}

## تمیز کردن

برای متوقف کردن فرآیند انتقال پورت ssh روی ترمینال که در حال اجرا است، `CTRL+C` را فشار دهید.

در یک ترمینال، `unset https_proxy` را بنویسید تا از انتقال ترافیک http از طریق پروکسی جلوگیری کنید.

## مطالعه بیشتر

* [مشتری ورود از راه دور OpenSSH](https://man.openbsd.org/ssh)
