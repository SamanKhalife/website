---
title: انتخاب‌گرهای فیلد
content_type: concept
weight: 70
---

انتخاب‌گرهای فیلد به شما اجازه می‌دهند که بر اساس مقدار یک یا چند فیلد منبع، اشیاء Kubernetes را انتخاب کنید. در ادامه نمونه‌هایی از پرس‌وجوهای انتخاب‌گر فیلد آورده شده است:

- `metadata.name=my-service`
- `metadata.namespace!=default`
- `status.phase=Pending`

این دستور `kubectl` همه Pods را انتخاب می‌کند که مقدار فیلد [`status.phase`](/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase) آن `Running` باشد:

```shell
kubectl get pods --field-selector status.phase=Running
```

{{< note >}}
انتخاب‌گرهای فیلد در اصل *فیلترهای* منابع هستند. به طور پیش‌فرض، هیچ انتخاب‌گر یا فیلتری اعمال نمی‌شود، به این معنی که همه منابع از نوع مشخص شده انتخاب می‌شوند. این باعث می‌شود که پرس‌وجوهای `kubectl` مانند `kubectl get pods` و `kubectl get pods --field-selector ""` معادل باشند.
{{< /note >}}

## فیلدهای پشتیبانی‌شده

انتخاب‌گرهای فیلد پشتیبانی‌شده بر اساس نوع منبع Kubernetes متفاوت است. همه انواع منابع از فیلدهای `metadata.name` و `metadata.namespace` پشتیبانی می‌کنند. استفاده از انتخاب‌گرهای فیلد پشتیبانی‌نشده خطایی را نتیجه می‌دهد. به عنوان مثال:

```shell
kubectl get ingress --field-selector foo.bar=baz
```
```
Error from server (BadRequest): Unable to find "ingresses" that match label selector "", field selector "foo.bar=baz": "foo.bar" is not a known field selector: only "metadata.name", "metadata.namespace"
```

### لیست فیلدهای پشتیبانی‌شده

| نوع                      | فیلدها                                                                                                                                                                                                                                                         |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Pod                       | `spec.nodeName`<br>`spec.restartPolicy`<br>`spec.schedulerName`<br>`spec.serviceAccountName`<br>`spec.hostNetwork`<br>`status.phase`<br>`status.podIP`<br>`status.nominatedNodeName`                                                                            |
| Event                     | `involvedObject.kind`<br>`involvedObject.namespace`<br>`involvedObject.name`<br>`involvedObject.uid`<br>`involvedObject.apiVersion`<br>`involvedObject.resourceVersion`<br>`involvedObject.fieldPath`<br>`reason`<br>`reportingComponent`<br>`source`<br>`type` |
| Secret                    | `type`                                                                                                                                                                                                                                                          |
| Namespace                 | `status.phase`                                                                                                                                                                                                                                                  |
| ReplicaSet                | `status.replicas`                                                                                                                                                                                                                                               |
| ReplicationController     | `status.replicas`                                                                                                                                                                                                                                               |
| Job                       | `status.successful`                                                                                                                                                                                                                                             |
| Node                      | `spec.unschedulable`                                                                                                                                                                                                                                            |
| CertificateSigningRequest | `spec.signerName`                                                                                                                                                                                                                                               |

## عملگرهای پشتیبانی‌شده

شما می‌توانید از عملگرهای `=`, `==`, و `!=` با انتخاب‌گرهای فیلد استفاده کنید (`=` و `==` به معنی یکسان بودن هستند). به عنوان مثال، این دستور `kubectl` تمام خدمات Kubernetes را انتخاب می‌کند که در فضای نام `default` نیستند:

```shell
kubectl get services --all-namespaces --field-selector metadata.namespace!=default
```
{{< note >}}
عملگرهای مبتنی بر مجموعه‌ها (`in`, `notin`, `exists`) برای انتخاب‌گرهای فیلد پشتیبانی نمی‌شوند.
{{< /note >}}

## انتخاب‌گرهای متصل‌شده

همانند [برچسب‌ها](/docs/concepts/overview/working-with-objects/labels) و انتخاب‌گرهای دیگر، انتخاب‌گرهای فیلد می‌توانند به صورت یک لیست جداشده با کاما به هم پیوند داده شوند. این دستور `kubectl` تمام Pods را انتخاب می‌کند که `status.phase` آنها برابر `Running` نباشد و فیلد `spec.restartPolicy` برابر `Always` باشد:

```shell
kubectl get pods --field-selector=status.phase!=Running,spec.restartPolicy=Always
```

## انواع منابع چندگانه

شما می‌توانید از انتخاب‌گرهای فیلد در انواع منابع مختلف Kubernetes استفاده کنید. این دستور `kubectl` همه Statefulsets و Services را انتخاب می‌کند که در فضای نام `default` نیستند:

```shell
kubectl get statefulsets,services --all-namespaces --field-selector metadata.namespace!=default
```
