---
title: "دسترسی به خوشه‌ها"
weight: 20
content_type: concept
---

<!-- overview -->

در این موضوع، روش‌های مختلف برای تعامل با خوشه‌ها بررسی می‌شود.

<!-- body -->

## دسترسی اولیه با استفاده از kubectl

برای دسترسی به API Kubernetes برای اولین بار، پیشنهاد می‌شود از CLI Kubernetes، `kubectl`، استفاده کنید.

برای دسترسی به یک خوشه، نیاز دارید که موقعیت فیزیکی خوشه و اعتبارهای لازم برای دسترسی به آن را بدانید. معمولاً این موارد به‌طور خودکار تنظیم می‌شوند زمانی که از راهنمای شروع کار استفاده می‌کنید یا کسی خوشه را راه‌اندازی کرده و اعتبارها و موقعیت را به شما داده است.

موقعیت و اعتبارهایی که `kubectl` درباره آن‌ها اطلاع دارد را با این دستور بررسی کنید:

```shell
kubectl config view
```

بسیاری از [نمونه‌ها](/docs/reference/kubectl/quick-reference/) مقدمه‌ای برای استفاده از `kubectl` ارائه می‌دهند، و مستندات کامل در مرجع [kubectl](/docs/reference/kubectl/) قابل دسترسی است.

## دسترسی مستقیم به REST API

kubectl برای یافتن مکان و احراز هویت به apiserver راهنمایی می‌کند.
اگر می‌خواهید به طور مستقیم به REST API با استفاده از یک مشتری http مانند curl یا wget یا یک مرورگر دسترسی داشته باشید، چندین روش برای یافتن و احراز هویت وجود دارد:

- اجرای kubectl در حالت پروکسی.
  - روش توصیه شده.
  - از مکان ذخیره‌شده apiserver استفاده می‌کند.
  - هویت apiserver را با استفاده از گواهی احراز هویت خود-امضا شده بررسی می‌کند. هیچ امکان محتوای MITM وجود ندارد.
  - به apiserver احراز هویت می‌کند.
  - در آینده ممکن است توزیع بار هوشمند و failover را در سمت مشتری انجام دهد.
- مکان و اعتبارها را مستقیماً به مشتری http ارائه دهید.
  - رویکرد جایگزین.
  - با برخی از انواع کد مشتری که با استفاده از پروکسی گیج شده‌اند کار می‌کند.
  - نیاز به وارد کردن گواهی ریشه به مرورگر شما برای حفاظت در برابر MITM دارد.

### استفاده از kubectl proxy

دستور زیر kubectl را در حالتی اجرا می‌کند که به عنوان یک پروکسی معکوس عمل می‌کند. این دستور وظیفه یافتن apiserver و احراز هویت را دارد.
برای اجرای آن به این شکل عمل کنید:

```shell
kubectl proxy --port=8080
```

