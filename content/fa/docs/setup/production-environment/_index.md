---
title: "محیط تولیدی"
description: ایجاد یک کلاستر Kubernetes با کیفیت تولیدی
weight: 30
no_list: true
---
<!-- نمای کلی -->

یک کلاستر Kubernetes با کیفیت تولیدی نیاز به برنامه‌ریزی و آماده‌سازی دارد.
اگر کلاستر Kubernetes شما قرار است بارهای کاری مهمی را اجرا کند، باید به گونه‌ای پیکربندی شود که مقاوم باشد.
این صفحه مراحل راه‌اندازی یک کلاستر آماده برای تولید،
یا ارتقاء یک کلاستر موجود برای استفاده تولیدی را توضیح می‌دهد.
اگر با تنظیمات تولیدی آشنا هستید و فقط به دنبال لینک‌ها هستید، به بخش
[مراحل بعدی](#what-s-next) بروید.

<!-- بدنه -->

## ملاحظات تولیدی

به طور معمول، یک محیط کلاستر Kubernetes تولیدی نیازهای بیشتری نسبت به
یک محیط یادگیری شخصی، توسعه، یا آزمایش دارد. یک محیط تولیدی ممکن است نیاز به
دسترسی امن توسط کاربران متعدد، در دسترس بودن مداوم، و منابعی برای سازگاری
با تقاضاهای متغیر داشته باشد.

هنگام تصمیم‌گیری در مورد محل قرارگیری محیط تولیدی Kubernetes خود
(در محل یا در ابر) و میزان مدیریتی که می‌خواهید به عهده بگیرید
یا به دیگران واگذار کنید، ملاحظات زیر را در نظر بگیرید که چگونه نیازهای شما برای یک کلاستر Kubernetes تحت تأثیر قرار می‌گیرند:

- *دسترسی*: یک [محیط یادگیری](/docs/setup/#learning-environment) Kubernetes با یک ماشین واحد
  یک نقطه شکست دارد. ایجاد یک کلاستر با دسترسی بالا به معنای در نظر گرفتن موارد زیر است:
  - جدا کردن صفحه کنترل از نودهای کاری.
  - تکرار اجزای صفحه کنترل روی نودهای متعدد.
  - متوازن‌سازی بار ترافیک به {{< glossary_tooltip term_id="kube-apiserver" text="سرور API" >}} کلاستر.
  - داشتن تعداد کافی نودهای کاری که در دسترس باشند یا به سرعت بتوانند در دسترس شوند، به دلیل تغییر بارهای کاری.

- *مقیاس*: اگر انتظار دارید محیط Kubernetes تولیدی شما با تقاضای ثابتی مواجه شود،
  ممکن است بتوانید ظرفیت مورد نیاز خود را تنظیم کرده و کار را به پایان برسانید. اما،
  اگر انتظار دارید تقاضا با گذشت زمان افزایش یابد یا به شدت بر اساس عواملی مانند
  فصل یا رویدادهای ویژه تغییر کند، باید برنامه‌ریزی کنید که چگونه مقیاس را برای کاهش فشار
  افزایش درخواست‌ها به صفحه کنترل و نودهای کاری افزایش دهید یا مقیاس را برای کاهش منابع
  استفاده نشده کاهش دهید.

- *امنیت و مدیریت دسترسی*: شما در کلاستر یادگیری Kubernetes خود دسترسی کامل ادمین دارید.
  اما کلاسترهای مشترک با بارهای کاری مهم و بیشتر از یکی دو کاربر، نیاز به رویکردی دقیق‌تر
  برای اینکه چه کسی و چه چیزی می‌تواند به منابع کلاستر دسترسی داشته باشد، دارند.
  می‌توانید از کنترل دسترسی مبتنی بر نقش
  ([RBAC](/docs/reference/access-authn-authz/rbac/)) و سایر
  مکانیزم‌های امنیتی استفاده کنید تا اطمینان حاصل شود که کاربران و بارهای کاری می‌توانند به
  منابع مورد نیاز خود دسترسی پیدا کنند، در حالی که بارهای کاری و خود کلاستر ایمن باقی می‌مانند.
  می‌توانید محدودیت‌هایی برای منابعی که کاربران و بارهای کاری می‌توانند دسترسی داشته باشند
  با مدیریت [سیاست‌ها](/docs/concepts/policy/) و
  [منابع کانتینر](/docs/concepts/configuration/manage-resources-containers/) تنظیم کنید.

قبل از ساخت یک محیط تولیدی Kubernetes به تنهایی، در نظر بگیرید
که برخی یا تمام این کار را به
[راه‌حل‌های ابری آماده](/docs/setup/production-environment/turnkey-solutions/)
یا دیگر [شرکای Kubernetes](/partners/) واگذار کنید.
گزینه‌ها شامل موارد زیر است:

- *بدون سرور*: فقط بارهای کاری را روی تجهیزات شخص ثالث اجرا کنید بدون اینکه کلاستری را مدیریت کنید.
  شما برای مواردی مانند استفاده از CPU، حافظه، و درخواست‌های دیسک هزینه خواهید کرد.
- *صفحه کنترل مدیریت شده*: به ارائه‌دهنده اجازه دهید مقیاس و دسترسی
  صفحه کنترل کلاستر را مدیریت کند، و همچنین وصله‌ها و به‌روزرسانی‌ها را انجام دهد.
- *نودهای کاری مدیریت شده*: استخرهایی از نودها را برای نیازهای خود پیکربندی کنید،
  سپس ارائه‌دهنده مطمئن شود که آن نودها در دسترس هستند و آماده اجرای
  به‌روزرسانی‌ها در صورت نیاز هستند.
- *ادغام*: ارائه‌دهندگانی وجود دارند که Kubernetes را با سایر
  خدماتی که ممکن است نیاز داشته باشید، مانند ذخیره‌سازی، رجیستری کانتینر، روش‌های احراز هویت،
  و ابزارهای توسعه، ادغام می‌کنند.

چه خودتان یک کلاستر Kubernetes تولیدی بسازید و چه با
شرکای همکاری کنید، بخش‌های زیر را بررسی کنید تا نیازهای خود را با توجه به
*صفحه کنترل*، *نودهای کاری*، *دسترسی کاربر*، و
*منابع بار کاری* ارزیابی کنید.

## راه‌اندازی کلاستر تولیدی

در یک کلاستر Kubernetes با کیفیت تولیدی، صفحه کنترل
کلاستر را از سرویس‌هایی مدیریت می‌کند که می‌توانند به صورت گسترده در چندین کامپیوتر
در راه‌های مختلف توزیع شوند. هر نود کاری، با این حال، یک موجودیت واحد را نشان می‌دهد که
برای اجرای پادهای Kubernetes پیکربندی شده است.

### صفحه کنترل تولیدی

ساده‌ترین کلاستر Kubernetes شامل کل صفحه کنترل و سرویس‌های نود کاری
در حال اجرا بر روی یک ماشین واحد است. می‌توانید آن محیط را با اضافه کردن
نودهای کاری گسترش دهید، همانطور که در دیاگرام موجود در
[اجزای Kubernetes](/docs/concepts/overview/components/) نشان داده شده است.
اگر کلاستر قرار است برای مدت کوتاهی در دسترس باشد، یا می‌تواند
در صورت بروز مشکل جدی دور ریخته شود، این ممکن است نیازهای شما را برآورده کند.

با این حال، اگر به یک کلاستر دائمی و با دسترسی بالا نیاز دارید، باید
راه‌های گسترش صفحه کنترل را در نظر بگیرید. به طور طراحی، سرویس‌های
صفحه کنترل یک ماشینی که بر روی یک ماشین واحد اجرا می‌شوند، دسترسی بالایی ندارند.
اگر حفظ کلاستر و اطمینان از اینکه می‌توان آن را در صورت بروز مشکل تعمیر کرد مهم است،
مراحل زیر را در نظر بگیرید:

- *انتخاب ابزارهای استقرار*: می‌توانید صفحه کنترل را با استفاده از ابزارهایی مانند
  kubeadm، kops، و kubespray استقرار دهید. برای یادگیری نکاتی برای استقرارهای
  با کیفیت تولیدی با استفاده از هر یک از روش‌های استقرار، به
  [نصب Kubernetes با ابزارهای استقرار](/docs/setup/production-environment/tools/)
  مراجعه کنید. [ران‌تایم‌های کانتینری](/docs/setup/production-environment/container-runtimes/)
  مختلفی برای استفاده با استقرارهای شما موجود هستند.
- *مدیریت گواهی‌ها*: ارتباطات امن بین سرویس‌های صفحه کنترل
  با استفاده از گواهی‌ها پیاده‌سازی می‌شوند. گواهی‌ها به صورت خودکار در طول استقرار
  تولید می‌شوند یا می‌توانید آن‌ها را با استفاده از مرجع گواهی خود تولید کنید.
  برای جزئیات بیشتر به [گواهی‌های PKI و نیازها](/docs/setup/best-practices/certificates/) مراجعه کنید.
- *پیکربندی بار متوازن‌کننده برای apiserver*: یک بار متوازن‌کننده پیکربندی کنید
  تا درخواست‌های API خارجی را به نمونه‌های سرویس apiserver که بر روی نودهای مختلف اجرا می‌شوند، توزیع کند. برای جزئیات بیشتر به
  [ایجاد یک بار متوازن‌کننده خارجی](/docs/tasks/access-application-cluster/create-external-load-balancer/)
  مراجعه کنید.
- *جداسازی و پشتیبان‌گیری از سرویس etcd*: سرویس‌های etcd می‌توانند یا بر روی
  همان ماشین‌های سرویس‌های دیگر صفحه کنترل یا بر روی ماشین‌های جداگانه برای
  امنیت و دسترسی بیشتر اجرا شوند. به دلیل اینکه etcd داده‌های پیکربندی کلاستر را ذخیره می‌کند،
  پشتیبان‌گیری از پایگاه داده etcd باید به طور منظم انجام شود تا اطمینان حاصل شود که می‌توانید
  در صورت نیاز آن پایگاه داده را تعمیر کنید.
  برای جزئیات بیشتر در مورد پیکربندی و استفاده از etcd به [سوالات متداول etcd](https://etcd.io/docs/v3.5/faq/) مراجعه کنید.
  برای جزئیات بیشتر به [اجرای کلاسترهای etcd برای Kubernetes](/docs/tasks/administer-cluster/configure-upgrade-etcd/)
  و [راه‌اندازی یک کلاستر etcd با دسترسی بالا با kubeadm](/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/)
  مراجعه کنید.
- *ایجاد چندین سیستم صفحه کنترل*: برای دسترسی بالا،
  صفحه کنترل نباید محدود به یک ماشین باشد. اگر سرویس‌های صفحه کنترل
  توسط یک سرویس init (مانند systemd) اجرا شوند، هر سرویس باید بر روی حداقل سه ماشین اجرا شود.
  با این حال، اجرای سرویس‌های صفحه کنترل به عنوان پاد در
  Kubernetes اطمینان می‌دهد که تعداد تکراری سرویس‌هایی که درخواست کرده‌اید
  همیشه در دسترس خواهد بود.
  زمان‌بندی باید مقاوم در برابر خطا باشد،
  اما نه با دسترسی بالا. برخی از ابزارهای استقرار الگوریتم اجماع [Raft](https://raft.github.io/)
  را برای انتخاب رهبر سرویس‌های Kubernetes تنظیم می‌کنند. اگر
  سرویس اصلی از بین برود، سرویس دیگری خود را انتخاب می‌کند و مسئولیت را بر عهده می‌گیرد.
- *پوشش دادن چندین منطقه*: اگر حفظ کلاستر شما در تمام زمان‌ها
  حیاتی است، در نظر بگیرید که یک کلاستر ایجاد کنید که در چندین مرکز داده اجرا شود،
  که در محیط‌های ابری به عنوان مناطق شناخته می‌شوند. گروه‌هایی از مناطق به عنوان نواحی شناخته می‌شوند.
  با گسترش یک کلاستر در
  چندین منطقه در همان ناحیه، می‌توان احتمال اینکه کلاستر شما
  به کار خود ادامه دهد حتی اگر یک منطقه غیرقابل دسترس شود را افزایش داد.
  برای جزئیات بیشتر به [اجرای در چندین منطقه](/docs/setup/best-practices/multiple-zones/) مراجعه کنید.
- *مدیریت ویژگی‌های مستمر*: اگر قصد دارید کلاستر خود را در طول زمان نگه دارید،
  وظایفی وجود دارند که باید انجام دهید تا سلامت و امنیت آن را حفظ کنید. به عنوان مثال،
  اگر با kubeadm نصب کرده‌اید، دستورالعمل‌هایی برای کمک به شما در
  [مدیریت گواهی](/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)
  و [ارتقاء کلاسترهای kubeadm](/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
  وجود دارد. برای لیست بلندتری از وظایف مدیریت Kubernetes به
  [مدیریت کلاستر](/docs/tasks/administer-cluster/)
  مراجعه کنید.

برای یادگیری درباره گزینه‌های موجود هنگام اجرای سرویس‌های صفحه کنترل، به صفحات اجزای
[kube-apiserver](/docs/reference/command-line-tools-reference/kube-apiserver/)،
[kube-controller-manager](/docs/reference/command-line-tools-reference/kube-controller-manager/)،
و [kube-scheduler](/docs/reference/command-line-tools-reference/kube-scheduler/)
مراجعه کنید. برای مثال‌های دسترسی بالا برای صفحه کنترل، به
[گزینه‌های توپولوژی با دسترسی بالا](/docs/setup/production-environment/tools/kubeadm/ha-topology/)،
[ایجاد کلاسترهای با دسترسی بالا با kubeadm](/docs/setup/production-environment/tools/kubeadm/high-availability/)،
و [اجرای کلاسترهای etcd برای Kubernetes](/docs/tasks/administer-cluster/configure-upgrade-etcd/)
مراجعه کنید. برای اطلاعات در مورد برنامه‌ریزی پشتیبان‌گیری etcd به
[پشتیبان‌گیری از یک کلاستر etcd](/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)
مراجعه کنید.
### گره‌های کارگر تولیدی

بارهای کاری با کیفیت تولیدی باید مقاوم باشند و هر چیزی که به آنها وابسته است نیز باید مقاوم باشد (مانند CoreDNS). بگیرید کنترل خود را یا یک ارائه‌دهنده ابری آن را برای شما انجام دهد، همچنان باید در نظر داشته باشید که چگونه می‌خواهید گره‌های کارگر خود را مدیریت کنید (که به سادگی با نام *گره‌ها* هم اشاره می‌شود).

- *پیکربندی گره‌ها*: گره‌ها می‌توانند ماشین‌های فیزیکی یا مجازی باشند. اگر می‌خواهید گره‌های خود را ایجاد و مدیریت کنید، می‌توانید یک سیستم عامل پشتیبانی شده نصب کنید، سپس سرویس‌های مربوطه
  [سرویس‌های گره](/docs/concepts/overview/components/#node-components)
  را اضافه و اجرا کنید. در نظر بگیرید:
  - نیازهای بار کاری خود را هنگام تنظیم گره‌ها با داشتن حافظه، پردازنده و ظرفیت ذخیره سرعت و دیسک مناسب در نظر بگیرید.
  - آیا سیستم‌های کامپیوتر عمومی کافی هستند یا آیا بار کاری‌هایی دارید که نیاز به پردازنده‌های GPU، گره‌های ویندوز یا جداسازی VM دارند.

- *اعتبار سنجی گره‌ها*: برای اطمینان از اینکه یک گره شرایط برای پیوستن به یک کلاستر Kubernetes را دارد، به
  [راهنمای تنظیم گره معتبر](/docs/setup/best-practices/node-conformance/)
  مراجعه کنید.

- *اضافه کردن گره‌ها به کلاستر*: اگر دارید کلاستر خود را مدیریت می‌کنید می‌توانید
  گره‌ها را با نصب دستگاه‌های خود تنظیم کرده و یا به صورت دستی به آنها اضافه کنید
  و یا آنها را به طور خودکار به apiserver کلاستر ثبت کنید. برای اطلاعات بیشتر در مورد چگونگی تنظیم Kubernetes برای اضافه کردن گره‌ها به این روش‌ها به بخش
  [گره‌ها](/docs/concepts/architecture/nodes/)
  مراجعه کنید.

- *مقیاس‌پذیری گره‌ها*: برنامه‌ریزی برای گسترش ظرفیتی که کلاستر شما در نهایت نیاز دارد، را داشته باشید. برای کمک به تعیین تعداد گره‌های لازم بر اساس تعداد pods و containersی که نیاز دارید، به
  [ملاحظات برای کلاسترهای بزرگ](/docs/setup/best-practices/cluster-large/)
  مراجعه کنید. اگر گره‌ها را خودتان مدیریت می‌کنید، این می‌تواند به این معنی باشد که تجهیزات فیزیکی خود را خریداری و نصب کنید.

- *اتوماسیون مقیاس گره‌ها*: برای مدیریت خودکار گره‌های خود و ظرفیتی که ارائه می‌دهند، به [اتوماسیون مقیاس کلاستر](/docs/concepts/cluster-administration/cluster-autoscaling)
  مراجعه کنید.

- *راه‌اندازی چک‌های سلامت گره‌ها*: برای بارهای کاری مهم، مطمئن شوید که گره‌ها و podsی که بر روی آنها اجرا می‌شوند، سالم هستند. با استفاده از دیمون [تشخیص مشکل گره](/docs/tasks/debug/debug-cluster/monitor-node-health/)
  می‌توانید اطمینان حاصل کنید که گره‌های خود سالم هستند.

## مدیریت کاربر تولیدی

در محیط تولیدی، ممکن است از مدلی که شما یا یک گروه کوچک از افراد به کلاستر دسترسی داشته باشند به یک مدلی که ممکن است ده‌ها یا صدها نفر از آن استفاده کنند، حرکت کنید. در یک محیط یادگیری یا نمونه کارگاه، ممکن است یک حساب مدیریتی تنها برای همه کارهای شما وجود داشته باشد. در تولید، شما می‌خواهید حساب‌های بیشتری با سطوح دسترسی مختلف به فضاهای مختلف ایجاد کنید.

استفاده از یک کلاستر کیفیت تولیدی به این معنی است که تصمیم می‌گیرید چگونه می‌خواهید دسترسی را به صورت انتخابی به کاربران دیگر اجازه دهید. به خصوص، شما باید استراتژی‌ها را برای اعتبارسنجی هویت آن‌ها که تلاش می‌کنند به کلاستر شما دسترسی پیدا کنند (احراز هویت) و تصمیم بگیرید که آیا آنها دسترسی برای انجام آنچه که درخواست می‌کنند دارند یا نه (مجوز):

- *احراز هویت*: apiserver می‌تواند کاربران را با استفاده از گواهینامه‌های مشتری، توکن‌های برنده، یک پروکسی احراز هویت یا احراز هویت اساسی HTTP احراز هویت کند. شما می‌توانید روش‌های احراز هویت را که می‌خواهید استفاده کنید را انتخاب کنید. با استفاده از افزونه‌ها، apiserver می‌تواند از روش‌های احراز هویت موجود سازمان شما مانند LDAP یا Kerberos استفاده کند. به [احراز هویت](/docs/reference/access-authn-authz/authentication/)
  مراجعه کنید برای توضیح این روش‌های مختلف احراز هویت کاربران Kubernetes.

- *مجوزدهی*: زمانی که شما برای مجوزدهی به کاربران عادی خود شروع به کار می‌کنید، احتمالاً بین مجوز RBAC و ABAC تصمیم می‌گیرید. به [بررسی مجوزها](/docs/reference/access-authn-authz/authorization/)
  برای مشاهده حالات مختلف برای دسترسی به حساب‌های کاربری (و همچنین دسترسی حساب‌های سرویس به کلاستر شما) توجه کنید:

  - *کنترل دسترسی مبتنی بر نقش* ([RBAC](/docs/reference/access-authn-authz/rbac/)): به شما اجازه می‌دهد دسترسی به کلاستر خود را با اجازه‌های مشخص برای کاربران احراز هویت شده اختصاص دهید. اجازه‌ها می‌تواند برای یک فضای نام مشخص (نقش) یا در سراسر کلاستر (ClusterRole) اختصاص داده شود. سپس با استفاده از RoleBindings و ClusterRoleBindings، این اجازه‌ها می‌توانند به کاربران خاص ارتباط داده شوند.

  - *کنترل دسترسی مبتنی بر ویژگی* ([ABAC](/docs/reference/access-authn-authz/abac/)): به شما اجازه می‌دهد سیاست‌هایی بر اساس ویژگی‌های منبع در کلاستر ایجاد کنید و بر اساس آن ویژگی‌ها دسترسی را اجازه یا امتناع دهید. هر خط از یک فایل سیاست نسخه‌بندی خاصی (apiVersion و kind) و نقشه‌ای از خصوصیات مشخصات را برای تطابق با موضوع (کاربر یا گروه)، خصوصیت منبع، خصوصیت غیر منبع (/version یا /apis) و فقط خواندن مشخص می‌کند. برای کسب اطلاعات بیشتر، به [مثال‌ها](/docs/reference/access-authn-authz/abac/#examples) مراجعه کنید.

به عنوان کسی که در حال تنظیم احراز هویت و مجوزدهی بر روی کلاستر Kubernetes تولیدی خود هستید، چندین مورد را در نظر داشته باشید:

- *تنظیم حالت مجوزدهی*: زمانی که apiserver Kubernetes
  ([kube-apiserver](/docs/reference/command-line-tools-reference/kube-apiserver/))
  شروع به کار می‌کند، حالت‌های احراز هویت پشتیبانی شده باید با استفاده از پرچم *--authorization-mode*
  تنظیم شود. به عنوان مثال، این پرچم در فایل *kube-adminserver.yaml* (در */etc/kubernetes/manifests*)
  می‌تواند به صورت Node,RBAC تنظیم شود. این امر به Node و RBAC اجازه می‌دهد تا برای درخواست‌های احراز هویت شده استفاده شوند.

- *ایجاد گواهینامه‌های کاربر و پیوندهای نقش (RBAC)*: اگر از احراز هویت RBAC
  استفاده می‌کنید، کاربران می‌توانند یک CertificateSigningRequest (CSR) ایجاد کنند که می‌تواند
  توسط CA کلاستر امضا شود. سپس می‌توانید نقش‌ها و ClusterRoles را به هر کاربر اختصاص دهید.
  برای جزئیات بیشتر به [درخواست‌های امضای گواهینامه](/docs/reference/access-authn-authz/certificate-signing-requests/)
  مراجعه کنید.

- *ایجاد سیاست‌هایی که ویژگی‌ها را ترکیب می‌کنند (ABAC)*: اگر از احراز هویت ABAC
  استفاده می‌کنید، می‌توانید ترکیبی از ویژگی‌ها را برای ایجاد سیاست‌ها اختصاص دهید
  اجازه دسترسی به کاربران یا گروه‌های انتخاب شده به منابع خاص (مانند یک
  pod)، فضای نام یا apiGroup. برای کسب اطلاعات بیشتر، به
  [مثال‌ها](/docs/reference/access-authn-authz/abac/#examples) مراجعه کنید.

- *در نظر گرفتن کنترل‌کننده‌های پذیرش*: اشکالات اضافی از اجازه‌دهی برای
  درخواست‌هایی که ممکن است از طریق apiserver بیاید شامل
  [احراز هویت توکن Webhook](/docs/reference/access-authn-authz/authentication/#webhook-token-authentication).
  وبهوک‌ها و انواع احراز هویت ویژه دیگر نیاز به فعال‌سازی از طریق افزودنی‌های
  [کنترل‌کننده‌های پذیرش](/docs/reference/access-authn-authz/admission-controllers/)
  به apiserver دارند.

## تعیین محدودیت‌ها بر روی منابع بار کاری

نیازهای بار کاری تولیدی می‌توانند فشار را هم درون و هم خارج از صفحه کنترل Kubernetes ایجاد کنند. این موارد را در نظر بگیرید وقتی که برای
نیازهای بار کاری خوشه خود تنظیم می‌کنید:

- *تنظیم محدودیت فضای نام*: محدودیت‌هایی برای هر فضای نام بر روی چیزهایی مانند حافظه و CPU تعیین کنید. برای کسب اطلاعات بیشتر به
  [مدیریت حافظه، CPU و منابع API](/docs/tasks/administer-cluster/manage-resources/)
  مراجعه کنید. همچنین می‌توانید
  [فضای نام‌های سلسله مراتبی](/blog/2020/08/14/introducing-hierarchical-namespaces/)
  را برای به ارث بردن محدودیت‌ها تنظیم کنید.

- *آماده‌سازی برای تقاضای DNS*: اگر انتظار دارید که بار کاری‌ها به طور گسترده ای بالا برود،
  سرویس DNS شما باید آماده برای بالا بردن مقیاس باشد. برای کسب اطلاعات بیشتر مراجعه کنید
  [خودکارسازی سرویس DNS در یک خوشه](/docs/tasks/administer-cluster/dns-horizontal-autoscaling/).

- *ایجاد حساب‌های سرویس اضافی*: حساب‌های کاربری تعیین می‌کنند که کاربران چه کارهایی در یک خوشه انجام می‌دهند، در حالی که یک حساب سرویس
  دسترسی پاد به داخل یک فضای نام خاص را تعریف می‌کند. به طور پیش‌فرض، یک پاد از حساب سرویس پیش‌فرض فضای نام خود استفاده می‌کند.
  به [مدیریت حساب‌های سرویس](/docs/reference/access-authn-authz/service-accounts-admin/)
  برای کسب اطلاعات بیشتر در مورد ایجاد حساب سرویس مراجعه کنید. به عنوان مثال، شما ممکن است بخواهید:

  - اسراری را که یک پاد می‌تواند برای بلوک تصاویر از یک ثبت‌کننده مخصوص استفاده کند، اضافه کنید. به
    [پیکربندی حساب‌های سرویس برای پادها](/docs/tasks/configure-pod-container/configure-service-account/)
    برای مثال.

  - اجازه‌های RBAC را به یک حساب سرویس اختصاص دهید. به
    [اجازه‌های حساب سرویس](/docs/reference/access-authn-authz/rbac/#service-account-permissions)
    برای جزئیات مراجعه کنید.

## {{% عنوان "whatsnext" %}}

- تصمیم بگیرید که آیا می‌خواهید خوشه Kubernetes تولیدی خود را ایجاد کنید یا از
  [روش‌های ارائه نمای چرخه ای](/docs/setup/production-environment/turnkey-solutions/)
  یا [شرکای Kubernetes](/partners/)
  در دسترس استفاده کنید.

- اگر تصمیم به ایجاد خوشه خود کرده‌اید، برنامه‌ریزی کنید که چطور می‌خواهید
  با [گواهینامه‌ها](/docs/setup/best-practices/certificates/)
  و تنظیم پایداری برای ویژگی‌هایی مانند
  [etcd](/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/)
  و
  [apiserver](/docs/setup/production-environment/tools/kubeadm/ha-topology/)
  انجام دهید.

- از بین [kubeadm](/docs/setup/production-environment/tools/kubeadm/),
  [kops](https://kops.sigs.k8s.io/) یا
  [Kubespray](https://kubespray.io/) روش‌های ارائه نمای چرخه ای را انتخاب کنید.

- با [مدیریت کاربران](/docs/reference/access-authn-authz/authentication/) خود را تنظیم کنید
  و [اجازه‌ها](/docs/reference/access-authn-authz/authorization/) خود را.

- برای بار کاری‌های برنامه آماده باشید با تنظیم
  [محدودیت‌های منابع](/docs/tasks/administer-cluster/manage-resources/)،
  [خودکارسازی DNS](/docs/tasks/administer-cluster/dns-horizontal-autoscaling/)
  و [حساب‌های سرویس](/docs/reference/access-authn-authz/service-accounts-admin/).
