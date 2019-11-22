## 什么是Connection
只有在建立连接后才能与数据库进行交互。 TypeORM 的`Connection`不会像看起来那样设置单个数据库连接，而是设置连接池。 如果你对数据库连接感兴趣，请参阅`QueryRunner`文档。 QueryRunner的每个实例都是一个独立的数据库连接。一旦调用`Connection`的`connect`方法，就建立连接池设置。 如果使用`createConnection`函数设置连接，则会自动调用connect方法。调用`close`时会断开连接（关闭池中的所有连接）。 通常情况下，你只能在应用程序启动时创建一次连接，并在完全使用数据库后关闭它。实际上，如果要为站点构建后端，并且后端服务器始终保持运行,则不需要关闭连接。

## 创建新的连接
但是最简单和最常用的方法是使用`createConnection`和`createConnections`函数。
### 创建数据源
1. 传递参数配置
    1.1 创建的单个数据源
    ```js   
    import { createConnection, Connection } from "typeorm";
    const connection = await createConnection({
        type: "mysql",
        host: "localhost",
        port: 3306,
        username: "test",
        password: "test",
        database: "test"
    });
    ```
    1.2 创建多个数据源
    ```js   
    import { createConnections, Connection } from "typeorm";
    const connection = await createConnections([
        {
            name: "default",
            type: "mysql",
            host: "localhost",
            port: 3306,
            username: "test",
            password: "test",
            database: "test"
        }, {
            name: "test2-connection",
            type: "mysql",
            host: "localhost",
            port: 3306,
            username: "test",
            password: "test",
            database: "test"
        }
    ]);
    ```
    
