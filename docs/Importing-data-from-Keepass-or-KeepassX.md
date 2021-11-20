## 介绍
Bitwarden 可以从大量[应用程序](https://help.bitwarden.com/article/import-data/) 中导入您的数据。

当前的导入器只允许您选择格式，而不是如何将数据转换为 Bitwarden。

## Keepass 和 KeepassX 的导入结果不同

从 Keepass 或 KeepassX 导入会得到完全不同的结果，尽管它们使用相同的 Keepass 2.x kbdx 数据库：

- Keepass CSV 文件在 **组织** 级别(每个条目的所有者)导入，并将 Keepass 组转换为 Bitwarden **集合**。
- Keepass XML 文件在 **用户** 级别(每个条目的所有者)导入，并将 Keepass 组转换为 Bitwarden **文件夹**，主文件夹为 Keepass 数据库的名称。

Bitwarden 本身需要做很多工作来将集合更改为文件夹或转移所有条目的所有权。
因此，根据您的需要，选择合适的方法！

## 例子

### 名为`MyVault`的 Keepass 数据库

**Groups:**

```
- Group1
  -   Group1Sub1
  -   Group2Sub2
- Group2
```

### 通过 Keepass (CSV) 导入：

**所有者** = 组织

**Collections:**

```
- Group1
  -   Group1Sub1
  -   Group2Sub2
- Group2
```

### 通过 Keepass (XML) 导入：

**所有者** = 登录用户

**Folders:**

```
- MyVault
  - Group1
    - Group1Sub1
    - Group2Sub2
  - Group2
```

注意：您可能需要手动创建主文件夹，因为导入将 MyVault/Group1 显示为文件夹。创建文件夹 MyVault 会显示 MMI 中的子文件夹。

注意2：您可以编辑文件夹以删除主文件夹`MyVault`，或编辑导出的CSV文件并在导入Bitwarden之前删除每个条目中的`MyVault/`字符串。