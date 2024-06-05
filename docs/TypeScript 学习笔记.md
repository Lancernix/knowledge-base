# 基础知识

```typescript
interface Person {
	name: string;
	address: string;
	age: number;
	isAdult: boolean;
}
// 获取对象类型的所有key的类型
type PersonKeys = keyof Person; // 'name' | 'address' | 'age' | 'isAdult'
// 获取对象类型的所有value的类型
type PersonValues = Person[keyof Person]; // string | number | boolean

// ======

const arr = [1, 2, 'test', true] as const;
type ArrayType = typeof arr;
// 获取数组某个元素的类型
type EleType = ArrayType[0]; // 1
// 获取数组所有元素的类型
type EleTypes = ArrayType[number]; // true | 1 | 2 | "test"

// ======

enum Color {
  Red, // 0
  Green, // 1
  Blue, // 2
}
// 获取枚举所有成员名的类型，注意enum有点特殊，需要先用typeof
type EnumKeys = keyof typeof Color; // "Red" | "Green" | "Blue"
// 可以用in运算符来获取枚举的所有值
type EnumValues = { [value in Color]: value } // 其实就是本身Color，这里关注一下in操作即可
```

# `as const` 类型断言

`as const` 是 TypeScript 中的一个类型断言，主要有两个作用：

- **将变量的类型提升到『字面量类型』，而不是更泛化的类型**。例如，`const str = 'hello' as const` 其中 `str` 的类型将变成 `'hello'` 而不是 `string`。
- **将变量视为只读，不能再被修改**。这意味着你不能修改数组/元组的元素或长度，不能修改对象的属性的值等。

它**只能用在数组、对象和基本类型（字符串、数字等）的字面量上**，以下是一些示例：

```typescript
// 将一个数组转换为只读的元组，元组元素的类型是它们的字面量类型
// arr 的类型为 readonly [1, 'a', true]
const arr = [1, 'a', true] as const;

// 对象的属性变为只读的，并且属性值的类型是它们的字面量类型
// obj 的类型为 { readonly a: 1, readonly b: '2', readonly c: true }
const obj = { a: 1, b: '2', c: true } as const;

// 变量的类型被提升为具体的字面量类型，而非宽泛的 string、number 等类型
// str 的类型为 'hello'
const str = 'hello' as const;
// num 的类型为 123
const num = 123 as const;
// bool 的类型为 true
const bool = true as const;
```

# 只读数组

两种定义方式：

- **使用 `readonly` 修饰符，只能用在数组、元组的字面量类型上**。例如：`readonly string[]`，不能用在数组泛型类型上。
- **使用 `Readonly` 或者 `ReadonlyArray` 两个泛型**。例如：`Readonly<Array<string>>`、`ReadonlyArray<string>`。

通常理解，只读数组和数组的关系应该是子类和父类的关系，因为只读数组属于数组嘛，但是在 TypeScript 中是不对的。**Typescript 中数组是只读数组的子类型**。这是因为只读数组没有 `pop()`、`push()` 之类会改变原数组的方法，而数组显然是有的。也就是说，数组有只读数组的所有特性，并且还有自己专有的特性（即子类型继承了父类型的所有特征，并加上了自己的特征）。

# 扩展运算符 `…`

在类型定义中，同样可以使用扩展运算符 `…`，用法和在 JavaScript 类似，但是有**一个显著的区别：不能对『对象类型』使用**。

> 这里的对象类型在 Typescript 中并没有，不过我们日常使用最常用的就是使用 `interface`、`type` 给对象定义一个对应的类型，所以这里用这样的描述也比较好理解。

扩展运算符常用于数组、元组、函数参数等场景的类型定义中，例如：

