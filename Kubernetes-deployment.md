在 Kubernetes 上部署有两种选择：
- 原生
- 通过 [Helm](https://helm.sh/)

### 本地：
请查看 [kubernetes-bitwarden_rs](https://github.com/icicimov/kubernetes-bitwarden_rs) 存储库，例如在 Kubernetes 中的部署。

它将在 [nginx-ingress-controller](https://github.com/kubernetes/ingress-nginx) 和 AWS [ELBv1](https://aws.amazon) 后面的 Kubernetes 中设置一个功能齐全且安全的“vaultwarden”应用程序.com/elasticloadbalancing/features/#Details_for_Elastic_Load_Balancing_Products）。它提供的不仅仅是简单的部署，而且您可以根据需要和设置使用全部或部分清单。

### 通过头盔​​：
请查看 [helm-bitwarden_rs](https://github.com/Skeen/helm-bitwarden_rs) 存储库，例如在 Kubernetes 中的部署。

它将在您选择的 nginx 控制器后面的 Kubernetes 中设置一个功能齐全且安全的“Vaultwarden”应用程序。它运行良好，并通过 [microk8s](https://microk8s.io/) 设置进行了测试。也支持通过 [cert-manager](https://github.com/jetstack/cert-manager) 生成 SSL 证书。

另一个具有同样甚至更多灵活性的选择是：https://github.com/gissilabs/charts/tree/master/vaultwarden