转[Unity Shader-反射效果（CubeMap，Reflection Probe，Planar Reflection，Screen Space Reflection）](https://www.daimajiaoliu.com/daima/4796dfbe19003f8)

## 简介

反射效果，是一个渲染中很重要的一个效果。在表现光滑表面（金属，光滑地面），水面（湖面，地面积水）等材质的时候，加上反射，都可以画面效果有很大的提升。来看几张图：
先来张最近比较火爆的国产大作《逆水寒》的镜湖，我木有钱钱进副本，只好从网上找了个截图，感觉很漂亮哈。

《罗马之子》主角的金属盔甲反射&远处水面倒影反射。

《耻辱-外魔之死》地面积水的反射，只反射了静态场景，没有反射动态物体，似乎是一个比较省的做法。

《底特律-变人》神作，画面和剧情都相当给力，就像看电影一样，比如上面图的地面积水的效果，实时反射效果，包括场景的动态物体。

可见，反射效果对于场景整体提升有很大作用，今天本人就来学习一下几种常见的反射的实现。如有错误还望各位高手批评指正。

## 反射的相关基本内容

### 反射的原理及分类
反射，应该属于间接光照的范畴，而非直接光照。我们正常计算光的dot（N，L）或者dot（H，N）时计算的均为直接光照，光源出发经过物体表面该像素点反射进入眼睛的光照。然而这只是一部分，该点还可以接受来自场景中所有其他点反射的光照，如果表面光滑，则表面就可以反射周围的环境（如镜面，金属），到那时这个计算相当复杂，相当于需要在该点法线方向对应的半球空间上做积分运算才可能计算完全的间接光照，而且光线与物体碰撞后并不会消亡，而是经过反射或折射，改变方向后继续传递，相当于无限递归。实时计算现在来看应该还是不太可能的。正好最近在玩这个，尝试了一下使用RayTracing，屏幕空间发射射线碰撞，反弹次数限制在5次，渲染几个球体，分辨率很低的情况下，在PC平台离线渲染用了将近一个小时（虽然是cpu计算，没有并行）

既然真的反射如此之费，但是反射的效果又如此诱人，前辈们就开始想各种办法来模拟反射。于是乎，各种性能友好的反射方法就应运而生了。最常见的就是环境贴图的方案，目前使用最多的应该是cube map，将对象周围环境烘焙到贴图上进行采样，进阶版其实就是功能更强大的Reflection Probe了。另外平面反射（Planar Reflection）可以用一个反转的相机再渲染一次场景模拟，更复杂一些的是屏幕空间反射Screen Space Reflection（SSR，额，不是某游戏抽的那个）。

### Reflect方法推导

我们在各种反射效果的实现中几乎都会用到一个函数：reflect函数，cg语言自带了这个函数，不过还是要来看一下这个函数的实现，之前本人在《图形学相关数学》这篇blog里面推导过，此处就只贴出一张原理图了：

需要注意的就是Reflect的计算过程要求入射方向和法向量都是单位向量，否则结果是不对的。cg的reflect函数自身有没有在计算前做normalize就不得而知了（normalize相对还是一个比较费的操作，无谓增加消耗可能不是很值得，不知道会不会像C++ std的检查一样交给使用者去做喽，如果有知道的大佬可以告诉我哈）。

### 环境反射

先来看一发最简单的反射，环境反射。本文中的环境反射，指的是静态的环境反射，也就是预先烘焙好的环境贴图。主要优点是性能较好，可以用于任意表面。环境反射包含极坐标映射（效率低，变换非线性），球面映射（生成复杂，非线性），立方体映射（即Cube Map，简单，线性，需要存储6个面），八面体映射（线性，一张图）。

最常用的应该就是Cube Map了，Cube Map其实是上个世纪90年代左右就已经提出的技术了，应用也很广泛，比如天空盒，点光源的Shadow Map，模拟反射等等。所谓Cube Map，顾名思义就是一个立方体的贴图，六个面分别对应一张贴图，由相机朝向6个方向分别渲染得到的。

### Cube Map生成（旧版）

先看一下最基本的生成Cubemap的方法，Unity已经为我们提供好了接口（注意，这种方式已经不推荐了，没有HDR效果，但是对Cube Map比较直观的认识）：
```c#
/********************************************************************
 FileName: CubeMapTool.cs
 Description: 生成Cube Map 工具
*********************************************************************/
using UnityEngine; 
using UnityEditor; 

public class CubeMapTool : EditorWindow
{

    private Cubemap cubeMap = null;

    [MenuItem("Tools/Cube Map Generate")]
	public static void GenerateCubeMap()
    {
        GetWindow<CubeMapTool>();
    }

    private void OnGUI()
    {
        cubeMap = EditorGUILayout.ObjectField(cubeMap, typeof(Cubemap), false, GUILayout.Width(400)) as Cubemap;
        if (GUILayout.Button("Render To Cube Map"))
        {
            SceneView.lastActiveSceneView.camera.RenderToCubemap(cubeMap);
        }
    }

}

``` 
Project面板下右键可以创建一个Cube Map资源，然后勾选Readable（否则写不进去），拖入槽中，然后在编辑器下移动相机到合适位置，点击按钮，就可以将当前场景渲染到Cube Map中了：

### Cube Map使用

shader代码如下：
```cpp 
//基本的Cube Map反射效果
Shader "Reflection/CubeMapReflection"
{
	Properties
	{
		_CubeTex ("Cube Tex", Cube) = ""{}
	}
	SubShader
	{
		Tags { "RenderType"="Opaque" }

		Pass
		{
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			
			#include "UnityCG.cginc"

			struct v2f
			{
				float4 vertex : SV_POSITION;
				float3 reflectionDir : TEXCOORD0;
			};
			
			uniform samplerCUBE _CubeTex;
			
			v2f vert (appdata_base v)
			{
				v2f o;
				o.vertex = UnityObjectToClipPos(v.vertex);
				float3 worldNormal = UnityObjectToWorldNormal(v.normal);
				float3 worldViewDir = WorldSpaceViewDir(v.vertex);
				o.reflectionDir = reflect(-worldViewDir, worldNormal);
				return o;
			}
			
			fixed4 frag (v2f i) : SV_Target
			{
				fixed4 col = texCUBE(_CubeTex, i.reflectionDir);
				return col;
			}
			ENDCG
		}
	}
}
```

效果如下：

### Reflection Probe

Reflection Probe实际上就是环境映射，可以说是进阶版的环境映射了（此处与环境映射并列并非表示它是一种新类型的反射方式，Reflection Probe就是环境映射，只不过是环境映射的扩展，包含环境映射的全部属性）。传统意义的环境映射，更多的时候是对应材质上的一个属性（想象一下，在每个场景里的材质球上拖一个cube map是多么的蛋疼），但是实际上，用环境映射表示反射，表示的是当前的环境属性，属于当前场景的信息，而不再是属于某个材质，这才更对得起环境映射这个名字。这个环境属性实际上就相当于一个全局的变量，Unity已经为我们设置好了相关的属性信息，我们在shader中可以直接使用。

### Reflection Probe配置

Reflection Probe分为几种模式，RealTime，Bake, Custom模式。RealTime是实时生成反射贴图，虽然可以设置成每帧更新一个面，不过效率也比较堪忧；Custom模式可以放进去一个自定义的Cube Map；Bake模式需要提前烘焙，Linghting面板烘焙时可以生成场景自身的Reflection Probe，将Reflection Probe Static标签的对象+天空盒进行烘焙。我们可以在场景中放置ReflectionProbe，但是如果不放置，也不至于完全没有效果，当场景有天空盒并且开启了Refrection，Unity默认会给我们一个天空盒的ReflectionProbe的效果，在烘焙的时候就会生成这个信息，与lightmap在同级目录。

上文中提到过相机渲染场景到CubeMap中，比较麻烦，所以新版本的方式就是直接设置Reflection Probe，然后烘焙。Unity直接为我们提供了这个功能，在Reflection Probe面板或者Lighting面板均可以进行烘焙。

关于Reflection Probe的设置方法，可以参考Unity官方文档：Reflection Probe，Lightting中Environment Reflection相关。

### Reflection Probe使用

直接使用Reflection Probe比较简单，Unity已经为我们定义好了Reflection Probe的相关shader uniform变量，而且会根据各种设置把合适的Reflection Probe填充到shader变量中。

先看一下Unity中定义的Reflection Probe相关变量：

``` cpp
// ----------------------------------------------------------------------------
// Reflection Probes

UNITY_DECLARE_TEXCUBE(unity_SpecCube0);
UNITY_DECLARE_TEXCUBE_NOSAMPLER(unity_SpecCube1);

CBUFFER_START(UnityReflectionProbes)
    float4 unity_SpecCube0_BoxMax;
    float4 unity_SpecCube0_BoxMin;
    float4 unity_SpecCube0_ProbePosition;
    half4  unity_SpecCube0_HDR;

    float4 unity_SpecCube1_BoxMax;
    float4 unity_SpecCube1_BoxMin;
    float4 unity_SpecCube1_ProbePosition;
    half4  unity_SpecCube1_HDR;
CBUFFER_END
```

定义，采样，传递CubeMap的函数建议也使用Unity官方提供的函数（HLSLSupport，此处贴出非DX11定义）：

``` cpp
// Cubemaps
#define UNITY_DECLARE_TEXCUBE(tex) samplerCUBE tex
#define UNITY_ARGS_TEXCUBE(tex) samplerCUBE tex
#define UNITY_PASS_TEXCUBE(tex) tex
#define UNITY_PASS_TEXCUBE_SAMPLER(tex,samplertex) tex
#define UNITY_DECLARE_TEXCUBE_NOSAMPLER(tex) samplerCUBE tex
#define UNITY_SAMPLE_TEXCUBE(tex,coord) texCUBE (tex,coord)
```

这样，在使用Reflection Probe就很简单啦，上面的CubeMap采样方法的fragment shader修改一下即可，使用Reflection Probe渲染时支持HDR格式，勾选之后可以将贴图编码为HDR格式，所以在使用之前，需要先进行一步解码操作才能得到正确的结果（与宏和配置有关）。

``` cpp
half4 frag (v2f i) : SV_Target
{
	half4 rgbm = UNITY_SAMPLE_TEXCUBE(unity_SpecCube0, i.reflectionDir);
	half3 color = DecodeHDR(rgbm, unity_SpecCube0_HDR);
	return half4(color, 1.0);
}
```

直接使用天空盒作为Reflection Probe效果如下：

使用环境反射+高光+Bloom+HDR套餐，可以做出金属效果（不由得让我想起《终结者2》里面的液态机器人了）。

### Box Projection Reflection Probe

Cube Map正常来说是不考虑物体与被反射物体之间距离的，类似人物盔甲，圆球等不容易看出穿帮的物体效果较好，而如果反射的物体距离被反射的物体很远时，例如无限远的天空盒，反射效果也比较好。比如上图中的反射天空盒效果。但是当反射对象与被反射对象距离较近时，用Reflection Probe（Cube Map）效果就有些差了。我们把反射材质放在地面上，下图中天空效果反射较好，但是栅栏的反射就有些奇怪了：

如果离近看，效果就不对了，在地面的反射完全看不到栅栏：

不过，我们可以用一种叫做Box Projection Cube Map的技术，通过重新修正反射采样方向，得到相对较好的效果，如下图：

使用Box Projection Cube Map很简单，因为Unity已经为我们写好了函数以及所需的全部数据已经传递给了内置shader变量中，我们可以很容易地通过修改几句代码得到上图的效果：

``` cpp
half4 frag (v2f i) : SV_Target
{
	float3 reflectDir = i.reflectionDir;
	//通过BoxProjectedCubemapDirection函数修正reflectDir
	reflectDir = BoxProjectedCubemapDirection(reflectDir, i.worldPos, unity_SpecCube0_ProbePosition, unity_SpecCube0_BoxMin, unity_SpecCube0_BoxMax);
	half4 rgbm = UNITY_SAMPLE_TEXCUBE(unity_SpecCube0, reflectDir);
	half3 color = DecodeHDR(rgbm, unity_SpecCube0_HDR);
	return half4(color, 1.0);
}
```

为何直接采样效果不对，而用了这个Magic的函数修改了反射方向后，反射效果就大大提升了呢。

来分析一下，反射时，我们采样Reflection Probe时的向量对应的起点是Reflection Probe的中心点R，方向是在当前像素点计算的视线方向对应法线方向的反射向量PK，如下图所示：

ABCD为Reflection Probe对应的Cube，P为当前像素点，E为相机位置，N为当前像素点对应的法线方向，PK为视线方向计算得到的反射方向。如果我们按照PK方向采样Reflection Probe的话，那么就会是RL（平行于PK）方向。如果R与P点重合，采样结果是正确的，但是只有这一个点正确，其他所有点的结果都是不正确的。为了保证采样效果正确，需要求得真正的采样方向RK。

Reflection Probe的位置R，采样位置P已知，RK = RP+PK。RP向量已知，PK方向已知，所以问题就简化成了求PK方向距离。

首先，K是在Cube包围盒上的点，所以第一个问题就是在于我们需要求得包围盒边界的坐标值才好计算采样碰撞点K，Unity在unity_SpecCube0_BoxMin，unity_SpecCube0_BoxMax这两个内置的shader uniform变量中为我们存储了包围盒的BoundingBox最大最小值，不仅仅在划定Reflection Probe影响范围上有用，还有一个重要的作用就在于Box Projection上。一个Cube，八个顶点，分为八个象限，我们只需要关心PK所在方向上的边界值即可。

这里我们需要用到一个很好玩的运算符，? ：运算符。与C中作用一致，但是在shader中，对一个向量使用这个运算符，会分别对向量的每个分量进行? ：运算。比如下面的计算：

``` cpp
float3 test = float3(0,0,0.5);
fixed3 col1 = fixed3(1,1,1);
fixed3 col2 = fixed3(0,0,0);
color = (test > 0.0f) ? col1 : col2;
//result : fixed3(0, 0, 1);
```

有了这个运算符，我们就可以通过（PK > 0.0f） ？ rbmax ： rbmin得到在PK方向上的包围盒坐标值了，我们假设其为Bound。

不过还有一个问题，在三维空间中，一个向量可能与三维的面三个面相交，如下图所示（这里只画了一个二维方向的示意图，横向表示x轴，纵向表示y轴，结论可以扩展到三维空间）：

我们用正常表示射线的方式表达一个向量，已知起始点P，向量方向PK表示为Dir，P + t *Dir = K。t就为该方向上的距离也就是我们最终要求得的PK长度。上图中，我们可能有两个交点坐标，分别为K和H：

K = P + t1 * Dir

H = P + t2 * Dir

Z = P + t3 * Dir（扩展到三维方向）

K和H的具体位置我们不知道，但是我们知道它们在一个方向上的值，即K.x = Bound.x，H.y = Bound.y，Z.z = Bound.z，分别考虑各自方向：

K.x = P.x + t1 * Dir.x

H.y = P.y + t2 * Dir.y

Z.z = P.z + t3 * Dir.z（扩展到三维方向）

再重新整合到一个向量来表示的话，就可以表示成Bound = P + T * Dir，其中T = （t1，t2，t3）。T = （Bound - P) / Dir