2. 通过配置文件自动读取
### 创建配置文件
在根目录下创建`ormconfig.[format]`文件存放连接配置，并在应用程序中调用createConnection()，而不传递任何参数配置
支持的 ormconfig 文件格式有：.json, .js, .env, .yml 和 .xml.
[不同文件格式配置方式](https://typeorm.io/#/using-ormconfig)

### 文件加载顺序
- 来自环境变量。 Typeorm将尝试使用dotEnv加载.env文件（如果存在）。 如果设置了环境变量`TYPEORM_CONNECTION`或`TYPEORM_URL`，Typeorm将使用此方法。
- 来自`ormconfig.env`。
- 从另一个`ormconfig.[format]`文件，按此顺序：[js，ts，json，yml，yaml，xml]。
注意，Typeorm将使用找到的第一个有效方法，而不会加载其他方法。 例如，如果在环境中找到配置，Typeorm将不会加载ormconfig.[format]文件。

### 代码示例
```js
    import { createConnection, createConnections, Connection } from "typeorm";

    // createConnection将从ormconfig.json / ormconfig.js / ormconfig.yml / ormconfig.env / ormconfig.xml 文件或特殊环境变量中加载连接选项
    const connection: Connection = await createConnection();

    // 你可以指定要创建的连接的名称
    // （如果省略名称，则将创建没有指定名称的连接）
    const secondConnection: Connection = await createConnection("test2-connection");

    // 如果调用createConnections而不是createConnection
    // 它将初始化并返回ormconfig文件中定义的所有连接
    const connections: Connection[] = await createConnections();
```

3. 使用ConnectionManager 
你可以使用`ConnectionManager`类创建连接
```js
    import { getConnectionManager, ConnectionManager,Connection } from "typeorm";

    const connectionManager = getConnectionManager(); // new ConnectionManager()  官方不推荐此写法 你将无法再使用getConnection()你需要存储连接管理器实例，并使用connectionManager.get来获取所需的连接。通常情况下为避免应用程序中出现不必要的复杂情况，应尽量少使用此方法，除非你确实认为需要时才使用ConnectionManager。
    const connection = connectionManager.create({
        type: "mysql",
        host: "localhost",
        port: 3306,
        username: "test",
        password: "test",
        database: "test"
    });
    await connection.connect(); // 执行连接
```

### 获取创建的数据源
设置连接后，可以使用`getConnection`函数在应用程序的任何位置使用它：
不同的连接必须具有不同的名称。默认情况下，如果未指定连接名称，则为default。 通常在你使用多个数据库或多个连接配置时才会使用多连接。

- `getConnectionManager()` - 获取存储所有已创建（使用`createConnection()`或`createConnections()`）连接的管理器。
```js
    import {getConnectionManager} from "typeorm";

    const defaultConnection = getConnectionManager().get("default");
    const secondaryConnection = getConnectionManager().get("secondary");
```
- `getConnection()` - 获取使用`createConnection`方法创建的连接。
```js
    import { getConnection } from "typeorm";

    // 可以在调用createConnection后使用并解析
    const connection = getConnection();

    // 如果你有多个连接，则可以按名称获取连接
    const secondConnection = getConnection("test2-connection");
```
- `getEntityManager()` - 获取`EntityManager`。 可以指定连接名称以指示应该采用哪个连接的实体管理器。
```js
    import {getEntityManager} from "typeorm";

    const manager = getEntityManager();
    // you can use manager methods now

    const secondaryManager = getEntityManager("secondary-connection");
    // you can use secondary connection manager methods
```
- `getRepository()` - 可以指定连接名称以指示应该采用哪个连接的实体管理器。

```js
import {getRepository} from "typeorm";

const userRepository = getRepository(User);
// you can use repository methods now

const blogRepository = getRepository(Blog, "secondary-connection");
// you can use secondary connection repository methods
```

- `getTreeRepository()` -  可以指定连接名称以指示应该采用哪个连接的实体管理器。

```js
import {getTreeRepository} from "typeorm";

const userRepository = getTreeRepository(User);
// 使用存储库方法

const blogRepository = getTreeRepository(Blog, "secondary-connection");
// 使用另一个存储库方法
```

- `getMongoRepository()` - 获取给定实体的`MongoRepository`。 可以指定连接名称以指示应该采用哪个连接的实体管理器。

```js
import {getMongoRepository} from "typeorm";

const userRepository = getMongoRepository(User);
//使用存储库方法

const blogRepository = getMongoRepository(Blog, "secondary-connection");
// 使用另一个存储库方法
```

### 操作数据库
一般来说，不要太多使用Connection。大多数情况下，你只需创建连接并使用`getRepository()`和`getManager()`来访问连接的管理器和存储库，而无需直接使用连接对象。
```js
import { getManager, getRepository } from "typeorm";
import { User } from "../entity/User";

export class UserController {
  @Get("/users")
  getAll() {
    return getManager().find(User);
  }

  @Get("/users/:id")
  getAll(@Param("id") userId: number) {
    return getRepository(User).findOne(userId);
  }
}
```

## 主从复制
所有模式更新和写入操作都使用`master`服务器执行。 `find`方法或`select` `query` `builder`执行的所有简单查询都使用随机的slave实例。
### 配置主从复制
```json
{
  type: "mysql",
  logging: true,
  replication: {
    master: {
      host: "server1",
      port: 3306,
      username: "test",
      password: "test",
      database: "test"
    },
    slaves: [{
      host: "server2",
      port: 3306,
      username: "test",
      password: "test",
      database: "test"
    }, {
      host: "server3",
      port: 3306,
      username: "test",
      password: "test",
      database: "test"
    }]
  }
}
```
如果在此时我们想读取主表的数据时我们可以使用 `getConnection().createQueryRunner("master")`在使用此方法的时候需要注意的时要将此链接释放`release()`
### 代码示例
```js
const masterQueryRunner = connection.createQueryRunner("master");
try {
    const postsFromMaster = await connection.createQueryBuilder(Post, "post")
        .setQueryRunner(masterQueryRunner)
        .getMany();
} finally {
      await masterQueryRunner.release();
}
```

### 深度配置
`mysql`，`postgres`和`sql server`数据库都支持复制。

Mysql支持深度配置：
```json
{
  replication: {
    master: {
      host: "server1",
      port: 3306,
      username: "test",
      password: "test",
      database: "test"
    },
    slaves: [{
      host: "server2",
      port: 3306,
      username: "test",
      password: "test",
      database: "test"
    }, {
      host: "server3",
      port: 3306,
      username: "test",
      password: "test",
      database: "test"
    }],

    /**
    * 如果为true，则PoolCluster将在连接失败时尝试重新连接。 （默认值：true）
    */
    canRetry: true,

    /**
     * 如果连接失败，则节点的errorCount会增加。
     * 当errorCount大于removeNodeErrorCount时，删除PoolCluster中的节点。 （默认值：5）
     */
    removeNodeErrorCount: 5,

    /**
     * 如果连接失败，则指定在进行另一次连接尝试之前的毫秒数。
     * 如果设置为0，则将删除节点，并且永远不会重复使用。 （默认值：0）
     */
     restoreNodeTimeout: 0,

    /**
     * 确定如何选择从库：
     * RR：交替选择一个（Round-Robin）。
     * RANDOM: 通过随机函数选择节点。
     * ORDER: 无条件选择第一个
     */
    selector: "RR"
  }
}
```

