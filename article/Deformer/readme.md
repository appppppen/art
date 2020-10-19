目前 fbx 2015.1 中支持三种变形器：skinDeformer，blendShapeDeformer，vertexCacheDeformer。定义在 fbxdeformer.h 中：

```csharp
enum EDeformerType {
　　eUnknown, //!< Unknown deformer type
　　eSkin, //!< Type FbxSkin
　　eBlendShape, //!< Type FbxBlendShape
　　eVertexCache //!< Type FbxVertexCacheDeformer

};
```

前两种变形器分别对应：骨骼蒙皮动画 和 变形动画，（第三种还没研究）。

相应的播放代码都可以在 ViewScene 示例中找到。

其中 骨骼蒙皮动画 游戏里用得最多，所以研究得较早。这几天又看了一下 变形动画(BlendShape/Morph animation)，散乱记录一下若干注意事项。

## 一，变形(blendShape/morph)基本原理。

大体上讲就是在拓扑结构相同的 mesh 之间插值。细节在下文中会提到。
典型应用是做表情动画。

## 二，BlendShape 动画使用

### Maya 中创建 BlendShape 动画

Blend Shape（形状融合变形器）是制作面部表情动画的有力武器，它能通过使用一系列的目标形状物体（Target）使基础物体得到非常平顺、高精度的变形效果。它在角色动画的时候非常受用，尤其是在表情的制作，基本上都是用它来完成。

1. 创建一个 Sphere 球体名为 pSphere1，之后复制一个球体修改名为 BlendShape_sphere；

