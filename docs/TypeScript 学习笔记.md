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

# 