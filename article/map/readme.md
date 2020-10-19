## TypeScript map用法和操作 – TypeScript开发教程

TypeScript map是在ES6版本的JavaScript中添加的一个新的数据结构。它允许我们将数据存储在键值对中，并记住与其他编程语言类似的键的原始插入顺序。在TypeScript映射中，我们可以使用任何值作为键或值。

### 创建map

我们可以创建一个map如下。
```typescript
var map = new Map();  
```

### map的操作方法
下面列出了TypeScript map方法。
 编号|方法 |描述
 -- | -- |--
 1. | map.set(key, value)	 | 它用于在map中添加条目。
 2. | map.get(key) | 它用于从map检索条目。如果键在map中不存在，则返回undefined。
 3. | map.has(key) | 如果键在map中存在，则返回true。否则，返回false。
 4. | map.delete(key) | 它用于通过键删除条目。
 5. |  map.size() | 它用于返回map的大小。
 6. |  map.clear() | 它删除了map上的所有内容。
### 例子
我们可以从下面的例子中理解map方法。

```typescript
let map = new Map();  
  
map.set('1', 'Oreja');     
map.set(1, 'www.srcmini.com');       
map.set(true, 'bool1');   
map.set('2', 'c++');  
  
console.log( "Value1= " +map.get(1)   );   
console.log("Value2= " + map.get('1') );   
console.log( "Key is Present= " +map.has(3) );   
console.log( "Size= " +map.size );   
console.log( "Delete value= " +map.delete(1) );   
console.log( "New Size= " +map.size );  
```

### 遍历map数据

我们可以通过使用for…of循环遍历map，下面的示例有助于更清楚地理解它。

例子

```typescript
let ageMapping = new Map();  
   
ageMapping.set("Oreja", 40);  
ageMapping.set("Kinm", 25);  
ageMapping.set("Ompa", 30);  
   
//对map键进行迭代
for (let key of ageMapping.keys()) {  
    console.log("Map Keys= " +key);          
}  
//对map值进行迭代
for (let value of ageMapping.values()) {  
    console.log("Map Values= " +value);      
}  
console.log("The Map Enteries are: ");   
//对map条目进行迭代 
for (let entry of ageMapping.entries()) {  
    console.log(entry[0], entry[1]);   
}  
```