![](https://img-blog.csdnimg.cn/20181105135406439.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMyOTM1ODA=,size_16,color_FFFFFF,t_70)

2. 选中 pSphere1 进入点编辑模式，使用软选择工具，控制调整点的位置，创建球体的变形终结状态；

![](https://img-blog.csdnimg.cn/20181105135445367.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMyOTM1ODA=,size_16,color_FFFFFF,t_70)

3. 选中 pSphere1 之后加选 BlendShape_sphere 物体，在 Rigging 模块下选择菜单中的 Deform——Blend Shape 选项给物体 BlendShape_sphere 添加 BlendShape 属性节点；

![](https://img-blog.csdnimg.cn/20181105135508498.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMyOTM1ODA=,size_16,color_FFFFFF,t_70)

4. 之后可以看到 BlendShape_sphere 属性编辑器中多了一个输入属性

![](https://img-blog.csdnimg.cn/20181105135521327.png)

5. BlendShape_sphere 就是带有 BlendShape 属性的模型了，修改 pSphere1 属性的值就可以看到球体从默认圆形状态过度为变形终结状态，可以导出测试效果了

### 带有 BlendeShape 属性的物体导入 Unity

Maya 制作的融合变形动画（带有融合变形属性）的物体，导出时设置，可以没有骨骼，如果带有动画需要烘培动画，之后保证 DeformedModels——BlendShapes 属性勾选，若没有动画关键帧可以不烘培动画。下图为 Maya 输出 fbx 文件设置；

![](https://img-blog.csdnimg.cn/20181105135633701.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMyOTM1ODA=,size_16,color_FFFFFF,t_70)

带有 blendShape 节点属性的模型导入 Unity 之后，在模型的 SkinnedMeshRenderer 组件下多出 BlendShapes 属性，控制该属性下的值可以实现变形的效果，实现自主控制动画效果。下图为导入 Unity 中的 Skinned Mesh Renderer 组件属性列表

![](https://img-blog.csdnimg.cn/20181105135719822.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMyOTM1ODA=,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20181105135727942.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMyOTM1ODA=,size_16,color_FFFFFF,t_70)
之后就可以根据需求控制修改 blendShape1.pSphere2 值的变化实现动画效果 。

## 三，fbx 变形动画原理。

fbx sdk 的 ViewScene 示例中的 ComputeShapeDeformation()函数实现变形动画的数据提取和播放，通过阅读这部分代码，可以了解 fbx 中变形动画数据的逻辑结构：

- 一个 mesh 包含多个 blendShape，
- 一个 blendShape 包含多个 blendShapeChannel，
- 一个 blendShapeChannel 包含多个 targetShape 和一个 weight 动画曲线 weightCurve。

targetShape 中包含 controlPoints 和一个 fullWeight 值。

这些结构与 3dmax 中的对应关系如下：

- mesh 对应的就是 3dmax 中的模型，
- blendShape 对应“变形器”修改器，一个模型可以添加多个“变形器”修改器，
- blendShapeChannel 对应“变形器”修改器的通道，- blendShapeChannel->GetName()得到通道名称.
- targetShape 对应变形目标，一个通道可以添加多个变形目标。- targetShape->GetName()得到变形目标名称。

通道的 weightCurve 给出各时间点此通道变形进度 weight。

targetShape 的 fullWeight 值表示的是本通道变形进度 weight 值达到多少时恰好完全变形为本 targetShape。对于只有一个 targetShape 的通道，targetShape 的 fullWeight 必定是 100；而对于含有多个 targetShape 的通道，各 targetShape 的 fullWeight 值按顺序依次增大，并且最后一个 targetShape 的 fullWeight 必定是 100。

可见，非常符合直观，至此变形插值算法几乎不用再看 ComputeShapeDeformation()的代码便可以直接想象出来了：

1. 对于只有一个变形目标的通道（非渐近变形），只要根据通道变形进度 weight 在变形物体和变形目标间插值。

2. 对于含有多个变形目标的通道（渐近变形），看变形进度 weight 落在哪两个变形目标的 fullWeight 之间，然后计算 weight 将此区间分成的比例得到此区间上的变形进度 weightOfSpan，再根据此区间变形进度在上述两个变形目标间插值。

下面是 ComputeShapeDeformation()中的注释，用具体例子说明在 progressive morph 和 progressive morph 情况下的插值算法，与我们的直观想象完全一致：

If there is only one targetShape on this channel, the influence is easy to calculate:

influence = (targetShape - baseGeometry) _ weight _ 0.01

dstGeometry = baseGeometry + influence

But if there are more than one targetShapes on this channel, this is an in-between

blendshape, also called progressive morph. The calculation of influence is different.

For example, given two in-between targets, the full weight percentage of first target

is 50, and the full weight percentage of the second target is 100.

When the weight percentage reach 50, the base geometry is already be fully morphed

to the first target shape. When the weight go over 50, it begin to morph from the

first target shape to the second target shape.

To calculate influence when the weight percentage is 25:

1. 25 falls in the scope of 0 and 50, the morphing is from base geometry to the first target.

2. And since 25 is already half way between 0 and 50, so the real weight percentage change to

the first target is 50.

influence = (firstTargetShape - baseGeometry) _ (25-0)/(50-0) _ 100

dstGeometry = baseGeometry + influence

To calculate influence when the weight percentage is 75:

1. 75 falls in the scope of 50 and 100, the morphing is from the first target to the second.

2. And since 75 is already half way between 50 and 100, so the real weight percentage change

to the second target is 50.

influence = (secondTargetShape - firstTargetShape) _ (75-50)/(100-50) _ 100

dstGeometry = firstTargetShape + influence

## 四，多通道同时影响时的叠加方式。

假设变形物体 X 受 channel1 和 channel2 两个通道影响，channel1 中只有一个变形目标 shape1，通道变形进度为 w1；channel2 中只有一个变形目标 shape2，通道变形进度为 w2。设 v 是 X 上一点，原始坐标为 p；v1 是 shape1 上等位点，坐标为 p1；v2 是 shape2 上等位点，坐标为 p2。

则 v 在两个通道影响下变形后的坐标 p_deformed 应计算如下：

p_deformedByChannel1=p+(p1-p)\*w1

p_deformedByChannel1andChannel2=p_deformedByChannel1+(p2-p_deformedByChannel1)\*w2

p_deformed=p_deformedByChannel1andChannel2

## 五，法线问题。

对变形物体进行变形，如果只是顶点位置发生变化，而法线不变的话，那么在有光照的情况下显示效果是不对的，所以法线也需要在变形物体和变形目标之间进行插值。

前面分析 fbx 变形动画数据逻辑结构时提到“targetShape 中包含 controlPoints 和一个 fullWeight 值”，是因为在 viewScene 示例中，只使用 targetShape 的 controlPoints 对变形物体的顶点位置进行了变形，而根本没有处理法线的变形。我没有找到由 targetShape 获得有效 normals 的方法。而且当我把动画导出为 ascii 的 fbx，查看 targetShape 的 normals 数据时发现全是 0。

我目前的解决办法是通过 targetShape 获得 targetShapeName，然后再按名称在场景中搜索到与 targetShape 相对应的模型（mesh），然后从 mesh 中取得法线数据。这样就要求变形目标模型不能删，且要随变形物体一同导出到 fbx 文件中。另外一个需要注意的问题是，由于我的引擎中只支持三角网，于是要求或者在 3dmax 建模时物体就建成三角网，或者在导出为 fbx 文件时勾选“三角化”选项，或者在引擎载入 fbx 文件之后调用 fbx sdk 提供的 api 转换为三角网。但是后两者都不能保证变形物体 mesh 和变形目标 mesh 在三角化后仍具有完全相同的拓扑结构。而如果变形物体 mesh 和变形目标 mesh 的拓扑结构不同，法线插值就无法进行。

注意：顶点位置插值是不要求变形物体 mesh 和变形目标 mesh 拓扑结构相同的，只要顶点能一一对应，顶点位置插值就可以进行，所以如果像 ViewScene 示例那样只处理顶点变形而不考虑法线变形的话，不必要求模型在 3dmax 中就建成三角网。（不需要考虑法线变形的情况确实是存在的，例如的像《地铁跑酷》那种无光照的 Q 版 3d 游戏，根本不需要法线，因此做变形时也不用考虑法线变形）。

法线插值是向量间插值，与顶点位置插值（点之间插值）是不同的。点之间的插值直接线性插值即可，而向量间插值严格来讲应该使用球面插值（slerp）。不过经过试验，发现在此处两种方法的视觉差异并不很大，且线性插值速度要快得多，所以最后我仍然选择了线性插值。

## 六，ViewScene 示例中变形动画代码有错误。

ViewScene 示例中播放变形动画的代码是有错误的，将动画由 3dmax 中导出为 fbx 再用 viewScene 示例播放，会发现 viewScene 示例播放结果与 3dmax 中不一样，而且动画会出现跳变等明显错误。

我现在已经记不清具体是什么原因造成的了，但我在自己的播放程序中解决了这些问题。
