---
title: 快速入门：将 Java 和 JDBC 与 Azure Database for MySQL 配合使用
description: 了解如何将 Java 和 JDBC 与 Azure Database for MySQL 数据库配合使用。
author: WenJason
ms.author: v-jay
ms.service: mysql
ms.custom: mvc, devcenter, devx-track-azurecli
ms.topic: quickstart
ms.devlang: java
origin.date: 08/17/2020
ms.date: 11/23/2020
ms.openlocfilehash: 9ec12be2e016ac0fc59186f346b225dcbf25f9ec
ms.sourcegitcommit: c2c9dc65b886542d220ae17afcb1d1ab0a941932
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "94977143"
---
# <a name="quickstart-use-java-and-jdbc-with-azure-database-for-mysql"></a>快速入门：将 Java 和 JDBC 与 Azure Database for MySQL 配合使用

本主题演示如何创建示例应用程序，使其使用 Java 和 [JDBC](https://en.wikipedia.org/wiki/Java_Database_Connectivity) 在 [Azure Database for MySQL](./index.yml) 中存储和检索信息。

JDBC 是标准的 Java API，用于连接到传统的关系数据库。

## <a name="prerequisites"></a>先决条件

- 一个 Azure 帐户。 如果没有帐户，可[获取试用帐户](https://wd.azure.cn/zh-cn/pricing/1rmb-trial-full/?form-type=identityauth)。
- [Azure CLI](/cli/azure/install-azure-cli)。
- 一个受支持的 [Java 开发工具包](https://docs.microsoft.com/azure/developer/java/fundamentals/java-jdk-long-term-support)，版本 8。
- [Apache Maven](https://maven.apache.org/) 生成工具。

## <a name="prepare-the-working-environment"></a>准备工作环境

我们将使用环境变量来减少键入错误，并使你能够更轻松地根据特定需求自定义以下配置。

使用以下命令设置这些环境变量：

```bash
AZ_RESOURCE_GROUP=database-workshop
AZ_DATABASE_NAME=<YOUR_DATABASE_NAME>
AZ_LOCATION=<YOUR_AZURE_REGION>
AZ_MYSQL_USERNAME=demo
AZ_MYSQL_PASSWORD=<YOUR_MYSQL_PASSWORD>
AZ_LOCAL_IP_ADDRESS=<YOUR_LOCAL_IP_ADDRESS>
```

使用以下值替换占位符，在本文中将使用这些值：

- `<YOUR_DATABASE_NAME>`：MySQL 服务器的名称。 它在 Azure 中应是唯一的。
- `<YOUR_AZURE_REGION>`：将使用的 Azure 区域。 建议配置一个离居住位置更近的区域。 可输入 `az account list-locations`，获取可用区域的完整列表。
- `<YOUR_MYSQL_PASSWORD>`：MySQL 数据库服务器的密码。 该密码应该至少有八个字符。 这些字符应该属于以下类别中的三个类别：英文大写字母、英文小写字母、数字 (0-9)和非字母数字字符（!, $, #, % 等）。
- `<YOUR_LOCAL_IP_ADDRESS>`：你将在其中运行 Java 应用程序的本地计算机的 IP 地址。 若要找到该地址，一种简便方法是使浏览器指向 [whatismyip.akamai.com](http://whatismyip.akamai.com/)。

接下来，创建一个资源组：

```azurecli
az group create \
    --name $AZ_RESOURCE_GROUP \
    --location $AZ_LOCATION \
```

## <a name="create-an-azure-database-for-mysql-instance"></a>创建 Azure Database for MySQL 实例

首先，我们将创建一个托管 MySQL 服务器。

> [!NOTE]
> 可以参阅[使用 Azure 门户创建 Azure Database for MySQL 服务器](./quickstart-create-mysql-server-database-using-azure-portal.md)，详细了解如何创建 MySQL 服务器。

运行以下脚本：

```azurecli
az mysql server create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name $AZ_DATABASE_NAME \
    --location $AZ_LOCATION \
    --sku-name B_Gen5_1 \
    --storage-size 5120 \
    --admin-user $AZ_MYSQL_USERNAME \
    --admin-password $AZ_MYSQL_PASSWORD \
```

此命令创建一个小型 MySQL 服务器。

### <a name="configure-a-firewall-rule-for-your-mysql-server"></a>为 MySQL 服务器配置防火墙规则

Azure Database for MySQL 实例在默认情况下受保护。 它们有不允许任何传入连接的防火墙。 为了能够使用数据库，需要添加一项防火墙规则，允许本地 IP 地址访问数据库服务器。

由于我们已在本文开头配置了本地 IP 地址，因此可通过运行以下命令来打开服务器的防火墙：

```azurecli
az mysql server firewall-rule create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name $AZ_DATABASE_NAME-database-allow-local-ip \
    --server $AZ_DATABASE_NAME \
    --start-ip-address $AZ_LOCAL_IP_ADDRESS \
    --end-ip-address $AZ_LOCAL_IP_ADDRESS \
```

### <a name="configure-a-mysql-database"></a>配置 MySQL 数据库

之前创建的 MySQL 服务器为空。 它没有任何可以与 Java 应用程序配合使用的数据库。 创建一个名为 `demo`的新数据库：

```azurecli
az mysql db create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name demo \
    --server-name $AZ_DATABASE_NAME \
```

### <a name="create-a-new-java-project"></a>新建一个 Java 项目

使用最喜欢的 IDE 创建一个新的 Java 项目，并在其根目录中添加一个 `pom.xml` 文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>

    <properties>
        <java.version>1.8</java.version>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.20</version>
        </dependency>
    </dependencies>
</project>
```

此文件是一个 [Apache Maven](https://maven.apache.org/)，用于配置要使用的项目：

- Java 8
- 最新的用于 Java 的 MySQL 驱动程序

### <a name="prepare-a-configuration-file-to-connect-to-azure-database-for-mysql"></a>准备用于连接到 Azure Database for MySQL 的配置文件

创建 src/main/resources/application.properties 文件，添加以下内容：

```properties
url=jdbc:mysql://$AZ_DATABASE_NAME.mysql.database.chinacloudapi.cn:3306/demo?serverTimezone=UTC
user=demo@$AZ_DATABASE_NAME
password=$AZ_MYSQL_PASSWORD
```

- 将两个 `$AZ_DATABASE_NAME` 变量替换为在本文开头部分配置的值。
- 将 `$AZ_MYSQL_PASSWORD` 变量替换为在本文开头部分配置的值。

> [!NOTE]
> 我们将 `?serverTimezone=UTC` 追加到配置属性 `url` 中，以指示 JDBC 驱动程序在连接到数据库时使用 UTC 日期格式（或协调世界时）。 否则，Java 服务器将不使用与数据库相同的日期格式，这将导致错误。

### <a name="create-an-sql-file-to-generate-the-database-schema"></a>创建 SQL 文件以生成数据库架构

我们将使用 src/main/resources/`schema.sql` 文件来创建数据库架构。 创建包含以下内容的文件：

```sql
DROP TABLE IF EXISTS todo;
CREATE TABLE todo (id SERIAL PRIMARY KEY, description VARCHAR(255), details VARCHAR(4096), done BOOLEAN);
```

## <a name="code-the-application"></a>编写应用程序代码

### <a name="connect-to-the-database"></a>连接到数据库

接下来添加 Java 代码，以便使用 JDBC 在 MySQL 服务器中存储并检索数据。

创建 src/main/java/DemoApplication.java 文件，其中包含：

```java
package com.example.demo;

import com.mysql.cj.jdbc.AbandonedConnectionCleanupThread;

import java.sql.*;
import java.util.*;
import java.util.logging.Logger;

public class DemoApplication {

    private static final Logger log;

    static {
        System.setProperty("java.util.logging.SimpleFormatter.format", "[%4$-7s] %5$s %n");
        log =Logger.getLogger(DemoApplication.class.getName());
    }

    public static void main(String[] args) throws Exception {
        log.info("Loading application properties");
        Properties properties = new Properties();
        properties.load(DemoApplication.class.getClassLoader().getResourceAsStream("application.properties"));

        log.info("Connecting to the database");
        Connection connection = DriverManager.getConnection(properties.getProperty("url"), properties);
        log.info("Database connection test: " + connection.getCatalog());

        log.info("Create database schema");
        Scanner scanner = new Scanner(DemoApplication.class.getClassLoader().getResourceAsStream("schema.sql"));
        Statement statement = connection.createStatement();
        while (scanner.hasNextLine()) {
            statement.execute(scanner.nextLine());
        }

        /*
        Todo todo = new Todo(1L, "configuration", "congratulations, you have set up JDBC correctly!", true);
        insertData(todo, connection);
        todo = readData(connection);
        todo.setDetails("congratulations, you have updated data!");
        updateData(todo, connection);
        deleteData(todo, connection);
        */

        log.info("Closing database connection");
        connection.close();
        AbandonedConnectionCleanupThread.uncheckedShutdown();
    }
}
```

此 Java 代码将使用之前创建的“application.properties”和“schema.sql”文件，来连接到 MySQL 服务器并创建用于存储数据的架构 。

在此文件中，你可以看到，我们注释了用于插入、读取、更新和删除数据的方法：我们将在本文的其余部分对这些方法进行编码，并且你将能够逐个取消注释。

> [!NOTE]
> 数据库凭据存储在“application.properties”文件的“user”和“password”属性中  。 执行 `DriverManager.getConnection(properties.getProperty("url"), properties);` 时使用这些凭据，因为属性文件是作为参数传递的。

> [!NOTE]
> 结尾处的 `AbandonedConnectionCleanupThread.uncheckedShutdown();` 行是 MySQL 驱动程序特定的命令，用于在关闭应用程序时销毁内部线程。
> 可放心忽略此警告。 

现在可以通过喜欢的工具执行此主类：

- 使用你的 IDE，你应该能够右键单击 DemoApplication 类并执行它。
- 使用 Maven，可以通过执行以下操作来运行应用程序：`mvn exec:java -Dexec.mainClass="com.example.demo.DemoApplication"`。

应用程序应连接到 Azure Database for MySQL，创建数据库架构，然后关闭连接，如控制台日志中所示：

```
[INFO   ] Loading application properties 
[INFO   ] Connecting to the database 
[INFO   ] Database connection test: demo 
[INFO   ] Create database schema 
[INFO   ] Closing database connection 
```

### <a name="create-a-domain-class"></a>创建域类

在 `DemoApplication` 类旁创建新的 `Todo` Java 类并添加以下代码：

```java
package com.example.demo;

public class Todo {

    private Long id;
    private String description;
    private String details;
    private boolean done;

    public Todo() {
    }

    public Todo(Long id, String description, String details, boolean done) {
        this.id = id;
        this.description = description;
        this.details = details;
        this.done = done;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getDetails() {
        return details;
    }

    public void setDetails(String details) {
        this.details = details;
    }

    public boolean isDone() {
        return done;
    }

    public void setDone(boolean done) {
        this.done = done;
    }

    @Override
    public String toString() {
        return "Todo{" +
                "id=" + id +
                ", description='" + description + '\'' +
                ", details='" + details + '\'' +
                ", done=" + done +
                '}';
    }
}
```

此类是在执行 schema .sql 脚本时创建的 `todo` 表上映射的域模型。

### <a name="insert-data-into-azure-database-for-mysql"></a>将数据插入到 Azure Database for MySQL

在 src/main/java/DemoApplication.java 文件中，在 main 方法之后添加以下方法，以将数据插入数据库：

```java
private static void insertData(Todo todo, Connection connection) throws SQLException {
    log.info("Insert data");
    PreparedStatement insertStatement = connection
            .prepareStatement("INSERT INTO todo (id, description, details, done) VALUES (?, ?, ?, ?);");

    insertStatement.setLong(1, todo.getId());
    insertStatement.setString(2, todo.getDescription());
    insertStatement.setString(3, todo.getDetails());
    insertStatement.setBoolean(4, todo.isDone());
    insertStatement.executeUpdate();
}
```

你现在可以对 `main` 方法中的以下两行执行取消注释操作：

```java
Todo todo = new Todo(1L, "configuration", "congratulations, you have set up JDBC correctly!", true);
insertData(todo, connection);
```

现在，执行主类应会生成以下输出：

```
[INFO   ] Loading application properties 
[INFO   ] Connecting to the database 
[INFO   ] Database connection test: demo 
[INFO   ] Create database schema 
[INFO   ] Insert data 
[INFO   ] Closing database connection
```

### <a name="reading-data-from-azure-database-for-mysql"></a>从 Azure Database for MySQL 读取数据

接下来，请阅读前面插入的数据，验证代码是否正常工作。

在 src/main/java/DemoApplication.java 文件中，在 `insertData` 方法之后添加以下方法，以从数据库中读取数据：

```java
private static Todo readData(Connection connection) throws SQLException {
    log.info("Read data");
    PreparedStatement readStatement = connection.prepareStatement("SELECT * FROM todo;");
    ResultSet resultSet = readStatement.executeQuery();
    if (!resultSet.next()) {
        log.info("There is no data in the database!");
        return null;
    }
    Todo todo = new Todo();
    todo.setId(resultSet.getLong("id"));
    todo.setDescription(resultSet.getString("description"));
    todo.setDetails(resultSet.getString("details"));
    todo.setDone(resultSet.getBoolean("done"));
    log.info("Data read from the database: " + todo.toString());
    return todo;
}
```

你现在可以对 `main` 方法中的以下行执行取消注释操作：

```java
todo = readData(connection);
```

现在，执行主类应会生成以下输出：

```
[INFO   ] Loading application properties 
[INFO   ] Connecting to the database 
[INFO   ] Database connection test: demo 
[INFO   ] Create database schema 
[INFO   ] Insert data 
[INFO   ] Read data 
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have set up JDBC correctly!', done=true} 
[INFO   ] Closing database connection 
```

### <a name="updating-data-in-azure-database-for-mysql"></a>更新 Azure Database for MySQL 中的数据

让我们更新之前插入的数据。

在 src/main/java/DemoApplication.java 文件中，在 `readData` 方法之后添加以下方法，以更新数据库中的数据：

```java
private static void updateData(Todo todo, Connection connection) throws SQLException {
    log.info("Update data");
    PreparedStatement updateStatement = connection
            .prepareStatement("UPDATE todo SET description = ?, details = ?, done = ? WHERE id = ?;");

    updateStatement.setString(1, todo.getDescription());
    updateStatement.setString(2, todo.getDetails());
    updateStatement.setBoolean(3, todo.isDone());
    updateStatement.setLong(4, todo.getId());
    updateStatement.executeUpdate();
    readData(connection);
}
```

你现在可以对 `main` 方法中的以下两行执行取消注释操作：

```java
todo.setDetails("congratulations, you have updated data!");
updateData(todo, connection);
```

现在，执行主类应会生成以下输出：

```
[INFO   ] Loading application properties 
[INFO   ] Connecting to the database 
[INFO   ] Database connection test: demo 
[INFO   ] Create database schema 
[INFO   ] Insert data 
[INFO   ] Read data 
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have set up JDBC correctly!', done=true} 
[INFO   ] Update data 
[INFO   ] Read data 
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have updated data!', done=true} 
[INFO   ] Closing database connection 
```

### <a name="deleting-data-in-azure-database-for-mysql"></a>删除 Azure Database for MySQL 中的数据

最后，让我们删除之前插入的数据。

在 src/main/java/DemoApplication.java 文件中，在 `updateData` 方法之后添加以下方法，以删除数据库中的数据：

```java
private static void deleteData(Todo todo, Connection connection) throws SQLException {
    log.info("Delete data");
    PreparedStatement deleteStatement = connection.prepareStatement("DELETE FROM todo WHERE id = ?;");
    deleteStatement.setLong(1, todo.getId());
    deleteStatement.executeUpdate();
    readData(connection);
}
```

你现在可以对 `main` 方法中的以下行执行取消注释操作：

```java
deleteData(todo, connection);
```

现在，执行主类应会生成以下输出：

```
[INFO   ] Loading application properties 
[INFO   ] Connecting to the database 
[INFO   ] Database connection test: demo 
[INFO   ] Create database schema 
[INFO   ] Insert data 
[INFO   ] Read data 
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have set up JDBC correctly!', done=true} 
[INFO   ] Update data 
[INFO   ] Read data 
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have updated data!', done=true} 
[INFO   ] Delete data 
[INFO   ] Read data 
[INFO   ] There is no data in the database! 
[INFO   ] Closing database connection 
```

## <a name="clean-up-resources"></a>清理资源

祝贺你！ 你已创建了一个 Java 应用程序，该应用程序使用 JDBC 在 Azure Database for MySQL 中存储和检索数据。

若要清理本快速入门中使用的所有资源，请使用以下命令删除该资源组：

```azurecli
az group delete \
    --name $AZ_RESOURCE_GROUP \
    --yes
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用转储和还原将 MySQL 数据库迁移到 Azure Database for MySQL](concepts-migrate-dump-restore.md)
