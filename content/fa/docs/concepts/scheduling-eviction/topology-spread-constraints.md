---
title: محدودیت‌های پراکندگی توپولوژی پاد
content_type: concept
weight: 40
---

<!-- overview -->

می‌توانید از _محدودیت‌های پراکندگی توپولوژی_ برای کنترل نحوه پراکندگی 
{{< glossary_tooltip text="پادها" term_id="Pod" >}}
در سراسر کلاستر خود بین دامنه‌های شکست مانند نواحی، مناطق، گره‌ها و سایر دامنه‌های توپولوژی تعریف‌شده توسط کاربر استفاده کنید. این کار می‌تواند به دستیابی به دسترسی بالا و همچنین استفاده کارآمد از منابع کمک کند.

می‌توانید [محدودیت‌های سطح کلاستر](#cluster-level-default-constraints) را به‌عنوان پیش‌فرض تنظیم کنید یا محدودیت‌های پراکندگی توپولوژی را برای بارهای کاری خاص پیکربندی کنید.

<!-- body -->

## انگیزه

تصور کنید که یک کلاستر با حداکثر بیست گره دارید و می‌خواهید یک
{{< glossary_tooltip text="بار کاری" term_id="workload" >}}
را اجرا کنید که به‌طور خودکار تعداد کپی‌های مورد استفاده خود را مقیاس می‌کند. ممکن است به اندازه دو پاد یا به اندازه پانزده پاد وجود داشته باشد.
وقتی فقط دو پاد وجود دارد، ترجیح می‌دهید هر دو پاد روی یک گره اجرا نشوند: در این صورت خطر این وجود دارد که خرابی یک گره بار کاری شما را آفلاین کند.

علاوه بر این استفاده اساسی، مثال‌هایی از استفاده پیشرفته نیز وجود دارد که به بارهای کاری شما کمک می‌کند از دسترسی بالا و بهره‌وری کلاستر بهره‌مند شوند.

با افزایش مقیاس و اجرای تعداد بیشتری پاد، نگرانی دیگری اهمیت پیدا می‌کند. تصور کنید که سه گره دارید که هر کدام پنج پاد اجرا می‌کنند. گره‌ها ظرفیت کافی برای اجرای این تعداد کپی را دارند؛ با این حال، کلاینت‌هایی که با این بار کاری تعامل دارند، بین سه مرکز داده (یا مناطق زیرساختی) مختلف تقسیم شده‌اند. اکنون نگرانی کمتری درباره خرابی یک گره دارید، اما متوجه می‌شوید که تأخیر بالاتر از حد انتظار است و برای هزینه‌های شبکه مربوط به ارسال ترافیک شبکه بین مناطق مختلف پرداخت می‌کنید.

تصمیم می‌گیرید که در شرایط عادی ترجیح می‌دهید تعداد مشابهی از کپی‌ها به هر منطقه زیرساختی [زمان‌بندی شده](/docs/concepts/scheduling-eviction/) باشد و می‌خواهید کلاستر در صورت بروز مشکل خود-شفا باشد.

محدودیت‌های پراکندگی توپولوژی به شما یک روش اعلامی برای پیکربندی این موارد ارائه می‌دهد.

## فیلد `topologySpreadConstraints`

API پاد شامل یک فیلد به نام `spec.topologySpreadConstraints` است. استفاده از این فیلد به شکل زیر است:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  # پیکربندی یک محدودیت پراکندگی توپولوژی
  topologySpreadConstraints:
    - maxSkew: <integer>
      minDomains: <integer> # اختیاری
      topologyKey: <string>
      whenUnsatisfiable: <string>
      labelSelector: <object>
      matchLabelKeys: <list> # اختیاری؛ نسخه بتا از v1.27
      nodeAffinityPolicy: [Honor|Ignore] # اختیاری؛ نسخه بتا از v1.26
      nodeTaintsPolicy: [Honor|Ignore] # اختیاری؛ نسخه بتا از v1.26
  ### فیلدهای دیگر پاد در اینجا قرار می‌گیرند
```

می‌توانید با اجرای `kubectl explain Pod.spec.topologySpreadConstraints` یا مراجعه به بخش [زمان‌بندی](/docs/reference/kubernetes-api/workload-resources/pod-v1/#scheduling) در مستندات مرجع API پاد، اطلاعات بیشتری درباره این فیلد بخوانید.

### تعریف محدودیت پراکندگی

می‌توانید یک یا چند ورودی `topologySpreadConstraints` را تعریف کنید تا به زمان‌بند kube بگویید چگونه هر پاد ورودی را نسبت به پادهای موجود در سراسر کلاستر قرار دهد. این فیلدها عبارتند از:

- **maxSkew** میزان نابرابری توزیع پادها را توصیف می‌کند. شما باید این فیلد را مشخص کنید و عدد باید بیشتر از صفر باشد. معنای این فیلد بسته به مقدار `whenUnsatisfiable` متفاوت است:

  - اگر `whenUnsatisfiable: DoNotSchedule` را انتخاب کنید، `maxSkew` حداکثر تفاوت مجاز بین تعداد پادهای مطابقت‌دار در توپولوژی هدف و حداقل جهانی را تعریف می‌کند
    (حداقل تعداد پادهای مطابقت‌دار در یک دامنه واجد شرایط یا صفر اگر تعداد دامنه‌های واجد شرایط کمتر از MinDomains باشد).
    به عنوان مثال، اگر 3 منطقه با 2، 2 و 1 پاد مطابقت‌دار داشته باشید،
    `MaxSkew` برابر با 1 باشد، آنگاه حداقل جهانی 1 است.
  - اگر `whenUnsatisfiable: ScheduleAnyway` را انتخاب کنید، زمان‌بند اولویت بیشتری به توپولوژی‌هایی می‌دهد که به کاهش نابرابری کمک می‌کنند.

- **minDomains** تعداد حداقل دامنه‌های واجد شرایط را نشان می‌دهد. این فیلد اختیاری است.
  یک دامنه، نمونه خاصی از یک توپولوژی است. یک دامنه واجد شرایط، دامنه‌ای است که گره‌های آن با انتخاب‌گر گره مطابقت دارند.

  <!-- OK برای حذف این یادداشت پس از پشتیبانی از Kubernetes v1.29 -->
  {{< note >}}
  قبل از Kubernetes v1.30، فیلد `minDomains` فقط در صورتی در دسترس بود که 
  [دروازه ویژگی](/docs/reference/command-line-tools-reference/feature-gates/)
  `MinDomainsInPodTopologySpread` فعال باشد (به طور پیش‌فرض از v1.28). در کلاسترهای قدیمی‌تر Kubernetes ممکن است به‌طور صریح غیرفعال شده باشد یا فیلد در دسترس نباشد.
  {{< /note >}}

  - مقدار `minDomains` باید بزرگ‌تر از 0 باشد، در صورت مشخص شدن.
    شما می‌توانید فقط `minDomains` را همراه با `whenUnsatisfiable: DoNotSchedule` مشخص کنید.
  - هنگامی که تعداد دامنه‌های واجد شرایط با کلیدهای توپولوژی کمتر از `minDomains` باشد،
    پراکندگی توپولوژی پاد حداقل جهانی را صفر در نظر می‌گیرد و سپس محاسبه نابرابری انجام می‌شود.
    حداقل جهانی حداقل تعداد پادهای مطابقت‌دار در یک دامنه واجد شرایط است،
    یا صفر اگر تعداد دامنه‌های واجد شرایط کمتر از `minDomains` باشد.
  - هنگامی که تعداد دامنه‌های واجد شرایط با کلیدهای توپولوژی برابر یا بیشتر از
    `minDomains` باشد، این مقدار تأثیری بر زمان‌بندی ندارد.
  - اگر `minDomains` را مشخص نکنید، محدودیت به‌گونه‌ای عمل می‌کند که گویی `minDomains` برابر با 1 است.

- **topologyKey** کلید [برچسب‌های گره](#node-labels) است. گره‌هایی که دارای برچسب با این کلید و مقادیر یکسان هستند، در یک توپولوژی در نظر گرفته می‌شوند.
  هر نمونه از توپولوژی (به عبارتی یک جفت <کلید، مقدار>) را یک دامنه می‌نامیم. زمان‌بند سعی می‌کند تعداد متعادلی از پادها را در هر دامنه قرار دهد.
  همچنین، یک دامنه واجد شرایط را دامنه‌ای تعریف می‌کنیم که گره‌های آن با الزامات
  nodeAffinityPolicy و nodeTaintsPolicy مطابقت دارند.

- **whenUnsatisfiable** نحوه برخورد با یک پاد را زمانی که محدودیت پراکندگی را برآورده نمی‌کند، نشان می‌دهد:
  - `DoNotSchedule` (پیش‌فرض) به زمان‌بند می‌گوید که آن را زمان‌بندی نکند.
  - `ScheduleAnyway` به زمان‌بند می‌گوید که آن را همچنان زمان‌بندی کند در حالی که اولویت بیشتری به گره‌هایی می‌دهد که نابرابری را کاهش می‌دهند.

- **labelSelector** برای یافتن پادهای مطابقت‌دار استفاده می‌شود. پادهایی
  که با این انتخاب‌گر برچسب مطابقت دارند شمارش می‌شوند تا تعداد پادها در دامنه توپولوژی مربوطه آنها تعیین شود.
  برای جزئیات بیشتر به [انتخاب‌گرهای برچسب](/docs/concepts/overview/working-with-objects/labels/#label-selectors) مراجعه کنید.

- **matchLabelKeys** لیستی از کلیدهای برچسب پاد است که برای انتخاب پادهایی که پراکندگی بر روی آنها محاسبه می‌شود، استفاده می‌شود. این کلیدها برای جستجوی مقادیر از برچسب‌های پاد استفاده می‌شوند،
  این برچسب‌های کلید-مقدار با `labelSelector` به‌طور منطقی AND می‌شوند تا گروه پادهای موجودی را که پراکندگی بر روی آنها محاسبه می‌شود برای پاد ورودی انتخاب کنند. وجود همان کلید در هر دو `matchLabelKeys` و `labelSelector` ممنوع است. `matchLabelKeys` نمی‌تواند زمانی که `labelSelector` تنظیم نشده است، تنظیم شود. کلیدهایی که در برچسب‌های پاد وجود ندارند، نادیده گرفته می‌شوند. لیست null یا خالی به معنای مطابقت تنها با `labelSelector` است.

  با استفاده از `matchLabelKeys` نیازی به به‌روزرسانی `pod.spec` بین نسخه‌های مختلف ندارید.
  کنترلر/اپراتور فقط نیاز دارد که مقادیر مختلفی به همان کلید برچسب برای نسخه‌های مختلف تنظیم کند.
  زمان‌بند مقادیر را به‌طور خودکار بر اساس `matchLabelKeys` فرض خواهد کرد. برای
  مثال، اگر یک Deployment را پیکربندی می‌کنید، می‌توانید از برچسبی با کلید
  [pod-template-hash](/docs/concepts/workloads/controllers/deployment/#pod-template-hash-label) استفاده کنید که به‌طور خودکار توسط کنترلر Deployment اضافه می‌شود تا بین نسخه‌های مختلف در یک Deployment تمایز قائل شود.

  ```yaml
      topologySpreadConstraints:
          - maxSkew: 1
            topologyKey: kubernetes.io/hostname
            whenUnsatisfiable: DoNotSchedule
            labelSelector:
              matchLabels:
                app: foo
            matchLabelKeys:
              - pod-template-hash
  ```

  {{< note >}}
  فیلد `matchLabelKeys` یک فیلد در سطح بتا است و به‌طور پیش‌فرض در نسخه 1.27 فعال است. می‌توانید آن را با غیرفعال کردن 
  [دروازه ویژگی](/docs/reference/command-line-tools-reference/feature-gates/)
  `MatchLabelKeysInPodTopologySpread` غیرفعال کنید.
  {{< /note >}}

- **nodeAffinityPolicy** نشان می‌دهد چگونه با nodeAffinity/nodeSelector پاد هنگام محاسبه نابرابری پراکندگی توپولوژی پاد برخورد کنیم. گزینه‌ها عبارتند از:
  - Honor: تنها گره‌هایی که با nodeAffinity/nodeSelector مطابقت دارند در محاسبات گنجانده می‌شوند.
  - Ignore: nodeAffinity/nodeSelector نادیده گرفته می‌شوند. همه گره‌ها در محاسبات گنجانده می‌شوند.

  اگر این مقدار null باشد، رفتار معادل با سیاست Honor خواهد بود.

  {{< note >}}
  `nodeAffinityPolicy` یک فیلد در سطح بتا است و به‌طور پیش‌فرض در نسخه 1.26 فعال است. می‌توانید آن را با غیرفعال کردن
  [دروازه ویژگی](/docs/reference/command-line-tools-reference/feature-gates/)
  `NodeInclusionPolicyInPodTopologySpread` غیرفعال کنید.
  {{< /note >}}

- **nodeTaintsPolicy** نشان می‌دهد چگونه با taints گره هنگام محاسبه نابرابری پراکندگی توپولوژی پاد برخورد کنیم. گزینه‌ها عبارتند از:
  - Honor: گره‌هایی بدون taints، همراه با گره‌های دارای taints که پاد ورودی برای آنها تحمل دارد، گنجانده می‌شوند.
  - Ignore: taints گره نادیده گرفته می‌شوند. همه گره‌ها گنجانده می‌شوند.

  اگر این مقدار null باشد، رفتار معادل با سیاست Ignore خواهد بود.

  {{< note >}}
  `nodeTaintsPolicy` یک فیلد در سطح بتا است و به‌طور پیش‌فرض در نسخه 1.26 فعال است. می‌توانید آن را با غیرفعال کردن
  [دروازه ویژگی](/docs/reference/command-line-tools-reference/feature-gates/)
  `NodeInclusionPolicyInPodTopologySpread` غیرفعال کنید.
  {{< /note >}}

وقتی یک پاد بیش از یک `topologySpreadConstraint` تعریف می‌کند، این محدودیت‌ها با استفاده از یک عملگر منطقی AND ترکیب می‌شوند: زمان‌بند kube به دنبال گره‌ای برای پاد ورودی می‌گردد که تمام محدودیت‌های پیکربندی‌شده را برآورده کند.

### برچسب‌های گره

محدودیت‌های پراکندگی توپولوژی به برچسب‌های گره متکی هستند تا دامنه(های) توپولوژی که هر 
{{< glossary_tooltip text="گره" term_id="node" >}} 
در آن قرار دارد را شناسایی کنند.
برای مثال، یک گره ممکن است دارای برچسب‌های زیر باشد:
```yaml
  region: us-east-1
  zone: us-east-1a
```

{{< note >}}
برای اختصار، این مثال از کلیدهای برچسب [معروف](/docs/reference/labels-annotations-taints/)
`topology.kubernetes.io/zone` و `topology.kubernetes.io/region` استفاده نمی‌کند. با این حال،
توصیه می‌شود که از این کلیدهای برچسب ثبت‌شده استفاده شود نه کلیدهای برچسب خصوصی
(بدون پیش‌وند) `region` و `zone` که در اینجا استفاده شده‌اند.

نمی‌توانید در زمینه‌های مختلف فرضیات قابل اعتمادی درباره معنی یک کلید برچسب خصوصی بکنید.
{{< /note >}}

فرض کنید یک کلاستر 4-گرهی با برچسب‌های زیر دارید:

```
NAME    STATUS   ROLES    AGE     VERSION   LABELS
node1   Ready    <none>   4m26s   v1.16.0   node=node1,zone=zoneA
node2   Ready    <none>   3m58s   v1.16.0   node=node2,zone=zoneA
node3   Ready    <none>   3m17s   v1.16.0   node=node3,zone=zoneB
node4   Ready    <none>   2m43s   v1.16.0   node=node4,zone=zoneB
```

سپس کلاستر به‌صورت منطقی به شکل زیر مشاهده می‌شود:

{{<mermaid>}}
graph TB
    subgraph "zoneB"
        n3(Node3)
        n4(Node4)
    end
    subgraph "zoneA"
        n1(Node1)
        n2(Node2)
    end

    classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
    classDef k8s fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
    classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
    class n1,n2,n3,n4 k8s;
    class zoneA,zoneB cluster;
{{< /mermaid >}}
## سازگاری

باید محدودیت‌های پراکندگی توپولوژی پاد را برای تمام پادهای یک گروه یکسان تنظیم کنید.

معمولاً، اگر از یک کنترلر بارکاری مانند Deployment استفاده می‌کنید، الگوی پاد
این کار را برای شما انجام می‌دهد. اگر محدودیت‌های پراکندگی مختلف را مخلوط کنید، Kubernetes
از تعریف API فیلد پیروی می‌کند؛ با این حال، رفتار بیشتر به احتمال زیاد گیج‌کننده می‌شود و عیب‌یابی کمتر مستقیم است.

شما به یک مکانیزم نیاز دارید تا اطمینان حاصل کنید که تمام گره‌ها در یک دامنه توپولوژی (مانند
یک منطقه ارائه‌دهنده ابر) به‌طور سازگار برچسب‌گذاری شده‌اند.
برای جلوگیری از نیاز به برچسب‌گذاری دستی گره‌ها، اکثر کلاسترها به‌طور خودکار
برچسب‌های معروفی مانند `kubernetes.io/hostname` را پر می‌کنند. بررسی کنید که
کلاستر شما از این پشتیبانی می‌کند یا خیر.

## مثال‌های محدودیت پراکندگی توپولوژی

### مثال: یک محدودیت پراکندگی توپولوژی {#example-one-topologyspreadconstraint}

فرض کنید یک کلاستر ۴-گرهی دارید که در آن ۳ پاد با برچسب `foo: bar` به‌ترتیب در
node1، node2 و node3 قرار دارند:

{{<mermaid>}}
graph BT
    subgraph "zoneB"
        p3(Pod) --> n3(Node3)
        n4(Node4)
    end
    subgraph "zoneA"
        p1(Pod) --> n1(Node1)
        p2(Pod) --> n2(Node2)
    end

    classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
    classDef k8s fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
    classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
    class n1,n2,n3,n4,p1,p2,p3 k8s;
    class zoneA,zoneB cluster;
{{< /mermaid >}}

اگر می‌خواهید یک پاد ورودی به‌طور مساوی با پادهای موجود در سراسر مناطق پراکنده شود،
می‌توانید از یک مانفیست مشابه استفاده کنید:

{{% code_sample file="pods/topology-spread-constraints/one-constraint.yaml" %}}

از آن مانفیست، `topologyKey: zone` به این معنا است که توزیع یکنواخت فقط بر روی
گره‌هایی که برچسب `zone: <any value>` دارند اعمال خواهد شد (گره‌هایی که برچسب `zone` ندارند
نادیده گرفته می‌شوند). فیلد `whenUnsatisfiable: DoNotSchedule` به زمان‌بند می‌گوید که
اگر نتواند راهی برای برآورده کردن محدودیت پیدا کند، اجازه دهد پاد ورودی در حالت در حال انتظار باقی بماند.

اگر زمان‌بند این پاد ورودی را در منطقه `A` قرار دهد، توزیع پادها به `[3, 1]` تبدیل می‌شود. این به این معنا است که نابرابری واقعی سپس ۲ می‌شود (که به‌عنوان `3 - 1` محاسبه می‌شود)، که
`maxSkew: 1` را نقض می‌کند. برای برآورده کردن محدودیت‌ها و زمینه این مثال، پاد ورودی فقط می‌تواند بر روی یک گره در منطقه `B` قرار گیرد:

{{<mermaid>}}
graph BT
    subgraph "zoneB"
        p3(Pod) --> n3(Node3)
        p4(mypod) --> n4(Node4)
    end
    subgraph "zoneA"
        p1(Pod) --> n1(Node1)
        p2(Pod) --> n2(Node2)
    end

    classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
    classDef k8s fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
    classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
    class n1,n2,n3,n4,p1,p2,p3 k8s;
    class p4 plain;
    class zoneA,zoneB cluster;
{{< /mermaid >}}

یا

{{<mermaid>}}
graph BT
    subgraph "zoneB"
        p3(Pod) --> n3(Node3)
        p4(mypod) --> n3
        n4(Node4)
    end
    subgraph "zoneA"
        p1(Pod) --> n1(Node1)
        p2(Pod) --> n2(Node2)
    end

    classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
    classDef k8s fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
    classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
    class n1,n2,n3,n4,p1,p2,p3 k8s;
    class p4 plain;
    class zoneA,zoneB cluster;
{{< /mermaid >}}

می‌توانید مشخصات پاد را تغییر دهید تا انواع مختلفی از نیازها را برآورده کنید:

- `maxSkew` را به یک مقدار بزرگ‌تر - مانند `۲` - تغییر دهید تا پاد ورودی بتواند
  در منطقه `A` نیز قرار گیرد.
- `topologyKey` را به `node` تغییر دهید تا پادها به‌طور مساوی در سراسر گره‌ها توزیع شوند
  نه مناطق. در مثال بالا، اگر `maxSkew` باقی بماند `۱`، پاد ورودی فقط می‌تواند بر روی گره `node4` قرار گیرد.
- `whenUnsatisfiable: DoNotSchedule` را به `whenUnsatisfiable: ScheduleAnyway` تغییر دهید
  تا اطمینان حاصل شود که پاد ورودی همیشه قابل زمان‌بندی باشد (فرض کنید سایر APIهای زمان‌بندی
  برآورده شده باشند). با این حال، ترجیح داده می‌شود که در دامنه توپولوژی قرار گیرد که
  پادهای کمتری دارد. (توجه داشته باشید که این ترجیح به‌طور مشترک با سایر اولویت‌های داخلی زمان‌بندی
  مانند نسبت استفاده از منابع نرمال‌سازی می‌شود).

### مثال: محدودیت‌های پراکندگی توپولوژی متعدد {#example-multiple-topologyspreadconstraints}

این مورد بر اساس مثال قبلی است. فرض کنید یک کلاستر ۴-گرهی دارید که در آن ۳
پاد موجود با برچسب `foo: bar` به‌ترتیب در node1، node2 و node3 قرار دارند:

{{<mermaid>}}
graph BT
    subgraph "zoneB"
        p3(Pod) --> n3(Node3)
        n4(Node4)
    end
    subgraph "zoneA"
        p1(Pod) --> n1(Node1)
        p2(Pod) --> n2(Node2)
    end

    classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
    classDef k8s fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
    classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
    class n1,n2,n3,n4,p1,p2,p3 k8s;
    class p4 plain;
    class zoneA,zoneB cluster;
{{< /mermaid >}}

می‌توانید دو محدودیت پراکندگی توپولوژی را ترکیب کنید تا پراکندگی پادها را هم
براساس گره و هم براساس منطقه کنترل کنید:

{{% code_sample file="pods/topology-spread-constraints/two-constraints.yaml" %}}

در این مورد، برای مطابقت با اولین محدودیت، پاد ورودی فقط می‌تواند بر روی
گره‌ها در منطقه `B` قرار گیرد؛ در حالی که از نظر دومین محدودیت، پاد ورودی فقط می‌تواند
به گره `node4` زمان‌بندی شود. زمان‌بند فقط گزینه‌هایی را در نظر می‌گیرد که تمام
محدودیت‌های تعریف‌شده را برآورده کنند، بنابراین تنها مکان معتبر برای قرارگیری گره `node4` است.
### مثال: محدودیت‌های پراکندگی توپولوژی متناقض {#example-conflicting-topologyspreadconstraints}

محدودیت‌های متعدد می‌توانند منجر به تضاد شوند. فرض کنید یک کلاستر ۳-گرهی در دو منطقه دارید:

{{<mermaid>}}
graph BT
    subgraph "zoneB"
        p4(Pod) --> n3(Node3)
        p5(Pod) --> n3
    end
    subgraph "zoneA"
        p1(Pod) --> n1(Node1)
        p2(Pod) --> n1
        p3(Pod) --> n2(Node2)
    end

    classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
    classDef k8s fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
    classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
    class n1,n2,n3,n4,p1,p2,p3,p4,p5 k8s;
    class zoneA,zoneB cluster;
{{< /mermaid >}}

اگر بخواهید [`two-constraints.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/topology-spread-constraints/two-constraints.yaml)
(مانفیست از مثال قبلی)
را به **این** کلاستر اعمال کنید، مشاهده خواهید کرد که پاد `mypod` در حالت `Pending` باقی می‌ماند.
این اتفاق به این دلیل می‌افتد که: برای برآورده کردن اولین محدودیت، پاد `mypod` تنها
می‌تواند در منطقه `B` قرار گیرد؛ در حالی که از نظر دومین محدودیت، پاد `mypod`
تنها می‌تواند در گره `node2` زمان‌بندی شود. تقاطع دو محدودیت یک مجموعه خالی را برمی‌گرداند و زمان‌بند نمی‌تواند پاد را قرار دهد.

