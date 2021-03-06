## Three.js

## 简介

Three.js是众多WebGL三维引擎框架其中之一，源自github的一个开源项目, 项目地址：https://github.com/mrdoob/three.js 。
可以利用three.js进行网页上的三维场景（机械、建筑、游戏等）创建，能写出在浏览器上流畅运行的3D程序。如果没有前端基础，最好预先学习一点HTML/JavaScript方面的知识。
官方文档：https://threejs.org/docs/index.html#manual/en/introduction/Creating-a-scene

## 一、基本概念篇：第一个three.js三维场景

在Three.js中，要渲染物体到网页中，需要3个基本对象：

* 场景
* 相机
* 渲染器

场景对应于整个布景空间，相机是拍摄镜头，渲染器用来把拍摄好的场景转换成胶卷。

``` javascript
var scene = new THREE.Scene();
var camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
var renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);
```

1. 场景

 `var scene = new THREE. Scene(); `

场景对应于整个布景空间，在Threejs中场景就只有一种，用THREE. Scene来表示。

2. 相机

在Threejs中最常用的相机有两种

正投影相机THREE. OrthographicCamera
透视投影相机THREE. PerspectiveCamera
在这里插入图片描述
(1) 正投影相机THREE. OrthographicCamera

``` javascript
var camera = new THREE.OrthographicCamera(width / -2, width / 2, height / 2, height / -2, 1, 1000);
scene.add(camera);
```

函数构造：OrthographicCamera( left : Number, right : Number, top : Number, bottom : Number, near : Number, far : Number )
参考：https://threejs.org/docs/index.html#api/zh/cameras/OrthographicCamera

