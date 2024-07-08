---
title: استفاده از پروکسی HTTP برای دسترسی به API Kubernetes
content_type: task
weight: 40
---

<!-- overview -->
این صفحه نشان می‌دهد که چگونه از پروکسی HTTP برای دسترسی به API Kubernetes استفاده کنید.


## {{% heading "پیش‌نیازها" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

اگر در حال حاضر برنامه‌ای در خوشه خود اجرا ندارید، با وارد کردن این دستور یک برنامه Hello World را شروع کنید:

```shell
kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0 --port=8080
```

<!-- steps -->

## استفاده از kubectl برای شروع یک سرور پروکسی

این دستور یک پروکسی به سرور API Kubernetes راه‌اندازی می‌کند:

    kubectl proxy --port=8080

## کاوش در API Kubernetes

زمانی که سرور پروکسی در حال اجرا است، می‌توانید از طریق `curl`، `wget` یا مرورگر API را کاوش کنید.

دریافت نسخه‌های API:

    curl http://localhost:8080/api/

خروجی باید مشابه این باشد:

    {
      "kind": "APIVersions",
      "versions": [
        "v1"
      ],
      "serverAddressByClientCIDRs": [
        {
          "clientCIDR": "0.0.0.0/0",
          "serverAddress": "10.0.2.15:8443"
        }
      ]
    }

دریافت لیستی از pods:

    curl http://localhost:8080/api/v1/namespaces/default/pods

خروجی باید مشابه این باشد:

    {
      "kind": "PodList",
      "apiVersion": "v1",
      "metadata": {
        "resourceVersion": "33074"
      },
      "items": [
        {
          "metadata": {
            "name": "kubernetes-bootcamp-2321272333-ix8pt",
            "generateName": "kubernetes-bootcamp-2321272333-",
            "namespace": "default",
            "uid": "ba21457c-6b1d-11e6-85f7-1ef9f1dab92b",
            "resourceVersion": "33003",
            "creationTimestamp": "2016-08-25T23:43:30Z",
            "labels": {
              "pod-template-hash": "2321272333",
              "run": "kubernetes-bootcamp"
            },
            ...
    }

## {{% heading "مراحل بعدی" %}}

بیشتر در مورد [kubectl proxy](/docs/reference/generated/kubectl/kubectl-commands#proxy) بیاموزید.