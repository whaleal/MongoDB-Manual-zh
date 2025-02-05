# 将分片集群升级到4.2 

> 重要：
>
> 在尝试任何升级之前，请熟悉本文档的内容。

如果您需要升级到4.2的指导，[MongoDB专业服务](https://www.mongodb.com/products/consulting?tck=docs_server)提供主要版本升级支持，以帮助确保顺利过渡到MongoDB应用程序而不中断。

## 升级建议和核对清单

升级时，请考虑以下几点：

### 升级版本路径

要将现有的MongoDB部署升级到4.2，您必须运行4.0系列版本。

要从4.0系列之前的版本升级，您必须先后升级主要版本，直到升级到4.0系列。例如，如果您运行的是3.6系列，则必须[先升级到4.0](https://www.mongodb.com/docs/upcoming/release-notes/4.0/#std-label-4.0-upgrade)，*然后*才能升级到4.2。

### 检查驱动程序兼容性

在升级MongoDB之前，请检查您是否使用的是与MongoDB 4.2兼容的驱动程序。咨询[驱动程序文档](https://www.mongodb.com/docs/drivers/)供您的特定驱动程序验证与MongoDB 4.2的兼容性。

在不兼容驱动程序上运行的升级部署可能会遇到意外或未定义的行为。

### 准备工作

在开始升级之前，请参阅[MongoDB 4.2中的兼容性更改](https://www.mongodb.com/docs/upcoming/release-notes/4.2-compatibility/)文档，以确保您的应用程序和部署与MongoDB 4.2兼容。在开始升级之前，解决部署中的不兼容性。

在升级MongoDB之前，在将升级部署到生产环境之前，请务必在分期环境中测试您的应用程序。

### 降级注意事项

一旦升级到4.2，如果您需要降级，我们建议[降级](https://www.mongodb.com/docs/upcoming/release-notes/4.2-downgrade-sharded-cluster/)到4.0的最新补丁版本。

## 阅读关注多数（3个成员初级-中级仲裁器架构）

从MongoDB 3.6开始，MongoDB默认支持[`"majority"`](https://www.mongodb.com/docs/upcoming/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)读取问题。

您可以禁用读取关注[`"majority"`](https://www.mongodb.com/docs/upcoming/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)以防止存储缓存压力固定具有主二级仲裁器（PSA）架构的三元副本集或带有三元PSA碎片的分片集群。

> 笔记：
>
> 禁用[`"majority"`](https://www.mongodb.com/docs/upcoming/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)读取关注会影响对分片集群[交易](https://www.mongodb.com/docs/upcoming/core/transactions/#std-label-transactions)的支持。具体来说：
>
> - 如果事务涉及已禁用读关注“多数”的碎片，则该事务不能使用读关注“快照”。
> - 如果事务的任何读或写操作涉及到禁用了读关注“多数”的碎片，则写入多个碎片的事务将出错。
>
> 然而，它不影响复制集上的[交易](https://www.mongodb.com/docs/upcoming/core/transactions/#std-label-transactions)。对于副本集上的事务，即使禁用了读取关注[`"majority"`](https://www.mongodb.com/docs/upcoming/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)您也可以为多文档事务指定读取关注[`"majority"`](https://www.mongodb.com/docs/upcoming/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)”（或[`"snapshot"`](https://www.mongodb.com/docs/upcoming/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)或[`"local"`](https://www.mongodb.com/docs/upcoming/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)”）。
>
> 禁用[`"majority"`](https://www.mongodb.com/docs/upcoming/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)读取关注阻止修改索引的[`collMod`](https://www.mongodb.com/docs/upcoming/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod)命令[回滚](https://www.mongodb.com/docs/upcoming/core/replica-set-rollbacks/#std-label-replica-set-rollbacks)。如果此类操作需要回滚，您必须将受影响的节点与[主](https://www.mongodb.com/docs/upcoming/reference/glossary/#std-term-primary)节点重新同步。
>
> 禁用[`"majority"`](https://www.mongodb.com/docs/upcoming/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)读取关注对更改流的可用性没有影响。
>
> 当升级到4.2并禁用读取关注“多数”时，您可以使用更改流进行部署。

有关更多信息，请参阅[主-二级仲裁器副本集。](https://www.mongodb.com/docs/upcoming/reference/read-concern-majority/#std-label-disable-read-concern-majority)

## 更改流恢复令牌

MongoDB 4.2使用版本1（即`v1`）更改流[恢复令牌](https://www.mongodb.com/docs/upcoming/changeStreams/#std-label-change-stream-resume-token)，[该令牌](https://www.mongodb.com/docs/upcoming/changeStreams/#std-label-change-stream-resume-token)在版本4.0.7中引入。

恢复令牌`_data`类型取决于MongoDB版本，在某些情况下取决于更改流打开/恢复时的功能兼容性版本（fcv）（即fcv值的更改不会影响已打开的更改流的恢复令牌）：

| MongoDB版本             | 功能兼容性版本 | 恢复令牌`_data`类型    |
| :---------------------- | :------------- | :--------------------- |
| MongoDB 4.2及更高版本   | "4.2"或"4.0"   | 六角编码字符串（`v1`） |
| MongoDB 4.0.7及更高版本 | "4.0"或"3.6"   | 六角编码字符串（`v1`） |
| MongoDB 4.0.6及更早版本 | "4.0"          | 六角编码字符串（`v0`） |
| MongoDB 4.0.6及更早版本 | "3.6"          | BinData                |
| MongoDB 3.6             | "3.6"          | BinData                |

> 笔记：
>
> **从MongoDB 4.0.6或更早版本升级到MongoDB 4.2时**
>
> 在升级过程中，分片集群的成员将继续生成`v0`令牌，直到第一个[`mongos`](https://www.mongodb.com/docs/upcoming/reference/program/mongos/#mongodb-binary-bin.mongos)实例升级。升级[`mongos`](https://www.mongodb.com/docs/upcoming/reference/program/mongos/#mongodb-binary-bin.mongos)实例将开始生成`v1`更改流恢复令牌。这些不能用于恢复尚未升级的[`mongos`](https://www.mongodb.com/docs/upcoming/reference/program/mongos/#mongodb-binary-bin.mongos)流。

### 先决条件

## 所有会员版本

要将分片集群升级到4.2，集群**的所有**成员必须至少是4.0版本。升级过程检查集群的所有组件，如果任何组件运行版本早于4.0，则会发出警告。

## MMAPv1到WiredTiger存储引擎

MongoDB 4.2取消了对已弃用的MMAPv1存储引擎的支持。

如果您的4.0部署使用MMAPv1，则在升级到MongoDB 4.2之前，您必须将4.0部署更改为[WiredTiger存储引擎](https://www.mongodb.com/docs/upcoming/core/wiredtiger/)。有关详细信息，请参阅[将分片集群更改为WiredTiger。](https://www.mongodb.com/docs/upcoming/tutorial/change-sharded-cluster-wiredtiger/)

## 查看当前配置

使用MongoDB 4.2，[`mongod`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#mongodb-binary-bin.mongod)和[`mongos`](https://www.mongodb.com/docs/upcoming/reference/program/mongos/#mongodb-binary-bin.mongos)进程将不会从[MMAPv1特定配置选项](https://www.mongodb.com/docs/upcoming/release-notes/4.2/#std-label-4.2-mmapv1-conf-options)开始。运行WiredTiger的先前版本的MongoDB忽略了MMAPv1配置选项（如果指定）。使用MongoDB 4.2，您必须从配置中删除这些。

## 功能兼容性版本

4.0分片集群必须将`featureCompatibilityVersion`设置为`4.0`。

为了确保分片集群的所有成员都将`featureCompatibilityVersion`设置为`4.0`，请连接到每个碎片副本集成员和每个配置服务器副本集成员，并检查`featureCompatibilityVersion`：

> 提示：
>
> 对于启用了访问控制的分片集群，要对碎片副本集成员运行以下命令，您必须以[碎片本地用户](https://www.mongodb.com/docs/upcoming/core/security-users/#std-label-shard-local-users)的身份连接到该成员[。](https://www.mongodb.com/docs/upcoming/core/security-users/#std-label-shard-local-users)

```
db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
```

所有成员都应该返回一个包含`"featureCompatibilityVersion" : { "version" : "4.0" }`的结果。

要设置或更新`featureCompatibilityVersion`，请在[`mongos`](https://www.mongodb.com/docs/upcoming/reference/program/mongos/#mongodb-binary-bin.mongos)上运行以下命令[：](https://www.mongodb.com/docs/upcoming/reference/program/mongos/#mongodb-binary-bin.mongos)

```
db.adminCommand( { setFeatureCompatibilityVersion: "4.0" } )
```

有关更多信息，请参阅[`setFeatureCompatibilityVersion`。](https://www.mongodb.com/docs/upcoming/reference/command/setFeatureCompatibilityVersion/#mongodb-dbcommand-dbcmd.setFeatureCompatibilityVersion)

### 副本集成员状态

对于碎片和配置服务器，请确保没有副本集成员处于[`ROLLBACK`](https://www.mongodb.com/docs/upcoming/reference/replica-states/#mongodb-replstate-replstate.ROLLBACK)或[`RECOVERING`](https://www.mongodb.com/docs/upcoming/reference/replica-states/#mongodb-replstate-replstate.RECOVERING)状态。

## 备份`config`数据库[![img](https://www.mongodb.com/docs/upcoming/assets/link.svg)](https://www.mongodb.com/docs/upcoming/release-notes/4.2-upgrade-sharded-cluster/#back-up-the-config-database)

*可选但推荐。*作为预防措施，在升级分片集群*之前*，请先备份`config`数据库。

## PowerPC上的哈希索引[![img](https://www.mongodb.com/docs/upcoming/assets/link.svg)](https://www.mongodb.com/docs/upcoming/release-notes/4.2-upgrade-sharded-cluster/#hashed-indexes-on-powerpc)

*仅适用于PowerPC*

对于[散列索引](https://www.mongodb.com/docs/upcoming/core/index-hashed/)，MongoDB 4.2确保PowerPC上浮点值2 63的散列值与其他平台一致。

虽然可能包含大于263的浮点值的字段上的[散列索引](https://www.mongodb.com/docs/upcoming/core/index-hashed/)是不支持的配置，但客户端仍然可以插入索引字段值为2 63的文档。

* 如果*PowerPC*上的当前MongoDB 4.0分片集群具有2 63的散列碎片键值，那么，在升级之前：

  1. 备份文档；例如[`mongoexport`](https://www.mongodb.com/docs/database-tools/mongoexport/#mongodb-binary-bin.mongoexport)使用`--query`选择碎片键字段中包含2 63的文档。
  2. 删除具有263值的文档。

  按照以下程序升级后，您将导入已删除的文档。

* 如果*PowerPC*上的现有MongoDB 4.0集合具有值2 63的散列索引项，而不是用作碎片键，您还可以选择在升级前删除索引，然后在升级完成后重新创建索引。

列出部署的所有哈希索引，并查找索引字段包含值2 63[哈希索引和PowerPC检查](https://www.mongodb.com/docs/upcoming/core/index-hashed/#std-label-hashed-index-power-pc-check)的文档[。](https://www.mongodb.com/docs/upcoming/core/index-hashed/#std-label-hashed-index-power-pc-check)

## 下载4.2二进制文件

### 使用软件包管理器

如果您从MongoDB `apt`、`yum`、`dnf`或`zypper`存储库安装了MongoDB，则应使用软件包管理器升级到4.2。

按照适用于Linux系统的相应4.2[安装说明](https://www.mongodb.com/docs/upcoming/installation/#std-label-tutorial-installation)进行操作。这将涉及为新版本添加一个存储库，然后执行实际的升级过程。

## 手动下载4.2 二进制日志

如果您尚未使用软件包管理器安装MongoDB，您可以手动从[MongoDB下载中心](https://www.mongodb.com/download-center?tck=docs_server)。

有关更多信息，请参阅[4.2安装说明](https://www.mongodb.com/docs/upcoming/installation/#std-label-tutorial-installation)。

### 升级流程

#### 1、停用平衡器

将mongo shell连接到分片集群中的 [`mongos`](https://www.mongodb.com/docs/upcoming/reference/program/mongos/#mongodb-binary-bin.mongos)实例，并运行[`sh.stopBalancer()`](https://www.mongodb.com/docs/upcoming/reference/method/sh.stopBalancer/#mongodb-method-sh.stopBalancer)禁用均衡器：

```
sh.stopBalancer()
```

> 笔记：
>
> 如果迁移正在进行中，系统将在停止平衡器之前完成正在进行的迁移。您可以runsh[`sh.isBalancerRunning()`](https://www.mongodb.com/docs/upcoming/reference/method/sh.isBalancerRunning/#mongodb-method-sh.isBalancerRunning)来检查平衡器的当前状态。

要验证平衡器是否已禁用，请运行[`sh.getBalancerState()`](https://www.mongodb.com/docs/upcoming/reference/method/sh.getBalancerState/#mongodb-method-sh.getBalancerState)），如果平衡器被禁用，则返回false：

```
sh.getBalancerState()
```

有关禁用平衡器的更多信息，请参阅[禁用平衡器。](https://www.mongodb.com/docs/upcoming/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-balancing-disable-temporarily)

#### 2、升级配置服务器。

1. 一次升级一个副本的[辅助](https://www.mongodb.com/docs/upcoming/core/replica-set-members/#std-label-replica-set-secondary-members)成员：

   * 关闭二级[`mongod`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#mongodb-binary-bin.mongod)实例，并将4.0二进制文件替换为4.2二进制文件。

   * 使用[`--configsvr`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--configsvr)、[`--replSet`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--replSet)和[`--port`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--port)启动4.2二进制文件。包括部署中使用的任何其他选项。

     ```
     mongod --configsvr --replSet <replSetName> --port <port> --dbpath <path> --bind_ip localhost,<ip address>
     ```

     如果使用[配置文件](https://www.mongodb.com/docs/upcoming/reference/configuration-options/)，请更新文件以指定[`sharding.clusterRole: configsvr`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-sharding.clusterRole)、[`replication.replSetName`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-replication.replSetName)、[`net.port`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.port)和[`net.bindIp`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.bindIp)，然后启动4.2二进制文件：

     ```
     sharding:
        clusterRole: configsvr
     replication:
        replSetName: <string>
     net:
        port: <port>
        bindIp: localhost,<ip address>
     storage:
        dbpath: <path>
     ```

     包括适合您部署的任何其他设置。

   * 请等待成员恢复到SECONDARY状态，然后再升级下一个辅助成员。要检查成员的状态，请在mongo shell中发出[`rs.status()`](https://www.mongodb.com/docs/upcoming/reference/method/rs.status/#mongodb-method-rs.status)。

     对每个辅助成员重复。

2. 逐步关闭副本集主副本。

   * 将一个mongo shell连接到主服务器，并使用[`rs.stepDown()`](https://www.mongodb.com/docs/upcoming/reference/method/rs.stepDown/#mongodb-method-rs.stepDown)将主服务器降级，强制选举一个新的主服务器：

     ```
     rs.stepDown()
     ```

   * 当[`rs.status()`](https://www.mongodb.com/docs/upcoming/reference/method/rs.status/#mongodb-method-rs.status)显示主制已下级，另一个成员已处于`PRIMARY`状态时，请关闭降级主制，并将[`mongod`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#mongodb-binary-bin.mongod)二进制文件替换为4.2二进制文件。

   * 使用[`--configsvr`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--configsvr)、[`--replSet`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--replSet)、[`--port`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--port)和[`--bind_ip`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--bind_ip)选项启动4.2二进制文件。包括上一次部署中使用的任何可选命令行选项：

     ```
     mongod --configsvr --replSet <replSetName> --port <port> --dbpath <path> --bind_ip localhost,<ip address>
     ```

     如果使用[配置文件](https://www.mongodb.com/docs/upcoming/reference/configuration-options/)，请更新文件以指定[`sharding.clusterRole: configsvr`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-sharding.clusterRole)、[`replication.replSetName`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-replication.replSetName)、[`net.port`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.port)和[`net.bindIp`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.bindIp)，然后启动4.2二进制文件：

     ```
     sharding:
        clusterRole: configsvr
     replication:
        replSetName: <string>
     net:
        port: <port>
        bindIp: localhost,<ip address>
     storage:
        dbpath: <path>
     ```

     包括适合您部署的任何其他配置。

#### 3、升级分片。

一次升级一个碎片。

对于每个碎片副本集：

1. 一次升级一个副本的[辅助](https://www.mongodb.com/docs/upcoming/core/replica-set-members/#std-label-replica-set-secondary-members)成员：

   * 关闭[`mongod`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#mongodb-binary-bin.mongod)实例，并将4.0二进制文件替换为4.2二进制文件。

   * 使用[`--shardsvr`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--shardsvr)、[`--replSet`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--replSet)、[`--port`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--port)和[`--bind_ip`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--bind_ip)选项启动4.2二进制文件。包括适合您部署的任何其他命令行选项：

   * ```
     mongod --shardsvr --replSet <replSetName> --port <port> --dbpath <path> --bind_ip localhost,<ip address>
     ```

     如果使用[配置文件](https://www.mongodb.com/docs/upcoming/reference/configuration-options/)，请将文件更新为包括[`sharding.clusterRole: shardsvr`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-sharding.clusterRole)、[`replication.replSetName`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-replication.replSetName)、[`net.port`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.port)和[`net.bindIp`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.bindIp)，然后启动4.2二进制文件：

     ```
     sharding:
        clusterRole: shardsvr
     replication:
        replSetName: <string>
     net:
        port: <port>
        bindIp: localhost,<ip address>
     storage:
        dbpath: <path>
     ```

     包括适合您部署的任何其他配置。

   * 请等待成员恢复到SECONDARY状态，然后再升级下一个辅助成员。要检查成员的状态，可以在mongo shell中发出[`rs.status()`](https://www.mongodb.com/docs/upcoming/reference/method/rs.status/#mongodb-method-rs.status)。

     对每个辅助成员重复。

2. 逐步关闭副本集主副本。

   * 将一个mongo shell连接到主服务器，并使用rs.stepDown（）将主服务器降级，强制选举一个新的主服务器：

     ```
     rs.stepDown()
     ```

3. 当[`rs.status()`](https://www.mongodb.com/docs/upcoming/reference/method/rs.status/#mongodb-method-rs.status)显示初选已下台，而其他成员已处于`PRIMARY`状态时，请升级降级初选：

   * 关闭降级主制，用4.2二进制文件替换为[`mongod`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#mongodb-binary-bin.mongod)二进制文件。

   * 使用[`--shardsvr`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--shardsvr)、[`--replSet`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--replSet)、[`--port`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--port)和[`--bind_ip`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--bind_ip)选项启动4.2二进制文件。包括适合您部署的任何其他命令行选项：

     ```
     mongod --shardsvr --replSet <replSetName> --port <port> --dbpath <path> --bind_ip localhost,<ip address>
     ```

     如果使用[配置文件](https://www.mongodb.com/docs/upcoming/reference/configuration-options/)，请更新文件以指定[`sharding.clusterRole: shardsvr`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-sharding.clusterRole)、[`replication.replSetName`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-replication.replSetName)、[`net.port`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.port)和[`net.bindIp`](https://www.mongodb.com/docs/upcoming/reference/configuration-options/#mongodb-setting-net.bindIp)，然后启动4.2二进制文件：

     ```
     sharding:
        clusterRole: shardsvr
     replication:
        replSetName: <string>
     net:
        port: <port>
        bindIp: localhost,<ip address>
     storage:
        dbpath: <path>
     ```

     包括适合您部署的任何其他配置。

#### 4、升级`mongos`实例。

将每个[`mongos`](https://www.mongodb.com/docs/upcoming/reference/program/mongos/#mongodb-binary-bin.mongos)实例替换为4.2二进制文件并重新启动。包括适合您部署的任何其他配置。

> 笔记：
>
> 当分片集群成员在不同主机上运行或远程客户端连接到分片集群时，必须指定[`--bind_ip`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#std-option-mongod.--bind_ip)选项。有关更多信息，请参阅[Localhost绑定兼容性更改。](https://www.mongodb.com/docs/upcoming/release-notes/3.6-compatibility/#std-label-3.6-bind_ip-compatibility)

```
mongos --configdb csReplSet/<rsconfigsver1:port1>,<rsconfigsver2:port2>,<rsconfigsver3:port3> --bind_ip localhost,<ip address>
```

**如果从MongoDB 4.0.6或更低版本升级，**

一旦部署的mongos实例升级，该mongos实例开始生成v1更改流恢复令牌。这些令牌不能用于在尚未升级的mongos实例上恢复流。

#### 5、重新启用平衡器。

使用4.2 mongo shell，连接到集群中的 [`mongos`](https://www.mongodb.com/docs/upcoming/reference/program/mongos/#mongodb-binary-bin.mongos)并运行[`sh.startBalancer()`](https://www.mongodb.com/docs/upcoming/reference/method/sh.startBalancer/#mongodb-method-sh.startBalancer)重新启用平衡器：

```
sh.startBalancer()
```

从MongoDB 6.1开始，不执行自动分割块。这是因为平衡了政策的改进。自动拆分命令仍然存在，但不执行操作。有关详细信息，请参阅[平衡策略更改。](https://www.mongodb.com/docs/upcoming/release-notes/6.1/#std-label-release-notes-6.1-balancing-policy-changes)

在6.1之前的MongoDB版本中，[`sh.startBalancer()`](https://www.mongodb.com/docs/upcoming/reference/method/sh.startBalancer/#mongodb-method-sh.startBalancer)还支持分片集群的自动拆分。

如果您不希望在启用平衡器时启用自动拆分，您还必须运行[`sh.disableAutoSplit()`。](https://www.mongodb.com/docs/upcoming/reference/method/sh.disableAutoSplit/#mongodb-method-sh.disableAutoSplit)

有关重新启用平衡器的更多信息，请参阅[启用平衡器。](https://www.mongodb.com/docs/upcoming/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-balancing-enable)

#### 6、启用向后不兼容的4.2功能

此时，您可以在没有与4.0不兼容的4.2[功能的情况下](https://www.mongodb.com/docs/upcoming/release-notes/4.2-compatibility/#std-label-4.2-compatibility-enabled)运行4.2二进制文件。

要启用这些4.2功能，请将功能兼容性版本（`FCV`）设置为4.2。

>提示：
>
>启用这些向后不兼容的功能可能会使降级过程复杂化，因为在降级之前，您必须删除任何持续存在的向后不兼容的功能。
>
>建议在升级后，在烧毁期间允许您的部署在不启用这些功能的情况下运行，以确保降级的可能性最小。当您确信降级的可能性很小时，请启用这些功能。

在[`mongos`](https://www.mongodb.com/docs/upcoming/reference/program/mongos/#mongodb-binary-bin.mongos)实例上，在`admin`数据库中运行[`setFeatureCompatibilityVersion`](https://www.mongodb.com/docs/upcoming/reference/command/setFeatureCompatibilityVersion/#mongodb-dbcommand-dbcmd.setFeatureCompatibilityVersion)命令：

```
db.adminCommand( { setFeatureCompatibilityVersion: "4.2" } )

```

此命令必须对内部系统集合执行写入。如果由于任何原因命令未能成功完成，您可以安全地在[`mongos`](https://www.mongodb.com/docs/upcoming/reference/program/mongos/#mongodb-binary-bin.mongos)上重试该命令，因为操作是幂等的。

> 笔记：
>
> **..包括:: /includes/fact-mongos-fcv.rst**

### 升级后

`TLS`选项替换已弃用的`SSL`选

从MongoDB 4.2开始，MongoDB不再支持[mongod](https://www.mongodb.com/docs/manual/reference/program/mongod/#std-label-ssl-mongod-options)、 [mongos](https://www.mongodb.com/docs/manual/reference/program/mongos/#std-label-mongos-ssl-options)和mongo shell的SSL选项以及相应的 [`net.ssl` Options](https://www.mongodb.com/docs/manual/reference/configuration-options/#std-label-net-ssl-conf-options)配置文件选项。为了避免出现弃用消息，请对 [mongod](https://www.mongodb.com/docs/manual/reference/program/mongod/#std-label-tls-mongod-options), [mongos](https://www.mongodb.com/docs/manual/reference/program/mongos/#std-label-mongos-tls-options)和mongo使用新的TLS选项。

* 对于命令行TLS选项，请参考mongod、mongos和mongo shell页面。
* 有关相应的`mongod`和`mongos`配置文件选项，请参阅[配置文件](https://www.mongodb.com/docs/manual/reference/configuration-options/#std-label-net-tls-conf-options)页面。
* 有关连接字符串`tls`选项，请参阅[连接字符串](https://www.mongodb.com/docs/manual/reference/connection-string/#std-label-uri-options-tls)页面。

**4.2+兼容驱动程序默认重试写入**

与MongoDB 4.2及更高版本兼容的驱动程序默认启用[可重试写入](https://www.mongodb.com/docs/manual/core/retryable-writes/#std-label-retryable-writes)。较早的驱动程序需要[`retryWrites=true`](https://www.mongodb.com/docs/manual/reference/connection-string/#mongodb-urioption-urioption.retryWrites)选项。在使用与MongoDB 4.2及更高版本兼容的驱动程序的应用程序中，可以省略[`retryWrites=true`](https://www.mongodb.com/docs/manual/reference/connection-string/#mongodb-urioption-urioption.retryWrites)选项。

要禁用可重试写入，使用与MongoDB 4.2及更高版本兼容的驱动程序的应用程序必须在连接字符串中包含[`retryWrites=false`](https://www.mongodb.com/docs/manual/reference/connection-string/#mongodb-urioption-urioption.retryWrites)。

**PowerPC和哈希指数值为2 63**

如果在PowerPC上，您找到了值为2 63的散列索引字段，

* 如果您删除了文档，请从导出中替换它们（作为先决条件的一部分完成）。

* 如果您在升级前删除了哈希索引，请重新创建索引。

### 其他升级程序

- 要升级独立设备，请参阅[将独立版升级到4.2。](https://www.mongodb.com/docs/manual/release-notes/4.2-upgrade-standalone/#std-label-4.2-upgrade-standalone)
- 要升级副本集，请参阅[将副本集升级到4.2。](https://www.mongodb.com/docs/manual/release-notes/4.2-upgrade-replica-set/#std-label-4.2-upgrade-replica-set)



原文 - [Upgrade a Sharded Cluster to 4.2]( https://docs.mongodb.com/manual/release-notes/4.2-upgrade-sharded-cluster/ )

