# Zod 4 中文教程

> 本文档基于 [zod.dev](https://zod.dev) 官方文档（Zod 4 / `zod@4.x`）整理编写，面向中文读者，按"从入门到实战"的顺序重排内容，补充了说明与心法。
>
> - 原文来源：https://zod.dev/packages/zod 、https://zod.dev/api 、https://zod.dev/basics 等
> - 整理时间：2026-10
> - 代码示例默认使用常规版 Zod（`import * as z from "zod"`），涉及 Zod Mini 的地方会单独标注

---

## 目录

1. [Zod 是什么](#1-zod-是什么)
2. [安装与工程配置](#2-安装与工程配置)
3. [快速上手：定义、解析、推断、报错](#3-快速上手定义解析推断报错)
4. [定义 Schema 全览](#4-定义-schema-全览)
5. [校验与变换：refine / transform / default / brand](#5-校验与变换)
6. [双向编解码：Codecs](#6-双向编解码codecs)
7. [错误处理与国际化](#7-错误处理与国际化)
8. [元数据与注册表](#8-元数据与注册表)
9. [JSON Schema 互转](#9-json-schema-互转)
10. [性能：AOT 编译](#10-性能aot-编译)
11. [包结构与选型：zod / zod-mini / zod-v4-core](#11-包结构与选型)
12. [给库作者：peer 依赖与子路径](#12-给库作者peer-依赖与子路径)
13. [从 Zod 3 迁移到 Zod 4](#13-从-zod-3-迁移到-zod-4)
14. [API 速查表](#14-api-速查表)
15. [常见坑与最佳实践](#15-常见坑与最佳实践)

---

## 1. Zod 是什么

Zod 是一个 **TypeScript 优先（TypeScript-first）的 schema 声明与校验库**，核心理念是：**一份 schema，既是运行时校验器，也是静态类型来源**。

```ts
import * as z from "zod";

const User = z.object({
  name: z.string(),
});

// 不可信的外部数据
const input: unknown = { name: "billie" };

// 解析结果既通过运行时校验，也带有静态类型
const data = User.parse(input);
console.log(data.name); // string
```

**核心特性：**

- 零外部依赖
- 同时支持 Node.js 与所有现代浏览器
- 体积极小（核心包 gzip 约 2kb）
- **不可变 API**：所有方法都返回新实例，schema 可以安全复用
- 链式调用，自动补全友好
- TypeScript 与纯 JS 均可用
- 内置 JSON Schema 互转
- 生态庞大（tRPC、React Hook Form、Prisma 生态等）

Zod 4 的 schema API 目标是与 TypeScript 的类型系统 **一一对应**，所以你会看到 `z.string()`、`z.union([...])`、`z.discriminatedUnion()` 这些与 TS 概念同名的 API。

---

## 2. 安装与工程配置

```sh
npm install zod
```

也发布了 JSR 版本：`@zod/zod`。

**环境要求：**

- TypeScript **5.5+**（更低版本可能可用，但不受官方支持）
- 必须在 `tsconfig.json` 中开启 `strict` 模式：

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

Zod 官方还提供了 [llms.txt](https://zod.dev/llms.txt)（供 AI 读取的文档索引）和一个可搜索文档的 MCP server。

**导入方式（Zod 4）：**

```ts
import * as z from "zod";        // 常规版（推荐绝大多数项目）
import * as z from "zod/mini";   // 极致体积的 tree-shakable 版
import * as z from "zod/v4/core"; // 底层核心，给库作者用
```

---

## 3. 快速上手：定义、解析、推断、报错

### 3.1 定义 schema

```ts
import * as z from "zod";

const Player = z.object({
  username: z.string(),
  xp: z.number(),
});
```

### 3.2 解析数据：`.parse()`

传入任意输入，校验通过则返回一份 **强类型的深拷贝**：

```ts
Player.parse({ username: "billie", xp: 100 });
// => { username: "billie", xp: 100 }
```

> 如果 schema 中使用了异步 API（异步 `refine` / `transform`），必须改用 `.parseAsync()`。

### 3.3 处理错误：`try/catch` 或 `.safeParse()`

校验失败时 `.parse()` 会抛出 `ZodError`，里面包含非常细粒度的 `issues`：

```ts
try {
  Player.parse({ username: 42, xp: "100" });
} catch (error) {
  if (error instanceof z.ZodError) {
    error.issues;
    /* [
      {
        expected: 'string',
        code: 'invalid_type',
        path: [ 'username' ],
        message: 'Invalid input: expected string, received number'
      },
      {
        expected: 'number',
        code: 'invalid_type',
        path: [ 'xp' ],
        message: 'Invalid input: expected number, received string'
      }
    ] */
  }
}
```

不想写 `try/catch` 就用 `.safeParse()`，返回值是一个可辨识联合（discriminated union）：

```ts
const result = Player.safeParse({ username: 42, xp: "100" });

if (!result.success) {
  result.error;  // ZodError
} else {
  result.data;   // { username: string; xp: number }
}
```

### 3.4 只想知道"能不能过"：`.validate()`

只关心布尔结果时，`.validate()` 比 `.safeParse().success` 快得多（官方说明在非法输入下最多快 16 倍），而且不会构造错误对象：

```ts
Player.validate({ username: "billie", xp: 100 }); // true
Player.validate({ username: 42, xp: "100" });     // false
```

异步 schema 用 `.validateAsync()`；对已编译的 schema，`.validate()` 直接走编译后的快路径。

### 3.5 推断类型：`z.infer<>`

```ts
const Player = z.object({
  username: z.string(),
  xp: z.number(),
});

type Player = z.infer<typeof Player>;
// => { username: string; xp: number }

const player: Player = { username: "billie", xp: 100 };
```

当输入类型与输出类型不同时（例如用了 `.transform()`），可以分别取：

```ts
const mySchema = z.string().transform((val) => val.length);

type MySchemaIn = z.input<typeof mySchema>;   // string
type MySchemaOut = z.output<typeof mySchema>; // number（等价于 z.infer）
```

---

## 4. 定义 Schema 全览

### 4.1 原始类型与类型强制转换

```ts
z.string();
z.number();
z.bigint();
z.boolean();
z.symbol();
z.undefined();
z.null();
```

需要把输入"强制转换"成目标类型时，用 `z.coerce`：

```ts
z.coerce.string(); // String(input)
z.coerce.number(); // Number(input)
z.coerce.boolean();// Boolean(input)
z.coerce.bigint(); // BigInt(input)
z.coerce.date();   // new Date(input)
```

```ts
const schema = z.coerce.string();
schema.parse(42);   // "42"
schema.parse(true); // "true"
schema.parse(null); // "null"
```

> ⚠️ `z.coerce.boolean()` 用的是 JS 的真理值判断，`"false"`、`"0"`、`[]` 都会变成 `true`。想精确控制请用 `z.stringbool()`、`.transform()` 或 `z.pipe()`。

`z.coerce.*` 的**输入类型默认是 `unknown`**，可用泛型收窄：

```ts
const A = z.coerce.number();          // z.input<typeof A> => unknown
const B = z.coerce.number<number>();  // z.input<typeof B> => number
```

### 4.2 字面量、枚举、字符串布尔

**字面量：**

```ts
const tuna = z.literal("tuna");
const twelve = z.literal(12);
const twobig = z.literal(2n);
const tru = z.literal(true);

// 多个字面量的简写
const colors = z.literal(["red", "green", "blue"]);
colors.parse("green"); // ✅
colors.parse("yellow"); // ❌

colors.values; // Set<"red" | "green" | "blue">（仅常规版 Zod）
```

`z.null()` / `z.undefined()`（`z.void()` 等价于 `z.undefined()`）。

**枚举：**

```ts
const FishEnum = z.enum(["Salmon", "Tuna", "Trout"]);

FishEnum.parse("Salmon");  // ✅
FishEnum.parse("Swordfish"); // ❌

FishEnum.enum; // { Salmon: "Salmon", Tuna: "Tuna", Trout: "Trout" }
```

> ⚠️ 如果先把数组存进变量，Zod 推断不出字面量类型，必须 `as const` 或直接把数组写进 `z.enum()` 参数里。

也支持枚举对象、TypeScript `enum`：

```ts
const Fish = { Salmon: 0, Tuna: 1 } as const;
const FishEnum = z.enum(Fish);   // 也支持数字枚举值
FishEnum.parse(0); // ✅

enum Fish2 { Salmon = "Salmon" }
const FishEnum2 = z.enum(Fish2); // z.nativeEnum() 已废弃，统一用 z.enum()
```

派生新枚举：

```ts
const FishEnum = z.enum(["Salmon", "Tuna", "Trout"]);
const TunaOnly = FishEnum.exclude(["Salmon", "Trout"]);
const SalmonAndTroutOnly = FishEnum.extract(["Salmon", "Trout"]);
```

**字符串布尔（解析环境变量神器）：**

```ts
const strbool = z.stringbool();

strbool.parse("true");  // true
strbool.parse("1");     // true
strbool.parse("yes");   // true
strbool.parse("on");    // true
strbool.parse("enabled"); // true
strbool.parse("false"); // false
strbool.parse("0");     // false
strbool.parse("off");   // false
// 其他值一律 ZodError
```

自定义真假值集合与大小写：

```ts
z.stringbool({
  truthy: ["true", "1", "yes", "on", "y", "enabled"],
  falsy: ["false", "0", "no", "off", "n", "disabled"],
  case: "sensitive", // 默认 insensitive
});
```

### 4.3 字符串

**校验：**

```ts
z.string().max(5);
z.string().min(5);
z.string().length(5);
z.string().nonempty();          // 等价 .min(1)
z.string().regex(/^[a-z]+$/);
z.string().startsWith("aaa");
z.string().endsWith("zzz");
z.string().includes("---");
z.string().uppercase();         // 必须全大写
z.string().lowercase();         // 必须全小写
```

> 长度按 **Unicode 码点** 计算，不是 UTF-16 单元。`"😀"` 长度是 1，`"e\u0301"` 长度是 2，带 ZWJ 的表情（如 `🧑‍🍼`）是 3。

**变换：**

```ts
z.string().trim();          // 去首尾空白
z.string().toLowerCase();
z.string().toUpperCase();
z.string().normalize();     // Unicode 归一化
```

**内置格式校验：**

```ts
z.email();
z.uuid();              // 可指定版本：z.uuid({ version: "v4" })
z.uuidv4(); z.uuidv6(); z.uuidv7();
z.guid();
z.url();
z.httpUrl();           // 仅 http/https
z.hostname();
z.e164();              // E.164 手机号
z.emoji();
z.base64(); z.base64url(); z.hex();
z.jwt();
z.nanoid(); z.cuid(); z.cuid2(); z.ulid();
z.ipv4(); z.ipv6(); z.mac();
z.cidrv4(); z.cidrv6();
z.creditCard();        // Luhn 校验
z.currencyCode();      // ISO 4217
z.iban();              // ISO 7064 mod-97
z.hash("sha256");      // sha1 / sha384 / sha512 / md5
z.iso.date(); z.iso.time(); z.iso.datetime(); z.iso.duration();
```

**邮箱的可选正则：**

```ts
z.email();                              // 默认（接近 Gmail 规则）
z.email({ pattern: z.regexes.html5Email });  // 浏览器 input[type=email] 规则
z.email({ pattern: z.regexes.rfc5322Email });// RFC 5322
z.email({ pattern: z.regexes.unicodeEmail });// 宽松，支持国际化邮箱
```

**URL 细节：**

```ts
z.url({ hostname: /^example\.com$/ });  // 限定域名
z.url({ protocol: /^https$/ });         // 限定协议
```

**自定义格式：** 用 `z.string().check(z.regex(...))` 或直接把正则做成可复用的 check。

**模板字面量类型：**

```ts
const schema = z.templateLiteral([ "hello, ", z.string(), "!" ]);
// 类型：`hello, ${string}!`

z.templateLiteral([ "high", z.literal(5) ]);              // `high5`
z.templateLiteral([ z.number(), z.enum(["px","em","rem"]) ]);
// `${number}px` | `${number}em` | `${number}rem`
```

### 4.4 数字、整数、BigInt、布尔、日期

```ts
z.number().gt(5);
z.number().gte(5);        // 别名 .min(5)
z.number().lt(5);
z.number().lte(5);        // 别名 .max(5)
z.number().positive();    // .gt(0)
z.number().nonnegative();
z.number().negative();
z.number().nonpositive();
z.number().multipleOf(5); // 别名 .step(5)
```

- `z.number()` 只接受**有限数**：`NaN`、`Infinity` 会被拒绝
- `z.nan()` 专门校验 `NaN`
- `z.int()` 限定安全整数范围；`z.int32()` 限定 int32 范围；`z.int64()` 是 BigInt
- `z.bigint()` 支持同样的 `gt/gte/lt/lte/positive/multipleOf` 等
- `z.boolean()`, `z.date()`（校验 `Date` 实例且有效）

### 4.5 可选性：optional / nullable / nullish / unknown / never

```ts
z.string().optional();      // string | undefined
z.string().nullable();      // string | null
z.string().nullish();       // string | null | undefined
z.string().nonoptional();   // 从 optional 中"剥掉" undefined
z.unknown();                // 任意值
z.never();                  // 什么都不匹配
z.any();                    // 任意值（不推断类型，尽量少用）
```

### 4.6 对象（重点）

```ts
const Person = z.object({
  name: z.string(),
  age: z.number(),
});
// 默认所有字段必填；未知字段会被"剥离"（strip）
```

**未知字段的三种策略：**

```ts
z.object({...})        // strip：静默丢弃未知字段（默认）
z.strictObject({...})  // strict：出现未知字段就报错
z.looseObject({...})   // loose：原样保留未知字段
```

```ts
const Dog = z.object({ name: z.string() });
Dog.parse({ name: "Yeller", extraKey: true });
// => { name: "Yeller" }

z.looseObject({ name: z.string() }).parse({ name: "Yeller", extraKey: true });
// => { name: "Yeller", extraKey: true }
```

**catchall：** 用指定 schema 校验所有未知字段：

```ts
const DogWithStrings = z.object({
  name: z.string(),
  age: z.number().optional(),
}).catchall(z.string());

DogWithStrings.parse({ name: "Yeller", extraKey: "extraValue" }); // ✅
DogWithStrings.parse({ name: "Yeller", extraKey: 42 });           // ❌
```

**读取与改造：**

```ts
Dog.shape.name;      // 拿到内部 schema

Dog.keyof();          // "name" | "age" 的枚举 schema

const Base = z.object({ name: z.string() });
const Extended = Base.extend({ age: z.number() });

// safeExtend：只在"类型兼容"时才覆盖已有字段，避免类型被悄悄改变
const Safe = Base.safeExtend({ name: z.string().min(1) });

const Picked = Extended.pick({ name: true });
const Omitted = Extended.omit({ age: true });

Extended.partial();       // 全部变可选（可为 undefined）
Extended.exactPartial();  // 全部变可选，但不会引入 undefined 到输出类型
Extended.required();      // 全部变必填

z.deepPartial(schema);    // 深度可选（嵌套对象也变可选）
```

**Symbol 键：** `z.object({ [sym]: z.string() })` 支持，输出类型为 `{ [sym]: string }`。

**递归对象：** 用 getter 延迟求值：

```ts
const Category = z.object({
  name: z.string(),
  get subcategories() {
    return z.array(Category);
  },
});

type Category = z.infer<typeof Category>;
// { name: string; subcategories: Category[] }
```

循环引用输入开箱即用（输出图镜像输入图）：

```ts
const input: any = { name: "root", subcategories: [] };
input.subcategories.push(input);
const result = Category.parse(input);
result.subcategories[0] === result; // true
```

> Zod Mini 出于体积考虑，需要先注册 memoizer：`z.config({ memoizer: z.memoizer() })`。

**互递归：**

```ts
const User = z.object({
  email: z.email(),
  get posts() { return z.array(Post); },
});

const Post = z.object({
  title: z.string(),
  get author() { return User; },
});
```

**循环类型报错（TS 7023）的解法：** 给 getter 显式标注返回类型：

```ts
const Activity = z.object({
  name: z.string(),
  get subactivities(): z.ZodNullable<z.ZodArray<typeof Activity>> {
    return z.nullable(z.array(Activity));
  },
});
```

### 4.7 数组、元组、Record、Map、Set

```ts
z.array(z.string());
z.array(z.string()).min(2).max(10).length(3).nonempty();

z.tuple([z.string(), z.number()]);
z.tuple([z.string(), z.number()]).rest(z.boolean()); // 变长部分

z.record(z.string(), z.string());     // Record<string, string>
z.record(z.enum(["id","name"]), z.string()); // { id: string; name: string }
z.partialRecord(z.enum(["id","name"]), z.string()); // 全部可选
z.looseRecord(z.string().regex(/_phone$/), z.e164()); // 放行不匹配的键

z.map(z.string(), z.number());
z.set(z.string());
z.set(z.string()).min(1).max(10).size(3);
```

Record 的补充说明：

- key 的 schema 必须可赋值给 `string | number | symbol`
- 用 `z.number()` 做 key 时，会校验"数字字符串"（`"1.5"`、`"-3"` 通过，`"abc"` 失败），并继续应用数值约束
- 用 `z.enum()` 做 key 时，Zod 会**穷尽检查**所有枚举值都必须出现（与 TS 的 `Record` 一致）；想要部分键就用 `z.partialRecord()`
- 值的 schema 若是 `.default()` / `.prefault()` / `.optional()`，对应 key 在输入侧也变可选

### 4.8 联合、交叉、判别联合、XOR

```ts
// 普通联合：按顺序尝试，第一个通过的胜出
const stringOrNumber = z.union([z.string(), z.number()]);
stringOrNumber.options; // [ZodString, ZodNumber]

// 交叉：A & B
const EmployedPerson = z.intersection(Person, Employee);

// 判别联合：靠"判别键"直接定位分支，大联合下性能更好
const MyResult = z.discriminatedUnion("status", [
  z.object({ status: z.literal("success"), data: z.string() }),
  z.object({ status: z.literal("failed"), error: z.string() }),
]);

// 取某个分支的 schema（返回原 schema，.shape 等方法仍可用）
z.getDiscriminatedOption(MyResult, "success");

// 异或：必须"恰好一个"分支匹配
const schema = z.xor([z.string(), z.number()]);
schema.parse("hello"); // ✅
schema.parse(true);    // ❌ 零个匹配

// 判别联合可嵌套，Zod 会自动选择最优解析策略
```

> ⚠️ 合并两个对象 schema 时，优先用 `A.extend(B)` 而不是 `z.intersection(A, B)`：后者返回 `ZodIntersection`，没有 `pick` / `omit` 等对象方法。

> ⚠️ `z.xor()` 与 `z.object()` 一起用时容易"两个分支都匹配"，因为 object 默认 strip 未知键。对更窄的分支加 `.strict()` 即可互斥：
> ```ts
> z.xor([name, version]).parse({ name: "zod", version: "4" });          // ❌ 两个都匹配
> z.xor([name.strict(), version]).parse({ name: "zod", version: "4" }); // ✅
> ```

### 4.9 其它类型

```ts
z.file();                         // File 实例
z.file().min(10_000).max(1_000_000).mime("image/png");

z.instanceof(MyClass);
z.json();                         // 任意 JSON 可编码值
z.custom<Decimal>((v) => Decimal.isDecimal(v)); // 第三方类型

z.function({
  input: [z.string()],
  output: z.number(),
}).implement((input) => input.trim().length);

z.templateLiteral(["id-", z.string()]);
```

`z.promise()` 已废弃——真要处理 Promise，先 `await` 再解析。

---

## 5. 校验与变换

### 5.1 `.refine()`：自定义校验

```ts
const myString = z.string().refine((val) => val.length <= 255);

// 自定义错误
z.string().refine((val) => val.length > 8, { error: "太短了！" });

// 错误信息可以是函数，拿到 issue
z.string().refine((val) => val.length > 8, {
  error: (iss) => `太短了："${iss.input}"`,
});
```

> ⚠️ 校验函数**永远不要抛异常**，应返回假值表示失败；Zod 不会捕获你抛出的错误。

**abort：** 默认所有校验都会跑完，尽可能多地报错。加 `abort: true` 让某个校验失败后立即终止：

```ts
const myString = z.string()
  .refine((val) => val.length > 8, { error: "太短了！", abort: true })
  .refine((val) => val === val.toLowerCase(), { error: "必须小写", abort: true });
```

### 5.2 `.check()`：更底层的校验

`.refine()` 是 `.check()` 的语法糖。需要一条校验里产出多个 issue，或完全控制 issue 对象时用它：

```ts
const UniqueStringArray = z.array(z.string()).check((ctx) => {
  if (ctx.value.length > 3) {
    ctx.issues.push({
      code: "too_big",
      maximum: 3,
      origin: "array",
      inclusive: true,
      message: "元素太多了 😡",
      input: ctx.value,
    });
  }

  if (ctx.value.length !== new Set(ctx.value).size) {
    ctx.issues.push({
      code: "custom",
      message: "不允许重复",
      input: ctx.value,
      continue: true, // 让这个 issue 可继续（默认 false）
    });
  }
});
```

> `.superRefine()` 已废弃，等价于 `.check()`。

### 5.3 `.transform()` 与 `.pipe()`

```ts
// 单向变换
const stringToLength = z.string().transform((val) => val.length);
stringToLength.parse("hello"); // 5

// 管道：前一个 schema 的输出喂给后一个 schema
const schema = z.string()
  .transform((val) => val.length)
  .pipe(z.number().min(3));
```

### 5.4 `z.preprocess()`

先把输入洗干净，再交给 schema：

```ts
const coercedInt = z.preprocess((val) => {
  if (typeof val === "string") return Number.parseInt(val);
  return val;
}, z.int());
```

输入类型默认 `unknown`；想收窄就在参数上直接标注：

```ts
const trimmed = z.preprocess(
  (val: string | null | undefined) => val?.trim() ?? "",
  z.string()
);
type Input = z.input<typeof trimmed>;   // string | null | undefined
```

### 5.5 `default` vs `prefault`

```ts
// default：直接短路，返回默认值（不再解析），因此必须是"输出类型"
const a = z.string().transform((v) => v.length).default(0);
a.parse(undefined); // 0

// prefault："预解析默认值"，会继续走解析管道，因此必须是"输入类型"
const b = z.string().transform((v) => v.length).prefault("tuna");
b.parse(undefined); // 4
```

```ts
// 默认值也可以是函数，每次需要时重新生成
const randomDefault = z.number().default(Math.random);

// 组合变换时区别很明显
z.string().trim().toUpperCase().prefault("  tuna  ").parse(undefined); // "TUNA"
z.string().trim().toUpperCase().default("  tuna  ").parse(undefined);  // "  tuna  "
```

### 5.6 `.catch()`：失败兜底

```ts
const numberWithCatch = z.number().catch(42);
numberWithCatch.parse(5);      // 5
numberWithCatch.parse("tuna"); // 42

// 兜底值可以是函数，能拿到错误
z.number().catch((ctx) => {
  ctx.error;   // 捕获到的 ZodError
  return Math.random();
});
```

### 5.7 品牌类型 `.brand()`

TS 是结构化类型系统，`{ name: string }` 与 `{ name: string }` 完全等价。品牌类型用来模拟"名义类型"：

```ts
const Cat = z.object({ name: z.string() }).brand<"Cat">();
const Dog = z.object({ name: z.string() }).brand<"Dog">();

type Cat = z.infer<typeof Cat>; // { name: string } & z.$brand<"Cat">
type Dog = z.infer<typeof Dog>; // { name: string } & z.$brand<"Dog">

const pluto = Dog.parse({ name: "pluto" });
const simba: Cat = pluto; // ❌ 类型不兼容
```

- 品牌只影响静态类型，不影响运行时结果
- 默认只给 **输出类型** 打品牌；可指定方向：

```ts
z.string().brand<"Cat", "out">();   // 默认
z.string().brand<"Cat", "in">();
z.string().brand<"Cat", "inout">();
```

### 5.8 只读与工具方法

```ts
z.object({ name: z.string() }).readonly(); // Readonly<...>

// 把外部函数接进链式调用
const schema = z.number()
  .apply((s: z.ZodNumber) => s.min(0).max(100))
  .nullable();

// 由目标类型反推 schema（类型必须"精确相等"，否则编译报错）
type Player = { username: string; xp: number };
const PlayerSchema = z.toZod<Player>()(
  z.object({ username: z.string(), xp: z.number() })
);
```

`z.toZod<T>()` 非常适合"类型先行"的场景：数据库模型、代码生成的 client 类型、别人维护的 interface。

---

## 6. 双向编解码：Codecs

> Zod 4.1 引入。常规 schema 只有"输入 → 输出"一个方向，**codec 同时定义了两个方向**。

所有 Zod schema 都支持两个方向：

- **正向（Forward）**：`Input → Output`，方法为 `.parse()` / `.decode()`
- **反向（Backward）**：`Output → Input`，方法为 `.encode()`

对大多数 schema 来说两者没有区别（输入输出同类型）。真正的双向价值体现在 codec 上：

```ts
const stringToDate = z.codec(
  z.iso.datetime(),  // 输入 schema：ISO 字符串
  z.date(),          // 输出 schema：Date 对象
  {
    decode: (isoString) => new Date(isoString), // ISO → Date
    encode: (date) => date.toISOString(),       // Date → ISO
  }
);
```

使用：

```ts
// 正向：parse 等价于 decode
stringToDate.parse("2024-01-15T10:30:00.000Z");  // => Date
z.decode(stringToDate, "2024-01-15T10:30:00.000Z"); // => Date（入参类型更强）

// 反向
z.encode(stringToDate, new Date("2024-01-15"));
// => "2024-01-15T00:00:00.000Z"

// 反转 codec（输入输出对调）
const dateToString = z.invertCodec(stringToDate);
z.decode(dateToString, new Date("2024-01-15")); // => string
```

**实例方法同样可用：**

```ts
stringToDate.decode(...);
stringToDate.decodeAsync(...);
stringToDate.safeDecode(...);       // { success, data } | { success: false, error }
stringToDate.safeDecodeAsync(...);
stringToDate.encode(...);
```

**异步 codec：**

```ts
const asyncCodec = z.codec(z.string(), z.number(), {
  decode: async (str) => Number(str),
  encode: async (num) => num.toString(),
});
```

### 6.1 编码时的一些细节

理解这些规则能避免"解码正常、编码报错"的困惑：

| 场景 | 行为 |
| --- | --- |
| Pipe | 解码时先 A 后 B；编码时先 B 后 A |
| 校验（refine/min/max） | **两个方向都会执行**；编码时先做一次类型检查，再跑校验逻辑 |
| `default` / `prefault` | 只在正向生效；`encode(undefined)` 会报错 |
| `catch` | 只在正向生效 |
| 可变异变换（`trim`、`toLowerCase`） | 两个方向都会执行 |
| `stringbool` | 编码时 `true` → `"true"` |

带 codec 的 schema 嵌套在对象里时，可以用 `z.input()` / `z.output()` 取对应一侧的运行时 schema：

```ts
const Event = z.object({ name: z.string(), at: stringToDate });

z.input(Event).parse({ name: "launch", at: "2024-01-01T00:00:00Z" }); // ✅
z.output(Event).parse({ name: "launch", at: new Date() });            // ✅
```

### 6.2 常用 codec（可复制）

官方文档提供了这些"拿来即用"的实现（不是内置导出，需自行复制）：

`stringToNumber`、`stringToInt`、`stringToBigInt`、`numberToBigInt`、`isoDatetimeToDate`、`epochSecondsToDate`、`epochMillisToDate`、`json(schema)`、`utf8ToBytes`、`bytesToUtf8`、`base64ToBytes`、`base64urlToBytes`、`hexToBytes`、`stringToURL`、`stringToHttpURL`、`uriComponent`、`stringToBoolean`。

典型用法：**HTTP 入参是字符串，内部用强类型**：

```ts
const Query = z.object({
  page: stringToInt,     // "2" → 2
  at: isoDatetimeToDate, // ISO 字符串 → Date
});
```

---

## 7. 错误处理与国际化

### 7.1 错误对象结构

校验错误是 `z.core.$ZodError` 的实例（`zod` 包里的 `ZodError` 是它的子类，多了些便利方法）。核心是 `issues` 数组：

```ts
const result = z.string().safeParse(12);
result.error.issues;
// [
//   {
//     expected: 'string',
//     code: 'invalid_type',
//     path: [],
//     message: 'Invalid input: expected string, received number'
//   }
// ]
```

每条 issue 都包含 `code`、`path`、`message`、`input` 等字段，`code` 是判别依据（`invalid_type`、`too_small`、`too_big`、`invalid_format`、`unrecognized_keys`、`invalid_union`、`custom`…）。

### 7.2 自定义错误消息：`error` 参数

几乎每个 API 都接受错误信息——可以直接传字符串：

```ts
z.string("不是字符串！");
z.string().min(5, "太短了！");
z.uuid("UUID 格式不对");
z.array(z.string()).min(5, "元素太少");
```

也可以传对象（推荐，可扩展）：

```ts
z.string({ error: "错误" });
z.string().min(5, { error: "太短了！" });
```

`error` 还可以是函数（即 **error map**），在解析时按需生成消息：

```ts
z.string({
  error: (iss) => iss.input === undefined ? "该字段必填" : "输入无效",
});
```

`iss` 上可用的信息：

```ts
z.string({
  error: (iss) => {
    iss.code;    // issue code
    iss.input;   // 输入值
    iss.inst;    // 触发该 issue 的 schema/check
    iss.schema;  // 拥有该 issue 的 schema
    iss.path;    // 错误路径
    // 某些 API 还会有 minimum、maximum、inclusive 等
    return "自定义消息";
  },
});
```

返回 `undefined` 表示"我不处理，交给下一个 error map / 默认消息"，便于只覆盖部分 code：

```ts
z.int64({
  error: (issue) => {
    if (issue.code === "too_big") return `值必须小于 ${issue.maximum}`;
    return undefined; // 其余走默认
  },
});
```

### 7.3 单次解析级别的自定义

```ts
schema.parse(12, { error: (iss) => "本次解析的错误" });
```

> ⚠️ Zod 4 中，**schema 级错误优先级高于 parse 级**（与 Zod 3 相反）。

各种 issue code 可以在 error map 里分支处理：

```ts
schema.safeParse(12, {
  error: (iss) => {
    if (iss.code === "invalid_type") return `类型错误，期望 ${iss.expected}`;
    if (iss.code === "too_small") return `最小值是 ${iss.minimum}`;
  },
});
```

### 7.4 错误优先级（从高到低）

1. **check 级错误** —— 定义在具体校验上的错误
   ```ts
   z.string().min(5, "太短了！");
   ```
2. **schema 级错误** —— 写死在 schema 定义里的错误（对自身所有 check 生效，是 check 无自定义错误时的兜底）
   ```ts
   z.string("名字无效").safeParse(12);          // "名字无效"
   z.string("名字无效").min(5).safeParse("ab"); // "名字无效"
   ```
3. **数组 / 对象元素自身的错误**（沿用元素 schema 定义）
4. **parse 级 error map**
5. **全局 `customError`**
6. **locale 默认消息**

### 7.5 国际化：内置中文

Zod 自带多语言错误消息，从 `zod/locales` 导入并 `z.config()`：

```ts
import * as z from "zod";
import { zhCN } from "zod/locales";

z.config(zhCN());
```

懒加载（按需异步加载语言包）：

```ts
async function loadLocale(locale: string) {
  const { default: localeFn } = await import(`zod/v4/locales/${locale}.js`);
  z.config(localeFn());
}
await loadLocale("zhCN");
```

> 常规 `zod` 会自动加载 `en`；**Zod Mini 不加载任何 locale**，所有消息默认是 `Invalid input`。

可用语言（部分）：`ar az be bg bn ca cs da de el en eo es fa fi fr frCA gu he hi hr hu hy id is it ja ka km kn ko lt mk ms ne nl nn no ota ps pl pt ptBR ro ru sk sl sv ta tg th tk tr uk ur uz vi zhCN zhTW yo`。

如果你想自己翻译，有两条路：

```ts
// 解析时翻译
z.string().min(5, { error: (iss) => t("too_short", iss) });

// 渲染时翻译（把 issue 的 code 当 key，其余字段当插值参数）
result.error?.issues.map((iss) => t(iss.code, iss));
```

### 7.6 格式化错误

```ts
const schema = z.strictObject({
  username: z.string(),
  favoriteNumbers: z.array(z.number()),
});

const result = schema.safeParse({
  username: 1234,
  favoriteNumbers: [1234, "4567"],
  extraKey: 1234,
});
// 三条 issue：username 类型错误、favoriteNumbers[1] 类型错误、extraKey 未识别
```

**转为人类可读文本 `z.prettifyError()`（最常用）：**

```ts
console.log(z.prettifyError(result.error));
/*
✖ Unrecognized key: "extraKey"
✖ Invalid input: expected string, received number
  → at username
✖ Invalid input: expected number, received string
  → at favoriteNumbers[1]
*/
```

**转为嵌套树 `z.treeifyError()`：**

```ts
const tree = z.treeifyError(result.error);
/*
{
  errors: [ 'Unrecognized key: "extraKey"' ],
  properties: {
    username: { errors: [ '...' ] },
    favoriteNumbers: {
      errors: [],
      items: [ undefined, { errors: [ '...' ] } ]
    }
  }
}
*/

tree.properties?.username?.errors;
tree.properties?.favoriteNumbers?.items?.[1]?.errors;
```

**转为扁平结构 `z.flattenError()`（适合表单）：**

```ts
const flattened = z.flattenError(result.error);
/*
{
  formErrors: [ 'Unrecognized key: "extraKey"' ],
  fieldErrors: {
    username: [ '...' ],
    favoriteNumbers: [ '...' ]
  }
}
*/

flattened.fieldErrors.username;
```

> `z.formatError()` 与 `error.format()` / `error.flatten()` 均已废弃（前者用 `treeifyError`，后者用 `flattenError`）。

**全局自定义错误 `z.config({ customError })`：**

```ts
z.config({
  customError: (iss) => {
    const meta = iss.schema && z.globalRegistry.get(iss.schema);
    return `${meta?.title ?? "字段"} 不合法`;
  },
});

z.string().min(5).meta({ title: "密码" }).safeParse("abc");
// => "密码 不合法"
```

---

## 8. 元数据与注册表

### 8.1 注册表（Registry）

注册表是"schema → 强类型元数据"的映射集合：

```ts
const myRegistry = z.registry<{ description: string }>();

const mySchema = z.string();
myRegistry.add(mySchema, { description: "一个很酷的 schema" });
myRegistry.has(mySchema);  // true
myRegistry.get(mySchema);  // { description: "一个很酷的 schema" }
myRegistry.remove(mySchema);
myRegistry.clear();
```

TypeScript 会强制元数据类型匹配：

```ts
myRegistry.add(mySchema, { description: 123 }); // ❌ 类型错误
```

**`.register()` 的特殊之处**：它返回 **原 schema 本身**（不是新实例），所以可以写在 schema 定义内部：

```ts
const mySchema = z.object({
  name: z.string().register(myRegistry, { description: "用户名" }),
  age: z.number().register(myRegistry, { description: "年龄" }),
});
```

> 注册表对 `id` 有特殊处理：同一个 `id` 注册多次会抛错。

### 8.2 全局注册表与 `.meta()` / `.describe()`

```ts
export interface GlobalMeta {
  id?: string;
  title?: string;
  description?: string;
  deprecated?: boolean;
  [k: string]: unknown;
}
```

最简用法：

```ts
const emailSchema = z.email().meta({
  id: "email_address",
  title: "邮箱",
  description: "请输入有效的邮箱地址",
});

// 等价于
z.email().register(z.globalRegistry, { id: "email_address", title: "邮箱" });

// 只有描述时更方便的写法
z.string().describe("用户名");
```

扩展 `GlobalMeta`（常用 `zod.d.ts` + 声明合并）：

```ts
declare module "zod" {
  interface GlobalMeta {
    examples?: unknown[];
  }
}
export {};
```

这些元数据在 `z.toJSONSchema()` 里会被自动写进 JSON Schema（`title` / `description` / `examples` / `deprecated`），带 `id` 的还会被提取成 `$defs`。

---

## 9. JSON Schema 互转

### 9.1 Zod → JSON Schema

```ts
const schema = z.object({
  name: z.string(),
  age: z.number(),
});

z.toJSONSchema(schema);
// {
//   type: 'object',
//   properties: { name: { type: 'string' }, age: { type: 'number' } },
//   required: [ 'name', 'age' ],
//   additionalProperties: false,
// }
```

**无法表达的类型**（默认抛错）：

```ts
z.bigint(); z.int64(); z.symbol(); z.undefined(); z.void();
z.date(); z.map(); z.set(); z.transform(); z.nan(); z.custom();
z.number().multipleOf(0);
```

**参数一览：**

```ts
z.toJSONSchema(schema, {
  target: "draft-2020-12", // "draft-07" | "draft-04" | "openapi-3.0"
  io: "output",            // "input"：按输入类型生成
  metadata: z.globalRegistry, // 元数据来源
  unrepresentable: "throw",// "throw" | "any" | 函数
  cycles: "ref",           // "ref"（用 $defs 打断）| "throw"
  reused: "inline",        // "inline" | "ref"（复用 schema 提取为 $defs）
  override: (ctx) => { /* 自定义某个 schema 的转换结果 */ },
  uri: (id) => id,         // id → 外部 $ref 的 URI
});
```

**`io` 的区别（管道 / 默认值 / coerce 会用到）：**

```ts
const mySchema = z.string().transform(v => v.length).pipe(z.number());

z.toJSONSchema(mySchema);                // { type: "number" }（输出侧）
z.toJSONSchema(mySchema, { io: "input" });// { type: "string" }（输入侧）
```

**处理不可表达类型：**

```ts
z.toJSONSchema(schema, {
  unrepresentable: "any",  // 无法表达的变成 {}
});

z.toJSONSchema(schema, {
  unrepresentable: (ctx) => {
    // ctx: { zodSchema, path, message }
    if (ctx.zodSchema instanceof z.ZodDate) return { type: "string", format: "date-time" };
  },
});
```

### 9.2 JSON Schema → Zod

> ⚠️ 实验性 API，未来可能变化。

```ts
const jsonSchema = {
  type: "object",
  properties: {
    name: { type: "string" },
    age: { type: "number" },
  },
  required: ["name", "age"],
};

const zodSchema = z.fromJSONSchema(jsonSchema);
```

---

## 10. 性能：AOT 编译

Zod 4 提供了把 schema **预编译成扁平 JS 校验函数**的能力（基于 `new Function()`），在热路径上显著提速。

### 10.1 编译单个 schema

```ts
const schema = z.compile(z.object({
  name: z.string(),
  xp: z.number(),
}));
```

> ⚠️ **编译最后一个 schema**。`.refine()` / `.extend()` / `.optional()` / `.meta()` 这些"派生新 schema"的方法返回的都是**未编译**的新实例：
>
> ```ts
> // ❌ refine 的结果没有编译
> const a = z.compile(z.string()).refine(v => v.length > 1);
> // ✅ 最后编译
> const b = z.compile(z.string().refine(v => v.length > 1));
> ```

### 10.2 全局开启编译

```ts
import "zod/compile"; // 必须在定义 schema 的模块之前
import * as z from "zod";

const schema = z.object({ name: z.string() });
schema.parse({ name: "ok" }); // 首次 parse 时编译
```

编译是惰性的：只有真正 parse 过的 schema 才会被编译。

也可以作为 Node 启动参数（保证早于任何模块执行）：

```sh
node --import zod/compile app.js    # ESM
node --require zod/compile app.cjs  # CommonJS
```

> 这是给**应用**用的，不要写进库的入口。

### 10.3 原理与限制

编译产物大概长这样：

```js
const isPoint = new Function("input", `
  if (typeof input !== "object" || input === null) return false;
  if (typeof input.x !== "number") return false;
  if (typeof input.y !== "number") return false;
  return true;
`);
```

- 校验通过 → 直接走快路径，性能最好
- 校验失败 → 回退到普通解析器，产出完整错误（所以编译对"失败路径"提速有限）
- 不支持的场景会**原样返回未编译 schema**：异步 refine/transform/check、`z.xor()`、递归 schema

CSP 受限环境（禁止 `new Function`）下不要开启编译。

---

## 11. 包结构与选型

Zod 4 一次发布了三个包/入口：

| 入口 | 定位 | 典型体积（gzip） |
| --- | --- | --- |
| `zod`（即 `zod/v4`） | 旗舰版：方法链式 API，DX 最好 | 简单脚本 ~5.9kb；三字段对象 ~13.1kb |
| `zod/mini` | tree-shakable 版：函数式 API，体积最小 | 简单脚本 ~2.1kb；三字段对象 ~4.0kb |
| `zod/v4/core` | 底层核心（`$` 前缀类），给库作者共享用 | — |

### 11.1 Zod Mini 的写法差异

Mini 把方法换成顶层函数，所有校验塞进 `.check()`：

```ts
// 常规 Zod
z.string().min(5).max(10).trim().optional();

// Zod Mini
z.optional(z.string().check(z.minLength(5), z.maxLength(10), z.trim()));
```

```ts
// 常规
z.object({ name: z.string(), age: z.number().optional() });

// Mini
z.object({ name: z.string(), age: z.optional(z.number()) });
```

其它差异：

- 用 `z.parse(schema, data)` / `z.safeParse(schema, data)`，而不是 `schema.parse()`
- 读取内部定义要走 `.def`：`Dog.def.shape.name`
- 默认**不带任何 locale**，错误消息统一是 `Invalid input`
- 递归 schema 需要 `z.config({ memoizer: z.memoizer() })`
- 部分便捷属性没有对应（如 `enum.enum`、`literal.values`）

### 11.2 什么时候该用 Mini？

官方态度很明确：**除非你有非常严苛的体积约束，否则用常规 Zod。**

理由：

- Mini 的 API 更啰嗦、自动补全发现性更差
- 后端（含 Lambda）场景下，5-10kb 的差异对冷启动几乎无感（1kb 与 128kb 的冷启动差异只有 5ms 量级）
- 前端只有在"面向低速移动网络的用户"时，10kb 才真正有意义

---

## 12. 给库作者：peer 依赖与子路径

如果你在写一个"接受用户传入 schema"的库：

**✅ 应该从 `zod/v4/core` 导入：**

```ts
import * as z4 from "zod/v4/core";
```

这是 Zod 4 的**永久子路径**（permalink），跨大版本都可用；而且 `$ZodType` 这些基类同时被常规 Zod 和 Zod Mini 继承，你的库能同时兼容两者。

**❌ 不要从这些路径导入：**

- `"zod"` —— 3.x 时导出 Zod 3，4.x 时导出 Zod 4，语义会变
- `"zod/v4"` / `"zod/v4/mini"` —— 绑定了具体实现，另一版就用不了

**同时支持 Zod 3 与 4：**

```json
{
  "peerDependencies": {
    "zod": "^3.25.0 || ^4.0.0"
  }
}
```

```ts
import * as z3 from "zod/v3";
import * as z4 from "zod/v4/core";

type Schema = z3.ZodTypeAny | z4.$ZodType;
```

---

## 13. 从 Zod 3 迁移到 Zod 4

社区 codemod：`zod-v3-to-v4`。以下按影响面从大到小排列。

### 13.1 错误定制统一为 `error`

```ts
// Zod 3
z.string().min(5, { message: "Too short." });
z.string({ required_error: "...", invalid_type_error: "..." });
z.string({ errorMap: (issue, ctx) => ({ message: "..." }) });

// Zod 4
z.string().min(5, { error: "Too short." });
z.string({
  error: (issue) => issue.input === undefined ? "必填" : "类型错误",
});
z.string().min(5, { error: (issue) => `必须 >${issue.minimum}` });
```

- `message` → `error`（旧参数仍可用但已废弃）
- `invalid_type_error` / `required_error` 被移除
- `errorMap` → `error`

### 13.2 错误相关 API

- `.format()` → `z.treeifyError()`
- `.flatten()` → `z.flattenError()`
- `.errors` 被移除，用 `.issues`
- `.addIssue()` / `.addIssues()` 废弃，直接 `err.issues.push()`
- **parse 级 error map 不再覆盖 schema 级错误**（优先级反转）
- issue 类型改名/合并（`ZodInvalidTypeIssue` → `z.core.$ZodIssueInvalidType` 等），但基础结构不变，多数处理逻辑无需改动

### 13.3 数字

- `z.number()` 不再接受 `Infinity` / `-Infinity`
- `z.number().safe()` 等价于 `.int()`，不再接受浮点
- `z.number().int()` 只接受安全整数；新代码请用 `z.int()` / `z.int32()`

### 13.4 字符串

- 格式校验上移为顶层函数：`z.string().email()` → `z.email()`（方法形式保留但废弃）
- `z.uuid()` 更严格（符合 RFC 9562/4122，变体位必须 `10`）；宽松版用 `z.guid()`
- `z.base64url()` 不再允许 padding
- `z.string().ip()` 拆成 `z.ipv4()` / `z.ipv6()`
- `z.string().cidr()` 拆成 `z.cidrv4()` / `z.cidrv6()`

### 13.5 `z.coerce`

- 输入类型变为 `unknown`
- 对象里缺失的 coerce 字段会**报错**（Zod 3 会得到 `false` 之类），需要显式 `.default()`

### 13.6 `.default()`

- Zod 4 的 `.default()` 会**短路**，值必须是**输出类型**
- 想保留"先解析默认值"的旧行为，用 `.prefault()`

### 13.7 对象

- `optional` 字段里的 `.default()` 现在也会生效
  ```ts
  z.object({ a: z.string().default("tuna").optional() }).parse({}); // { a: "tuna" }
  ```
- `.strict()` / `.passthrough()` 废弃 → `z.strictObject()` / `z.looseObject()`
- `.strip()` / `.nonstrict()` / `.deepPartial()` 被移除或废弃
- `z.unknown()` / `z.any()` 不再标记为"key optional"，`{}` 会报错
- `.merge()` 废弃 → `.extend(Other.shape)` 或直接对象展开（后者对 tsc 性能最好）

### 13.8 其它

- `z.nativeEnum()` 废弃 → `z.enum(TSEnum)`
- `z.array().nonempty()` 类型变为 `string[]`（不再是 `[string, ...string[]]`）；想要元组语义用 `z.tuple([z.string()], z.string())`
- `z.promise()` 废弃
- `z.function()` 不再是 schema，而是函数工厂：`z.function({ input: [...], output })` + `.implement()` / `.implementAsync()`

---

## 14. API 速查表

### 解析与校验

| API | 说明 |
| --- | --- |
| `.parse(data)` | 校验并返回深拷贝，失败抛 `ZodError` |
| `.parseAsync(data)` | 异步版 |
| `.safeParse(data)` | 返回 `{ success, data }` / `{ success: false, error }` |
| `.safeParseAsync(data)` | 异步安全版 |
| `.validate(data)` | 只返回布尔值，最快 |
| `.validateAsync(data)` | 异步版 |
| `.decode()` / `.encode()` | codec 双向 |
| `z.decode(schema, x)` / `z.encode(schema, x)` | 顶层函数形式 |

### 类型工具

| API | 说明 |
| --- | --- |
| `z.infer<typeof S>` | 输出类型 |
| `z.input<typeof S>` | 输入类型 |
| `z.output<typeof S>` | 输出类型 |
| `z.input(S)` / `z.output(S)` | 运行时取输入/输出侧 schema |
| `z.toZod<T>()` | 用目标类型校验 schema |

### 包装与变换

`optional` `nonoptional` `nullable` `nullish` `default` `prefault` `catch` `array` `or` `and` `transform` `pipe` `readonly` `brand` `apply` `annotate`(`.check`) `register` `describe` `meta` `clone` `isOptional()` `isNullable()`

### 顶层构造器

`z.object` `z.strictObject` `z.looseObject` `z.array` `z.tuple` `z.union` `z.xor` `z.discriminatedUnion` `z.intersection` `z.record` `z.partialRecord` `z.looseRecord` `z.map` `z.set` `z.enum` `z.literal` `z.templateLiteral` `z.stringbool` `z.file` `z.json` `z.custom` `z.instanceof` `z.function` `z.codec` `z.preprocess` `z.lazy` `z.compile` `z.toJSONSchema` `z.fromJSONSchema`

### 字符串格式

`z.email` `z.uuid` `z.guid` `z.url` `z.httpUrl` `z.hostname` `z.e164` `z.emoji` `z.base64` `z.base64url` `z.hex` `z.jwt` `z.nanoid` `z.cuid` `z.cuid2` `z.ulid` `z.ipv4` `z.ipv6` `z.mac` `z.cidrv4` `z.cidrv6` `z.creditCard` `z.currencyCode` `z.iban` `z.hash` `z.iso.date` `z.iso.time` `z.iso.datetime` `z.iso.duration`

### 错误工具

`z.prettifyError` `z.treeifyError` `z.flattenError` `z.config` `z.registry` `z.globalRegistry` `z.regexes` `z.locales`

---

## 15. 常见坑与最佳实践

**1）校验函数不要抛异常**

```ts
// ❌ Zod 不会捕获你抛的错
z.string().refine((v) => { if (!ok(v)) throw new Error("bad"); return true; });

// ✅ 返回假值
z.string().refine((v) => ok(v), { error: "bad" });
```

**2）异步校验记得用 `parseAsync` / `safeParseAsync`**

```ts
await schema.parseAsync(data);      // schema 里有 async refine/transform 时
```

**3）`z.enum` 的数组别先存变量**

```ts
const arr = ["a", "b"];           // ❌ 推断成 string
const s = z.enum(arr as const);   // ✅
```

**4）对象合并优先 `.extend()` / 展开，而不是 `z.intersection()`**

`ZodIntersection` 没有 `pick` / `omit` / `shape` 等方法。

**5）`.default()` vs `.prefault()` 看你要不要走解析管道**

- 默认值是"最终值" → `.default()`
- 默认值是"待解析的输入"（比如想顺便 trim / 转大写）→ `.prefault()`

**6）`z.coerce.boolean()` 不是"字符串转布尔"**

`"false"` 会变成 `true`。解析环境变量/表单串请用 `z.stringbool()`。

**7）`z.xor()` 里对象分支容易"双匹配"**

对更窄的分支加 `.strict()`，或用 `z.discriminatedUnion()`。

**8）性能敏感场景：编译 + `validate`**

```ts
const fast = z.compile(schema);
if (!fast.validate(input)) { /* 再走 safeParse 拿详细错误 */ }
```

**9）编译要放在链式调用的最后**

任何派生新 schema 的方法都会让编译失效。

**10）库作者从 `zod/v4/core` 导入**

否则你的库只能支持常规 Zod 或 Mini 其中之一。

**11）中文错误消息**

```ts
import { zhCN } from "zod/locales";
z.config(zhCN());
```

Zod Mini 需要手动配置，否则永远只有 `Invalid input`。

**12）`z.unknown()` 不是"可选"**

```ts
z.object({ a: z.unknown() }).parse({}); // ❌ Zod 4 起报错
z.object({ a: z.unknown().optional() }).parse({}); // ✅
```

---

## 附：官方参考链接

- 首页：https://zod.dev
- 基础用法：https://zod.dev/basics
- Schema API：https://zod.dev/api
- Codecs：https://zod.dev/codecs
- 错误定制：https://zod.dev/error-customization
- 错误格式化：https://zod.dev/error-formatting
- 元数据：https://zod.dev/metadata
- JSON Schema：https://zod.dev/json-schema
- AOT 编译：https://zod.dev/compile
- Zod Mini：https://zod.dev/packages/mini
- Zod Core：https://zod.dev/packages/core
- 迁移指南：https://zod.dev/v4/changelog
- 版本说明：https://zod.dev/v4

> 本文为中文整理稿，API 以 Zod 4 官方文档为准；如遇版本差异，请以官方最新文档为依据。