不过实际上，我们最终只需要一个碰撞点，从上图很容易看出，我们需要的碰撞点是距离P点最近的点，即沿着P + t * Dir方向行进t最短距离即可，也就是说，我们最终需要的t值只需要从T向量的（t1，t2，t3）中取得最小的值，就是最终PK的距离了。

代码如下：

``` cpp
 half3 nrdir = normalize(worldRefl); //PK方向
 half3 bounding = (nrdir > 0.0f) ? rbmax : rbmin //求得PK方向上包围盒的坐标
 half3 T = (bounding - worldPos) / ndir； //求得三方向上的T值
 half t = min(min(T.x, T.y), T.z); //求得最小t
 
 half3 rp = worldPos - cubeMapCenterPos; //向量RP
 half3 pk = worldPos + t * nrdir; //向量pk
 half3 rk = rp + pk; //向量rk为最终采样方向

// 再来看一下官方源码的BoxProjectedCubemapDirection函数中的计算：
inline half3 BoxProjectedCubemapDirection (half3 worldRefl, float3 worldPos, float4 cubemapCenter, float4 boxMin, float4 boxMax)
{
    // Do we have a valid reflection probe?
    UNITY_BRANCH
    if (cubemapCenter.w > 0.0)
    {
        half3 nrdir = normalize(worldRefl);

        #if 1
            half3 rbmax = (boxMax.xyz - worldPos) / nrdir;
            half3 rbmin = (boxMin.xyz - worldPos) / nrdir;

            half3 rbminmax = (nrdir > 0.0f) ? rbmax : rbmin;

        #else // Optimized version
            half3 rbmax = (boxMax.xyz - worldPos);
            half3 rbmin = (boxMin.xyz - worldPos);

            half3 select = step (half3(0,0,0), nrdir);
            half3 rbminmax = lerp (rbmax, rbmin, select);
            rbminmax /= nrdir;
        #endif

        half fa = min(min(rbminmax.x, rbminmax.y), rbminmax.z);

        worldPos -= cubemapCenter.xyz;
        worldRefl = worldPos + nrdir * fa;
    }
    return worldRefl;
}
```

从Box Projection Cube Map的实现来看，的确可以通过修正采样的方向，找到正确的采样方向，达到较好的反射效果。不过买家秀和卖家秀总是有区别的，Box Projection方法在边界有畸变（上图的山有些扭曲），而且，最重要的问题在于，反射的范围要和Reflection Probe的范围一致，否则结果是不对的。所以个人认为，室内场景等边界与Reflection Probe边界一致的这种场景，使用这个技术比较合适，可以用一个比较cheap的方案达到近似plannar reflection的效果。

Box Projection Cube Map似乎也是个挺新的技术，关于BoxProjectionCubeMap的原理可以参考GameDev上发明这个的帖子。

### Planar Reflection

Planar Reflection，平面反射。顾名思义，就是在平面上运用的反射，名字也正是这种反射方式的限制，只能用于平面，而且是高度一致的平面，多个高度不一的平面效果也不正确（除非针对每个平面单独计算，消耗嘛，你懂得）。一般情况下，一个平面整体反射效果基本可以满足需求，而且对于实时渲染反射效果，需要将要反射的物体多渲染一次，控制好层级数量的话，性能至少是在可以接受的范围，并且相对于其他几种反射效果来说，平面反射的效果是最好的，所以Planar Reflection目前是实时反射效果中使用得比较多的一个方案。

### 平面方程以及平面反射相关推导

既然说到平面反射，自然，我们得先找到一个方法表示一个平面。常见的两种平面表示方法：

一般式：可以表示为Ax + By + Cz + D = 0（其中A, B, C, D为已知常数，并且A, B, C不同时为零）。

点法式：平面可以由平面上任意一点和垂直于平面的任意一个向量确定，这个垂直于平面的向量称之为平面的法向量。假设平面上一个确定点P0（x0，y0，z0），法向量为N（a，b，c），那么平面上任意点P（x，y，z），满足PP0 · N = 0，即（x - x0， y - y0，z - z0） · （a，b，c） = 0。

两种方式各有优点，一般式变量数量少，但是没有什么有用信息，点法式包含面法线信息，但是需要变量数较多。那么把二者结合一下，或者说变形一下即可。比如点法式点乘展开后得到ax + by + cz + (-ax0 - by0 - cz0) = 0，即为一般式。其中-ax0 - by0 - cz0为常数，表示为d。那么点法式方程就可以表示为N·P + d = 0的形式，其中P为平面上任意一点，N为法向量。那么，d表示神魔恋？要是硬说几何意义的话，可以表示为原点到平面上任意点在法线上的投影长度。所以要表示一个面，我们知道面的法向量N，然后再知道面上任意一点P0，就可以求得d = -Dot(N，P0)。

知道了平面方程的表示，我们还需要再温故一下反射的基本原理，开头我们推导Reflect函数时有过一个示意图，这里面我们再画一张针对平面的：

我们要想求得一个点A相对于平面的反射点A‘，根据反射定律可知，|AB| = |A'B|，BA与N同向，已知A点坐标的话，我们就可以求得A’ = A - 2*|AB| * N。

所以，问题就变成了求空间中任意一点A，到平面N·P + d = 0的距离|AB|，再来一张图，推导一下AB距离：

P为平面上任意一点，A为空间中一点，B为点A在平面上的投影点，N为平面法线（为简化问题，N视为单位向量）。PAB构成三角形，BAP夹角θ。从三角形余弦定理易得|BA| = |PA|cosθ；再通过向量点乘的公式，BA与N同向，PA与N夹角也为θ，所以dot（PA，N）= |PA||N|cosθ => cosθ = dot（PA，N）/ （|PA||N|），代入|BA| = |PA|cosθ，|BA| = dot（PA，N）/ |N|，由于N为单位向量，故|BA| = dot（PA，N）。将PA拆分，|BA| = dot（A - P， N） = dot（A，N） - dot（P，N）。这时候就需要我们的平面方程推导公式了，根据点法式的结果我们知道，对于平面上任意一点P，可以表示为P·N + d = 0的形式，即dot（P，N） = -d. 所以最终|BA| = dot（A，N）+ d。进而得到A对于平面的对称点A’ = A - 2（dot（A，N） + d）N。

我们已知N（nx，ny，nz），d的值，A（x，y，z）点作为我们要变换的点，我们需要将A’（x’，y’，z‘）的计算公式表示为矩阵的形式，需要将各分量拆分出来：

x’ = x - 2（x * nx + y * ny + z * nz + d）* nx = （1 - 2nx * nx）x +（ -2nx * ny）y + （-2nx * nz）z + （-2dnx）

y’ = y - 2（x * nx + y * ny + z * nz + d） * ny = （-2nx * ny）x + （1 - 2ny * ny）y + （-2ny * nz）z + （-2dny）

z’ = z - 2（x * nx + y * ny + z * nz + d） * nz = （-2nx * nz）x  + （-2ny * nz）y + （1 - 2nz * nz）z + （-2dnz）

如此复杂的计算公式，我们已经把x，y，z对应的系数和常数抽取出来了，是时候看一下矩阵的威力了，把上述计算公式改为矩阵的形式，Unity是OpenGL风格的矩阵，即矩阵 * 列向量的形式：

至此，我们就得到了用于变换空间中任意一点A相对于平面P·N + d = 0的反射变换矩阵，我们假设其为R。

注：关于平面方程的表示，可以参考这篇文章。

### 平面反射效果实现

下面看一下要怎样用这个矩阵来实现平面反射的效果。首先，也是最重要的，我们需要一个相机，与当前正常相机关于平面对称，也就是说，我们把正常相机变换到这个对称的位置，然后将这个相机的渲染结果输出到一张RT上，就可以得到对称位置的图像了。我们得到了平面反射的矩阵R，下面我们需要考虑的就是在哪个阶段使用这个反射矩阵R。我们知道，渲染物体需要通过MVP变换（可以参考本人之前关于软渲染的blog），物体首先通过M矩阵，从物体空间变换到世界空间，然后通过V矩阵，从世界空间变换到视空间，最后通过投影矩阵P变换到裁剪空间。我们把R矩阵插在V之后，在一个物体在进行MV变换后，变到正常相机坐标系下，然后再进行一次反射变换，就相当于变换到了相对于平面对称的相机坐标系下，然后再进行正常的投影变换，就可以得到反射贴图了。代码如下：
```c#
/********************************************************************
 FileName: PlanarReflection.cs
 Description: 平面反射效果
*********************************************************************/
using UnityEngine; 

[ExecuteInEditMode]
public class PlanarReflection : MonoBehaviour
{

    private Camera reflectionCamera = null;
    private RenderTexture reflectionRT = null;
    private static bool isReflectionCameraRendering = false;
    private Material reflectionMaterial = null;

    private void OnWillRenderObject()
    {
        if (isReflectionCameraRendering)
            return;

        isReflectionCameraRendering = true;
       
        if (reflectionCamera == null)
        {
            var go = new GameObject("Reflection Camera");
            reflectionCamera = go.AddComponent<Camera>();
            reflectionCamera.CopyFrom(Camera.current);
        }
        if (reflectionRT == null)
        {
            reflectionRT = RenderTexture.GetTemporary(1024, 1024, 24);
        }
        //需要实时同步相机的参数，比如编辑器下滚动滚轮，Editor相机的远近裁剪面就会变化
        UpdateCamearaParams(Camera.current, reflectionCamera);
        reflectionCamera.targetTexture = reflectionRT;
        reflectionCamera.enabled = false;

        //根据上文平面定义，需要平面法向量和平面上任意点，此处使用transform.up为法向量，transform.position为平面上的点
        //即需要保证平面模型的原点在平面上，否则可以尝试增加offset偏移
        var reflectM = CaculateReflectMatrix(transform.up, transform.position);

        reflectionCamera.worldToCameraMatrix = Camera.current.worldToCameraMatrix * reflectM;
        
        //需要将背面裁剪反过来，因为仅改变了顶点，没有改变法向量，绕序反向，裁剪会不对
        GL.invertCulling = true;
        reflectionCamera.Render();
        GL.invertCulling = false;

        if (reflectionMaterial == null)
        {
            var renderer = GetComponent<Renderer>();
            reflectionMaterial = renderer.sharedMaterial;
        }
        reflectionMaterial.SetTexture("_ReflectionTex", reflectionRT);

        isReflectionCameraRendering = false;
    }

    Matrix4x4 CaculateReflectMatrix(Vector3 normal, Vector3 positionOnPlane)
    {
        var d = -Vector3.Dot(normal, positionOnPlane);
        var reflectM = new Matrix4x4();
        reflectM.m00 = 1 - 2 * normal.x * normal.x;
        reflectM.m01 = -2 * normal.x * normal.y;
        reflectM.m02 = -2 * normal.x * normal.z;
        reflectM.m03 = -2 * d * normal.x;

        reflectM.m10 = -2 * normal.x * normal.y;
        reflectM.m11 = 1 - 2 * normal.y * normal.y;
        reflectM.m12 = -2 * normal.y * normal.z;
        reflectM.m13 = -2 * d * normal.y;

        reflectM.m20 = -2 * normal.x * normal.z;
        reflectM.m21 = -2 * normal.y * normal.z;
        reflectM.m22 = 1 - 2 * normal.z * normal.z;
        reflectM.m23 = -2 * d * normal.z;

        reflectM.m30 = 0;
        reflectM.m31 = 0;
        reflectM.m32 = 0;
        reflectM.m33 = 1;
        return reflectM;
    }

    private void UpdateCamearaParams(Camera srcCamera, Camera destCamera)
    {
        if (destCamera == null || srcCamera == null)
            return;

        destCamera.clearFlags = srcCamera.clearFlags;
        destCamera.backgroundColor = srcCamera.backgroundColor;
        destCamera.farClipPlane = srcCamera.farClipPlane;
        destCamera.nearClipPlane = srcCamera.nearClipPlane;
        destCamera.orthographic = srcCamera.orthographic;
        destCamera.fieldOfView = srcCamera.fieldOfView;
        destCamera.aspect = srcCamera.aspect;
        destCamera.orthographicSize = srcCamera.orthographicSize;
    }

}

``` 
要使用反射贴图，我们在shader中增加相应的Texture变量即可。不过这个贴图的采样并非使用正常的uv坐标，因为我们的贴图是反射相机的输出的RT，假设这个RT我们输出在屏幕上，反射平面上当前像素点对应位置我们屏幕位置上的位置作为uv坐标才能找到这一点对应RT上的位置。类似之前热扭曲blog中GrabPass的采样操作。需要在vertex shader中通过ComputeScreenPos计算屏幕坐标，fragment shader采样时进行透视校正纹理采样：
```cpp
//Planar Reflection效果
Shader "Reflection/PlanarReflection"
{	
	SubShader
	{
		Tags { "RenderType"="Opaque" }

		Pass
		{
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			
			#include "UnityCG.cginc"

			struct appdata
			{
				float4 vertex : POSITION;
			};

			struct v2f
			{
				float4 screenPos : TEXCOORD0;
				float4 vertex : SV_POSITION;
			};

			sampler2D _ReflectionTex;
			
			v2f vert (appdata v)
			{
				v2f o;
				o.vertex = UnityObjectToClipPos(v.vertex);
				o.screenPos = ComputeScreenPos(o.vertex);
				return o;
			}
			
			fixed4 frag (v2f i) : SV_Target
			{
				fixed4 col = tex2D(_ReflectionTex, i.screenPos.xy / i.screenPos.w);
				//或者
				//fixed4 col = tex2Dproj(_ReflectionTex, i.screenPos);
				return col;
			}
			ENDCG
		}
	}
}
```

反射效果如下：

近处的人物和球体以及远处的栅栏，反射严丝合缝，可以说，Planar Reflection是几种反射中效果最好的一种啦。

### 斜视锥体裁剪

上面的反射看似完美，但是却有一个非常致命的问题，看下面一幅图，我们把其中一个模型向下移动，让其逐渐到平面以下：

似乎不太对。。。虚像倒着升了起来，恩，不太符合常理。其实仔细考虑一下Planar Reflection的原理，应该就能想明白啦。我们把相机在相对于平面对称的位置进行渲染，物体在平面上没有问题，但是物体在平面下的时候，平面并没有挡住虚像的渲染，在反射图中还会会存在反射图像，在采样的时候，就会得到错误的效果。如下图：

C为正常相机，AB为平面，D为反射相机，GH为相机D的近裁剪面，EF为相机D的远裁剪面。在平面AB上方的物体I渲染正常，但是在AB下方的物体，并没有在D的近裁剪面内，所以仍然会渲染，就导致了错误的结果。

那么核心需就是，怎样用反射平面进行裁剪，把在反射平面以下的内容全部裁剪掉。也就是说上图中反射相机D的近裁剪面不再是GH，而是替换为AB平面。这个技术也就是所谓的斜视锥体裁剪-Oblique View Frustum Clippling，可以参考《Oblique View Frustum   Depth Projection and Clipping》这篇论文。

