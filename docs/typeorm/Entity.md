## 什么是实体
实体是一个映射到数据库表（或使用 MongoDB 时的集合）的类。 你可以通过定义一个新类来创建一个实体，并用@Entity()来标记。

``` js
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity() // 设置别名 @Entity("my_users")
export class User {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    isActive: boolean;
}
```

基本实体由列和关系组成。 每个实体必须有一个主列。每个实体都必须在连接选项中注册。在数据库连接中为我们提供了`entities`属性
```js
import { createConnection, Connection } from "typeorm";
import { User } from "./entity/User";
 const connection: Connection = await createConnection({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test",
    entities: [User], // ["entity/*.js"] 直接扫描entity下的全部文件
    entityPrefix:"my" // 如果需要设置统一前缀的话，直接设置此属性
});
```

在[Decorators reference](https://typeorm.io/#/decorator-reference/)中了解有关参数@Entity 的更多信息。

### 实体列
由于数据库表由列组成，因此实体也必须由列组成。 标有`@Column`的每个实体类属性都将映射到数据库表列。

### 主列
每个实体必须至少有一个主列。 有几种类型的主要列：
- `@PrimaryColumn()` 创建一个主列，它可以获取任何类型的任何值。你也可以指定列类型。 如果未指定列类型，则将从属性类型自动推断。

```js
import { Entity, PrimaryColumn } from "typeorm";

@Entity()
export class User {
    @PrimaryColumn()
    id: number;
}
```

存在多个`@PrimaryColumn()` 为复合主列

- `@PrimaryGeneratedColumn()` 创建一个主列，该值将使用自动增量值自动生成。 它将使用auto-increment /serial /sequence创建int列（取决于数据库）。 你不必在保存之前手动分配其值，该值将会自动生成。

```js
import { Entity, PrimaryGeneratedColumn } from "typeorm";

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number;
}
```

- `@PrimaryGeneratedColumn("uuid")` 创建一个主列，该值将使用uuid自动生成。 Uuid 是一个独特的字符串 id。 你不必在保存之前手动分配其值，该值将自动生成。

```js
import { Entity, PrimaryGeneratedColumn } from "typeorm";
@Entity()
export class User {
    @PrimaryGeneratedColumn("uuid")
    id: string;
}
```
当您使用save保存实体时，它总是先尝试使用给定的实体 ID（或 ids）在数据库中查找实体。 如果找到 id / ids，则将更新数据库中的这一行。 如果没有包含 id / ids 的行，则会插入一个新行。

### 特殊列
有几种特殊的列类型可以使用

- `@CreateDateColumn`是一个特殊列，自动为实体插入日期。无需设置此列，该值将自动设置。

- `@UpdateDateColumn` 是一个特殊列，在每次调用实体管理器或存储库的save时，自动更新实体日期。无需设置此列，该值将自动设置。

- `@VersionColumn` 是一个特殊列，在每次调用实体管理器或存储库的save时自动增长实体版本（增量编号）。无需设置此列，该值将自动设置。


### 列类型
TypeORM 支持所有最常用的数据库支持的列类型。 列类型是特定于数据库类型的. 这为数据库架构提供了更大的灵活性。 你可以将列类型指定为@Column的第一个参数 或者在@Column的列选项中指定。

> @Column("int") 或 @Column({ type: "int" })

|名称|类型|描述|
|:-|:-:|:-|
|type| ColumnType | 列类型。其中之一在上面.|
|name| string | 数据库表中的列名。默认情况下，列名称是从属性的名称生成的。 你也可以通过指定自己的名称来更改它。|
|length| number | 列类型的长度。 例如，如果要创建varchar（150）类型，请指定列类型和长度选项。|
|width| number | 列类型的显示范围。 仅用于MySQL integer types|
|onUpdate| string | ON UPDATE触发器。 仅用于 MySQL.|
|nullable| boolean | 在数据库中使列NULL或NOT NULL。 默认情况下，列是nullable：false。|
|update| boolean | 指示"save"操作是否更新列值。如果为false，则只能在第一次插入对象时编写该值。 默认值为"true"。|
|select| boolean | 定义在进行查询时是否默认隐藏此列。 设置为false时，列数据不会显示标准查询。 默认情况下，列是select：true|
|default| string | 添加数据库级列的DEFAULT值。|
|primary| boolean | 将列标记为主要列。 使用方式和@ PrimaryColumn相同。|
|unique| boolean | 将列标记为唯一列（创建唯一约束）。|
|comment| string | 数据库列备注，并非所有数据库类型都支持。|
|precision| number | 十进制（精确数字）列的精度（仅适用于十进制列），这是为值存储的最大位数。仅用于某些列类型。|
|scale| number | 十进制（精确数字）列的比例（仅适用于十进制列），表示小数点右侧的位数，且不得大于精度。 仅用于某些列类型。|
|zerofill| boolean | 将ZEROFILL属性设置为数字列。 仅在 MySQL 中使用。 如果是true，MySQL 会自动将UNSIGNED属性添加到此列。|
|unsigned| boolean | 将UNSIGNED属性设置为数字列。 仅在 MySQL 中使用。|
|charset| string | 定义列字符集。 并非所有数据库类型都支持。|
|collation| string | 定义列排序规则。|
|enum| string[]/AnyEnum | 在enum列类型中使用，以指定允许的枚举值列表。 你也可以指定数组或指定枚举类。|
|asExpression| string | 生成的列表达式。 仅在MySQL中使用。|
|generatedType| "VIRTUAL"/"STORED" | 生成的列类型。 仅在MySQL中使用。|
|hstoreType| "object"/"string" |返回HSTORE列类型。 以字符串或对象的形式返回值。 仅在Postgres中使用。|
|array| boolean | 用于可以是数组的 postgres 列类型（例如 int []）|
|transformer| { from(value: DatabaseType): EntityType, to(value: EntityType): DatabaseType } | 用于将任意类型EntityType的属性编组为数据库支持的类型DatabaseType。|

如果要指定其他类型参数，可以通过列选项来执行。 
> @Column("varchar", { length: 200 }) 或 @Column({ type: "int", length: 200 })

### mysql 数据类型
`int`, `tinyint`, `smallint`, `mediumint`, `bigint`, `float`, `double`, `dec`, `decimal`, `numeric`, `date`, `datetime`, `timestamp`, `time`, `year`, `char`, `varchar`, `nvarchar`, `text`, `tinytext`, `mediumtext`, `blob`, `longtext`, `tinyblob`, `mediumblob`, `longblob`, `enum`, `json`, `binary`, `geometry`, `point`, `linestring`, `polygon`, `multipoint`, `multilinestring`, `multipolygon`, `geometrycollection`

#### enum 列类型
使用typescript枚举
```js
export enum UserRole {
    ADMIN = "admin",
    EDITOR = "editor",
    GHOST = "ghost"
}

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column({
        type: "enum",
        enum: UserRole,
        default: UserRole.GHOST
    })
    role: UserRole

}
```
注意：支持字符串，数字和异构枚举。

```js
export type UserRoleType = "admin" | "editor" | "ghost",

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column({
        type: "enum",
        enum: ["admin", "editor", "ghost"],
        default: "ghost"
    })
    role: UserRoleType
}
```
### 具有生成值的列
使用 `@Generated` 标识符
```js
@Entity()
export class User {
    @PrimaryColumn()
    id: number;

    @Column()
    @Generated("uuid")
    uuid: string;
}
```
除了`uuid`之外，还有`increment`生成类型，但是对于这种类型的生成，某些数据库平台存在一些限制（例如，某些数据库只能有一个增量列，或者其中一些需要增量才能成为主键）。

## 嵌入式实体
通过使用`embedded` `columns`，可以减少应用程序中的重复（使用组合而不是继承）。
可以减少实体类中的代码重复。 你可以根据需要在嵌入式类中使用尽可能多的列（或关系）。 甚至可以在嵌入式类中嵌套嵌套列。

>  @Column(type => Name)

### 公共参数部分
```js
import {Entity, Column} from "typeorm";

export class Name {

    @Column()
    first: string;

    @Column()
    last: string;

}
```
### 引入
会生成实体参数+组合的参数 
```js
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";
import {Name} from "./Name";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: string;

    @Column(type => Name)
    name: Name; // 也就是nameFirst nameLast 两个参数

    @Column()
    isActive: boolean;

}
```

## 树实体
TypeORM支持用于存储树结构的Adjacency列表和Closure表模式。

### 快速导航
*  [邻接清单](#物化路径又名路径枚举)
*  [嵌套集](#嵌套集)
*  [物化路径(又名路径枚举)](#物化路径又名路径枚举)
*  [闭合表](#闭合表)
*  [使用树实体](#使用树实体)

### 邻接清单
邻接列表是一个具有自引用的简单模型。 这种方法的好处是简单，缺点是由于连接限制，您无法一次性加载整个树结构。
```js
import {Entity, Column, PrimaryGeneratedColumn, ManyToOne, OneToMany} from "typeorm";

@Entity()
export class Category {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @Column()
    description: string;

    @ManyToOne(type => Category, category => category.children)
    parent: Category;

    @OneToMany(type => Category, category => category.parent)
    children: Category[];
}
```
### 嵌套集
嵌套集是在数据库中存储树结构的另一种模式。 它对读取非常有效，但对写入不利。 且不能在嵌套集
中有多个根。
```js 
import {Entity, Tree, Column, PrimaryGeneratedColumn, TreeChildren, TreeParent, TreeLevelColumn} from "typeorm";

@Entity()
@Tree("nested-set")
export class Category {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @TreeChildren()
    children: Category[];

    @TreeParent()
    parent: Category;
}
```
### 物化路径(又名路径枚举)
物化路径（也称为路径枚举）是在数据库中存储树结构的另一种模式。 它简单有效。
```js
import {Entity, Tree, Column, PrimaryGeneratedColumn, TreeChildren, TreeParent, TreeLevelColumn} from "typeorm";

@Entity()
@Tree("materialized-path")
export class Category {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @TreeChildren()
    children: Category[];

    @TreeParent()
    parent: Category;
}
```
### 闭合表
闭合表以特殊方式在单独的表中存储父和子之间的关系。 它在读取和写入方面都很有效。
```js
import {Entity, Tree, Column, PrimaryGeneratedColumn, TreeChildren, TreeParent, TreeLevelColumn} from "typeorm";

@Entity()
@Tree("closure-table")
export class Category {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @TreeChildren()
    children: Category[];

    @TreeParent()
    parent: Category;
}
```
### 使用树实体
要使绑定树实体彼此关系，将其父项设置为子实体并保存它们
```js
const manager = getManager();

const a1 = new Category("a1");
a1.name = "a1";
await manager.save(a1);

const a11 = new Category();
a11.name = "a11";
a11.parent = a1;
await manager.save(a11);

const a12 = new Category();
a12.name = "a12";
a12.parent = a1;
await manager.save(a12);

const a111 = new Category();
a111.name = "a111";
a111.parent = a11;
await manager.save(a111);

const a112 = new Category();
a112.name = "a112";
a112.parent = a11;
await manager.save(a112);
```

### 加载树时使用

> 使用`TreeRepository`

```js
const manager = getManager();
const trees = await manager.getTreeRepository(Category).findTrees();
```
返回结果
```json
[{
    "id": 1,
    "name": "a1",
    "children": [{
        "id": 2,
        "name": "a11",
        "children": [{
            "id": 4,
            "name": "a111"
        }, {
            "id": 5,
            "name": "a112"
        }]
    }, {
        "id": 3,
        "name": "a12"
    }]
}]
```

### API
- `findTrees` - 返回数据库中所有树，包括所有子项，子项的子项等。
```js
const treeCategories = await repository.findTrees();
// 返回包含子类别的根类别
```
- `findRoots` - 根节点是没有祖先的实体。 找到所有根节点但不加载子节点。
```js
const rootCategories = await repository.findRoots();
// 返回没有子类别的根类别
```
- `findDescendants` - 获取给定实体的所有子项（后代）。 将它们全部返回到数组中。
```js
const childrens = await repository.findDescendants(parentCategory);
// 返回parentCategory的所有直接子类别（没有其嵌套类别）
```
- `findDescendantsTree` - 获取给定实体的所有子项（后代）。
```js
const childrensTree = await repository.findDescendantsTree(parentCategory);
// 返回parentCategory的所有直接子类别（及其嵌套类别）
```
- `createDescendantsQueryBuilder` - 创建用于获取树中实体的后代的查询构建器。
```js
const childrens = await repository
    .createDescendantsQueryBuilder("category", "categoryClosure", parentCategory)
    .andWhere("category.type = 'secondary'")
    .getMany();
```
- `countDescendants` - 获取实体的后代数。
```js
const childrenCount = await repository.countDescendants(parentCategory);
```
- `findAncestors` - 获取给定实体的所有父（祖先）。 将它们全部返回到数组中。
```js
const parents = await repository.findAncestors(childCategory);
// 返回所有直接childCategory的父类别（和"parent 的 parents"）
```
- `findAncestorsTree` - 返回所有直接childCategory的父类别 ( "parent 的 parents")
```js
const parentsTree = await repository.findAncestorsTree(childCategory);
// 返回所有直接childCategory的父类别 (和 "parent 的 parents")
```
- `createAncestorsQueryBuilder` - 创建用于获取树中实体的祖先的查询构建器。
```js
const parents = await repository
    .createAncestorsQueryBuilder("category", "categoryClosure", childCategory)
    .andWhere("category.type = 'secondary'")
    .getMany();
```
- `countAncestors` - 获取实体的祖先数。
```js
const parentsCount = await repository.countAncestors(childCategory);
```


