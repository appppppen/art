> 本文总结一下 TypeScript 类型声明的书写，很多时候写 TypeScript 不是问题，写类型就特别纠结，我总结下，我在使用 TypeScript 中遇到的问题。如果你遇到类型声明不会写的时候，多看看 lodash 的声明，因为 lodash 对数据进行各种变形操作，所以你能遇到的，都有参考示例。

### 基本类型

```typescript
// 变量
const num: number = 1;
const str: string = "str";
const bool: boolean = true;

const nulls: null = null;
const undefine: undefined = undefined;
const symbols: symbol = Symbol("symbal");

const any: any = "any types"; // typescript的any类型，相当于什么类型约束都没有
```

### 数组

```typescript
// 数组: 推荐使用T[]这种写法
const nums: number[] = [1, 2, 3, 4];

// 不推荐：Array<T>泛型写法，因为在JSX中不兼容，所以为了统一都使用T[]这种类型
const strs: Array<string> = ["s", "t", "r"];

const dates: Date[] = [new Date(), new Date()];
```

#### 数组的 concat 方法，返回类型为 never[]问题

```typescript
// 数组concat方法的never问题
// 提示： Type 'string' is not assignable to type 'never'.
const arrNever: string[] = [].concat(["s"]);

// 主要问题是：[]数组，ts无法根据上下文判断数组内部元素的类型
// @see https://github.com/Microsoft/TypeScript/issues/10479
const fixArrNever: string[] = ([] as string[]).concat(["s"]);
```

### 接口

接口是 TypeScript 的一个核心知识，它能合并众多类型声明至一个类型声明：

而且接口可以用来声明：函数，类，对象等数据类型

```typescript
interface Name {
  first: string;
  second: string;
}

let username: Name = {
  first: "John",
  second: "Doe",
};
```

### any、null、undefined、void 类型

```typescript
// 特殊类型
const any: any = "any types"; // typescript的any类型，相当于什么类型都没写
let nobody: any = "nobody, but you";
nobody = 123;

let nulls: number = null;
let bool: boolean = undefined;

// void
function printUsername(name: string): void {
  console.log(name);
}
```

### 联合类型

> 联合类型在 option bags 模式场景非常实用，使用 **| **来做标记

```typescript
function options(opts: { types?: string; tag: string | number }): void {}
```

### 交叉类型

> 最典型的使用场景就是继承和 mixin，或者 copy 等操作

```typescript
// 交叉类型：如果以后遇到此种类型声明不会写，直接看Object.assign声明写法
function $extend<T, U>(first: T, second: U): T & U {
  return Object.assign(first, second); // 示意而已
}
```

### 元组 tuple

> 元组很少使用

```typescript
let nameNumber: [string, number];

// Ok
nameNumber = ["Jenny", 221345];

// Error
// nameNumber = ['Jenny', '221345'];

let tuple: [string, number];
nameNumber = ["Jenny", 322134];

const [usernameStr, uselessNum] = nameNumber;
```

### type 的作用

> type 用来创建新的类型，也可以重命名（别名）已有的类型，建议使用 type 创建简单类型，无嵌套的或者一层嵌套的类型，其它复杂的类型都应该使用 interface, 结合 implements ,extends 实现。

```typescript
type StrOrNum = string | number;

// 使用
let sample: StrOrNum;
sample = 123;
sample = "123";

// 会检查类型
sample = true; // Error
```

### 实践中遇到的问题

#### 第三方库没有提供声明 d.ts 文件