Unity已经为我们提供了一个接口，直接可以对相机和一个平面计算出斜视锥体裁剪投影矩阵，代码如下：

``` cpp
/********************************************************************
 FileName: PlanarReflection.cs
 Description: 平面反射效果
*********************************************************************/
using UnityEngine;

[ExecuteInEditMode]
public class PlanarReflection : MonoBehaviour
{
    private Camera reflectionCamera = null;
    private RenderTexture reflectionRT = null;
    private bool isReflectionCameraRendering = false;
    private Material reflectionMaterial = null;

    private void OnWillRenderObject()
    {
        if (isReflectionCameraRendering)
            return;

        isReflectionCameraRendering = true;
       
        if (reflectionCamera == null)
        {
            var go = new GameObject("Reflection Camera");
            reflectionCamera = go.AddComponent<Camera>();
            reflectionCamera.CopyFrom(Camera.current);
        }
        if (reflectionRT == null)
        {
            reflectionRT = RenderTexture.GetTemporary(1024, 1024, 24);
        }
        //需要实时同步相机的参数，比如编辑器下滚动滚轮，Editor相机的远近裁剪面就会变化
        UpdateCamearaParams(Camera.current, reflectionCamera);
        reflectionCamera.targetTexture = reflectionRT;
        reflectionCamera.enabled = false;

        var reflectM = CaculateReflectMatrix();
        reflectionCamera.worldToCameraMatrix = Camera.current.worldToCameraMatrix * reflectM;

        var normal = transform.up;
        var d = -Vector3.Dot(normal, transform.position);
        var plane = new Vector4(normal.x, normal.y, normal.z, d);
        //用逆转置矩阵将平面从世界空间变换到反射相机空间
        var viewSpacePlane = reflectionCamera.worldToCameraMatrix.inverse.transpose * plane;
        var clipMatrix = reflectionCamera.CalculateObliqueMatrix(viewSpacePlane);
        reflectionCamera.projectionMatrix = clipMatrix;
        
        GL.invertCulling = true;
        reflectionCamera.Render();
        GL.invertCulling = false;

        if (reflectionMaterial == null)
        {
            var renderer = GetComponent<Renderer>();
            reflectionMaterial = renderer.sharedMaterial;
        }
        reflectionMaterial.SetTexture("_ReflectionTex", reflectionRT);

        isReflectionCameraRendering = false;
    }

    Matrix4x4 CaculateReflectMatrix()
    {
        var normal = transform.up;
        var d = -Vector3.Dot(normal, transform.position);
        var reflectM = new Matrix4x4();
        reflectM.m00 = 1 - 2 * normal.x * normal.x;
        reflectM.m01 = -2 * normal.x * normal.y;
        reflectM.m02 = -2 * normal.x * normal.z;
        reflectM.m03 = -2 * d * normal.x;

        reflectM.m10 = -2 * normal.x * normal.y;
        reflectM.m11 = 1 - 2 * normal.y * normal.y;
        reflectM.m12 = -2 * normal.y * normal.z;
        reflectM.m13 = -2 * d * normal.y;

        reflectM.m20 = -2 * normal.x * normal.z;
        reflectM.m21 = -2 * normal.y * normal.z;
        reflectM.m22 = 1 - 2 * normal.z * normal.z;
        reflectM.m23 = -2 * d * normal.z;

        reflectM.m30 = 0;
        reflectM.m31 = 0;
        reflectM.m32 = 0;
        reflectM.m33 = 1;
        return reflectM;
    }

    private void UpdateCamearaParams(Camera srcCamera, Camera destCamera)
    {
        if (destCamera == null || srcCamera == null)
            return;

        destCamera.clearFlags = srcCamera.clearFlags;
        destCamera.backgroundColor = srcCamera.backgroundColor;
        destCamera.farClipPlane = srcCamera.farClipPlane;
        destCamera.nearClipPlane = srcCamera.nearClipPlane;
        destCamera.orthographic = srcCamera.orthographic;
        destCamera.fieldOfView = srcCamera.fieldOfView;
        destCamera.aspect = srcCamera.aspect;
        destCamera.orthographicSize = srcCamera.orthographicSize;
    }

    private Matrix4x4 CaculateObliqueViewFrustumMatrix(Vector4 plane, Camera camera)
    {
        var viewSpacePlane = camera.worldToCameraMatrix.inverse.transpose * plane;
        return camera.CalculateObliqueMatrix(viewSpacePlane);
    }
}
```

这样，通过这样一个API，我们很容易可以求得被视空间的一个平面裁剪过的投影矩阵，再应用回摄像机，这样就不会出现穿帮啦：

API虽然简单，但是API背后的原理还是需要一番推导的，下面看一下斜视锥体裁剪的推导过程。

首先需要几个预备的知识点。第一点，既然需要平面裁剪，就免不了对平面进行一些坐标空间的变换，上面我们推导过平面的表示，可以用平面法向量和平面上一点与法向量点积的相反数。平面的变换与法线变换类似，不能直接进行变换，对于非uniform类型可能导致法线不垂直于平面，所以平面的变换也采用矩阵逆转置的方式进行（关于法线的变换，可以参考之前描边效果的blog）。即，如果我们已知一个View空间的平面Pv，要想将其转化到裁剪空间Pc，就需要投影矩阵M的逆转置矩阵：

这里就需要线性代数里面的一个性质啦，如果一个矩阵可逆，那么这个矩阵的转置的逆等于逆的转置，对于上面公式来说：

进而可以将Pv和Pc的变换公式进一步化简为：

下面是第二个预备知识点，关于裁剪空间的。我们知道，经过投影矩阵变换后，会被变换到裁剪空间（实际上此时还没有经过透视除法，属于用齐次坐标系表示的坐标，此处我们为了方便表示最终的立方体，假设进行了透视除法，各个分量除以w分量，将变换的结果置为一个标准的立方体，二者的表示结果实际上是等价的），视锥体这个平头截体会被变换成一个立方体（OpenGL是正方体，区间（-1, 1），DX是普通立方体，xy区间（-1, 1），z区间（0, 1）），我们以Unity用的OpenGL风格变换为例，最终一个视锥体的前后左右上下六个面在裁剪空间就都会被变换到标准立方体的六个面上。

通过我们之前推导的平面表示的方程，我们很容易地可以表示出裁剪空间下视锥体六个面的平面方程，然后根据，我们就可以求得在视空间下视锥体六个面的平面方程，如下图（该图片来自上文中提到的论文）：

根据上图中的变换结果，N = M4 + M3，F = M4 - M3，我们需要用一个自定义的平面P来代替默认的近裁剪平面Near，也就是说新的P裁剪面也需要满足P = M4 + M3这个条件。要想修改近裁剪面N使之变成P，我们就需要调整M4或者M3这两个向量中的值。M4是投影矩阵的最后一行，包含了z值透视投影等信息，是后续透视除法必须的，所以我们就只能改动M3这一行。

M3' = P - M4，F = M4 - M3，F' = M4 - M3' => F' = 2M4 - P

似乎我们只需要求出M3'然后带入就大功告成了。不过这样有一个问题，在于本身P可能不平行于XY平面，得到的远裁剪面也可能不平行，保证了近裁剪面正确，远裁剪面的位置是一个未知的值，可能截断了原来的视锥体，这样可能会导致一些不该被裁剪掉的物体也被裁减掉了；也可能偏移出视锥体很远，进而对深度值造成影响。因此，我们需要考虑让远裁剪面也位于一个合适的位置。

由于公式F' = 2M4 - P，M4不能动，P平面也不能动，那么我们可以考虑给P乘以一个系数，一个平面方程，整体乘以一个系数后，表示的仍然是原来的平面，即F' = 2M4 - uP，这样M4，uP都没有变化，但是最终的F'就会变化了。我们可以求得一个合适的u，使远裁剪面不截断原来的视锥体同时又与P平面夹角最小。那么这个平面就是过视锥体原始边界的一个顶点即可，如下图所示：

HG为原始的近裁剪面，FE为原始的远裁剪面，AB为新的近裁剪面（也就是上文的P平面），在裁剪空间（此时应该没进行透视除法，但是为了方便，我们假设w = 1）下的坐标边界为E（+-1，+-1, 1, 1），那么我们要求的远裁剪面的边界点就是AB平面面对的裁剪面的边界点即可。也就是说，E点xy坐标的正负取决于AB平面的朝向，我们可以用AB平面（P平面）的法线进行表示，由于我们目前可以得到视空间的P平面方程，而E点坐标目前是裁剪空间的，不过投影变换不会再去改变xy的符号，所以我们直接取视空间P平面xy值的符号即可。

那么，裁剪空间下E点坐标为E（Sign（P.x）， Sign（P.y），1，1），我们可以将其乘以投影矩阵的逆矩阵变换回视空间的E点，即E = （Sign（P.x）， Sign（P.y），1，1） *  ProjectionMatrix. Inverse。此时，我们知道了新的远裁剪面方程F' = 2M4 - P，又知道新的远裁剪面过E点，根据平面公式，F’·E = 0可得：

（2M4 - uP）·E = 0 => u= 2*（M4 · E） /  （P · E）

我们就可以求得u值，进而根据M3' = uP - M4得到最终的M3值，对投影矩阵进行修改。

不过，此处有一个小优化。首先，看一下OpenGL版本的投影矩阵：

投影矩阵的逆矩阵：

关于投影矩阵的推导，可以参考之前软渲染这篇blog，不过本人之前推导的是DX风格的矩阵，GL风格原理也是一样的。

M4（0，0，-1，0），E = （+-1，+-1，1，1） * Projection. Inverse，看似比较复杂，但是实际上M4·E ，由于M4仅z项非零，我们只看E点的z项即可。那么其实只需要考虑的是Porjection. Inverse的第三行与E点乘的结果，而Porjection. InverseM3  = （0，0，0，-1），即E.z = （0，0，0，-1）· （+-1， +-1， 1， 1） = -1，最终得到M4·E为定值1，因此可以直接省去公式u= 2*（M4 · E） /  （P · E）中M4·E的计算：

u = 2 / （P · E）

完整C#代码如下：
```c#
/********************************************************************
 FileName: PlanarReflection.cs
 Description: 平面反射效果
*********************************************************************/
using UnityEngine; 

[ExecuteInEditMode]
public class PlanarReflection : MonoBehaviour
{

    private Camera reflectionCamera = null;
    private RenderTexture reflectionRT = null;
    private bool isReflectionCameraRendering = false;
    private Material reflectionMaterial = null;

    private void OnWillRenderObject()
    {
        if (isReflectionCameraRendering)
            return;

        isReflectionCameraRendering = true;
       
        if (reflectionCamera == null)
        {
            var go = new GameObject("Reflection Camera");
            reflectionCamera = go.AddComponent<Camera>();
            reflectionCamera.CopyFrom(Camera.current);
            reflectionCamera.hideFlags = HideFlags.HideAndDontSave;
        }
        if (reflectionRT == null)
        {
            reflectionRT = RenderTexture.GetTemporary(1024, 1024, 24);
        }
        //需要实时同步相机的参数，比如编辑器下滚动滚轮，Editor相机的远近裁剪面就会变化
        UpdateCamearaParams(Camera.current, reflectionCamera);
        reflectionCamera.targetTexture = reflectionRT;
        reflectionCamera.enabled = false;

        var reflectM = CaculateReflectMatrix();
        reflectionCamera.worldToCameraMatrix = Camera.current.worldToCameraMatrix * reflectM;

        var normal = transform.up;
        var d = -Vector3.Dot(normal, transform.position);
        var plane = new Vector4(normal.x, normal.y, normal.z, d);
        //用逆转置矩阵将平面从世界空间变换到反射相机空间
        var clipMatrix = CalculateObliqueMatrix(plane, reflectionCamera);
        reflectionCamera.projectionMatrix = clipMatrix;
        
        GL.invertCulling = true;
        reflectionCamera.Render();
        GL.invertCulling = false;

        if (reflectionMaterial == null)
        {
            var renderer = GetComponent<Renderer>();
            reflectionMaterial = renderer.sharedMaterial;
        }
        reflectionMaterial.SetTexture("_ReflectionTex", reflectionRT);

        isReflectionCameraRendering = false;
    }

    Matrix4x4 CaculateReflectMatrix()
    {
        var normal = transform.up;
        var d = -Vector3.Dot(normal, transform.position);
        var reflectM = new Matrix4x4();
        reflectM.m00 = 1 - 2 * normal.x * normal.x;
        reflectM.m01 = -2 * normal.x * normal.y;
        reflectM.m02 = -2 * normal.x * normal.z;
        reflectM.m03 = -2 * d * normal.x;

        reflectM.m10 = -2 * normal.x * normal.y;
        reflectM.m11 = 1 - 2 * normal.y * normal.y;
        reflectM.m12 = -2 * normal.y * normal.z;
        reflectM.m13 = -2 * d * normal.y;

        reflectM.m20 = -2 * normal.x * normal.z;
        reflectM.m21 = -2 * normal.y * normal.z;
        reflectM.m22 = 1 - 2 * normal.z * normal.z;
        reflectM.m23 = -2 * d * normal.z;

        reflectM.m30 = 0;
        reflectM.m31 = 0;
        reflectM.m32 = 0;
        reflectM.m33 = 1;
        return reflectM;
    }

    private void OnDisable()
    {
        DestroyImmediate(reflectionCamera.gameObject);
        RenderTexture.ReleaseTemporary(reflectionRT);
        reflectionCamera = null;
        reflectionRT = null;
    }

    private void UpdateCamearaParams(Camera srcCamera, Camera destCamera)
    {
        if (destCamera == null || srcCamera == null)
            return;

        destCamera.clearFlags = srcCamera.clearFlags;
        destCamera.backgroundColor = srcCamera.backgroundColor;
        destCamera.farClipPlane = srcCamera.farClipPlane;
        destCamera.nearClipPlane = srcCamera.nearClipPlane;
        destCamera.orthographic = srcCamera.orthographic;
        destCamera.fieldOfView = srcCamera.fieldOfView;
        destCamera.aspect = srcCamera.aspect;
        destCamera.orthographicSize = srcCamera.orthographicSize;
    }

    private Matrix4x4 CalculateObliqueMatrix(Vector4 plane, Camera camera)
    {
        var viewSpacePlane = camera.worldToCameraMatrix.inverse.transpose * plane;
        var projectionMatrix = camera.projectionMatrix;

        var clipSpaceFarPanelBoundPoint = new Vector4(Mathf.Sign(viewSpacePlane.x), Mathf.Sign(viewSpacePlane.y), 1, 1);
        var viewSpaceFarPanelBoundPoint = camera.projectionMatrix.inverse * clipSpaceFarPanelBoundPoint;
        
        var m4 = new Vector4(projectionMatrix.m30, projectionMatrix.m31, projectionMatrix.m32, projectionMatrix.m33);
        //u = 2 * (M4·E)/(E·P)，而M4·E == 1，化简得
        //var u = 2.0f * Vector4.Dot(m4, viewSpaceFarPanelBoundPoint) / Vector4.Dot(viewSpaceFarPanelBoundPoint, viewSpacePlane);
        var u = 2.0f / Vector4.Dot(viewSpaceFarPanelBoundPoint, viewSpacePlane);
        var newViewSpaceNearPlane = u * viewSpacePlane;

        //M3' = P - M4
        var m3 = newViewSpaceNearPlane - m4;

        projectionMatrix.m20 = m3.x;
        projectionMatrix.m21 = m3.y;
        projectionMatrix.m22 = m3.z;
        projectionMatrix.m23 = m3.w;

        return projectionMatrix;
    }

}

``` 
效果与API版本的一致，均可以裁减掉位于原始近裁剪面和平面之间的内容：

