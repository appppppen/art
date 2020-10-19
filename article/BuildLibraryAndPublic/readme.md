## 1. 前言

Angular CLI 7.0.0 增加了一个令人兴奋的脚手架配置项：--create-application，其中默认值是 true，如果不设置，则在新工作空间的 src 文件夹中创建一个新的初始应用程序项目。如果为 false，则创建一个没有初始应用程序的空工作区。想了解更多配置项，动起你的小鼠标用力点击这里。

## 2. 第 1 步：创建 Library 库工作空间

Angular CLI 7.0.0 的键入以下命令：

```
ng new foo-lib --create-application=false
```

这个时候我们会看到项目文件中的一些变化：

-   package.json

angular 需要的所有常用依赖项

-   angular.json

Angular 配置文件，但没有项目

-   README.md、tsconfig.json、tslint.json、node_modules

基本和我们的构建初始化项目的内容结构一致

等等，这...，你会惊奇的发现项目目录中竟然没有 src 目录！别着急，因为我们使用--create-application=false 所以创建的应用是空的。

## 3. 第 2 步：初始化 Library 库项目

```
ng generate library foo-lib --prefix=foo
```

其中--prefix 指令是用于初始项目的时候生成选择器（ng genreate）的前缀。详细配置项请看前言部分的超链接。如果你不指定，默认是 lib。

执行完命令之后，我们发现项目中多了一个 project 文件夹，里边有个 Library 工程：foo-lib。

## 4. 第 3 步：创建 Library 库测试项目

我们需要一个可以用来调用我们的 Angular 库的项目，键入以下命令：

```
ng generate application foo-tester
```

执行完命令之后，我们可以看到，project 文件下又多出了一个文件夹：foo-tester，即我们的测试项目。另外，Angular CLI 还添加了一个 foo-tester-e2e 项目，用于端到端测试。对于不写测试用例.spec 的强迫症患者拯救大心丸：--minimal=true。

## 5. 第 4 步：开发 Library 和测试 Library

## 6.第 5 步：构建打包 Library

Angular CLI 从 6.1 开始，始终在生产模式下构建库，因此我们不使用--prod,只需键入以下命令：

```
ng build foo-lib
```

### 以下为温馨提醒：

> 如果想构建自己的测试项目则键入以下命令：

```
ng build foo-tester --prod
```

和构建 Library 库不一样的是，构建测试应用必须指定：--prod。

> 如果想启动自己的测试项目，则键入以下命令：

```
ng serve foo-tester
```

> 如果想测试自己的 Library，则键入以下命令：

```
ng test foo-lib
```

> 如果想测试自己的测试项目，则键入以下命令：

```
ng test foo-tester
```

## 7. 第 6 步：发布我们自己的 Library

如果想发布到 npm，则需注册一个自己的 npm 账号，如果已经有了且已经登录，

```
npm login // 用户密码在控制台会隐藏，且光标未变化，正常输入密码确认即可。
```

则键入以下命令：在打包好的文件夹下

```
npm publish
```

## 8. 第 7 步：使用我们的 Library

和其他第三方包一样，只需要 npm install 你的自己发布的 Library 包即可，项目根目录终端键入以下命令：

```
npm i -S foo-lib
```

这个时候你会看到你的项目 package.json 中的 dependencies 依赖项中增加了一项：foo-lib。然后在 Angular 模块中引入即可。

## 9. 第 8 步：最后的惊喜，如果维护自己的 Library

npm 发布版本有些注意事项，每次构建发布需要注意以下规则：

```typescript
// 1.npm插件发布
npm addUser  // 分别输入用户名、密码、邮箱
npm publish  // 直接发布
npm login    // 非第一次发版本则用此命令
npm unpublish --force // 取消插件发布【谨慎使用】
npm deprecate <pkg>[@<version>] <message> // 并不会在社区里撤销你已有的包，但会在任何人尝试安装这个包的时候得到警告
npx force-unpublish package-name '原因描述' // 撤销不了？？试试这个

// 2.npm插件更新
npm version patch  // 补丁【1.0.1】
npm version minor  // 小改【1.1.0】
npm version major  // 大改【2.0.0】
                   // 注意需要再一次执行：npm publish

// 3.查看远程包版本信息
npm view xxx versions

// 4.npm查看本地全局安装过的包
npm list -g --depth=0

// 5.npm查看全局的包的安装路径
npm root -g

// 6.npm查看当前包的安装路径
npm root

// 7.npm将包安装到全局环境中
npm install xxx -g

// 8.npm将信息写入package.json，并自动把模块和版本号添加到dependencies部分
npm install xxx –save
npm i -S xxx // 简写版本

// 9.npm将信息写入package.json，并自动把模块和版本号添加到devdependencies部分
npm install xxx –save-dve
npm i -D xxx // 简写版本

// 10.npm单独更新某个包
npm update xxx

// 11.npm更新至最新版
npm install -g npm

// 12.npm淘宝镜像
npm config set registry http://registry.npm.taobao.org
```

## 解决 bug

> ERROR: Trying to publish a package that has been compiled by Ivy. This is not allowed. Please delete and rebuild the package, without compiling with Ivy, before attempting to publish.

### 解决方案

Within the library project there is a `tsconfig.lib.json` file. Here's my Angular compiler configuration:

```
"angularCompilerOptions": {
    "skipTemplateCodegen": true,
    "strictMetadataEmit": true,
    "enableResourceInlining": true,
    "enableIvy": false
  },
```