```typescript
type Tuple1 = [1, true];
// Tuple2 的类型为 [1, true, string]
type Tuple2 = [...Tuple1, string];

type Params = [number, string, boolean];
// Func 的类型为 (args_0: number, args_1: string, args_2: boolean) => void
type Func = (...args: Params) => void;

type Arr1 = string[];
// Arr2 的类型为 string[]
type Arr2 = [...Arr1];
// Arr3 的类型为 [...string[], boolean, number]，一个不定长的元组
type Arr3 = [...Arr1, boolean, number];
```

# 类型递归

利用 `extends … ? … : …` 的条件运算符来调用自身，就可以实现类似于递归函数效果的类型递归。例如：

```typescript
// Type Challenges 189题：给定 Promise<ExampleType>，获取 ExampleType 类型
// 这里有一个注意的地方就是：如果 ExampleType 是一个 Promise，那就需要递归来处理
type MyAwaited<T extends PromiseLike<any>> = T extends PromiseLike<infer U>
  ? U extends PromiseLike<any>
  	? MyAwaited<U>
	: U
  : never;

// Type Challenges 108题：实现一个类似 JS 中 Trim 功能的类型
// 比如 Trim<'  hello '>  ==> 'hello'
type Space = " " | "\n" | "\t";
type Trim<S extends string> = S extends | `${Space}${infer Res}` | `${infer Res}${Space}`
  ? Trim<Res>
  : S;
```

# 枚举

我们在谈论 TypeScript 语法时，一般都说的是其独有的类型语法，这些会在编译后全部被去除。但是枚举比较特殊，它既是一种类型，也是一个值，编译之后会变成一个 JavaScript 对象（常量枚举 `const enum` 除外）。

```typescript
// enum成员如果都没赋值，默认会从0开始按照顺序为每个成员赋值
enum Color {
  Red, // 0
  Green, // 1
  Blue, // 2
}

// 如果只设定某一个成员的值，后面成员的值就会从这个值开始递增，前面的依然默认赋值
enum Color {
  Red, // 0
  Green = 7,
  Blue, // 8
}

// 字符串enum的所有成员值，都必须显式设置。如果没有设置，成员值默认为数值，且位置必须在字符串成员之前
enum Foo {
  A, // 0
  B = 'hello',
  C, // 报错
}
enum Bar {
  A, // 0
  One = 'One',
  Two = 'Two',
  Three = 3,
  Four = 4,
}
```

`enum` 还有几个比较值得注意的点：

- 枚举值如果是数字，则存在反向映射，即可以通过值来获取对应的成员名。而字符串是不可以的。
- 常量枚举 `const enum` 编译之后不会生成对应的对象，而是直接将枚举值在使用的地方进行替换。
- 如果你想获取一个 `enum` 的所有枚举值，可以通过 `Object.values()` 来实现，不过只适用于全部是字符串的枚举。如果有数字，则需要进行特殊的处理。

```typescript
// 字符串枚举
enum Status {
  Pending = 'PENDING',
  Fullfilled = 'FULLFILLED',
  Rejected = 'REJECTED',
}
// 编译之后
let Status = {
  Pending: 'PENDING',
  Fullfilled: 'FULLFILLED',
  Rejected: 'REJECTED',
};

// ======

// 数字枚举
enum Color {
  black,
  red,
  blue,
}
// 编译之后
// 这里就可以看出如果值是数字，确实存在反向映射的现象
let Color = {
  black: 0,
  red: 1,
  blue: 2,
  0: 'black',
  1: 'red',
  2: 'nlue',
};
// 此时用Object.values()获取到的并不只有[0, 1, 2]，就需要再进一步处理才可以

// ======

// 常量枚举无法获取所有的枚举值或者枚举成员，因为它编译之后并不存在对应的对象
const enum Color {
  black, // 0
  red, // 1
  blue, // 2
}
Object.values(Color); // 报错
```

实际应用中，其实并不那么推荐使用枚举类型（因为它既是一个类型，也可能是一个 JavaScript 对象），除非有特殊的需求（比如你要用到反向映射的特性）。一般情况我们可以通过增加了 `as const` 断言的对象类型来替代，或者用更加简单的联合类型来替代：