### Screen Space Reflection

Screen Space Reflection-屏幕空间反射（SSR），是一个逼格绝对是SSR级别的技术，但是效果有些情况下有硬伤，而且性能堪忧，在前向渲染的情况下性价比较低（需要额外的全屏深度+法线），移动平台就更不知道有没有哪位勇者尝试过了。不过这个技术自从Crytek（Local Reflection）提出之后，各种大作争相把这个技术集成了进来，并且衍生出了很多变种实现，直到最近仍然还在发展，铺天盖地的paper和ppt看得我眼花缭乱。为何SSR会如此受追捧，还是需要看一下SSR的原理，我们就会了解其优缺点了。

### SSR的基本原理与优缺点

SSR，凡是带SS（屏幕空间的）技术，最近这几年发展的很多，如屏幕空间环境光遮蔽，屏幕空间阴影，屏幕空间次表面散射等等技术，一方面是屏幕空间计算降低了一些消耗，降低不必要的重复计算，再者主要因为延迟渲染，一些操作不方便在直接渲染时进行，所以SS系列的技术随着延迟渲染技术发扬光大了，其实延迟渲染本身就是SS技术。而SSR就是最明显的例子之一，Unity官方后处理包的SSR也只支持延迟渲染。

根据上面的几种反射，我们已经知道，要想计算反射，我们需要求出反射向量。这就需要该点的法线方向，以及该点的视线方向，相机位置已知的话，我们只需要求得该点的位置就可以求得视线方向了。即我们需要物体在视空间的位置以及法线（假设我们在视空间计算反射的话），但是SS阶段，一听就是个后处理，这个阶段我们已经没有每个物体的输入了，所以我们就需要从几个特殊的RT下手，找到所需要的信息。在深度相关blog中，我们推导过通过深度图重建世界空间坐标或视空间坐标，以此可以求得物体在视空间的位置。另外，视空间的法线可以通过DepthNormalTexture（前向）或者GBuffer（延迟，需转换到视空间）得到。即该效果需要全屏深度图，全屏法线图。我们在前向渲染可以通过DepthNormalTexture得到这两者，而在延迟渲染阶段，GBuffer中就包含了法线等信息，我们可以直接使用。

求得了全屏幕各个像素点的反射向量后，我们也知道了每个像素点在视空间的位置，要想求得反射值，要怎么办呢？其实如果不看变种的话，基本SSR的原理是各种反射里面最简单的。那就是直接从当前点出发，沿着反射方向进行步进，直到碰到东西位为止，碰到的位置对应的颜色值就是该点的反射颜色。哇，这不就是RayMarching嘛，在体积光的blog中我们也使用过类似的方式。那么接下来就只剩下一个问题了，就是怎样确定碰到了东西。还是Depth，因为我们要反射的所有物体都在屏幕上，不可能出现屏幕外的东西，而一旦沿着反射方向的光线已经超过了当前这一点的屏幕空间深度，那么就认为光线已经有了碰撞，直接采样该点对应的屏幕空间Frame Buffer值就得到了反射值。

当然，这只是最基本的原理，如果完全按照这个原理不进行进一步处理的话，效果不是没法看就是性能极差或者穿帮太多。不过，以上已经足够我们判断SSR技术的优缺点了。

优点：

1.可以实现真·实时反射，并且可以用于任意面，无需平面。

2.无需额外DrawCall，没有Planar Reflection那种翻倍DC的问题，计算都在GPU，解放CPU，尤其在被反射对象shader及其复杂的情况下，更能节省大幅反射渲染的消耗。

3.一个后处理，无需大规模改动材质系统，容易集成。

4.最关键的，延迟渲染实现实时反射（静态的还reflection probe还是可以用的），貌似也没啥别的好办法啊！！！总不能再来个前向的Planar Reflection吧？

缺点：

1.光线追踪！！！一听这词儿，就不是个省的效果，尤其手机那GPU。虽然省了CPU，但是对GPU来说负载很大。（后续优化可以大幅度降低消耗，但是相比于普通的后处理，还是很费很费，而且有大量分支计算）。

2.需要全屏深度和全屏法线。延迟渲染的话可以免费拿到，然而前向渲染的话，单说渲染一遍DepthNormalMap的消耗，恐怕就和Planar Reflection翻倍的DC差不多了，更不要说后续的计算（当然渲染深度图的shader比较简单，不过Planar Reflection也可以用lod shader渲染嘛，而且不管shader复杂度，DC在cpu的消耗是移动设备上消耗最大的一点）。当然DepthNormal可能还有别的用处，毕竟这个东西当年Crytek定义为一个Mini G Buffer，还是很好很好用的，一些好玩的效果都可以使用DepthNormal实现。

3.效果硬伤，这是这个技术本身的瓶颈，原理上可能就解决不了。只能反射屏幕上的出现过的像素。如果不在屏幕内的，就完全不会反射。比如，角色正对着镜子，背后是摄像机，那么，镜子里面是没有角色的正脸反射的。类似的，还有在视口边界经常容易出现反射丢失的情况。

个人感觉，SSR对于延迟渲染下是一个比较不错的实时反射的方案。

### 基本SSR效果实现

上面大概描述了一遍SSR的基本原理，我们按照这个原理实现一版代码。此处我就先使用前向渲染+DepthNormalTexture的方式进行SSR渲染，暂时没有切换到延迟渲染。

shader代码如下：
```cpp
//Screen Space Reflection效果
Shader "Reflection/ScreenSpaceReflection"
{
	Properties
	{
		_MainTex ("Texture", 2D) = "white" {}
	}
	
	SubShader
	{
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }

			
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			
			#include "UnityCG.cginc"
			
			struct appdata
			{
				float4 vertex : POSITION;
				float2 uv : TEXCOORD0;
			};

			struct v2f
			{
				float4 vertex : SV_POSITION;
				float2 uv : TEXCOORD0;	
				float3 viewRay : TEXCOORD1;
			};

			sampler2D _MainTex;
			float4 _MainTex_ST;//(1 / width, 1 / height, width, height)
			sampler2D _CameraDepthTexture;
			//使用外部传入的matrix，实际上等同于下面两个unity内置matrix
			//似乎在老版本unity中在pss阶段使用正交相机绘制Quad，矩阵会被替换，2017.3版本测试使用内置矩阵也可以
			float4x4 _InverseProjectionMatrix;//unity_CameraInvProjection
			float4x4 _CameraProjectionMatrix;//unity_CameraProjection
			float _maxRayMarchingDistance;
			float _maxRayMarchingStep;
			float _rayMarchingStepSize;
			float _depthThickness;
			
			sampler2D _CameraDepthNormalsTexture;
			
			bool checkDepthCollision(float3 viewPos, out float2 screenPos)
			{
				float4 clipPos = mul(_CameraProjectionMatrix, float4(viewPos, 1.0));

				clipPos = clipPos / clipPos.w;
				screenPos = float2(clipPos.x, clipPos.y) * 0.5 + 0.5;
				float4 depthnormalTex = tex2D(_CameraDepthNormalsTexture, screenPos);
				float depth = DecodeFloatRG(depthnormalTex.zw) * _ProjectionParams.z;
				//判断当前反射点是否在屏幕外，或者超过了当前深度值
				return screenPos.x > 0 && screenPos.y > 0 && screenPos.x < 1.0 && screenPos.y < 1.0 && depth < -viewPos.z;
			}
			
			bool viewSpaceRayMarching(float3 rayOri, float3 rayDir, out float2 hitScreenPos)
			{
				int maxStep = _maxRayMarchingStep;
				UNITY_LOOP
				for(int i = 0; i < maxStep; i++)
				{
					float3 currentPos = rayOri + rayDir * _rayMarchingStepSize * i;
					if (length(rayOri - currentPos) > _maxRayMarchingDistance)
						return false;
					if (checkDepthCollision(currentPos, hitScreenPos))
					{
						return true;
					}
				}
				return false;
			}
			
			v2f vert (appdata v)
			{
				v2f o;
				o.vertex = UnityObjectToClipPos(v.vertex);
				o.uv = TRANSFORM_TEX(v.uv, _MainTex);
				
				float4 clipPos = float4(v.uv * 2 - 1.0, 1.0, 1.0);
				float4 viewRay = mul(_InverseProjectionMatrix, clipPos);
				o.viewRay = viewRay.xyz / viewRay.w;
				return o;
			}
			
			fixed4 frag (v2f i) : SV_Target
			{
				fixed4 mainTex = tex2D(_MainTex, i.uv);
				float linear01Depth;
				float3 viewNormal;
				
				float4 cdn = tex2D(_CameraDepthNormalsTexture, i.uv);
				DecodeDepthNormal(cdn, linear01Depth, viewNormal);
				//重建视空间坐标
				float3 viewPos = linear01Depth * i.viewRay;
				float3 viewDir = normalize(viewPos);
				viewNormal = normalize(viewNormal);
				//视空间方向反射方向
				float3 reflectDir = reflect(viewDir, viewNormal);
				float2 hitScreenPos = float2(0,0);
				//从该点开始RayMarching
				if (viewSpaceRayMarching(viewPos, reflectDir, hitScreenPos))
				{
					float4 reflectTex = tex2D(_MainTex, hitScreenPos);
					mainTex.rgb += reflectTex.rgb;
				}
				return mainTex;
			}
			
			ENDCG
		}
	}
}
```

C#代码如下：
```c#
/********************************************************************
 FileName: ScreenSpaceReflection.cs
 Description: 屏幕空间反射SSR
*********************************************************************/
using System. Collections; 
using System. Collections. Generic; 
using UnityEngine; 
[ExecuteInEditMode]
public class ScreenSpaceReflection : MonoBehaviour
{

    Material reflectionMaterial = null;
    Camera currentCamera = null;
    [Range(0, 1000.0f)]
    public float maxRayMarchingDistance = 500.0f;
    [Range(0, 256)]
    public int maxRayMarchingStep = 64;
    [Range(0, 2.0f)]
    public float rayMarchingStepSize = 0.05f;
    [Range(0, 2.0f)]
    public float depthThickness = 0.01f;
    private void Awake()
    {
        var shader = Shader.Find("Reflection/ScreenSpaceReflection");
        reflectionMaterial = new Material(shader);
        currentCamera = GetComponent<Camera>();
    }

    private void OnEnable()
    {
        currentCamera.depthTextureMode |= DepthTextureMode.DepthNormals;    
    }

    private void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.DepthNormals;
    }

    private void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (reflectionMaterial == null)
        {
            Graphics.Blit(source, destination);
            return;
        }

        reflectionMaterial.SetMatrix("_InverseProjectionMatrix", currentCamera.projectionMatrix.inverse);
        reflectionMaterial.SetMatrix("_CameraProjectionMatrix", currentCamera.projectionMatrix);

        reflectionMaterial.SetFloat("_maxRayMarchingDistance", maxRayMarchingDistance);
        reflectionMaterial.SetFloat("_maxRayMarchingStep", maxRayMarchingStep);
        reflectionMaterial.SetFloat("_rayMarchingStepSize", rayMarchingStepSize);
        reflectionMaterial.SetFloat("_depthThickness", depthThickness);
        Graphics.Blit(source, destination, reflectionMaterial, 0);
    }

}

``` 
这里，我们使用了DepthNormalTexture，其中的Normal取出来就是视空间的法线值，可以直接使用，深度是视空间的01区间深度，我们需要使用这个深度进行视空间坐标重建的操作，可以参考本人之前的深度相关内容的blog，此处不再赘述。然后根据反射计算出视空间的反射方向，进而每像素进行RayMarching操作，每次步进一定步长，查询是否与超过了当前视深度，超过则认为已经碰撞，对该点采样就是对应的反射颜色。然后将这个颜色直接叠加到屏幕颜色上。

效果如下：

### 逆着视线方向的光线拖影的问题

效果嘛，似乎不太对，每个反射下面都有一个长长的尾巴，感觉看上去好像是已经超过了反射的深度值，我们没有及时制止反射的计算，其实并不是，我们在光线碰撞到之后，就会直接返回，因此这个尾巴另有原因。再看我们判断碰撞的条件，我们只考虑了反射光线在深度之后就认为是碰撞了，但是我们没考虑这个反射光线是从哪里来的。看下面一张示意图：

理想情况下，我们的反射光线是KH，沿KH方向步进，直到深度达到视空间深度后，认为已经进入了圆IEG中，停止步进，采样该点作为颜色值。但是如果类似有CD反射平面，MF反射方向并非沿着视线方向，而是与视线方向相反，而MF方向的点一定满足大于视方向深度，所以在MF方向上步进就会马上返回采样成功。但是这个位置采样并不正确，更不用说这个点本身就没有渲染信息，多个类似的光方向采样都不正确造成了上面的穿帮效果。

要解决这个问题，我们先往简单了想，实际上我们希望的反射都是在屏幕空间内有信息的才需要反射，而类似F点的，即使采样正确，这个反射信息也没有，采样出来也是错误的结果。那么，索性，就直接让我们只取距离视空间深度附近的采样点即可。也就是说我们需要一个阈值进行判断，当深度超过H所在的深度，并且不超过太多的情况下才认为是正确的深度碰撞，其他部分都舍弃掉。

我们稍微修改一下碰撞判断函数，增加一个深度阈值进行判断：
```cpp
bool checkDepthCollision(float3 viewPos, out float2 screenPos)
{
	float4 clipPos = mul(_CameraProjectionMatrix, float4(viewPos, 1.0));

	clipPos = clipPos / clipPos.w;
	screenPos = float2(clipPos.x, clipPos.y) * 0.5 + 0.5;
	float4 depthnormalTex = tex2D(_CameraDepthNormalsTexture, screenPos);
	float depth = DecodeFloatRG(depthnormalTex.zw) * _ProjectionParams.z;
	//判断当前反射点是否在屏幕外，或者超过了当前深度值并且不超过太多的情况下
	return screenPos.x > 0 && screenPos.y > 0 && screenPos.x < 1.0 && screenPos.y < 1.0 && depth < -viewPos.z && depth + _depthThickness > -viewPos.z;
}
```

效果如下：

这一次，看起来稍微好了一些，没有每个反射下面的拖影了，反射也比较清晰。至于左下角和右下角缺角的情况，这就是所谓的SSR的效果硬伤，反射对象在屏幕外，没有信息，反射不到。所以大部分SSR的效果都是封闭室内，或者向下视角，尽可能包含被反射物体。

还有一种方法可以得到更加逼真的结果，就是渲染一张背面的深度图，进而得到物体的厚度，但是开销比较大，这里就不再实现了。

### 二分搜索优化

