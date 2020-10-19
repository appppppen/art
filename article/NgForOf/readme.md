[官方文档](https://angular.cn/api/common/NgForOf)，查看“变更的传导机制”即有详细的表述：

> 即使数据没有变化，迭代器中的元素标识符也可能会发生变化。比如，如果迭代器处理的目标是通过 RPC 从服务器取来的， 而 RPC 又重新执行了一次。那么即使数据没有变化，第二次的响应体还是会生成一些具有不同标识符的对象。Angular 将会清除整个 DOM， 并重建它（就仿佛把所有老的元素都删除，并插入所有新元素）。这是很昂贵的操作，应该尽力避免。

那么如何来避免呢？

> 要想自定义默认的跟踪算法，NgForOf 支持 trackBy 选项。trackBy 接受一个带两个参数（index 和 item）的函数。 如果给出了 trackBy，Angular 就会使用该函数的返回值来跟踪变化。

### 例子

```xml
<a (click)="add()">添加</a>
<ul>
  <li
    *ngFor="let i of arr; index as ii; trackBy: trackFunc">
    {{i.id}} / {{ i.name }}
   </li>
</ul>
```

```typescript
arr = [
  { id: 1, name: "a" },
  { id: 2, name: "b" },
  { id: 3, name: "c" },
  { id: 4, name: "e" },
];
...

trackFunc(index: number, item: any) {
    // 改变这里查看页面dom刷新状况
    // return index;
    // return 'xxx';
    // return item.id;
    return item.name;
}

add() {
    this.arr.unshift({
        id: 4, // 新加的元素id与开始定义的最后一个元素id相同，请注意！
        name: Math.random().toString() // 新加的元素name是随机字符
    });
}
```

以不停的往数组开始插入元素，查看页面刷新的情况。
| | |
|-----------------|-----------------------|
return index; | 刷新全部 li
return item.id; | 只会刷新 id 相同的 li
return item.name; | 只会刷新 name 变化的 li
return 'xxx'; | 情况跟 return index 一样，刷新全部的 li

通过以上的实践可知：

- return index: 数组索引变化触发刷新。
- return item.id: 最后个元素 Id 与新增的元素 id 相同，但也在刷新；但 id 为 1，2，3 的元素，从未刷新；似乎这里只能从源码中一探究竟。
- return item.name: name 属性变化触发了刷新。
- return 'xxx': 返回与 index 和 item 都不相关的固定值，也会触发全部刷新。

抛开第二点得出一个猜测结果：使用 trackBy 的好处是自定义返回跟踪结果，以比对上次的跟踪结果，如果不一样，那么就刷新变化的页面实例（减少不必要的 dom 刷新而带来性能的提升）。
