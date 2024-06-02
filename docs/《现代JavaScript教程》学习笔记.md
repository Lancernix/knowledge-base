# JavaScript 基础

## 基本数据类型转换

大多数的基本数据类型转换都是比较容易理解的。需要注意的是几种特定的转换，不太符合我们**自以为**的理解：

- `null` 和 `undefined` 表现并不一致。  
   - `Number(undefined) => NaN`  
   - `Number(null) => 0`
- `string` 类型转换成 `number` 时会忽略两边的空白字符（包括 `\t\n` 等）之后再尝试转换，如果为空字符会转换为 0；其他情况下，则需要剩下的整体看上去是一个合法的数字才可以成功转换。

> 这一转换规则和 `Number.parseInt` 或 `Number.parseFloat` 在解析十进制时所展现的结果是不同的，需要特别注意

```javascript
Number('') => 0
parseInt('', 10) => NaN
Number('   ') => 0
parseInt('   ') => NaN
Number(' 123 ') => 123
Number(' 12q ') => NaN
parseInt(' 12q ') => 12
```

> `parseInt` 和 `parseFloat` 是从字符串中从左至右读取数字，直到不能没办法读取为止。如果第一个（不包括空格、`\t`、`\n`）非数字，则会直接返回 `NaN`。

- `'0'` 和 `' '`（只有空格的字符串）的布尔转换都是 `true`（简单理解就是**空字符串才会转换成 `false`**。  
   - `Boolean('') => false`  
   - `Boolean(' ') => true`  
   - `Boolean('0') => true`

## 数学运算符

- 一元运算符 `+/-` 和自增/自减运算符 `++/--` 的优先级高于二元运算符。
- 赋值运算符 `=` 的优先级很低。
- `+` 作为二元运算符和其他的二元运算符有些不同，主要表现在字符串的处理上。通常二元运算符在遇到字符串时，会先将其转换成数字再进行运算，但 `+` 对待字符串是一个合并操作。  
   - `4 + 5 + 'px' = '9px'`  
   - `'$' + 4 + 5 = '$45'`

一些容易出错的例子：

```javascript
"" + 1 + 0 = "10"
"" - 1 + 0 = -1
true + false = 1
6 / "3" = 2
"2" * "3" = 6
4 + 5 + "px" = "9px"
"$" + 4 + 5 = "$45"
"4" - 2 = 2
"4px" - 2 = NaN
"  -9  " + 5 = "  -9  5"
"  -9  " - 5 = -14
null + 1 = 1
undefined + 1 = NaN
" \t \n" - 2 = -2
```

## 值的比较

- 字符串的比较是从左往右逐个字符进行比较的，大小的依据则是单个字符在**Unicode 编码**中的顺序，顺序在后的大于在前的。
- 不同基本类型的两个值进行比较，会先转换成数字（严格相等性检查 `===` 除外，它不会做任何类型转换），然后再进行比较。**但是在相等性检查 `==` 时，`null` 和 `undefined` 是一个例外，它俩仅仅等于对方而不等于其他任何的值**（可以理解为，在 `==` 时，`null` 和 `undefined` 就不进行数字转换了，它俩只认对方）。

```javascript
null == undefined // true
null === undefined // false
null > 0 // false
null == 0 // false
null >= 0 // true（null 转换成了 0，然后比较）
undefined > 0 // false（undefined 转换成 NaN，然后比较）
undefined < 0 // false
undefined == 0 // false
```

## 循环跳出

通常 `break/continue` 只能用于跳过当前的循环。在多层嵌套的循环中，如果想跳出到最外层的循环，需要**标签**来帮助实现这一功能：

```javascript
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 2) {
      break outer; // 这里跳出指定标签的循环
    }
    console.log(`${i}, ${j}`); // 最终只会输出 0，1
  }
}
```

## 对象转换为原始类型