似乎可以结束这篇长长的blog了吗？非也！目前的效果，RayMarching过程中步长很小，迭代次数很多，所以超级费！这和之前体积光的raymarching还略有不同，体积光中没有采样，这里的每次步进都需要采样深度图。好在我们加了UNITY_LOOP，让shader不在编译时展开，否则可能指令数就直接超限制了。步长大一些之后就会出现断层的问题，而迭代次数小的话又会出现可能稍微远一点的内容就反射不到的问题。如下图，略微加大步长：

所以下一个问题就是怎样用较低的步进次数和较大的步长进行RayMarching。

既然说到循环步进，最简单的优化就是二分法了。也就是说，我们开始的时候用一个比较大的步长，步进，如果遇到碰撞，那么缩回去一次，改二分之一步长再步进，再碰，再退缩，再碰，再退缩，以此类推，直到达到阈值或者超过缩步长的次数。

重新修改后的C#代码：
```c#
/********************************************************************
 Description: 屏幕空间反射SSR
*********************************************************************/
using System. Collections; 
using System. Collections. Generic; 
using UnityEngine; 
[ExecuteInEditMode]
public class ScreenSpaceReflection : MonoBehaviour
{

    Material reflectionMaterial = null;
    Camera currentCamera = null;
    [Range(0, 1000.0f)]
    public float maxRayMarchingDistance = 500.0f;
    [Range(0, 256)]
    public int maxRayMarchingStep = 64;
    [Range(0, 32)]
    public int maxRayMarchingBinarySearchCount = 8;
    [Range(0, 8.0f)]
    public float rayMarchingStepSize = 0.05f;
    [Range(0, 2.0f)]
    public float depthThickness = 0.01f;
    private void Awake()
    {
        var shader = Shader.Find("Reflection/ScreenSpaceReflection");
        reflectionMaterial = new Material(shader);
        currentCamera = GetComponent<Camera>();
    }

    private void OnEnable()
    {
        currentCamera.depthTextureMode |= DepthTextureMode.DepthNormals;    
    }

    private void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.DepthNormals;
    }

    private void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (reflectionMaterial == null)
        {
            Graphics.Blit(source, destination);
            return;
        }

        reflectionMaterial.SetMatrix("_InverseProjectionMatrix", currentCamera.projectionMatrix.inverse);
        reflectionMaterial.SetMatrix("_CameraProjectionMatrix", currentCamera.projectionMatrix);

        reflectionMaterial.SetFloat("_maxRayMarchingBinarySearchCount", maxRayMarchingBinarySearchCount);
        reflectionMaterial.SetFloat("_maxRayMarchingDistance", maxRayMarchingDistance);
        reflectionMaterial.SetFloat("_maxRayMarchingStep", maxRayMarchingStep);
        reflectionMaterial.SetFloat("_rayMarchingStepSize", rayMarchingStepSize);
        reflectionMaterial.SetFloat("_depthThickness", depthThickness);
        Graphics.Blit(source, destination, reflectionMaterial, 0);
    }

}

``` 
Shader代码：
```cpp
//Screen Space Reflection效果,Binary Search
Shader "Reflection/ScreenSpaceReflection"
{
	Properties
	{
		_MainTex ("Texture", 2D) = "white" {}
	}
	
	SubShader
	{
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }

			
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			
			#include "UnityCG.cginc"
			
			struct appdata
			{
				float4 vertex : POSITION;
				float2 uv : TEXCOORD0;
			};

			struct v2f
			{
				float4 vertex : SV_POSITION;
				float2 uv : TEXCOORD0;	
				float3 viewRay : TEXCOORD1;
			};

			sampler2D _MainTex;
			float4 _MainTex_ST;//(1 / width, 1 / height, width, height)
			sampler2D _CameraDepthTexture;
			//使用外部传入的matrix，实际上等同于下面两个unity内置matrix
			//似乎在老版本unity中在pss阶段使用正交相机绘制Quad，矩阵会被替换，2017.3版本测试使用内置矩阵也可以
			float4x4 _InverseProjectionMatrix;//unity_CameraInvProjection
			float4x4 _CameraProjectionMatrix;//unity_CameraProjection
			float _maxRayMarchingDistance;
			float _maxRayMarchingStep;
			float _maxRayMarchingBinarySearchCount;
			float _rayMarchingStepSize;
			float _depthThickness;
			
			sampler2D _CameraDepthNormalsTexture;
			
			bool checkDepthCollision(float3 viewPos, out float2 screenPos, inout float depthDistance)
			{
				float4 clipPos = mul(_CameraProjectionMatrix, float4(viewPos, 1.0));

				clipPos = clipPos / clipPos.w;
				screenPos = float2(clipPos.x, clipPos.y) * 0.5 + 0.5;
				float4 depthnormalTex = tex2D(_CameraDepthNormalsTexture, screenPos);
				float depth = DecodeFloatRG(depthnormalTex.zw) * _ProjectionParams.z;
				//判断当前反射点是否在屏幕外，或者超过了当前深度值
				depthDistance = abs(depth + viewPos.z);
				return screenPos.x > 0 && screenPos.y > 0 && screenPos.x < 1.0 && screenPos.y < 1.0 && depth < -viewPos.z;
			}
			
			bool viewSpaceRayMarching(float3 rayOri, float3 rayDir, float currentRayMarchingStepSize, inout float depthDistance, inout float3 currentViewPos, inout float2 hitScreenPos)
			{
				int maxStep = _maxRayMarchingStep;	
				UNITY_LOOP
				for(int i = 0; i < maxStep; i++)
				{
					float3 currentPos = rayOri + rayDir * currentRayMarchingStepSize * i;
					if (length(rayOri - currentPos) > _maxRayMarchingDistance)
						return false;
					if (checkDepthCollision(currentPos, hitScreenPos, depthDistance))
					{
						currentViewPos = currentPos;
						return true;
					}
				}
				return false;
			}
			
			bool binarySearchRayMarching(float3 rayOri, float3 rayDir, inout float2 hitScreenPos)
			{
				float currentStepSize = _rayMarchingStepSize;
				float3 currentPos = rayOri;
				float depthDistance = 0;
				UNITY_LOOP
				for(int i = 0; i < _maxRayMarchingBinarySearchCount; i++)
				{
					if(viewSpaceRayMarching(rayOri, rayDir, currentStepSize, depthDistance, currentPos, hitScreenPos))
					{
						if (depthDistance < _depthThickness)
						{
							return true;
						}
						rayOri = currentPos - rayDir * currentStepSize;
						currentStepSize *= 0.5;
					}
					else
					{
						return false;
					}
				}
				return false;
			}
			
			v2f vert (appdata v)
			{
				v2f o;
				o.vertex = UnityObjectToClipPos(v.vertex);
				o.uv = TRANSFORM_TEX(v.uv, _MainTex);
				
				float4 clipPos = float4(v.uv * 2 - 1.0, 1.0, 1.0);
				float4 viewRay = mul(_InverseProjectionMatrix, clipPos);
				o.viewRay = viewRay.xyz / viewRay.w;
				return o;
			}
			
			fixed4 frag (v2f i) : SV_Target
			{
				fixed4 mainTex = tex2D(_MainTex, i.uv);
				float linear01Depth;
				float3 viewNormal;
				
				float4 cdn = tex2D(_CameraDepthNormalsTexture, i.uv);
				DecodeDepthNormal(cdn, linear01Depth, viewNormal);
				//重建视空间坐标
				float3 viewPos = linear01Depth * i.viewRay;
				float3 viewDir = normalize(viewPos);
				viewNormal = normalize(viewNormal);
				//视空间方向反射方向
				float3 reflectDir = reflect(viewDir, viewNormal);
				float2 hitScreenPos = float2(0,0);
				//从该点开始RayMarching
				if (binarySearchRayMarching(viewPos, reflectDir, hitScreenPos))
				{
					float4 reflectTex = tex2D(_MainTex, hitScreenPos);
					mainTex.rgb += reflectTex.rgb;
				}
				return mainTex;
			}
			
			ENDCG
		}
	}
}
```

这样，在步长可以设置为2左右，每次步进10次左右，二分搜索6次就可以达到之前0.2步长，256次步进的效果：

不过从上图我们也看出，反射信息缺失是多么蛋疼，人挡住的栅栏反射是看不到的，人的小腿不在画面中，因而反射也是缺失的。官方的Post Processing Stack中也是一样，但是似乎做了一些模糊，并且反射效果较弱，但是看官方的拖影很严重：

### 屏幕空间光栅化RayMarching

基本的SSR到此就结束了。然而不断有对SSR进行的优化，我们还是简单看一下。最著名的一个优化就是按照屏幕空间光栅化的方式进行RayMarching。这个优化在这篇论文中提出，也是后续各方SSR参考的重点。

我们上文是通过视空间进行RayMarching的，三维空间有一个很大的性质就是近大远小，就是所谓的透视效果，那么我们在视空间步进一步，光栅化后对应到屏幕空间，可能就不一定是一个像素了。即在远处可能我们步进一步或者几步结果只对应到一个像素，这就浪费了计算；而到近处时，可能我们在视空间步进一个单位，就已经跨过好多个像素了，这又涉及到了采样不足，可能效果不好。如下图（图片来自上述论文）。

为了解决以上的问题，屏幕空间进行RayMarching的方案就应运而生了，我们希望在屏幕空间可以保证沿着反射光线方向进行步进，不漏掉一个像素，也不重复采样一个像素。其实也就是光栅化的方式，之前在本人的软渲染实现过Bresenham算法，不过那是为使用整型并减少除法使之在CPU模拟计算时尽可能快速，实际上在shader上使用GPU再去实现，就没必要考虑整型的问题了，浮点型更快，因此可以直接就使用更加容易理解一些的画线算法-DDA算法，直接用斜率计算。

我们可以用最大步进距离，步进出发点和方向得到结束点。另外，我们需要剔除近裁剪面后面的点，并且可以直接剔除与视线方向一致的反射光线。既然涉及到屏幕空间计算了，首先需要将光线的出发点和结束点转化到屏幕空间（注意，并非shader中使用的01空间，而是真正的屏幕空间<或者渲染RT>，即乘以过屏幕<或者RT>长宽的屏幕空间）。可以将视空间的顶点用投影矩阵变换到裁剪空间，然后需要做透视除法，转换到NDC空间，再进行*0.5 + 0.5，转换到（0, 1）屏幕区间，最后再乘以屏幕长宽，转换为真正的屏幕空间。我们可以暂时不考虑透视除法，使之仍然用齐次坐标表示，然后将着一些列变换封装到一个矩阵中，然后将矩阵与投影矩阵相乘，得到的结果矩阵就可以直接将视空间的顶点转换到屏幕空间了。直接用点乘以矩阵得到的还是齐次空间下的坐标。这个齐次坐标的w值，其实也就是视空间的z值非常重要。直接对屏幕空间齐次坐标除以w，就得到了透视除法后的屏幕坐标。

得到了光线的起始点和结束点，我们就可以在屏幕空间进行光线的步进。这个过程与光栅化过程有些类似，即需要沿着光线的方向绘制一条直线，并且需要将一些信息进行插值计算。为何需要插值呢，很明显我们将计算转化到屏幕空间，是为了控制步进的距离，虽然这样做步进控制得比较好，但是我们需要这一点的深度信息来和当前深度图（存储的是视空间深度<或1/z>，取决于是DepthNormalTexture还是普通DepthTexture）该像素的深度进行比较来判断是否发生了碰撞。每次步进再将其反推回视空间肯定不现实，那么就可以考虑类似光栅化中，我们在vertex阶段的输出值传递给fragment阶段时进行插值的做法，将起点和终点的视空间深度进行插值。不过有一点需要注意，这个深度并非直接与屏幕空间的步进值直接成正比，而是与屏幕空间步进值 * 1/z成正比，也就是所谓的透视校正插值。而这个z也就是我们上文得到的w值。这样，得到每次步进前后的深度值，再与深度图中的值进行比较，就得到了是否碰撞的结果。

关于画线算法，坐标变换，透视投影校正插值等内容，可以参考本人之前的SoftRenderer的blog。

基本原理说到这里，上代码。

