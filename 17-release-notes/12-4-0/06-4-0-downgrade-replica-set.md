#  将4.0副本集降级为3.6

在尝试降级之前，请熟悉本文档的内容。

## 降级路径

一旦升级到4.0，如果您需要降级，我们建议您降级到最新的3.6补丁版本。

### 更改流注意事项

MongoDB 4.0引入了新的十六进制编码字符串更改流[恢复令牌：](https://www.mongodb.com/docs/upcoming/changeStreams/#std-label-change-stream-resume-token)

恢复令牌`_data`类型取决于MongoDB版本，在某些情况下取决于更改流打开/恢复时的功能兼容性版本（fcv）（即fcv值的更改不会影响已打开的更改流的恢复令牌）：

| MongoDB版本             | 功能兼容性版本 | 恢复令牌`_data`类型    |
| :---------------------- | :------------- | :--------------------- |
| MongoDB 4.0.7及更高版本 | "4.0"或"3.6"   | 六角编码字符串（`v1`） |
| MongoDB 4.0.6及更早版本 | "4.0"          | 六角编码字符串（`v0`） |
| MongoDB 4.0.6及更早版本 | "3.6"          | BinData                |
| MongoDB 3.6             | "3.6"          | BinData                |

* 从MongoDB 4.0.7或更高版本降级时，客户端无法使用从4.0.7+部署返回的恢复令牌。要恢复更改流，客户端需要使用4.0.7之前的升级恢复令牌（如果有）。否则，客户将需要启动新的更改流。
* 从MongoDB 4.0.6或更低版本降级时，客户端可以使用从4.0部署返回的BinData恢复令牌，但不能使用`v0`令牌。

## 创建备份

*可选但推荐。*创建数据库的备份。

## 先决条件

在降级二进制文件之前，您必须降级功能兼容性版本，并删除与3.6或更低版本[不兼容](https://www.mongodb.com/docs/upcoming/release-notes/4.0-compatibility/#std-label-4.0-compatibility-enabled)的任何4.0功能，概述如下所述。只有当`featureCompatibilityVersion`被设置为`"4.0"`时，这些步骤才是必要的。

### 1、降级功能兼容性版本

> 提示：
>
> - 确保没有进行初始同步。在初始同步进行时，Running[`setFeatureCompatibilityVersion`](https://www.mongodb.com/docs/upcoming/reference/command/setFeatureCompatibilityVersion/#mongodb-dbcommand-dbcmd.setFeatureCompatibilityVersion)命令将导致初始同步重新启动。
> - 确保没有副本集成员处于[`ROLLBACK`](https://www.mongodb.com/docs/upcoming/reference/replica-states/#mongodb-replstate-replstate.ROLLBACK)或[`RECOVERING`](https://www.mongodb.com/docs/upcoming/reference/replica-states/#mongodb-replstate-replstate.RECOVERING)状态。

1. 将mongo shell连接到主服务器。

2. 降级`featureCompatibilityVersion`为`"3.6"`

   ```
   db.adminCommand({setFeatureCompatibilityVersion: "3.6"})
   ```

   [`setFeatureCompatibilityVersion`](https://www.mongodb.com/docs/upcoming/reference/command/setFeatureCompatibilityVersion/#mongodb-dbcommand-dbcmd.setFeatureCompatibilityVersion)命令执行对内部系统集合的写入，并且是幂等的。如果由于任何原因该命令未能成功完成，请在主命令上重试该命令。

为了确保副本集的所有成员都反映更新的`featureCompatibilityVersion`，请连接到每个副本集成员并检查`featureCompatibilityVersion`：

```
db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
```

所有成员都应返回一个结果，其中包括：

```
"featureCompatibilityVersion" : { "version" : "3.6" }
```

如果任何成员返回包含`"4.0"`的`version`值或`targetVersion`字段`featureCompatibilityVersion`，请等待成员反映版本`"3.6"`后再继续。

有关返回的`featureCompatibilityVersion`值的详细信息，请参阅 [Get FeatureCompatibilityVersion.](https://www.mongodb.com/docs/upcoming/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)

### 2、移除向后不兼容的持久功能

删除所有与4.0[不兼容](https://www.mongodb.com/docs/upcoming/release-notes/4.0-compatibility/#std-label-4.0-compatibility-enabled)的持久功能。例如，如果您定义了使用4.0查询功能（如[聚合转换运算符）](https://www.mongodb.com/docs/upcoming/release-notes/4.0/#std-label-4.0-agg-type-conversion)的任何视图定义、文档验证器和部分索引过滤器，则必须删除它们。

如果您的用户只有`SCRAM-SHA-256`凭据，您应该在降级之前为这些用户创建`SCRAM-SHA-1`凭据。要更新仅具有`SCRAM-SHA-256`凭据的用户，请运行[`db.updateUser()`](https://www.mongodb.com/docs/upcoming/reference/method/db.updateUser/#mongodb-method-db.updateUser)，其`mechanisms`仅为`SCRAM-SHA-1`，并将`pwd`设置为密码：

```shell
db.updateUser(
   "reportUser256",
   {
     mechanisms: [ "SCRAM-SHA-1" ],
     pwd: <newpwd>
   }
)
```

## 程序

> 警告:
>
> 在继续进行降级过程之前，请确保所有复制集成员，包括延迟的复制集成员，都反映先决条件更改。也就是说，在降级之前，检查`featureCompatibilityVersion`并删除每个节点的不兼容功能。

> 笔记：
>
> 如果您使用包含`SCRAM-SHA-256`[`authenticationMechanisms`](https://www.mongodb.com/docs/upcoming/reference/parameters/#mongodb-parameter-param.authenticationMechanisms)运行MongoDB 4.0，请使用3.6二进制文件重新启动时省略`SCRAM-SHA-256`。

### 1、下载最新的3.6二进制文件。

使用软件包管理器或手动下载，获取3.6系列的最新版本。如果使用软件包管理器，请为3.6二进制文件添加一个新存储库，然后执行实际的降级过程。

一旦升级到4.0，如果您需要降级，我们建议您降级到最新的3.6补丁版本。

### 2、降级副本集的辅助成员。

降级副本集的每个[辅助](https://www.mongodb.com/docs/upcoming/reference/glossary/#std-term-secondary)成员，一次降级一个：

1. 关掉[`mongod`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#mongodb-binary-bin.mongod)。有关安全终止[`mongod`](https://www.mongodb.com/docs/upcoming/tutorial/manage-mongodb-processes/#std-label-terminate-mongod-processes)的说明，请参阅[停止](https://www.mongodb.com/docs/upcoming/tutorial/manage-mongodb-processes/#std-label-terminate-mongod-processes)[`mongod`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#mongodb-binary-bin.mongod)进程。
2. 将4.0二进制文件替换为3.6二进制文件并重新启动。
3. 请等待成员恢复到SECONDARY状态，然后再降级下一个辅助节点。要检查成员的状态，请使用mongo shell中的[`rs.status()`](https://www.mongodb.com/docs/upcoming/reference/method/rs.status/#mongodb-method-rs.status)方法。

### 3、降级主节点

在mongo shell中使用 [`rs.stepDown()`](https://www.mongodb.com/docs/upcoming/reference/method/rs.stepDown/#mongodb-method-rs.stepDown) 来逐步关闭主服务器并强制执行正常的故障转移过程。

```
rs.stepDown()
```

[`rs.stepDown()`](https://www.mongodb.com/docs/upcoming/reference/method/rs.stepDown/#mongodb-method-rs.stepDown)加快故障转移程序，比直接关闭主故障转移更可取。

### 4、更换并重新启动以前的主`mongod`。

当[`rs.status()`](https://www.mongodb.com/docs/upcoming/reference/method/rs.status/#mongodb-method-rs.status)显示主制文件已下台，另一个成员已处于`PRIMARY`状态时，请关闭上一个主主二进制文件，用3.6二进制文件替换为[`mongod`](https://www.mongodb.com/docs/upcoming/reference/program/mongod/#mongodb-binary-bin.mongod)二进制文件，然后启动新实例。

> 笔记：
>
> MongoDB 3.6部署可以使用从对4.0部署打开的更改流返回的BinData恢复令牌，但不能使用`v0`或`v1`十六进制编码的字符串恢复令牌。

放入方法



 参见

原文 - [Downgrade 4.0 Replica Set to 3.6]( https://docs.mongodb.com/manual/release-notes/4.0-downgrade-replica-set/ )

