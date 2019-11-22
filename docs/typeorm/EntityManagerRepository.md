## 什么是 EntityManager
使用EntityManager，你可以管理（insert, update, delete, load 等）任何实体。 EntityManager 就像放一个实体存储库的集合的地方。
你可以通过getManager（）或Connection访问实体管理器。
```js
import { getManager } from "typeorm";
import { User } from "./entity/User";

const entityManager = getManager(); // 你也可以通过 getConnection().manager 获取
const user = await entityManager.findOne(User, 1);
user.name = "Umed";
await entityManager.save(user);
```

## 什么是 Repository
Repository就像EntityManager一样，但其操作仅限于具体实体。
你可以通过getRepository（Entity），Connection＃getRepository或EntityManager＃getRepository访问存储库。
```js
import { getRepository } from "typeorm";
import { User } from "./entity/User";

const userRepository = getRepository(User); // 你也可以通过getConnection().getRepository()或getManager().getRepository() 获取
const user = await userRepository.findOne(1);
user.name = "Umed";
await userRepository.save(user);
```

### 有三种类型的存储库

- Repository - 任何实体的常规存储库。
- TreeRepository - 用于树实体的Repository的扩展存储库（比如标有@Tree装饰器的实体）。有特殊的方法来处理树结构。
- MongoRepository - 具有特殊功能的存储库，仅用于 MongoDB。

## Find 选项

### 基础选项
所有存储库和管理器`find`方法都接受可用于查询所需数据的特殊选项，而无需使用`QueryBuilder`
- `select` - 表示必须选择对象的哪些属性

```js
    userRepository.find({ select: ["firstName", "lastName"] });
```

- `relations` - 关系需要加载主体。 也可以加载子关系（join 和 leftJoinAndSelect 的简写）

```js
userRepository.find({ relations: ["profile", "photos", "videos"] });
userRepository.find({ relations: ["profile", "photos", "videos", "videos.video_attributes"] });
```

- `join` - 需要为实体执行联接，扩展版对的"relations"。

```js
userRepository.find({
    join: {
        alias: "user",
        leftJoinAndSelect: {
            profile: "user.profile",
            photo: "user.photos",
            video: "user.videos"
        }
    }
});
```

- `where` -查询实体的简单条件。

```js
userRepository.find({ where: { firstName: "Timber", lastName: "Saw" } });
```
查询嵌入实体列应该根据定义它的层次结构来完成。 例：

```js
userRepository.find({ where: { name: { first: "Timber", last: "Saw" } } });
```

使用 OR 运算符查询
```js
userRepository.find({
    where: [{ firstName: "Timber", lastName: "Saw" }, { firstName: "Stan", lastName: "Lee" }]
});
```

- order - 选择排序
```js
userRepository.find({
    order: {
        name: "ASC",
        id: "DESC"
    }
});
```

返回多个实体的`find`方法（`find`，`findAndCount`，`findByIds`），
### find选项

- skip - 偏移（分页）

```js
userRepository.find({
    skip: 5
});
```

- take - limit (分页) - 得到的最大实体数。
```js
userRepository.find({
    take: 10
});
```
> 如果你正在使用带有 MSSQL 的 typeorm，并且想要使用take或limit，你必须正确使用 order，否则将会收到以下错误：'FETCH语句中NEXT选项的使用无效。'
```js
userRepository.find({
    order: {
        columnName: "ASC"
    },
    skip: 0,
    take: 10
});
```

- cache - 启用或禁用查询结果缓存 

```js
userRepository.find({
    cache: true
});
```

- lock - 启用锁查询。 只能在findOne方法中使用。 lock是一个对象

> { mode: "optimistic", version: number|Date } 或 { mode: "pessimistic_read"|"pessimistic_write"|"dirty_read" }

```js
userRepository.findOne(1, {
    lock: { mode: "optimistic", version: 1 }
})
```

find 选项的完整示例
```js
userRepository.find({
    select: ["firstName", "lastName"],
    relations: ["profile", "photos", "videos"],
    where: {
        firstName: "Timber",
        lastName: "Saw"
    },
    order: {
        name: "ASC",
        id: "DESC"
    },
    skip: 5,
    take: 10,
    cache: true
});
```

### TypeORM 提供了许多内置运算符
|内置函数|对照数据库|例子|
|:-|:-:-|:-|
|Not|!=|find({name:Not('abc')})| 
|LessThan|<|find({name:LessThan('abc')})|
|LessThanOrEqual|<=|find({name:LessThanOrEqual('abc')})|
|MoreThan|>|find({name:MoreThan('abc')})|
|MoreThanOrEqual|>=|find({name:MoreThanOrEqual('abc')})|
|Equal|=|find({name:Equal('abc')})|
|Like|like|find({name:Like('%abc%')})|
|Between|BETWEEN 1 AND 10|find({name:Between(1,10)})|
|In|In|find({name:In(['abc','bcd']})|
|Any|In|find({name:Any(['abc','bcd']})|
|IsNull|IS NULL|find({name:IsNull()})|


