# TypeScript 完整入门指南：七天学习计划

> 适合人群：有 JavaScript 基础（会写变量、函数、对象、数组、`async/await`）的初学者。
> 学习节奏：每天 1.5 ~ 2.5 小时（约 40% 阅读 + 60% 动手敲代码）。
> 核心原则：**不要只看，一定要亲手敲，并故意写错来看编译器报错。**

---

## 目录

- [学习路线总览](#学习路线总览)
- [Day 0：环境准备（30 分钟）](#day-0环境准备30-分钟)
- [Day 1：基础类型与类型推断](#day-1基础类型与类型推断)
- [Day 2：函数、对象类型、联合与交叉类型](#day-2函数对象类型联合与交叉类型)
- [Day 3：类型收窄与可辨识联合](#day-3类型收窄与可辨识联合)
- [Day 4：泛型](#day-4泛型)
- [Day 5：类、模块与声明文件](#day-5类模块与声明文件)
- [Day 6：高级类型与内置工具类型](#day-6高级类型与内置工具类型)
- [Day 7：工程实践与综合项目](#day-7工程实践与综合项目)
- [附录 A：常见报错速查](#附录-a常见报错速查)
- [附录 B：速查表](#附录-b速查表)
- [附录 C：学习资源与后续路线](#附录-c学习资源与后续路线)
- [附录 D：七天自测清单](#附录-d七天自测清单)

---

## 学习路线总览

| 天数 | 主题 | 你将学会 |
|------|------|----------|
| Day 0 | 环境准备 | 安装、运行、编译、`tsconfig.json` |
| Day 1 | 基础类型 | 类型注解、类型推断、数组、元组、`any`/`unknown`/`never` |
| Day 2 | 函数与对象 | 函数类型、`interface` 与 `type`、联合/交叉类型、字面量类型 |
| Day 3 | 类型收窄 | `typeof`/`in`/`instanceof`、自定义类型守卫、可辨识联合 |
| Day 4 | 泛型 | 泛型函数、泛型约束、泛型接口与类 |
| Day 5 | 类与模块 | class、访问修饰符、抽象类、模块、`.d.ts`、`@types` |
| Day 6 | 高级类型 | `keyof`、`typeof`、映射类型、条件类型、`infer`、工具类型 |
| Day 7 | 工程实践 | 异步类型、错误处理、严格模式、综合小项目 |

**TypeScript 是什么？**
TypeScript = JavaScript + 静态类型系统。它在**编译阶段**检查类型错误，最终输出（或直接运行时剥离类型后）仍是普通 JavaScript。类型信息**不会**存在于运行时。

---

## Day 0：环境准备（30 分钟）

### 1. 安装 Node.js

建议安装 LTS 版本，然后确认：

```bash
node -v
npm -v
```

### 2. 创建练习项目

```bash
mkdir ts-learn && cd ts-learn
npm init -y
npm install -D typescript tsx @types/node
npx tsc --init
mkdir src
```

- `typescript`：编译器 `tsc`，负责类型检查与编译
- `tsx`：可以直接运行 `.ts` 文件，练习时非常方便
- `@types/node`：Node.js 的类型声明

### 3. 第一个程序

新建 `src/hello.ts`：

```ts
const message: string = "Hello, TypeScript!";
console.log(message);
```

运行与检查：

```bash
npx tsx src/hello.ts       # 直接运行
npx tsc --noEmit           # 只做类型检查，不生成文件
npx tsc --noEmit --watch   # 监听模式，边写边检查
```

### 4. 推荐的 `tsconfig.json`（学习阶段）

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "outDir": "dist",
    "rootDir": "src"
  },
  "include": ["src"]
}
```

> **务必开启 `"strict": true`。** 这是 TypeScript 最有价值的部分，关掉它就失去了大半意义。

### 5. 编辑器

推荐 VS Code：内置 TypeScript 支持，**鼠标悬停**可查看推断出的类型，`F12` 跳转定义，`F2` 重命名。

---

## Day 1：基础类型与类型推断

### 学习目标

- 会使用类型注解，理解类型推断
- 掌握原始类型、数组、元组、枚举的基本用法
- 分清 `any`、`unknown`、`never`、`void`

### 1.1 类型注解与类型推断

```ts
// 显式注解
let username: string = "Alice";
let age: number = 30;
let isActive: boolean = true;

// 类型推断：右侧值已经能确定类型，通常不需要再写注解
let city = "Kaohsiung"; // 推断为 string
// city = 123;          // ❌ 不能将 number 赋值给 string

const PI = 3.14;        // const 会推断为字面量类型 3.14
```

**经验法则：** 变量初始化时能推断就不写；函数参数和公开 API 的返回值建议写清楚。

### 1.2 原始类型

```ts
let s: string = "text";
let n: number = 42;          // 整数和浮点数都是 number
let b: boolean = false;
let big: bigint = 100n;
let sym: symbol = Symbol("id");
let nothing: null = null;
let notSet: undefined = undefined;
```

> 注意小写：用 `string`、`number`、`boolean`，**不要**用 `String`、`Number`、`Boolean`（那是包装对象类型）。

### 1.3 数组与元组

```ts
const nums: number[] = [1, 2, 3];
const names: Array<string> = ["a", "b"]; // 等价写法

// 元组：固定长度、每个位置类型固定
const pair: [string, number] = ["age", 30];
const [key, value] = pair;

// 带名字的元组（可读性更好）
type Point = [x: number, y: number];
const p: Point = [10, 20];

// 只读数组
const readonlyNums: readonly number[] = [1, 2, 3];
// readonlyNums.push(4); // ❌ 只读数组没有 push
```

### 1.4 `any`、`unknown`、`never`、`void`

```ts
// any：关闭类型检查（尽量避免）
let a: any = 1;
a.foo.bar(); // 编译不报错，运行时可能崩溃

// unknown：类型安全的"未知"，使用前必须先收窄
let u: unknown = "hello";
// u.toUpperCase();          // ❌ 不能直接使用
if (typeof u === "string") {
  console.log(u.toUpperCase()); // ✅ 收窄后可用
}

// void：函数没有返回值
function log(msg: string): void {
  console.log(msg);
}

// never：永远不会有返回值（抛异常、死循环）
function fail(msg: string): never {
  throw new Error(msg);
}
```

| 类型 | 含义 | 建议 |
|------|------|------|
| `any` | 放弃检查 | 尽量不用 |
| `unknown` | 不知道是什么，用之前必须检查 | 处理外部数据时优先使用 |
| `void` | 无返回值 | 函数返回类型 |
| `never` | 不可能出现的值 | 穷尽检查 |

### 1.5 `null` 与 `undefined`（严格模式下）

```ts
let x: string = "a";
// x = null;                 // ❌ 严格模式下不允许

let y: string | null = null; // ✅ 用联合类型显式允许
y = "hello";

// 可选链与空值合并
const user = { profile: { nickname: undefined as string | undefined } };
const nick = user.profile?.nickname ?? "匿名";
```

### 1.6 枚举与字面量联合（推荐后者）

```ts
// 数字枚举
enum Direction {
  Up,    // 0
  Down,  // 1
}

// 字符串枚举
enum Status {
  Active = "ACTIVE",
  Inactive = "INACTIVE",
}

// 更轻量、更常用：字面量联合类型
type Role = "admin" | "user" | "guest";
let role: Role = "admin";
// role = "root"; // ❌ 不在联合里
```

> 现代 TypeScript 项目里，大多数场景用**字面量联合类型**代替 `enum`：没有运行时代码，也更容易和 JavaScript 互操作。

### 1.7 类型断言（谨慎使用）

```ts
const input = document.getElementById("name") as HTMLInputElement;
// 断言是"我比编译器更清楚"，写错了编译器不会拦你
```

### ✍️ Day 1 练习

1. 声明一个学生信息：姓名（string）、年龄（number）、成绩数组（number[]），并写一个函数计算平均分。
2. 用元组表示 `[名称, 价格]`，声明一个商品列表。
3. 声明 `type Weekday = "Mon" | "Tue" | ...`，写函数 `isWeekend(d: Weekday): boolean`。
4. 故意把 string 赋给 number 变量，观察报错信息并读懂它。

---

## Day 2：函数、对象类型、联合与交叉类型

### 学习目标

- 给函数参数、返回值、回调加类型
- 掌握 `interface` 与 `type` 的用法和区别
- 会使用联合类型（`|`）、交叉类型（`&`）与字面量类型

### 2.1 函数类型

```ts
function add(a: number, b: number): number {
  return a + b;
}

const mul = (a: number, b: number): number => a * b;

// 可选参数、默认参数、剩余参数
function greet(name: string, greeting = "Hello", punctuation?: string): string {
  return `${greeting}, ${name}${punctuation ?? "!"}`;
}

function sum(...nums: number[]): number {
  return nums.reduce((acc, n) => acc + n, 0);
}

// 函数类型别名
type BinaryOp = (a: number, b: number) => number;
const subtract: BinaryOp = (a, b) => a - b; // 参数类型自动推断

// 回调参数
function repeat(times: number, fn: (index: number) => void): void {
  for (let i = 0; i < times; i++) fn(i);
}
```

**函数重载**（同一函数针对不同入参给出不同返回类型）：

```ts
function parse(input: string): number;
function parse(input: string[]): number[];
function parse(input: string | string[]) {
  return Array.isArray(input) ? input.map(Number) : Number(input);
}

const one = parse("42");        // number
const many = parse(["1", "2"]); // number[]
```

### 2.2 对象类型：`interface` 与 `type`

```ts
interface User {
  readonly id: number;   // 只读属性
  name: string;
  email?: string;        // 可选属性
}

const u: User = { id: 1, name: "Alice" };
// u.id = 2;             // ❌ 只读

// type 别名
type Point = {
  x: number;
  y: number;
};
```

**继承 / 扩展：**

```ts
interface Animal {
  name: string;
}
interface Dog extends Animal {
  breed: string;
}

// type 用交叉类型达到类似效果
type Cat = Animal & { color: string };
```

**`interface` vs `type` 怎么选？**

| 对比项 | `interface` | `type` |
|--------|-------------|--------|
| 描述对象结构 | ✅ | ✅ |
| 联合类型 / 元组 / 基础类型别名 | ❌ | ✅ |
| 同名声明自动合并 | ✅ | ❌ |
| 继承方式 | `extends` | `&` |

> 简单建议：**描述对象结构、需要被 class 实现时用 `interface`；其余（联合、元组、工具类型运算）用 `type`。** 团队统一风格比纠结哪个更重要。

### 2.3 索引签名与 `Record`

```ts
// 键是任意字符串，值是 number
interface ScoreMap {
  [subject: string]: number;
}
const scores: ScoreMap = { math: 90, english: 85 };

// 更简洁的写法
const scores2: Record<string, number> = { math: 90 };

// 开启 noUncheckedIndexedAccess 后，按索引取值会带上 undefined
const m = scores["math"]; // number | undefined
```

### 2.4 联合类型 `|` 与交叉类型 `&`

```ts
// 联合：多选一
type ID = string | number;
function printId(id: ID) {
  if (typeof id === "string") {
    console.log(id.toUpperCase());
  } else {
    console.log(id.toFixed(0));
  }
}

// 交叉：合并多个类型
type WithTimestamp = { createdAt: Date; updatedAt: Date };
type Article = { title: string; content: string } & WithTimestamp;
```

### 2.5 字面量类型与 `as const`

```ts
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";

function request(url: string, method: HttpMethod) {
  /* ... */
}
request("/api", "GET");
// request("/api", "get"); // ❌

// as const：把对象/数组变成"深度只读的字面量类型"
const config = {
  env: "prod",
  retries: 3,
} as const;
// config.env 的类型是 "prod"，而不是 string

const COLORS = ["red", "green", "blue"] as const;
type Color = (typeof COLORS)[number]; // "red" | "green" | "blue"
```

> 最后这个 `typeof COLORS)[number]` 技巧很常用：**从值反推类型**，避免值和类型两处重复维护。

### ✍️ Day 2 练习

1. 定义 `interface Product`（id、name、price、可选 description），写函数 `formatPrice(p: Product): string`。
2. 定义 `type Result = { ok: true; data: string } | { ok: false; error: string }`，写函数处理两种情况。
3. 用 `as const` 定义一个角色数组，导出对应的联合类型。
4. 分别用 `interface extends` 和 `type &` 实现"员工继承人员"，对比体验。

---

## Day 3：类型收窄与可辨识联合

### 学习目标

- 理解"类型收窄（Narrowing）"：编译器如何根据代码逻辑缩小类型范围
- 掌握各类守卫写法
- 会用**可辨识联合 + 穷尽检查**建模业务状态（TypeScript 最实用的模式之一）

### 3.1 常见收窄方式

```ts
// typeof
function double(x: string | number) {
  if (typeof x === "string") return x.repeat(2);
  return x * 2;
}

// 真值判断
function printLen(s?: string) {
  if (s) console.log(s.length); // s 此处是 string
}

// 相等判断
function compare(a: string | number, b: string | boolean) {
  if (a === b) {
    // a、b 此处都被收窄为 string
  }
}

// in 运算符
type Fish = { swim: () => void };
type Bird = { fly: () => void };
function move(animal: Fish | Bird) {
  if ("swim" in animal) animal.swim();
  else animal.fly();
}

// instanceof
function fmt(d: Date | string) {
  return d instanceof Date ? d.toISOString() : d;
}
```

### 3.2 自定义类型守卫（`is`）

```ts
interface Cat { meow(): void }
interface Dog { bark(): void }

function isCat(pet: Cat | Dog): pet is Cat {
  return "meow" in pet;
}

function talk(pet: Cat | Dog) {
  if (isCat(pet)) pet.meow();
  else pet.bark();
}

// 常见用途：过滤掉 null / undefined
const raw = ["a", null, "b", undefined];
const cleaned = raw.filter((x): x is string => x != null); // string[]
```

### 3.3 可辨识联合（Discriminated Union）

给每个成员一个共同的**字面量字段**（如 `kind`），TypeScript 就能据此精准收窄：

```ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; size: number }
  | { kind: "rect"; width: number; height: number };

function area(s: Shape): number {
  switch (s.kind) {
    case "circle":
      return Math.PI * s.radius ** 2;
    case "square":
      return s.size ** 2;
    case "rect":
      return s.width * s.height;
    default: {
      // 穷尽检查：如果将来新增了 Shape 成员却忘了处理，这里会编译报错
      const _exhaustive: never = s;
      return _exhaustive;
    }
  }
}
```

**实战：用可辨识联合表达异步请求状态**

```ts
type RequestState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: string };

function render(state: RequestState<string[]>): string {
  switch (state.status) {
    case "idle":
      return "点击加载";
    case "loading":
      return "加载中…";
    case "success":
      return state.data.join(", ");
    case "error":
      return `出错：${state.error}`;
  }
}
```

相比 `{ loading: boolean; data?: T; error?: string }`，这种建模**让非法状态无法被表达**（比如同时 loading 和 error）。

### 3.4 断言函数与非空断言（了解即可）

```ts
// 非空断言 !：告诉编译器"这里绝对不是 null"。写错会在运行时崩，尽量少用
const el = document.querySelector("#app")!;

// 断言函数
function assertIsString(v: unknown): asserts v is string {
  if (typeof v !== "string") throw new Error("Not a string");
}
```

### ✍️ Day 3 练习

1. 写一个 `Result<T>` 可辨识联合（`ok: true/false`），并写函数 `unwrap` 处理两个分支。
2. 给上面的 `Shape` 新增一个 `triangle`，观察 `area` 中 `never` 的报错，然后补全。
3. 写自定义守卫 `isNonEmptyString(x: unknown): x is string`。
4. 用可辨识联合建模支付方式：信用卡（卡号）、支付宝（账号）、现金（无字段），并写函数输出描述。

---

## Day 4：泛型

### 学习目标

- 理解泛型的动机：**类型的"参数化"**，写一次逻辑，适配多种类型
- 会写泛型函数、泛型接口、泛型类
- 会用 `extends` 约束泛型，会用默认类型参数

### 4.1 为什么需要泛型

```ts
// ❌ 用 any：丢失了类型信息
function identityAny(x: any): any {
  return x;
}

// ✅ 泛型：入什么类型，出什么类型
function identity<T>(x: T): T {
  return x;
}

const a = identity("hi");     // T 推断为 "hi"（字面量）/ string
const b = identity<number>(1); // 也可以显式指定
```

### 4.2 泛型函数

```ts
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}
first([1, 2, 3]);      // number | undefined
first(["a", "b"]);     // string | undefined

// 多个类型参数
function pair<A, B>(a: A, b: B): [A, B] {
  return [a, b];
}

// 泛型 + 回调
function map<T, U>(arr: T[], fn: (item: T) => U): U[] {
  return arr.map(fn);
}
const lens = map(["a", "bb", "ccc"], (s) => s.length); // number[]
```

### 4.3 泛型约束 `extends`

```ts
// 要求 T 必须有 length 属性
function longest<T extends { length: number }>(a: T, b: T): T {
  return a.length >= b.length ? a : b;
}
longest("abc", "de");       // ✅
longest([1, 2], [1, 2, 3]); // ✅
// longest(1, 2);           // ❌ number 没有 length

// keyof 约束：安全地按属性名取值
function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
const user = { id: 1, name: "Alice" };
getProp(user, "name"); // string
// getProp(user, "age"); // ❌ "age" 不是 user 的键
```

### 4.4 泛型接口、类型别名与类

```ts
// 通用 API 响应
interface ApiResponse<T> {
  code: number;
  data: T;
  message?: string;
}
type UserResp = ApiResponse<{ id: number; name: string }>;

// 泛型类
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }
  pop(): T | undefined {
    return this.items.pop();
  }
  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }
  get size(): number {
    return this.items.length;
  }
}

const s = new Stack<number>();
s.push(1);
// s.push("a"); // ❌
```

### 4.5 默认类型参数

```ts
interface Page<T = unknown> {
  list: T[];
  total: number;
}
const p1: Page = { list: [], total: 0 };          // T = unknown
const p2: Page<string> = { list: ["a"], total: 1 };
```

### 4.6 泛型使用心得

- 类型参数命名：简单场景用 `T`、`U`、`K`、`V`；复杂场景用有意义的名字，如 `TItem`、`TResponse`。
- **不要为了泛型而泛型**：如果类型参数只出现一次，通常不需要泛型。
- 泛型参数越少越好，约束越准确越好。

### ✍️ Day 4 练习

1. 实现泛型函数 `last<T>(arr: T[]): T | undefined`。
2. 实现 `groupBy<T, K extends string | number>(arr: T[], keyFn: (item: T) => K): Record<K, T[]>`。
3. 实现泛型类 `Queue<T>`（`enqueue` / `dequeue` / `size`）。
4. 实现 `pluck<T, K extends keyof T>(arr: T[], key: K): T[K][]`，如从用户数组取出所有 `name`。

---

## Day 5：类、模块与声明文件

### 学习目标

- 掌握 class 的类型特性（访问修饰符、参数属性、抽象类、`implements`）
- 会用 ES Module 组织代码，理解 `import type`
- 理解 `.d.ts` 声明文件与 `@types` 包

### 5.1 类的基础

```ts
class Animal {
  // 参数属性：构造函数参数前加修饰符，自动声明并赋值同名属性
  constructor(
    public readonly name: string,
    protected age: number,
  ) {}

  speak(): string {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  constructor(name: string, age: number, private breed: string) {
    super(name, age);
  }
  override speak(): string {
    return `${this.name} (${this.breed}, ${this.age}岁) barks`;
  }
}

const d = new Dog("Lucky", 3, "Corgi");
d.speak();
// d.age;    // ❌ protected，外部不可访问
// d.breed;  // ❌ private
```

| 修饰符 | 类内 | 子类 | 类外 |
|--------|------|------|------|
| `public`（默认） | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ❌ |
| `private` | ✅ | ❌ | ❌ |

> TS 的 `private` 只在**编译期**限制。若需要运行时真正私有，使用 JavaScript 原生的 `#field` 语法。

```ts
class Counter {
  #count = 0; // 运行时真私有
  inc() {
    this.#count++;
    return this.#count;
  }
}
```

### 5.2 `implements` 与抽象类

```ts
interface Printable {
  print(): void;
}

class Report implements Printable {
  print() {
    console.log("printing report");
  }
}

abstract class Shape {
  abstract area(): number;       // 子类必须实现
  describe(): string {
    return `area = ${this.area().toFixed(2)}`;
  }
}

class Circle extends Shape {
  constructor(private r: number) {
    super();
  }
  area() {
    return Math.PI * this.r ** 2;
  }
}
// new Shape(); // ❌ 抽象类不能实例化
```

### 5.3 模块（ES Module）

`src/math.ts`：

```ts
export const PI = 3.14159;

export function add(a: number, b: number): number {
  return a + b;
}

export interface Point {
  x: number;
  y: number;
}

export default function greet(name: string) {
  return `Hi, ${name}`;
}
```

`src/main.ts`：

```ts
import greet, { add, PI } from "./math";
import type { Point } from "./math"; // 仅导入类型，编译后会被完全擦除

const p: Point = { x: 1, y: 2 };
console.log(greet("TS"), add(1, 2), PI, p);
```

要点：

- 每个包含 `import`/`export` 的文件都是一个模块。
- 仅用作类型的导入，推荐写 `import type`，语义清晰且利于打包工具优化。
- 类型和值可以同时从一个文件导出，但**类型在运行时不存在**。

### 5.4 声明文件 `.d.ts` 与 `@types`

当你使用的 JavaScript 库没有内置类型时，需要声明文件：

```bash
npm install -D @types/lodash   # 很多流行库的类型都在 @types 下
```

自己编写声明（例如给一个无类型的库补声明）：

`src/types/legacy-lib.d.ts`

```ts
declare module "legacy-lib" {
  export function doSomething(input: string): number;
}
```

给全局对象扩展属性：

```ts
// src/types/global.d.ts
export {};

declare global {
  interface Window {
    appVersion: string;
  }
}
```

> 现在大多数新库自带类型（`package.json` 里有 `types` 字段），先检查库本身，再考虑 `@types/xxx`。

### ✍️ Day 5 练习

1. 写抽象类 `Employee`（`name`、抽象方法 `salary()`），实现 `FullTime` 与 `PartTime` 两个子类。
2. 写一个模块 `utils/string.ts` 导出 3 个函数，在 `main.ts` 中导入使用。
3. 尝试 `npm i -D @types/node`，然后使用 `import { readFileSync } from "node:fs"` 读取一个文件并观察类型提示。
4. 用 `implements` 让两个不同的类实现同一个 `Logger` 接口（控制台版与内存版）。

---

## Day 6：高级类型与内置工具类型

### 学习目标

- 掌握 `keyof`、`typeof`、索引访问类型
- 理解映射类型、条件类型、`infer`
- 熟练使用常用内置工具类型
- 了解模板字面量类型与 `satisfies`

### 6.1 `keyof`、`typeof`、索引访问类型

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

type UserKeys = keyof User;         // "id" | "name" | "email"
type NameType = User["name"];       // string
type IdOrName = User["id" | "name"]; // number | string

// typeof：从"值"得到"类型"
const defaultUser = { id: 0, name: "guest", email: "" };
type DefaultUser = typeof defaultUser; // { id: number; name: string; email: string }

// 数组元素类型
type Fruit = (typeof ["apple", "pear"])[number]; // string（配合 as const 可得到字面量联合）
```

### 6.2 映射类型（Mapped Types）

```ts
// 把所有属性变为可选
type MyPartial<T> = {
  [K in keyof T]?: T[K];
};

// 把所有属性变为只读
type MyReadonly<T> = {
  readonly [K in keyof T]: T[K];
};

// 去掉只读（- 修饰符）
type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};

// 键重映射：为每个属性生成 getter 名
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};
type UserGetters = Getters<User>;
// { getId: () => number; getName: () => string; getEmail: () => string }
```

### 6.3 条件类型与 `infer`

```ts
// T extends U ? X : Y
type IsString<T> = T extends string ? true : false;
type A = IsString<"a">; // true
type B = IsString<1>;   // false

// 分布式条件类型：对联合类型逐个判断
type ToArray<T> = T extends unknown ? T[] : never;
type C = ToArray<string | number>; // string[] | number[]

// infer：在条件类型中"提取"类型
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type R1 = MyReturnType<() => number>; // number

type ElementOf<T> = T extends (infer U)[] ? U : never;
type E = ElementOf<string[]>; // string

// 拆开 Promise
type Unwrap<T> = T extends Promise<infer U> ? U : T;
type P = Unwrap<Promise<number>>; // number
```

### 6.4 常用内置工具类型

| 工具类型 | 作用 | 示例 |
|----------|------|------|
| `Partial<T>` | 所有属性可选 | `Partial<User>` |
| `Required<T>` | 所有属性必填 | `Required<User>` |
| `Readonly<T>` | 所有属性只读 | `Readonly<User>` |
| `Pick<T, K>` | 选取部分属性 | `Pick<User, "id" \| "name">` |
| `Omit<T, K>` | 排除部分属性 | `Omit<User, "email">` |
| `Record<K, V>` | 构造键值对象 | `Record<string, number>` |
| `Exclude<T, U>` | 从联合中排除 | `Exclude<"a" \| "b", "a">` → `"b"` |
| `Extract<T, U>` | 从联合中提取 | `Extract<string \| number, number>` → `number` |
| `NonNullable<T>` | 去掉 null/undefined | `NonNullable<string \| null>` → `string` |
| `ReturnType<F>` | 函数返回类型 | `ReturnType<typeof fn>` |
| `Parameters<F>` | 函数参数元组 | `Parameters<typeof fn>` |
| `Awaited<T>` | 展开 Promise | `Awaited<Promise<number>>` → `number` |

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

// 更新用户：id 不可改，其他可选
type UpdateUserDto = Partial<Omit<User, "id">>;

function updateUser(id: number, patch: UpdateUserDto): User {
  const old: User = { id, name: "old", email: "old@example.com" };
  return { ...old, ...patch };
}

// 从函数推导类型，而不是重复手写
function createUser() {
  return { id: 1, name: "Alice", createdAt: new Date() };
}
type CreatedUser = ReturnType<typeof createUser>;
```

### 6.5 模板字面量类型

```ts
type Lang = "zh" | "en";
type Page = "home" | "about";
type Route = `/${Lang}/${Page}`;
// "/zh/home" | "/zh/about" | "/en/home" | "/en/about"

type EventName<T extends string> = `on${Capitalize<T>}`;
type ClickEvent = EventName<"click">; // "onClick"
```

### 6.6 `satisfies`：校验类型但保留精确推断

```ts
type Theme = Record<string, string | number[]>;

const theme = {
  primary: "#3366ff",
  spacing: [4, 8, 16],
} satisfies Theme;

theme.primary.toUpperCase(); // ✅ 仍然知道它是 string
theme.spacing.map((n) => n * 2); // ✅ 仍然知道它是 number[]
// 如果只写 const theme: Theme = ...，上面两行都会报错，因为类型被"放宽"了
```

### 6.7 学习提示

高级类型不需要一次全部掌握。**先学会熟练使用工具类型（`Partial`/`Pick`/`Omit`/`Record`/`ReturnType`），再逐步理解如何自己写。** 遇到复杂类型时，多用鼠标悬停查看推断结果，也可以把中间步骤拆成多个 `type` 逐步验证。

### ✍️ Day 6 练习

1. 手写 `MyPick<T, K extends keyof T>` 与 `MyOmit`（提示：`Exclude<keyof T, K>`）。
2. 手写 `DeepReadonly<T>`（递归映射类型）。
3. 定义 `const ROLES = ["admin", "editor", "viewer"] as const`，导出 `Role` 类型，并生成 `Record<Role, string[]>` 的权限表。
4. 用模板字面量类型写出 `type CssUnit = `${number}px` | `${number}rem``，并测试哪些字符串合法。

---

## Day 7：工程实践与综合项目

### 学习目标

- 掌握异步代码与错误处理的类型写法
- 理解 `tsconfig` 关键选项
- 了解"运行时校验"与类型系统的边界
- 完成一个综合小项目，巩固前六天内容

### 7.1 异步与 `Promise`

```ts
interface User {
  id: number;
  name: string;
}

async function fetchUser(id: number): Promise<User> {
  const res = await fetch(`https://api.example.com/users/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return (await res.json()) as User; // ⚠️ 这里只是断言，并不会真正校验数据
}

// 并发
async function loadAll(ids: number[]): Promise<User[]> {
  return Promise.all(ids.map(fetchUser));
}

// Promise.allSettled 的结果也是可辨识联合
async function safeLoad(ids: number[]) {
  const results = await Promise.allSettled(ids.map(fetchUser));
  for (const r of results) {
    if (r.status === "fulfilled") console.log(r.value.name);
    else console.error(r.reason);
  }
}
```

### 7.2 错误处理

```ts
try {
  // ...
} catch (e: unknown) {
  // 严格模式下 catch 变量是 unknown，必须先收窄
  if (e instanceof Error) {
    console.error(e.message);
  } else {
    console.error("未知错误", e);
  }
}

// 用返回值代替异常（Result 模式）
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

function parseJson(text: string): Result<unknown> {
  try {
    return { ok: true, value: JSON.parse(text) };
  } catch (e) {
    return { ok: false, error: e instanceof Error ? e : new Error(String(e)) };
  }
}
```

### 7.3 类型 ≠ 运行时校验（重要）

TypeScript 的类型在运行时**完全不存在**。来自网络、文件、用户输入、`JSON.parse` 的数据，即使你写了 `as User`，也可能是错的。

- 处理外部数据：先当 `unknown`，再**校验**后使用。
- 可以手写守卫，或使用运行时校验库（如 **Zod**、Valibot 等）从校验规则**反推类型**，做到"一份定义、两处受益"：

```ts
import { z } from "zod";

const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
});

