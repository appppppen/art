今天来说说如何书写好的 commit message。

Commit message 大家应该每天都在写，每次使用 Git 提交代码时，我们都会在执行 git commit 命令时附带上一句话来简要描述此次提交的改动。

Commit message 看似简单，作用却很重要。在阅读代码时，可以通过 commit message 了解到作者编写某行代码时的背景；调查 bug 时可以通过搜索 commit message 快速定位相关的提交记录。

那么什么样的 commit message 才算好的 commit message 呢？

开源社区已经为我们总结出了一套名为 [Conventional Commits](https://conventionalcommits.org/#conventional-commits-100-beta2) 的书写规范。很多流行的开源项目都使用了这套规范，如 [Karma](http://karma-runner.github.io/2.0/index.html)，[Angular](https://angular.io/) 等。其规定的格式如下：

``` 
<type>[optional scope]: <description>

[optional body]

[optional footer]
```

下面我们分别来介绍一下其中的各个组成部分：

* type ：用于表明我们这次提交的改动类型，是新增了功能？还是修改了测试代码？又或者是更新了文档？开源社区目前总结出了以下 11 种类型：

* build：主要目的是修改项目构建系统(例如 glup，webpack，rollup 的配置等)的提交
* ci：主要目的是修改项目继续集成流程(例如 Travis，Jenkins，GitLab CI，Circle 等)的提交
* docs：文档更新
* feat：新增功能
* fix：bug 修复
* perf：性能优化
* refactor：重构代码(既没有新增功能，也没有修复 bug)
* style：不影响程序逻辑的代码修改(修改空白字符，补全缺失的分号等)
* test：新增测试用例或是更新现有测试
* revert：回滚某个更早之前的提交
* chore：不属于以上类型的其他类型
* optional scope：一个可选的修改范围。用于标识此次提交主要涉及到代码中哪个模块。根据项目实际情况填写即可，最好在项目中规定好模块列表，保持一致性。

* description：一句话描述此次提交的主要内容，做到言简意赅。

* **optional body** 和 **optional footer** 通常我们都不会用到，此处不再赘述。

介绍完了格式，让我们来看看真实世界中符合该规范的 commit message 长什么样子吧。

以 Angular 为例，来看看它最新的提交历史：

![](https://img20.360buyimg.com/uba/jfs/t20611/308/677480810/132585/940d5656/5b1525e6Nc605636d.png)

是不是很整洁？

现在规范是有了，但是还需要一个工具自动帮我们检测我们编写的 commit message 是否真的符合规范。这就好比是我们创建了一套 JavaScript 的语法规范，还需要 ESLint 这样的功能来自动帮我们检查一样。

对于 commit message 来说，这个工具就是 [commitlint](http://marionebl.github.io/commitlint/#/)。

## 在项目中启用 commitlint

和 ESLint 一样的是，commitlint 自身只提供了检测的功能和一些最基础的规则。使用者需要根据这些规则配置出自己的规范。

对于 Conventional Commits 规范，社区已经整理好了 @commitlint/config-conventional 包，我们只需要安装并启用它就可以了。

首先安装 commitlint 以及 conventional 规范：

``` 
npm install --save-dev @commitlint/cli @commitlint/config-conventional
```

接着创建 commitlint.config.js 文件，并写入以下内容：

``` 
module.exports = {
  extends: ['@commitlint/config-conventional'],
};
```

至此，commitlint 的配置就基本完成了。但目前我们还没有说何时来执行 commitlint。

答案显而易见，在每次执行 git commit 的时候咯~

为了可以在每次 commit 时执行 commitlint 检查我们输入的 message，我们还需要用到一个工具 —— husky。

husky 是一个增强的 git hook 工具。可以在 git hook 的各个阶段执行我们在 package.json 中配置好的 npm script。

首先安装 husky：

``` 
npm install --save-dev husky
```

接着在 package.json 中配置 commitmsg 脚本：

``` 
{
  "scripts": {
    "commitmsg": "commitlint -E GIT_PARAMS"
  }
}
```

至此就彻底配置完成啦。

让我们来感受一下：

![](https://img12.360buyimg.com/uba/jfs/t21337/289/681073221/88718/eac5bc39/5b15340bN8af4d42f.gif)

还在等什么？赶快在自己的项目中启用 commitlint 吧！
