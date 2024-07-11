# ابزار بررسی لینک داخلی

می‌توانید از [htmltest](https://github.com/wjdp/htmltest) استفاده کنید تا لینک‌های خراب در [`/content/en/`](https://git.k8s.io/website/content/en/) را بررسی کنید. این ابزار در زمان بازسازی بخش‌های محتوا، جابجایی صفحات یا تغییر نام فایل‌ها یا سربرگ‌های صفحه مفید است.

## نحوه عمل ابزار

`htmltest` لینک‌ها را در فایل‌های HTML تولید شده در مخزن وب سایت کوبرنتیز اسکن می‌کند. این ابزار با استفاده از دستور `make` اجرا می‌شود که عملیات زیر را انجام می‌دهد:

- ساخت وب‌سایت و تولید فایل‌های HTML خروجی در دایرکتوری `/public` مخزن `kubernetes/website` محلی شما
- دریافت تصویر Docker `wdjp/htmltest`
- متصل کردن مخزن محلی `kubernetes/website` به تصویر Docker
- اسکن فایل‌های تولید شده در دایرکتوری `/public` و ارائه خروجی خط فرمان هنگامی که با لینک‌های داخلی خراب مواجه می‌شود

## آنچه که بررسی می‌کند و نمی‌کند

بررسی‌کننده لینک‌ها فایل‌های HTML تولید شده را اسکن می‌کند، نه Markdownهای خام. ابزار htmltest به یک فایل پیکربندی نیاز دارد، [`.htmltest.yml`](https://git.k8s.io/website/.htmltest.yml)، برای تعیین محتوایی که باید بررسی شود.

بررسی‌کننده لینک‌ها شامل موارد زیر است:

- همه محتوا تولید شده از Markdown در دایرکتوری [`/content/en/docs`](https://git.k8s.io/website/content/en/docs/)، با استثنای:
  - مراجعه‌های API تولید شده، به عنوان مثال https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/
- تمامی لینک‌های داخلی، با استثنای:
  - هش‌های خالی (`<a href="#">` یا `[title](#)`) و hrefهای خالی (`<a href="">` یا `[title]()`)
  - لینک‌های داخلی به تصاویر و فایل‌های رسانه‌ای دیگر

بررسی‌کننده لینک‌ها شامل موارد زیر نمی‌شود:

- لینک‌های موجود در نوارهای بالا و کناری، لینک‌های پایین صفحه یا لینک‌های بخش `<head>` صفحه مانند لینک‌های به صفحات استایل CSS، اسکریپت‌ها و اطلاعات متا
- صفحات سطح بالا و زیرمجموعه‌های آن‌ها، به عنوان مثال: `/training`، `/community`، `/case-studies/adidas`
- پست‌های وبلاگ
- مستندات مرجع API، به عنوان مثال: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/
- محلیت‌ها

## پیش‌نیازها و نصب

شما باید نرم‌افزارهای زیر را نصب کنید:
* [Docker](https://docs.docker.com/get-docker/)
* [make](https://www.gnu.org/software/make/)

## اجرای بررسی لینک

برای اجرای بررسی لینک:

1. به دایرکتوری ریشه مخزن محلی `kubernetes/website` خود بروید.

2. دستور زیر را اجرا کنید:

  ```
  make container-internal-linkcheck
  ```

## درک خروجی

اگر بررسی لینک لینک‌های خراب را پیدا کند، خروجی مشابه زیر است:

```
tasks/access-kubernetes-api/custom-resources/index.html
  hash does not exist --- tasks/access-kubernetes-api/custom-resources/index.html --> #preserving-unknown-fields
  hash does not exist --- tasks/access-kubernetes-api/custom-resources/index.html --> #preserving-unknown-fields
```

این یک مجموعه از لینک‌های خراب است. لاگ برای هر صفحه با لینک‌های خراب افزوده می‌شود.

در این خروجی، فایل دارای لینک‌های خراب `tasks/access-kubernetes-api/custom-resources.md` است.

ابزار دلیلی ارائه می‌دهد: `hash does not exist`. در اکثر موارد، می‌توانید این مورد را نادیده بگیرید.

آدرس URL مقصد `#preserving-unknown-fields` است.

یک راه برای اصلاح این مورد این است:

1. به فایل Markdown با لینک خراب بروید.
2. از ویرایشگر متنی استفاده کنید و جستجوی تمام متن (معمولاً Ctrl+F یا Command+F) را برای URL لینک خراب، `#preserving-unknown-fields`، انجام دهید.
3. لینک را اصلاح کنید. برای یک لینک صفحه هش (یا _آنکور_) خراب، بررسی کنید که آیا موضوع تغییر نام یا حذف شده است.

htmltest را اجرا کنید تا اطمینان حاصل شود که لینک‌های خراب تصحیح شده‌اند.