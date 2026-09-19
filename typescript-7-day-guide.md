# TypeScript 完整入门指南 + 七天循序渐进学习路线

> 面向：已经接触 JavaScript / Node.js、希望系统掌握 TypeScript 的开发者
>
> 学习目标：从“会写类型注解”进阶到“能利用 TypeScript 设计可靠的类型、阅读泛型代码、配置工程、开发真实 Node.js / 前端项目”。
>
> 建议节奏：每天 2～4 小时，**理解概念 30% + 写代码 60% + 总结复盘 10%**。
>
> 当前版本说明：截至 2026 年 9 月，TypeScript 7.0.2 已发布。TypeScript 7 使用新的原生实现，官方称完整构建通常可获得约 8～12 倍的加速；但你学习时最重要的仍然是 TypeScript 的类型系统、模块系统和工程配置，而不是版本特有的小语法变化。

---

## 目录

- [一、先理解 TypeScript 到底是什么](#一先理解-typescript-到底是什么)
- [二、学习前的 JavaScript 基础](#二学习前的-javascript-基础)
- [三、环境搭建](#三环境搭建)
- [四、TypeScript 核心知识地图](#四typescript-核心知识地图)
- [五、基础语法与类型系统](#五基础语法与类型系统)
- [六、对象、接口与类型别名](#六对象接口与类型别名)
- [七、联合类型、交叉类型与类型缩小](#七联合类型交叉类型与类型缩小)
- [八、函数类型](#八函数类型)
- [九、泛型：TypeScript 的核心能力](#九泛型typescript-的核心能力)
- [十、any、unknown、never、void](#十anyunknownnevervoid)
- [十一、keyof / typeof / 索引访问类型](#十一keyof--typeof--索引访问类型)
- [十二、条件类型、映射类型与 infer](#十二条件类型映射类型与-infer)
- [十三、内置工具类型](#十三内置工具类型)
- [十四、数组、元组与 readonly](#十四数组元组与-readonly)
- [十五、class 与面向对象](#十五class-与面向对象)
- [十六、模块与 import / export](#十六模块与-import--export)
- [十七、tsconfig.json 工程配置](#十七tsconfigjson-工程配置)
- [十八、运行时与类型系统的边界](#十八运行时与类型系统的边界)
- [十九、Node.js 项目中的 TypeScript](#十九nodejs-项目中的-typescript)
- [二十、API / 数据库项目中的 TypeScript](#二十api--数据库项目中的-typescript)
- [二十一、七天学习计划](#二十一七天学习计划)
- [二十二、每天练习题](#二十二每天练习题)
- [二十三、常见坑](#二十三常见坑)
- [二十四、TypeScript 高频速查表](#二十四typescript-高频速查表)
- [二十五、七天之后如何继续进阶](#二十五七天之后如何继续进阶)

---

# 一、先理解 TypeScript 到底是什么

## 1.1 一句话理解

**TypeScript = JavaScript + 静态类型系统 + 编译器/工具链。**

更准确地说，TypeScript 是 JavaScript 的静态类型检查工具，同时提供类型语法、编辑器提示和编译工具。

官方 Handbook 对 TypeScript 的核心定位就是：在 JavaScript 程序运行之前进行静态类型检查。

因此要牢牢记住：

```text
TypeScript
   ↓
类型检查
   ↓
JavaScript
   ↓
Node.js / Browser / Bun / Deno 等运行环境
```

TypeScript 类型本身通常不会在 JavaScript 运行时存在。

例如：

```ts
const username: string = "Alice";
const age: number = 20;
```

编译后的 JavaScript 大致就是：

```js
const username = "Alice";
const age = 20;
```

`string` 和 `number` 主要服务于开发阶段的类型检查。

---

## 1.2 TypeScript 解决什么问题？

JavaScript 很灵活：

```js
function add(a, b) {
  return a + b;
}
```

问题是：调用者很容易传入错误类型。

```js
add(10, 20);       // 30
add("10", "20"); // "1020"
add(10, "20");    // "1020"
```

TypeScript 可以提前发现：

```ts
function add(a: number, b: number): number {
  return a + b;
}

add(10, 20);

add("10", "20");
// ❌ 类型错误
```

这意味着 TypeScript 最重要的价值并不是“让代码更长”，而是：

> **把大量运行时才会发现的问题提前到开发阶段。**

---

# 二、学习前的 JavaScript 基础

如果 JavaScript 基础比较薄弱，不建议直接背 TypeScript 类型语法。

至少应该掌握：

```text
变量
↓
函数
↓
对象
↓
数组
↓
解构
↓
展开运算符
↓
Promise / async / await
↓
模块 import / export
↓
Map / Set
↓
class
```

重点掌握以下代码：

```js
const user = {
  id: 1,
  name: "Alice",
  age: 20,
};

const { name, age } = user;

const users = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" },
];

const result = users
  .filter(user => user.age >= 18)
  .map(user => user.name);

async function fetchData() {
  const response = await fetch("/api/users");
  return response.json();
}
```

如果这些内容已经熟悉，就可以直接进入 TypeScript。

---

# 三、环境搭建

## 3.1 创建项目

```bash
mkdir ts-demo
cd ts-demo
npm init -y
```

安装 TypeScript：

```bash
npm install -D typescript
```

查看版本：

```bash
npx tsc --version
```

推荐学习时使用项目本地安装，而不是只依赖全局 TypeScript。

这样不同项目可以使用各自锁定的版本。

---

## 3.2 初始化 tsconfig

```bash
npx tsc --init
```

建议初学阶段从严格模式开始。

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noEmit": true,
    "skipLibCheck": true
  }
}
```

实际项目中的 `module` / `moduleResolution` 要根据 Node、Bundler、库发布方式决定。TypeScript 官方目前分别介绍 `NodeNext` 和 `bundler` 两套常见解析策略。

---

## 3.3 第一个 TypeScript 程序

创建：

```text
src/
  index.ts
```

写入：

```ts
function greet(name: string): string {
  return `Hello, ${name}`;
}

console.log(greet("TypeScript"));
```

检查类型：

```bash
npx tsc --noEmit
```

注意：

```text
tsc --noEmit
```

只是做类型检查，不输出 JavaScript。

实际构建是否由 `tsc` 输出，还是交给 Vite、esbuild、webpack、rollup、tsx 等工具，需要根据项目架构决定。

---

# 四、TypeScript 核心知识地图

把整个 TypeScript 看成 6 层，会非常容易理解。

```text
第一层：JavaScript
变量 / 函数 / 对象 / 数组 / Promise / 模块

第二层：基础类型
string / number / boolean / null / undefined
array / tuple / object

第三层：类型建模
interface / type
union / intersection
enum / literal

第四层：类型推导
inference
narrowing
keyof
typeof
indexed access

第五层：类型编程
generic
conditional type
mapped type
infer
template literal type
utility type

第六层：工程化
module
npm
.ts / .d.ts
API types
tsconfig
Node.js / Browser / Bundler
```

**不要一开始就研究条件类型和 infer。**

正确顺序是：

```text
基础类型
→ 对象
→ interface/type
→ union
→ narrowing
→ function
→ generic
→ keyof/typeof
→ utility types
→ conditional/mapped/infer
→ 工程化
```

---

# 五、基础语法与类型系统

## 5.1 string

```ts
let username: string = "Alice";
```

---

## 5.2 number

```ts
let age: number = 20;
let price: number = 99.9;
```

JavaScript 中只有一种 number 类型。

```ts
let count: number = 100;
```

---

## 5.3 boolean

```ts
let isAdmin: boolean = true;
```

---

## 5.4 null / undefined

严格模式下：

```ts
let value: string | undefined;
let result: number | null = null;
```

这也是为什么推荐：

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

`strictNullChecks` 开启后，`null` 和 `undefined` 会成为需要明确处理的独立类型，从而减少大量“以为一定存在、实际上不存在”的问题。

---

## 5.5 Array

两种常见写法：

```ts
const numbers: number[] = [1, 2, 3];

const names: Array<string> = ["Alice", "Bob"];
```

泛型形式：

```ts
Array<string>
Array<number>
Array<User>
```

初学阶段把它们看成等价即可。

---

## 5.6 Object

简单对象：

```ts
const user: {
  name: string;
  age: number;
} = {
  name: "Alice",
  age: 20,
};
```

实际项目里通常会进一步抽成 `interface` 或 `type`。

---

## 5.7 类型推导

TypeScript 并不是要求所有东西都手动标注。

```ts
const username = "Alice";
```

TypeScript 会自动推断：

```text
username: string
```

所以不要写成：

```ts
const username: string = "Alice";
```

除非显式类型能增加可读性或约束。

更推荐：

```ts
const username = "Alice";
const age = 20;
const numbers = [1, 2, 3];
```

让 TypeScript 自动推断。

---

## 5.8 literal type

```ts
let direction: "left" | "right";

direction = "left";

direction = "up";
// ❌ 错误
```

字面量类型非常重要，因为它是 TypeScript API 设计的重要基础。

例如：

```ts
type Method = "GET" | "POST" | "PUT" | "DELETE";
```

---

# 六、对象、接口与类型别名

## 6.1 interface

```ts
interface User {
  id: number;
  name: string;
  age: number;
}

const user: User = {
  id: 1,
  name: "Alice",
  age: 20,
};
```

---

## 6.2 可选属性

```ts
interface User {
  id: number;
  name: string;
  age?: number;
}
```

意味着：

```ts
const user1: User = {
  id: 1,
  name: "Alice",
};
```

也可以：

```ts
const user2: User = {
  id: 1,
  name: "Alice",
  age: 20,
};
```

---

## 6.3 readonly

```ts
interface User {
  readonly id: number;
  name: string;
}
```

```ts
user.id = 2;
// ❌ 不允许修改
```

注意：`readonly` 是 TypeScript 类型层面的约束，并不等价于 JavaScript 运行时的真正不可变对象。

---

## 6.4 type

```ts
type User = {
  id: number;
  name: string;
};
```

以及：

```ts
type ID = string | number;
```

这是 `type` 的强项之一。

---

## 6.5 interface 与 type 怎么选？

可以先采用一个非常实用的规则：

```text
描述对象结构
→ interface

组合类型 / 联合类型 / 条件类型
→ type
```

例如：

```ts
interface User {
  id: number;
  name: string;
}

type UserID = string | number;

type Result<T> = {
  data: T;
  success: boolean;
};
```

不要陷入“interface 和 type 谁绝对更好”的争论。

真实项目最重要的是：**统一团队规范、保持一致。**

---

## 6.6 extends

```ts
interface Admin extends User {
  permissions: string[];
}
```

也可以多继承：

```ts
interface Admin extends User, Serializable {
  permissions: string[];
}
```

---

# 七、联合类型、交叉类型与类型缩小

这是 TypeScript 最重要的章节之一。

## 7.1 Union 联合类型

```ts
type ID = string | number;

let id: ID;

id = 1;
id = "abc";
```

含义：

```text
ID 可以是 string
或者 number
```

---

## 7.2 为什么联合类型必须 narrowing？

```ts
function printId(id: string | number) {
  console.log(id.toUpperCase());
}
```

错误。

因为 `number` 没有 `toUpperCase()`。

所以需要缩小：

```ts
function printId(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase());
  } else {
    console.log(id.toFixed(2));
  }
}
```

这就是：

```text
Union
↓
检查运行时信息
↓
Narrowing
↓
获得更精确的类型
```

---

## 7.3 typeof narrowing

```ts
function format(value: string | number) {
  if (typeof value === "string") {
    return value.toUpperCase();
  }

  return value.toFixed(2);
}
```

---

## 7.4 Array.isArray

```ts
function process(value: string | string[]) {
  if (Array.isArray(value)) {
    return value.length;
  }

  return value.length;
}
```

---

## 7.5 in narrowing

```ts
interface User {
  name: string;
}

interface Admin {
  name: string;
  permissions: string[];
}

function printPerson(person: User | Admin) {
  if ("permissions" in person) {
    console.log(person.permissions);
  }
}
```

---

## 7.6 discriminated union

这是实际业务开发非常重要的模式。

```ts
type Result =
  | {
      status: "success";
      data: User[];
    }
  | {
      status: "error";
      message: string;
    };
```

然后：

```ts
function handleResult(result: Result) {
  if (result.status === "success") {
    result.data;
  } else {
    result.message;
  }
}
```

这种设计在：

```text
API 返回
状态机
Redux / Zustand 状态
表单状态
异步任务
WebSocket 消息
命令模式
```

中都非常有用。

---

## 7.7 Intersection 交叉类型

```ts
interface User {
  id: number;
  name: string;
}

interface Timestamps {
  createdAt: Date;
  updatedAt: Date;
}

type UserEntity = User & Timestamps;
```

结果必须同时满足：

```text
User
AND
Timestamps
```

---

# 八、函数类型

## 8.1 参数和返回值

```ts
function add(a: number, b: number): number {
  return a + b;
}
```

---

## 8.2 void

```ts
function log(message: string): void {
  console.log(message);
}
```

这里的 `void` 通常表示函数调用者不应该依赖返回值。

---

## 8.3 可选参数

```ts
function greet(name: string, age?: number) {
  if (age !== undefined) {
    return `${name} is ${age}`;
  }

  return name;
}
```

---

## 8.4 默认参数

```ts
function greet(name: string, language = "en") {
  return `${name}-${language}`;
}
```

---

## 8.5 函数类型

```ts
type MathFn = (a: number, b: number) => number;

const add: MathFn = (a, b) => a + b;
```

---

## 8.6 回调函数

```ts
function processUsers(
  users: User[],
  callback: (user: User) => void
) {
  for (const user of users) {
    callback(user);
  }
}
```

---

# 九、泛型：TypeScript 的核心能力

如果你只记住 TypeScript 的几个关键词，泛型一定应该在里面。

## 9.1 为什么需要泛型？

先看问题：

```ts
function identity(value: any) {
  return value;
}
```

虽然能运行，但类型信息丢失。

```ts
const result = identity(123);
```

使用泛型：

```ts
function identity<T>(value: T): T {
  return value;
}
```

现在：

```ts
const numberValue = identity(123);
// number

const stringValue = identity("hello");
// string
```

泛型解决的是：

> **同一套代码处理不同类型，同时保留类型关系。**

---

## 9.2 泛型数组

```ts
function first<T>(items: T[]): T | undefined {
  return items[0];
}
```

调用：

```ts
const a = first([1, 2, 3]);
// number | undefined

const b = first(["a", "b"]);
// string | undefined
```

---

## 9.3 泛型约束

```ts
function getLength<T extends { length: number }>(value: T) {
  return value.length;
}
```

可以：

```ts
getLength("hello");
getLength([1, 2, 3]);
```

不能：

```ts
getLength(123);
```

---

## 9.4 keyof + 泛型

非常重要：

```ts
function getProperty<T, K extends keyof T>(
  obj: T,
  key: K
): T[K] {
  return obj[key];
}
```

使用：

```ts
const user = {
  id: 1,
  name: "Alice",
};

const id = getProperty(user, "id");
// number

const name = getProperty(user, "name");
// string
```

这就是 TypeScript 类型系统真正强大的地方：

```text
对象类型
↓
提取 key
↓
限制 key
↓
根据 key 推导 value
```

---

## 9.5 泛型类

```ts
class Box<T> {
  constructor(public value: T) {}
}

const numberBox = new Box(123);
const stringBox = new Box("hello");
```

官方文档也将泛型类作为 TypeScript 中的重要类型建模方式。

---

# 十、any、unknown、never、void

这几个类型必须真正理解。

## 10.1 any

```ts
let value: any;
```

`any` 基本上相当于：

```text
“别检查我。”
```

例如：

```ts
let value: any = 123;

value.foo.bar.baz();
```

TypeScript 不一定会阻止你。

所以：

> `any` 是逃生舱，不应该作为默认设计方案。

---

## 10.2 unknown

```ts
let value: unknown;
```

`unknown` 表示：

> 我不知道它是什么，但我要求你在使用之前证明它是什么。

例如：

```ts
function print(value: unknown) {
  if (typeof value === "string") {
    console.log(value.toUpperCase());
  }
}
```

这通常比 `any` 更安全。

---

## 10.3 never

`never` 常见于“不可能发生”的场景。

```ts
function fail(message: string): never {
  throw new Error(message);
}
```

另一个经典用途是穷尽性检查：

```ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; size: number };

function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${value}`);
}

function area(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.size ** 2;
    default:
      return assertNever(shape);
  }
}
```

未来新增 `triangle` 时，TypeScript 可以帮助发现遗漏。

---

# 十一、keyof / typeof / 索引访问类型

这部分是从“会用 TS”迈向“看懂高级类型”的关键。

## 11.1 keyof

```ts
type User = {
  id: number;
  name: string;
  age: number;
};

type UserKeys = keyof User;
```

等价于：

```ts
type UserKeys = "id" | "name" | "age";
```

---

## 11.2 索引访问类型

```ts
type User = {
  id: number;
  name: string;
};

type UserName = User["name"];
// string
```

多个 key：

```ts
type UserField = User["id" | "name"];
// number | string
```

这是 TypeScript 官方文档中的重要类型操作。

---

## 11.3 typeof（类型层面）

```ts
const config = {
  host: "localhost",
  port: 3000,
};

type Config = typeof config;
```

这样可以直接从值提取类型。

---

## 11.4 as const

```ts
const config = {
  host: "localhost",
  port: 3000,
} as const;
```

此时属性会变成更精确的字面量类型，并带有 readonly 特征。

另一个常见例子：

```ts
const roles = ["admin", "user", "guest"] as const;

type Role = typeof roles[number];
```

得到：

```ts
type Role = "admin" | "user" | "guest";
```

这是非常值得熟练掌握的模式。

---

# 十二、条件类型、映射类型与 infer

这一章可以放到进阶阶段。

## 12.1 Conditional Type

```ts
type IsString<T> = T extends string ? true : false;
```

例如：

```ts
type A = IsString<string>;
// true

type B = IsString<number>;
// false
```

基本结构：

```ts
T extends X ? A : B
```

理解成：

```text
如果 T 满足 X
→ A
否则
→ B
```

---

## 12.2 infer

`infer` 用来在条件类型中“提取”类型。

例如：

```ts
type GetReturnType<T> =
  T extends (...args: never[]) => infer R
    ? R
    : never;
```

然后：

```ts
type A = GetReturnType<() => string>;
// string

type B = GetReturnType<() => number>;
// number
```

官方文档也使用类似模式解释如何从函数类型中推导返回值。

---

## 12.3 Mapped Type

例如：

```ts
type User = {
  id: number;
  name: string;
};
```

我们可以让所有字段变成 readonly：

```ts
type ReadonlyUser = {
  readonly [K in keyof User]: User[K];
};
```

这就是“遍历一个类型的所有 key，然后重新生成一个类型”。

---

## 12.4 Template Literal Types

```ts
type EventName = `on${string}`;
```

例如：

```ts
const event: EventName = "onClick";
```

大型类型系统、事件 API、路由类型等场景中会看到这类写法。

---

# 十三、内置工具类型

TypeScript 自带大量实用工具类型。

## Partial

```ts
interface User {
  id: number;
  name: string;
  age: number;
}

type UpdateUser = Partial<User>;
```

变成：

```ts
{
  id?: number;
  name?: string;
  age?: number;
}
```

---

## Required

```ts
type RequiredUser = Required<User>;
```

所有属性变成必填。

---

## Pick

```ts
type UserPreview = Pick<User, "id" | "name">;
```

---

## Omit

```ts
type UserWithoutId = Omit<User, "id">;
```

---

## Record

```ts
type UserMap = Record<string, User>;
```

非常适合：

```ts
const users: Record<string, User> = {};
```

---

## Readonly

```ts
type ReadonlyUser = Readonly<User>;
```

---

## ReturnType

```ts
function getUser() {
  return {
    id: 1,
    name: "Alice",
  };
}

type User = ReturnType<typeof getUser>;
```

---

## Parameters

```ts
function login(username: string, password: string) {}

type LoginParams = Parameters<typeof login>;
```

结果：

```ts
[string, string]
```

---

# 十四、数组、元组与 readonly

## 14.1 普通数组

```ts
const numbers: number[] = [1, 2, 3];
```

---

## 14.2 元组 Tuple

元组强调“位置 + 类型”。

```ts
type UserTuple = [number, string, boolean];

const user: UserTuple = [1, "Alice", true];
```

例如数据库查询中：

```ts
type Row = [number, string, number];
```

---

## 14.3 readonly array

```ts
const numbers: readonly number[] = [1, 2, 3];
```

不能：

```ts
numbers.push(4);
```

函数参数也常写：

```ts
function sum(values: readonly number[]) {
  return values.reduce((a, b) => a + b, 0);
}
```

这样函数承诺：不会修改传入数组。

---

# 十五、class 与面向对象

TypeScript 完整支持 JavaScript class，并增加访问修饰符和类型约束。

## 15.1 基础 class

```ts
class User {
  constructor(
    public id: number,
    public name: string
  ) {}

  greet() {
    return `Hello, ${this.name}`;
  }
}
```

---

## 15.2 private / protected / public

```ts
class BankAccount {
  private balance = 0;

  public deposit(amount: number) {
    this.balance += amount;
  }
}
```

---

## 15.3 abstract

```ts
abstract class Animal {
  abstract speak(): string;
}

class Dog extends Animal {
  speak() {
    return "woof";
  }
}
```

---

## 15.4 interface + class

```ts
interface Repository<T> {
  findById(id: number): Promise<T | null>;
  save(data: T): Promise<T>;
}
```

然后实现：

```ts
class UserRepository implements Repository<User> {
  async findById(id: number): Promise<User | null> {
    return null;
  }

  async save(data: User): Promise<User> {
    return data;
  }
}
```

这里已经开始进入真实工程中的 TypeScript。

---

# 十六、模块与 import / export

现代 TypeScript 项目核心就是 ES Module。

## 16.1 export

```ts
export interface User {
  id: number;
  name: string;
}

export function createUser(name: string): User {
  return {
    id: Date.now(),
    name,
  };
}
```

---

## 16.2 import

```ts
import { User, createUser } from "./user.js";
```

Node ESM 项目中需要特别理解“源代码扩展名、编译输出扩展名、运行时模块解析”的关系。

TypeScript 官方当前文档明确区分：

```text
module
moduleResolution
NodeNext
bundler
```

不要只背配置，要理解“最终是谁负责加载这个模块”。

---

## 16.3 type-only import

```ts
import type { User } from "./user.js";
```

用于明确表示这里只需要类型。

---

# 十七、tsconfig.json 工程配置

`tsconfig.json` 可以理解为：

> **告诉 TypeScript：如何理解、检查、编译这个项目。**

---

## 17.1 strict

强烈建议：

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

官方 TSConfig 文档说明，`strict` 会开启一组更严格的类型检查选项，包括 `strictNullChecks`、`noImplicitAny`、`strictFunctionTypes` 等。

---

## 17.2 noImplicitAny

避免：

```ts
function test(value) {
  return value;
}
```

变成显式类型：

```ts
function test(value: string) {
  return value;
}
```

---

## 17.3 strictNullChecks

避免：

```ts
const user = users.find(...);
console.log(user.name);
```

因为 `find()` 可能找不到。

严格模式下需要：

```ts
if (user) {
  console.log(user.name);
}
```

这正是 TypeScript 帮你把潜在运行时错误提前暴露出来的典型场景。

---

## 17.4 target

例如：

```json
{
  "compilerOptions": {
    "target": "ES2022"
  }
}
```

含义可以理解为：

> 最终 JavaScript 需要兼容到什么级别。

实际项目应根据 Node / 浏览器最低版本选择。

---

## 17.5 module

例如 Node 项目常见：

```json
{
  "compilerOptions": {
    "module": "NodeNext"
  }
}
```

Bundler 项目可能使用：

```json
{
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "bundler"
  }
}
```

不要机械复制配置。

---

## 17.6 rootDir / outDir

传统编译型项目可以：

```json
{
  "compilerOptions": {
    "rootDir": "./src",
    "outDir": "./dist"
  }
}
```

结构：

```text
src/
  index.ts
  user.ts

↓ tsc

dist/
  index.js
  user.js
```

---

## 17.7 noEmit

如果你的项目由 Vite / esbuild /其他 Bundler 负责构建，TypeScript 可以只做类型检查：

```json
{
  "compilerOptions": {
    "noEmit": true
  }
}
```

---

# 十八、运行时与类型系统的边界

这是很多 TypeScript 初学者最容易产生误解的地方。

## 18.1 TypeScript 不会保护运行时数据

假设：

```ts
interface User {
  id: number;
  name: string;
}
```

你从 HTTP API 获得 JSON：

```ts
const data = await response.json();
```

你即使写：

```ts
const user = data as User;
```

也不代表服务器真的返回了：

```json
{
  "id": 1,
  "name": "Alice"
}
```

`as User` 只是告诉 TypeScript：

> “相信我。”

它不会自动验证 JSON。

---

## 18.2 Type Assertion 不是数据校验

不要把：

```ts
const user = data as User;
```

理解成：

```text
JSON → 自动验证 → User
```

实际是：

```text
JSON
 ↓
你声明它应该是 User
 ↓
TypeScript 相信你
```

这就是类型系统和运行时世界的边界。

---

## 18.3 API 项目为什么需要 runtime validation

实际项目通常需要：

```text
HTTP Request
↓
Runtime Validation
↓
转换 / 解析
↓
业务代码
↓
数据库
```

常见运行时 schema 工具包括：

```text
Zod
Valibot
ArkType
JSON Schema
Ajv
```

TypeScript 负责“开发期类型”。

Schema Validator 负责“运行时数据”。

两者是互补关系。

---

# 十九、Node.js 项目中的 TypeScript

结合 Node.js 学习 TypeScript，是非常高效的一条路线。

推荐项目结构：

```text
my-api/
├─ src/
│  ├─ app.ts
│  ├─ server.ts
│  ├─ routes/
│  │  └─ user.ts
│  ├─ services/
│  │  └─ user-service.ts
│  ├─ repositories/
│  │  └─ user-repository.ts
│  ├─ models/
│  │  └─ user.ts
│  └─ types/
│     └─ common.ts
├─ test/
├─ package.json
├─ tsconfig.json
└─ README.md
```

---

## 19.1 数据模型

```ts
export interface User {
  id: number;
  name: string;
  email: string;
}
```

---

## 19.2 Service

```ts
export class UserService {
  async findUser(id: number): Promise<User | null> {
    return null;
  }
}
```

---

## 19.3 Repository

```ts
export interface UserRepository {
  findById(id: number): Promise<User | null>;
  create(user: CreateUserInput): Promise<User>;
}
```

---

## 19.4 DTO

API 中建议区别：

```text
Database Model
≠
Domain Model
≠
API Request DTO
≠
API Response DTO
```

例如：

```ts
interface CreateUserRequest {
  name: string;
  email: string;
}
```

数据库对象可能是：

```ts
interface UserEntity {
  id: number;
  name: string;
  email: string;
  passwordHash: string;
  createdAt: Date;
}
```

不要为了方便把数据库完整对象直接当 API Response。

---

# 二十、API / 数据库项目中的 TypeScript

这部分非常适合拿来练习你正在学习的 Node.js / Fastify / SQL。

推荐理解这一条数据流：

```text
HTTP 请求
    ↓
Schema / DTO
    ↓
Runtime Validation
    ↓
Controller / Route
    ↓
Service
    ↓
Repository
    ↓
SQL / ORM
    ↓
Database
    ↓
Repository
    ↓
Service
    ↓
Response DTO
    ↓
JSON
```

TypeScript 可以贯穿其中：

```text
Request Type
↓
Domain Type
↓
DB Type
↓
Response Type
```

这就是 TypeScript 在真实后端工程中的价值。

---

# 二十一、七天学习计划

## 总安排

| 天数 | 核心目标 | 重点知识 | 实战 |
|---|---|---|---|
| Day 1 | 建立 TypeScript 基础概念 | 基础类型、推导、数组、对象 | 类型化基础 JS |
| Day 2 | 学会建模 | interface、type、union、intersection | 用户 / 商品模型 |
| Day 3 | 掌握类型推导 | narrowing、函数、泛型 | 通用工具函数 |
| Day 4 | 理解高级类型 | keyof、typeof、indexed access、utility | API DTO |
| Day 5 | 进入工程化 | module、tsconfig、Node | TS Node 项目 |
| Day 6 | 真实业务开发 | API、Schema、Service、Repository | 用户 CRUD API |
| Day 7 | 综合项目 | 类型设计 + 工程化 + 重构 | 完整小项目 |

---

# Day 1：基础类型 + 类型推导

## 学习目标

理解：

```text
TypeScript 是什么
↓
类型是什么
↓
类型推导是什么
↓
为什么 strict 很重要
```

## 上午

学习：

```ts
string
number
boolean
null
undefined
array
tuple
object
```

练习：

```ts
const username: string = "Alice";
const age: number = 20;
const isAdmin: boolean = false;
const scores: number[] = [80, 90, 95];
```

## 下午

学习类型推导：

```ts
const name = "Alice";
const age = 20;
const list = [1, 2, 3];
```

然后故意制造错误：

```ts
let age = 20;
age = "20";
```

观察 TypeScript 报错。

## 晚上练习

设计：

```ts
interface Product {
  id: number;
  name: string;
  price: number;
  stock: number;
}
```

然后建立：

```ts
const products: Product[] = [];
```

---

# Day 2：interface / type / union

## 上午

掌握：

```text
interface
type
optional property
readonly
extends
```

练习：

```ts
interface User {
  readonly id: number;
  name: string;
  age?: number;
}
```

## 下午

重点：

```ts
type ID = string | number;
```

理解：

```text
Union = OR
Intersection = AND
```

练习：

```ts
type Admin = User & {
  permissions: string[];
};
```

## 晚上

设计一个电商模型：

```text
User
Product
Order
OrderItem
Address
Payment
```

不要急着写数据库。

先把现实世界映射成 TypeScript 类型。

---

# Day 3：narrowing + function + generic

这是整个 7 天里非常关键的一天。

## 上午：narrowing

必须熟练：

```ts
typeof
instanceof
in
Array.isArray()
```

练习：

```ts
function formatValue(value: string | number | null) {
  // 自己完成
}
```

---

## 下午：函数

练习：

```ts
function calculateTotal(
  price: number,
  quantity: number,
  discount?: number
): number {
  // 自己实现
}
```

然后定义函数类型：

```ts
type Calculator = (
  a: number,
  b: number
) => number;
```

---

## 晚上：泛型

必须手写：

```ts
function identity<T>(value: T): T {
  return value;
}
```

然后：

```ts
function first<T>(items: T[]): T | undefined {
  return items[0];
}
```

再挑战：

```ts
function getProperty<T, K extends keyof T>(
  obj: T,
  key: K
): T[K] {
  return obj[key];
}
```

如果这三个函数你能真正理解，已经进入 TypeScript 的核心领域。

---

# Day 4：高级类型

## 上午

学习：

```text
keyof
typeof
indexed access
as const
```

练习：

```ts
const roles = ["admin", "user", "guest"] as const;

type Role = typeof roles[number];
```

---

## 下午

学习：

```text
Partial
Required
Pick
Omit
Record
Readonly
ReturnType
Parameters
```

重点理解，而不是背 API。

例如：

```ts
interface User {
  id: number;
  name: string;
  age: number;
}

type UpdateUser = Partial<User>;
type UserPreview = Pick<User, "id" | "name">;
type UserWithoutId = Omit<User, "id">;
```

---

## 晚上

第一次接触：

```text
Conditional Types
Mapped Types
infer
Template Literal Types
```

不用要求一次掌握。

今天的目标是：

> 看得懂基本结构。

---

# Day 5：模块 + tsconfig + Node.js

## 上午

掌握：

```ts
export
import
export default
import type
```

建立：

```text
src/
  user.ts
  product.ts
  index.ts
```

让它们互相 import/export。

---

## 下午

学习：

```text
tsconfig.json
strict
target
module
moduleResolution
noEmit
rootDir
outDir
```

重点不是背配置，而是回答：

```text
TypeScript 在哪里运行？
谁负责模块加载？
谁负责构建？
谁负责类型检查？
最终输出什么？
```

---

## 晚上

建立一个最小 Node 项目：

```text
src/
  index.ts
  user.ts
```

完成：

```bash
npx tsc --noEmit
```

确保项目没有类型错误。

---

# Day 6：真实 API 项目

推荐做一个：

> 用户管理 API

功能：

```text
POST   /users
GET    /users
GET    /users/:id
PATCH  /users/:id
DELETE /users/:id
```

---

## 第一步：User

```ts
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
}
```

---

## 第二步：CreateUserInput

```ts
interface CreateUserInput {
  name: string;
  email: string;
}
```

---

## 第三步：UpdateUserInput

```ts
type UpdateUserInput = Partial<CreateUserInput>;
```

---

## 第四步：Repository

```ts
interface UserRepository {
  findAll(): Promise<User[]>;
  findById(id: number): Promise<User | null>;
  create(input: CreateUserInput): Promise<User>;
  update(id: number, input: UpdateUserInput): Promise<User | null>;
  delete(id: number): Promise<boolean>;
}
```

---

## 第五步：Service

```ts
class UserService {
  constructor(private readonly repository: UserRepository) {}

  async getUser(id: number) {
    return this.repository.findById(id);
  }
}
```

这样你已经不是在“练 TypeScript 语法”，而是在用 TypeScript 设计程序。

---

# Day 7：综合项目 + 重构

最后一天不要继续看新语法。

重点是：

```text
设计
↓
实现
↓
发现类型问题
↓
重构
↓
总结
```

推荐做：

## 项目：Todo / Task Manager API

数据模型：

```ts
interface User {
  id: number;
  name: string;
}

interface Task {
  id: number;
  title: string;
  completed: boolean;
  userId: number;
}
```

API：

```text
POST   /users
GET    /users
POST   /tasks
GET    /tasks
PATCH  /tasks/:id
DELETE /tasks/:id
```

---

## Day 7 必须完成的检查

### 类型

- 不允许随便使用 `any`
- 打开 `strict`
- 对外部数据进行明确建模
- API Request / Response 分离
- Service / Repository 有类型

### 工程

- import / export 清晰
- tsconfig 正确
- `tsc --noEmit` 无错误
- 文件职责清晰

### 代码质量

- 尽可能利用类型推导
- 不重复定义相同类型
- 合理使用泛型
- 避免为了炫技使用复杂类型

---

# 二十二、每天练习题

## 基础题

### 题目 1

定义用户：

```ts
interface User {
  // 自己完成
}
```

要求：

```text
id: number
name: string
email: string
age: optional number
```

---

### 题目 2

定义：

```ts
type ID = ?
```

要求 ID 可以是字符串或数字。

---

### 题目 3

实现：

```ts
function getFirst<T>(items: T[]): T | undefined {
  // 自己完成
}
```

---

### 题目 4

实现：

```ts
function getProperty<T, K extends keyof T>(
  obj: T,
  key: K
): T[K] {
  // 自己完成
}
```

---

### 题目 5

定义：

```ts
type UpdateUser = ?
```

要求：

```text
User 所有字段变为 optional
```

---

### 题目 6

定义 API 状态：

```ts
type ApiResult<T> =
  | { status: "success"; data: T }
  | { status: "error"; message: string };
```

写一个函数正确处理两种情况。

---

# 二十三、常见坑

## 坑 1：把 TypeScript 当 JavaScript 新语法

错误理解：

```text
TypeScript = JavaScript 的新版本
```

正确理解：

```text
TypeScript = JavaScript + 类型系统 + 工具链
```

---

## 坑 2：到处写 any

错误：

```ts
function parse(data: any) {
  return data;
}
```

更推荐：

```ts
function parse(data: unknown) {
  // 先验证
}
```

---

## 坑 3：过度手写类型

例如：

```ts
const user: {
  id: number;
  name: string;
} = {
  id: 1,
  name: "Alice",
};
```

如果 TypeScript 已经能正确推导，就没必要把所有地方写满。

---

## 坑 4：滥用 as

```ts
const user = response as User;
```

`as` 很方便，但越多通常意味着你正在绕过 TypeScript。

特别是在外部 JSON、数据库、HTTP 请求等场景，真正需要的是 runtime validation。

---

## 坑 5：把 interface 当数据库模型万能替代物

现实项目通常至少有：

```text
Database Entity
Domain Model
Request DTO
Response DTO
```

它们可以相似，但不应该强制全部相同。

---

## 坑 6：为了炫技使用高级类型

看到：

```ts
T extends infer U ? ...
```

不代表必须使用。

好的 TypeScript 是：

```text
简单
可读
可维护
类型安全
```

而不是：

```text
类型体操越复杂越厉害
```

---

# 二十四、TypeScript 高频速查表

## 基础类型

```ts
string
number
boolean
bigint
symbol
null
undefined
unknown
never
void
any
```

---

## 对象

```ts
interface User {
  id: number;
}
```

```ts
type User = {
  id: number;
};
```

---

## Union

```ts
type ID = string | number;
```

---

## Intersection

```ts
type Admin = User & Permission;
```

---

## Array

```ts
string[]
Array<string>
```

---

## Tuple

```ts
[string, number]
```

---

## Optional

```ts
age?: number;
```

---

## Readonly

```ts
readonly id: number;
```

---

## Function

```ts
(a: number, b: number) => number
```

---

## Generic

```ts
function identity<T>(value: T): T {
  return value;
}
```

---

## keyof

```ts
keyof User
```

---

## typeof

```ts
type Config = typeof config;
```

---

## Indexed Access

```ts
User["name"]
```

---

## Conditional

```ts
T extends U ? A : B
```

---

## Mapped

```ts
[K in keyof T]
```

---

## infer

```ts
T extends (...args: never[]) => infer R
  ? R
  : never
```

---

## Utility Types

```text
Partial
Required
Readonly
Pick
Omit
Record
ReturnType
Parameters
Awaited
NonNullable
Extract
Exclude
```

---

# 二十五、七天之后如何继续进阶

七天只能完成“建立体系”，不能让你真正达到 TypeScript 熟练水平。

建议第二阶段继续按照下面路线：

```text
第二周
↓
泛型强化
↓
第三周
↓
类型体操
↓
第四周
↓
Node.js + TypeScript
↓
第五周
↓
Fastify + TypeScript
↓
第六周
↓
SQL / PostgreSQL + TypeScript
↓
第七周
↓
API Schema / DTO / Validation
↓
第八周
↓
测试 / 工程化 / Monorepo
```

---

## 第二阶段：重点突破泛型

要能够看懂：

```ts
type Result<T> = {
  data: T;
  error?: string;
};
```

然后继续：

```ts
interface Repository<T> {
  findById(id: string): Promise<T | null>;
}
```

再到：

```ts
function createRepository<T>() {
  // ...
}
```

最终形成：

```text
类型参数
↓
类型约束
↓
keyof
↓
索引访问
↓
条件类型
↓
映射类型
```

---

# 最终学习框架

把整个 TypeScript 压缩成下面这一张图：

```text
                         TypeScript
                              │
             ┌────────────────┼────────────────┐
             │                │                │
          基础类型          类型建模          工程化
             │                │                │
      string/number      interface/type      tsconfig
      boolean/array      union/intersection   module
      tuple/object       literal             npm
             │                │                │
             └────────────────┼────────────────┘
                              │
                         类型推导
                              │
                    ┌─────────┼─────────┐
                    │         │         │
                 inference narrowing generic
                    │         │         │
                    └─────────┼─────────┘
                              │
                         高级类型
                              │
             ┌────────────────┼────────────────┐
             │                │                │
            keyof           typeof          indexed access
             │                │                │
             └────────────────┼────────────────┘
                              │
                   conditional / mapped
                              │
                            infer
                              │
                              ↓
                       真实项目设计
                              │
          ┌───────────────────┼──────────────────┐
          │                   │                  │
        Node.js             Fastify             前端
          │                   │                  │
       Service            Schema/DTO          React/Vue
       Repository          API Type             State
          │                   │                  │
          └───────────────────┼──────────────────┘
                              │
                           数据库
                              │
                           SQL
```

---

# 学习时最重要的 10 条原则

1. **先学 JavaScript，再学 TypeScript。**
2. **不要把所有变量都手动标注类型，优先利用类型推导。**
3. **项目尽量开启 `strict`。**
4. **`any` 是逃生舱，不是默认类型。**
5. **外部数据默认不可信，`unknown` + runtime validation 比 `any` 更可靠。**
6. **理解 Union 后再学 narrowing。**
7. **理解 generic 后再学 `keyof` / `typeof` / mapped / conditional。**
8. **不要只学语法，要拿真实 API、数据库、Node.js 项目练习。**
9. **TypeScript 类型安全只发生在类型系统边界内，不能替代运行时校验。**
10. **最终目标不是写很多类型，而是设计出容易维护、容易重构的程序。**

---

# 推荐官方资料

- TypeScript Handbook：<https://www.typescriptlang.org/docs/handbook/>
- TypeScript TSConfig Reference：<https://www.typescriptlang.org/tsconfig/>
- TypeScript Modules：<https://www.typescriptlang.org/docs/handbook/2/modules.html>
- TypeScript Conditional Types：<https://www.typescriptlang.org/docs/handbook/2/conditional-types.html>
- TypeScript Releases：<https://github.com/microsoft/TypeScript/releases>

---

# 一句话总结

> **TypeScript 的学习重点不是“把 JavaScript 每个变量都加上类型”，而是逐步建立一套“用类型描述程序、用类型约束程序、用类型辅助重构程序”的思维方式。**

推荐学习顺序最终固定为：

```text
JavaScript 基础
    ↓
基础类型
    ↓
interface / type
    ↓
union / intersection
    ↓
narrowing
    ↓
函数类型
    ↓
generic
    ↓
keyof / typeof / indexed access
    ↓
utility types
    ↓
conditional / mapped / infer
    ↓
module / tsconfig
    ↓
Node.js / Fastify
    ↓
API / SQL / Database
    ↓
大型项目 TypeScript 工程化
```

完成第一轮 7 天学习后，最值得继续做的不是继续背 API，而是亲手把一个小型 Node.js 项目完整用 TypeScript 重写一遍。
