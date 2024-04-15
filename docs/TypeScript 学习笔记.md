# `as const` 类型断言

`as const` 是 TypeScript 中的一个类型断言，主要有两个作用：

- **将变量的类型提升到『字面量类型』，而不是更泛化的类型**。例如，`const str = 'hello' as const;` 其中 `str` 的类型将变成 `'hello'` 而不是 `string`。

- **将变量视为只读，不能再被修改**。这意味着你不能修改数组/元组的元素或长度，不能修改对象的属性的值等。

它**只能用在数组、对象和基本类型（字符串、数字等）的字面量上**，以下是一些示例：

```typescript
 
const arr = [1, 'a', true] as const;  // arr 的类型为 readonly [1, 'a', true]
```



- 对于对象：

复制插入新建

```typescript
const obj = { a: 1, b: '2', c: true } as const;
// obj 的类型为 { readonly a: 1, readonly b: '2', readonly c: true }
```

对象的属性变为只读的，并且属性值的类型是它们的字面量类型。

- 对于字符串、数字和布尔值等基本类型：

复制插入新建

```typescript
const str = 'hello' as const;
// str 的类型为 'hello'
const num = 123 as const;
// num 的类型为 123
```

变量的类型被提升为具体的字面量类型，而非宽泛的 `string`、`number` 等类型。

总的来说，`as const` 是一个非常强大的工具，它能够帮助你准确地描述你的数据的类型和结构，同时允许你保护数据的不变性，防止无意的修改。
