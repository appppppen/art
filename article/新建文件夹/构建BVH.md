BVH的构建分为三个阶段。

1. 计算有关每个图元的边界信息并将其存储在数组中，该数组将在树构建期间使用。
2. 使用相应的分割算法来构建树，本文主要介绍四种分区算法(SAH, HLBVH, Middle, EqualCounts) ，构建的结果是一棵二叉树，其中每个内部节点持有指向其子节点的指针，每个叶子节点持有对一个或多个基元的引用。
3. 最后该树被转换为更紧凑(因此效率更高)的无指针表示，以在渲染期间使用。

我们根据这三个阶段写出BVH构建代码：

``` 
// BVH构造函数，构造BCH2叉树
BVHAccel::BVHAccel(std::vector<std::shared_ptr<Primitive>> p,
                   int maxPrimsInNode, SplitMethod splitMethod)
    : maxPrimsInNode(std::min(255, maxPrimsInNode)),
      splitMethod(splitMethod),
      primitives(std::move(p)) {

    // 利用图元构造BVH2叉树，从以下三个阶段进行

    // 1.初始化图元信息数组
    // 2.利用图元信息并使用相应的图元分割算法构建BVH
    // 3.树的节点用深度优先排列表示
}
```

对于要存储在BVH中的每个图元，我们将其边界框的质心，其完整边界框及其索引存储在BVHPrimitiveInfo结构体的图元数组中。该结构体如下：

``` 
struct BVHPrimitiveInfo {
    BVHPrimitiveInfo(size_t primitiveNumber, const Bounds3f &bounds)
        : primitiveNumber(primitiveNumber), bounds(bounds),
          centroid(.5f * bounds.pMin + .5f * bounds.pMax) { }
    size_t primitiveNumber;//图元索引
    Bounds3f bounds;//图元边界框
    Point3f centroid;//图元边界框的质心
};
```

我们初始化图元信息数组的代码则为：

``` 
//初始化图元信息数组
std::vector<BVHPrimitiveInfo> primitiveInfo(primitives.size());
for (size_t i = 0; i < primitives.size(); ++i)
    primitiveInfo[i] = { i, primitives[i]->WorldBound() };
```

接下来使用相应的图元分区算法以及图元信息构建SVH树，我们贴出代码：

``` 
// 使用图元信息与相应的图元分割算法构建SVH树
    MemoryArena arena(1024 * 1024);
    int totalNodes = 0;
    std::vector<std::shared_ptr<Primitive>> orderedPrims;
    orderedPrims.reserve(primitives.size());
    BVHBuildNode *root;//构建完成树的根节点
    if (splitMethod == SplitMethod::HLBVH)
        root = HLBVHBuild(arena, primitiveInfo, &totalNodes, orderedPrims);
    else
        root = recursiveBuild(arena, primitiveInfo, 0, primitives.size(), &totalNodes, orderedPrims);
    primitives.swap(orderedPrims);//orderedPrims是构造后排序过的图元，我们用其覆盖原图元
```

其中，我们把图元分区算法用枚举类型表示：

//SAH: 表面积启发式法。HLBVH：线性边界体积层次结构法。Middle：中点分割法。EqualCounts：等数量分割法
enum class SplitMethod { SAH, HLBVH, Middle, EqualCounts }; 

HLBVH方法我们利用函数HLBVHBuild()构建，其他三个方法在函数recursiveBuild()中区分构建。MemoryArena为PBRT中内存管理类，用以优化内存分配。

每个BVHBuildNode代表BVH的一个节点。 所有节点都存储一个Bounds3f，它表示该节点下所有子级的边界框。 每个内部节点在其子级中存储指向其两个子级的指针。 内部节点还记录了坐标轴，图元沿该坐标轴进行了划分，以分配给它们的两个子节点。叶子节点需要记录其中存储了哪些图元，索引从firstPrimOffset到firstPrimOffset+nPrimitives之间的图元都为该叶子结点内的图元。内部结点则不保存图元，仅保存边界框信息。BVHBuildNode结构如下：