```typescript
enum Foo {
  A,
  B,
  C,
}

const Bar = {
  A: 0,
  B: 1,
  C: 2,
} as const;

if (x === Foo.A) {}
// 等同于
if (x === Bar.A) {}

// ======

enum Direction {
  Up = 'UP',
  Down = 'DOWN',
  Left = 'LEFT',
  Right = 'RIGHT',
}

type Direction = 'UP' | 'DOWN' | 'LEFT' | 'RIGHT'
```

# 条件类型 `extends`

 `extends … ? … : …` 和 JavaScript 中的三元运算很相似，通过判断来获得不同条件下给定的类型。通常来说，`A extends B ? true : false` 中如果 `A` 可以赋值给 `B` （可以理解为 `A` 是 `B` 的子类型），那么结果就是 `true`，反之则是 `false`。例如：

```typescript
type A = string;
type B = string | number;
// IsAString 的类型为 true
type IsAString = A extends string ? true : false;

type C = { name: string; age: number; };
type D = { name: string };
// E 的类型为 true
type E = C extends D ? true : false;
```

> 如何来判断一个类型可以赋值给另一个类型？可以类比与 Java 中的子类与父类，即：**如果类型 A 至少包含了类型 B 的所有属性和方法，那么类型 A 就可以赋值给类型 B**。我一般习惯使用『子类型和父类型』的关系来理解。

但如果 `extends` 左边的类型参数是一个联合类型（即：`Type1 | Type2`），处理逻辑会有些不同：**当 `T extends U ? X : Y` 中的 `T` 是一个联合类型的类型参数时，该条件类型将被分解为多个条件类型的联合（即分布式条件类型规则）**。

> 特别注意：**条件类型的分布式规则只有在条件类型的检查类型（`extends` 左边）位置为类型参数（而非具体的联合类型）时，才会进行分布**。如果在检查类型位置使用一个具体的联合类型，比如 `string | number`，那么这个联合类型就不会被拆解，它会被当作一整个类型来对待。

例如：

```typescript
type A = string | number;
type B = number | boolean;
// 这里的 A 并不是一个类型参数，而是一个具体的联合类型，所以并不会触发分布式规则
// string | number 显然不能赋值给 number | boolean
// 所以 C 的类型为 false
type C = A extends B ? true : false;

type ToArray<T> = T extends any ? T[] : never;
// 这里的条件类型进行处理的时候，如果给定的 T 是一个联合类型，就会触发分布式规则。因为 T 这个类型是泛型 ToArray 的类型参数
// 例如 type Result = ToArray<string | number>
type Result = ToArray<string | number>;
// 使用分布式规则分解一下就是拆分成这样
type Result = ToArray<string> | ToArray<number>;
// 最终的结果就是 Result 的类型为 string[] | number[]
```

常用的工具类型 `Exclude` 就是利用这个规则实现的：

```typescript
type Exclude<T, U> = T extends U ? never : T;
// A 的类型为 'b' | 'c'
type A = Exclude<'a' | 'b' | 'c', 'a'>；
```

如果在条件类型中处理联合类型的类型参数时不想默认的分布式规则生效，可以使用方括号（`[]`）将 `extends` 左右的两个类型包裹一下。例如：

```typescript
type ToArray<T> = [T] extends [any] ? T[] : never;
// 这里用方括号处理了一下，即使给定的 T 是一个联合类型，也不会触发分布式规则
// Result 的类型为 (string | number)[]
type Result = ToArray<string | number>;
```

# `Omit`