C#部分如下，增加了从视空间直接转化到屏幕空间的变换矩阵：
```c#
using System. Collections; 
using System. Collections. Generic; 
using UnityEngine; 
[ExecuteInEditMode]
public class ScreenSpaceReflection : MonoBehaviour
{

    Material reflectionMaterial = null;
    Camera currentCamera = null;
    [Range(0, 1000.0f)]
    public float maxRayMarchingDistance = 500.0f;
    [Range(0, 1024)]
    public int maxRayMarchingStep = 64;
    [Range(0, 32)]
    public int maxRayMarchingBinarySearchCount = 8;
    [Range(1, 10)]
    public int rayMarchingStepSize = 2;
    [Range(0, 2.0f)]
    public float depthThickness = 0.01f;

    private void Awake()
    {
        var shader = Shader.Find("Reflection/ScreenSpaceReflection");
        reflectionMaterial = new Material(shader);
        currentCamera = GetComponent<Camera>();
    }

    private void OnEnable()
    {
        currentCamera.depthTextureMode |= DepthTextureMode.DepthNormals;    
    }

    private void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.DepthNormals;
    }

    private void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (reflectionMaterial == null)
        {
            Graphics.Blit(source, destination);
            return;
        }

        var width = source.width;
        var height = source.height;
        var screenSize = new Vector4(1.0f / width, 1.0f / height, width, height);
        var clipToScreenMatrix = new Matrix4x4();
        // (clip * 0.5 + 0.5)变换到screenspace，*width或height，得到真正的像素位置
        clipToScreenMatrix.SetRow(0, new Vector4(width * 0.5f, 0, 0, width * 0.5f));
        clipToScreenMatrix.SetRow(1, new Vector4(0, height * 0.5f, 0, height * 0.5f));
        clipToScreenMatrix.SetRow(2, new Vector4(0, 0, 1.0f, 0));
        clipToScreenMatrix.SetRow(3, new Vector4(0, 0, 0, 1.0f));
        var projectionMatrix = GL.GetGPUProjectionMatrix(currentCamera.projectionMatrix, false);
        var viewToScreenMatrix = clipToScreenMatrix * projectionMatrix;
        reflectionMaterial.SetMatrix("_ViewToScreenMatrix", viewToScreenMatrix); 
        reflectionMaterial.SetVector("_ScreenSize", screenSize);

        reflectionMaterial.SetMatrix("_InverseProjectionMatrix", currentCamera.projectionMatrix.inverse);
        reflectionMaterial.SetMatrix("_CameraProjectionMatrix", currentCamera.projectionMatrix);

        reflectionMaterial.SetFloat("_maxRayMarchingBinarySearchCount", maxRayMarchingBinarySearchCount);
        reflectionMaterial.SetFloat("_maxRayMarchingDistance", maxRayMarchingDistance);
        reflectionMaterial.SetFloat("_maxRayMarchingStep", maxRayMarchingStep);
        reflectionMaterial.SetFloat("_rayMarchingStepSize", rayMarchingStepSize);
        reflectionMaterial.SetFloat("_depthThickness", depthThickness);
        Graphics.Blit(source, destination, reflectionMaterial, 0);
    }

}

``` 
Shader代码如下，改用屏幕空间步进的方式：
```cpp
//Screen Space Reflection效果，屏幕空间步进方式
Shader "Reflection/ScreenSpaceReflection"
{
	Properties
	{
		_MainTex ("Texture", 2D) = "white" {}
	}
	
	SubShader
	{
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }

			
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			
			#include "UnityCG.cginc"
			
			struct appdata
			{
				float4 vertex : POSITION;
				float2 uv : TEXCOORD0;
			};

			struct v2f
			{
				float4 vertex : SV_POSITION;
				float2 uv : TEXCOORD0;	
				float3 viewRay : TEXCOORD1;
			};

			sampler2D _MainTex;
			float4 _MainTex_ST;
			float4 _ScreenSize;//(1 / width, 1 / height, width, height)
			sampler2D _CameraDepthTexture;
			//使用外部传入的matrix，实际上等同于下面两个unity内置matrix
			//似乎在老版本unity中在pss阶段使用正交相机绘制Quad，矩阵会被替换，2017.3版本测试使用内置矩阵也可以
			float4x4 _InverseProjectionMatrix;//unity_CameraInvProjection
			float4x4 _CameraProjectionMatrix;//unity_CameraProjection
			float4x4 _ViewToScreenMatrix;
			
			float _maxRayMarchingDistance;
			float _maxRayMarchingStep;
			float _maxRayMarchingBinarySearchCount;
			float _rayMarchingStepSize;
			float _depthThickness;
			
			sampler2D _CameraDepthNormalsTexture;
			
			void swap(inout float v0, inout float v1)
			{
				float temp = v0;
				v0 = v1;
				v1 = temp;
			}
			
			float distanceSquared(float2 A, float2 B)
			{
				A -= B;
				return dot(A, A);
			}
			
			bool screenSpaceRayMarching(float3 rayOri, float3 rayDir, inout float2 hitScreenPos)
			{
				//反方向反射的，本身也看不见，索性直接干掉
				if (rayDir.z > 0.0)
					return false;
				//首先求得视空间终点位置，不超过最大距离
				float magnitude = _maxRayMarchingDistance;
				float end = rayOri.z + rayDir.z * magnitude;
				//如果光线反过来超过了近裁剪面，需要截取到近裁剪面
				if (end > -_ProjectionParams.y)
					magnitude = (-_ProjectionParams.y - rayOri.z) / rayDir.z;
				float3 rayEnd = rayOri + rayDir * magnitude;
				//直接把cliptoscreen与projection矩阵结合，得到齐次坐标系下屏幕位置
				float4 homoRayOri = mul(_ViewToScreenMatrix, float4(rayOri, 1.0));
				float4 homoRayEnd = mul(_ViewToScreenMatrix, float4(rayEnd, 1.0));
				//w
				float kOri = 1.0 / homoRayOri.w;
				float kEnd = 1.0 / homoRayEnd.w;
				//屏幕空间位置
				float2 screenRayOri = homoRayOri.xy * kOri;
				float2 screenRayEnd = homoRayEnd.xy * kEnd;
				screenRayEnd = (distanceSquared(screenRayEnd, screenRayOri) < 0.0001) ? screenRayOri + float2(0.01, 0.01) : screenRayEnd;
				
				float3 QOri = rayOri * kOri;
				float3 QEnd = rayEnd * kEnd;
				
				float2 displacement = screenRayEnd - screenRayOri;
				bool permute = false;
				if (abs(displacement.x) < abs(displacement.y))
				{
					permute = true;
					
					displacement = displacement.yx;
					screenRayOri.xy = screenRayOri.yx;
					screenRayEnd.xy = screenRayEnd.yx;
				}
				float dir = sign(displacement.x);
				float invdx = dir / displacement.x;
				float2 dp = float2(dir, invdx * displacement.y);
				float3 dq = (QEnd - QOri) * invdx;
				float  dk = (kEnd - kOri) * invdx;
				float rayZmin = rayOri.z;
				float rayZmax = rayOri.z;
				float preZ = rayOri.z;
				
				float2 screenPoint = screenRayOri;
				float3 Q = QOri;
				float k = kOri;
				
				UNITY_LOOP
				for(int i = 0; i < _maxRayMarchingStep; i++)
				{
					//向前步进一个单位
					screenPoint += dp;
					Q.z += dq.z;
					k += dk;
					
					//得到步进前后两点的深度
					rayZmin = preZ;
					rayZmax = (dq.z * 0.5 + Q.z) / (dk * 0.5 + k);
					preZ = rayZmax;
					if (rayZmin > rayZmax)
					{
						swap(rayZmin, rayZmax);
					}
					
					//得到当前屏幕空间位置，交换过的xy换回来，并且根据像素宽度还原回（0,1）区间而不是屏幕区间
					hitScreenPos = permute ? screenPoint.yx : screenPoint;
					hitScreenPos *= _ScreenSize.xy;
					
					//转换回屏幕（0,1）区间，剔除出屏幕的反射
					if (any(hitScreenPos.xy < 0.0) || any(hitScreenPos.xy > 1.0))
						return false;
					
					//采样当前点深度图，转化为视空间的深度（负值）
					float4 depthnormalTex = tex2D(_CameraDepthNormalsTexture, hitScreenPos);
					float depth = -DecodeFloatRG(depthnormalTex.zw) * _ProjectionParams.z;
					
					bool isBehand = (rayZmin <= depth);
					bool intersecting = isBehand && (rayZmax >= depth - _depthThickness);
					
					if (intersecting)
						return true;
				}
				return false;
			}
			
			v2f vert (appdata v)
			{
				v2f o;
				o.vertex = UnityObjectToClipPos(v.vertex);
				o.uv = TRANSFORM_TEX(v.uv, _MainTex);
				
				float4 clipPos = float4(v.uv * 2 - 1.0, 1.0, 1.0);
				float4 viewRay = mul(_InverseProjectionMatrix, clipPos);
				o.viewRay = viewRay.xyz / viewRay.w;
				return o;
			}
			
			fixed4 frag (v2f i) : SV_Target
			{
				fixed4 mainTex = tex2D(_MainTex, i.uv);
				float linear01Depth;
				float3 viewNormal;
				
				float4 cdn = tex2D(_CameraDepthNormalsTexture, i.uv);
				DecodeDepthNormal(cdn, linear01Depth, viewNormal);
				//重建视空间坐标
				float3 viewPos = linear01Depth * i.viewRay;
				float3 viewDir = normalize(viewPos);
				viewNormal = normalize(viewNormal);
				//视空间方向反射方向
				float3 reflectDir = reflect(viewDir, viewNormal);
				float2 hitScreenPos = float2(-1,-1);
				//从该点开始RayMarching
				if (screenSpaceRayMarching(viewPos, reflectDir, hitScreenPos))
				{
					float4 reflectTex = tex2D(_MainTex, hitScreenPos);
					mainTex.rgb += reflectTex.rgb;
				}
				return mainTex;
			}
			
			ENDCG
		}
	}
}
```

没有Binary，最大步进次数450效果如下：

在屏幕空间步进的效果几乎没有重影，效果也能保证，然而最大步进次数着实让人有点害怕。我们可以尝试添加一些步长，比如我们在步进距离上乘以一个系数（1, 10）正数，使之每次步进超过一个像素，加快碰撞过程：

``` cpp
float2 dp = float2(dir, invdx * displacement.y) * _rayMarchingStepSize;
float3 dq = (QEnd - QOri) * invdx * _rayMarchingStepSize;
float  dk = (kEnd - kOri) * invdx * _rayMarchingStepSize;
```

这样，我们步进100次，步长4的情况下效果也与之前差不多：

但是如果再提高步长，效果就差了很多，尤其是在近处会出现断带，步进20次，步长10效果：

远处效果勉强可以忍，近处基本忍不了了。下一步我们还是考虑可变步长，Binary Search可以，也可以考虑在近处减少步长，在远处加大步长，不过我们此处选择一个更加好玩的方式，并且可以进一步减少步进次数。

### Dither RayMarching

这是一个老套路了，在GodRay的blog里面就使用过这个套路，对于RayMarching类型的效果来说，Dither确实是一个神级优化了。简单来说，Dither（Jitter）就是通过增加随机噪声来增加一些随机性，进而大幅度减少真正步进的次数。依然使用的是KillZone分享的那个DitherMap，然后在RayMarching时，第一步增加一个随机的值。

DitherMap生成代码：
```c#

    private Texture2D GenerateDitherMap()
    {
        int texSize = 4;
        var ditherMap = new Texture2D(texSize, texSize, TextureFormat.Alpha8, false, true);
        ditherMap.filterMode = FilterMode.Point;
        Color32[] colors = new Color32[texSize * texSize];

        colors[0] = GetDitherColor(0.0f);
        colors[1] = GetDitherColor(8.0f);
        colors[2] = GetDitherColor(2.0f);
        colors[3] = GetDitherColor(10.0f);

        colors[4] = GetDitherColor(12.0f);
        colors[5] = GetDitherColor(4.0f);
        colors[6] = GetDitherColor(14.0f);
        colors[7] = GetDitherColor(6.0f);

        colors[8] = GetDitherColor(3.0f);
        colors[9] = GetDitherColor(11.0f);
        colors[10] = GetDitherColor(1.0f);
        colors[11] = GetDitherColor(9.0f);

        colors[12] = GetDitherColor(15.0f);
        colors[13] = GetDitherColor(7.0f);
        colors[14] = GetDitherColor(13.0f);
        colors[15] = GetDitherColor(5.0f);

        ditherMap.SetPixels32(colors);
        ditherMap.Apply();
        return ditherMap;
    }

    private Color32 GetDitherColor(float value)
    {
        byte byteValue = (byte)(value / 16.0f * 255);
        return new Color32(byteValue, byteValue, byteValue, byteValue);
    }

    

``` 
Shader中RayMarching第一步时增加Dither值：
```cpp
//dither
float2 offsetUV = (fmod(floor(screenRayOri), 4.0));
float ditherValue = tex2D(_ditherMap, offsetUV / 4.0).a;
screenPoint += dp * ditherValue;
Q.z += dq.z * ditherValue;
k += dk * ditherValue;
```

依然是步进20次，步长为10的情况下，效果会好很多：

但是使用Dither之后，会出现小格子。不过没有关系，Dither可以和模糊配套使用，使用模糊去噪。简单来说就是Jitter工作流（只不过本人的这套比较简陋喽）。既然需要模糊了，我们就不能直接将结果输出了，我们可以申请两块中间RT，用于将结果渲染到RT上，并且使用高斯模糊，然后再将结果与原始RT叠加得到最终结果。此外，如果不直接输出，我们在申请RT的时候就可以申请一张二分之一或者更小的RT，来降低RayMarching的消耗，这也是各家SSR经常使用的一个优化（如KlayGE），毕竟反射效果无需特别清晰，而且这个效果的逐像素计算消耗实在太大。