``` 
struct BVHBuildNode {
    // 构造叶子结点
    void InitLeaf(int first, int n, const Bounds3f &b) {
        firstPrimOffset = first;
        nPrimitives = n;
        bounds = b;
        children[0] = children[1] = nullptr;
    }
    //构造内部结点
    void InitInterior(int axis, BVHBuildNode *c0, BVHBuildNode *c1) {
        children[0] = c0;
        children[1] = c1;
        bounds = Union(c0->bounds, c1->bounds);//Union表示联合c1和c2的边界框
        splitAxis = axis;
        nPrimitives = 0;
    }
    Bounds3f bounds;
    BVHBuildNode *children[2];
    int splitAxis, firstPrimOffset, nPrimitives;
};
```

除了用于分配节点内存的MemoryArena和保存图元信息的数组BVHPrimitiveInfo之外，recursiveBuild()还将范围[start，end）作为参数。它负责返回由从primaryInfo [start]到包括primitiveInfo [end-1]的范围表示的图元子集的BVH。如果此范围仅涵盖单个图元，则递归已触底并创建叶节点。否则，将使用一种分区算法在该范围内对数组的元素进行分区，并相应地对该范围内的数组元素进行重新排序，以使[start，mid）和[mid，end）的范围代表已分区的子集。如果分区成功，则将这两个图元集依次传递给递归调用，这些递归调用本身将返回指向当前节点的两个子节点的指针。totalNodes跟踪已创建的BVH节点的总数。orderedPrims数组用于存储已排序的图元引用。代码如下：

``` 
BVHBuildNode *BVHAccel::recursiveBuild(MemoryArena &arena,
        std::vector<BVHPrimitiveInfo> &primitiveInfo, int start,
        int end, int *totalNodes,
        std::vector<std::shared_ptr<Primitive>> &orderedPrims) {
    BVHBuildNode *node = arena.Alloc<BVHBuildNode>();
    (*totalNodes)++;
    //计算所有SVH图元的边界框
    int nPrimitives = end - start;
    if (nPrimitives == 1) {
        //创建叶子结点
    } else {
        //计算图元质心的边界，选择分割的坐标轴
        //将图元集分割成两个子集
    }
    return node;
}
```

首先展开计算边界框的代码：

``` 
//计算所有SVH图元的边界框
Bounds3f bounds;
for (int i = start; i < end; ++i)
    bounds = Union(bounds, primitiveInfo[i].bounds);
```

在叶节点，将与叶重叠的图元附加到orderedPrims数组，并初始化叶节点对象：

``` 
//创建叶子结点
int firstPrimOffset = orderedPrims.size();
for (int i = start; i < end; ++i) {
    int primNum = primitiveInfo[i].primitiveNumber;
    orderedPrims.push_back(primitives[primNum]);//由于前序遍历，所以保存是有序的
}
node->InitLeaf(firstPrimOffset, nPrimitives, bounds);
return node;
```

对于内部结点我们按什么方式为图元分区呢？我们通常选择一个坐标轴，图元覆盖该坐标轴具有最大的范围，如下图所示：

![](https://i2.wp.com/img-blog.csdnimg.cn/20200628132513258.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5MzAwMjM1,size_16,color_FFFFFF,t_70)

此处进行分区的总体目标是选择一个基本图元分区，该图元与两个所得基本图元集的边界框没有太多的重叠-如果存在实质性的重叠，遍历两个子树可能比更有效地分割图元的集合需要更多的计算。在讨论表面积启发式方法(SAH)时，将很快更加严格地找到有效的原始分区。

我们这里先写出寻找最大范围的坐标轴：

``` 
//计算图元质心的边界，选择分割的坐标轴
Bounds3f centroidBounds;
for (int i = start; i < end; ++i)
    centroidBounds = Union(centroidBounds, primitiveInfo[i].centroid);
int dim = centroidBounds.MaximumExtent();
```

如果所有质心点都在同一位置（即质心边界的体积为零），则递归停止，并使用图元创建叶节点； 在这种(异常)情况下，此处的分割方法均无效。 否则，将使用所选方法对图元进行分区，并将其传递给recursiveBuild()，对其进行两个递归调用。代码如下：

``` 
//将图元集分割成两个子集
int mid = (start + end) / 2;
if (centroidBounds.pMax[dim] == centroidBounds.pMin[dim]) {
    //创建叶子结点
} else {
    //根据不同的分割方法，分割图元集
    node->InitInterior(dim,
                       recursiveBuild(arena, primitiveInfo, start, mid,
                                      totalNodes, orderedPrims),
                       recursiveBuild(arena, primitiveInfo, mid, end,
                                      totalNodes, orderedPrims));
}
```

这里创建叶子结点代码与上部分创建叶子结点完全相同，在递归调用之前我们需要完成图元集分区，一个简单的splitMethod是Middle，它首先计算沿分割轴的图元质心的中点。根据它们的质心是在中点之上还是之下，将图元分为两组。使用C ++标准库函数std :: partition()可以轻松完成此分区，该函数接受数组中的一系列元素和比较函数，并对数组中的元素进行排序。std :: partition()返回一个指针，该指针指向该谓词具有假值的第一个元素，该元素将转换为对primitiveInfo数组的偏移量，以便我们可以将其传递给递归调用:

``` 
switch (splitMethod) {
    case SplitMethod::Middle: {
    // 中点法
    Float pmid = (centroidBounds.pMin[dim] + centroidBounds.pMax[dim]) / 2;
    BVHPrimitiveInfo *midPtr = std::partition(
        &primitiveInfo[start], &primitiveInfo[end - 1] + 1,
        [dim, pmid](const BVHPrimitiveInfo &pi) {
            return pi.centroid[dim] < pmid;
        });
    mid = midPtr - &primitiveInfo[0];
    // 对于许多带有大重叠边界框的素数，可能无法分区； 在这种情况下将不执行break跳出，而是进行下一种分区法(等数量分区法)
    if (mid != start && mid != end) break;
    }
    //...其他分区方法
}
```

如果所有图元都具有大的重叠边界框，则此拆分方法可能无法将图元分为两组。在这种情况下，执行将落入SplitMethod :: EqualCounts方法来重试。EqualCounts方法将图元划分为两个相等大小的子集，以使它们中的n个的前半部分是沿选定轴的质心坐标值最小的n / 2个，而后半部分是质心坐标值最大的半个子集。尽管这种方法有时效果很好，下图(b)的情况则这种方法的效果较差：
![](https://i2.wp.com/img-blog.csdnimg.cn/20200628145822722.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5MzAwMjM1,size_16,color_FFFFFF,t_70)

如图，a)对于某些图元分布，例如此处所示的分布，基于质心沿所选轴的中点(粗蓝线)进行分割的效果很好。(用虚线显示两个结果原始组的边界框。)(b)对于这种分布，中点是次优的选择；两个结果边界框基本重叠。(c)如果将(b)中的同一组图元沿着此处所示的线进行拆分，则生成的边界框会更小并且完全不重叠，从而在渲染时会带来更好的性能。

我们列出EqualCounts方法的代码：

``` 
switch (splitMethod) {
    //中点法
    case SplitMethod::Middle:{
    //...
    }
    //等数量法
    case SplitMethod::EqualCounts: {
    // 利用一半数量分区
    mid = (start + end) / 2;
    std::nth_element(&primitiveInfo[start], &primitiveInfo[mid],
                     &primitiveInfo[end - 1] + 1,
                     [dim](const BVHPrimitiveInfo &a,
                           const BVHPrimitiveInfo &b) {
                         return a.centroid[dim] < b.centroid[dim];
                     });
    break;
    }
}
```

通过标准库调用std :: nth_element()也可以轻松实现此方案。 它需要一个开始，中间和结束指针以及一个比较函数。 它对数组进行排序，以使中间指针处的元素是数组完全排序后将存在的元素，并且使得中间一个元素之前的所有元素小于中间元素，而后面一个元素的所有元素都小于中间元素 它比它更大。 这种排序可以在O(n)时间内完成，元素数量为n，这比对数组进行完全排序的O(nlogn)更有效。

EqualCounts和Middle是比较简单的方法，但同时会有比较大的不稳定性，比如一些特殊情况会使分区之后的遍历性能大大降低，如上述提到的图片示例。接下来我们介绍两种更好的方法。
