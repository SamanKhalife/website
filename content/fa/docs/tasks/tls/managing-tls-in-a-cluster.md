---
title: "مدیریت گواهی‌نامه‌های TLS در یک خوشه"
content_type: task
reviewers:
- mikedanese
- beacham
- liggit
---

<!-- overview -->

Kubernetes از API `certificates.k8s.io` پشتیبانی می‌کند که به شما امکان می‌دهد تا گواهی‌نامه‌های TLS را که توسط یک Certificate Authority (CA) کنترل شده‌اند، تأمین کنید. این CA و گواهی‌نامه‌ها می‌توانند توسط بارگذاری‌های شما برای برقراری اعتماد استفاده شوند.

API `certificates.k8s.io` از یک پروتکل استفاده می‌کند که مشابه با [پیش‌نویس ACME](https://github.com/ietf-wg-acme/acme/) است.

{{< note >}}
گواهی‌نامه‌هایی که با استفاده از API `certificates.k8s.io` ایجاد می‌شوند، توسط یک [CA اختصاصی](#configuring-your-cluster-to-provide-signing) امضا می‌شوند. امکان تنظیم خوشه‌ی شما برای استفاده از CA اصلی خوشه برای این منظور وجود دارد، اما هرگز باید به آن اعتماد نکنید. از این CA استفاده نکنید که گواهی‌نامه‌ها به طور اشتباه با CA اصلی خوشه تأیید شوند.
{{< /note >}}


## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}}

شما نیاز به ابزار `cfssl` دارید. می‌توانید `cfssl` را از
[https://github.com/cloudflare/cfssl/releases](https://github.com/cloudflare/cfssl/releases)
دانلود کنید.

برخی از مراحل در این صفحه از ابزار `jq` استفاده می‌کنند. اگر `jq` را ندارید، می‌توانید آن را از طریق منابع نرم‌افزاری سیستم عامل خود نصب کنید یا از
[https://jqlang.github.io/jq/](https://jqlang.github.io/jq/)
دریافت کنید.

<!-- steps -->

## اعتماد به TLS در یک خوشه

اعتماد به [CA سفارشی](#configuring-your-cluster-to-provide-signing) توسط یک برنامه‌ای که به‌عنوان یک pod اجرا می‌شود، معمولاً نیازمند پیکربندی اضافی برنامه است. شما باید باید پرونده گواهی CA را به لیست گواهی‌نامه‌های CA که مشتری TLS یا سرور به آن اعتماد می‌کند، اضافه کنید. به‌عنوان مثال، شما می‌توانید این کار را با پیکربندی TLS golang انجام دهید با تجزیه‌وتحلیل زنجیره گواهی و اضافه کردن گواهی‌نامه‌های تجزیه‌شده به فیلد `RootCAs` در ساختار [`tls.Config`](https://pkg.go.dev/crypto/tls#Config).

{{< note >}}
اگرچه گواهی CA سفارشی ممکن است در فایل سیستم (در ConfigMap `kube-root-ca.crt`) درج شده باشد، شما نباید از آن Authority CA برای هیچ منظوری جز برای تأیید نقاط پایانی داخلی Kubernetes استفاده کنید. یک نمونه از یک نقطه پایانی داخلی Kubernetes، سرویس با نام `kubernetes` در فضای‌نامه پیش‌فرض است.
اگر می‌خواهید از یک Authority CA سفارشی برای بارگذاری‌های خود استفاده کنید، باید آن CA را جداگانه تولید کرده و گواهی CA خود را با استفاده از یک
[ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap)
که پادهای شما به خواندن دسترسی دارند، پخش کنید.
{{< /note >}}

## درخواست یک گواهی‌نامه

بخش زیر نشان می‌دهد که چگونه یک گواهی‌نامه TLS برای یک سرویس Kubernetes که از طریق DNS دسترسی پیدا می‌کند، ایجاد کنید.

{{< note >}}
این آموزش از CFSSL استفاده می‌کند: CFSSL ابزار PKI و TLS Cloudflare. [اینجا کلیک کنید](https://blog.cloudflare.com/introducing-cfssl/) تا بیشتر بدانید.
{{< /note >}}

## ایجاد یک درخواست امضای گواهی‌نامه

برای ایجاد یک کلید خصوصی و درخواست امضای گواهی (یا CSR)، دستور زیر را اجرا کنید:

```shell
cat <<EOF | cfssl genkey - | cfssljson -bare server
{
  "hosts": [
    "my-svc.my-namespace.svc.cluster.local",
    "my-pod.my-namespace.pod.cluster.local",
    "192.0.2.24",
    "10.0.34.2"
  ],
  "CN": "my-pod.my-namespace.pod.cluster.local",
  "key": {
    "algo": "ecdsa",
    "size": 256
  }
}
EOF
```

که `192.0.2.24` آدرس IP خوشه سرویس است،
`my-svc.my-namespace.svc.cluster.local` نام DNS سرویس است،
`10.0.34.2` IP پاد است و `my-pod.my-namespace.pod.cluster.local`
نام DNS پاد است. شما باید خروجی مشابه زیر را ببینید:

```
2022/02/01 11:45:32 [INFO] generate received request
2022/02/01 11:45:32 [INFO] received CSR
2022/02/01 11:45:32 [INFO] generating key: ecdsa-256
2022/02/01 11:45:32 [INFO] encoded CSR
```

این دستور دو پرونده تولید می‌کند؛ فایل `server.csr` شامل درخواست گواهی انکد شده با PEM
[P‌KCS#10](https://tools.ietf.org/html/rfc2986)، و `server-key.pem` که شامل کلید انکد شده با PEM است که ب

رای گواهی که هنوز ایجاد نشده است.

## ایجاد یک شیء CertificateSigningRequest برای ارسال به API Kubernetes

یک مانیفست CSR (در YAML) را تولید کنید و آن را به سرور API ارسال کنید. شما می‌توانید این کار را با اجرای دستور زیر انجام دهید:

```shell
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: my-svc.my-namespace
spec:
  request: $(cat server.csr | base64 | tr -d '\n')
  signerName: example.com/serving
  usages:
  - digital signature
  - key encipherment
  - server auth
EOF
```

توجه داشته باشید که فایل `server.csr` که در مرحله اول ایجاد شد، با استفاده از base64 کد شده و در فیلد `.spec.request` نهفته شده است. شما همچنین یک گواهی با استفاده از کلید‌های "digital signature"، "key encipherment" و "server auth" درخواست کرده‌اید که توسط یک signer `example.com/serving` مثالی امضا شود.
می‌توانید اطلاعات مستند را برای [نام‌های signer پشتیبانی شده](/docs/reference/access-authn-authz/certificate-signing-requests/#signers)
برای اطلاعات بیشتر مشاهده کنید.

CSR اکنون باید از API در وضعیت Pending قابل مشاهده باشد. شما می‌توانید آن را با اجرای دستور زیر مشاهده کنید:

```shell
kubectl describe csr my-svc.my-namespace
```

```none
Name:                   my-svc.my-namespace
Labels:                 <none>
Annotations:            <none>
CreationTimestamp:      Tue, 01 Feb 2022 11:49:15 -0500
Requesting User:        yourname@example.com
Signer:                 example.com/serving
Status:                 Pending
Subject:
        Common Name:    my-pod.my-namespace.pod.cluster.local
        Serial Number:
Subject Alternative Names:
        DNS Names:      my-pod.my-namespace.pod.cluster.local
                        my-svc.my-namespace.svc.cluster.local
        IP Addresses:   192.0.2.24
                        10.0.34.2
Events: <none>
```

## تأیید درخواست امضای گواهی‌نامه‌ها {#get-the-certificate-signing-request-approved}

تأیید [درخواست امضای گواهی‌نامه](/docs/reference/access-authn-authz/certificate-signing-requests/)
ممکن است به‌طور خودکار یا توسط یک مدیر خوشه انجام شود. اگر شما مجاز به تأیید درخواست گواهی‌نامه هستید، می‌توانید این کار را با استفاده از `kubectl` به‌صورت دستی انجام دهید. به‌عنوان مثال:

```shell
kubectl certificate approve my-svc.my-namespace
```

```none
certificatesigningrequest.certificates.k8s.io/my-svc.my-namespace approved
```

حالا باید مشاهده کنید که:

```shell
kubectl get csr
```

```none
NAME                  AGE   SIGNERNAME            REQUESTOR              REQUESTEDDURATION   CONDITION
my-svc.my-namespace   10m   example.com/serving   yourname@example.com   <none>              Approved
```

این به این معنی است که درخواست گواهی‌نامه تأیید شده است و منتظر امضای signer درخواست شده است.

## امضای درخواست امضای گواهی‌نامه {#sign-the-certificate-signing-request}

حالا نقش امضاکننده را بازی کنید، گواهی را صادر کنید و آن را به API آپلود کنید.

امضاکننده معمولاً API `CertificateSigningRequest` را برای اشیاء با `signerName` خودش نظارت می‌کند، مطمئن می‌شود که آنها تأیید شده‌اند، گواهی‌نامه‌ها را برای این درخواست‌ها امضا می‌کند و وضعیت شیء API را با گواهی‌نامه صادر شده به‌روزرسانی می‌کند.

### ایجاد یک Certificate Authority

شما نیاز به یک اتوریته برای امضای دیجیتال بر روی گواهی‌نامه جدید دارید.

اولین گام، ایجاد یک گواهی امضایی با اجرای دستور زیر است:

```shell
cat <<EOF | cfssl gencert -initca - | cfssljson -bare ca
{
  "CN": "My Example Signer",
  "key": {
    "algo": "rsa",
    "size": 2048
  }
}
EOF
```

شما باید خروجی مشابه زیر را ببینید:

```none
2022/02/01 11:50:39 [INFO] generating a new CA key and certificate from CSR
2022/02/01 11:50:39 [INFO] generate received request
2022/02/01 11:50:39 [INFO] received CSR
2022/02/01 11:50:39 [INFO] generating key: rsa-2048
2022/02/01 11:50:39 [INFO] encoded CSR
2022/02/01 11:50:39 [INFO] signed certificate with serial number 263983151013686720899716354349605500797834580472
```

این یک فایل کلید گواهی‌نامه اتوریته (`ca-key.pem`) و گواهی‌نامه (`ca.pem`) تولید می‌کند.

### صادر کردن یک گواهی‌نامه

استفاده از یک تنظیم امضایی `server-signing-config.json` و فایل‌های کلیدی و گواهی‌نامه اتوریته برای امضای درخواست گواهی‌نامه به صورت زیر است:

```shell
kubectl get csr my-svc.my-namespace -o jsonpath='{.spec.request}' | \
  base64 --decode | \
  cfssl sign -ca ca.pem -ca-key ca-key.pem -config server-signing-config.json - | \
  cfssljson -bare ca-signed-server
```

شما باید خروجی مشابه زیر را ببینید:

```
2022/02/01 11:52:26 [INFO] signed certificate with serial number 576048928624926584381415936700914530534472870337
```

این یک فایل گواهی‌نامه سرویس امضایی صادر شده (`ca-signed-server.pem`) تولید می‌کند.

### آپلود گواهی‌نامه امضا شده

در نهایت، گواهی‌نامه امضا شده را در وضعیت شیء API آپلود کنید:

```shell
kubectl get csr my-svc.my-namespace -o json | \
  jq '.status.certificate = "'$(base64 ca-signed-server.pem | tr -d '\n')'"' | \
  kubectl replace --raw /apis/certificates.k8s.io/v1/certificatesigningrequests/my-svc.my-namespace/status -f -
```

{{< note >}}
این از ابزار خط فرمان [`jq`](https://jqlang.github.io/jq/) برای پر کردن محتوای base64-کد شده در فیلد `.status.certificate` استفاده می‌کند. اگر `jq` را ندارید، می‌توانید خروجی JSON را در یک فایل ذخیره کنید، این فیلد را به‌صورت دستی پر کنید و فایل نتیجه‌گرا را آپلود کنید.
{{< /note >}}

بعد از تأیید درخواست گواهی‌نامه و آپلود گواهی‌نامه امضا شده، اجرای دستور زیر را انجام دهید:

```shell
kubectl get csr
```

خروجی مشابه زیر خواهد بود:

```none
NAME                  AGE   SIGNERNAME            REQUESTOR              REQUESTEDDURATION   CONDITION
my-svc.my-namespace   20m   example.com/serving   yourname@example.com   <none>              Approved,Issued
```

## دانلود گواهی‌نامه و استفاده از آن

حالا به عنوان کاربر درخواست‌کننده، می‌توانید گواهی‌نامه صادر شده را دانلود کرده و آن را به فایل `server.crt` ذخیره کنید با اجرای دستور زیر:

```shell
kubectl get csr my-svc.my-namespace -o jsonpath='{.status.certificate}' \
    | base64 --decode > server.crt
```

حالا می‌توانید `server.crt` و `server-key.pem` را در یک
{{< glossary_tooltip text="Secret" term_id="secret" >}}
که بعداً می‌توانید آن را در یک Pod مانت کنید (برای مثال، برای استفاده با یک وب‌سرور که HTTPS را ارائه می‌دهد)، پر کنید.

```shell
kubectl create secret tls server --cert server.crt

 --key server-key.pem
```

```none
secret/server created
```

در نهایت، می‌توانید `ca.pem` را در یک {{< glossary_tooltip text="ConfigMap" term_id="configmap" >}} پر کنید و آن را به عنوان ریشه اعتماد برای تأیید گواهی‌نامه سرویس استفاده کنید:

```shell
kubectl create configmap example-serving-ca --from-file ca.crt=ca.pem
```

```none
configmap/example-serving-ca created
```

## تأیید درخواست‌های امضای گواهی‌نامه {#approving-certificate-signing-requests}

یک مدیر Kubernetes (با مجوزهای مناسب) می‌تواند به‌صورت دستی (یا رد) درخواست‌های امضای گواهی‌نامه‌ها را با استفاده از دستورات `kubectl certificate approve` و `kubectl certificate deny` انجام دهد. با این حال، اگر قصد دارید از این API استفاده سنگین کنید، ممکن است در نظر بگیرید که یک کنترل‌گر خودکار گواهی‌نامه‌ها بنویسید.

{{< caution >}}
قدرت تأیید CSRها تصمیم می‌دهد که هر کس به چه کسی اعتماد می‌کند. باید مطمئن شوید که شما به درستی الزامات تأیید را می‌فهمید و همچنین عواقب صدور یک گواهی‌نامه خاص را قبل از اعطای اجازه `approve` فهمیده‌اید.
{{< /caution >}}

اگر این دو نیاز ارضا شود، امضاکننده باید CSR را تأیید کند و در غیر این صورت باید CSR را رد کند.

برای اطلاعات بیشتر در مورد تأیید گواهی‌نامه و کنترل دسترسی، صفحه مرجع [درخواست‌های امضای گواهی‌نامه](/docs/reference/access-authn-authz/certificate-signing-requests/) را بخوانید.

## پیکربندی خوشه خود برای ارائه امضا

این صفحه فرض می‌کند که یک امضاکننده برای ارائه API گواهی‌نامه‌ها تنظیم شده است. مدیر کنترل‌گر Kubernetes پیاده‌سازی پیش‌فرض یک امضاکننده را فراهم می‌کند. برای فعال‌سازی آن، پارامترهای `--cluster-signing-cert-file` و
`--cluster-signing-key-file` را به مدیر کنترل‌گر با مسیرهای جفت کلیدی مرجع‌تان منتقل کنید.
