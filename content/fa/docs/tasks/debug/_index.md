---
title: "Monitoring, Logging, and Debugging"
description: تنظیم مانیتورینگ و لاگ‌گیری برای رفع اشکال یک کلاستر، یا دیباگ کردن یک برنامه کانتینریز شده.
weight: 40
reviewers:
- brendandburns
- davidopp
content_type: concept
no_list: true
card:
  name: tasks
  weight: 999
  title: راهنمایی برداشتن
---

<!-- overview -->

گاهی اوقات اتفاقات ناخواسته رخ می‌دهند. این راهنما به هدف برطرف کردن مشکلات اختصاص دارد. دو بخش دارد:

* [دیباگ کردن برنامه شما](/docs/tasks/debug/debug-application/) - مفید برای کاربرانی که کد خود را در Kubernetes پیاده‌سازی کرده و دلیل عدم کارکردن آن را می‌خواهند.
* [دیباگ کردن کلاستر شما](/docs/tasks/debug/debug-cluster/) - مفید برای مدیران کلاستر و افرادی که کلاستر Kubernetes آن‌ها راضی نیست.

همچنین باید مشکلات معروف برای [نسخه](https://github.com/kubernetes/kubernetes/releases) مورد استفاده خود را بررسی کنید.

<!-- body -->

## راهنمایی برداشتن

اگر مشکل شما توسط هیچ یک از راهنماهای بالا پاسخ داده نشده است، چندین راه برای شما برای دریافت کمک از جامعه Kubernetes وجود دارد.

### سوالات

مستندات این سایت به گونه‌ای ساختاردهی شده‌اند که پاسخ‌های گسترده‌ای به سوالات مختلف ارائه می‌دهند. [مفاهیم](/docs/concepts/) معماری Kubernetes و نحوه کار هر جزء را توضیح می‌دهند، در حالی که [راه‌اندازی](/docs/setup/) دستورالعمل‌های عملی برای شروع کار ارائه می‌دهد. [وظایف](/docs/tasks/) نشان می‌دهند که چگونه وظایف معمول استفاده شده را انجام دهید، و [آموزش‌ها](/docs/tutorials/) راهنمایی‌های جامع‌تری از سناریوهای توسعه واقعی، صنعتی-خاص یا پایان به پایان فراهم می‌آورند. بخش [مرجع](/docs/reference/) مستندات دقیق راجع به [API Kubernetes](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/) و رابط‌های خط فرمانی (CLI) مانند [`kubectl`](/docs/reference/kubectl/) فراهم می‌کند.

## کمک! سوال من پوشش داده نشده است! به حالا کمک نیاز دارم!

### Stack Exchange، Stack Overflow، یا Server Fault {#stack-exchange}

اگر سوالات شما مربوط به *توسعه نرم‌افزار* برای برنامه کانتینری‌سازی شما هستند، می‌توانید آن‌ها را در [Stack Overflow](https://stackoverflow.com/questions/tagged/kubernetes) مطرح کنید.

اگر سوالات شما درباره *مدیریت کلاستر* یا *پیکربندی* Kubernetes هستند، می‌توانید آن‌ها را در [Server Fault](https://serverfault.com/questions/tagged/kubernetes) مطرح کنید.

همچنین چندین سایت شبکه Stack Exchange وجود دارد که ممکن است مکان مناسبی برای پرسیدن سوالات Kubernetes در زمینه‌های مانند [DevOps](https://devops.stackexchange.com/questions/tagged/kubernetes)، [مهندسی نرم‌افزار](https://softwareengineering.stackexchange.com/questions/tagged/kubernetes)، یا [امنیت اطلاعات](https://security.stackexchange.com/questions/tagged/kubernetes) باشند.

شخص دیگری از جامعه ممکن است پیش از این یک سوال مشابه را مطرح کرده باشد یا به حل مشکل شما کمک کند.

تیم Kubernetes همچنین مطالب مربوط به Kubernetes را در [پست‌های برچسب‌خورده Kubernetes](https://stackoverflow.com/questions/tagged/kubernetes) نظارت می‌کند. اگر سوالات موجودی ندارند که کمک کنند، **لطفاً اطمینان حاصل کنید که سوال شما [در Stack Overflow موضوعی است که می‌توانید در آن کمک بگیرید](https://stackoverflow.com/help/on-topic)، [Server Fault](https://serverfault.com/help/on-topic)، یا سایت شبکه Stack Exchange دیگر که در آن می‌پرسید، و راهنمایی راجع به [چگونگی پرسیدن یک سوال جدید](https://stackoverflow.com/help/how-to-ask) را مطالعه کنید!**

### Slack

بسیاری از افراد جامعه Kubernetes در Slack Kubernetes در کانال `#kubernetes-users` گرد هم می‌آیند.
Slack نیاز به ثبت‌نام دارد؛ شما می‌توانید [دعوتنامه را درخواست دهید](https://slack.kubernetes.io)، و ثبت‌نام برای همه باز است). لطفاً بیایید و هرگونه سوالی را بپرسید.
بعد از ثبت‌نام، به [سازمان Kubernetes در Slack](https://kubernetes.slack.com) از طریق مرورگر وب خود یا از طریق برنامه اختصاصی Slack دسترسی داشته باشید.

هنگامی که ثبت‌نام شده‌اید، فهرست رو به رشدی از کانال‌های مختلف موضوعات
مختلف را مشاهده کنید. به عنوان مثال، افراد جدید به Kubernetes همچنین می‌خواهند به کانال
[`#kubernetes-novice`](https://kubernetes.slack.com/messages/kubernetes-novice) بپیوندند. به عنوان مثال دیگر، توسعه دهندگان باید به کانال
[`#kubernetes-contributors

`](https://kubernetes.slack.com/messages/kubernetes-contributors) بپیوندند.

همچنین بسیاری از کانال‌های محلی / زبانی در کشورهای مختلف وجود دارند. لطفاً به
این کانال‌ها بپیوندید تا پشتیبانی و اطلاعات محلی را دنبال کنید:

{{< table caption="کانال‌های Slack محلی / زبانی خاص کشور" >}}
کشور | کانال‌ها
:---------|:------------
چین | [`#cn-users`](https://kubernetes.slack.com/messages/cn-users), [`#cn-events`](https://kubernetes.slack.com/messages/cn-events)
فنلاند | [`#fi-users`](https://kubernetes.slack.com/messages/fi-users)
فرانسه | [`#fr-users`](https://kubernetes.slack.com/messages/fr-users), [`#fr-events`](https://kubernetes.slack.com/messages/fr-events)
آلمان | [`#de-users`](https://kubernetes.slack.com/messages/de-users), [`#de-events`](https://kubernetes.slack.com/messages/de-events)
هند | [`#in-users`](https://kubernetes.slack.com/messages/in-users), [`#in-events`](https://kubernetes.slack.com/messages/in-events)
ایتالیا | [`#it-users`](https://kubernetes.slack.com/messages/it-users), [`#it-events`](https://kubernetes.slack.com/messages/it-events)
ژاپن | [`#jp-users`](https://kubernetes.slack.com/messages/jp-users), [`#jp-events`](https://kubernetes.slack.com/messages/jp-events)
کره | [`#kr-users`](https://kubernetes.slack.com/messages/kr-users)
هلند | [`#nl-users`](https://kubernetes.slack.com/messages/nl-users)
نروژ | [`#norw-users`](https://kubernetes.slack.com/messages/norw-users)
لهستان | [`#pl-users`](https://kubernetes.slack.com/messages/pl-users)
روسیه | [`#ru-users`](https://kubernetes.slack.com/messages/ru-users)
اسپانیا | [`#es-users`](https://kubernetes.slack.com/messages/es-users)
سوئد | [`#se-users`](https://kubernetes.slack.com/messages/se-users)
ترکیه | [`#tr-users`](https://kubernetes.slack.com/messages/tr-users), [`#tr-events`](https://kubernetes.slack.com/messages/tr-events)
{{< /table >}}

### انجمن

شما می‌توانید به انجمن رسمی Kubernetes بپیوندید: [discuss.kubernetes.io](https://discuss.kubernetes.io).

### باگ‌ها و درخواست‌های ویژگی

اگر به نظر می‌رسد که یک باگ دارید، یا می‌خواهید یک درخواست ویژگی ارسال کنید، لطفاً از [سیستم پیگیری مشکلات GitHub](https://github.com/kubernetes/kubernetes/issues) استفاده کنید.

قبل از ارسال یک مشکل، لطفاً مشکلات موجود را جستجو کنید تا ببینید که آیا مشکل شما قبلاً پوشش داده شده است.

اگر یک باگ ارسال می‌کنید، لطفاً اطلاعات دقیقی درباره نحوه بازتولید مشکل را شامل کنید، مانند:

* نسخه Kubernetes: `kubectl version`
* ارائه دهنده ابر، توزیع OS، تنظیمات شبکه، و نسخه نرم‌افزار اجرایی کانتینر
* مراحل برای بازتولید مشکل

