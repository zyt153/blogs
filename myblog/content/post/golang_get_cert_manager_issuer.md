---
title: "golang - 获取cluster中的issuer"
date: 2023-12-21T17:37:30+08:00
draft: false
tags: ["golang", "k8s"]
categories: ["golang"]
author: "大白猫"
---

最近需要在项目中检查用户提供的cluster issuer是否存在，需要用到[cert-manager pkg](https://pkg.go.dev/github.com/cert-manager/cert-manager/pkg/apis/certmanager/v1)，但是对于golang和k8s新手来说没有详细的文档，google也没找到什么参考（可能大家觉得太简单了？），我几乎是靠chatgpt完成了这部分，记录一下。

需要import cert-manager相关的pkg（参考[cert-manager-doc-contributing](https://cert-manager.io/docs/contributing/importing/)）:

```go
"sigs.k8s.io/controller-runtime/pkg/client/config"
cmapi "github.com/cert-manager/cert-manager/pkg/apis/certmanager/v1"
"github.com/cert-manager/cert-manager/pkg/client/clientset/versioned"
```

创建一个certManagerClient：

```go
func getCertManagerClient() (versioned.Interface, error) {
	cfg, e := config.GetConfig()
	if e != nil {
		return nil, e
	}

	certManagerClient, e := versioned.NewForConfig(cfg)
	if e != nil {
		return nil, e
	}

	return certManagerClient, nil
}
```

利用GET方法获取指定的cluster issuer：

```go
func getClusterIssuer(ctx context.Context, issuerName string) (*cmapi.ClusterIssuer, error) {
	certManagerClient, e := getCertManagerClient()
	if e != nil {
		return nil, e
	}

	ci, e := certManagerClient.CertmanagerV1().ClusterIssuers().Get(ctx, issuerName, metav1.GetOptions{})
	if e != nil {
		return nil, e
	}

	return ci, nil
}
```

不过最终没有采用这种方法，而是使用项目中已有的k8s controller runtime client来获取cluster issuer。这样需要将`cmapi`注册到`runtime.scheme`中（`cmapi.AddToScheme(scheme)`），再使用client中的GET方法。

