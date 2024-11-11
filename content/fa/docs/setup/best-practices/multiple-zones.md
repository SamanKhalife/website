---
reviewers:
- jlowdermilk
- justinsb
- quinton-hoole
title: اجرا در چند منطقه
weight: 20
content_type: concept
---

<!-- overview -->

این صفحه به اجرای Kubernetes در چند منطقه می‌پردازد.

<!-- body -->

## پس زمینه

Kubernetes به گونه‌ای طراحی شده است که یک کلاستر Kubernetes می‌تواند در چندین منطقه خرابی اجرا شود، به طور معمول این مناطق در یک گروه منطقی به نام _منطقه_ قرار می‌گیرند. ارائه‌دهندگان بزرگ ابر یک منطقه را به عنوان مجموعه‌ای از مناطق خرابی (که به آنها _مناطق دسترسی_ نیز گفته می‌شود) تعریف می‌کنند که مجموعه‌ای یکسان از ویژگی‌ها را ارائه می‌دهند: در یک منطقه، هر منطقه یکسان APIها و خدمات را ارائه می‌دهد.

معماری‌های ابری معمولی هدف کاهش احتمال آنکه یک خرابی در یک منطقه به خدمات در منطقه دیگر نیز آسیب برساند را دارند.

## رفتار صفحه کنترل

