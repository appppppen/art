依赖注入(DI) 是 Angular 2 的核心，在深入了解 DI 的工作原理之前，我们必须先搞清楚 Provider 的概念。
![](https://upload-images.jianshu.io/upload_images/12890819-eddcdfc489bfc993.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在 Angular 2 中我们使用 Provider 来描述与 Token 关联的依赖对象的创建方式。Angular 2 中依赖对象的创建方式有四种，它们分别是：

- useClass
- useValue
- useExisting
- useFactory

### useClass

```typescript
@Injectable()
export class ApiService {
   constructor(
      public http: Http,
      public loadingCtrl: LoadingController) {
   }
   ...
}

@NgModule({
  ...
  providers: [
       { provide: ApiService, useClass: ApiService } // 可使用简洁的语法，即直接使用ApiService
  ]
})
export class CoreModule { }
```

### useValue

```typescript
{ provide: 'API_URL', useValue: 'http://my.api.com/v1' }
```

### useExisting

```typescript
{ provide: 'ApiServiceAlias', useExisting: ApiService }
```

### useFactory

```typescript
export function configFactory(config: AppConfig) {
  return () => config.load();
}

@NgModule({
  ...
  providers: [
       { provide: APP_INITIALIZER, useFactory: configFactory,
        deps: [AppConfig], multi: true }
  ]
})
export class CoreModule { }
```

## 使用 Provider 的正确姿势

1. 创建 Token

Token 的作用是用来标识依赖对象，Token 值可能是 Type、InjectionToken、OpaqueToken 类的实例或字符串。通常不推荐使用字符串，因为如果使用字符串存在命名冲突的可能性比较高。在 Angular 4.x 以前的版本我们一般使用 OpaqueToken 来创建 Token，而在 Angular 4.x 以上的版本版本，推荐使用 InjectionToken 来创建 Token 。详细的内容可以参考， [如何解决 Angular 2 中 Provider 命名冲突。](https://segmentfault.com/a/1190000008626348)

2. 根据实际需求选择依赖对象的创建方式，如 useClass 、useValue、useExisting、useFactory

3. 在 NgModule 或 Component 中注册 providers

4. 使用构造注入的方式，注入与 Token 关联的依赖对象

```typescript
/**
* 封装Http服务，如在每个Http的请求头中添加token，类似于Ng1.x中的拦截器
*/
@Injectable()
export class ApiService {
   constructor(
   // 注入Angular 2 中的Http服务，与Ng1.x的区别：
   // 在Ng1.x中调用Http服务后，返回Promise对象
   // 在Ng2.x中调用Http服务后，返回Observable对象
      public http: Http) {
   }
   ...
}

/**
* AppModule
*/
@NgModule({
  ...
  providers: [
       { provide: ApiService, useClass: ApiService } // 可使用简洁的语法，即直接使用ApiService
  ]
})
export class AppModule { }

/**
* 系统首页
*/
@Component({
  selector: 'page-home',
  templateUrl: 'home.html'
})
export class HomePage {
  constructor(
    public apiService: ApiService // 使用构造注入的方式，注入ApiService的实例对象
  ) { }

  ngOnInit(): void {
    this.apiService.get(HOME_URL) // 获取首页相关的数据
    .map(res => res.json()) // 返回的res对象是Response类型的实例
    .subscribe(result => {
      ...
    })
  }
}
```

## 我有话说

1.当 DI 解析 Providers 时，都会对提供的每个 provider 进行规范化处理，即转换成标准的形式。

```javascript

function _normalizeProviders(providers: Provider[], res: Provider[]): Provider[] {
  providers.forEach(b => {
    if (b instanceof Type) { // 支持简洁的语法，转换为标准格式
      res.push({provide: b, useClass: b});
    } else if (b && typeof b == 'object' && (b as any).provide !== undefined) {
      res.push(b as NormalizedProvider);
    } else if (b instanceof Array) {
      _normalizeProviders(b, res); // 如果是数组，进行递归处理
    } else {
      throw invalidProviderError(b);
    }
  });
  return res;
}


```

2.创建 Token 时为了避免命名冲突，尽量避免使用字符串作为 Token。

3.若要创建模块内通用的依赖对象，需要在 NgModule 中注册相关的 provider，若在每个组件中，都有唯一的依赖对象，就需要在 Component 中注册相关的 provider。

4.multi providers 的具体作用，具体请参考 - Angular2 Multi Providers

5.Provider 是用来描述与 Token 关联的依赖对象的创建方式。当我们使用 Token 向 DI 系统获取与之相关连的依赖对象时，DI 会根据已设置的创建方式，自动的创建依赖对象并返回给使用者。

Provider 接口

```typescript
export interface ClassProvider {
  // 用于设置与依赖对象关联的Token值，Token值可能是Type、InjectionToken、OpaqueToken的实例或字符串
  provide: any;
  useClass: Type<any>;
  // 用于标识是否multiple providers，若是multiple类型，则返回与Token关联的依赖对象列表
  multi?: boolean;
}

export interface ValueProvider {
  provide: any;
  useValue: any;
  multi?: boolean;
}

export interface ExistingProvider {
  provide: any;
  useExisting: any;
  multi?: boolean;
}

export interface FactoryProvider {
  provide: any;
  useFactory: Function;
  deps?: any[]; // 用于设置工厂函数的依赖对象
  multi?: boolean;
}
```

## 总结

在文章的最后，想举一个现实生活中的例子，帮助初学者更好地理解 Angular 2 DI 和 Provider。

Provider 中的 token 可以理解为菜名，useClass、useValue 可以理解为菜的烹饪方式，而依赖对象就是我们所点的菜，而 DI 系统就是我们的厨师了。如果没有厨师，我们就得关心煮这道菜需要哪些原材料，怎么煮菜，重要的是还得自己煮，可想而知多麻烦。而有了厨师(DI)，我们只要在菜谱上点菜，必要时备注一下烹饪方式，不过多久香喷喷的菜就上桌鸟~~~。