برای غلبه بر این وضعیت، می‌توانید مقدار `maxSkew` را افزایش دهید یا یکی از محدودیت‌ها را به `whenUnsatisfiable: ScheduleAnyway` تغییر دهید. بسته به شرایط، ممکن است تصمیم بگیرید که یک پاد موجود را به صورت دستی حذف کنید - به عنوان مثال، اگر در حال عیب‌یابی دلیل عدم پیشرفت یک رفع اشکال هستید.

#### تعامل با وابستگی گره و انتخاب‌کننده‌های گره

زمان‌بند گره‌های نامطابق را از محاسبات نابرابری نادیده می‌گیرد اگر پاد ورودی دارای `spec.nodeSelector` یا `spec.affinity.nodeAffinity` تعریف شده باشد.

### مثال: محدودیت‌های پراکندگی توپولوژی با وابستگی گره {#example-topologyspreadconstraints-with-nodeaffinity}

فرض کنید یک کلاستر ۵-گرهی دارید که در مناطق A تا C گسترده شده است:

{{<mermaid>}}
graph BT
    subgraph "zoneB"
        p3(Pod) --> n3(Node3)
        n4(Node4)
    end
    subgraph "zoneA"
        p1(Pod) --> n1(Node1)
        p2(Pod) --> n2(Node2)
    end

classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
classDef k8s fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
class n1,n2,n3,n4,p1,p2,p3 k8s;
class p4 plain;
class zoneA,zoneB cluster;
{{< /mermaid >}}

{{<mermaid>}}
graph BT
    subgraph "zoneC"
        n5(Node5)
    end

classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
classDef k8s fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
class n5 k8s;
class zoneC cluster;
{{< /mermaid >}}

و می‌دانید که باید منطقه `C` را حذف کنید. در این حالت، می‌توانید یک مانفیست به صورت زیر ترکیب کنید تا پاد `mypod` در منطقه `B` قرار گیرد نه منطقه `C`.
به همین ترتیب، Kubernetes نیز `spec.nodeSelector` را محترم می‌شمارد.

{{% code_sample file="pods/topology-spread-constraints/one-constraint-with-nodeaffinity.yaml" %}}

## قراردادهای ضمنی

برخی از قراردادهای ضمنی در اینجا وجود دارند که ارزش ذکر دارند:

- فقط پادهایی که همان فضای نام به عنوان پاد ورودی را دارند می‌توانند کاندیدای تطابق باشند.

- زمان‌بند هر گره‌ای که هیچ یک از `topologySpreadConstraints[*].topologyKey` را نداشته باشد نادیده می‌گیرد. این به این معنا است که:

  1. هر پادی که در آن گره‌های نادیده قرار گرفته‌اند بر محاسبات `maxSkew` تأثیری ندارد - در
     مثال بالا، فرض کنید گره `node1` برچسب "zone" ندارد، سپس ۲ پاد نادیده گرفته می‌شوند، بنابراین پاد ورودی در منطقه `A` قرار می‌گیرد.
  2. پاد ورودی هیچ شانسی برای زمان‌بندی روی این نوع گره‌ها ندارد -
     در مثال بالا، فرض کنید گره‌ای `node5` برچسب **اشتباه تایپی** `zone-typo: zoneC`
     (و برچسب `zone` تنظیم نشده باشد). بعد از پیوستن گره `node5` به کلاستر، نادیده گرفته می‌شود و
     پادها برای این بارکاری در آنجا زمان‌بندی نمی‌شوند.

