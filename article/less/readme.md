### less 中解决 CSS3 的 calc 冲突问题

最近一直在用 less，基本上还挺好用的。但使用 CSS3 的 calc 函数时却发现个冲突问题。

### 问题

```less
.class {
  height: calc(100% - 50px);
}
```

谁知道 less 自己就把它当表达式计算掉了，导致到浏览器那变成了 calc(50%) o(╯□╰)o

### 解决方法

解决方法也挺简单，加上~用""包起来。

```less
.class {
  height: calc(~"100% - 50px");
}
```

可如果要用变量怎么用呢？也不复杂，像下面这样就搞定啦。

```less
.class {
  @cap: 50px;
  height: calc(~"100% - @{cap}");
}
```