type User = z.infer<typeof UserSchema>; // 类型自动生成

const data: unknown = JSON.parse('{"id":1,"name":"Alice"}');
const user = UserSchema.parse(data); // 校验失败会抛错；成功则得到类型安全的 User
```

（需要先 `npm i zod`。）

### 7.4 `tsconfig` 关键选项速览

| 选项 | 作用 | 建议 |
|------|------|------|
| `strict` | 开启一组严格检查（`strictNullChecks`、`noImplicitAny` 等） | **必开** |
| `noUncheckedIndexedAccess` | 索引访问结果带 `undefined` | 推荐 |
| `noImplicitReturns` | 函数所有分支都要有返回 | 推荐 |
| `noFallthroughCasesInSwitch` | 禁止 `switch` 意外穿透 | 推荐 |
| `exactOptionalPropertyTypes` | 区分"缺失"与"显式 undefined" | 进阶 |
| `target` | 输出的 JS 语法版本 | 现代环境用 `ES2022` 及以上 |
| `module` / `moduleResolution` | 模块系统与解析方式 | 按运行环境选择（Node / 打包器） |
| `paths` | 路径别名 | 需要打包器/运行时同步配置 |
| `skipLibCheck` | 跳过 `.d.ts` 检查 | 通常开启，加快速度 |

> 不同项目类型（Node 服务、Vite 前端、库开发）的 `tsconfig` 差异较大，**优先使用对应框架/模板生成的配置**，再按需微调。

### 7.5 常用工具链

- **ESLint + typescript-eslint**：代码质量与风格检查
- **Prettier**：格式化
- **Vitest / Jest**：测试（都有很好的 TS 支持）
- **Vite / esbuild / tsup**：构建与打包
- **tsx**：开发时直接运行 TS
- 较新的 Node.js 版本已支持直接运行 `.ts`（类型剥离），具体版本与限制请查阅 Node.js 官方文档

### 7.6 综合项目：内存版 Todo 管理

目标：整合泛型、联合类型、工具类型、类与模块。

**`src/todo.ts`**

```ts
export type Priority = "low" | "medium" | "high";