تمام [اجزای صفحه کنترل](/docs/concepts/overview/components/#control-plane-components)
از اجرای به عنوان یک مجموعه منابع قابل تعویض پشتیبانی می‌کنند که برای هر جزء تکرار شده‌اند.

هنگامی که یک صفحه کنترل کلاستر را مستقر می‌کنید، نسخه‌های تکراری اجزای صفحه کنترل را در چندین منطقه خرابی قرار دهید. اگر دسترس‌پذیری یک نگرانی مهم است، حداقل سه منطقه خرابی را انتخاب کرده و هر جزء فردی صفحه کنترل (سرور API، زمانبندی‌کننده، etcd، مدیر کنترل کلاستر) را در حداقل سه منطقه خرابی تکرار کنید. اگر یک مدیر کنترل ابر اجرا می‌کنید، باید آن را نیز در تمام مناطق خرابی انتخاب شده تکرار کنید.

{{< note >}}
Kubernetes برای نقاط انتهایی سرور API تاب‌آوری بین مناطق را فراهم نمی‌کند. شما می‌توانید از تکنیک‌های مختلفی برای بهبود دسترس‌پذیری برای سرور API کلاستر استفاده کنید، از جمله DNS round-robin، رکوردهای SRV، یا یک راه‌حل بار متوازن‌کننده شخص ثالث با بررسی سلامت.
{{< /note >}}

## رفتار نودها

Kubernetes به طور خودکار پادها برای منابع بار کاری (مانند {{< glossary_tooltip text="Deployment" term_id="deployment" >}} یا {{< glossary_tooltip text="StatefulSet" term_id="statefulset" >}}) را در نودهای مختلف یک کلاستر پراکنده می‌کند. این پراکندگی به کاهش تاثیر خرابی‌ها کمک می‌کند.

هنگامی که نودها راه‌اندازی می‌شوند، kubelet در هر نود به طور خودکار
{{< glossary_tooltip text="برچسب‌ها" term_id="label" >}} را به شیء نود که نشان‌دهنده آن kubelet خاص در API Kubernetes است اضافه می‌کند. این برچسب‌ها می‌توانند شامل
[اطلاعات منطقه](/docs/reference/labels-annotations-taints/#topologykubernetesiozone) باشند.

اگر کلاستر شما در چندین منطقه یا ناحیه گسترده شده است، می‌توانید از برچسب‌های نود در کنار
[محدودیت‌های پراکندگی توپولوژی پادها](/docs/concepts/scheduling-eviction/topology-spread-constraints/)
برای کنترل نحوه پراکندگی پادها در کلاستر خود در میان حوزه‌های خرابی: مناطق، نواحی و حتی نودهای خاص استفاده کنید. این نکات به
{{< glossary_tooltip text="زمانبندی‌کننده" term_id="kube-scheduler" >}} کمک می‌کنند که پادها را برای دسترس‌پذیری بهتر مورد انتظار قرار دهد و ریسک آنکه یک خرابی مرتبط کل بار کاری شما را تحت تاثیر قرار دهد را کاهش دهد.

به عنوان مثال، می‌توانید محدودیتی را تعیین کنید تا مطمئن شوید که
3 نسخه از یک StatefulSet هر یک در مناطق مختلف اجرا می‌شوند، هر زمان که امکان‌پذیر باشد. می‌توانید این را به صورت اعلانی تعریف کنید بدون اینکه به طور صریح تعیین کنید که کدام مناطق دسترسی برای هر بار کاری استفاده می‌شود.

### توزیع نودها در مناطق

هسته Kubernetes نودها را برای شما ایجاد نمی‌کند؛ شما باید این کار را خودتان انجام دهید، یا از ابزاری مانند [Cluster API](https://cluster-api.sigs.k8s.io/) برای مدیریت نودها به نمایندگی از خود استفاده کنید.

با استفاده از ابزارهایی مانند Cluster API می‌توانید مجموعه‌ای از ماشین‌ها را برای اجرای به عنوان نودهای کارگر برای کلاستر خود در چندین حوزه خرابی تعریف کنید و قوانینی برای بهبود خودکار کلاستر در صورت اختلال خدمات در کل منطقه تعیین کنید.

## تخصیص دستی منطقه برای پادها

می‌توانید [محدودیت‌های انتخابگر نود](/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector) را به پادهایی که ایجاد می‌کنید اعمال کنید، همچنین به الگوهای پاد در منابع بار کاری مانند Deployment، StatefulSet یا Job.

## دسترسی به ذخیره‌سازی برای مناطق

هنگامی که حجم‌های پایدار ایجاد می‌شوند، Kubernetes به طور خودکار برچسب‌های منطقه را به هر PersistentVolumes که به یک منطقه خاص مرتبط هستند اضافه می‌کند.
{{< glossary_tooltip text="زمانبندی‌کننده" term_id="kube-scheduler" >}} سپس اطمینان می‌دهد،
از طریق پیش‌بینی `NoVolumeZoneConflict`، که پادهایی که یک PersistentVolume معین را مطالبه می‌کنند فقط در همان منطقه‌ای که آن حجم قرار دارد مستقر می‌شوند.

لطفا توجه داشته باشید که روش افزودن برچسب‌های منطقه می‌تواند به ارائه‌دهنده ابری شما و پروویژنینگ ذخیره‌سازی که استفاده می‌کنید بستگی داشته باشد. همیشه به مستندات خاص محیط خود مراجعه کنید تا از پیکربندی صحیح اطمینان حاصل کنید.

می‌توانید یک {{< glossary_tooltip text="کلاس ذخیره‌سازی" term_id="storage-class" >}}
برای PersistentVolumeClaims مشخص کنید که دامنه‌های خرابی (مناطق) را که ذخیره‌سازی در آن کلاس ممکن است استفاده شود مشخص می‌کند.
برای یادگیری درباره پیکربندی یک کلاس ذخیره‌سازی که از دامنه‌های خرابی یا مناطق آگاه است، به [توپولوژی‌های مجاز](/docs/concepts/storage/storage-classes/#allowed-topologies) مراجعه کنید.

## شبکه

به خودی خود، Kubernetes شامل شبکه‌ای آگاه از منطقه نیست. می‌توانید از یک
[پلاگین شبکه](/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)
برای پیکربندی شبکه کلاستر استفاده کنید، و آن راه‌حل شبکه ممکن است عناصر خاص منطقه داشته باشد. به عنوان مثال، اگر ارائه‌دهنده ابری شما از سرویس‌هایی با
`type=LoadBalancer` پشتیبانی می‌کند، ممکن است تعادل بار فقط ترافیک را به پادهایی که در همان منطقه با عنصر تعادل بار که یک اتصال معین را پردازش می‌کند ارسال کند.
مستندات ارائه‌دهنده ابری خود را برای جزئیات بررسی کنید.

برای استقرارهای سفارشی یا درون‌سازمانی، ملاحظات مشابهی اعمال می‌شود.
{{< glossary_tooltip text="سرویس" term_id="service" >}} و
{{< glossary_tooltip text="ورودی" term_id="ingress" >}} رفتار، از جمله برخورد با مناطق خرابی مختلف، بستگی به نحوه تنظیم دقیق کلاستر شما دارد.

## بازیابی خرابی

هنگامی که کلاستر خود را تنظیم می‌کنید، ممکن است نیاز به در نظر گرفتن اینکه آیا و چگونه تنظیمات شما می‌تواند خدمات را در صورتی که تمام مناطق خرابی در یک ناحیه به طور همزمان از خط خارج شوند بازگرداند. به عنوان مثال، آیا به وجود حداقل یک نود قادر به اجرای پادها در یک منطقه متکی هستید؟  
مطمئن شوید که هر گونه کار تعمیر کلاستر مهم به وجود حداقل یک نود سالم در کلاستر شما وابسته نیست. به عنوان مثال: اگر همه نودها ناسالم هستند، ممکن است نیاز به اجرای یک کار تعمیر با یک
{{< glossary_tooltip text="تحمل" term_id="toleration" >}} خاص داشته باشید تا تعمیر به اندازه کافی کامل شود تا حداقل یک نود به خدمت بازگردد.

Kubernetes برای این چالش پاسخی ندارد؛ با این حال، این چیزی است که باید در نظر گرفت.

## {{% heading "whatsnext" %}}

برای یادگیری اینکه چگونه زمانبندی‌کننده پادها را در یک کلاستر مستقر می‌کند و محدودیت‌های پیکربندی شده را رعایت می‌کند، به [زمانبندی و اخراج](/docs/concepts/scheduling-eviction/) مراجعه کنید.