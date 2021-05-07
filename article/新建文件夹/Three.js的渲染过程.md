主要基于[WebGLRendere类](https://github.com/mrdoob/three.js/blob/master/src/renderers/WebGLRenderer.js)的render方法开展，需要读者有基本的计算机图形知识，比如[计算机图形管线](https://blog.csdn.net/from_the_star/article/details/106646859)之类的。
在Three.js的渲染中，大概可以分为以下几步：

1. 清空当前帧缓冲区，更新MVP矩阵；
2. 将物体分为透明和不透明两类，按照离摄像机从近到远排序（也可在Object3D单独设置renderOrder）；
3. 根据灯光信息，阴影计算，如果有开启平面裁剪就对进行剪裁；
4. 开始逐个渲染物体，按以下顺序，背景、不透明物体、透明物体；
5. 渲染前后还有两个类似于生命周期的回调函数，scene.onBeforeRender和scene.onAfterRender；
6. 最后将深度、模版测试、多边形偏移恢复默认。

其中1、2和5、6都是准备和善后工作，2中为物体分为不透明、透明进行排序，原因有二：
一、为了最大限度地避免overdraw，一个重要的优化策略就是控制绘制顺序。由于深度测试的存在，如果我们可以保证物体都是从前往后绘制的，那么就可以很大程度上减少overdraw。这是因为，在后面绘制的物体由于无法通过深度测试，因此，就不会再进行后面的渲染处理。
二、把透明物体抽出来最后渲染，是为了和不透明物体进行颜色混合。

## 阴影计算

简单说明下，在threejs的阴影渲染的原理：

1. 将相机移动光源位置称为shadowCamera，记录下当前像素点的深度值，记为Z1；
2. 将相机移动原来位置, 绘制是将当前像素点的深度值Z与Z1比较，如果小于Z1则说明是光源照不到的地方，画上阴影

（为什么是小于，因为有一个默认规定，相机的上方向是以相机为原点的坐标轴的Y轴，视线为-Z轴方向）。
这一部分也可以叫做实时阴影计算，与光照贴图添加阴影(·lightMap)区别。具体逻辑在WebGLShadowMap类里，它也是做了第一步，将深度信息存在texture里，具体第二步还是要到物体渲染里进行。
WebGLShadowMap的render方法，设置了WebGL的状态，然后创建阴影贴图，最后还是使用shadowCamera，上交给WebGLRendere类的renderBufferDirect方法处理去了（这是个累死累活的打工方法，物体渲染最后也交给它了）。

## 物体渲染

重点到了，但还是只能简单描述下，里面细节太多了。threejs说到底是简化我们对webgl的使用，了解物体渲染，我需要简单描述下	[webgl怎么绘制](https://app.yinxiang.com/shard/s66/nl/17863266/eedf6836-8617-46c0-99b0-98a43f44794e?title=%E5%88%9D%E6%AD%A5WebGL%E7%A8%8B%E5%BA%8F)，大致分为五步：

1. 获取webgl上下文
2. 初始化着色器代码
    - 创建指定类型的着色器代码，上传source源码，并编译
    - 创建着色器程序，添加着色器，链接webgl
3. 初始化着色器缓冲区
    - 创建缓冲区，绑定到程序，并推入数据（position、normal、uv、color以及MVP矩阵、灯光信息等数据）
    - 将缓冲区写入着色器指定属性，并激活
4. 纹理绑定
5. 绘制（选择模式、设置顶点个数）

这里需要控制的就是 指定不同的着色器代码、推入不同的数据，绑定纹理、选择模式进行绘制，也是物体渲染这块所做的。
renderObjects方法：
遍历渲染列表，如果是	[摄像机阵列](http://www.webgl3d.cn/threejs/docs/#api/zh/cameras/ArrayCamera)，再遍历摄像机，将object, scene, camera, geometry, material, group丢给renderObject处理单个物体。
renderObject方法：
renderObject就一个作用，判断物体是否为立即渲染物体（object.isImmediateRenderObject）：
立即渲染物体和普通物体表现在代码上的区别是，没有用到顶点数组对象（Vertex Array Object，简称 VAO），立即渲染物体直接获取object上的position、normal、uv和color数据丢进buffer，进行绘制。
普通物体会在第一次绘制保存下position、normal、uv和color数据，下一帧绘制，如果没有改变，直接bindVertexArrayOES，就可以了，不用一个个bindBuffer。
详细代码讲解可以看看[这篇文章](http://www.jiazhengblog.com/blog/2017/04/17/3127/)。
renderBufferDirect方法：渲染普通物体。

1. 先调用setProgram拿到添加着色器后的着色器程序，setProgram不仅仅添加shader，也会绑定一些除顶点以外的数据。

setProgram内部，在material变化时调用initMaterial初始化着色器，每种Material有自己对应的shader代码（通过WebGLPrograms获得，这是threejs内置的），在根据environment、fog、envMap、needsLights等不同设置，会再拼接其他的shader代码，最后组成一个完整的shader代码，传入一些需要的uniform数据（包括绑定纹理），把这个program挂在materia，materia存入WebGLProperties对象里。
setProgram内部，在material不变化时，直接取出保存的数据，推入数据即可。
所以说setProgram解决了指定不同的着色器代码、推入不同的数据（除了顶点相关），绑定纹理的问题。

2. WebGLBindingStates对象的setup是用来推入顶点相关的数据，使用了 VAO，上面提及了。
3. 最后renderBufferDirect会根据object的类型，设置绘制模式和顶点个数，选择绘制方法，进行绘制。

指定不同的着色器代码、推入不同的数据，绑定纹理、选择模式进行绘制四个工作就都完成了，这也就是renderBufferDirect的大概的功能。渲染器会保存物体的状态和绘制所需数据，只在物体发生改变时，才重新计算获取，不变时就直接推入着色器程序进行绘制。
其实WebGL绘制时还需要设置各种状态，比如深度测试、模版测试等，具体控制在WebGLState类里，对应的的state.setMaterial。

``` typescript
	this.renderBufferDirect = function ( camera, scene, geometry, material, object, group ) {
		if ( scene === null ) scene = _emptyScene; // renderBufferDirect second parameter used to be fog (could be null)
		const frontFaceCW = ( object.isMesh && object.matrixWorld.determinant() < 0 );
		// 获取着色器程序
		const program = setProgram( camera, scene, material, object );
		// 设置绘制所需状态
		state.setMaterial( material, frontFaceCW );
		// 获取索引和顶点数据
		let index = geometry.index;
		const position = geometry.attributes.position;
		// 判断有无索引和顶点数据
		if ( index === null ) {
			if ( position === undefined || position.count === 0 ) return;
		} else if ( index.count === 0 ) {
			return;
		}
		// 线框模式
		let rangeFactor = 1;
		if ( material.wireframe === true ) {
			index = geometries.getWireframeAttribute( geometry );
			rangeFactor = 2;
		}
		//设置变形动画
		if ( material.morphTargets || material.morphNormals ) {
			morphtargets.update( object, geometry, material, program );
		}
		//绑定顶点数据
		bindingStates.setup( object, material, program, geometry, index );
		let attribute;
		let renderer = bufferRenderer;
		if ( index !== null ) {
			attribute = attributes.get( index );
			renderer = indexedBufferRenderer;
			renderer.setIndex( attribute );
		}
		//计算需要绘制顶点个数
		const dataCount = ( index !== null ) ? index.count : position.count;

		const rangeStart = geometry.drawRange.start * rangeFactor;
		const rangeCount = geometry.drawRange.count * rangeFactor;

		const groupStart = group !== null ? group.start * rangeFactor : 0;
		const groupCount = group !== null ? group.count * rangeFactor : Infinity;

		const drawStart = Math.max( rangeStart, groupStart );
		const drawEnd = Math.min( dataCount, rangeStart + rangeCount, groupStart + groupCount ) - 1;

		const drawCount = Math.max( 0, drawEnd - drawStart + 1 );

		if ( drawCount === 0 ) return;
		//设置绘制模式和绘制方法
		if ( object.isMesh ) {
			if ( material.wireframe === true ) {
				state.setLineWidth( material.wireframeLinewidth * getTargetPixelRatio() );
				renderer.setMode( _gl.LINES );
			} else {
				renderer.setMode( _gl.TRIANGLES );
			}
		} else if ( object.isLine ) {
			let lineWidth = material.linewidth;
			if ( lineWidth === undefined ) lineWidth = 1; // Not using Line*Material
			state.setLineWidth( lineWidth * getTargetPixelRatio() );
			if ( object.isLineSegments ) {
				renderer.setMode( _gl.LINES );
			} else if ( object.isLineLoop ) {
				renderer.setMode( _gl.LINE_LOOP );
			} else {
				renderer.setMode( _gl.LINE_STRIP );
			}
		} else if ( object.isPoints ) {
			renderer.setMode( _gl.POINTS );
		} else if ( object.isSprite ) {
			renderer.setMode( _gl.TRIANGLES );
		}
		if ( object.isInstancedMesh ) {
			renderer.renderInstances( drawStart, drawCount, object.count );
		} else if ( geometry.isInstancedBufferGeometry ) {
			const instanceCount = Math.min( geometry.instanceCount, geometry._maxInstanceCount );
			renderer.renderInstances( drawStart, drawCount, instanceCount );
		} else {
			renderer.render( drawStart, drawCount );
		}
	};
```