export interface Todo {
  id: number;
  title: string;
  done: boolean;
  priority: Priority;
  createdAt: Date;
}

// 创建时只需要 title，priority 可选
export type NewTodo = Pick<Todo, "title"> & Partial<Pick<Todo, "priority">>;

export interface TodoFilter {
  done?: boolean;
  priority?: Priority;
}
```

**`src/repository.ts`**

```ts
// 通用的内存仓库：任何带 id 的实体都能用
export class Repository<T extends { id: number }> {
  private items = new Map<number, T>();

  add(item: T): void {
    this.items.set(item.id, item);
  }

  get(id: number): T | undefined {
    return this.items.get(id);
  }

  update(id: number, patch: Partial<Omit<T, "id">>): T | undefined {
    const current = this.items.get(id);
    if (!current) return undefined;
    const next: T = { ...current, ...patch };
    this.items.set(id, next);
    return next;
  }

  remove(id: number): boolean {
    return this.items.delete(id);
  }

  list(predicate?: (item: T) => boolean): T[] {
    const all = [...this.items.values()];
    return predicate ? all.filter(predicate) : all;
  }
}
```

**`src/service.ts`**

```ts
import { Repository } from "./repository";
import type { NewTodo, Todo, TodoFilter } from "./todo";

export class TodoService {
  private repo = new Repository<Todo>();
  private nextId = 1;