在C#中增加中间RT，模糊操作，叠加操作：
```c#
/********************************************************************
 FileName: ScreenSpaceReflection.cs
 Description: 屏幕空间反射SSR，DitherRayMarching，延迟渲染下
*********************************************************************/
using System. Collections; 
using System. Collections. Generic; 
using UnityEngine; 
[ExecuteInEditMode]
public class ScreenSpaceReflection : MonoBehaviour
{

    Material reflectionMaterial = null;
    Camera currentCamera = null;
    [Range(0, 1000.0f)]
    public float maxRayMarchingDistance = 500.0f;
    [Range(0, 1024)]
    public int maxRayMarchingStep = 64;
    [Range(0, 32)]
    public int maxRayMarchingBinarySearchCount = 8;
    [Range(1, 10)]
    public int rayMarchingStepSize = 2;
    [Range(0, 2.0f)]
    public float depthThickness = 0.01f;

    [Range(0, 3)]
    public int downSample = 1;

    [Range(0,3)]
    public int samplerScale = 1;

    private Texture2D ditherMap = null;

    private void Awake()
    {
        var shader = Shader.Find("Reflection/ScreenSpaceReflection");
        reflectionMaterial = new Material(shader);
        currentCamera = GetComponent<Camera>();
    }

    private void OnEnable()
    {
        if (ditherMap == null)
            ditherMap = GenerateDitherMap();
        currentCamera.depthTextureMode |= DepthTextureMode.DepthNormals;
    }

    private void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.DepthNormals;
    }

    private void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (reflectionMaterial == null)
        {
            Graphics.Blit(source, destination);
            return;
        }

        var width = source.width >> downSample;
        var height = source.height >> downSample;
        var screenSize = new Vector4(1.0f / width, 1.0f / height, width, height);
        var clipToScreenMatrix = new Matrix4x4();
        // (clip * 0.5 + 0.5)变换到screenspace，*width或height，得到真正的像素位置
        clipToScreenMatrix.SetRow(0, new Vector4(width * 0.5f, 0, 0, width * 0.5f));
        clipToScreenMatrix.SetRow(1, new Vector4(0, height * 0.5f, 0, height * 0.5f));
        clipToScreenMatrix.SetRow(2, new Vector4(0, 0, 1.0f, 0));
        clipToScreenMatrix.SetRow(3, new Vector4(0, 0, 0, 1.0f));
        var projectionMatrix = GL.GetGPUProjectionMatrix(currentCamera.projectionMatrix, false);
        var viewToScreenMatrix = clipToScreenMatrix * projectionMatrix;
        reflectionMaterial.SetMatrix("_ViewToScreenMatrix", viewToScreenMatrix); 
        reflectionMaterial.SetVector("_ScreenSize", screenSize);

        reflectionMaterial.SetMatrix("_InverseProjectionMatrix", currentCamera.projectionMatrix.inverse);
        reflectionMaterial.SetMatrix("_CameraProjectionMatrix", currentCamera.projectionMatrix);

        reflectionMaterial. SetMatrix("_WorldToCameraMatrix", currentCamera.worldToCameraMatrix); 

        reflectionMaterial.SetFloat("_maxRayMarchingBinarySearchCount", maxRayMarchingBinarySearchCount);
        reflectionMaterial.SetFloat("_maxRayMarchingDistance", maxRayMarchingDistance);
        reflectionMaterial.SetFloat("_maxRayMarchingStep", maxRayMarchingStep);
        reflectionMaterial.SetFloat("_rayMarchingStepSize", rayMarchingStepSize);
        reflectionMaterial.SetFloat("_depthThickness", depthThickness);
        reflectionMaterial.SetTexture("_ditherMap", ditherMap);

        var reflectRT = RenderTexture.GetTemporary(width, height, 0, source.format);
        var tempBlurRT = RenderTexture.GetTemporary(width, height, 0, source.format);
        Graphics.Blit(source, reflectRT, reflectionMaterial, 0);

        //高斯模糊，两次模糊，横向纵向，使用pass1进行高斯模糊
        reflectionMaterial.SetVector("_offsets", new Vector4(0, samplerScale, 0, 0));
        Graphics.Blit(reflectRT, tempBlurRT, reflectionMaterial, 1);
        reflectionMaterial.SetVector("_offsets", new Vector4(samplerScale, 0, 0, 0));
        Graphics.Blit(tempBlurRT, reflectRT, reflectionMaterial, 1);

        //将反射贴图叠加到原图
        reflectionMaterial.SetTexture("_ReflectTex", reflectRT);
        Graphics.Blit(source, destination, reflectionMaterial, 2);

        RenderTexture.ReleaseTemporary(reflectRT);
        RenderTexture.ReleaseTemporary(tempBlurRT);
    }

    private Texture2D GenerateDitherMap()
    {
        int texSize = 4;
        var ditherMap = new Texture2D(texSize, texSize, TextureFormat.Alpha8, false, true);
        ditherMap.filterMode = FilterMode.Point;
        Color32[] colors = new Color32[texSize * texSize];

        colors[0] = GetDitherColor(0.0f);
        colors[1] = GetDitherColor(8.0f);
        colors[2] = GetDitherColor(2.0f);
        colors[3] = GetDitherColor(10.0f);

        colors[4] = GetDitherColor(12.0f);
        colors[5] = GetDitherColor(4.0f);
        colors[6] = GetDitherColor(14.0f);
        colors[7] = GetDitherColor(6.0f);

        colors[8] = GetDitherColor(3.0f);
        colors[9] = GetDitherColor(11.0f);
        colors[10] = GetDitherColor(1.0f);
        colors[11] = GetDitherColor(9.0f);

        colors[12] = GetDitherColor(15.0f);
        colors[13] = GetDitherColor(7.0f);
        colors[14] = GetDitherColor(13.0f);
        colors[15] = GetDitherColor(5.0f);

        ditherMap.SetPixels32(colors);
        ditherMap.Apply();
        return ditherMap;
    }

    private Color32 GetDitherColor(float value)
    {
        byte byteValue = (byte)(value / 16.0f * 255);
        return new Color32(byteValue, byteValue, byteValue, byteValue);
    }

}

``` 
增加了模糊Pass和叠加Pass的shader：
```cpp
//Screen Space Reflection效果，屏幕空间步进方式,DitherRayMarching
Shader "Reflection/ScreenSpaceReflection"
{
	Properties
	{
		_MainTex ("Texture", 2D) = "white" {}
	}
	
	CGINCLUDE
	
	#include "UnityCG.cginc"
	
	struct appdata
	{
		float4 vertex : POSITION;
		float2 uv : TEXCOORD0;
	};
	
	struct v2f
	{
		float4 vertex : SV_POSITION;
		float2 uv : TEXCOORD0;	
		float3 viewRay : TEXCOORD1;
	};
	
	sampler2D _MainTex;
	float4 _MainTex_ST;
	float4 _ScreenSize;//(1 / width, 1 / height, width, height)
	sampler2D _CameraDepthTexture;
	sampler2D _ditherMap;
	//使用外部传入的matrix，实际上等同于下面两个unity内置matrix
	//似乎在老版本unity中在pss阶段使用正交相机绘制Quad，矩阵会被替换，2017.3版本测试使用内置矩阵也可以
	float4x4 _InverseProjectionMatrix;//unity_CameraInvProjection
	float4x4 _CameraProjectionMatrix;//unity_CameraProjection
	float4x4 _ViewToScreenMatrix;
	
	float _maxRayMarchingDistance;
	float _maxRayMarchingStep;
	float _maxRayMarchingBinarySearchCount;
	float _rayMarchingStepSize;
	float _depthThickness;
	
	sampler2D _CameraDepthNormalsTexture;
	
	void swap(inout float v0, inout float v1)
	{
		float temp = v0;
		v0 = v1;
		v1 = temp;
	}
	
	float distanceSquared(float2 A, float2 B)
	{
		A -= B;
		return dot(A, A);
	}
	
	bool screenSpaceRayMarching(float3 rayOri, float3 rayDir, inout float2 hitScreenPos)
	{
		//反方向反射的，本身也看不见，索性直接干掉
		if (rayDir.z > 0.0)
			return false;
		//首先求得视空间终点位置，不超过最大距离
		float magnitude = _maxRayMarchingDistance;
		float end = rayOri.z + rayDir.z * magnitude;
		//如果光线反过来超过了近裁剪面，需要截取到近裁剪面
		if (end > -_ProjectionParams.y)
			magnitude = (-_ProjectionParams.y - rayOri.z) / rayDir.z;
		float3 rayEnd = rayOri + rayDir * magnitude;
		//直接把cliptoscreen与projection矩阵结合，得到齐次坐标系下屏幕位置
		float4 homoRayOri = mul(_ViewToScreenMatrix, float4(rayOri, 1.0));
		float4 homoRayEnd = mul(_ViewToScreenMatrix, float4(rayEnd, 1.0));
		//w
		float kOri = 1.0 / homoRayOri.w;
		float kEnd = 1.0 / homoRayEnd.w;
		//屏幕空间位置
		float2 screenRayOri = homoRayOri.xy * kOri;
		float2 screenRayEnd = homoRayEnd.xy * kEnd;
		screenRayEnd = (distanceSquared(screenRayEnd, screenRayOri) < 0.0001) ? screenRayOri + float2(0.01, 0.01) : screenRayEnd;
		
		float3 QOri = rayOri * kOri;
		float3 QEnd = rayEnd * kEnd;
		
		float2 displacement = screenRayEnd - screenRayOri;
		bool permute = false;
		if (abs(displacement.x) < abs(displacement.y))
		{
			permute = true;
			
			displacement = displacement.yx;
			screenRayOri.xy = screenRayOri.yx;
			screenRayEnd.xy = screenRayEnd.yx;
		}
		float dir = sign(displacement.x);
		float invdx = dir / displacement.x;
		//float stride = 2.0 - min(1.0, -rayOri * 0.01);
		float stride = _rayMarchingStepSize;
		
		float2 dp = float2(dir, invdx * displacement.y) * stride;
		float3 dq = (QEnd - QOri) * invdx * stride;
		float  dk = (kEnd - kOri) * invdx * stride;
		float rayZmin = rayOri.z;
		float rayZmax = rayOri.z;
		float preZ = rayOri.z;
		
		float2 screenPoint = screenRayOri;
		float3 Q = QOri;
		float k = kOri;
		
		float2 offsetUV = (fmod(floor(screenRayOri), 4.0));
		float ditherValue = tex2D(_ditherMap, offsetUV / 4.0).a;
		
		screenPoint += dp * ditherValue;
		Q.z += dq.z * ditherValue;
		k += dk * ditherValue;
		
		UNITY_LOOP
		for(int i = 0; i < _maxRayMarchingStep; i++)
		{
			//向前步进一个单位
			screenPoint += dp;
			Q.z += dq.z;
			k += dk;
			
			//得到步进前后两点的深度
			rayZmin = preZ;
			rayZmax = (dq.z * 0.5 + Q.z) / (dk * 0.5 + k);
			preZ = rayZmax;
			if (rayZmin > rayZmax)
			{
				swap(rayZmin, rayZmax);
			}
			
			//得到当前屏幕空间位置，交换过的xy换回来，并且根据像素宽度还原回（0,1）区间而不是屏幕区间
			hitScreenPos = permute ? screenPoint.yx : screenPoint;
			hitScreenPos *= _ScreenSize.xy;
			
			//转换回屏幕（0,1）区间，剔除出屏幕的反射
			if (any(hitScreenPos.xy < 0.0) || any(hitScreenPos.xy > 1.0))
				return false;
			
			//采样当前点深度图，转化为视空间的深度（负值）
			float4 depthnormalTex = tex2D(_CameraDepthNormalsTexture, hitScreenPos);
			float depth = -DecodeFloatRG(depthnormalTex.zw) * _ProjectionParams.z;
			
			bool isBehand = (rayZmin <= depth);
			bool intersecting = isBehand && (rayZmax >= depth - _depthThickness);
			
			if (intersecting)
				return true;
		}
		return false;
	}
	
	v2f vert_raymarching (appdata v)
	{
		v2f o;
		o.vertex = UnityObjectToClipPos(v.vertex);
		o.uv = TRANSFORM_TEX(v.uv, _MainTex);
		
		float4 clipPos = float4(v.uv * 2 - 1.0, 1.0, 1.0);
		float4 viewRay = mul(_InverseProjectionMatrix, clipPos);
		o.viewRay = viewRay.xyz / viewRay.w;
		return o;
	}
	
	fixed4 frag_raymarching (v2f i) : SV_Target
	{
		float linear01Depth;
		float3 viewNormal;
		
		float4 cdn = tex2D(_CameraDepthNormalsTexture, i.uv);
		DecodeDepthNormal(cdn, linear01Depth, viewNormal);
		//重建视空间坐标
		float3 viewPos = linear01Depth * i.viewRay;
		float3 viewDir = normalize(viewPos);
		viewNormal = normalize(viewNormal);
		//视空间方向反射方向
		float3 reflectDir = reflect(viewDir, viewNormal);
		float2 hitScreenPos = float2(-1,-1);
		fixed4 color = fixed4(0,0,0,1);
		
		//从该点开始RayMarching
		if (screenSpaceRayMarching(viewPos, reflectDir, hitScreenPos))
		{
			float4 reflectTex = tex2D(_MainTex, hitScreenPos);
			color.rgb += reflectTex.rgb;
		}
		return color;
	}
	
	
	//用于blur
	struct v2f_blur
	{
		float4 pos : SV_POSITION;
		float2 uv  : TEXCOORD0;
		float4 uv01 : TEXCOORD1;
		float4 uv23 : TEXCOORD2;
		float4 uv45 : TEXCOORD3;
	};
 
	//用于叠加
	struct v2f_add
	{
		float4 pos : SV_POSITION;
		float2 uv  : TEXCOORD0;
		float2 uv1 : TEXCOORD1;
	};
 
	float4 _MainTex_TexelSize;
	float4 _offsets;
 
	//高斯模糊 vert shader
	v2f_blur vert_blur(appdata_img v)
	{
		v2f_blur o;
		_offsets *= _MainTex_TexelSize.xyxy;
		o.pos = UnityObjectToClipPos(v.vertex);
		o.uv = v.texcoord.xy;
 
		o.uv01 = v.texcoord.xyxy + _offsets.xyxy * float4(1, 1, -1, -1);
		o.uv23 = v.texcoord.xyxy + _offsets.xyxy * float4(1, 1, -1, -1) * 2.0;
		o.uv45 = v.texcoord.xyxy + _offsets.xyxy * float4(1, 1, -1, -1) * 3.0;
 
		return o;
	}
 
	//高斯模糊 pixel shader
	fixed4 frag_blur(v2f_blur i) : SV_Target
	{
		fixed4 color = fixed4(0,0,0,0);
		color += 0.40 * tex2D(_MainTex, i.uv);
		color += 0.15 * tex2D(_MainTex, i.uv01.xy);
		color += 0.15 * tex2D(_MainTex, i.uv01.zw);
		color += 0.10 * tex2D(_MainTex, i.uv23.xy);
		color += 0.10 * tex2D(_MainTex, i.uv23.zw);
		color += 0.05 * tex2D(_MainTex, i.uv45.xy);
		color += 0.05 * tex2D(_MainTex, i.uv45.zw);
		return color;
	}
	
	sampler2D _ReflectTex;
 
	fixed4 frag_add(v2f_add i) : SV_Target
	{
		fixed4 ori = tex2D(_MainTex, i.uv);
		fixed4 reflect = tex2D(_ReflectTex, i.uv);
		return ori + reflect;
	}

	
	ENDCG
	
	SubShader
	{
		//Pass0 : RayMarching
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
			CGPROGRAM
			#pragma vertex vert_raymarching
			#pragma fragment frag_raymarching
			ENDCG
			
		}
		
		//Pass1 : Blur
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
			#pragma vertex vert_blur
			#pragma fragment frag_blur
			ENDCG
		}
 
		//pass 1: Add
		Pass
		{
 
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
			#pragma vertex vert_img
			#pragma fragment frag_add
			ENDCG
		}

	}
}
```

20次步进+10步长+Dither + Gaussian Blur效果，反射不是很清晰，但是已经看不到噪点了：

### 延迟渲染下的SSR

前面的各种SSR都是在前向渲染的情况下实现的。虽然也可以凑合使用，然而也就到这里了。有一个很要命的问题，SSR是一个全屏的后处理效果，我们没有办法区分哪里需要反射，哪里不需要反射。当然可以再去渲染一个Mask或者放到Alpha通道里面，但是毕竟不是一个很正统的做法。所以，还是在延迟下使用SSR会更加合适一些。

要改回延迟渲染，其实修改并不是很大。延迟渲染下默认就有CameraDepthTexture，不过这个深度是非线性的，我们需要将其改为线性，才能正常使用；另外的一个必要条件是法线，在延迟渲染下可以通过_CameraGBufferTexture2中得到法线，不过这个法线是世界空间的，我们需要通过世界到相机空间的逆矩阵将法线变换回法线空间。最后融合时，我们可以通过_CameraGBufferTexture1.a通道，即roughness值与界面叠加融合。关于Unity延迟渲染的GBuffer，可以参考官方文档。

