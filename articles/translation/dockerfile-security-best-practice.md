## 渣翻： `Dockerfile` 安全性最佳实践

> 原文地址：https://cloudberry.engineering/article/dockerfile-security-best-practices/

容器安全是一个广阔的问题空间，这里有许多轻松实现的方法来减轻风险。一个很好的开端就是在编写 `Dockerfile` 时遵循一些规则。

### 不要在环境变量中存储密钥

密钥分散存在是一个毛骨悚然的问题，而且很容易出错。对于容器化应用，可以通过安装卷从文件系统，也可以通过环境变量更方便地显示它们。

使用 `ENV` 来存储密钥是不好的做法，因为 `Dockerfile` 通常随应用程序一起分布式存在，因此这与应编码密钥没有区别。

如何检测：

```
secrets_env = [
    "passwd",
    "password",
    "pass",
 #  "pwd", can't use this one   
    "secret",
    "key",
    "access",
    "api_key",
    "apikey",
    "token",
    "tkn"
]

deny[msg] {    
    input[i].Cmd == "env"
    val := input[i].Value
    contains(lower(val[_]), secrets_env[_])
    msg = sprintf("Line %d: Potential secret in ENV key found: %s", [i, val])
}
```

### 只使用可信的基础镜像

容器化应用的供应链攻击也会来自与构建容器本身的层次结构。

罪魁祸首显然是所使用的基础镜像。不可信的基础镜像具有很高的风险，应尽可能避免使用。

`Docker` 为大多数操作系统和应用程序提供了一系列官方基础镜像。使用这些镜像，我们可以利用与 `Docker` 的分担来最大程度的降低遭受破坏的风险。

如何检测：

```
deny[msg] {
    input[i].Cmd == "from"
    val := split(input[i].Value[0], "/")
    count(val) > 1
    msg = sprintf("Line %d: use a trusted base image", [i])
}
```

这规则针对 `DockerHub` 的官方镜像进行了调整。由于我只检测到缺失 `namespace` 就报错，这显得有点愚蠢。

可信的定义取决于您的上下文：相应地更改此规则。

### 不要使用基础镜像的 `latest` 标签

固定基础镜像的版本，可以让你在构建容器的可预测性方面放心。

如果你使用最新镜像，则可能默默集成更新的程序包，最坏的情况下可能会影响应用程序的可靠性，而且可能会引入漏洞。

如何检测：

```
deny[msg] {
    input[i].Cmd == "from"
    val := split(input[i].Value[0], ":")
    contains(lower(val[1]), "latest"])
    msg = sprintf("Line %d: do not use 'latest' tag for base images", [i])
}
```

### 避免 `curl` 脚本

从互联网上拉取软件（依赖）并将其用于 `shell` 中可能会很糟糕。不幸的是，它却是简化软件安装的广泛解决方案。

```
wget https://cloudberry.engineering/absolutely-trustworthy.sh | sh
```

这与供应链攻击的风险是相同的，并且**可以划分为可信软件**。如果您真的要使用 `curl` 脚本，请正确执行以下操作：

- 使用可信任的源
- 使用安全的连接
- 验证下载内容的真实性和完整性

如何检测：

```
deny[msg] {
    input[i].Cmd == "run"
    val := concat(" ", input[i].Value)
    matches := regex.find_n("(curl|wget)[^|^>]*[|>]", lower(val), -1)
    count(matches) > 0
    msg = sprintf("Line %d: Avoid curl bashing", [i])
}
```

### 不要升级你的系统包

这可能有点费劲，但原因如下：您想要固定软件依赖项的版本，但如果执行 `apt-get upgrade`，则会将它全部有效地升级到最新版本。

如果您执行了升级并且基础镜像使用了 `latest` 标签，则会放大依赖关系树的不可预测性。

您要做的是固定基础镜像的版本，然后只是执行 `apt/apk update`。

如何检测：

```
upgrade_commands = [
    "apk upgrade",
    "apt-get upgrade",
    "dist-upgrade",
]

deny[msg] {
    input[i].Cmd == "run"
    val := concat(" ", input[i].Value)
    contains(val, upgrade_commands[_])
    msg = sprintf(“Line: %d: Do not upgrade your system packages", [i])
}
```

### 尽可能不使用 `ADD` 命令

`ADD` 命令的一个小功能是，您可以指向远程 URL，它将在构建时获取内容：

```
ADD https://cloudberry.engineering/absolutely-trust-me.tar.gz
```

讽刺的是，官方文档建议改为使用 `curl` 脚本替代。

从安全的角度来看，这里给出同样的建议：不使用（`ADD`）。无论想要获取什么内容，要先进行验证，然后使用 `COPY` 命令。但是如果你不得不这么做，请通过安全的连接使用可信任的源。

注意：如果您有一个动态生成 `Dockerfile` 的高级构建系统，则 `ADD` 实际上是要求使用的连接器。

如何检测：

```
deny[msg] {
    input[i].Cmd == "add"
    msg = sprintf("Line %d: Use COPY instead of ADD", [i])
}
```

### 不要使用 `root` 

容器中的 `root` 与主机上的 `root` 用户相同，但受 `docker` 守护程序配置限制。无论有什么限制，如果一名操作员突破了容器，他仍然可以找到一种方法来获得对主机的完全访问权限。

当然这不是空想，您的威胁模型是不能忽略以 `root` 用户身份运行所带来的风险的。

因此，最好一直指定一个用户：

```
USER hopefullynotroot
```

请注意，在 `Dockerfile` 中明确设置用户只是防御的一层，并不能解决整个 `root` 用户运行的问题。

对应的是，可以并且应该采用深度防御方法，并在整个堆栈中进一步降低风险：严格配置 `docker` 守护进程，或使用 无 `root` 容器解决方案，限制运行时配置（如果可能，禁用 `--privileged` 等），以上。

如何检测：

```
any_user {
    input[i].Cmd == "user"
 }

deny[msg] {
    not any_user
    msg = "Do not run as root, use USER instead"
}
```

### 不要使用 `sudo` 指令

作为不使用 `root` 用户的必然结果，您也不应使用 `sudo` 。

即使您以用户身份运行，也请确保该用户不在 `sudo` 用户列表中。

```
deny[msg] {
    input[i].Cmd == "run"
    val := concat(" ", input[i].Value)
    contains(lower(val), "sudo")
    msg = sprintf("Line %d: Do not use 'sudo' command", [i])
}
```

### 链接：

- 作者`repo`：https://github.com/gbrindisi/dockerfile-security

