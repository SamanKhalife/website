---
title: مهاجرت اشیاء Kubernetes با استفاده از مهاجرت نسخه ذخیره‌سازی
reviewers:
- deads2k
- jpbetz
- enj
- nilekhc
content_type: task
min-kubernetes-server-version: v1.30
weight: 60
---

<!-- overview -->

{{< feature-state feature_gate_name="StorageVersionMigrator" >}}

Kubernetes برای پشتیبانی از برخی فعالیت‌های نگهداری مرتبط با ذخیره‌سازی در حال استراحت به داده‌های API که فعالانه بازنویسی می‌شوند، متکی است. دو مثال برجسته عبارتند از: طرح نسخه‌بندی شده منابع ذخیره‌شده (به عبارت دیگر، تغییر طرح ترجیحی ذخیره‌سازی از v1 به v2 برای یک منبع داده شده) و رمزگذاری در حالت استراحت (یعنی بازنویسی داده‌های منقضی شده بر اساس تغییر در نحوه رمزگذاری داده‌ها).

## {{% heading "prerequisites" %}}

نصب [`kubectl`](/docs/tasks/tools/#kubectl).

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!-- steps -->

## بازرمزگذاری اسرار Kubernetes با استفاده از مهاجرت نسخه ذخیره‌سازی

- برای شروع، [پیکربندی ارائه‌دهنده KMS](/docs/tasks/administer-cluster/kms-provider/) را برای رمزگذاری داده‌ها در حالت استراحت در etcd با استفاده از پیکربندی رمزگذاری زیر انجام دهید.

  ```yaml
  kind: EncryptionConfiguration
  apiVersion: apiserver.config.k8s.io/v1
  resources:
  - resources:
    - secrets
    providers:
    - aescbc:
      keys:
      - name: key1
        secret: c2VjcmV0IGlzIHNlY3VyZQ==
  ```

  اطمینان حاصل کنید که بارگذاری خودکار فایل پیکربندی رمزگذاری را با تنظیم `--encryption-provider-config-automatic-reload` بر روی true فعال کنید.

- ایجاد یک Secret با استفاده از kubectl.

  ```shell
  kubectl create secret generic my-secret --from-literal=key1=supersecret
  ```

- [تأیید](/docs/tasks/administer-cluster/kms-provider/#verifying-that-the-data-is-encrypted) کنید که داده‌های سریالیزه شده برای آن شیء Secret با `k8s:enc:aescbc:v1:key1` پیشوند شده است.

- فایل پیکربندی رمزگذاری را به صورت زیر به‌روزرسانی کنید تا کلید رمزگذاری چرخانده شود.

  ```yaml
  kind: EncryptionConfiguration
  apiVersion: apiserver.config.k8s.io/v1
  resources:
  - resources:
    - secrets
    providers:
    - aescbc:
      keys:
      - name: key2
        secret: c2VjcmV0IGlzIHNlY3VyZSwgaXMgaXQ/
    - aescbc:
      keys:
      - name: key1
        secret: c2VjcmV0IGlzIHNlY3VyZQ==
  ```

- برای اطمینان از اینکه Secret قبلاً ایجاد شده `my-secret` با کلید جدید `key2` بازرمزگذاری شده است، از _مهاجرت نسخه ذخیره‌سازی_ استفاده خواهید کرد.

- ایجاد یک فایل مانفیست StorageVersionMigration با نام `migrate-secret.yaml` به صورت زیر:

  ```yaml
  kind: StorageVersionMigration
  apiVersion: storagemigration.k8s.io/v1alpha1
  metadata:
    name: secrets-migration
  spec:
    resource:
      group: ""
      version: v1
      resource: secrets
  ```

  شیء را با استفاده از _kubectl_ به صورت زیر ایجاد کنید:

  ```shell
  kubectl apply -f migrate-secret.yaml
  ```

- مهاجرت اسرار را با بررسی `.status` از StorageVersionMigration نظارت کنید.
  یک مهاجرت موفق باید وضعیت `Succeeded` را به true تنظیم کند. شیء StorageVersionMigration را به صورت زیر دریافت کنید:

  ```shell
  kubectl get storageversionmigration.storagemigration.k8s.io/secrets-migration -o yaml
  ```

  خروجی مشابه زیر است:

  ```yaml
  kind: StorageVersionMigration
  apiVersion: storagemigration.k8s.io/v1alpha1
  metadata:
    name: secrets-migration
    uid: 628f6922-a9cb-4514-b076-12d3c178967c
    resourceVersion: "90"
    creationTimestamp: "2024-03-12T20:29:45Z"
  spec:
    resource:
      group: ""
      version: v1
      resource: secrets
  status:
    conditions:
    - type: Running
      status: "False"
      lastUpdateTime: "2024-03-12T20:29:46Z"
      reason: StorageVersionMigrationInProgress
    - type: Succeeded
      status: "True"
      lastUpdateTime: "2024-03-12T20:29:46Z"
      reason: StorageVersionMigrationSucceeded
    resourceVersion: "84"
  ```

- [تأیید](/docs/tasks/administer-cluster/kms-provider/#verifying-that-the-data-is-encrypted) کنید که Secret ذخیره شده اکنون با `k8s:enc:aescbc:v1:key2` پیشوند شده است.
## به‌روزرسانی طرح ذخیره‌سازی ترجیحی یک CRD

سناریویی را در نظر بگیرید که یک {{< glossary_tooltip term_id="CustomResourceDefinition" text="CustomResourceDefinition" >}}
(CRD) برای ارائه منابع سفارشی (CRs) ایجاد می‌شود و به عنوان طرح ذخیره‌سازی ترجیحی تنظیم می‌شود. زمانی که زمان معرفی نسخه v2 از CRD فرا می‌رسد، می‌توان آن را با استفاده از یک وب‌هوک تبدیل فقط برای ارائه اضافه کرد. این کار انتقال روان‌تری را ممکن می‌سازد که کاربران بتوانند CRها را با استفاده از طرح v1 یا v2 ایجاد کنند، با وب‌هوکی که برای انجام تبدیل طرح لازم بین آن‌ها وجود دارد. قبل از تنظیم نسخه v2 به عنوان طرح ذخیره‌سازی ترجیحی، مهم است که اطمینان حاصل کنید که همه CRهای موجود که به صورت v1 ذخیره شده‌اند به v2 مهاجرت کرده‌اند. این مهاجرت را می‌توان از طریق _مهاجرت نسخه ذخیره‌سازی_ انجام داد تا همه CRها از v1 به v2 مهاجرت کنند.

- یک مانفیست برای CRD به نام `test-crd.yaml` به صورت زیر ایجاد کنید:

  ```yaml
  apiVersion: apiextensions.k8s.io/v1
  kind: CustomResourceDefinition
  metadata:
    name: selfierequests.stable.example.com
  spec:
    group: stable.example.com
    names:
      plural: SelfieRequests
      singular: SelfieRequest
      kind: SelfieRequest
      listKind: SelfieRequestList
    scope: Namespaced
    versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            hostPort:
              type: string
    conversion:
      strategy: Webhook
      webhook:
        clientConfig:
          url: "https://127.0.0.1:9443/crdconvert"
          caBundle: <CABundle info>
      conversionReviewVersions:
      - v1
      - v2
  ```

  ایجاد CRD با استفاده از kubectl:

  ```shell
  kubectl apply -f test-crd.yaml
  ```

- یک مانفیست برای یک testcrd نمونه ایجاد کنید. نام مانفیست را `cr1.yaml` قرار داده و از این محتویات استفاده کنید:

  ```yaml
  apiVersion: stable.example.com/v1
  kind: SelfieRequest
  metadata:
    name: cr1
    namespace: default
  ```

  ایجاد CR با استفاده از kubectl:

  ```shell
  kubectl apply -f cr1.yaml
  ```

- تأیید کنید که CR به صورت v1 نوشته و ذخیره شده است با دریافت شیء از etcd.

  ```shell
  ETCDCTL_API=3 etcdctl get /kubernetes.io/stable.example.com/testcrds/default/cr1 [...] | hexdump -C
  ```

  که `[...]` شامل آرگومان‌های اضافی برای اتصال به سرور etcd است.

- به‌روزرسانی CRD `test-crd.yaml` برای شامل شدن نسخه v2 برای ارائه و ذخیره‌سازی و v1 به عنوان تنها ارائه‌شده، به صورت زیر:

  ```yaml
  apiVersion: apiextensions.k8s.io/v1
  kind: CustomResourceDefinition
  metadata:
    name: selfierequests.stable.example.com
  spec:
    group: stable.example.com
    names:
      plural: SelfieRequests
      singular: SelfieRequest
      kind: SelfieRequest
      listKind: SelfieRequestList
    scope: Namespaced
    versions:
      - name: v2
        served: true
        storage: true
        schema:
          openAPIV3Schema:
            type: object
            properties:
              host:
                type: string
              port:
                type: string
      - name: v1
        served: true
        storage: false
        schema:
          openAPIV3Schema:
            type: object
            properties:
              hostPort:
                type: string
    conversion:
      strategy: Webhook
      webhook:
        clientConfig:
          url: "https://127.0.0.1:9443/crdconvert"
          caBundle: <CABundle info>
        conversionReviewVersions:
          - v1
          - v2
  ```

  به‌روزرسانی CRD با استفاده از kubectl:

  ```shell
  kubectl apply -f test-crd.yaml
  ```

- ایجاد یک فایل منبع CR با نام `cr2.yaml` به صورت زیر:

  ```yaml
  apiVersion: stable.example.com/v2
  kind: SelfieRequest
  metadata:
    name: cr2
    namespace: default
  ```

- ایجاد CR با استفاده از kubectl:

  ```shell
  kubectl apply -f cr2.yaml
  ```

- تأیید کنید که CR به صورت v2 نوشته و ذخیره شده است با دریافت شیء از etcd.

  ```shell
  ETCDCTL_API=3 etcdctl get /kubernetes.io/stable.example.com/testcrds/default/cr2 [...] | hexdump -C
  ```

  که `[...]` شامل آرگومان‌های اضافی برای اتصال به سرور etcd است.

- یک مانفیست StorageVersionMigration با نام `migrate-crd.yaml` ایجاد کنید، با محتویات به صورت زیر:

  ```yaml
  kind: StorageVersionMigration
  apiVersion: storagemigration.k8s.io/v1alpha1
  metadata:
    name: crdsvm
  spec:
    resource:
      group: stable.example.com
      version: v1
      resource: SelfieRequest
  ```

  ایجاد شیء با استفاده از _kubectl_ به صورت زیر:

  ```shell
  kubectl apply -f migrate-crd.yaml
  ```

- مهاجرت اسرار را با استفاده از وضعیت نظارت کنید. مهاجرت موفق باید وضعیت `Succeeded` را در فیلد وضعیت به "True" تنظیم کند. دریافت منبع مهاجرت به صورت زیر:

  ```shell
  kubectl get storageversionmigration.storagemigration.k8s.io/crdsvm -o yaml
  ```

  خروجی مشابه زیر است:

  ```yaml
  kind: StorageVersionMigration
  apiVersion: storagemigration.k8s.io/v1alpha1
  metadata:
    name: crdsvm
    uid: 13062fe4-32d7-47cc-9528-5067fa0c6ac8
    resourceVersion: "111"
    creationTimestamp: "2024-03-12T22:40:01Z"
  spec:
    resource:
      group: stable.example.com
      version: v1
      resource: testcrds
  status:
    conditions:
      - type: Running
        status: "False"
        lastUpdateTime: "2024-03-12T22:40:03Z"
        reason: StorageVersionMigrationInProgress
      - type: Succeeded
        status: "True"
        lastUpdateTime: "2024-03-12T22:40:03Z"
        reason: StorageVersionMigrationSucceeded
    resourceVersion: "106"
  ```

- تأیید کنید که `cr1` ایجاد شده قبلی اکنون به صورت v2 نوشته و ذخیره شده است با دریافت شیء از etcd.

  ```shell
  ETCDCTL_API=3 etcdctl get /kubernetes.io/stable.example.com/testcrds/default/cr1 [...] | hexdump -C
  ```

  که `[...]` شامل آرگومان‌های اضافی برای اتصال به سرور etcd است.