- 所有对象转换为布尔值均为 `true`。
- 在对象（这里的对象指的是我们最常用的普通对象）进行数字转换或者字符串转换时，有一定的转换知识和规则：  
   - 需要将对象转换成原始类型时，有 3 种情况，在 ECMA 规范中被称为 "hint"。它们分别是：`string`、`number`、`default`。  
      - 预期是一个字符串的操作时，hint 会是 `string`。如 `alert` 方法、对象作为属性 `obj1[obj] = 123` 等。  
      - 预期是一个数字的操作时，hint 会是 `number`。如数学运算。  
      - 当不确定是什么样的操作时，hint 会是 `default`。如二元加运算 `+`（字符串和数字都可以用）。此外，有两个情况需要注意：对象被用于与字符串、数字或 symbol 进行 `==` 比较，这时到底应该进行哪种转换也不是很明确，因此使用 `default` hint；由于历史原因 `<`、`>` 运算符虽然也可以同时用于字符串和数字，但它们使用 `number`hint，而非 `default`。  
   - 进行转换时，JavaScript 会尝试查找并调用三个特定的方法：`Symbol.toPrimitive`、`toString`、`valueOf`。  
      - `Symbol.toPrimitive` 接受一个名为 hint 的参数（`hint: 'string' | 'number' | 'default'`），并返回一个原始值。  
      - `toString`、`valueOf` 是很常规的方法，在没有 symbol 时，它们被用于实现类型转换（当然现在仍然用于类型转换，只不过优先级没有 `Symbol.toPrimitive` 高）。  
      - 默认情况下，`toString` 返回 `[object Object]` 字符串；`valueOf` 返回对象本身。  
      - 这三个方法都应该返回一个原始值，这样才有实际意义。`Symbol.toPrimitive` 必须返回一个原始值，否则会出现 error；但 `toString` 或 `valueOf` 其中一个返回对象并不会出现 error；不过如果 `toString` 或 `valueOf` 都返回了对象，同样也会出现 error。  
   - 具体的转换规则如下：  
      1. 如果 `Symbol.toPrimitive` 方法存在，则调用它进行转换。  
      2. 否则，如果为 `string` hint，则按照顺序调用 `toString`、`valueOf`：`toString` 的返回值为原始值，则不会再调用 `valueOf`；否则将调用 `valueOf`；若 `valueOf` 的值仍然是一个对象，则出现 error。  
      3. 否则，如果为 `number` 或者 `default` hint，则按顺序调用 `valueOf`、`toString`（处理方式同第二点，不过顺序不同）。

可以参照以下例子理解：

```javascript
const user = {
  name: 'John',
  money: 1000,
};
console.log(String(user)); // '[object Object]'
console.log(Number(user)); // NaN
console.log('500' + user); // '500[object Object]'
console.log(500 > user); // false

// ------  

const user = {
  name: 'John',
  money: 1000,
  
  toString() {
    console.log('调用 toString');
    return '[object Object]';
  },
  valueOf() {
    console.log('调用 valueOf');
    return user;
  },
};
console.log(String(user)); // '调用 toString' => '[object Object]'
console.log(Number(user)); // '调用 valueOf' => '调用 toString' => NaN
console.log('500' + user); // '调用 valueOf' => '调用 toString' => '500[object Object]'
console.log(500 > user); // '调用 valueOf' => '调用 toString' => false

// ------  

const user = {
  name: 'John',
  money: 100,
  
  toString() {
    console.log('调用 toString');
    return user.name;
  },
  valueOf() {
    console.log('调用 valueOf');
    return user.money;
  },
};
console.log(String(user)); // '调用 toString' => 'John'
console.log(Number(user)); // '调用 valueOf' => 1000
console.log('500' + user); // '调用 valueOf' => '5001000'
console.log(500 > user); // '调用 valueOf' => false

// ------  

const user = {
  name: 'John',
  money: 1000,
  
  toString() {
    console.log('调用 toString');
    return user;
  },
  valueOf() {
    console.log('调用 valueOf');
    return user.money;
  },
};
console.log(String(user)); // '调用 toString' => '调用 valueOf' => '1000'
console.log(Number(user)); // '调用 valueOf' => 1000
console.log('500' + user); // '调用 valueOf' => '5001000'
console.log(500 > user); // '调用 valueOf' => false

// ------

const user = {
  name: 'John',
  money: 1000,
  
  toString() {
    console.log('调用 toString');
    return user;
  },
  valueOf() {
    console.log('调用 valueOf');
    return user;
  },
};
console.log(String(user)); // 'TypeError: Cannot convert object to primitive value'
console.log(Number(user)); // 'TypeError: Cannot convert object to primitive value'
console.log('500' + user); // 'TypeError: Cannot convert object to primitive value'
console.log(500 > user); // 'TypeError: Cannot convert object to primitive value'

// ------

const user = {
  name: 'John',
  money: 1000,
  
  [Symbol.toPrimitive](hint) {
    console.log(`hint: ${hint}`);
    return hint == 'string' ? this.name : this.money;
  },
};
console.log(String(user)); // 'hint: string' => 'John'
console.log(Number(user)); // 'hint: number' => 1000
console.log('500' + user); // 'hint: default' => '5001000'
console.log(500 > user); // 'hint: number' => false

// ------ 

const user = {
  name: 'John',
  money: 1000,
  
  [Symbol.toPrimitive](hint) {
    console.log(`hint: ${hint}`);
    return hint == 'string' ? this : this;
  },
};
console.log(String(user)); // 'TypeError: Cannot convert object to primitive value'
console.log(Number(user)); // 'TypeError: Cannot convert object to primitive value'
console.log('500' + user); // 'TypeError: Cannot convert object to primitive value'
console.log(500 > user); // 'TypeError: Cannot convert object to primitive value'
```