  add(input: NewTodo): Todo {
    const todo: Todo = {
      id: this.nextId++,
      title: input.title,
      done: false,
      priority: input.priority ?? "medium",
      createdAt: new Date(),
    };
    this.repo.add(todo);
    return todo;
  }

  complete(id: number): Todo | undefined {
    return this.repo.update(id, { done: true });
  }

  remove(id: number): boolean {
    return this.repo.remove(id);
  }

  list(filter: TodoFilter = {}): Todo[] {
    return this.repo.list(
      (t) =>
        (filter.done === undefined || t.done === filter.done) &&
        (filter.priority === undefined || t.priority === filter.priority),
    );
  }
}
```

**`src/index.ts`**

```ts
import { TodoService } from "./service";

const service = new TodoService();

service.add({ title: "学习 TypeScript 基础类型", priority: "high" });
service.add({ title: "完成 Day 4 泛型练习" });
service.add({ title: "阅读 tsconfig 文档", priority: "low" });

service.complete(1);

console.log("未完成：", service.list({ done: false }));
console.log("高优先级：", service.list({ priority: "high" }));
```

运行：

```bash
npx tsx src/index.ts
npx tsc --noEmit
```

**挑战任务（选做）：**

1. 增加 `searchByTitle(keyword: string): Todo[]`。
2. 给 `TodoService` 增加把数据保存到 JSON 文件的功能，读取时用类型守卫或 Zod 校验。
3. 把 `Priority` 改为 `as const` 数组反推类型，并按优先级排序。
4. 用可辨识联合为操作结果建模：`{ ok: true; todo } | { ok: false; reason: "NOT_FOUND" }`。
5. 使用 Vitest 为 `Repository` 编写单元测试。

---

## 附录 A：常见报错速查

| 报错信息（大意） | 常见原因 | 解决思路 |
|------------------|----------|----------|
| `Type 'X' is not assignable to type 'Y'` | 类型不匹配 | 检查赋值两端类型；悬停查看推断结果 |
| `Object is possibly 'undefined'` / `'null'` | 严格空值检查 | 用 `?.`、`??`、`if` 判断收窄，避免滥用 `!` |
| `Property 'x' does not exist on type 'Y'` | 属性不存在或类型未收窄 | 先收窄类型或补全类型定义 |
| `Parameter 'x' implicitly has an 'any' type` | 缺少参数类型（`noImplicitAny`） | 给参数加类型注解 |
| `Cannot find module 'xxx' or its type declarations` | 缺少库或类型 | 安装库 / `@types/xxx` / 写 `.d.ts` |
| `Argument of type 'string' is not assignable to parameter of type '"a" \| "b"'` | 字面量被拓宽为 string | 用 `as const`、显式类型注解或 `satisfies` |
| `'x' is declared but its value is never read` | 未使用变量 | 删除，或以 `_` 开头表示故意未使用 |
| `Type instantiation is excessively deep` | 类型递归过深 | 简化类型，拆分中间类型 |

**读报错的技巧：** TypeScript 的错误信息通常从**下往上**读更容易看到根本原因（最后一行往往是最具体的差异）。

---

## 附录 B：速查表

```ts
// 基础
let a: string | number;
let list: number[] = [];
let tuple: [string, number];
let fn: (x: number) => string;

