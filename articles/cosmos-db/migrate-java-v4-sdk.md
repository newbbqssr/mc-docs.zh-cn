---
title: 迁移应用程序以使用 Azure Cosmos DB Java SDK v4 (com.azure.cosmos)
description: 了解如何将现有 Java 应用程序从使用较旧的 Azure Cosmos DB Java SDK 升级到用于 Core (SQL) API 的较新的 Java SDK 4.0（com.azure.cosmos 包）。
ms.custom: devx-track-java
ms.service: cosmos-db
ms.topic: how-to
origin.date: 06/11/2020
author: rockboyfor
ms.date: 11/16/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.reviewer: sngun
ms.openlocfilehash: 18c27d4861d36d253ba7514568d74a596898bb91
ms.sourcegitcommit: 5f07189f06a559d5617771e586d129c10276539e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/12/2020
ms.locfileid: "94552825"
---
<!--Verified successfully-->
# <a name="migrate-your-application-to-use-the-azure-cosmos-db-java-sdk-v4"></a>迁移应用程序以使用 Azure Cosmos DB Java SDK v4
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!IMPORTANT]  
> 有关此 SDK 的详细信息，请查看 Azure Cosmos DB Java SDK v4 [发行说明](sql-api-sdk-java-v4.md)、[Maven 存储库](https://mvnrepository.com/artifact/com.azure/azure-cosmos)、Azure Cosmos DB Java SDK v4 [性能提示](performance-tips-java-sdk-v4-sql.md)和 Azure Cosmos DB Java SDK v4 [故障排除指南](troubleshoot-java-sdk-v4-sql.md)。
>

本文介绍如何将使用较旧 Azure Cosmos DB Java SDK 的现有 Java 应用程序升级为使用较新的适用于 Core (SQL) API 的 Azure Cosmos DB Java SDK 4.0。 Azure Cosmos DB Java SDK v4 对应于 `com.azure.cosmos` 包。 如果要从以下任何 Azure Cosmos DB Java SDK 迁移应用程序，则可以使用此文档中的说明： 

* Sync Java SDK 2.x.x
* Async Java SDK 2.x.x
* Java SDK 3.x.x

## <a name="azure-cosmos-db-java-sdks-and-package-mappings"></a>Azure Cosmos DB Java SDK 和包映射

下表列出了不同的 Azure Cosmos DB Java SDK、包名称和发布信息：

| Java SDK| 发布日期 | 捆绑 API   | Maven Jar  | Java 包名称  |API 参考   | 发行说明  |
|-------|------|-----------|-----------|--------------|-------------|---------------------------|
| Async 2.x.x  | 2018 年 6 月    | Async(RxJava)  | `com.microsoft.azure::azure-cosmosdb` | `com.microsoft.azure.cosmosdb.rx` | [API](https://azure.github.io/azure-cosmosdb-java/2.0.0/) | [发行说明](sql-api-sdk-async-java.md) |
| Sync 2.x.x     | 2018 年 9 月    | 同步   | `com.microsoft.azure::azure-documentdb` | `com.microsoft.azure.cosmosdb` | [API](https://azure.github.io/azure-cosmosdb-java/2.0.0/) | [发行说明](sql-api-sdk-java.md)  |
| 3.x.x    | 2019 年 7 月    | Async(Reactor)/Sync  | `com.microsoft.azure::azure-cosmos`  | `com.azure.data.cosmos` | [API](https://azure.github.io/azure-cosmosdb-java/3.0.0/) | - |
| 4.0   | 2020 年 6 月   | Async(Reactor)/Sync  | `com.azure::azure-cosmos` | `com.azure.cosmos`   | - | [API](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-cosmos/4.0.1/index.html)  |

## <a name="sdk-level-implementation-changes"></a>SDK 级别实现更改

以下是不同 SDK 之间的主要实现差异：

### <a name="rxjava-is-replaced-with-reactor-in-azure-cosmos-db-java-sdk-versions-3xx-and-40"></a>在 Azure Cosmos DB Java SDK 3.x.x 和 4.0 版本中，RxJava 已替换为 Reactor

如果你不熟悉异步编程或响应式编程，请参阅 [Reactor 模式指南](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/reactor-pattern-guide.md)，了解异步编程和 Project Reactor。 如果你过去一直在使用 Azure Cosmos DB Sync Java SDK 2.x.x 或 Azure Cosmos DB Java SDK 3.x.x Sync API，本指南可能很有用。

如果你一直在使用 Azure Cosmos DB Async Java SDK 2.x.x，并计划迁移到 4.0 SDK，请参阅 [Reactor 与 RxJava 指南](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/reactor-rxjava-guide.md)，获取有关将 RxJava 代码转换为使用 Reactor 的指导。

### <a name="azure-cosmos-db-java-sdk-v4-has-direct-connectivity-mode-in-both-async-and-sync-apis"></a>Azure Cosmos DB Java SDK v4 在异步和同步 API 中具有直接连接模式

如果你一直在使用 Azure Cosmos DB Sync Java SDK 2.x.x，请注意，基于 TCP（而不是 HTTP）的直接连接模式已在适用于异步和同步 API 的 Azure Cosmos DB Java SDK 4.0 中实现。

## <a name="api-level-changes"></a>API 级别更改

以下是 Azure Cosmos DB Java SDK 4.x.x 与以前的 SDK（Java SDK 3.x.x、Async Java SDK 2.x.x 和 Sync Java SDK 2.x.x）相比的 API 级别更改：

:::image type="content" source="./media/migrate-java-v4-sdk/java-sdk-naming-conventions.png" alt-text="Azure Cosmos DB Java SDK 命名约定":::

* Azure Cosmos DB Java SDK 3.x.x 和 4.0 将客户端资源作为 `Cosmos<resourceName>` 引用。 例如：`CosmosClient`、`CosmosDatabase`、`CosmosContainer`。 而在版本 2.x.x 中，Azure Cosmos DB Java SDK 没有统一的命名方案。

* Azure Cosmos DB Java SDK 3.x.x 和 4.0 同时提供同步和异步 API。

    * **Java SDK 4.0**：除非类名称在 `Cosmos` 后附加 `Async`，否则所有类都属于同步 API。

    * **Java SDK 3.x.x**：除非类名称在 `Cosmos` 后附加`Async`，否则所有类都属于异步 API。

    * **Async Java SDK 2.x.x**：类名称类似于 Sync Java SDK 2.x.x，但名称以 Async 开头。

### <a name="hierarchical-api-structure"></a>分层 API 结构

Azure Cosmos DB Java SDK 4.0 和 3.x.x 引入了分层 API 结构，该结构以嵌套方式组织客户端、数据库和容器，如以下 4.0 SDK 代码片段所示：

```java
CosmosContainer container = client.getDatabase("MyDatabaseName").getContainer("MyContainerName");

```

在 Azure Cosmos DB Java SDK 版本 2.x.x 中，对资源和文档执行的所有操作都通过客户端实例执行。

### <a name="representing-documents"></a>表示文档

在 Azure Cosmos DB Java SDK 4.0 中，自定义 POJO 和 `JsonNodes` 是从 Azure Cosmos DB 读取和写入文档的两个选项。

在 Azure Cosmos DB Java SDK 3.x.x 中，公共 API 公开 `CosmosItemProperties` 对象并用作文档表示形式。 在版本 4.0 中，不再公开此类。

### <a name="imports"></a>导入

* Azure Cosmos DB Java SDK 4.0 包以 `com.azure.cosmos` 开头
    * Azure Cosmos DB Java SDK 3.x.x 包以 `com.azure.data.cosmos` 开头

* Azure Cosmos DB Java SDK 4.0 将几个类放在嵌套包 `com.azure.cosmos.models` 中。 其中一些包包括：

    * `CosmosContainerResponse`
    * `CosmosDatabaseResponse`
    * `CosmosItemResponse`
    * 所有上述包的异步 API 模拟
    * `CosmosContainerProperties`
    * `FeedOptions`
    * `PartitionKey`
    * `IndexingPolicy`
    * `IndexingMode` 等

### <a name="accessors"></a>访问器

Azure Cosmos DB Java SDK 4.0 公开了访问实例成员 `get` 和 `set` 方法。 例如，`CosmosContainer` 实例具有 `container.getId()` 和 `container.setId()` 方法。

这与 Azure Cosmos DB Java SDK 3.x.x 不同，后者公开了 Fluent 界面。 例如，`CosmosSyncContainer` 实例具有 `container.id()`，重载它可获取或设置 `id` 值。

## <a name="code-snippet-comparisons"></a>代码片段比较

### <a name="create-resources"></a>创建资源

以下代码片段显示了 4.0 和 3.x.x 异步 API 在资源创建方式上的差异：

# <a name="java-sdk-40-async-api"></a>[Java SDK 4.0 异步 API](#tab/java-v4-async)

```java

// Create Async client.
// Building an async client is still a sync operation.
CosmosAsyncClient client = new CosmosClientBuilder()
        .endpoint("your.hostname")
        .key("yourmasterkey")
        .consistencyLevel(ConsistencyLevel.EVENTUAL)
        .buildAsyncClient();

// Create database with specified name
client.createDatabaseIfNotExists("YourDatabaseName")
        .flatMap(databaseResponse -> {
            testDatabaseAsync = client.getDatabase("YourDatabaseName");
            // Container properties - name and partition key
            CosmosContainerProperties containerProperties =
                    new CosmosContainerProperties("YourContainerName", "/id");

            // Provision manual throughput
            ThroughputProperties throughputProperties = ThroughputProperties.createManualThroughput(400);

            // Create container
            return database.createContainerIfNotExists(containerProperties, throughputProperties);
        }).flatMap(containerResponse -> {
    testContainerAsync = database.getContainer("YourContainerName");
    return Mono.empty();
}).subscribe();

```

# <a name="java-sdk-3xx-async-api"></a>[Java SDK 3.x.x 异步 API](#tab/java-v3-async)

```java
ConnectionPolicy defaultPolicy = ConnectionPolicy.defaultPolicy();
//  Setting the preferred location to Cosmos DB Account region
defaultPolicy.preferredLocations(Lists.newArrayList("Your Account Location"));

//  Create async client
//  <CreateAsyncClient>
client = new CosmosClientBuilder()
        .endpoint("your.hostname")
        .key("yourmasterkey")
        .connectionPolicy(defaultPolicy)
        .consistencyLevel(ConsistencyLevel.EVENTUAL)
        .build();

// Create database with specified name
client.createDatabaseIfNotExists("YourDatabaseName")
      .flatMap(databaseResponse -> {
        database = databaseResponse.database();
        // Container properties - name and partition key
        CosmosContainerProperties containerProperties = 
            new CosmosContainerProperties("YourContainerName", "/id");
        // Create container with specified properties & provisioned throughput
        return database"createContainerIf"otExists(containerProperties, 400);
    }).flatMap(containerResponse -> {
        container = containerResponse.container();
        return Mono.empty();
}).subscribe();
```
---

### <a name="item-operations"></a>项操作

以下代码片段显示了 4.0 和 3.x.x 异步 API 在项操作执行方式上的差异：

# <a name="java-sdk-40-async-api"></a>[Java SDK 4.0 异步 API](#tab/java-v4-async)

```java

// Container is created. Generate many docs to insert.
int number_of_docs = 50000;
ArrayList<JsonNode> docs = generateManyDocs(number_of_docs);

// Insert many docs into container...
Flux.fromIterable(docs)
        .flatMap(doc -> testContainerAsync.createItem(doc))
        .subscribe(); // ...Subscribing triggers stream execution.

```

# <a name="java-sdk-3xx-async-api"></a>[Java SDK 3.x.x 异步 API](#tab/java-v3-async)

```java
// Container is created. Generate many docs to insert.
int number_of_docs = 50000;
ArrayList<JsonNode> docs = generateManyDocs(number_of_docs);

// Insert many docs into container...
Flux.fromIterable(docs)
    .flatMap(doc -> container.createItem(doc))
    .subscribe(); // ...Subscribing triggers stream execution.
```
---

### <a name="indexing"></a>索引

以下代码片段显示了 4.0 和 3.x.x 异步 API 在索引创建方式上的差异：

# <a name="java-sdk-40-async-api"></a>[Java SDK 4.0 异步 API](#tab/java-v4-async)

```java

CosmosContainerProperties containerProperties = new CosmosContainerProperties(containerName, "/lastName");

// Custom indexing policy
IndexingPolicy indexingPolicy = new IndexingPolicy();
indexingPolicy.setIndexingMode(IndexingMode.CONSISTENT);

// Included paths
List<IncludedPath> includedPaths = new ArrayList<>();
includedPaths.add(new IncludedPath("/*"));
indexingPolicy.setIncludedPaths(includedPaths);

// Excluded paths
List<ExcludedPath> excludedPaths = new ArrayList<>();
excludedPaths.add(new ExcludedPath("/name/*"));
indexingPolicy.setExcludedPaths(excludedPaths);

containerProperties.setIndexingPolicy(indexingPolicy);

ThroughputProperties throughputProperties = ThroughputProperties.createManualThroughput(400);

database.createContainerIfNotExists(containerProperties, throughputProperties);
CosmosAsyncContainer containerIfNotExists = database.getContainer(containerName);

```

# <a name="java-sdk-3xx-async-api"></a>[Java SDK 3.x.x 异步 API](#tab/java-v3-async)

```java
CosmosContainerProperties containerProperties = new CosmosContainerProperties(containerName, "/lastName");

// Custom indexing policy
IndexingPolicy indexingPolicy = new IndexingPolicy();
indexingPolicy.setIndexingMode(IndexingMode.CONSISTENT); //To turn indexing off set IndexingMode.NONE

// Included paths
List<IncludedPath> includedPaths = new ArrayList<>();
IncludedPath includedPath = new IncludedPath();
includedPath.path("/*");
includedPaths.add(includedPath);
indexingPolicy.setIncludedPaths(includedPaths);

// Excluded paths
List<ExcludedPath> excludedPaths = new ArrayList<>();
ExcludedPath excludedPath = new ExcludedPath();
excludedPath.path("/name/*");
excludedPaths.add(excludedPath);
indexingPolicy.excludedPaths(excludedPaths);

containerProperties.indexingPolicy(indexingPolicy);

CosmosContainer containerIfNotExists = database.createContainerIfNotExists(containerProperties, 400)
                                               .block()
                                               .container();
```
---

### <a name="stored-procedures"></a>存储过程

以下代码片段显示了 4.0 和 3.x.x 异步 API 在存储过程创建方式上的差异：

# <a name="java-sdk-40-async-api"></a>[Java SDK 4.0 异步 API](#tab/java-v4-async)

```java

logger.info("Creating stored procedure...\n");

String sprocId = "createMyDocument";

String sprocBody = "function createMyDocument() {\n" +
        "var documentToCreate = {\"id\":\"test_doc\"}\n" +
        "var context = getContext();\n" +
        "var collection = context.getCollection();\n" +
        "var accepted = collection.createDocument(collection.getSelfLink(), documentToCreate,\n" +
        "    function (err, documentCreated) {\n" +
        "if (err) throw new Error('Error' + err.message);\n" +
        "context.getResponse().setBody(documentCreated.id)\n" +
        "});\n" +
        "if (!accepted) return;\n" +
        "}";

CosmosStoredProcedureProperties storedProcedureDef = new CosmosStoredProcedureProperties(sprocId, sprocBody);
container.getScripts()
        .createStoredProcedure(storedProcedureDef,
                new CosmosStoredProcedureRequestOptions()).block();

// ...

logger.info(String.format("Executing stored procedure %s...\n\n", sprocId));

CosmosStoredProcedureRequestOptions options = new CosmosStoredProcedureRequestOptions();
options.setPartitionKey(new PartitionKey("test_doc"));

container.getScripts()
        .getStoredProcedure(sprocId)
        .execute(null, options)
        .flatMap(executeResponse -> {
            logger.info(String.format("Stored procedure %s returned %s (HTTP %d), at cost %.3f RU.\n",
                    sprocId,
                    executeResponse.getResponseAsString(),
                    executeResponse.getStatusCode(),
                    executeResponse.getRequestCharge()));
            return Mono.empty();
        }).block();

```

# <a name="java-sdk-3xx-async-api"></a>[Java SDK 3.x.x 异步 API](#tab/java-v3-async)

```java
logger.info("Creating stored procedure...\n");

sprocId = "createMyDocument";
String sprocBody = "function createMyDocument() {\n" +
        "var documentToCreate = {\"id\":\"test_doc\"}\n" +
        "var context = getContext();\n" +
        "var collection = context.getCollection();\n" +
        "var accepted = collection.createDocument(collection.getSelfLink(), documentToCreate,\n" +
        "    function (err, documentCreated) {\n" +
        "if (err) throw new Error('Error' + err.message);\n" +
        "context.getResponse().setBody(documentCreated.id)\n" +
        "});\n" +
        "if (!accepted) return;\n" +
        "}";
CosmosStoredProcedureProperties storedProcedureDef = new CosmosStoredProcedureProperties(sprocId, sprocBody);
container.getScripts()
        .createStoredProcedure(storedProcedureDef,
                new CosmosStoredProcedureRequestOptions()).block();

// ...

logger.info(String.format("Executing stored procedure %s...\n\n", sprocId));

CosmosStoredProcedureRequestOptions options = new CosmosStoredProcedureRequestOptions();
options.partitionKey(new PartitionKey("test_doc"));

container.getScripts()
        .getStoredProcedure(sprocId)
        .execute(null, options)
        .flatMap(executeResponse -> {
            logger.info(String.format("Stored procedure %s returned %s (HTTP %d), at cost %.3f RU.\n",
                    sprocId,
                    executeResponse.responseAsString(),
                    executeResponse.statusCode(),
                    executeResponse.requestCharge()));
            return Mono.empty();
        }).block();
```
---

### <a name="change-feed"></a>更改源

以下代码片段显示了 4.0 和 3.x.x 异步 API 在更改源操作执行方式上的差异：

# <a name="java-sdk-40-async-api"></a>[Java SDK 4.0 异步 API](#tab/java-v4-async)

```java

ChangeFeedProcessor changeFeedProcessorInstance =
        new ChangeFeedProcessorBuilder()
                .hostName(hostName)
                .feedContainer(feedContainer)
                .leaseContainer(leaseContainer)
                .handleChanges((List<JsonNode> docs) -> {
                    logger.info("--->setHandleChanges() START");

                    for (JsonNode document : docs) {
                        try {
                            //Change Feed hands the document to you in the form of a JsonNode
                            //As a developer you have two options for handling the JsonNode document provided to you by Change Feed
                            //One option is to operate on the document in the form of a JsonNode, as shown below. This is great
                            //especially if you do not have a single uniform data model for all documents.
                            logger.info("---->DOCUMENT RECEIVED: " + OBJECT_MAPPER.writerWithDefaultPrettyPrinter()
                                    .writeValueAsString(document));

                            //You can also transform the JsonNode to a POJO having the same structure as the JsonNode,
                            //as shown below. Then you can operate on the POJO.
                            CustomPOJO pojo_doc = OBJECT_MAPPER.treeToValue(document, CustomPOJO.class);
                            logger.info("----=>id: " + pojo_doc.getId());

                        } catch (JsonProcessingException e) {
                            e.printStackTrace();
                        }
                    }
                    logger.info("--->handleChanges() END");

                })
                .buildChangeFeedProcessor();

// ...

changeFeedProcessorInstance.start()
        .subscribeOn(Schedulers.elastic())
        .subscribe();

```

# <a name="java-sdk-3xx-async-api"></a>[Java SDK 3.x.x 异步 API](#tab/java-v3-async)

```java
ChangeFeedProcessor changeFeedProcessorInstance = 
ChangeFeedProcessor.Builder()
        .hostName(hostName)
        .feedContainer(feedContainer)
        .leaseContainer(leaseContainer)
        .handleChanges((List<CosmosItemProperties> docs) -> {
            logger.info("--->setHandleChanges() START");

            for (CosmosItemProperties document : docs) {
                try {

                    // You are given the document as a CosmosItemProperties instance which you may
                    // cast to the desired type.
                    CustomPOJO pojo_doc = document.getObject(CustomPOJO.class);
                    logger.info("----=>id: " + pojo_doc.id());

                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            logger.info("--->handleChanges() END");

        })
        .build();

// ...

 changeFeedProcessorInstance.start()
                            .subscribeOn(Schedulers.elastic())
                            .subscribe();
```
---

### <a name="container-level-time-to-livettl"></a>容器级别生存时间 (TTL)

以下代码片段显示了 4.0 和 3.x.x 异步 API 在容器数据生存时间创建方式上的差异：

# <a name="java-sdk-40-async-api"></a>[Java SDK 4.0 异步 API](#tab/java-v4-async)

```java

CosmosAsyncContainer container;

// Create a new container with TTL enabled with default expiration value
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
containerProperties.setDefaultTimeToLiveInSeconds(90 * 60 * 60 * 24);
ThroughputProperties throughputProperties = ThroughputProperties.createManualThroughput(400);
database.createContainerIfNotExists(containerProperties, throughputProperties).block();
container = database.getContainer("myContainer");

```

# <a name="java-sdk-3xx-async-api"></a>[Java SDK 3.x.x 异步 API](#tab/java-v3-async)

```java
CosmosContainer container;

// Create a new container with TTL enabled with default expiration value
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
containerProperties.defaultTimeToLive(90 * 60 * 60 * 24);
container = database.createContainerIfNotExists(containerProperties, 400).block().container();
```
---

### <a name="item-level-time-to-livettl"></a>项级别生存时间 (TTL)

以下代码片段显示了 4.0 和 3.x.x 异步 API 在项生存时间创建方式上的差异：

# <a name="java-sdk-40-async-api"></a>[Java SDK 4.0 异步 API](#tab/java-v4-async)

```java

// Include a property that serializes to "ttl" in JSON
class SalesOrder
{
    private String id;
    private String customerId;
    private Integer ttl;

    public SalesOrder(String id, String customerId, Integer ttl) {
        this.id = id;
        this.customerId = customerId;
        this.ttl = ttl;
    }

    public String getId() {return this.id;}
    public void setId(String new_id) {this.id = new_id;}
    public String getCustomerId() {return this.customerId;}
    public void setCustomerId(String new_cid) {this.customerId = new_cid;}
    public Integer getTtl() {return this.ttl;}
    public void setTtl(Integer new_ttl) {this.ttl = new_ttl;}

    //...
}

```

```java

// Set the value to the expiration in seconds
SalesOrder salesOrder = new SalesOrder(
        "SO05",
        "CO18009186470",
        60 * 60 * 24 * 30  // Expire sales orders in 30 days
);

```

# <a name="java-sdk-3xx-async-api"></a>[Java SDK 3.x.x 异步 API](#tab/java-v3-async)

```java
// Include a property that serializes to "ttl" in JSON
public class SalesOrder
{
    private String id;
    private String customerId;
    private Integer ttl;

    public SalesOrder(String id, String customerId, Integer ttl) {
        this.id = id;
        this.customerId = customerId;
        this.ttl = ttl;
    }

    public String id() {return this.id;}
    public SalesOrder id(String new_id) {this.id = new_id; return this;}
    public String customerId() {return this.customerId; return this;}
    public SalesOrder customerId(String new_cid) {this.customerId = new_cid;}
    public Integer ttl() {return this.ttl;}
    public SalesOrder ttl(Integer new_ttl) {this.ttl = new_ttl; return this;}

    //...
}

// Set the value to the expiration in seconds
SalesOrder salesOrder = new SalesOrder(
    "SO05",
    "CO18009186470",
    60 * 60 * 24 * 30  // Expire sales orders in 30 days
);
```
---

## <a name="next-steps"></a>后续步骤

* 使用 V4 SDK [生成 Java 应用](create-sql-api-java.md)以管理 Azure Cosmos DB SQL API 数据
* 了解[基于 Reactor 的 Java SDK](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/reactor-pattern-guide.md)
* 了解如何通过 [Reactor 与 RxJava 指南](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/reactor-rxjava-guide.md)将 RxJava 异步代码转换为 Reactor 异步代码

<!-- Update_Description: update meta properties, wording update, update link -->