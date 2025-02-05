## 自动加密支持的操作

此页面记录了为自动客户端字段级加密配置的驱动程序支持的特定命令、查询运算符、更新运算符、聚合阶段和聚合表达式。

## 支持的读写命令

使用自动客户端字段级加密的驱动程序支持以下命令：

- [`aggregate`](https://www.mongodb.com/docs/manual/reference/command/aggregate/#mongodb-dbcommand-dbcmd.aggregate)
- [`count`](https://www.mongodb.com/docs/manual/reference/command/count/#mongodb-dbcommand-dbcmd.count)
- [`delete`](https://www.mongodb.com/docs/manual/reference/command/delete/#mongodb-dbcommand-dbcmd.delete)
- [`distinct`](https://www.mongodb.com/docs/manual/reference/command/distinct/#mongodb-dbcommand-dbcmd.distinct)
- [`explain`](https://www.mongodb.com/docs/manual/reference/command/explain/#mongodb-dbcommand-dbcmd.explain)
- [`find`](https://www.mongodb.com/docs/manual/reference/command/find/#mongodb-dbcommand-dbcmd.find)
- [`findAndModify`](https://www.mongodb.com/docs/manual/reference/command/findAndModify/#mongodb-dbcommand-dbcmd.findAndModify)
- [`insert`](https://www.mongodb.com/docs/manual/reference/command/insert/#mongodb-dbcommand-dbcmd.insert)
- [`update`](https://www.mongodb.com/docs/manual/reference/command/update/#mongodb-dbcommand-dbcmd.update)

对于任何受支持的命令，如果命令使用不受支持的运算符、聚合阶段或聚合表达式，驱动程序将返回错误。有关支持的运算符、阶段和表达式的完整列表，请参阅本页的以下部分：

- [支持的查询运算符](https://www.mongodb.com/docs/manual/core/csfle/reference/supported-operations/#std-label-csfle-supported-query-operators)
- [支持的更新操作员](https://www.mongodb.com/docs/manual/core/csfle/reference/supported-operations/#std-label-csfle-supported-update-operators)
- [支持的聚合阶段](https://www.mongodb.com/docs/manual/core/csfle/reference/supported-operations/#std-label-csfle-supported-aggregation-stages)
- [支持的聚合表达式](https://www.mongodb.com/docs/manual/core/csfle/reference/supported-operations/#std-label-csfle-supported-aggregation-expressions)

以下命令不需要自动加密。为自动客户端字段级加密配置的驱动程序将这些命令直接传递给 [`mongod`：](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)

* [`getMore`](https://www.mongodb.com/docs/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore) [[ 1 \]](https://www.mongodb.com/docs/manual/core/csfle/reference/supported-operations/#footnote-1)
* [`authenticate`](https://www.mongodb.com/docs/manual/reference/command/authenticate/#mongodb-dbcommand-dbcmd.authenticate)
* [`getnonce`](https://www.mongodb.com/docs/manual/reference/command/getnonce/#mongodb-dbcommand-dbcmd.getnonce)
* [`hello`](https://www.mongodb.com/docs/manual/reference/command/hello/#mongodb-dbcommand-dbcmd.hello)
* [`logout`](https://www.mongodb.com/docs/manual/reference/command/logout/#mongodb-dbcommand-dbcmd.logout)
* [`abortTransaction`](https://www.mongodb.com/docs/manual/reference/command/abortTransaction/#mongodb-dbcommand-dbcmd.abortTransaction)
* [`commitTransaction`](https://www.mongodb.com/docs/manual/reference/command/commitTransaction/#mongodb-dbcommand-dbcmd.commitTransaction)
* [`endSessions`](https://www.mongodb.com/docs/manual/reference/command/endSessions/#mongodb-dbcommand-dbcmd.endSessions)
* [`startSession`](https://www.mongodb.com/docs/manual/reference/command/startSession/#mongodb-dbcommand-dbcmd.startSession)
* [`create`](https://www.mongodb.com/docs/manual/reference/command/create/#mongodb-dbcommand-dbcmd.create)
* [`createIndexes`](https://www.mongodb.com/docs/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes)
* [`drop`](https://www.mongodb.com/docs/manual/reference/command/drop/#mongodb-dbcommand-dbcmd.drop)
* [`dropDatabase`](https://www.mongodb.com/docs/manual/reference/command/dropDatabase/#mongodb-dbcommand-dbcmd.dropDatabase)
* [`dropIndexes`](https://www.mongodb.com/docs/manual/reference/command/dropIndexes/#mongodb-dbcommand-dbcmd.dropIndexes)
* [`killCursors`](https://www.mongodb.com/docs/manual/reference/command/killCursors/#mongodb-dbcommand-dbcmd.killCursors)
* [`listCollections`](https://www.mongodb.com/docs/manual/reference/command/listCollections/#mongodb-dbcommand-dbcmd.listCollections)
* [`listDatabases`](https://www.mongodb.com/docs/manual/reference/command/listDatabases/#mongodb-dbcommand-dbcmd.listDatabases)
* [`listIndexes`](https://www.mongodb.com/docs/manual/reference/command/listIndexes/#mongodb-dbcommand-dbcmd.listIndexes)
* [`renameCollection`](https://www.mongodb.com/docs/manual/reference/command/renameCollection/#mongodb-dbcommand-dbcmd.renameCollection)
* [`ping`](https://www.mongodb.com/docs/manual/reference/command/ping/#mongodb-dbcommand-dbcmd.ping)

通过为自动客户端字段级加密配置的驱动程序发出任何其他[命令](https://www.mongodb.com/docs/manual/reference/command/#std-label-database-commands)会返回错误。

虽然自动客户端字段级加密 (CSFLE) 不加密命令 [`getMore`](https://www.mongodb.com/docs/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore)，但对命令的响应可能包含加密的字段值。

- 配置了正确 CSFLE 选项的应用程序会自动解密这些值。
- 没有正确 CSFLE 选项的应用程序只能看到加密的值。

## 支持的查询运算符

针对[确定性加密](https://www.mongodb.com/docs/manual/core/csfle/fundamentals/encryption-algorithms/#std-label-csfle-deterministic-encryption)字段发出时，为自动客户端字段级加密配置的驱动程序允许使用以下查询运算符：

- [`$eq`](https://www.mongodb.com/docs/manual/reference/operator/query/eq/#mongodb-query-op.-eq)
- [`$ne`](https://www.mongodb.com/docs/manual/reference/operator/query/ne/#mongodb-query-op.-ne)
- [`$in`](https://www.mongodb.com/docs/manual/reference/operator/query/in/#mongodb-query-op.-in)
- [`$nin`](https://www.mongodb.com/docs/manual/reference/operator/query/nin/#mongodb-query-op.-nin)
- [`$and`](https://www.mongodb.com/docs/manual/reference/operator/query/and/#mongodb-query-op.-and)
- [`$or`](https://www.mongodb.com/docs/manual/reference/operator/query/or/#mongodb-query-op.-or)
- [`$not`](https://www.mongodb.com/docs/manual/reference/operator/query/not/#mongodb-query-op.-not)
- [`$nor`](https://www.mongodb.com/docs/manual/reference/operator/query/nor/#mongodb-query-op.-nor)

将加密字段与`null`正则表达式进行比较的查询始终会返回错误，即使使用受支持的查询运算符也是如此。[针对随机加密](https://www.mongodb.com/docs/manual/core/csfle/fundamentals/encryption-algorithms/#std-label-csfle-random-encryption)字段发出这些运算符的查询会返回错误。

[`$exists`](https://www.mongodb.com/docs/manual/reference/operator/query/exists/#mongodb-query-op.-exists)当针对确定性和随机加密字段发出时，运算符具有正常行为。

针对加密字段指定任何其他查询运算符的查询返回错误。

*以下查询运算符即使*未针对加密字段发出，也会返回错误：

- [`$text`](https://www.mongodb.com/docs/manual/reference/operator/query/text/#mongodb-query-op.-text)
- [`$where`](https://www.mongodb.com/docs/manual/reference/operator/query/where/#mongodb-query-op.-where)
- [`$jsonSchema`](https://www.mongodb.com/docs/manual/reference/operator/query/jsonSchema/#mongodb-query-op.-jsonSchema)

> 警告:
>
> **BinData 的意外行为**
>
> MongoDB 将客户端字段级加密字段存储为 [`BinData`](https://www.mongodb.com/docs/manual/reference/mongodb-extended-json-v1/#mongodb-bsontype-data_binary)blob。`BinData`与针对解密值发出相同操作相比，针对加密值发出的读取和写入操作可能具有意外或不正确的行为。某些操作具有严格的 BSON 类型支持，其中针对`BinData`值发出它们会返回错误。
>
> - 使用自动客户端字段级加密的驱动程序解析不支持`BinData`值 *或*在针对值发出时具有意外行为的运算符或表达式的读取和写入操作`BinData` 。
> - 使用显式（手动）客户端字段级加密的应用程序*可以*使用此页面作为针对加密字段发出读取和写入操作的指南。

## 不支持的插入操作

为自动客户端字段级加密配置的驱动程序不*支持*具有以下行为的插入命令：

- `Timestamp(0,0)`插入与加密字段关联的文档。该`(0,0)`值表示 [`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)应该生成时间戳。当 [`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)无法生成加密字段时，生成的时间戳是未加密的。
- `_id` *如果*配置的自动模式指定加密字段，则插入没有加密的文档`_id`。当 [`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod)自动生成未加密的 [ObjectId](https://www.mongodb.com/docs/manual/reference/bson-types/#std-label-objectid)时，`_id`从文档中省略会导致文档不符合自动加密规则。
- [插入一个文档，其中包含一个与确定性加密](https://www.mongodb.com/docs/manual/core/csfle/fundamentals/encryption-algorithms/#std-label-csfle-deterministic-encryption)字段关联的数组 。自动客户端字段级加密不支持确定性加密数组。

## 支持的更新操作员

[针对确定性加密](https://www.mongodb.com/docs/manual/core/csfle/fundamentals/encryption-algorithms/#std-label-csfle-deterministic-encryption)字段发布时，为自动客户端字段级加密配置的驱动程序允许以下更新操作符：

- [`$set`](https://www.mongodb.com/docs/manual/reference/operator/update/set/#mongodb-update-up.-set)
- [`$unset`](https://www.mongodb.com/docs/manual/reference/operator/update/unset/#mongodb-update-up.-unset)
- [`$rename`](https://www.mongodb.com/docs/manual/reference/operator/update/rename/#mongodb-update-up.-rename)

在加密字段上使用[`$rename`](https://www.mongodb.com/docs/manual/reference/operator/update/rename/#mongodb-update-up.-rename)运算符时，自动 JSON 架构必须为源字段名称和目标字段名称指定相同的加密元数据。

针对加密字段指定任何其他更新运算符的更新返回错误。

*即使*使用受支持的运算符，具有以下行为的更新操作也会返回错误 ：

- 更新操作在加密路径内生成一个数组。
- 更新操作使用[聚合表达式语法。](https://www.mongodb.com/docs/manual/release-notes/4.2/#std-label-4.2-update-using-aggregation)

对于在确定性加密字段上指定[查询过滤器的](https://www.mongodb.com/docs/manual/reference/command/update/#std-label-update-command-q)更新操作，查询过滤器必须仅使用[支持的运营商](https://www.mongodb.com/docs/manual/core/csfle/reference/supported-operations/#std-label-csfle-supported-query-operators)在那些领域。

## 支持的聚合阶段

为自动客户端字段级加密配置的驱动程序支持以下聚合管道阶段：

- [`$addFields`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/addFields/#mongodb-pipeline-pipe.-addFields)
- [`$bucket`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/bucket/#mongodb-pipeline-pipe.-bucket)
- [`$bucketAuto`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/bucketAuto/#mongodb-pipeline-pipe.-bucketAuto)
- [`$collStats`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/collStats/#mongodb-pipeline-pipe.-collStats)
- [`$count`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/count/#mongodb-pipeline-pipe.-count)
- [`$geoNear`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear)
- [`$group`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)（使用要求见 [`$group`行为）](https://www.mongodb.com/docs/manual/core/csfle/reference/supported-operations/#std-label-csfle-group-behavior)
- [`$indexStats`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/indexStats/#mongodb-pipeline-pipe.-indexStats)
- [`$limit`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/limit/#mongodb-pipeline-pipe.-limit)
- [`$lookup`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/lookup/#mongodb-pipeline-pipe.-lookup)和[`$graphLookup`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/graphLookup/#mongodb-pipeline-pipe.-graphLookup)（有关使用要求，请参阅[`$lookup`和`$graphLookup`行为）](https://www.mongodb.com/docs/manual/core/csfle/reference/supported-operations/#std-label-csfle-lookup-graphLookup-behavior)
- [`$match`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/match/#mongodb-pipeline-pipe.-match)
- [`$project`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/project/#mongodb-pipeline-pipe.-project)
- [`$redact`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/redact/#mongodb-pipeline-pipe.-redact)
- [`$replaceRoot`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/replaceRoot/#mongodb-pipeline-pipe.-replaceRoot)
- [`$sample`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/sample/#mongodb-pipeline-pipe.-sample)
- [`$skip`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/skip/#mongodb-pipeline-pipe.-skip)
- [`$sort`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/sort/#mongodb-pipeline-pipe.-sort)
- [`$sortByCount`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/sortByCount/#mongodb-pipeline-pipe.-sortByCount)
- [`$unwind`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/unwind/#mongodb-pipeline-pipe.-unwind)

在为指定任何其他阶段的自动加密配置的集合上运行的管道返回错误。

对于每个受支持的管道阶段，MongoDB 跟踪 *必须*加密的字段，因为它们通过受支持的管道并将它们标记为加密。

每个受支持的阶段必须指定仅受支持 [查询运算符](https://www.mongodb.com/docs/manual/core/csfle/reference/supported-operations/#std-label-csfle-supported-query-operators)和 [聚合表达式。](https://www.mongodb.com/docs/manual/core/csfle/reference/supported-operations/#std-label-csfle-supported-aggregation-expressions)

### `$group`行为

[`$group`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)具有以下特定于客户端字段级加密的行为：

[`$group`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)支持：

- 对确定性加密字段进行分组。
- 在加密字段上使用[`$addToSet`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/addToSet/#mongodb-group-grp.-addToSet)和[`$push`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/push/#mongodb-group-grp.-push)累加器。

$group 不支持：

- [`$addToSet`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/addToSet/#mongodb-group-grp.-addToSet)匹配由和 累加器返回的数组 [`$push`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/push/#mongodb-group-grp.-push)。
- 加密字段上的算术累加器。

### `$lookup`和`$graphLookup`行为

自动客户端字段级加密支持[`$lookup`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/lookup/#mongodb-pipeline-pipe.-lookup)且 *仅当*集合与聚合运行所针对的集合匹配时（特别是自查找操作）。[`$graphLookup`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/graphLookup/#mongodb-pipeline-pipe.-graphLookup) `from`

[`$lookup`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/lookup/#mongodb-pipeline-pipe.-lookup)引用[`$graphLookup`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/graphLookup/#mongodb-pipeline-pipe.-graphLookup)不同`from`集合的阶段返回错误。

### 支持的聚合表达式

[为自动客户端字段级加密配置的驱动程序允许对确定性加密](https://www.mongodb.com/docs/manual/core/csfle/fundamentals/encryption-algorithms/#std-label-csfle-deterministic-encryption)字段使用以下表达式的聚合阶段：

- [`$cond`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/cond/#mongodb-expression-exp.-cond)
- [`$eq`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/eq/#mongodb-expression-exp.-eq)
- [`$ifNull`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/ifNull/#mongodb-expression-exp.-ifNull)
- [`$in`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/in/#mongodb-expression-exp.-in)
- [`$let`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/let/#mongodb-expression-exp.-let)
- [`$literal`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/literal/#mongodb-expression-exp.-literal)
- [`$ne`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/ne/#mongodb-expression-exp.-ne)
- [`$switch`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/switch/#mongodb-expression-exp.-switch)

如果针对加密字段发出所有其他聚合表达式，则会返回错误。

*即使*使用受支持的聚合表达式，具有以下行为的聚合阶段也会返回错误 ：

| 表达式                                                       | 拒绝行为                                                     | 例子                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`$cond`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/cond/#mongodb-expression-exp.-cond)[`$switch`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/switch/#mongodb-expression-exp.-switch) | 该表达式指定一个字段，其加密属性直到运行时才为人所知*，并且*后续聚合阶段包括引用该字段的表达式。 | `$addFields : {  "valueWithUnknownEncryption" : {    $cond : {      if : { "$encryptedField" : "value" },      then : "$encryptedField",      else: "unencryptedValue"    }  }},{  $match : {    "valueWithUnknownEncryption" : "someNewValue"  }}` |
| [`$eq`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/eq/#mongodb-expression-exp.-eq)[`$ne`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/ne/#mongodb-expression-exp.-ne) | 该表达式创建一个引用加密字段的新字段*，并*在同一表达式中对该新字段进行操作。 | `{  $eq : [    {"newField" : "$encryptedField"},    {"newField" : "value"  ]}` |
| [`$eq`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/eq/#mongodb-expression-exp.-eq)[`$ne`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/ne/#mongodb-expression-exp.-ne) | 该表达式引用比较表达式中加密字段的前缀。                     | `{ $eq : [ "$prefixOfEncryptedField" , "value"] }`           |
| [`$eq`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/eq/#mongodb-expression-exp.-eq)[`$ne`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/ne/#mongodb-expression-exp.-ne) | 表达式的结果与加密字段进行比较。                             | `{  $eq : [      "$encryptedField" ,      { $ne : [ "field", "value" ] }  ]}` |
| [`$let`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/let/#mongodb-expression-exp.-let) | 表达式将变量绑定到加密字段或尝试重新绑定[`$$CURRENT`。](https://www.mongodb.com/docs/manual/reference/aggregation-variables/#mongodb-variable-variable.CURRENT) | `{  $let: {    "vars" : {      "newVariable" : "$encryptedField"    }  }}` |
| [`$in`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/in/#mongodb-expression-exp.-in) | 表达式的第一个参数*是*一个加密字段， *并且*表达式的第二个参数不是*数组*文字*-或者-*表达式的第二个参数是一个加密字段。 | `{  $in : [    "$encryptedField" ,    "$otherEncryptedField"  ]}` |

## 不支持的字段类型

为自动客户端字段级加密 (CSFLE) 配置的驱动程序不*支持*任何需要加密以下值类型的读取或写入操作：

- [`MaxKey`](https://www.mongodb.com/docs/manual/reference/mongodb-extended-json/#mongodb-bsontype-MaxKey)
- [`MinKey`](https://www.mongodb.com/docs/manual/reference/mongodb-extended-json/#mongodb-bsontype-MinKey)
- `null`
- `undefined`

加密没有充分隐藏这些值的类型信息。

自动 CSFLE*也不*支持对确定性加密字段的读取或写入操作，其中操作将加密字段与以下值类型进行比较：

- `array`
- `bool`
- `decimal128`
- `double`
- `object`
- `javascriptWithScope` *（已弃用）*







译者：韩鹏帅

原文：[Supported Operations for Automatic Encryption](https://www.mongodb.com/docs/manual/core/csfle/reference/supported-operations/)