这个内建类型并不难实现，也不难理解。这里记录的原因是在 Type Challenges 做题时发现其实编辑器对于 `Omit` 的类型提示是有误导性的（通过 [第3题](https://github.com/type-challenges/type-challenges/tree/main/questions/00003-medium-omit) 和 [第8题](https://github.com/type-challenges/type-challenges/tree/main/questions/00008-medium-readonly-2) 可以验证）。

如果你在 VS Code 或者 [TS演练场](https://www.typescriptlang.org/zh/play) 中查看 `Omit` 的类型提示，会得到如下的结果：  
![Omit](../pics/ts-1.png)  
自然的，在自己实现 `Omit` 时，就会使用这种写法，在 Type Challenges 的第 3 题确实可以通过给出的 case。但是在第 8 题中，你会发现使用 `Omit` 可以通过给出的所有 case，但是如果用上面的实现方式替换 `Omit`，就会有部分 case 不通过。这两个不是完全一样的东西吗！？

其实并不是一样的，查看 [源码](https://github.com/microsoft/TypeScript/blob/main/src/lib/es5.d.ts) 就会看到 `Omit` 的实现并不是编辑器给出的类型提示那样，而是：`type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>`。其中的一个区别可以通过第 8 题的给出的例子类型来说明：

```typescript
interface Todo2 {
  readonly title: string;
  description?: string;
  completed: boolean;
}
// 根据编辑器的提示实现的 Omit
type MyOmit<T, K> = {
  [P in Exclude<keyof T, K>]: T[P];
}

type A = Omit<Todo2, 'description'>;
const a: A = { title: '123', completed: true };
// 修改 title 属性这里会报错：只读属性不可以赋值
a.title = 'title'; 

type B = MyOmit<Todo2, 'description'>;
const b: B = { title: '123', completed: true };
// 这里修改就是可以的
b.title = 'title'; 
```

从上述的代码中能发现是有差异的，通过编辑器的类型提示也可以看出来：  
![类型差异](../pics/ts-2.png)  
差异就在于：`Omit` 可以将属性修饰符 `readonly` 带过来，但是 `MyOmit` 不可以，具体解释可以参考这个 [issue](https://github.com/microsoft/TypeScript/issues/39802) 。

所以只使用 `Exclude` 并不能完全模拟 `Omit`。想要完全模拟，一种方式是官方的定义，另一种则是通过 `as` 的方式来实现（只能用在 v4.1 之后的版本，原理可以参考 [文档](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#key-remapping-via-as)）：

```typescript
type MyOmit<T, K extends PropertyKey> = {
  [P in keyof T as P extends K ? never : P]: T[P];
};
```

# 模板字符串类型

模板字符串类型是在 v4.1 版本中引入的，和 JavaScript 中的模板字符串用法类似，只不过是作用于类型。通过这个特性，我们可以实现定义更加精准的类型，也可以更加方便地定义一些类型。

```typescript
type UserInput = string;
// 使用模板字符串类型
type Greeting = `Hello, ${UserInput}!`;

const aGreeting: Greeting = "Hi there!"; // 类型检查错误，因为不符合模板
const bGreeting: Greeting = 'Hello, Tom!'; // 类型检查通过
const user = "Alice";
const greeting: Greeting = `Hello, ${user}!`; // 类型检查通过
```

再比如一个 tooltip 组件一般会有出现位置的属性，此时我们就可以通过模板字符串类型来定义这个属性的类型：

```typescript
type Placement = 'top'| 'bottom' | 'left' | 'right' | `${'top' | 'bottom'}-${'left' | 'right'}` | `${'left' | 'right'}-${'top' | 'bottom'}`
```

同时，也能实现一些类似 JavaScript 中的 `Trim`、`Replace` 等字符串方法的类型，比如 Type Challenges 的 [第108题](https://github.com/type-challenges/type-challenges/blob/main/questions/00108-medium-trim) 和 [第116题](https://github.com/type-challenges/type-challenges/blob/main/questions/00116-medium-replace)。同时，TypeScript 也提供了一些内置的字符串工具类型，比如：`Uppercase<StringType>`、`Lowercase<StringType>`、`Capitalize<StringType>`、`Uncapitalize<StringType>` 等。
