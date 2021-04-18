## 渣翻：如何清理你的 Docker 数据

> 原文地址：https://dockerwebdev.com/tutorials/clean-up-docker/

Docker 不会对你的系统进行任何配置更改……但是它会占用大量的磁盘空间。（使用 Docker）不一会儿，当你输入如下命令时，就会返回一些可怕的使用情况统计信息：

```shell
docker system df
```

幸运的是，Docker 允许您从未使用的镜像、容器以及卷中回收磁盘空间。

### 定期修剪（prune）

为了安全地删除已停止的容器，未使用的网络和悬挂的图像，最好每隔一段时间运行以下命令： 

```
docker system prune
```

更具风险的选择是：

```
docker system prune -a
```

这也将抹除任何与正在运行的容器无关的镜像。这可能有一点极端，但是 Docker 还是会重新下载其所需的镜像的。第一次下载会稍慢一些，但随后会将镜像缓存起来以备将来使用。

以下各节介绍了删除特定项目的其他方法。

### 镜像驱逐

Docker 镜像是应用程序（例如Web服务，语言运行时或者数据管理系统）的磁盘快照。您可以通过输入以下内容来查看所有的镜像，不管是运行中的还是悬空的（与容器无关的镜像）：

```
docker image ls -a
```

一个 Docker 镜像可以通过输入以下命令删除：

```
docker image rm <name_or_id>
```

可以将任意数量的镜像添加到此命令——用空格字符分隔它们。 

### 容器清理

Docker 容器是镜像运行的实例，并且可以从同一容器中启动任意数量的容器。容器通常很小，因为它们是无状态的，并且引用了镜像的文件系统。通过输入以下命令查看所有正在运行和已停止的容器：

```
docker container ls -a
```

一旦一个容器停止了，你便可以删除它。停止容器的命令如下：

```
docker container stop <name_or_id>
```

删除容器的命令如下：

```
docker container rm <name_or_id>
```

同样，可以在此命令中添加任意数量的以空格分隔的容器名称或者 ID。

几乎没有必要保留已停止的容器。 可以将 `--rm` 选项添加到 `docker run` 命令中，以在容器终止后自动删除该容器。 

### 网路整理

容器可以连接到 Docker 管理的网络，因此它们才可以相互通信。这些是不会占用太多磁盘空间的配置文件。通过输入以下内容查看所有 Docker 网络：

```
docker network ls
```

输入下面的命令可以删除一个或多个无用的网络：

```
docker network rm <name_or_id>
```

同样，可以在此命令中添加任意数量的以空格分隔的网络名称或者 ID。

### 卷的蒸发

Docker 卷是虚拟磁盘映像。 必须将其附加到正在运行的容器，以便它可以在两次重新启动之间保存文件或其他状态信息。 卷的大小取决于使用它的应用程序，但是典型的数据库即使在大多数情况下都是空的，也需要数百兆的空间。 

通过以下命令便可查看所有 Docker 管理的磁盘卷：

```
docker volume ls
```

移除一个 Docker 卷将会永远抹除其数据！没有回头路！

如果您要开发数据库驱动的应用程序，通常可以保留一个或多个数据转储，这些数据转储可用于重新创建一组特定的记录。大多数数据库客户端工具都提供转储功能，比如 Adminer 中的 `Export` 链接。

大多数数据库系统将提供备份工具，例如 MySQL 中的 `mysqldump` 实用程序。 可以使用 `docker exec` 命令在正在运行的容器上执行这些操作。 

以下 Linux / macOS 命令将在名为 mysql 的容器上运行的名为 mydb 的 MySQL 数据库备份到名为 `backup.sql` 的文件中。 使用密码为 `mysecret` 的 MySQL `root` 用户： 

```
docker exec mysql /usr/bin/mysqldump -u root -pmysecret mydb \
  > backup.sql
```

Windows PowerShell 的等效命令： 

```
docker exec mysql /usr/bin/mysqldump -u root -pmysecret -r mydb | \
  Set-Content backup.sql
```

您还可以使用 `docker cp` 命令将数据文件复制到正在运行的容器或从正在运行的容器复制数据文件。 这是通过源路径和目标路径传递的，容器由其 `名称/ ID` 区分，后跟冒号及其路径，例如，

```
docker cp mycontainer:/some/file ./host/directory
```

假设您的数据是安全的，则可以通过输入以下内容来删除任何未使用的卷：  

```
docker volume rm <name>
```

可以使用以下方法删除所有未使用的Docker卷——当前未连接到正在运行的容器的那些卷： 

```
docker volume prune
```

或者，`docker volume prune -a` 将全部卷删除。 毕竟你已经备份了，不是吗？ 

### 完全干净的开始

可以使用单个命令清除掉每个未使用的容器，镜像，卷和网络： 

```
docker system prune -a --volumes
```

如果要在没有确认提示的情况下强制清理，可以添加 `-f` 。 您的系统将恢复到没有任何 Docker 数据的原始状态。 





