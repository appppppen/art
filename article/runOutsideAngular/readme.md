### 前言：
> 在模板中有实时获取一个变量，模板中就频繁的更新显示。那么频繁的变动将造成性能损耗。
> 或者在双向绑定时，异步事件的发生会导致组件中的数据变化，但是你想要适当时机在改变模板显示。

### NgZone

> NgZone 是基于 Zone 来实现的,NgZone 从 Zone 中 fork 了一份实例,是 Zone 派生出的一个子 Zone,在 Angular 环境内注册的异步事件都运行在这个子 Zone 上.

```typescript
/* 在Angular源码中，有一个ApplicationRef类，其作用是用来监听ngZone中的onTurnDone事件，不论何时只要触发这个事件，那么将会执行
一个tick()方法用来告诉Angular去执行变化监测。*/

// very simplified version of actual source
class ApplicationRef {
  changeDetectorRefs:ChangeDetectorRef[] = [];

  constructor(private zone: NgZone) {
    this.zone.onTurnDone
      .subscribe(() => this.zone.run(() => this.tick());
  }

  tick() {
    this.changeDetectorRefs
      .forEach((ref) => ref.detectChanges());
  }
}
```

> 知道了 Angular 环境内注册的异步事件都运行在这个 NgZone 上.下面使用一下 runOutsideAngular 方法

下面是一个 DEMO

```typescript
import {Component, NgZone} from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: '<p>Progress: {{progress}}%</p>
   <p *ngIf="progress >= 100">Done processing {{label}} of Angular zone!</p>

    <button (click)="processWithinAngularZone()">Process within Angular zone</button>
     <button (click)="processOutsideOfAngularZone()">Process outside of Angular zone</button>
',
})

export class AppComponent {
  progress: number = 0;
 label: string;
  constructor(private _ngZone: NgZone) {}
  processWithinAngularZone() {
    this.label = 'inside';
     this.progress = 0;
      this._increaseProgress(() => console.log('Inside Done!'));
    }
  processOutsideOfAngularZone() {
       this.label = 'outside';
      this.progress = 0;
       this._ngZone.runOutsideAngular(() => {
          this._increaseProgress(() => {
             // reenter the Angular zone and display done
            this._ngZone.run(() => {console.log('Outside Done!') });
          })});

  }

  _increaseProgress(doneCallback: () => void) {
    this.progress += 1;
    console.log(`Current progress: ${this.progress}%`);

    if (this.progress < 100) {
        window.setTimeout(() => this._increaseProgress(doneCallback), 10)
     } else {
       doneCallback();
     }
   }
}
```

点击 Process within Angular zone 会因为双向绑定 Progress 会从 1%持续变到 100%.
而点击 Process outside of Angular zone 只会显示 1%直接到 100%.

### 结论

> runOutsideAngular() 使你的上下文不会触发 Angular 跟踪变化。 如果你想继续跟踪变化，使用 run()方法即可让 Angular 重新跟踪变化。