- آگاه باشید که اگر `topologySpreadConstraints[*].labelSelector` پاد ورودی با برچسب‌های خودش تطابق نداشته باشد چه اتفاقی می‌افتد. در
  مثال بالا، اگر برچسب‌های پاد ورودی را حذف کنید، هنوز هم می‌تواند بر روی
  گره‌ها در منطقه `B` قرار گیرد، زیرا محدودیت‌ها هنوز برآورده می‌شوند. با این حال، بعد از آن
  قرارگیری، درجه عدم تعادل کلاستر بدون تغییر باقی می‌ماند - هنوز هم منطقه `A`
  دارای ۲ پاد برچسب‌گذاری شده به عنوان `foo: bar` است، و منطقه `B` دارای ۱ پاد برچسب‌گذاری شده به عنوان
  `foo: bar` است. اگر این چیزی نیست که انتظار دارید، `topologySpreadConstraints[*].labelSelector` بارکاری را به‌روز کنید تا با برچسب‌های الگوی پاد تطابق داشته باشد.

## محدودیت‌های پیش‌فرض سطح کلاستر

امکان تنظیم محدودیت‌های پیش‌فرض پراکندگی توپولوژی برای یک کلاستر وجود دارد. محدودیت‌های پیش‌فرض
پراکندگی توپولوژی در صورتی به یک پاد اعمال می‌شوند که و فقط اگر:

