## 数据库关系

### 什么是关系？
关系可以帮助你轻松地与相关实体合作。 有几种类型的关系：
* [一对一](#一对一) 使用 @OneToOne
* [多对一](#多对一-一对多) 使用 @ManyToOne
* [一对多](#多对一-一对多) 使用 @OneToMany
* [多对多](#多对多) 使用 @ManyToMany

### 关系选项
|类型|描述|
|:-|:-:-|
|eager| boolean - 如果设置为 true，则在此实体上使用find * 或QueryBuilder时，将始终使用主实体加载关系|
|cascade| boolean - 如果设置为 true，则将插入相关对象并在数据库中更新。|
|onDelete| "RESTRICT"|"CASCADE"|"SET NULL" - 指定删除引用对象时外键的行为方式|
|primary| boolean - 指示此关系的列是否为主列。|
|nullable| boolean -指示此关系的列是否可为空。 默认情况下是可空。|

#### 一对一
> @OneToOne

一对一是一种 A 只包含一个 B 实例，而 B 只包含一个 A 实例的关系。我们以User和Profile实体为例。用户只能拥有一个配置文件，并且一个配置文件仅由一个用户拥有。

Profile.js
```js
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  gender: string;

  @Column()
  photo: string;

  @OneToOne(type => User, user => user.profile) // 将另一面指定为第二个参数
  user: User;
}
```
User.js
```js
import { Entity, PrimaryGeneratedColumn, Column, OneToOne, JoinColumn } from "typeorm";
import { Profile } from "./Profile";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToOne(type => Profile,profile => profile.user)
  @JoinColumn()
  profile: Profile;
}
```
##### 调用 
```js
const profile = new Profile();
profile.gender = "male";
profile.photo = "me.jpg";
await connection.manager.save(profile);

const user = new User();
user.name = "Joe Smith";
user.profile = profile;
await connection.manager.save(user);
```
要加载带有配置文件的用户，必须在Find Options 中指定关系 `{ relations: ["profile"] }`
```js
const userRepository = connection.getRepository(User);
const users = await userRepository.find({ relations: ["profile"] });
```
或者使用QueryBuilder:
```js
    const users = await connection
    .getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.profile", "profile")
    .getMany();
```

#### 多对一 / 一对多
> @ManyToOne / @OneToMany

多对一/一对多是指 A 包含多个 B 实例的关系，但 B 只包含一个 A 实例。 让我们以User 和 Photo 实体为例。 User 可以拥有多张 photos，但每张 photo 仅由一位 user 拥有。

```js
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from "typeorm";
import { User } from "./User";

@Entity()
export class Photo {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  url: string;

  @ManyToOne(type => User, user => user.photos)
  user: User;
}
```

```js
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from "typeorm";
import { Photo } from "./Photo";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(type => Photo, photo => photo.user)
  photos: Photo[];
}
```
- 这里我们将`@ManyToOne`添加到photos属性中，并将目标关系类型指定为Photo。 你也可以在`@ManyToOne` /`@OneToMany`关系中省略`@JoinColumn`。 
- 没有`@ManyToOne`，`@OneToMany`就不可能存在。 
- 如果你想使用`@OneToMany`，则需要`@ManyToOne`。 
- 在你设置`@ManyToOne`的地方，相关实体将有"关联 id"和外键。

##### 调用
```js
const photo1 = new Photo();
photo1.url = "me.jpg";
await connection.manager.save(photo1);

const photo2 = new Photo();
photo2.url = "me-and-bears.jpg";
await connection.manager.save(photo2);

const user = new User();
user.name = "John";
user.photos = [photo1, photo2];
await connection.manager.save(user);
```


#### 多对多
> @ManyToMany

多对多是一种 A 包含多个 B 实例，而 B 包含多个 A 实例的关系。 我们以Question 和 Category 实体为例。 Question 可以有多个 categories, 每个 category 可以有多个 questions。

* @JoinTable()是@ManyToMany关系所必需的。 你必须把@JoinTable放在关系的一个（拥有）方面

```js
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;
}
```

```js
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from "typeorm";
import { Category } from "./Category";

@Entity()
export class Question {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  text: string;

  @ManyToMany(type => Category)
  @JoinTable()
  categories: Category[];
}
```

##### 调用保存
```js
const category1 = new Category();
category1.name = "animals";
await connection.manager.save(category1);

const category2 = new Category();
category2.name = "zoo";
await connection.manager.save(category2);

const question = new Question();
question.categories = [category1, category2];
await connection.manager.save(question);
```



### 级联

> cascade: true

请记住，能力越大，责任越大。 级联可能看起来像是一种处理关系的好方法， 但是当一些不需要的对象被保存到数据库中时，它们也可能带来错误和安全问题。 此外，它们提供了一种将新对象保存到数据库中的不太明确的方法。

#### 示例
```js
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany } from "typeorm";
import { Question } from "./Question";

@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(type => Question, question => question.categories)
  questions: Question[];
}
```

```js
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from "typeorm";
import { Category } from "./Category";

@Entity()
export class Question {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  text: string;

  @ManyToMany(type => Category, category => category.questions, {
    cascade: true
  })
  @JoinTable()
  categories: Category[];
}
```
#### 调用
```js
    const category1 = new Category();
    category1.name = "animals";

    const category2 = new Category();
    category2.name = "zoo";

    const question = new Question();
    question.categories = [category1, category2];
    await connection.manager.save(question);
```



### @JoinColumn选项
`@JoinColumn`不仅定义了关系的哪一侧包含带有外键的连接列，还允许自定义连接列名和引用的列名。设置@JoinColumn的哪一方，哪一方的表将包含一个"relation id"和目标实体表的外键。
`@JoinColumn`必须仅设置在关系的一侧且必须在数据库表中具有外键的一侧。
当我们设置`@JoinColumn`时，它会自动在数据库中创建一个名为`propertyName + referencedColumnName`的列

```js
@ManyToOne(type => Category)
@JoinColumn() // 这个装饰器对于@ManyToOne是可选的，但@OneToOne是必需的
category: Category;
```
此代码将在数据库中创建categoryId列。 
#### 修改外键指定规则
@JoinColumn 默认指定的是标的主键 如果修改你默认指定的话可以是用`name`属性

> @JoinColumn({ name: "cat_id" })

#### 修改外键名称
> @JoinColumn({ referencedColumnName: "name" })


### @JoinTable选项
`@JoinTable`用于“多对多”关系，并描述联结表的连接列。 联结表是由 TypeORM 自动创建的一个特殊的单独表，其中的列引用相关实体。 你可以使用@JoinColumn更改联结表及其引用列中的列名
```js
@ManyToMany(type => Category)
@JoinTable({
    name: "question_categories" // 此关系的联结表的表名
    joinColumn: {
        name: "question",
        referencedColumnName: "id"
    },
    inverseJoinColumn: {
        name: "category",
        referencedColumnName: "id"
    }
})
categories: Category[];
```
