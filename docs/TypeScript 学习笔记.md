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