// 对象
interface I { readonly id: number; name?: string }
type T = { a: 1 } & { b: 2 };
type U = "a" | "b";

// 泛型
function f<T extends object, K extends keyof T>(o: T, k: K): T[K] { return o[k]; }

// 收窄
typeof x === "string"; x instanceof Date; "key" in obj;
function isFoo(x: unknown): x is Foo { /* ... */ }

// 工具类型
Partial<T>  Required<T>  Readonly<T>  Pick<T, K>  Omit<T, K>
Record<K, V>  Exclude<T, U>  Extract<T, U>  NonNullable<T>
ReturnType<F>  Parameters<F>  Awaited<T>

// 类型运算
keyof T   typeof value   T[K]   T extends U ? X : Y   infer R
```

---

## 附录 C：学习资源与后续路线

**官方与权威资源**

- TypeScript 官方文档与 Handbook：<https://www.typescriptlang.org/docs/>
- TypeScript Playground（在线练习，可查看编译结果）：<https://www.typescriptlang.org/play>
- 《Effective TypeScript》（Dan Vanderkam）：进阶最佳实践
- Type Challenges（类型体操练习）：<https://github.com/type-challenges/type-challenges>
- Total TypeScript（Matt Pocock）：系统化教程与技巧

**后续路线建议**

1. **前端方向**：React + TypeScript（组件 Props、Hooks 类型）、Vue 3 + TS、Next.js
2. **后端方向**：Node.js + Express / Fastify / NestJS，配合 Zod 做入参校验，Prisma / Drizzle 做数据库类型安全访问
3. **全栈类型安全**：tRPC、OpenAPI 代码生成，让前后端共享类型
4. **进阶类型系统**：递归类型、类型体操、品牌类型（Branded Types）、装饰器
5. **库开发**：`d.ts` 生成、`exports` 字段、双格式（ESM/CJS）发布

---

## 附录 D：七天自测清单

学完后，逐条自问，能不看资料说清楚就算掌握：

**基础**
- [ ] 类型推断什么时候可以省略注解？
- [ ] `any` 与 `unknown` 的区别？为什么外部数据优先用 `unknown`？
- [ ] 为什么推荐用字面量联合类型代替 `enum`？

**类型建模**
- [ ] `interface` 与 `type` 的差异与选择原则？
- [ ] 联合类型与交叉类型分别适合什么场景？
- [ ] 什么是可辨识联合？如何做穷尽检查？

**泛型与高级类型**
- [ ] 泛型解决了什么问题？`extends` 约束有什么用？
- [ ] `keyof`、`typeof`、`T[K]` 各做什么？
- [ ] 能不看资料写出 `Partial`、`Pick`、`ReturnType` 的实现思路吗？

**工程**
- [ ] 为什么一定要开启 `strict`？
- [ ] 为什么 `as User` 不等于校验？运行时数据该怎么处理？
- [ ] 遇到没有类型的第三方库时，有哪些处理办法？

---

> **最后的建议：** 类型系统的学习曲线在"泛型 + 高级类型"处最陡，遇到困难很正常。
> 每天写代码、多读报错、多看别人的类型定义（比如在 VS Code 里 `F12` 跳到库的 `.d.ts`），一两周内你就会明显感到"编译器在帮你写代码"。
>
> 祝学习顺利！🚀
