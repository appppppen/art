Angular2路由器(路由+导航)

这里给大家简单的总结下Angular2路由器相关的一些信息。但是我一向认为最好的文档还是官方文档，还是推荐大家参考官方文档[https://www.angular.cn/guide/router#displaying-multiple-routes-in-named-outlets](https://www.angular.cn/guide/router#displaying-multiple-routes-in-named-outlets) 

路由器简单来说就是让用户从一个视图导航到另一个视图。
### 一、路由器相关对象介绍
如果在我们的Angular应用里面想使用路由，首先我们需要做两件事情。

*在index.html的 <head> 标签下添加一个 <base> 元素，来告诉路由器该如何合成导航用的URL。通常在index.html的head部分添加如下代码。
``` xml
<base href="/">
```
*在AppModule里面的imports里面导入RouterModule。
#### 1.1、路由器-Router
每个带路由的Angular应用都有一个Router(路由器)服务的单例对象。当浏览器的URL变化时，路由器会查找对应的Route(路由)，并据此决定该显示哪个组件。路由器需要先配置路由才会有路由信息。

>路由器-Router就是为激活的URL显示相应的组件。管理从一个组件到另一个组件的导航。

路由器必须需要先配置路由信息。并用 RouterModule.forRoot方法来配置路由器，并把它的返回值添加到AppModule的imports数组中。
#### 1.2、路由数组-Routes
路由配置的时候需要先定义一个路由数组。路由数组描述如何进行导航。每个Route都会把一个URL的path 映射到一个组件。 注意，path 不能以斜杠（/）开头。 路由器会为解析和构建最终的 URL，这样当你在应用的多个视图之间导航时，可以任意使用相对路径和绝对路径。

>定义路由数组之后需要调用RouterModule.forRoot()方法把路由数组配置进入路由器里面去。

比如如下的代码Heroes模块对应的路由模块里面heroesRoutes就一个路由数组，然后我们调用RouterModule.forChild(heroesRoutes)把他配置到路由里面去来。
``` typescript
import {NgModule} from '@angular/core';
import {CommonModule} from '@angular/common';
import {RouterModule, Routes} from '@angular/router';
import {HeroListComponent} from './hero-list.component';
import {HeroDetailComponent} from './hero-detail.component';

const heroesRoutes: Routes = [
    {path: 'heroes', component: HeroListComponent},
    {path: 'hero/:id', component: HeroDetailComponent}
];

@NgModule({
    imports: [
        CommonModule,
        RouterModule.forChild(heroesRoutes)
    ],
    declarations: [],
    exports: [
        RouterModule
    ]
})
export class HeroesRoutingModule {
}

```
>关于路由Route对象属性说明，下面还会有更加详细的介绍

属性|		类型|		说明
------|	-----------------|	-----------------------
path|		`string`|		路由地址(不能以'/'开头)
pathMatch	|	`string`|		指定路由匹配策略的字符串(pathMatch有两个值:full、prefix)
matcher	|	`UrlMatcher`|		自定义路径匹配，取代自定义策略path和pathMatch
component	|	`Type<any>`|		路由映射的组件
redirectTo	|	`string`|		路由重定向地址
outlet|		`string`|		路由出口名称`<router-outlet>`标签里面name字段名称
canActivate|		`any[]`|		路由守卫接口，控制是否允许进入路由
canActivateChild|		`any[]`	|	路由守卫接口，控制是否允许进入子路由
canDeactivate|		`any[]`|		路由守卫接口，控制是否允许离开路由
canLoad|		`any[]`	|	路由守卫接口，控制是否允许延迟加载整个模块
data|		`Data`|		用来存放于路由有关的任意信息(组件需要的附加数据)
resolve|		`ResolveData`|		用于在路由激活之前获取路由数据
children|		`Routes`	|	子路由数组
loadChildren	|	`LoadChildren`|		是对延迟加载子路由的引用
runGuardsAndResolvers	|	`RunGuardsAndResolvers`|	


#### 1.3、路由出口 - RouterOutlet
有路由了，也把路由数组配置到路由里面去了。但是我们的路由对应的组件在哪里展示呢。这个时候就需要路由出口了。路由出口对应模板里面的```<router-outlet>``` 标签。路由出口是路由器渲染的地方，用来标记路由器该在哪里显示对应的视图。

#### 1.4、路由器链接-Router links
路由器链接把可点击的HTML元素绑定到某个路由。点击带有 routerLink指令(绑定到字符串或链接参数数组)的元素时就会触发一次导航。比如下面的代码，我们就定义了两个路由链接crisis-center、heroes。

``` xml
template: `
  <h1>Angular Router</h1>
  <nav>
    <a routerLink="/crisis-center" routerLinkActive="active">Crisis Center</a>
    <a routerLink="/heroes" routerLinkActive="active">Heroes</a>
  </nav>
  <router-outlet></router-outlet>
`
```

>还是上述代码中，RouterLinkActive指令会基于当前的RouterState对象来为激活的 RouterLink 切换CSS类。 他会一直沿着路由树往下进行级联处理，所以父路由链接和子路由链接可能会同时激活。 要改变这种行为，可以把 [routerLinkActiveOptions] 绑定到 {exact: true} 表达式。 如果使用了 { exact: true }，那么只有在其URL与当前URL精确匹配时才会激活指定的RouterLink。

``` xml
<a routerLink="/crisis-center" routerLinkActive="active" 
  [routerLinkActiveOptions]="{exact: true}">Bob</a>
```
#### 1.5、路由器状态 - Router state
在导航时的每个生命周期成功完成时，路由器会构建出一个ActivatedRoute 组成的树，它表示路由器的当前状态。 你可以在应用中的任何地方用Router 服务及其routerState属性来访问当前的RouterState值。

#### 1.6、激活的路由 - ActivatedRoute
  ActivatedRoute是为每个路由组件提供提供的一个服务，它包含特定于路由的信息，比如路由参数、静态数据、解析数据、全局查询参数和全局碎片（fragment）。它有一大堆有用的信息，包括：

  属性|	说明
  ----|---------
url	|路由路径的 
Observable| 对象，是一个由路由路径中的各个部分组成的字符串数组
data|	一个 Observable，其中包含提供给路由的 data 对象。也包含由解析守卫（resolve guard）解析而来的值
paramMap|		一个 Observable，其中包含一个由当前路由的必要参数和可选参数组成的map对象。用这个 map 可以获取来自同名参数的单一值或多重值
queryParamMap|		一个 Observable，其中包含一个对所有路由都有效的查询参数组成的map对象。 用这个 map 可以获取来自查询参数的单一值或多重值
fragment|		一个适用于所有路由的 URL 的 fragment（片段）的 Observable
outlet|		要把该路由渲染到的 RouterOutlet 的名字。对于无名路由，它的路由名是 primary，而不是空串
routeConfig|		用于该路由的路由配置信息，其中包含原始路径
parent|		当该路由是一个子路由时，表示该路由的父级 ActivatedRoute
firstChild|		包含该路由的子路由列表中的第一个 ActivatedRoute
children|		包含当前路由下所有已激活的子路由

比如如下的代码，当路由导航到HeroDetailComponent组件的时候，我们通过ActivatedRoute从路由里面获取id。

``` typescript
import {Component, HostBinding, OnInit} from '@angular/core';
import {slideInDownAnimation} from '../animations';
import {Observable} from 'rxjs';
import {Hero, HeroService} from './hero.service';
import {ActivatedRoute, Router} from '@angular/router';

@Component({
    selector: 'app-hero-detail',
    templateUrl: './hero-detail.component.html',
    styleUrls: ['./hero-detail.component.css'],
    animations: [slideInDownAnimation]
})
export class HeroDetailComponent implements OnInit {

    @HostBinding('@routeAnimation') routeAnimation = true;
    @HostBinding('style.display') display = 'block';
    @HostBinding('style.position') position = 'absolute';

    hero$: Observable<Hero>;

    constructor(
        private route: ActivatedRoute,
        private router: Router,
        private service: HeroService
    ) {
    }

    ngOnInit() {
        // 一进来界面就先获取id对应的Hero 有两种方式
        // 第一种(最保险的方式)
        // this.hero$ = this.route.paramMap.pipe(
        //     switchMap((params: ParamMap) =>
        //         this.service.getHero(params.get('id')))
        // );
        // 第二种方式，快照的方式
        const id = this.route.snapshot.paramMap.get('id');
        this.hero$ = this.service.getHero(id);
    }

    gotoHeroes(hero: Hero) {
        const heroId = hero ? hero.id : null;
        // id 和 foo 不会影响到/heroes的路由
        this.router.navigate(['/heroes', {id: heroId, foo: 'foo'}]);
    }

}

```
#### 1.7、路由事件
在每次导航中，Router都会通过Router.events 属性发布一些导航事件。这些事件的范围涵盖了从开始导航到结束导航之间的很多时间点。下表中列出了全部导航事件：
>Router.events是一个Observable对象

路由器事件|	说明|
----------|---------------
NavigationStart|	事件会在导航开始时触发|
RoutesRecognized|	事件会在路由器解析完 URL，并识别出了相应的路由时触发|
RouteConfigLoadStart|

事件会在 Router 对一个路由配置进行惰性加载之前触发
RouteConfigLoadEnd | 事件会在路由被惰性加载之后触发
NavigationEnd | 事件会在导航成功结束之后触发
NavigationCancel | 事件会在导航被取消之后触发。 这可能是因为在导航期间某个路由守卫返回了 false
NavigationError | 事件会在导航由于意料之外的错误而失败时触发

 比如如下的代码，教我们怎么来获取路由事件

 ``` typescript
import {Component, HostBinding} from '@angular/core';
import {slideInDownAnimation} from '../animations';
import {NavigationCancel, NavigationEnd, NavigationError, NavigationStart, Router, RoutesRecognized} from '@angular/router';
import {filter} from 'rxjs/operators';

@Component({
    selector: 'app-crisis-detail',
    templateUrl: './crisis-detail.component.html',
    styleUrls: ['./crisis-detail.component.css'],
    animations: [slideInDownAnimation]
})
export class CrisisDetailComponent {

    @HostBinding('@routeAnimation') routeAnimation = true;
    @HostBinding('style.display') display = 'block';
    @HostBinding('style.position') position = 'absolute';

    constructor(
        private router: Router
    ) {
        // 监听路由变化
        router.events.subscribe(event => {
            if (event instanceof NavigationStart) {
                console.log('NavigationStart');
            } else if (event instanceof NavigationEnd) {
                console.log('NavigationEnd');
            } else if (event instanceof NavigationCancel) {
                console.log('NavigationCancel');
            } else if (event instanceof NavigationError) {
                console.log('NavigationError');
            } else if (event instanceof RoutesRecognized) {
                console.log('NavigationError');
            }
        });

        this.router.events.pipe(
            filter((event: Event) => event instanceof NavigationEnd)
        ).subscribe(x => console.log(x));
    }

}

 ```

#### 1.8、路由动画

### 二、路由守卫
路由守卫简单来说就是通过机制来控制那些路由可以被那些用户访问。说白了就会特定的路由需要有特定的权限才能访问。路由的守卫可以返回一个 Observable<boolean> 或 Promise<boolean>，并且路由器会等待这个可观察对象被解析为 true 或 false。true表明有权限访问，false表示无权限访问。

路由守卫提供的守卫接口有：

|    守卫接口   |          解释         |
| ------------- | ---------------------------|
|CanActivate|控制是否允许进入路由|
|CanActivateChild | 控制是否允许进入子路由|
|CanDeactivate|控制是否允许离开路由|
|Resolve | 处理再在路由激活之前获取路由数据的情况|
|CanLoad | 控制是否允许延迟加载整个模块|

 在分层路由的每个级别上，可以设置多个守卫。 路由器会先按照从最深的子路由由下往上检查的顺序来检查 CanDeactivate()和 CanActivateChild()守卫。 然后它会按照从上到下的顺序检查CanActivate()守卫。 如果特性模块是异步加载的，在加载它之前还会检查CanLoad()守卫。 如果任何一个守卫返回false，其它尚未完成的守卫会被取消，这样整个导航就被取消了。

路由守卫一般在路由配置里面配置，可以参考实例里面admin-routing.module.ts文件的路由配置。关于路由守卫所有的例子可以参考文章末尾DEMO里面的admin模块里面的内容。
#### 2.1、CanActivate
CanActivate用来处理进入当前路由的情况，返回true允许进入，返回false则不允许进入。对应路由配置里面的canActivate字段，他的配置可以是一个数组一旦某个返回false就认为认证失败，则不允许进入当前路由。

#### 2.1、CanActivateChild
CanActivateChild用来处理进入子路由的情况，会在任何子路由被激活之前运行。和CanActivate的用法一样，一个控制当前路由，一个控制子路由。
#### 2.1、CanDeactivate
CanDeactivate用来处理从当前路由离开的情况。比如我们可以控制是允许离开当前路由，或者在离开之前给出提示啥的。

#### 2.1、Resolve
想想当路由切换的时候，被路由的页面中的元素(标签)就会立马显示出来，同时，数据会被准备好并呈现出来。但是注意，数据和元素并不是同步的，在没有任何设置的情况下，AngularJS默认先呈现出元素，而后再呈现出数据。这样就会导致页面会被渲染两遍，导致“页面UI抖动”的问题，对用户不太友好。resolve就是来解决了这个问题的。Resolve守卫主要是在路由激活之前获取路由数据，预先加载数据。

具体可以参考文章末尾DEMO里面crisis-detail.component.ts文件CrisisDetailComponent组件的使用，注意crisis-center-routing.module.ts里面路由配置。

#### 2.1、CanLoad

CanLoad控制是否允许延迟加载整个模块。说白了就是只有通过了我的判断你才能娶加载整个模块。具体可以参见文章末尾DEMO里面app-routing.module.ts里面路由数组里面admin路径的配置。

最后也给出根据官方文档敲出来的例子的下载地址[DEMO](https://github.com/tuacy/AngularRouterExample)下载地址。关于路由咱们就先说这么多，其实路由里面的东西咱们要学的还是很多的，咱也是个初学者。就当做个笔记了。还是强烈推荐大家去看官方文档 https://www.angular.cn/guide/router#milestone-5-route-guards

作者：tuacy
链接：https://www.jianshu.com/p/9b6f91184764
来源：简书