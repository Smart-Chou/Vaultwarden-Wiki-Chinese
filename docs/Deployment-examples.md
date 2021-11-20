
此页面是独立部署示例的索引。如果添加新示例，请酌情创建一个新类别，并总体上保持井井有条。

### 谷歌云

- https://github.com/dadatuputi/bitwarden_gcloud

    Vaultwarden 安装针对 Google Cloud 的`永远免费`e2-micro 计算实例进行了优化

### Kubernetes

- https://github.com/icicimov/kubernetes-bitwarden_rs

    在 Kubernetes 中的 [nginx-ingress-controller](https://github.com/kubernetes/ingress-nginx) 和 AWS [ELBv1](https://aws.amazon.com/elasticloadbalancing/features/#Details_for_Elastic_Load_Balancing_Products)。它提供的不仅仅是简单的部署，而且您可以根据需要和设置使用全部或部分清单。
- https://github.com/Skeen/helm-bitwarden_rs

    在您选择的 nginx 控制器后面的 Kubernetes 中设置功能齐全且安全的`Vaultwarden`应用程序。它运行良好，并通过 [microk8s](https://microk8s.io) 设置进行了测试。也支持通过 [cert-manager](https://github.com/jetstack/cert-manager) 生成 SSL 证书。

### 树莓派

- https://github.com/martient/vaultwarden-ansible

    树莓派上 vaultwarden 的 Ansible 部署，要从以前的配置迁移，请遵循本指南 https://martient.medium.com/migrate-from-bitwarden-rs-to-vaultwarden-199aeb6927a3

## 共享主机

- https://github.com/jjlin/vaultwarden-shared-hosting

    在 [DreamHost](https://www.dreamhost.com) 上运行 `vaultwarden` 的示例配置，但应该很容易适应许多其他共享托管服务。
- https://lab.uberspace.de/guide_bitwarden.html

    有关如何从源代码安装并在 [Uberspace](https://uberspace.de/en/) 共享托管服务提供商上运行的说明。

### NixOS(由 tklitschi 提供)

NixOS 有一个示例 bitwarden 配置。这不是很复杂，你有后端选项，你想使用的数据库类型，专用备份系统的备份目录，启用它的选项和配置选项。对于配置选项，您只需以 nix 语法传递 .env 变量 [来自 .env 模板](https://github.com/dani-garcia/vaultwarden/blob/1.13.1/.env.template)。 有关 nixos-nginx 示例配置，请参阅 [代理示例](Proxy-examples)。

Example Config


```nix
{pkgs,...}:
{
  services.bitwarden_rs = {
  enable = true;
  backupDir = "/mnt/bitwarden";
  

  config = {
      WEB_VAULT_FOLDER = "${pkgs.bitwarden_rs-vault}/share/bitwarden_rs/vault";
      WEB_VAULT_ENABLED = true;
      LOG_FILE = "/var/log/bitwarden";
      WEBSOCKET_ENABLED= true;
      WEBSOCKET_ADDRESS = "0.0.0.0";
      WEBSOCKET_PORT = 3012;
      SIGNUPS_VERIFY = true;
      ADMIN_TOKEN = (import /etc/nixos/secret/bitwarden.nix).ADMIN_TOKEN;
      DOMAIN = "https://exmaple.com";
      YUBICO_CLIENT_ID = (import /etc/nixos/secret/bitwarden.nix).YUBICO_CLIENT_ID;
      YUBICO_SECRET_KEY = (import /etc/nixos/secret/bitwarden.nix).YUBICO_SECRET_KEY;
      YUBICO_SERVER = "https://api.yubico.com/wsapi/2.0/verify";
      SMTP_HOST = "mx.example.com";
      SMTP_FROM = "bitwarden@example.com";
      SMTP_FROM_NAME = "Bitwarden_RS";
      SMTP_PORT = 587;
      SMTP_SSL = true;
      SMTP_USERNAME= (import /etc/nixos/secret/bitwarden.nix).SMTP_USERNAME;
      SMTP_PASSWORD = (import /etc/nixos/secret/bitwarden.nix).SMTP_PASSWORD;
      SMTP_TIMEOUT = 15;
      ROCKET_PORT = 8812;
    };
  };

  environment.systemPackages = with pkgs; [
    bitwarden_rs-vault
  ];

}
```

如果您对这部分有任何疑问，请随时与我联系。我在@litschi:litschi.xyz 上的矩阵和 IRC(hackint 和 freenode)上的 litschi，或者只是在 vaultwarden matrix.org chanel 中询问。

### QNAP NAS (ARM and x86)

* https://github.com/umireon/vaultwarden-qnap

您可以使用 Let's Encrypt 将 Vaultwarden 安装到您的安全网络附加存储 (NAS) 中。 由于 QNAP 内置 HTTP(S) 服务器，您无法在标准 HTTP(S) 端口 (80 / 443) 上发布 Vaultwarden。
