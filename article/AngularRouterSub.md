```typescript
private router: Router,

// 声明订阅对象
rooterChange: Subscription;

ngOnDestroy() {
    if (this.rooterChange) {
        this.rooterChange.unsubscribe();
    }
}

/**
 * 监听路由变化
 */
listenRouterChange() {
    this.rooterChange = this.router.events.subscribe((event) => {
        if (event instanceof NavigationEnd) {
            //...
        }
    });
}
```

## 获取路由名称

```typescript
constructor(private route: ActivatedRoute, private router: Router) { }
this.route.url.subscribe(url => {
    this.activeRouteName = url[0].path;
});
// 或者
this.router.url
```
