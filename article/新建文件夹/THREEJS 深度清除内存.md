首先清除内存根据官方文档使用dispose。
但是其中有很多坑导致dispose之后scene或者内存中依然存在

如何完全清除一个模型，可能有些细节存在问题，欢迎指正

### 1、清除模型。

``` javascript
function dispose(parent, child) {
    if (child.children.length) {
        let arr = child.children.filter(x => x);
        arr.forEach(a => {
            dispose(child, a)
        })
    }
    if (child instanceof THREE.Mesh || child instanceof THREE.Line) {
        if (child.material.map) child.material.map.dispose();
        child.material.dispose();
        child.geometry.dispose();
    } else if (child.material) {
        child.material.dispose();
    }
    child.remove();
    parent.remove(child);
}
```

### 2、清除存储模型的变量。

如果你在使用时把模型存在了全局变量里，在页面销毁时不仅需要在场景中找到这个模型dispose还需要把存储的变量dispose，不然你可能会发现场景里没有了，但是变量还能正常获取

``` javascript
 geometry.dispose();
 geometry = null;
```

### 踩坑

1、不要用forEach遍历父级的children去remove和dispose
你可以尝试一下，forEach遍历时去dispose，都打印一下，你可能会发现明明有5个孩子，却只遍历了3个。这种类似于forEach时候去splice，在你remove中父级的孩子就会跟着remove导致children变化

2、不要在traverse中remove，原因同1
3、有些时候发现内存没清掉，其实是scene/composer等有自带，具体打印可以看到
解决方法

``` javascript
let arr = scene.children.filter(x => x)
arr.forEach(a => {
    dispose(scene, a);
})
```

### 查看是否清除内存

``` javascript
console.log(renderer.info) //查看memery字段即可
```

### 收尾，清除scene

``` javascript
scene.remove();
renderer.dispose();
renderer.forceContextLoss();
renderer.content = null;
renderer.domElement = null;
cancelAnimationFrame(loop)
```

### ·dispose()方法

关于·dispose()方法可以查看几何体BufferGeometry或Geometry、材质Material、纹理Texture等对象了解。

* Geometry·dispose()

从内存中删除对象. 删除几何体时不要忘记调用此方法，因为它可能导致内存泄漏

``` javascript
// 释放网格模型绑定几何体顶点数据在顶点缓冲区占用的内存
Mesh.geometry· dispose()
```

* Texture·dispose()

具有’dispose’事件类型的dispatchEvent, 调用EventDispatcher

* Material·dispose()

处理材质对象. 材质的纹理不能得到处理. 材质的纹理需要通过纹理对象Texture的dispose方法实现

* WebGLRenderer·dispose()

处理当前渲染上下文

Three.js几何体Geometry和材质Material对象都具有·dispose()方法，如果你不想深入理解在删除网格模型等对象的时候·dispose()方法是如何删除模型内存占用的，会查看文档使用该方法就可以。如果你想深入理解·dispose()方法，最好对原生WebGL有一定的了解，比如通过WebGL APIgl.createBuffer创建一个缓冲区可以用来存储顶点数据，GPU执行着色器代码渲染的时候可以从缓冲区中读取顶点数据，通过gl.deleteBuffer(buffer )可以删除一个顶点缓冲区，释放顶点数据占用的内存空间。有了一定的WebGL基础，然后阅读src目录下源码自然就可以理解。

一个网格模型Mesh是包含几何体geometry和材质对象Material的，几何体geometry本质上就是顶点数据，Three.js通过WebGL渲染器解析几何体的时候会调用WebGL API创建顶点缓冲区来存储顶点数据。

如果仅仅执行scene.remove(Mesh)只是把网格模型从场景对象的.children属性中删除，解析网格模型Mesh几何体的顶点数据通过WebGL API创建的顶点缓冲区占用的内存并不会释放。

BufferGeometry.js、Geometry.js和Material.js源码关于·dispose()方法的封装

``` javascript
dispose: function() {
    // 几何体或材质对象自动触发一个事件dispose
    this.dispatchEvent({
        type: 'dispose'
    });
}
```

WebGLGeometries.js源码中监听几何体对象的dispose事件

``` javascript
function get(object, geometry) {
    ...
    geometry.addEventListener('dispose', onGeometryDispose);
    ...
}

function onGeometryDispose(event) {
    ...
    // 调用WebGLAttributes.js封装的remove方法，删除顶点缓冲区
    attributes.remove(buffergeometry.index);
    ...
}
```

WebGLAttributes.js封装了remove方法可以删除Threejs解析模型几何体顶点数据的时候创建的顶点缓冲区。

``` javascript
function remove(attribute) {
    ...
    if (data) {
        // 调用gl.deleteBuffer()删除顶点缓冲区
        gl.deleteBuffer(data.buffer);
    }
    ...
}

function createBuffer(attribute, bufferType) {
    // 调用gl.createBuffer()创建顶点缓冲区
    var buffer = gl.createBuffer();
}
```
