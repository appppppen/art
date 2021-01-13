通过Threejs开发Web3D应用的时候，可能需要删除场景中的模型对象，如果想从一个场景Scene或组对象Group删除一个三维模型对象，可以通过.remove()和·dispose()方法来实现。

删除场景对象中Scene一个子对象Group，并释放组对象Group中所有网格模型几何体的顶点缓冲区占用内存
``` javascript
// 递归遍历组对象group释放所有后代网格模型绑定几何体占用内存
group.traverse(function(obj) {
  if (obj.type === 'Mesh') {
    obj.geometry.dispose();
    obj.material.dispose();
  }
})
// 删除场景对象scene的子对象group
scene.remove(group);
```
- .add()方法
通过.add()方法可以把一个模型或光源对象添加到场景Scene或组对象Group中，Three.js渲染器场景的时候，一个模型只有插入场景中才会被渲染出来显示在Canvas画布上。
``` javascript
// 一个网格模型插入到场景中
scene.add(Mesh)
// 一个光源对象插入场景中
scene.add(light)

// 一个网格模型插入到组对象中，作为group的子对象
const group = new THREE.Group()
group.add(mesh)
// group作为scene的子对象
scene.add(group)
```
- .remove()方法
.add()方法是给父对象添加一个子对象，.remove()方法是删除父对象中的一个子对象。 一个对象的全部子对象可以通过该对象的.children()属性访问获得，执行该对象的删除方法.remove()和添加方法.add()一样改变的都是父对象的.children()属性。

场景Scene或组对象Group的.remove()方法threejs文档介绍可以参考它们的基类Object3D。
``` javascript
// 删除场景对象的子对象网格模型Mesh
scene.remove(Mesh)
// 删除场景对象的子对象光源对象
scene.remove(light)
// 删除父对象group的子对象网格模型mesh
group.add(mesh)
```
.remove()方法和.add()方法一样可以有多个参数对象,同时往父对象中添加多个子对象。
``` javascript
// 同场景插入多个对象
scene.add(light1,Mesh1,Mesh2)
// 一次删除场景中多个对象
scene.remove(light1,Mesh1,Mesh2)
```
- Object3D.js源码
如果像你深入理解删除方法.remove()和添加方法.add()， 可以查看Object3D构造函数源码中是如何封装这两个方法的。
``` javascript
add: function ( object ) {
  if ( arguments.length > 1 ) {
    // 如果add方法的参数有多个，执行下面代码
    for ( var i = 0; i < arguments.length; i ++ ) {
      this.add( arguments[ i ] );
    }
  }
    // 如果对象object已经有父对象，有添加到新的父对象，需要从原来父对象中删除
    if ( object.parent !== null ) {
      object.parent.remove( object );
    }
    // 子对象object的parent属性指向父对象索引地址
    object.parent = this;
    // 把子对象object插入到父对象的children属性数组中
    this.children.push( object );
},
remove: function ( object ) {
...
// 获得子对象object在父对象的children数组属性中的索引位置
  var index = this.children.indexOf( object );
...
// 被删除子对象的父对象属性原来指向父对象，要设置为空null
    object.parent = null;
// 删除children数组中元素：子对象object
    this.children.splice( index, 1 );
...
},
```
- ·dispose()方法
关于·dispose()方法可以查看几何体BufferGeometry或Geometry、材质Material、纹理Texture等对象了解。

Geometry·dispose()

从内存中删除对象.删除几何体时不要忘记调用此方法，因为它可能导致内存泄漏
``` javascript
// 释放网格模型绑定几何体顶点数据在顶点缓冲区占用的内存
Mesh.geometry·dispose()
```
Texture·dispose()

具有'dispose'事件类型的dispatchEvent,调用EventDispatcher

Material·dispose()

处理材质对象. 材质的纹理不能得到处理. 材质的纹理需要通过纹理对象Texture的dispose方法实现

WebGLRenderer·dispose() 处理当前渲染上下文

Three.js几何体Geometry和材质Material对象都具有·dispose()方法，如果你不想深入理解在删除网格模型等对象的时候·dispose()方法是如何删除模型内存占用的，会查看文档使用该方法就可以。如果你想深入理解·dispose()方法，最好对原生WebGL有一定的了解，比如通过WebGL APIgl.createBuffer创建一个缓冲区可以用来存储顶点数据，GPU执行着色器代码渲染的时候可以从缓冲区中读取顶点数据，通过gl.deleteBuffer(buffer )可以删除一个顶点缓冲区，释放顶点数据占用的内存空间。有了一定的WebGL基础，然后阅读src目录下源码自然就可以理解。

一个网格模型Mesh是包含几何体geometry和材质对象Material的，几何体geometry本质上就是顶点数据，Three.js通过WebGL渲染器解析几何体的时候会调用WebGL API创建顶点缓冲区来存储顶点数据。

如果仅仅执行scene.remove(Mesh)只是把网格模型从场景对象的.children属性中删除，解析网格模型Mesh几何体的顶点数据通过WebGL API创建的顶点缓冲区占用的内存并不会释放。

BufferGeometry.js、Geometry.js和Material.js源码关于·dispose()方法的封装
``` javascript
dispose: function () {
  // 几何体或材质对象自动触发一个事件dispose
  this.dispatchEvent( { type: 'dispose' } );
}
```
WebGLGeometries.js源码中监听几何体对象的dispose事件
``` javascript
function get( object, geometry ) {
  ...
  geometry.addEventListener( 'dispose', onGeometryDispose );
  ...
}

function onGeometryDispose( event ) {
  ...
  // 调用WebGLAttributes.js封装的remove方法，删除顶点缓冲区
    attributes.remove( buffergeometry.index );
  ...
}
```
WebGLAttributes.js封装了remove方法可以删除Threejs解析模型几何体顶点数据的时候创建的顶点缓冲区。
``` javascript
function remove( attribute ) {
  ...
  if ( data ) {
    // 调用gl.deleteBuffer()删除顶点缓冲区
    gl.deleteBuffer( data.buffer );
  }
  ...
}

function createBuffer( attribute, bufferType ) {
  // 调用gl.createBuffer()创建顶点缓冲区
  var buffer = gl.createBuffer();
}
```