برای جزئیات بیشتر به [kubectl proxy](/docs/reference/generated/kubectl/kubectl-commands/#proxy) مراجعه کنید.

سپس می‌توانید API را با curl، wget یا یک مرورگر بررسی کنید، localhost را با [::1] برای IPv6 جایگزین کنید، به این صورت:

```shell
curl http://localhost:8080/api/
```

خروجی مشابه زیر خواهد بود:

```json
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "10.0.1.149:443"
    }
  ]
}
```

### بدون استفاده از kubectl proxy

استفاده از `kubectl apply` و `kubectl describe secret...` برای ایجاد یک توکن برای حساب خدمت پیش‌فرض با grep/cut:

ابتدا، Secret را ایجاد کنید و یک توکن برای ServiceAccount پیش‌فرض درخواست کنید:

```shell
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: default-token
  annotations:
    kubernetes.io/service-account.name: default
type: kubernetes.io/service-account-token
EOF
```

سپس، منتظر کنید تا کنترل‌کننده توکن Secret را با یک توکن پر کند:

```shell
while ! kubectl describe secret default-token | grep -E '^token' >/dev/null; do
  echo "waiting for token..." >&2
  sleep 1
done
```

تولید و استفاده از توکن تولید شده:

```shell
APISERVER=$(kubectl config view --minify | grep server | cut -f 2- -d ":" | tr -d " ")
TOKEN=$(kubectl describe secret default-token | grep -E '^token' | cut -f2 -d':' | tr -d " ")

curl $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure
```

خروجی مشابه زیر خواهد بود:

```json
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "10.0.1.149:443"
    }
  ]
}
```

از `jsonpath` استفاده می‌کنیم:

```shell
APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
TOKEN=$(kubectl get secret default-token -o jsonpath='{.data.token}' | base64 --decode)

curl $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure
```

خروجی مشابه زیر خواهد بود:

```json
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "10.0.1.149:443"
    }
  ]
}
```

مثال‌های فوق از پرچم `--insecure` استفاده می‌کنند. این موضوع باعث آسیب‌پذیری MITM م

ی‌شود. زمانی که kubectl به خوشه دسترسی دارد، از گواهی ریشه و گواهی‌های مشتری برای دسترسی به سرور استفاده می‌کند. (این‌ها در دایرکتوری `~/.kube` نصب می‌شوند). اغلب گواهی‌های خوشه به صورت خود-امضا هستند، لذا برای استفاده از مشتری http نیاز به پیکربندی خاصی است.

برخی از خوشه‌ها، apiserver به احراز هویت نیاز ندارد؛ ممکن است بر روی localhost فعال باشد یا توسط یک فایروال محافظت شود. برای این موضوع یک استاندارد وجود ندارد. [کنترل دسترسی به API](/docs/concepts/security/controlling-access) توضیح می‌دهد که یک مدیر خوشه چگونه می‌تواند این را پیکربندی کند.

## دسترسی برنامه‌نویسی به API

Kubernetes به‌طور رسمی کتابخانه‌های مشتری برای [Go](#go-client) و [Python](#python-client) را پشتیبانی می‌کند.

### کتابخانه Go

- برای دریافت کتابخانه، دستور زیر را اجرا کنید: `go get k8s.io/client-go@kubernetes-<شماره نسخه Kubernetes>`. برای دستورات نصب دقیق‌تر به [INSTALL.md](https://github.com/kubernetes/client-go/blob/master/INSTALL.md#for-the-casual-user) مراجعه کنید. برای دیدن کدام نسخه‌ها پشتیبانی می‌شوند، به [client-go compatibility matrix](https://github.com/kubernetes/client-go#compatibility-matrix) مراجعه کنید.
- برنامه‌ی خود را بر روی کتابخانه‌های client-go بنویسید. توجه داشته باشید که client-go شی‌های API خود را تعریف می‌کند، بنابراین در صورت نیاز، لطفاً تعریف‌های API را از client-go و نه از مخزن اصلی وارد کنید، مانند `import "k8s.io/client-go/kubernetes"`.

کتابخانه Go می‌تواند از همان فایل kubeconfig استفاده کند که CLI kubectl برای یافتن و احراز هویت به apiserver استفاده می‌کند. برای مثالی از این عملکرد، به [این صفحه](https://git.k8s.io/client-go/examples/out-of-cluster-client-configuration/main.go) مراجعه کنید.

### کتابخانه Python

برای استفاده از [کتابخانه Python](https://github.com/kubernetes-client/python)، دستور زیر را اجرا کنید: `pip install kubernetes`. برای گزینه‌های نصب بیشتر به صفحه [Python Client Library](https://github.com/kubernetes-client/python) مراجعه کنید.

کتابخانه Python می‌تواند از همان فایل kubeconfig استفاده کند که CLI kubectl برای یافتن و احراز هویت به apiserver استفاده می‌کند. برای مثالی از این عملکرد، به [این صفحه](https://github.com/kubernetes-client/python/tree/master/examples) مراجعه کنید.

### زبان‌های دیگر

برای دسترسی به API از زبان‌های دیگر نیز [کتابخانه‌های مشتری](/docs/reference/using-api/client-libraries/) وجود دارند. برای اطلاعات بیشتر در مورد کتابخانه‌های دیگر، به مستندات آن‌ها برای نحوه احراز هویت مراجعه کنید.

## دسترسی به API از یک Pod

هنگام دسترسی به API از یک Pod، یافتن و احراز هویت به سرور API کمی متفاوت است.

لطفاً برای اطلاعات بیشتر به [دسترسی به API از درون یک Pod](/docs/tasks/run-application/access-api-from-pod/) مراجعه کنید.

## دسترسی به خدمات در حال اجرا در خوشه

بخش قبلی توضیح داد که چگونه به سرور API Kubernetes متصل شوید. برای اطلاعات بیشتر در مورد اتصال به خدمات دیگر در حال اجرا در یک خوشه Kubernetes، به [دسترسی به خدمات خوشه](/docs/tasks/access-application-cluster/access-cluster-services/) مراجعه کنید.

## درخواست‌های تغییرمسیر

قابلیت‌های تغییرمسیر قدیمی شده و حذف شده‌اند. لطفاً به جای آن از یک پروکسی (مشاهده زیر) استفاده کنید.

## تعداد زیادی پروکسی

چندین نوع پروکسی مختلفی که ممکن است در استفاده از Kubernetes برخورد کنید:

1. [پروکسی kubectl](#directly-accessing-the-rest-api):

   - در روی دسکتاپ کاربر یا در یک Pod اجرا می‌شود
   - از آدرس localhost به apiserver Kubernetes پروکسی می‌کند
   - مشتری به پروکسی از HTTP استفاده می‌کند
   - پروکسی به apiserver از HTTPS استفاده می‌کند
   - apiserver را یافته و سرورهای احراز هویت را اضافه می‌کند

1. [پروکسی apiserver](/docs/tasks/access-application-cluster/access-cluster-services/#discovering-builtin-services):

   - یک برجسته داخلی است که به apiserver تعبیه شده است
   - یک کاربر خارج از خوشه را به IP‌های خوشه که به طور دیگر قابل دسترسی نیستند، متصل می‌کند
   - در فرآیندهای apiserver اجرا می‌شود
   - مشتری به پروکسی از HTTPS استفاده می‌کند (یا از HTTP اگر apiserver مانند این کانفیگ شده است)
   - پروکسی به هدف ممکن است از HTTP یا HTTPS به انتخاب پروکسی با استفاده از اطلاعات موجود استفاده کند
   - می‌تواند برای رسیدن به یک Node، Pod یا Service استفاده شود
   - هنگام استفاده برای رسیدن به یک Service، بار توازن‌سازی را انجام می‌دهد

1. [پروکسی kube](/docs/concepts/services-networking/service/#ips-and-vips):

   - در هر گره اجرا می‌شود
   - UDP و TCP را پروکسی می‌کند
   - HTTP را درک نمی‌کند
   - بار توازن‌سازی ارائه می‌دهد
   - فقط برای رسیدن به خدمات استفاده می‌شود

1. یک پروکسی/بار توازن‌دهنده در جلوی apiserver(s):

   - وجود و اجرا متفاوت است (مانند nginx)
   - بین تمام مشتری‌ها و یک یا چند apiserver قرار دارد
   - در صورت وجود چندین apiserver به عنو

ان یک بار توازن کار می‌کند

1. بار توازن‌دهنده‌های بار ابری بر روی خدمات خارجی:

   - توسط برخی از ارائه‌دهندگان ابر فراهم می‌شود (مانند AWS ELB، Google Cloud Load Balancer)
   - هنگامی که خدمات Kubernetes دارای نوع `LoadBalancer` هستند، به طور خودکار ایجاد می‌شوند
   - فقط از UDP/TCP استفاده می‌کنند
   - پیاده‌سازی توسط ارائه‌دهنده ابر متفاوت است.

کاربران Kubernetes به طور معمول نیازی به نگرانی در مورد چیزهای دیگر غیر از دو نوع اولیه ندارند. مدیر خوشه به طور معمول اطمینان می‌یابد که انواع دیگر صحیح تنظیم شده باشند.
