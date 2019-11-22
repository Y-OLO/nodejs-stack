## 什么是TypeOrm
TypeORM 是一个 ORM 框架，它可以运行在 NodeJS、Browser、Cordova、PhoneGap、Ionic、React Native、Expo 和 Electron 平台上，可以与 TypeScript 和 JavaScript (ES5,ES6,ES7,ES8)一起使用。 它的目标是始终支持最新的 JavaScript 特性并提供额外的特性以帮助你开发任何使用数据库的（不管是只有几张表的小型应用还是拥有多数据库的大型企业应用）应用程序。

不同于现有的所有其他 JavaScript ORM 框架，TypeORM 支持 Active Record 和 Data Mapper 模式，这意味着你可以以最高效的方式编写高质量的、松耦合的、可扩展的、可维护的应用程序。

TypeORM 参考了很多其他优秀 ORM 的实现, 比如 Hibernate, Doctrine 和 Entity Framework。

TypeORM 的一些特性:

- 支持 DataMapper 和 ActiveRecord (随你选择)
- 实体和列
- 数据库特性列类型
- 实体管理
- 存储库和自定义存储库
- 清晰的对象关系模型
- 关联（关系）
- 贪婪和延迟关系
- 单向的，双向的和自引用的关系
- 支持多重继承模式
- 级联
- 索引
- 事务
- 迁移和自动迁移
- 连接池
- 主从复制
- 使用多个数据库连接
- 使用多个数据库类型
- 跨数据库和跨模式查询
- 优雅的语法，灵活而强大的 QueryBuilder
- 左联接和内联接
- 使用联查查询的适当分页
- 查询缓存
- 原始结果流
- 日志
- 监听者和订阅者（钩子）
- 支持闭包表模式
- 在模型或者分离的配置文件中声明模式
- json / xml / yml / env 格式的连接配置
- 支持 MySQL / MariaDB / Postgres / SQLite / - Microsoft SQL Server / Oracle / sql.js
- 支持 MongoDB NoSQL 数据库
- 可在 NodeJS / 浏览器 / Ionic / Cordova / React - Native / Expo / Electron 平台上使用
- 支持 TypeScript 和 JavaScript
- 生成高性能、灵活、清晰和可维护的代码
- 遵循所有可能的最佳实践
- 命令行工具
还有更多...

## 准备工作
1. 通过 npm 安装:

> npm install typeorm --save

2. 你还需要安装 reflect-metadata:

> npm install reflect-metadata --save

3. 并且需要在应用程序的全局位置导入（例如在app.ts中）

> import "reflect-metadata";

4. 你可能还需要安装 node typings(以此来使用 Node 的智能提示):

> npm install @types/node --save

4. 安装数据库驱动:

    4.1 MySQL 或者 MariaDB

    > npm install mysql --save (也可以安装 mysql2)

    4.2 PostgreSQL

    > npm install pg --save

    4.3 SQLite

    > npm install sqlite3 --save

    4.4 Microsoft SQL Server

    > npm install mssql --save

    4.5 sql.js

    > npm install sql.js --save

    4.6 Oracle

    > npm install oracledb --save

   根据你使用的数据库，仅安装其中一个即可。 要使 Oracle 驱动程序正常工作，需要参照[官方配置](https://github.com/oracle/node-oracledb)中的安装说明进行操作。

    4.7 MongoDB (试验性)

    > npm install mongodb --save

## TypeScript 配置
此外，请确保你使用的 TypeScript 编译器版本是2.3或更高版本，并且已经在 `tsconfig.json` 中启用了以下设置:
```json
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
```
完整示例：
```json
{
   "compilerOptions": {
      "lib": [
         "es5",
         "es6"
      ],
      "target": "es5",
      "module": "commonjs",
      "moduleResolution": "node",
      "outDir": "./build",
      "emitDecoratorMetadata": true,
      "experimentalDecorators": true,
      "sourceMap": true
   }
}
```

## 创建第一个应用
TypeORM 的方法是使用其 CLI 命令生成启动项目。 但是只有在 NodeJS 应用程序中使用 TypeORM 时，此操作才有效。

1. 首先全局安装 TypeORM:
> npm install typeorm -g

2. 然后转到要创建新项目的目录并运行命令：
> typeorm init --name MyProject --database mysql

其中 name 是项目的名称，database 是将使用的数据库。
数据库可以是以下值之一: mysql、 mariadb、 postgres、 sqlite、 mssql、 oracle、 mongodb、 cordova、 react-native、 expo、 nativescript.

3. 此命令将在 MyProject 目录中生成一个包含以下文件的新项目:
```
    MyProject
    ├── src              // TypeScript 代码
    │   ├── entity       // 存储实体（数据库模型）的位置
    │   │   └── User.ts  // 示例 entity
    │   ├── migration    // 存储迁移的目录
    │   └── index.ts     // 程序执行主文件
    ├── .gitignore       // gitignore文件
    ├── ormconfig.json   // ORM和数据库连接配置
    ├── package.json     // node module 依赖
    ├── README.md        // 简单的 readme 文件
    └── tsconfig.json    // TypeScript 编译选项
```
4. 配置数据源
   
   4.1 参数说明
   [配置说明](https://typeorm.io/#/connection-options)

   4.2 配置`ormconfig.json`文件
```json
{
   "type": "mysql", // 数据库类型。你必须指定要使用的数据库引擎。该值可以是"mysql"，"postgres"，"mariadb"，"sqlite"，"cordova"，"nativescript"，"oracle"，"mssql"，"mongodb"，"sqljs"，"react-native"。此选项是必需的。
   "host": "192.168.168.62",
   "port": 3308,
   "username": "root",
   "password": "asa",
   "database": "test_typeorm",
   "synchronize": true,
   "logging": true,
   "entities": [
      "src/entity/**/*.ts"
   ],
   "migrations": [
      "src/migration/**/*.ts"
   ],
   "subscribers": [
      "src/subscriber/**/*.ts"
   ],
   "cli": {
      "entitiesDir": "src/entity",
      "migrationsDir": "src/migration",
      "subscribersDir": "src/subscriber"
   }
}
```
绝大多数情况下，你只需要配置 host, username, password, database 或者 port 即可。
   

5. 升级为一个更高级的框架
> typeorm init --name MyProject --database mysql --express 来生成一个更高级的 Express 项目
