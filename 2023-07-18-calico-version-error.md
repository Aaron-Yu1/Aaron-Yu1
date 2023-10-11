---
title: 'unable to recognize “calico.yaml”: no matches for kind “PodDisruptionBudget” in version “policy/v1”'
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/free-photo/html-system-websites-concept_23-2150323526.jpg
date: 2023-07-18
categories:
- calico
- Kubernetes
tags:
- calico
- Kubernetes
- calico version
---

如果执行 kubectl apply -f calico.yaml 提示报错 error: unable to recognize "calico.yaml": no matches for kind "PodDisruptionBudget" in version "policy/v1" 说明k8s不支持当前calico版本。

<!--more-->

查看指定的版本的 calico [支持](https://projectcalico.docs.tigera.io/archive/v3.20/getting-started/kubernetes/requirements)的 kubernetes 版本  

```bash
it@testmaster1:~$ sudo kubectl apply -f calico.yaml --kubeconfig=/etc/kubernetes/kubeconfig
serviceaccount/calico-kube-controllers created
serviceaccount/calico-node created
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
deployment.apps/calico-kube-controllers created
error: unable to recognize "calico.yaml": no matches for kind "PodDisruptionBudget" in version "policy/v1"
```