正交投影相机示意图如下：
![](http://techbrood.com/threejs/docs/images/ortho.jpg)

(2) 透视投影相机THREE. PerspectiveCamera

``` javascript
var camera = new THREE.PerspectiveCamera(45, width / height, 1, 1000);
scene.add(camera);
```

函数构造：PerspectiveCamera( fov : Number, aspect : Number, near : Number, far : Number )
参考：https://threejs.org/docs/index.html#api/zh/cameras/PerspectiveCamera

透视投影相机示意图如下：
![](http://techbrood.com/ueditor/php/upload/image/20160525/1464141326848754.png)

3. 渲染器

``` javascript
var renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);
```

除了创建renderer实例，还需要设置渲染空间的尺寸，一般使用目标屏幕的宽高（window.innerWidth和window.innerHeight），也可以给定一个尺寸。
渲染器renderer的domElement元素，表示渲染器中的画布，所有的渲染都是画在domElement上的，所以这里的appendChild表示将这个domElement挂接在body下面，这样渲染的结果就能够在页面中显示了。

4. 添加对象

现在的场景中是空的，我们向场景中加入最简单的立方体。

``` javascript
var geometry = new THREE.BoxGeometry(1, 1, 1);
var material = new THREE.MeshBasicMaterial({
    color: 0x4ca7c1
});
var cube = new THREE.Mesh(geometry, material);
scene.add(cube);
camera.position.z = 5;
```

创建几何体，创建材质，利用几何体和材质创建对象，将对象加入场景scene中。
默认情况下，当我们调用scene.add()的时候，物体将会被添加到坐标为(0, 0, 0)的位置。但这可能会使得摄像机的位置和立方体相互重叠（摄像机位于立方体中）。为了防止这种情况的发生，需要将摄像机稍微向外移动一些。

5.  渲染场景

``` javascript
function animate() {
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
}
animate();
```

在这里我们创建了一个循环——这使得渲染器能够在每次屏幕刷新时对场景进行绘制（在大多数屏幕上，刷新率一般是60次/秒）。
requestAnimationFrame函数就是让浏览器去执行一次参数中的函数，这样通过上面render中调用requestAnimationFrame()函数，requestAnimationFrame()函数又让rander()再执行一次，就形成了我们通常所说的游戏循环了。

6. 使立方体动起来

在animate()函数中添加

``` javascript
cube.rotation.x += 0.01;
cube.rotation.y += 0.01;
```

这一段代码将在每一帧时被渲染时调用（正常情况下是60次/秒），这就让立方体有了一个看起来很不错的旋转动画。
除了改变立方体的旋转角度、位置，也可以通过改变相机位置角度达到同样动起来的效果。

完整代码（附详细备注）

``` css
body {
    margin: 0;
}

canvas {
    width: 100 %;
    height: 100 %
}
```

``` javascript
// 场景（scene）
var scene = new THREE.Scene();
// 相机（camera）
var camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
// 渲染器（renderer）
var renderer = new THREE.WebGLRenderer(); // 创建renderer实例
renderer.setSize(window.innerWidth, window.innerHeight); // 设置渲染空间的尺寸
document.body.appendChild(renderer.domElement); // 渲染出的画布加入页面
// 添加对象
var geometry = new THREE.BoxGeometry(1, 1, 1); // 创建几何体
var material = new THREE.MeshBasicMaterial({
    color: 0x4ca7c1
}); //创建材质
var cube = new THREE.Mesh(geometry, material); // 用几何体和材质创建对象
scene.add(cube); // 对象加入场景scene中
camera.position.z = 5;
// 渲染场景
var animate = function() {
    requestAnimationFrame(animate);
    cube.rotation.x += 0.01;
    cube.rotation.y += 0.01;
    renderer.render(scene, camera);
};
animate();
```

``` xml
< html>
    < head>
        < title> My first three.js app < /title>
                < /head>
                    < body>
                        < script src="build/three.js">
                            < /script>
                                < /body>
                                    < /html>
```

在浏览器中的效果：
![](https://zhoujie1994.cn/my/three/img/001-firstthree.gif)
在这里插入图片描述

## 二、实际应用篇：跳一跳（You_Jump_I_Jump）

有了基本概念后，我们来一步步实现跳一跳的代码复原，GitHub地址：https://github.com/zj19941113/You_Jump_I_Jump
最终效果图：
![](https://zhoujie1994.cn/my/three/img/001-jump.gif)

1. 创建场景与第一个盒子

（1）基本元素
场景：设置背景颜色

``` javascript
scene = new THREE.Scene();
scene.background = new THREE.Color(0x8797a4);
```

渲染器：渲染空间尺寸设置为屏幕的宽高

``` javascript
renderer = new THREE.WebGLRenderer({
    antialias: true
});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);
```

相机：选择正交投影相机，设置相机位置与朝向

``` javascript
camera = new THREE.OrthographicCamera(window.innerWidth / -2, window.innerWidth / 2, window.innerHeight / 2, window.innerHeight / -2, -1000, 2000);
camera.position.set(200, 180, 200);
camera.lookAt(new THREE.Vector3(0, 0, 0));
```

光源：添加平行光与环境光，平行光让几何体更层次分明，环境光提升整体亮度

``` javascript
light = new THREE.AmbientLight(0xFFFFFF, 0.4);
scene.add(light);
light2 = new THREE.DirectionalLight(0xFFFFFF, 1);
light2.position.set(3, 4, 2);
scene.add(light2);
```

环境光 函数构造：AmbientLight( color : Integer, intensity : Float )
color - (参数可选）颜色的rgb数值。缺省值为 0xffffff(白色)。
intensity - (参数可选)光照的强度。缺省值为 1。
参考：https://threejs.org/docs/index.html#api/zh/lights/AmbientLight

平行光 函数构造：DirectionalLight( color : Integer, intensity : Float )
参考：https://threejs.org/docs/index.html#api/zh/lights/DirectionalLight

（2）创建地面与盒子
ground = creatGround(0, 0, 0x8797a4)
cube01 = creatcube01(0, 0); 
函数 creatGround()，创建5000*5000大小，颜色为#8797a4的地面

``` javascript
function creatGround(x, z, color) {
    var geometry = new THREE.PlaneGeometry(5000, 5000, 1, 1);
    var material = new THREE.MeshLambertMaterial({
        color: color
    });
    mesh = new THREE.Mesh(geometry, material);
    mesh.rotation.x = -Math.PI / 2;
    mesh.position.set(x, -0.01, z);
    scene.add(mesh);
    return mesh;
}
```

函数 creatcube01（），创建一个其中一面有纹理的盒子

``` javascript
function creatcube01(x, z) {
    // 绘制纹理
    var canvas = document.createElement('canvas');
    canvas.width = 100;
    canvas.height = 50;
    var ctx = canvas.getContext('2d');
    ctx.rect(0, 0, 100, 50);
    ctx.fillStyle = "#e86014";
    ctx.fill();
    ctx.beginPath();
    ctx.arc(40, 25, 18, 0, 2 * Math.PI);
    ctx.fillStyle = "#ffffff";
    ctx.fill();
    ctx.beginPath();
    ctx.arc(45, 20, 6, 0, 2 * Math.PI);
    ctx.fillStyle = "#e86014";
    ctx.fill();
    ctx.beginPath();
    ctx.arc(65, 10, 6, 0, 2 * Math.PI);
    ctx.fillStyle = "#ffffff";
    ctx.fill();
    var texture = new THREE.Texture(canvas);
    group = canvasOneFace(x, z, texture, 0xe86014);
    return group;
}
// 创建一个纯色盒子，将纹理贴在平面上覆盖在盒子一面
function canvasOneFace(x, z, texture, color) {
    // 创建纯色盒子，高50，长宽100
    var geometry = new THREE.CubeGeometry(100, 50, 100);
    var material = new THREE.MeshLambertMaterial({
        color: color
    });
    mesh = new THREE.Mesh(geometry, material);
    mesh.position.set(x, 25, z); // 默认中心在(0,0,0)向上抬25，使盒子在地面上
    // 创建100*50的平面，和盒子侧面一样大，材料使用刚绘制的画布作纹理，而不是颜色
    var geometry = new THREE.PlaneGeometry(100, 50);
    var material = new THREE.MeshLambertMaterial({
        map: texture
    });
    texture.needsUpdate = true;
    mesh1 = new THREE.Mesh(geometry, material);
    // 使平面和盒子侧面基本重合，差了0.01的距离，纹理能遮盖住盒子本身颜色
    mesh1.rotation.y = Math.PI / 2;
    mesh1.position.set(x + 50.01, 25, z);
    // 创建阴影
    Shadow = makeShadow();
    Shadow.position.set(x - 30, 0, z + 8);
    // 组合起来方便使用
    group = new THREE.Object3D();
    group.add(mesh, mesh1, Shadow);
    scene.add(group);
    return group;
}
// 创建阴影
function makeShadow(x, z) {
    // 创建平面，贴上图片作为纹理
    vargeometry = new THREE.PlaneGeometry(116, 160, 1, 1);
    // 加载图片作为纹理
    var texture = new THREE.TextureLoader().load("source/shadow.png");
    var material = new THREE.MeshBasicMaterial({
        map: texture
    });
    material.transparent = true; // 材质透明
    meshShadow = new THREE.Mesh(geometry, material);
    // 调整阴影位置
    meshShadow.rotation.x = -Math.PI / 2;
    meshShadow.rotation.z = Math.PI / 2;
    meshShadow.position.set(x - 30, 0, z + 6);
    return meshShadow;
}
```

效果：
![](https://zhoujie1994.cn/my/three/img/001-result.JPG)
因为之后要创建各种盒子，所以这里用了函数，方便之后调用。用画布绘制右侧面的纹理，创建橙色的纯色盒子，将纹理贴在平面上覆盖在盒子侧面，最后再创建阴影，同样是创建平面贴上阴影图片作为纹理。

画布作为纹理：

``` javascript
 var canvas = document.createElement('canvas');
 canvas.width = 100;
 canvas.height = 50;
 var ctx = canvas.getContext('2d');
 // 画布的绘制
 // ……
 // 画布的绘制
 var texture = new THREE.Texture(canvas);
 var material = new THREE.MeshLambertMaterial({
     map: texture
 });
 texture.needsUpdate = true;
 png图片作为纹理：

 var texture = new THREE.TextureLoader().load("source/shadow.png");
 var material = new THREE.MeshBasicMaterial({
     map: texture
 });
 material.transparent = true; // 材质透明
```

注：尽量重用geometry，material，使性能优化。

（3）窗口自适应
window.addEventListener( 'resize', onWindowResize, false ); 
当窗口大小改变时，触发函数 onWindowResize()，实时改变相机参数。

``` javascript
function onWindowResize() {
    width = document.body.clientWidth;
    height = document.body.clientHeight;
    camera.left = width / -2;
    camera.right = width / 2;
    camera.top = height / 2;
    camera.bottom = height / -2;
    // 更新相机投影矩阵，在相机任何参数被改变以后必须被调用
    camera.updateProjectionMatrix();
    renderer.setSize(width, height);
}
```

效果：
![](https://zhoujie1994.cn/my/three/img/001-resize.gif)
完整代码：

``` html
<!DOCTYPE html>
<html lang="en">

<head>
    <title></title>
    <meta charset="utf-8">
</head>

<body>
    <script src="build/three.js"></script>
    <script>
        var camera, scene, renderer;
        init();
        animate();

        function init() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x8797a4);
            renderer = new THREE.WebGLRenderer({
                antialias: true
            });
            renderer.setSize(window.innerWidth, window.innerHeight);
            document.body.appendChild(renderer.domElement);
            camera = new THREE.OrthographicCamera(window.innerWidth / -2, window.innerWidth / 2, window.innerHeight / 2, window.innerHeight / -2, -1000, 2000);
            camera.position.set(200, 180, 200);
            camera.lookAt(new THREE.Vector3(0, 0, 0));
            light = new THREE.AmbientLight(0xFFFFFF, 0.4);
            scene.add(light);
            light2 = new THREE.DirectionalLight(0xFFFFFF, 1);
            light2.position.set(3, 4, 2);
            scene.add(light2);
            ground = creatGround(0, 0, 0x8797a4)
            cube01 = creatcube01(0, 0);
            window.addEventListener('resize', onWindowResize, false);
        }

        function creatGround(x, z, color) {
            var geometry = new THREE.PlaneGeometry(5000, 5000, 1, 1);
            var material = new THREE.MeshLambertMaterial({
                color: color
            });
            mesh = new THREE.Mesh(geometry, material);
            mesh.rotation.x = -Math.PI / 2;
            mesh.position.set(x, -0.01, z);
            scene.add(mesh);
            return mesh;
        }

        function creatcube01(x, z) {
            // 绘制纹理
            var canvas = document.createElement('canvas');
            canvas.width = 100;
            canvas.height = 50;
            var ctx = canvas.getContext('2d');
            ctx.rect(0, 0, 100, 50);
            ctx.fillStyle = "#e86014";
            ctx.fill();
            ctx.beginPath();
            ctx.arc(40, 25, 18, 0, 2 * Math.PI);
            ctx.fillStyle = "#ffffff";
            ctx.fill();
            ctx.beginPath();
            ctx.arc(45, 20, 6, 0, 2 * Math.PI);
            ctx.fillStyle = "#e86014";
            ctx.fill();
            ctx.beginPath();
            ctx.arc(65, 10, 6, 0, 2 * Math.PI);
            ctx.fillStyle = "#ffffff";
            ctx.fill();
            var texture = new THREE.Texture(canvas);
            group = canvasOneFace(x, z, texture, 0xe86014);
            return group;
        }
        // 创建一个纯色盒子，将纹理贴在平面上覆盖在盒子一面
        function canvasOneFace(x, z, texture, color) {
            // 创建纯色盒子，高50，长宽100
            var geometry = new THREE.CubeGeometry(100, 50, 100);
            var material = new THREE.MeshLambertMaterial({
                color: color
            });
            mesh = new THREE.Mesh(geometry, material);
            mesh.position.set(x, 25, z); // 默认中心在(0,0,0)向上抬25，使盒子在地面上
            // 创建100*50的平面，和盒子侧面一样大，材料使用刚绘制的画布作纹理，而不是颜色
            var geometry = new THREE.PlaneGeometry(100, 50);
            var material = new THREE.MeshLambertMaterial({
                map: texture
            });
            texture.needsUpdate = true;
            mesh1 = new THREE.Mesh(geometry, material);
            // 使平面和盒子侧面基本重合，差了0.01的距离，纹理能遮盖住盒子本身颜色
            mesh1.rotation.y = Math.PI / 2;
            mesh1.position.set(x + 50.01, 25, z);
            // 创建阴影
            Shadow = makeShadow();
            Shadow.position.set(x - 30, 0, z + 8);
            // 组合起来方便使用
            group = new THREE.Object3D();
            group.add(mesh, mesh1, Shadow);
            scene.add(group);
            return group;
        }
        // 创建阴影
        function makeShadow(x, z) {
            // 创建平面，贴上图片作为纹理
            var geometry = new THREE.PlaneGeometry(116, 160, 1, 1);
            // 加载图片作为纹理
            var texture = new THREE.TextureLoader().load("source/shadow.png");
            var material = new THREE.MeshBasicMaterial({
                map: texture
            });
            material.transparent = true; // 材质透明
            meshShadow = new THREE.Mesh(geometry, material);
            // 调整阴影位置
            meshShadow.rotation.x = -Math.PI / 2;
            meshShadow.rotation.z = Math.PI / 2;
            meshShadow.position.set(x - 30, 0, z + 6);
            return meshShadow;
        }

        function onWindowResize() {
            width = document.body.clientWidth;
            height = document.body.clientHeight;
            camera.left = width / -2;
            camera.right = width / 2;
            camera.top = height / 2;
            camera.bottom = height / -2;
            // 更新相机投影矩阵，在相机任何参数被改变以后必须被调用
            camera.updateProjectionMatrix();
            renderer.setSize(width, height);
        }

        function animate() {
            requestAnimationFrame(animate);
            renderer.render(scene, camera);
        }
    </script>
</body>

</html>
```

2. 更多盒子的生成

所有盒子的生成函数，见 https://github.com/zj19941113/You_Jump_I_Jump/blob/master/zjFirstStep.html
boxs大合照：
![](https://zhoujie1994.cn/my/three/img/001-all_boxes.JPG)

3. 性能与调试

创建stats对象进行性能监控
帧数：图形处理器每秒钟能够刷新几次，通常用fps（Frames Per Second）来表示。帧数越高，画面的感觉就会越好。所以大多数游戏都会有超过30的FPS。我们设置性能监视器来监视FPS。
（1）引入[Stats.js](https://github.com/mrdoob/stats.js)文件。

 `<script src="js/Stats.js"></script>`

（2）将stats对象加入到html网页中

``` javascript
var stats;
stats = new Stats();
stats.domElement.style.position = 'absolute';
stats.domElement.style.right = '8px';
stats.domElement.style.top = '8px';
document.body.appendChild(stats.domElement);
```

（3）在 animate() 中调用stats.update()统计时间和帧数

``` javascript
function animate() {
    requestAnimationFrame(animate);
    stats.update(); // 调用stats.update()统计时间和帧数
    renderer.render(scene, camera);
}
```

其中FPS表示：上一秒的帧数，这个值越大越好，一般都为60左右。点击后变成另一个视图。MS表示渲染一帧需要的毫秒数，这个数字是越小越好，再次点击又可以回到FPS视图中。
效果：
在这里插入图片描述

## OrbitControls控制器辅助调试

（1）引入[controls/OrbitControls.js](https://github.com/mrdoob/three.js/blob/017f2a9eb17820772359b1e1dbca2b626c9f32b1/examples/jsm/controls/OrbitControls.js)文件。

 `<script src="js/controls/OrbitControls.js"></script>`

（2）controls控制器设置

``` javascript
var controls;
controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.25;
controls.screenSpacePanning = false;
controls.minDistance = 100;
controls.maxDistance = 500;
controls.maxPolarAngle = Math.PI / 2;（
3） 在 animate() 中调用controls.update()

function animate() {
    requestAnimationFrame(animate);
    stats.update(); // 调用stats.update()统计时间和帧数
    controls.update(); // 调用controls.update()
    renderer.render(scene, camera);
}
```

OrbitControls控制器的作用是，鼠标左键进行视角的旋转，右键控制平移，滚轮控制缩放。
效果：
![](https://zhoujie1994.cn/my/three/img/001-controls.gif)
完整代码：

``` html
<!DOCTYPE html>
<html lang="en">

<head>
    <title></title>
    <meta charset="utf-8">
</head>

<body>
    <script src="build/three.js"></script>
    <script src="js/Stats.js"></script>
    <script src="js/controls/OrbitControls.js"></script>
    <script>
        var camera, scene, renderer;
        var stats, controls;
        init();
        animate();

        function init() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x8797a4);
            renderer = new THREE.WebGLRenderer({
                antialias: true
            });
            renderer.setSize(window.innerWidth, window.innerHeight);
            document.body.appendChild(renderer.domElement);
            stats = new Stats();
            stats.domElement.style.position = 'absolute';
            stats.domElement.style.right = '8px';
            stats.domElement.style.top = '8px';
            document.body.appendChild(stats.domElement);
            camera = new THREE.OrthographicCamera(window.innerWidth / -2, window.innerWidth / 2, window.innerHeight / 2, window.innerHeight / -2, -1000, 2000);
            camera.position.set(200, 180, 200);
            camera.lookAt(new THREE.Vector3(0, 0, 0));
            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.enableDamping = true;
            controls.dampingFactor = 0.25;
            controls.screenSpacePanning = false;
            controls.minDistance = 100;
            controls.maxDistance = 500;
            controls.maxPolarAngle = Math.PI / 2;
            light = new THREE.AmbientLight(0xFFFFFF, 0.4);
            scene.add(light);
            light2 = new THREE.DirectionalLight(0xFFFFFF, 1);
            light2.position.set(3, 4, 2);
            scene.add(light2);
            ground = creatGround(0, 0, 0x8797a4)
            cube01 = creatcube01(0, 0);
            window.addEventListener('resize', onWindowResize, false);
        }

        function creatGround(x, z, color) {
            // 地面创建函数
            // ……
            // 地面创建函数
        }

        function creatcube01(x, z) {
            // 盒子创建函数
            // ……
            // 盒子创建函数
        }

        function onWindowResize() {
            // 窗口自适应函数
            // ……
            // 窗口自适应函数
        }

        function animate() {
            requestAnimationFrame(animate);
            stats.update();
            controls.update();
            renderer.render(scene, camera);
        }
    </script>
</body>

</html>
```

有效利用Controls和Helper能使视图展示更加出色，从而方便代码的编写。
这里有官方提供的一些实例（包括模型加载、控制器、动画等相关内容）：
https://github.com/mrdoob/three.js/tree/017f2a9eb17820772359b1e1dbca2b626c9f32b1/examples