> 如果第三方库没有提供声明文件，第一时间去微软官方的[仓库](https://github.com/borisyankov/DefinitelyTyped) 查找，或者在 npmjs.com 上搜索@types/依赖的模块名大部分情况都可以找到。

- 手动添加声明文件

> 声明文件一般都是统一放置在 types 文件夹下

```typescript
// 例子： types/axios.d.ts
declare module "axios"; // 这里的axios声明为any类型
```

#### 全局变量

> 例如一些库直接把在 window 上添加的全局变量

```typescript
// globals.d.ts
// 例子：jQuery，现实中jQuery是有.d.ts
declare const jQuery: any;
declare const $: typeof jQuery;
```

### 非 JavaScript 资源

> 在前端工程中，import 很多非 js 资源，例如：css, html, 图片，vue, 这种 ts 无法识别的资源时，就需要告诉 ts，怎么识别这些导入的资源的类型。

```typescript
// 看看vue怎么处理的：shims-vue.d.ts
declare module "*.vue" {
  import Vue from "vue";
  export default Vue;
}

// html
declare module "*.html";
// css
declare module "*.css";
```

### 强制类型转换

> 有时候遇到需要强制类型转换，尤其是对联合类型或者可选属性时。

```typescript
// 第一种：使用<>括号
const convertArrType: string[] = <Array<string>>[].concat(["s"]);

// 第二种：使用as关键字
const fixArrNever: string[] = ([] as string[]).concat(["s"]);
```

> 建议使用第二种，因为兼容 JSX，第一种官方也不推荐了，虽然它是合法的。

### 可选属性和默认属性

> API 中提供的参数很多都有默认值，或者属性可选，怎么书写呢？

```typescript
class Socket {}
// 联合类型
export type SocketType = 'WebSocket' | 'SockJs';

export interface SocketOptions {
  type: SocketType;
  protocols?: string | string[]; // 可选
  pingMessage: string | (() => string); // 联合类型，可以为string或者函数
  pongMessage: string | (() => string);
}

// 默认值
export function eventHandler = (
  evt: CloseEvent | MessageEvent | Event,
  socket: Socket,
  type = 'WebSocket' // 默认值
) => any;

```

### 独立函数怎么声明类型

> 刚开始我也很纠结这个问题，我就是一个独立的函数，怎么声明类型呢？尤其是写事件处理函数的 API 时。

```typescript
class Socket {}
// 函数的声明方式
export type SocketEventHandler = (
  evt: CloseEvent | MessageEvent | Event,
  socket: Socket
) => any;

const eventHandler: SocketEventHandler = (evt, socket) => {};

// 可选参数和rest参数
let baz = (x = 1) => {};
let foo = (x: number, y: number) => {};
let bar = (x?: number, y?: number) => {};
let bas = (...args: number[]) => {};
```

### 索引属性类型声明

> JavaScript 中的对象都可以使用字符串索引直接取属性或者调用方法，TypeScript 中也有相应的类型声明方法。

```typescript
type Hello = {
  hello: "world";
  // key只是一个形式属性名（类似形参一样）
  [key: string]: string;
};

const greeting: Hello = {
  hi: "morning",
};

console.log(greeting["hi"]);
```

### 动态添加的属性声明

> 有的时候我们只声明了一个基本的类型结构，然后后续有扩展的情况 ，尤其时二次封装时的 options。

```typescript
interface AxiosOptions {}

type AjaxOptions = {
   axiosOptions: AxiosOptions;
   // 额外扩展的放入到单独的属性节点下
   extraOptions: {
       [prop: string]: any
   };
};

type AjaxOptions1 = {
  axiosOptions?: AxiosOptions;
  // 不要这样写，因为axiosOptions拼写错误时，ts不会提示
  // 尽量把后续扩展的属性，移动到独立的属性节点下
  [prop: string]: any
};

const ajaxOptions: AjaxOptions1 = {
  axiosOptions1: {}; // 本意是axiosOptions，但是ts不会提示
}

```

### !的使用

> ! 标识符告诉 ts 编译器，声明的变量没有问题，再运行期不会报错。

```typescript
class BaseSelect extends Vue {
  options: string[]; // 这里会提示没有在constructor中初始化

  created() {
    this.options = ["inited"];
  }
}

class BaseSelect extends Vue {
  options!: string[]; // 使用 ! 告诉编译器，我知道自己在做什么

  created() {
    this.options = ["inited"];
  }
}
```

### this 的使用

> 对于独立使用的函数，可以声明指定的调用上下文

```typescript
class Handler {
  info: string;
  // 声明指定的this上下文
  onClickBad(this: Handler, e: Event) {
    // oops, used this here. using this callback would crash at runtime
    this.info = e.message;
  }
}
let h = new Handler();
uiElement.addClickListener(h.onClickBad); // error!
```

### 声明合并（扩展 Vue 声明）

> 来看看使用场景，扩展 vue，在 vue 上添加全局的属性。

```typescript
// vue的声明在 vue/types/vue.d.ts

declare module "vue/types/vue" {
  // 相当于Vue.$eventBus
  interface Vue {
    $eventBus: Vue;
  }

  // 相当于在Vue.prototype.$eventBus
  interface VueConstructor {
    $eventBus: Vue;
  }
}
```

### 忽略 TypeScript 的校验

```typescript
Use; // @ts-check to opt in to type checking for a single file.
Use; // @ts-nocheck to opt out of type checking for a single file.
Use; // @ts-ignore to opt out of type checking for a single line.
```

### 总结

> TypeScript 声明还有很多高级的用法，目前我也没有用到那么多，在我纠结不会写声明的时候，我就会看看别人的声明文件时怎么写的。

注意：尽量不要把解构和声明写在一起，可读性极差。

```typescript
class Node {
  onNodeCheck(
    checkedKeys: any,
    {
      // 解构
      checked,
      checkedNodes,
      node,
      event,
    }: {
      // 声明
      node: any;
      [key: string]: any;
    }
  ) {}
}
```
