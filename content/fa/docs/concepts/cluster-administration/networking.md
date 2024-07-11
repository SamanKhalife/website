---
reviewers:
- thockin
title: شبکه‌های خوشه
content_type: مفهوم
weight: 50
---

<!-- مرور -->
شبکه‌بندی یک بخش مرکزی از Kubernetes است، اما ممکن است برای درک دقیق کارکردهای آن چالش‌هایی وجود داشته باشد. در اینجا ۴ مشکل شبکه مجزا برای حل و فصل وجود دارد:

1. ارتباطات نرمال به نرمال بین کانتینرها: این مشکل با استفاده از {{< glossary_tooltip text="Pods" term_id="pod" >}} و ارتباطات `localhost` حل می‌شود.
2. ارتباطات Pod به Pod: این مسئله اصلی این سند است.
3. ارتباطات Pod به سرویس: این مورد توسط [سرویس‌ها](/docs/concepts/services-networking/service/) پوشش داده می‌شود.
4. ارتباطات خارجی به سرویس: این نیز توسط سرویس‌ها پوشش داده می‌شود.

<!-- متن -->

Kubernetes درباره‌ی به اشتراک‌گذاری ماشین‌ها بین برنامه‌ها است. به طور معمول، به اشتراک‌گذاری ماشین‌ها نیاز دارد که اطمینان حاصل شود دو برنامه سعی نکنند از پورت‌های یکسانی استفاده کنند. هماهنگ کردن پورت‌ها در مقیاس بزرگ بسیار دشوار است و کاربران را به مسائل سطح خوشه که خارج از کنترل آن‌ها است فرا می‌گیرد.

تخصیص پویا پورت‌ها به سیستم بسیار پیچیدگی‌ها را به سیستم تحمیل می‌کند - هر برنامه باید پورت‌ها را به عنوان پرچم در نظر بگیرد، سرورهای API باید بدانند چگونه اعداد پورت پویا را در بلوک‌های پیکربندی قرار دهند، سرویس‌ها باید بدانند چگونه یکدیگر را پیدا کنند، و غیره. به جای این کار، Kubernetes به رویکردی متفاوت می‌پردازد.

برای یادگیری در مورد مدل شبکه Kubernetes، [اینجا را](/docs/concepts/services-networking/) ببینید.

## محدوده‌های آدرس IP Kubernetes

خوشه‌های Kubernetes نیاز به تخصیص آدرس‌های IP غیر همپوشانی برای Pods، Services و Nodes دارند، از مجموعه‌ای از آدرس‌های موجود پیکربندی شده در مؤلفه‌های زیر:

- پلاگین شبکه برای اختصاص آدرس‌های IP به Pods پیکربندی می‌شود.
- kube-apiserver برای اختصاص آدرس‌های IP به Services پیکربندی می‌شود.
- kubelet یا cloud-controller-manager برای اختصاص آدرس‌های IP به Nodes پیکربندی می‌شود.

{{< figure src="/docs/images/kubernetes-cluster-network.svg" alt="نموداری که محدوده‌های شبکه مختلف را در یک خوشه Kubernetes نشان می‌دهد" class="diagram-medium" >}}

## انواع شبکه‌بندی خوشه {#cluster-network-ipfamilies}

خوشه‌های Kubernetes، با توجه به خانواده‌های IP پیکربندی شده، می‌توانند در دسته‌های زیر دسته‌بندی شوند:

- فقط IPv4: پلاگین شبکه، kube-apiserver و kubelet/cloud-controller-manager برای اختصاص فقط آدرس‌های IPv4 پیکربندی شده‌اند.
- فقط IPv6: پلاگین شبکه، kube-apiserver و kubelet/cloud-controller-manager برای اختصاص فقط آدرس‌های IPv6 پیکربندی شده‌اند.
- IPv4/IPv6 یا IPv6/IPv4 [دوگانه‌استک](/docs/concepts/services-networking/dual-stack/):
  - پلاگین شبکه برای اختصاص آدرس‌های IPv4 و IPv6 پیکربندی شده است.
  - kube-apiserver برای اختصاص آدرس‌های IPv4 و IPv6 پیکربندی شده است.
  - kubelet یا cloud-controller-manager برای اختصاص آدرس‌های IPv4 و IPv6 پیکربندی شده‌اند.
  - تمام اجزاء باید با یکدیگر درباره‌ی خانواده IP اولیه پیکربندی موافقت کنند.

خوشه‌های Kubernetes تنها خانواده‌های IP حاضر در اشیاء Pods، Services و Nodes را مد نظر دارند، مستقل از IPهای موجود اشیاء نماینده. به عنوان مثال، یک سرور یا یک pod می‌تواند چندین آدرس IP را در رابط‌های خود داشته باشد، اما فقط آدرس‌های IP در `node.status.addresses` یا `pod.status.ips` برای اجرای مدل شبکه Kubernetes و تعریف نوع خوشه مد نظر هستند.

## چگونگی پیاده‌سازی مدل شبکه Kubernetes

مدل شبکه توسط محیط اجرایی کانتینر بر روی هر گره پیاده‌سازی می‌شود. محیط‌های اجرایی کانتینر معمول‌ترین از رابط شبکه و قابلیت‌های امنیتی خود استفاده می‌کنند [ابزارهای رابط شبکه کانتینر](https://github.com/containernetworking/cni) (CNI) برای مدیریت شبکه و قابلیت‌های امنیتی خود. از این ابزارهای مختلف CNI بسیاری از ارائه‌دهندگان مختلف وجود دارد. برخی از این ابزارها فقط ویژگی‌های پایه‌ای اضافه و حذف رابط‌های شبکه را فراهم می‌کنند، در حالی که دیگران راه‌حل‌های پیچیده‌تری را ارائه می‌دهند، مانند ادغام با سیستم‌های ارکان کانتینر دیگر، اجرای چندین ابزار CNI، ویژگی‌های پیشرفته IPAM و غیره.

برای دیدن [این صفحه](/docs/concepts/cluster-administration/addons/#networking-and-network-policy) لیست غیرمتمرکزی از افزونه‌های شبکه پشتیبانی شده توسط Kubernetes.

## {{% heading "whatsnext" %}}

طراحی اولیه مدل شبکه و عقلانیت آن در جزئیات بیشتر در [سند طراحی شبکه](https://git.k8s.io/design-proposals-archive/network/networking.md) توضیح داده شده است. برای طرح‌های آینده و تلاش‌هایی که به بهبود شبکه Kubernetes اشاره دارند، لطفاً به KEPs SIG-Network
[KEPs](https://github.com/kubernetes/enhancements/tree/master/keps/sig-network) مراجعه کنید.