---
title: "نصب ابزارها"
description: "راه‌اندازی ابزارهای Kubernetes بر روی کامپیوتر شما."
weight: 10
no_list: true
card:
  name: tasks
  weight: 20
  anchors:
  - anchor: "#kubectl"
    title: نصب kubectl
---

## kubectl

<!-- overview -->
ابزار خط فرمان Kubernetes، [kubectl](/docs/reference/kubectl/kubectl/)، به شما امکان می‌دهد تا دستوراتی را علیه خوشه‌های Kubernetes اجرا کنید.
می‌توانید از kubectl برای استقرار برنامه‌ها، بررسی و مدیریت منابع خوشه، و مشاهده‌ی log‌ها استفاده کنید. برای اطلاعات بیشتر از جمله لیست کامل عملیات kubectl، مستندات [kubectl](/docs/reference/kubectl/) را مشاهده کنید.

kubectl قابل نصب بر روی انواع مختلف سیستم‌های عامل Linux، macOS و Windows می‌باشد.
سیستم عامل مورد نظر خود را در زیر پیدا کنید.

- [نصب kubectl در Linux](/docs/tasks/tools/install-kubectl-linux)
- [نصب kubectl در macOS](/docs/tasks/tools/install-kubectl-macos)
- [نصب kubectl در Windows](/docs/tasks/tools/install-kubectl-windows)

## kind

[`kind`](https://kind.sigs.k8s.io/) به شما اجازه می‌دهد تا Kubernetes را بر روی کامپیوتر محلی خود اجرا کنید.
این ابزار نیازمند نصب [Docker](https://www.docker.com/) یا [Podman](https://podman.io/) است.

صفحه [شروع سریع kind](https://kind.sigs.k8s.io/docs/user/quick-start/) به شما نشان می‌دهد که چگونه می‌توانید با kind شروع به کار کنید.

<a class="btn btn-primary" href="https://kind.sigs.k8s.io/docs/user/quick-start/" role="button" aria-label="مشاهده راهنمای شروع سریع kind">مشاهده راهنمای شروع سریع kind</a>

## minikube

مشابه `kind`، [`minikube`](https://minikube.sigs.k8s.io/) ابزاری است که به شما امکان می‌دهد تا Kubernetes را به صورت محلی اجرا کنید.
minikube یک خوشه Kubernetes محلی all-in-one یا چند-گره‌ای را بر روی کامپیوتر شخصی خود اجرا می‌کند (شامل کامپیوترهای Windows، macOS و Linux) تا شما بتوانید Kubernetes را آزمایش کنید یا برای کارهای توسعه روزانه استفاده کنید.

می‌توانید [راهنمای شروع minikube](https://minikube.sigs.k8s.io/docs/start/) رسمی را دنبال کنید اگر تمرکز شما بر روی نصب این ابزار است.

<a class="btn btn-primary" href="https://minikube.sigs.k8s.io/docs/start/" role="button" aria-label="مشاهده راهنمای شروع minikube">مشاهده راهنمای شروع minikube</a>

هنگامی که minikube کار می‌کند، می‌توانید از آن برای [اجرای یک برنامه نمونه](/docs/tutorials/hello-minikube/) استفاده کنید.

## kubeadm

می‌توانید از ابزار {{< glossary_tooltip term_id="kubeadm" text="kubeadm" >}} برای ایجاد و مدیریت خوشه‌های Kubernetes استفاده کنید.
این ابزار اقدامات لازم برای راه‌اندازی یک خوشه امن و حداقلی Kubernetes را به شیوه‌ای کاربرپسندانه انجام می‌دهد.

[نصب kubeadm](/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) به شما نشان می‌دهد که چگونه می‌توانید kubeadm را نصب کنید.
بعد از نصب، می‌توانید از آن برای [ایجاد یک خوشه](/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) استفاده کنید.

<a class="btn btn-primary" href="/docs/setup/production-environment/tools/kubeadm/install-kubeadm/" role="button" aria-label="مشاهده راهنمای نصب kubeadm">مشاهده راهنمای نصب kubeadm</a>
