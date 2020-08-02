## Httpclient 引入

### 模块引入

```typescript
import { NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";
import { HttpClientModule } from "@angular/common/http";

@NgModule({
  imports: [
    BrowserModule,
    // import HttpClientModule after BrowserModule.
    HttpClientModule,
  ],
  declarations: [AppComponent],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

### httpclient 注入

```typescript
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";
@Injectable()
export class ConfigService {
  constructor(private http: HttpClient) {}
}
```

## get 请求

### ts 定义数据类型

```typescript
export interface Config {
  heroesUrl: string;
  textfile: string;
}
```

### 定义 service

```typescript
configUrl = 'assets/config.json';

getConfig() {
  // now returns an Observable of Config
  return this.http.get<Config>(this.configUrl);
}
//等效于
getConfig():Observable<Config> {
  // now returns an Observable of Config
  return this.http.get<Config>(this.configUrl);
}

```

### 调用 service

```typescript
config: Config;
 showConfig() {
  this.configService.getConfig()
    .subscribe((data: Config) => this.config = {
        heroesUrl: data['heroesUrl'],
        textfile:  data['textfile']
    });
	// or
	 .subscribe((data: Config) => this.config = { ...data });
}

```

### 修改返回数据类型

```typescript
getConfigResponse(): Observable<HttpResponse<Config>> {
  return this.http.get<Config>(
    this.configUrl, { observe: 'response' });
}

```

- 返回类型修改 HttpResponse< Config >
- 请求参数添加，{ observe: 'response' }

```typescript
showConfigResponse() {
  this.configService.getConfigResponse()
    // resp is of type `HttpResponse<Config>`
    .subscribe(resp => {
      // display its headers
      const keys = resp.headers.keys();
      this.headers = keys.map(key => `${key}: ${resp.headers.get(key)}`);
      // access the body directly, which is typed as `Config`.
      this.config = { ... resp.body };
    });
}

```

### 返回文本

```typescript
getTextFile(filename: string) {
  // The Observable returned by get() is of type Observable<string>
  // because a text response was specified.
  // There's no need to pass a <string> type parameter to get().
  return this.http.get(filename, {responseType: 'text'})
    .pipe(
      tap( // Log the result or error
        data => this.log(filename, data),
        error => this.logError(filename, error)
      )
    );
}

```

### jsonp 请求

```typescript
import { NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";
import { HttpClientModule } from "@angular/common/http";
import { HttpClientJsonpModule } from "@angular/common/http";
@NgModule({
  imports: [
    BrowserModule,
    // import HttpClientModule after BrowserModule.
    HttpClientModule,
    HttpClientJsonpModule,
  ],
  declarations: [AppComponent],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

```typescript
searchHeroes(term: string): Observable {  // 利用subscribe订阅成功数据
  term = term.trim();
  let heroesURL = `${this.heroesURL}?${term}`;
  return this.http.jsonp(heroesUrl, 'callback').pipe(
      catchError(this.handleError('searchHeroes', []) // then handle the error
    );
};

```

## post 请求

### 添加数据

```typescript
addHero (hero: Hero): Observable<Hero> {
  return this.http.post<Hero>(this.heroesUrl, hero, httpOptions).pipe(
      catchError(this.handleError('addHero', hero))
    );
}

```

### 调动触发

```typescript
this.heroesService.addHero(newHero).subscribe((hero) => this.heroes.push(hero));
```

## put 请求

### 更新数据

```typescript
updateHero (hero: Hero): Observable<Hero> {
  return this.http.put<Hero>(this.heroesUrl, hero, httpOptions).pipe(
      catchError(this.handleError('updateHero', hero))
    );
}
```

### 调用更新

```typescript
this.heroesService.updateHero(hero);
```

## delete 请求

### 删除数据

```typescript
deleteHero (id: number): Observable<{}> {
  const url = `${this.heroesUrl}/${id}`; // DELETE api/heroes/42
  return this.http.delete(url, httpOptions).pipe(
      catchError(this.handleError('deleteHero'))
    );
}
```

### 方法调用触发请求

```typescript
this.heroesService.deleteHero(hero.id).subscribe();
```

# 拦截器配置

## headers 配置

### 初始化

```typescript
import { HttpHeaders } from "@angular/common/http";
const httpOptions = {
  headers: new HttpHeaders({
    "Content-Type": "application/json",
    Authorization: "my-auth-token",
  }),
};
```

### 更新 header

```typescript
httpOptions.headers = httpOptions.headers.set(
  "Authorization",
  "my-new-auth-token"
);
```

## 错误处理

### pipe 提前处理

```typescript
getConfig() {
  return this.http.get<Config>(this.configUrl).pipe(
      catchError(this.handleError)
    );
}
```

```typescript
getConfig() {
  return this.http.get<Config>(this.configUrl).pipe(
      retry(3), // retry a failed request up to 3 times
      catchError(this.handleError) // then handle the error
    );
}
```

### subscribe 处理

```typescript
showConfig() {
  this.configService.getConfig().subscribe(
      (data: Config) => this.config = { ...data }, // success path
      error => this.error = error // error path
    );
}
```