延迟渲染下SSR的C#代码：
```c#
/********************************************************************
 FileName: ScreenSpaceReflection.cs
 Description: 屏幕空间反射SSR，DitherRayMarching，延迟渲染下
*********************************************************************/
using UnityEngine; 
[ExecuteInEditMode]
public class ScreenSpaceReflection : MonoBehaviour
{

    Material reflectionMaterial = null;
    Camera currentCamera = null;
    [Range(0, 1000.0f)]
    public float maxRayMarchingDistance = 500.0f;
    [Range(0, 1024)]
    public int maxRayMarchingStep = 64;
    [Range(0, 32)]
    public int maxRayMarchingBinarySearchCount = 8;
    [Range(1, 50)]
    public int rayMarchingStepSize = 2;
    [Range(0, 2.0f)]
    public float depthThickness = 0.01f;

    [Range(0, 3)]
    public int downSample = 1;

    [Range(0,3)]
    public int samplerScale = 1;

    private Texture2D ditherMap = null;

    private void Awake()
    {
        var shader = Shader.Find("Reflection/ScreenSpaceReflection");
        reflectionMaterial = new Material(shader);
        currentCamera = GetComponent<Camera>();
        if (ditherMap == null)
            ditherMap = GenerateDitherMap();
    }

    private void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (reflectionMaterial == null)
        {
            Graphics.Blit(source, destination);
            return;
        }

        var width = source.width >> downSample;
        var height = source.height >> downSample;
        var screenSize = new Vector4(1.0f / width, 1.0f / height, width, height);
        var clipToScreenMatrix = new Matrix4x4();
        // (clip * 0.5 + 0.5)变换到screenspace，*width或height，得到真正的像素位置
        clipToScreenMatrix.SetRow(0, new Vector4(width * 0.5f, 0, 0, width * 0.5f));
        clipToScreenMatrix.SetRow(1, new Vector4(0, height * 0.5f, 0, height * 0.5f));
        clipToScreenMatrix.SetRow(2, new Vector4(0, 0, 1.0f, 0));
        clipToScreenMatrix.SetRow(3, new Vector4(0, 0, 0, 1.0f));
        var projectionMatrix = GL.GetGPUProjectionMatrix(currentCamera.projectionMatrix, false);
        var viewToScreenMatrix = clipToScreenMatrix * projectionMatrix;
        reflectionMaterial.SetMatrix("_ViewToScreenMatrix", viewToScreenMatrix); 
        reflectionMaterial.SetVector("_ScreenSize", screenSize);

        reflectionMaterial.SetMatrix("_InverseProjectionMatrix", currentCamera.projectionMatrix.inverse);
        reflectionMaterial.SetMatrix("_CameraProjectionMatrix", currentCamera.projectionMatrix);

        reflectionMaterial. SetMatrix("_WorldToCameraMatrix", currentCamera.worldToCameraMatrix); 

        reflectionMaterial.SetFloat("_maxRayMarchingBinarySearchCount", maxRayMarchingBinarySearchCount);
        reflectionMaterial.SetFloat("_maxRayMarchingDistance", maxRayMarchingDistance);
        reflectionMaterial.SetFloat("_maxRayMarchingStep", maxRayMarchingStep);
        reflectionMaterial.SetFloat("_rayMarchingStepSize", rayMarchingStepSize);
        reflectionMaterial.SetFloat("_depthThickness", depthThickness);
        reflectionMaterial.SetTexture("_ditherMap", ditherMap);

        var reflectRT = RenderTexture.GetTemporary(width, height, 0, source.format);
        var tempBlurRT = RenderTexture.GetTemporary(width, height, 0, source.format);
        Graphics.Blit(source, reflectRT, reflectionMaterial, 0);

        //高斯模糊，两次模糊，横向纵向，使用pass1进行高斯模糊
        reflectionMaterial.SetVector("_offsets", new Vector4(0, samplerScale, 0, 0));
        Graphics.Blit(reflectRT, tempBlurRT, reflectionMaterial, 1);
        reflectionMaterial.SetVector("_offsets", new Vector4(samplerScale, 0, 0, 0));
        Graphics.Blit(tempBlurRT, reflectRT, reflectionMaterial, 1);

        //将反射贴图叠加到原图
        reflectionMaterial.SetTexture("_ReflectTex", reflectRT);
        Graphics.Blit(source, destination, reflectionMaterial, 2);

        RenderTexture.ReleaseTemporary(reflectRT);
        RenderTexture.ReleaseTemporary(tempBlurRT);
    }

    private Texture2D GenerateDitherMap()
    {
        int texSize = 4;
        var ditherMap = new Texture2D(texSize, texSize, TextureFormat.Alpha8, false, true);
        ditherMap.filterMode = FilterMode.Point;
        Color32[] colors = new Color32[texSize * texSize];

        colors[0] = GetDitherColor(0.0f);
        colors[1] = GetDitherColor(8.0f);
        colors[2] = GetDitherColor(2.0f);
        colors[3] = GetDitherColor(10.0f);

        colors[4] = GetDitherColor(12.0f);
        colors[5] = GetDitherColor(4.0f);
        colors[6] = GetDitherColor(14.0f);
        colors[7] = GetDitherColor(6.0f);

        colors[8] = GetDitherColor(3.0f);
        colors[9] = GetDitherColor(11.0f);
        colors[10] = GetDitherColor(1.0f);
        colors[11] = GetDitherColor(9.0f);

        colors[12] = GetDitherColor(15.0f);
        colors[13] = GetDitherColor(7.0f);
        colors[14] = GetDitherColor(13.0f);
        colors[15] = GetDitherColor(5.0f);

        ditherMap.SetPixels32(colors);
        ditherMap.Apply();
        return ditherMap;
    }

    private Color32 GetDitherColor(float value)
    {
        byte byteValue = (byte)(value / 16.0f * 255);
        return new Color32(byteValue, byteValue, byteValue, byteValue);
    }

}

``` 
延迟渲染下SSR的Shader代码：
```cpp
//Screen Space Reflection效果，屏幕空间步进方式,DitherRayMarching
Shader "Reflection/ScreenSpaceReflection"
{
	Properties
	{
		_MainTex ("Texture", 2D) = "white" {}
	}
	
	CGINCLUDE
	
	#include "UnityCG.cginc"
	
	struct appdata
	{
		float4 vertex : POSITION;
		float2 uv : TEXCOORD0;
	};
	
	struct v2f
	{
		float4 vertex : SV_POSITION;
		float2 uv : TEXCOORD0;	
		float3 viewRay : TEXCOORD1;
	};
	
	sampler2D _MainTex;
	float4 _MainTex_ST;
	float4 _ScreenSize;//(1 / width, 1 / height, width, height)
	sampler2D _CameraDepthTexture;
	sampler2D _CameraGBufferTexture0;
    sampler2D _CameraGBufferTexture1;
    sampler2D _CameraGBufferTexture2;
    sampler2D _CameraGBufferTexture3;
	sampler2D _ditherMap;
	//使用外部传入的matrix，实际上等同于下面两个unity内置matrix
	//似乎在老版本unity中在pss阶段使用正交相机绘制Quad，矩阵会被替换，2017.3版本测试使用内置矩阵也可以
	float4x4 _InverseProjectionMatrix;//unity_CameraInvProjection
	float4x4 _CameraProjectionMatrix;//unity_CameraProjection
	float4x4 _WorldToCameraMatrix;
	float4x4 _ViewToScreenMatrix;
	
	float _maxRayMarchingDistance;
	float _maxRayMarchingStep;
	float _maxRayMarchingBinarySearchCount;
	float _rayMarchingStepSize;
	float _depthThickness;
	
	sampler2D _CameraDepthNormalsTexture;
	
	void swap(inout float v0, inout float v1)
	{
		float temp = v0;
		v0 = v1;
		v1 = temp;
	}
	
	float distanceSquared(float2 A, float2 B)
	{
		A -= B;
		return dot(A, A);
	}
	
	bool screenSpaceRayMarching(float3 rayOri, float3 rayDir, inout float2 hitScreenPos)
	{
		//反方向反射的，本身也看不见，索性直接干掉
		if (rayDir.z > 0.0)
			return false;
		//首先求得视空间终点位置，不超过最大距离
		float magnitude = _maxRayMarchingDistance;
		float end = rayOri.z + rayDir.z * magnitude;
		//如果光线反过来超过了近裁剪面，需要截取到近裁剪面
		if (end > -_ProjectionParams.y)
			magnitude = (-_ProjectionParams.y - rayOri.z) / rayDir.z;
		float3 rayEnd = rayOri + rayDir * magnitude;
		//直接把cliptoscreen与projection矩阵结合，得到齐次坐标系下屏幕位置
		float4 homoRayOri = mul(_ViewToScreenMatrix, float4(rayOri, 1.0));
		float4 homoRayEnd = mul(_ViewToScreenMatrix, float4(rayEnd, 1.0));
		//w
		float kOri = 1.0 / homoRayOri.w;
		float kEnd = 1.0 / homoRayEnd.w;
		//屏幕空间位置
		float2 screenRayOri = homoRayOri.xy * kOri;
		float2 screenRayEnd = homoRayEnd.xy * kEnd;
		screenRayEnd = (distanceSquared(screenRayEnd, screenRayOri) < 0.0001) ? screenRayOri + float2(0.01, 0.01) : screenRayEnd;
		
		float3 QOri = rayOri * kOri;
		float3 QEnd = rayEnd * kEnd;
		
		float2 displacement = screenRayEnd - screenRayOri;
		bool permute = false;
		if (abs(displacement.x) < abs(displacement.y))
		{
			permute = true;
			
			displacement = displacement.yx;
			screenRayOri.xy = screenRayOri.yx;
			screenRayEnd.xy = screenRayEnd.yx;
		}
		float dir = sign(displacement.x);
		float invdx = dir / displacement.x;
		//float stride = 2.0 - min(1.0, -rayOri * 0.01);
		float stride = _rayMarchingStepSize;
		
		float2 dp = float2(dir, invdx * displacement.y) * stride;
		float3 dq = (QEnd - QOri) * invdx * stride;
		float  dk = (kEnd - kOri) * invdx * stride;
		float rayZmin = rayOri.z;
		float rayZmax = rayOri.z;
		float preZ = rayOri.z;
		
		float2 screenPoint = screenRayOri;
		float3 Q = QOri;
		float k = kOri;
		
		float2 offsetUV = (fmod(floor(screenRayOri), 4.0));
		float ditherValue = tex2D(_ditherMap, offsetUV / 4.0).a;
		
		screenPoint += dp * ditherValue;
		Q.z += dq.z * ditherValue;
		k += dk * ditherValue;
		
		UNITY_LOOP
		for(int i = 0; i < _maxRayMarchingStep; i++)
		{
			//向前步进一个单位
			screenPoint += dp;
			Q.z += dq.z;
			k += dk;
			
			//得到步进前后两点的深度
			rayZmin = preZ;
			rayZmax = (dq.z * 0.5 + Q.z) / (dk * 0.5 + k);
			preZ = rayZmax;
			if (rayZmin > rayZmax)
			{
				swap(rayZmin, rayZmax);
			}
			
			//得到当前屏幕空间位置，交换过的xy换回来，并且根据像素宽度还原回（0,1）区间而不是屏幕区间
			hitScreenPos = permute ? screenPoint.yx : screenPoint;
			hitScreenPos *= _ScreenSize.xy;
			
			//转换回屏幕（0,1）区间，剔除出屏幕的反射
			if (any(hitScreenPos.xy < 0.0) || any(hitScreenPos.xy > 1.0))
				return false;
			
			//采样当前点深度图，转化为视空间的深度（负值）
			float depth = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, hitScreenPos);
			depth = -LinearEyeDepth(depth);
			
			bool isBehand = (rayZmin <= depth);
			bool intersecting = isBehand && (rayZmax >= depth - _depthThickness);
			
			if (intersecting)
				return true;
		}
		return false;
	}
	
	v2f vert_raymarching (appdata v)
	{
		v2f o;
		o.vertex = UnityObjectToClipPos(v.vertex);
		o.uv = TRANSFORM_TEX(v.uv, _MainTex);
		
		float4 clipPos = float4(v.uv * 2 - 1.0, 1.0, 1.0);
		float4 viewRay = mul(_InverseProjectionMatrix, clipPos);
		o.viewRay = viewRay.xyz / viewRay.w;
		return o;
	}
	
	fixed4 frag_raymarching (v2f i) : SV_Target
	{
		float4 z = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);
		float linear01Depth = Linear01Depth(z);
		
		float3 worldNormal = tex2D(_CameraGBufferTexture2, i.uv).rgb * 2.0 - 1.0;
		float3 viewNormal = mul((float3x3)(_WorldToCameraMatrix), worldNormal);
		
		//重建视空间坐标
		float3 viewPos = linear01Depth * i.viewRay;
		float3 viewDir = normalize(viewPos);
		viewNormal = normalize(viewNormal);
		//视空间方向反射方向
		float3 reflectDir = reflect(viewDir, viewNormal);
		float2 hitScreenPos = float2(-1,-1);
		fixed4 color = fixed4(0,0,0,1);
		
		//从该点开始RayMarching
		if (screenSpaceRayMarching(viewPos, reflectDir, hitScreenPos))
		{
			float4 reflectTex = tex2D(_MainTex, hitScreenPos);
			color.rgb += reflectTex.rgb;
		}
		return color;
	}
	
	
	//用于blur
	struct v2f_blur
	{
		float4 pos : SV_POSITION;
		float2 uv  : TEXCOORD0;
		float4 uv01 : TEXCOORD1;
		float4 uv23 : TEXCOORD2;
		float4 uv45 : TEXCOORD3;
	};
 
	//用于叠加
	struct v2f_add
	{
		float4 pos : SV_POSITION;
		float2 uv  : TEXCOORD0;
		float2 uv1 : TEXCOORD1;
	};
 
	float4 _MainTex_TexelSize;
	float4 _offsets;
 
	//高斯模糊 vert shader
	v2f_blur vert_blur(appdata_img v)
	{
		v2f_blur o;
		_offsets *= _MainTex_TexelSize.xyxy;
		o.pos = UnityObjectToClipPos(v.vertex);
		o.uv = v.texcoord.xy;
 
		o.uv01 = v.texcoord.xyxy + _offsets.xyxy * float4(1, 1, -1, -1);
		o.uv23 = v.texcoord.xyxy + _offsets.xyxy * float4(1, 1, -1, -1) * 2.0;
		o.uv45 = v.texcoord.xyxy + _offsets.xyxy * float4(1, 1, -1, -1) * 3.0;
 
		return o;
	}
 
	//高斯模糊 pixel shader
	fixed4 frag_blur(v2f_blur i) : SV_Target
	{
		fixed4 color = fixed4(0,0,0,0);
		color += 0.40 * tex2D(_MainTex, i.uv);
		color += 0.15 * tex2D(_MainTex, i.uv01.xy);
		color += 0.15 * tex2D(_MainTex, i.uv01.zw);
		color += 0.10 * tex2D(_MainTex, i.uv23.xy);
		color += 0.10 * tex2D(_MainTex, i.uv23.zw);
		color += 0.05 * tex2D(_MainTex, i.uv45.xy);
		color += 0.05 * tex2D(_MainTex, i.uv45.zw);
		return color;
	}
	
	sampler2D _ReflectTex;
 
	fixed4 frag_add(v2f_add i) : SV_Target
	{
		fixed4 ori = tex2D(_MainTex, i.uv);
		fixed4 reflect = tex2D(_ReflectTex, i.uv);
		float s = tex2D(_CameraGBufferTexture1, i.uv).a;
		return ori + reflect * s;
	}

	
	ENDCG
	
	SubShader
	{
		//Pass0 : RayMarching
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
			CGPROGRAM
			#pragma vertex vert_raymarching
			#pragma fragment frag_raymarching
			ENDCG
			
		}
		
		//Pass1 : Blur
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
			#pragma vertex vert_blur
			#pragma fragment frag_blur
			ENDCG
		}
 
		//pass 1: Add
		Pass
		{
 
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
			#pragma vertex vert_img
			#pragma fragment frag_add
			ENDCG
		}

	}
}
```

最终效果来张动图：

目前仅仅实现了最基本的SSR，真正的SSR的进阶版有很多很多，比如保存采样点而非采样贴图，进行根据反射距离淡出的效果；增加模糊反射模拟真正PBR的效果，将不同级别的模糊图存在Mip中模拟粗糙度的问题；支持法线贴图；Hi-Z优化等等。如果有精力以后再加啦。

PS：如果真想用SSR的话，Unity官方提供的后处理包PostProcessingStack中已经带啦，而且也有知乎上的大佬实现了更好的SSR效果以及目前最新的SSR方案。膜拜一波这些大佬们。

## 总结

本篇blog主要是梳理了一下目前实时渲染中常见的一些反射技术，包括CubeMap（Reflection Probe），Box Projected Cube Map，Planar Reflection，Screen Space Reflection。每种技术都有其优缺点。很多情况下，这些反射技术都会被结合使用，比如上面图中，动态的人物是使用SSR渲染的，而地面上的天空盒反射仍然是使用Reflection Probe进行渲染的。比如《Thief》当年也分享过一个关于反射的PPT<很大，慎重>，主讲反射系统的构建，结合了很多很多种反射技术。