## 可迭代对象 & 类数组对象

首先给出两个名词比较准确的概念：

- **可以使用 `for…of` 进行迭代的对象，我们称之为可迭代对象**。
- **拥有索引属性和 `length` 属性的对象，我们称之为类数组对象**。

也可以从 TypeScript 的定义中帮助理解：

```typescript
interface ArrayLike<T> {
  readonly length: number;
  readonly [n: number]: T;
}
```

接下来深入了解一下两者：

如果一个对象想要能够被迭代，那么必须有一个名为 `Symbol.iterator` 的方法，这个方法通常称之为迭代器（iterator）。迭代器是一个必须包含 `next` 方法的对象，且需要 `next` 方法返回形如 `{done: Boolean, value: any}` 的一个对象。我们在使用 `for…of` 进行对象迭代的时候，`Symbol.iterator` 会被自动调用，从而完成这个迭代的过程。

```javascript
const iteratorObj = {
  min: 0,
  max: 5,  

  [Symbol.iterator]() {
    return {
      from: this.min,
      to: this.max,
      next() {
        while (this.from < this.to) {
          return { done: false, value: this.from++ };
        }
        return { done: true };
      },
    };
  },
};  

for (const item of iteratorObj) {
  console.log(item); // 依次输出：0，1，2，3，4
}
```

上面的例子中，我们通过给对象增加 `Symbol.iterator` 方法，将该对象变成了一个可迭代的对象。仔细想一想，其实迭代器对象和被迭代化的对象其实是两个对象，但也可以是同一个对象。比如这样：

```javascript
const iteratorObj = {
  min: 0,
  max: 5,  

  [Symbol.iterator]() {
    return this;
  },  
  next() {
    while (this.min < this.max) {
      return { done: false, value: this.min++ };
    }
    return { done: true };
  },
};
```

类数组对象首先是一个对象，这就意味着它不能使用数组所具有的方法，如 `pop`、`push` 等，即使该对象恰好有同名的方法，大概率结果也不会你预期的结果相同。

```javascript
const obj1 = {
  a: 'a',
  1: 'b',
  length: 2,
};
const arr1 = Array.from(obj1); // ['a', 'b']

// ------

const obj2 = {
  a: 'a',
  1: 'b',
  3: 'e',
  length: 2,
};
const arr2 = Array.from(obj2); // [undefined, 'b']

// ------

const obj3 = {
  a: 'a',
  1: 'b',
  3: 'e',
  length: 4,
};
const arr3 = Array.from(obj3); // [undefined, 'b', undefined, 'e']
```

数组和字符串这两种类型，既是可迭代的也是类数组的。`Array.from` 可以将可迭代对象或类数组对象转化为真正的数组。
