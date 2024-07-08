---
title: مدیریت اعلامی اشیاء Kubernetes با استفاده از Kustomize
content_type: task
weight: 20
---

<!-- overview -->

[Kustomize](https://github.com/kubernetes-sigs/kustomize) یک ابزار مستقل برای سفارشی‌سازی اشیاء Kubernetes از طریق یک [فایل kustomization](https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#kustomization) است.

از نسخه 1.14، `kubectl` نیز از مدیریت اشیاء Kubernetes با استفاده از یک فایل kustomization پشتیبانی می‌کند. برای مشاهده منابعی که در یک دایرکتوری حاوی فایل kustomization قرار دارند، فرمان زیر را اجرا کنید:

```shell
kubectl kustomize <kustomization_directory>
```

برای اعمال این منابع، `kubectl apply` را با پرچم `--kustomize` یا `-k` اجرا کنید:

```shell
kubectl apply -k <kustomization_directory>
```


## {{% heading "prerequisites" %}}

نصب [`kubectl`](/docs/tasks/tools/).

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!-- steps -->

## مرور کلی Kustomize

Kustomize ابزاری برای سفارشی‌سازی پیکربندی‌های Kubernetes است. این ابزار ویژگی‌های زیر را برای مدیریت فایل‌های پیکربندی برنامه ارائه می‌دهد:

* تولید منابع از منابع دیگر
* تنظیم فیلدهای عمومی برای منابع
* ترکیب و سفارشی‌سازی مجموعه‌ای از منابع

### تولید منابع

ConfigMap‌ها و Secret‌ها داده‌های پیکربندی یا حساس را نگه می‌دارند که توسط اشیاء دیگر Kubernetes مانند Pods استفاده می‌شوند. منبع اصلی ConfigMap‌ها یا Secret‌ها معمولاً خارجی به خوشه است، مانند یک فایل `.properties` یا یک فایل کلید SSH.
Kustomize دارای `secretGenerator` و `configMapGenerator` است که Secret و ConfigMap را از فایل‌ها یا متغیرها تولید می‌کنند.

#### configMapGenerator

برای تولید یک ConfigMap از یک فایل، یک ورودی به لیست `files` در `configMapGenerator` اضافه کنید. در اینجا مثالی از تولید یک ConfigMap با یک آیتم داده از یک فایل `.properties` آمده است:

```shell
# ایجاد یک فایل application.properties
cat <<EOF >application.properties
FOO=Bar
EOF

cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: example-configmap-1
  files:
  - application.properties
EOF
```

ConfigMap تولید شده را می‌توان با فرمان زیر بررسی کرد:

```shell
kubectl kustomize ./
```

ConfigMap تولید شده به صورت زیر است:

```yaml
apiVersion: v1
data:
  application.properties: |
    FOO=Bar
kind: ConfigMap
metadata:
  name: example-configmap-1-8mbdf7882g
```

برای تولید یک ConfigMap از یک فایل env، یک ورودی به لیست `envs` در `configMapGenerator` اضافه کنید. در اینجا مثالی از تولید یک ConfigMap با یک آیتم داده از یک فایل `.env` آمده است:

```shell
# ایجاد یک فایل .env
cat <<EOF >.env
FOO=Bar
EOF

cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: example-configmap-1
  envs:
  - .env
EOF
```

ConfigMap تولید شده را می‌توان با فرمان زیر بررسی کرد:

```shell
kubectl kustomize ./
```

ConfigMap تولید شده به صورت زیر است:

```yaml
apiVersion: v1
data:
  FOO: Bar
kind: ConfigMap
metadata:
  name: example-configmap-1-42cfbf598f
```

{{< note >}}
هر متغیر در فایل `.env` به یک کلید جداگانه در ConfigMap تولید شده تبدیل می‌شود. این متفاوت از مثال قبلی است که یک فایل به نام `application.properties` (و تمام ورودی‌های آن) را به عنوان مقدار یک کلید تکی جاسازی می‌کند.
{{< /note >}}

ConfigMap‌ها همچنین می‌توانند از جفت کلید-مقدارهای متناظر تولید شوند. برای تولید یک ConfigMap از یک جفت کلید-مقدار متناظر، یک ورودی به لیست `literals` در `configMapGenerator` اضافه کنید. در اینجا مثالی از تولید یک ConfigMap با یک آیتم داده از یک جفت کلید-مقدار آمده است:

```shell
cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: example-configmap-2
  literals:
  - FOO=Bar
EOF
```

ConfigMap تولید شده را می‌توان با فرمان زیر بررسی کرد:

```shell
kubectl kustomize ./
```

ConfigMap تولید شده به صورت زیر است:

```yaml
apiVersion: v1
data:
  FOO: Bar
kind: ConfigMap
metadata:
  name: example-configmap-2-g2hdhfc6tk
```

برای استفاده از یک ConfigMap تولید شده در یک Deployment، آن را با نام `configMapGenerator` مرجع دهید. Kustomize به صورت خودکار این نام را با نام تولید شده جایگزین می‌کند.

این یک مثال از Deployment است که از یک ConfigMap تولید شده استفاده می‌کند:

```yaml
# ایجاد یک فایل application.properties
cat <<EOF >application.properties
FOO=Bar
EOF

cat <<EOF >deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app
        volumeMounts:
        - name: config
          mountPath: /config
      volumes:
      - name: config
        configMap:
          name: example-configmap-1
EOF

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
configMapGenerator:
- name: example-configmap-1
  files:
  - application.properties
EOF
```

ConfigMap و Deployment را تولید کنید:

```shell
kubectl kustomize ./
```

Deployment تولید شده به نام ConfigMap تولید شده ارجاع می‌دهد:

```yaml
apiVersion: v1
data:
  application.properties: |
    FOO=Bar
kind: ConfigMap
metadata:
  name: example-configmap-1-g4hk9g2ff8
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: my-app
  name: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - image: my-app
        name: app
        volumeMounts:
        - mountPath: /config
          name: config
      volumes:
      - configMap:
          name: example-configmap-1-g4hk9g2ff8
        name: config
```#### secretGenerator

شما می‌توانید Secrets را از فایل‌ها یا جفت کلید-مقدارهای متنی تولید کنید. برای تولید یک Secret از یک فایل، یک ورودی به لیست `files` در `secretGenerator` اضافه کنید. در اینجا مثالی از تولید یک Secret با یک آیتم داده از یک فایل آمده است:

```shell
# ایجاد یک فایل password.txt
cat <<EOF >./password.txt
username=admin
password=secret
EOF

cat <<EOF >./kustomization.yaml
secretGenerator:
- name: example-secret-1
  files:
  - password.txt
EOF
```

Secret تولید شده به صورت زیر است:

```yaml
apiVersion: v1
data:
  password.txt: dXNlcm5hbWU9YWRtaW4KcGFzc3dvcmQ9c2VjcmV0Cg==
kind: Secret
metadata:
  name: example-secret-1-t2kt65hgtb
type: Opaque
```

برای تولید یک Secret از یک جفت کلید-مقدار متنی، یک ورودی به لیست `literals` در `secretGenerator` اضافه کنید. در اینجا مثالی از تولید یک Secret با یک آیتم داده از یک جفت کلید-مقدار آمده است:

```shell
cat <<EOF >./kustomization.yaml
secretGenerator:
- name: example-secret-2
  literals:
  - username=admin
  - password=secret
EOF
```

Secret تولید شده به صورت زیر است:

```yaml
apiVersion: v1
data:
  password: c2VjcmV0
  username: YWRtaW4=
kind: Secret
metadata:
  name: example-secret-2-t52t6g96d8
type: Opaque
```

مانند ConfigMap‌ها، Secrets تولید شده را می‌توان در Deployments با ارجاع به نام `secretGenerator` استفاده کرد:

```shell
# ایجاد یک فایل password.txt
cat <<EOF >./password.txt
username=admin
password=secret
EOF

cat <<EOF >deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app
        volumeMounts:
        - name: password
          mountPath: /secrets
      volumes:
      - name: password
        secret:
          secretName: example-secret-1
EOF

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
secretGenerator:
- name: example-secret-1
  files:
  - password.txt
EOF
```

#### generatorOptions

ConfigMap‌ها و Secrets تولید شده دارای یک پسوند هش محتوایی هستند که اطمینان می‌دهد یک ConfigMap یا Secret جدید در صورت تغییر محتوا تولید می‌شود. برای غیرفعال کردن این رفتار، می‌توانید از `generatorOptions` استفاده کنید. علاوه بر این، می‌توانید گزینه‌های مشترک برای ConfigMap‌ها و Secrets تولید شده را نیز مشخص کنید.

```shell
cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: example-configmap-3
  literals:
  - FOO=Bar
generatorOptions:
  disableNameSuffixHash: true
  labels:
    type: generated
  annotations:
    note: generated
EOF
```

فرمان `kubectl kustomize ./` را اجرا کنید تا ConfigMap تولید شده را مشاهده کنید:

```yaml
apiVersion: v1
data:
  FOO: Bar
kind: ConfigMap
metadata:
  annotations:
    note: generated
  labels:
    type: generated
  name: example-configmap-3
```

### تنظیم فیلدهای عمومی

بسیار رایج است که فیلدهای عمومی را برای تمام منابع Kubernetes در یک پروژه تنظیم کنید.
برخی موارد استفاده از تنظیم فیلدهای عمومی:

* تنظیم همان namespace برای تمام منابع
* افزودن همان پیشوند یا پسوند نام
* افزودن همان مجموعه برچسب‌ها
* افزودن همان مجموعه توضیحات

در اینجا مثالی آمده است:

```shell
# ایجاد یک فایل deployment.yaml
cat <<EOF >./deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
EOF

cat <<EOF >./kustomization.yaml
namespace: my-namespace
namePrefix: dev-
nameSuffix: "-001"
commonLabels:
  app: bingo
commonAnnotations:
  oncallPager: 800-555-1212
resources:
- deployment.yaml
EOF
```

فرمان `kubectl kustomize ./` را اجرا کنید تا مشاهده کنید که این فیلدها در منبع Deployment تنظیم شده‌اند:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    oncallPager: 800-555-1212
  labels:
    app: bingo
  name: dev-nginx-deployment-001
  namespace: my-namespace
spec:
  selector:
    matchLabels:
      app: bingo
  template:
    metadata:
      annotations:
        oncallPager: 800-555-1212
      labels:
        app: bingo
    spec:
      containers:
      - image: nginx
        name: nginx
```### ترکیب و سفارشی‌سازی منابع

ترکیب مجموعه‌ای از منابع در یک پروژه و مدیریت آن‌ها در یک فایل یا دایرکتوری مشترک، رایج است.
Kustomize امکان ترکیب منابع از فایل‌های مختلف و اعمال پچ‌ها یا دیگر سفارشی‌سازی‌ها به آن‌ها را فراهم می‌کند.

#### ترکیب

Kustomize از ترکیب منابع مختلف پشتیبانی می‌کند. فیلد `resources` در فایل `kustomization.yaml` لیستی از منابعی که باید در یک پیکربندی گنجانده شوند را تعریف می‌کند. مسیر فایل پیکربندی هر منبع را در لیست `resources` تنظیم کنید.
در اینجا مثالی از یک برنامه NGINX شامل یک Deployment و یک Service آمده است:

```shell
# ایجاد فایل deployment.yaml
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

# ایجاد فایل service.yaml
cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
EOF

# ایجاد فایل kustomization.yaml و ترکیب آن‌ها
cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
- service.yaml
EOF
```

منابع از `kubectl kustomize ./` شامل هر دو شیء Deployment و Service هستند.

#### سفارشی‌سازی

می‌توان از پچ‌ها برای اعمال سفارشی‌سازی‌های مختلف به منابع استفاده کرد. Kustomize از مکانیزم‌های پچ‌ مختلفی از طریق `patchesStrategicMerge` و `patchesJson6902` پشتیبانی می‌کند. `patchesStrategicMerge` لیستی از مسیر فایل‌ها است. هر فایل باید به [پچ ترکیبی استراتژیک](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md) تبدیل شود. نام‌های داخل پچ‌ها باید با نام منابعی که از قبل بارگذاری شده‌اند، مطابقت داشته باشد. پچ‌های کوچک که یک کار را انجام می‌دهند توصیه می‌شود. به عنوان مثال، یک پچ برای افزایش تعداد رپلیکای دیپلویمنت و پچ دیگری برای تنظیم محدودیت حافظه ایجاد کنید.

```shell
# ایجاد فایل deployment.yaml
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

# ایجاد پچ increase_replicas.yaml
cat <<EOF > increase_replicas.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
EOF

# ایجاد پچ دیگر set_memory.yaml
cat <<EOF > set_memory.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  template:
    spec:
      containers:
      - name: my-nginx
        resources:
          limits:
            memory: 512Mi
EOF

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
patchesStrategicMerge:
- increase_replicas.yaml
- set_memory.yaml
EOF
```

فرمان `kubectl kustomize ./` را اجرا کنید تا دیپلویمنت را مشاهده کنید:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - image: nginx
        name: my-nginx
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: 512Mi
```

تمام منابع یا فیلدها از پچ‌های ترکیبی استراتژیک پشتیبانی نمی‌کنند. برای پشتیبانی از تغییر فیلدهای دلخواه در منابع دلخواه، Kustomize امکان اعمال [پچ JSON](https://tools.ietf.org/html/rfc6902) را از طریق `patchesJson6902` فراهم می‌کند.
برای یافتن منبع صحیح برای یک پچ Json، گروه، نسخه، نوع و نام آن منبع باید در `kustomization.yaml` مشخص شود. به عنوان مثال، افزایش تعداد رپلیکای یک شیء Deployment را می‌توان از طریق `patchesJson6902` نیز انجام داد.

```shell
# ایجاد فایل deployment.yaml
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

# ایجاد پچ json
cat <<EOF > patch.yaml
- op: replace
  path: /spec/replicas
  value: 3
EOF

# ایجاد فایل kustomization.yaml
cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml

patchesJson6902:
- target:
    group: apps
    version: v1
    kind: Deployment
    name: my-nginx
  path: patch.yaml
EOF
```

فرمان `kubectl kustomize ./` را اجرا کنید تا مشاهده کنید که فیلد `replicas` به‌روزرسانی شده است:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - image: nginx
        name: my-nginx
        ports:
        - containerPort: 80
```

علاوه بر پچ‌ها، Kustomize امکان سفارشی‌سازی تصاویر کانتینر یا تزریق مقادیر فیلد از اشیاء دیگر به کانتینرها را بدون ایجاد پچ فراهم می‌کند. به عنوان مثال، شما می‌توانید تصویر مورد استفاده در داخل کانتینرها را با مشخص کردن تصویر جدید در فیلد `images` در `kustomization.yaml` تغییر دهید.

```shell
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
images:
- name: nginx
  newName: my.image.registry/nginx
  newTag: 1.4.0
EOF
```
فرمان `kubectl kustomize ./` را اجرا کنید تا مشاهده کنید که تصویر مورد استفاده به‌روزرسانی شده است:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - image: my.image.registry/nginx:1.4.0
        name: my-nginx
        ports:
        - containerPort: 80
```
### ترکیب و سفارشی‌سازی منابع

در یک پروژه، معمول است که مجموعه‌ای از منابع را ترکیب کرده و آن‌ها را در یک فایل یا دایرکتوری مدیریت کنید.
Kustomize امکان ترکیب منابع از فایل‌های مختلف و اعمال تغییرات یا سفارشی‌سازی‌های دیگر را به شما می‌دهد.

#### ترکیب

Kustomize از ترکیب منابع مختلف پشتیبانی می‌کند. فیلد `resources` در فایل `kustomization.yaml` لیستی از منابعی که باید در یک پیکربندی گنجانده شوند را تعریف می‌کند. مسیر فایل پیکربندی منابع را در لیست `resources` تنظیم کنید.
در اینجا یک نمونه از یک برنامه NGINX که شامل یک Deployment و یک Service است آورده شده است:

```shell
# ایجاد فایل deployment.yaml
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

# ایجاد فایل service.yaml
cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
EOF

# ایجاد فایل kustomization.yaml برای ترکیب آن‌ها
cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
- service.yaml
EOF
```

منابعی که از `kubectl kustomize ./` به‌دست‌آمده شامل هر دو شیء Deployment و Service است.

#### سفارشی‌سازی

می‌توان از پچ‌ها برای اعمال سفارشی‌سازی‌های مختلف به منابع استفاده کرد. Kustomize از مکانیزم‌های مختلف پچینگ از طریق `patchesStrategicMerge` و `patchesJson6902` پشتیبانی می‌کند. `patchesStrategicMerge` لیستی از مسیرهای فایل است. هر فایل باید به یک [پچ ترکیب استراتژیک](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md) منتهی شود. نام‌های داخل پچ‌ها باید با نام‌های منابعی که قبلاً بارگذاری شده‌اند مطابقت داشته باشند. توصیه می‌شود پچ‌های کوچک که فقط یک کار انجام می‌دهند ایجاد کنید. برای مثال، یک پچ برای افزایش تعداد رپلیکاهای دیپلویمنت و دیگری برای تنظیم محدودیت حافظه ایجاد کنید.

```shell
# ایجاد فایل deployment.yaml
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

# ایجاد پچ increase_replicas.yaml
cat <<EOF > increase_replicas.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
EOF

# ایجاد پچ دیگری set_memory.yaml
cat <<EOF > set_memory.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  template:
    spec:
      containers:
      - name: my-nginx
        resources:
          limits:
            memory: 512Mi
EOF

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
patchesStrategicMerge:
- increase_replicas.yaml
- set_memory.yaml
EOF
```

اجرای `kubectl kustomize ./` برای مشاهده دیپلویمنت:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - image: nginx
        name: my-nginx
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: 512Mi
```

همه منابع یا فیلدها از پچ‌های ترکیب استراتژیک پشتیبانی نمی‌کنند. برای پشتیبانی از تغییر فیلدهای دلخواه در منابع دلخواه، Kustomize اعمال [پچ JSON](https://tools.ietf.org/html/rfc6902) از طریق `patchesJson6902` را ارائه می‌دهد.
برای پیدا کردن منبع صحیح برای یک پچ JSON، گروه، نسخه، نوع و نام آن منبع باید در `kustomization.yaml` مشخص شود. برای مثال، افزایش تعداد رپلیکاهای یک شیء Deployment نیز می‌تواند از طریق `patchesJson6902` انجام شود.

```shell
# ایجاد فایل deployment.yaml
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

# ایجاد پچ JSON
cat <<EOF > patch.yaml
- op: replace
  path: /spec/replicas
  value: 3
EOF

# ایجاد فایل kustomization.yaml
cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml

patchesJson6902:
- target:
    group: apps
    version: v1
    kind: Deployment
    name: my-nginx
  path: patch.yaml
EOF
```

اجرای `kubectl kustomize ./` برای دیدن به‌روزرسانی فیلد `replicas`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - image: nginx
        name: my-nginx
        ports:
        - containerPort: 80
```

علاوه بر پچ‌ها، Kustomize همچنین امکان سفارشی‌سازی تصاویر کانتینر یا تزریق مقادیر فیلدها از اشیاء دیگر به کانتینرها را بدون ایجاد پچ‌ها ارائه می‌دهد. برای مثال، می‌توانید تصویر استفاده‌شده در داخل کانتینرها را با مشخص کردن تصویر جدید در فیلد `images` در `kustomization.yaml` تغییر دهید.

```shell
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
images:
- name: nginx
  newName: my.image.registry/nginx
  newTag: 1.4.0
EOF
```
اجرای `kubectl kustomize ./` برای مشاهده به‌روزرسانی تصویر:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - image: my.image.registry/nginx:1.4.0
        name: my-nginx
        ports:
        - containerPort: 80
```

### تزریق مقادیر پیکربندی از سایر اشیاء

گاهی اوقات، برنامه‌ای که در یک پاد اجرا می‌شود ممکن است به استفاده از مقادیر پیکربندی از سایر اشیاء نیاز داشته باشد. برای مثال، یک پاد از یک شیء Deployment نیاز دارد تا نام سرویس مربوطه را از متغیرهای محیطی یا به عنوان یک آرگومان فرمان بخواند. از آنجایی که نام سرویس ممکن است با اضافه شدن `namePrefix` یا `nameSuffix` در فایل `kustomization.yaml` تغییر کند، توصیه نمی‌شود نام سرویس را به‌صورت سخت‌کد در آرگومان فرمان قرار دهید. برای این منظور، Kustomize می‌تواند نام سرویس را از طریق `vars` به کانتینرها تزریق کند.

```shell
# ایجاد فایل deployment.yaml (با نقل قول برای تعیین‌کننده here doc)
cat <<'EOF' > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx


spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        command: ["start", "--host", "$(MY_SERVICE_NAME)"]
EOF

# ایجاد فایل service.yaml
cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
EOF

cat <<EOF >./kustomization.yaml
namePrefix: dev-
nameSuffix: "-001"

resources:
- deployment.yaml
- service.yaml

vars:
- name: MY_SERVICE_NAME
  objref:
    kind: Service
    name: my-nginx
    apiVersion: v1
EOF
```

اجرای `kubectl kustomize ./` برای دیدن نام سرویس تزریق‌شده به کانتینرها به صورت `dev-my-nginx-001`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dev-my-nginx-001
spec:
  replicas: 2
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - command:
        - start
        - --host
        - dev-my-nginx-001
        image: nginx
        name: my-nginx
```

### مبانی و پوشش‌ها

Kustomize دارای مفاهیم **مبانی** و **پوشش‌ها** است. **مبنا** یک دایرکتوری با `kustomization.yaml` است که مجموعه‌ای از منابع و سفارشی‌سازی‌های مربوطه را شامل می‌شود. یک مبنا می‌تواند دایرکتوری محلی یا دایرکتوری از یک مخزن راه دور باشد، به شرطی که یک `kustomization.yaml` درون آن موجود باشد. یک **پوشش** دایرکتوری‌ای با `kustomization.yaml` است که به سایر دایرکتوری‌های kustomization به عنوان مبناهای خود اشاره می‌کند. یک مبنا هیچ اطلاعی از پوشش ندارد و می‌تواند در چندین پوشش استفاده شود. یک پوشش ممکن است چندین مبنا داشته باشد و همه منابع را از مبناها ترکیب کرده و ممکن است سفارشی‌سازی‌هایی نیز بر روی آن‌ها اعمال کند.

در اینجا یک مثال از یک مبنا آورده شده است:

```shell
# ایجاد یک دایرکتوری برای نگهداری مبنا
mkdir base
# ایجاد فایل base/deployment.yaml
cat <<EOF > base/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
EOF

# ایجاد فایل base/service.yaml
cat <<EOF > base/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
EOF
# ایجاد فایل base/kustomization.yaml
cat <<EOF > base/kustomization.yaml
resources:
- deployment.yaml
- service.yaml
EOF
```

این مبنا می‌تواند در چندین پوشش استفاده شود. می‌توانید در پوشش‌های مختلف `namePrefix` یا سایر فیلدهای مشترک را اضافه کنید. در اینجا دو پوشش که از همان مبنا استفاده می‌کنند آورده شده است:

```shell
mkdir dev
cat <<EOF > dev/kustomization.yaml
resources:
- ../base
namePrefix: dev-
EOF

mkdir prod
cat <<EOF > prod/kustomization.yaml
resources:
- ../base
namePrefix: prod-
EOF
```

### نحوه اعمال/مشاهده/حذف اشیاء با استفاده از Kustomize

از `--kustomize` یا `-k` در دستورات `kubectl` برای تشخیص منابع مدیریت‌شده توسط `kustomization.yaml` استفاده کنید.
توجه داشته باشید که `-k` باید به یک دایرکتوری kustomization اشاره کند، مانند

```shell
kubectl apply -k <kustomization directory>/
```

با توجه به `kustomization.yaml` زیر،

```shell
# ایجاد فایل deployment.yaml
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

# ایجاد فایل kustomization.yaml
cat <<EOF >./kustomization.yaml
namePrefix: dev-
commonLabels:
  app: my-nginx
resources:
- deployment.yaml
EOF
```

دستور زیر را برای اعمال شیء Deployment `dev-my-nginx` اجرا کنید:

```shell
> kubectl apply -k ./
deployment.apps/dev-my-nginx created
```

یکی از دستورات زیر را برای مشاهده شیء Deployment `dev-my-nginx` اجرا کنید:

```shell
kubectl get -k ./
```

```shell
kubectl describe -k ./
```

دستور زیر را برای مقایسه شیء Deployment `dev-my-nginx` با وضعیتی که کلاستر در صورت اعمال مانفیست خواهد داشت، اجرا کنید:

```shell
kubectl diff -k ./
```

دستور زیر را برای حذف شیء Deployment `dev-my-nginx` اجرا کنید:

```shell
> kubectl delete -k ./
deployment.apps "dev-my-nginx" deleted
```

### لیست ویژگی‌های Kustomize

| فیلد                 | نوع                                                                                                         | توضیح                                                                        |
|-----------------------|--------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| namespace             | string                                                                                                       | افزودن namespace به همه منابع                                                     |
| namePrefix            | string                                                                                                       | مقدار این فیلد به نام همه منابع اضافه می‌شود                     |
| nameSuffix            | string                                                                                                       | مقدار این فیلد به نام همه منابع اضافه می‌شود                      |
| commonLabels          | map[string]string                                                                                            | برچسب‌هایی که به همه منابع و انتخاب‌کننده‌ها اضافه می‌شود                                       |
| commonAnnotations     | map[string]string                                                                                            | حاشیه‌نویسی‌هایی که به همه منابع اضافه می‌شود                                                |
| resources             | []string                                                                                                     | هر ورودی در این لیست باید به یک فایل پیکربندی منابع موجود منتهی شود    |
| configMapGenerator    | [][ConfigMapArgs](https://github.com/kubernetes-sigs/kustomize/blob/master/api/types/configmapargs.go#L7)    | هر ورودی در این لیست یک ConfigMap تولید می‌کند                                      |
| secretGenerator       | [][SecretArgs](https://github.com/kubernetes-sigs/kustomize/blob/master/api/types/secretargs.go#L7)          | هر ورودی در این لیست یک Secret تولید می‌کند                                         |
| generatorOptions      | [GeneratorOptions](https://github.com/kubernetes-sigs/kustomize/blob/master/api/types/generatoroptions.go#L7) | تغییر رفتارهای همه تولیدکننده‌های ConfigMap و Secret                             |
| bases                 | []string                                                                                                     | هر ورودی در این لیست باید به یک دایرکتوری حاوی یک فایل kustomization.yaml منتهی شود |
| patchesStrategicMerge | []string                                                                                                     | هر ورودی در این لیست باید به یک پچ ترکیب استراتژیک یک شیء Kubernetes منتهی شود |
| patchesJson6902       | [][Patch](https://github.com/kubernetes-sigs/kustomize/blob/master/api/types/patch.go#L10)                   | هر ورودی در این لیست باید به یک شیء Kubernetes و یک پچ JSON منتهی شود     |
| vars                  | [][Var](https://github.com/kubernetes-sigs/kustomize/blob/master/api/types/var.go#L19)                       | هر ورودی برای گرفتن متن از فیلد یک منبع است                            |
| images                | [][Image](https://github.com/kubernetes-sigs/kustomize/blob/master/api/types/image.go#L8)                    | هر ورودی برای تغییر نام، برچسب‌ها و/یا هضم یک تصویر بدون ایجاد پچ‌ها است |
| configurations        | []string                                                                                                     | هر ورودی در این لیست باید به یک فایل حاوی [پیکربندی‌های ترنسفورمر Kustomize](https://github.com/kubernetes-sigs/kustomize/tree/master/examples/transformerconfigs) منتهی شود |
| crds                  | []string                                                                                                     | هر ورودی در این لیست باید به یک فایل تعریف OpenAPI برای انواع Kubernetes منتهی شود |


### منابع بیشتر

* [Kustomize](https://github.com/kubernetes-sigs/kustomize)
* [کتاب Kubectl](https://kubectl.docs.kubernetes.io)
* [مرجع دستورات Kubectl](/docs/reference/generated/kubectl/kubectl-commands/)
* [مرجع API Kubernetes](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)