- هیچ محدودیتی در `.spec.topologySpreadConstraints` خود تعریف نکرده باشد.
- به یک سرویس، ReplicaSet، StatefulSet یا ReplicationController تعلق داشته باشد.

محدودیت‌های پیش‌فرض می‌توانند به عنوان بخشی از آرگومان‌های پلاگین `PodTopologySpread` در یک
[پروفایل زمان‌بندی](/docs/reference/scheduling/config/#profiles) تنظیم شوند.
محدودیت‌ها با استفاده از همان [API بالا](#topologyspreadconstraints-field) مشخص می‌شوند، به جز اینکه
`labelSelector` باید خالی باشد. سلکتورها از سرویس‌ها،
ReplicaSetها، StatefulSetها یا ReplicationControllerهایی که پاد به آنها تعلق دارد محاسبه می‌شوند.

یک نمونه پیکربندی به صورت زیر است:

```yaml
apiVersion: kubescheduler.config.k8s.io/v1beta3
kind: KubeSchedulerConfiguration

profiles:
  - schedulerName: default-scheduler
    pluginConfig:
      - name: PodTopologySpread
        args:
          defaultConstraints:
            - maxSkew: 1
              topologyKey: topology.kubernetes.io/zone
              whenUnsatisfiable: ScheduleAnyway
          defaultingType: List
```

### محدودیت‌های پیش‌فرض داخلی {#internal-default-constraints}

{{< feature-state for_k8s_version="v1.24" state="stable" >}}

اگر هیچ محدودیت پیش‌فرض سطح کلاستر برای پراکندگی توپولوژی پادها پیکربندی نکنید،
زمان‌بند kube-scheduler به گونه‌ای عمل می‌کند که انگار محدودیت‌های پیش‌فرض توپولوژی زیر را مشخص کرده‌اید:

```yaml
defaultConstraints:
  - maxSkew: 3
    topologyKey: "kubernetes.io/hostname"
    whenUnsatisfiable: ScheduleAnyway
  - maxSkew: 5
    topologyKey: "topology.kubernetes.io/zone"
    whenUnsatisfiable: ScheduleAnyway
```

همچنین، پلاگین قدیمی `SelectorSpread` که رفتار معادل را ارائه می‌دهد، به طور پیش‌فرض غیرفعال است.

{{< note >}}
پلاگین `PodTopologySpread` گره‌هایی را که کلیدهای توپولوژی مشخص شده در محدودیت‌های پراکندگی را ندارند، امتیازدهی نمی‌کند. این ممکن است منجر به رفتار پیش‌فرض متفاوتی نسبت به پلاگین قدیمی `SelectorSpread` هنگام استفاده از محدودیت‌های پیش‌فرض توپولوژی شود.

اگر انتظار ندارید گره‌های شما **هر دو** برچسب `kubernetes.io/hostname` و
`topology.kubernetes.io/zone` را داشته باشند، محدودیت‌های خود را به جای استفاده از پیش‌فرض‌های Kubernetes تعریف کنید.
{{< /note >}}

اگر نمی‌خواهید از محدودیت‌های پیش‌فرض پراکندگی پاد برای کلاستر خود استفاده کنید،
می‌توانید این پیش‌فرض‌ها را با تنظیم `defaultingType` به `List` و خالی گذاشتن `defaultConstraints` در پیکربندی پلاگین `PodTopologySpread` غیرفعال کنید:

```yaml
apiVersion: kubescheduler.config.k8s.io/v1beta3
kind: KubeSchedulerConfiguration

profiles:
  - schedulerName: default-scheduler
    pluginConfig:
      - name: PodTopologySpread
        args:
          defaultConstraints: []
          defaultingType: List
```

## مقایسه با podAffinity و podAntiAffinity {#comparison-with-podaffinity-podantiaffinity}

در Kubernetes، [وابستگی بین پادها و ضد وابستگی بین پادها](/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity)
نحوه زمان‌بندی پادها نسبت به یکدیگر را کنترل می‌کنند - یا متراکم‌تر
یا پراکنده‌تر.

`podAffinity`
: پادها را جذب می‌کند؛ می‌توانید هر تعداد پاد را در دامنه(های) توپولوژی واجد شرایط قرار دهید.

`podAntiAffinity`
: پادها را دفع می‌کند. اگر این گزینه را به حالت `requiredDuringSchedulingIgnoredDuringExecution` تنظیم کنید، تنها یک پاد می‌تواند در یک دامنه توپولوژی زمان‌بندی شود؛ اگر
  `preferredDuringSchedulingIgnoredDuringExecution` را انتخاب کنید، توانایی اجرای محدودیت را از دست می‌دهید.

برای کنترل دقیق‌تر، می‌توانید محدودیت‌های پراکندگی توپولوژی را برای توزیع
پادها در دامنه‌های توپولوژی مختلف مشخص کنید - برای دستیابی به یا قابلیت دسترسی بالا یا
صرفه‌جویی در هزینه. این همچنین می‌تواند در به‌روزرسانی غلتکی بارکاری‌ها و افزایش تعداد
نسخه‌ها به طور روان کمک کند.

برای جزئیات بیشتر، بخش
[انگیزه](https://github.com/kubernetes/enhancements/tree/master/keps/sig-scheduling/895-pod-topology-spread#motivation)
پیشنهاد بهبود در مورد محدودیت‌های پراکندگی توپولوژی پاد را ببینید.

## محدودیت‌های شناخته شده

- هیچ تضمینی وجود ندارد که محدودیت‌ها پس از حذف پادها باقی بمانند. به عنوان مثال، کاهش مقیاس یک استقرار ممکن است منجر به توزیع نابرابر پادها شود.

  می‌توانید از ابزاری مانند [Descheduler](https://github.com/kubernetes-sigs/descheduler) برای متعادل‌سازی مجدد توزیع پادها استفاده کنید.
- پادهایی که بر روی گره‌های آلوده قرار می‌گیرند، محترم شمرده می‌شوند.
  به [Issue 80921](https://github.com/kubernetes/kubernetes/issues/80921) مراجعه کنید.
- زمان‌بند دانش قبلی از تمامی مناطق یا دامنه‌های توپولوژی دیگر که یک کلاستر دارد ندارد. آنها از گره‌های موجود در
  کلاستر تعیین می‌شوند. این می‌تواند منجر به مشکل در کلاسترهای اتواسکیل شده شود، زمانی که یک گروه گره (یا
  گروه گره) به صفر گره مقیاس داده می‌شود و انتظار دارید که کلاستر بالا رود،
  زیرا در این حالت، آن دامنه‌های توپولوژی تا زمانی که حداقل یک گره در آنها وجود داشته باشد، در نظر گرفته نمی‌شوند.

  می‌توانید با استفاده از ابزاری برای اتواسکیل کردن کلاستر که از محدودیت‌های پراکندگی توپولوژی پاد آگاه است و همچنین از مجموعه کلی دامنه‌های توپولوژی آگاه است، این مشکل را دور بزنید.

## {{% heading "whatsnext" %}}

- مقاله وبلاگ [معرفی PodTopologySpread](/blog/2020/05/introducing-podtopologyspread/)
  `maxSkew` را با جزئیات بیشتری توضیح می‌دهد و همچنین مثال‌های پیشرفته‌ای از استفاده را پوشش می‌دهد.
- بخش [زمان‌بندی](/docs/reference/kubernetes-api/workload-resources/pod-v1/#scheduling) از
  مرجع API برای پاد را بخوانید.
