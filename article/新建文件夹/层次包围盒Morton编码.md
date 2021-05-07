Morton编码基于简单的变换：给定n维整数坐标值，通过对二进制中的坐标位进行交织来找到它们的Morton编码表示形式。例如，考虑2D坐标(x，y)，其中x和y的第i位二进制由x_{i}和y_{i}表示。相应的Morton编码值是:
![](https://img-blog.csdnimg.cn/20200628201043442.png)

下图以Morton顺序显示了2D点：
![](https://img-blog.csdnimg.cn/20200629091830999.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5MzAwMjM1,size_16,color_FFFFFF,t_70)

沿x和y轴的坐标值以二进制显示。如果我们按照整数坐标点的Morton索引的顺序连接它们，则会看到Morton曲线沿着“ z”形分层路径访问这些点，因此Morton路径有时称为“ z阶”。

Morton编码有一些非常重要的特点。比如我们在2D坐标中x和y的坐标是[0，15]中的整数，转换为Morton码有八位：，其中和为x，y的二进制第i位的值。我们会发现以下特点：

对于置高位的Morton编码值为1，我们知道设置了其基础y坐标的高位，因此y>=8(下图(a))，既坐标分布在整个空间的上半   区。下一个值等分了x轴(下图(b))。例如，如果置为1且为0，则对应点必须位于下图(c)的阴影区域中。的值将y轴区域四等分(下图4.8(d))。因此每个固定的Morton码都能指定一个唯一的空间位置，且位置和二次幂对齐。

![](https://img-blog.csdnimg.cn/20200629093412489.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5MzAwMjM1,size_16,color_FFFFFF,t_70)

LBVH是通过使用位于每个空间区域的中点的分割平面对基元进行分区而构建的BVH(即相当于先前定义的SplitMethod :: Middle方法)。因为它基于上述Morton编码的属性，所以分区非常有效。

在此处的实现中，我们将构建一个分层的线性边界体积层次(HLBVH)。通过这种方法，基于Morton的聚类首先用于为层次结构的较低级别构建树，然后使用表面积启发式方法(SAH)创建树的顶层。 HLBVHBuild()方法实现此方法，并返回结果树的根节点。代码如下：

``` cpp
BVHBuildNode *BVHAccel::HLBVHBuild(
    MemoryArena &arena, const std::vector<BVHPrimitiveInfo> &primitiveInfo,
    int *totalNodes,
    std::vector<std::shared_ptr<Primitive>> &orderedPrims) const {
    // +联合所有图元质心的边界框。(+)表示代码展开
    // +计算图元的morton编码
    // +基排序图元Morton编码
    // +在BVH底部创建LBVH子树
    // +从LBVH树中创建并返回SAH的BVH 
}
```

BVH是仅使用图元边界框的质心对它们进行排序而构建的，它不考虑每个图元的实际空间范围。 这种简化对于HLBVH提供的性能至关重要，但是这也意味着对于具有跨越多种大小的图元的场景，构建的树不会像基于SAH的树那样考虑这种变化。

由于Morton编码在整数坐标上进行操作，因此我们首先需要对所有图元的质心进行联合，以便可以相对于整个边界对质心位置进行插值量化：

``` cpp
// =计算所有图元质心的边界框 
Bounds3f bounds;
for (const BVHPrimitiveInfo &pi : primitiveInfo)
    bounds = Union(bounds, pi.centroid);
```

给定总体质心的边界，我们现在可以为每个图元计算Morton编码。 这是一个相当轻量级的计算，但是考虑到可能有数百万个图元，因此值得并行化。 请注意，循环块大小为512传递给下面的ParallelFor()， 这会导致为工作线程分配512个图元组进行处理，而不是一次处理一组，否则将是默认值(ParallelFor函数利用CPU执行并行计算，之后会写一篇介绍PBRT的并行计算)。 因为每个图元执行的用于计算Morton代码的工作量相对较小，所以这种粒度可以更好地分摊将任务分配给工作线程的开销：

``` cpp
// =计算Morton编码
std::vector<MortonPrimitive> mortonPrims(primitiveInfo.size());
ParallelFor([&](int i) {
    // +对第i个图元计算Morton编码
}, primitiveInfo.size(), 512);
```

为每个图元创建一个MortonPrimitive实例，它在图元信息数组中存储图元的索引及其Morton代码。结构体代码如下：

``` cpp
struct MortonPrimitive {
    int primitiveIndex;
    uint32_t mortonCode;
};
```

我们为x，y和z维度中的每一个使用10位，从而为Morton代码总共提供30位。 这种粒度允许值适合单个32位变量。 边界框内的浮点质心偏移位于[0, 1]中，因此我们将其缩放以获取适合10位的整数坐标。(对于偏移量完全等于1的边缘情况，可能会导致超出范围的量化值1024。这种情况在即将到来的LeftShift3()函数中处理。)

``` cpp
// =对第i个图元计算Morton编码
constexpr int mortonBits = 10;
constexpr int mortonScale = 1 << mortonBits
mortonPrims[i].primitiveIndex = primitiveInfo[i].primitiveNumber;
Vector3f centroidOffset = bounds.Offset(primitiveInfo[i].centroid);//计算质心偏移(归一化)
mortonPrims[i].mortonCode = EncodeMorton3(centroidOffset * mortonScale);
```

要计算3D的Morton代码，首先我们将定义一个辅助函数：LeftShift3()，接受32位值，并返回将第i位移位到第3i位的结果，而在其他位保留零。 下图说明了此操作：

![](https://img-blog.csdnimg.cn/20200629143534857.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5MzAwMjM1,size_16,color_FFFFFF,t_70)

实现此操作最明显的方法是将每个位值分别移位，但这并不是最有效的方法。(这将需要总共9个移位，以及逻辑OR才能计算最终值。)相反，我们可以将每个位的移位分解为2的幂次方的多个移位，这些移位将位的值一起移至其最终位置，比如9需要位移，8需要位移，6需要位移。然后，所有需要平移相同给定2的幂的所有位都可以一起平移，比如8和9可以一起平移，4到6可以一起平移。 LeftShift3()函数实现了此计算，下图显示了它是如何工作的：

![](https://img-blog.csdnimg.cn/20200629150631734.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5MzAwMjM1,size_16,color_FFFFFF,t_70)

上图步骤如下：

* 8，9一起平移2^{4}位。
* 4，5，6，7一起平移2^{3}位。
* 9，6，7，3，2一起平移2^{2}位。
* 9，7，5，3，1一起平移2^{1}位。

我们根据此写出LeftShift3()函数：

``` cpp
inline uint32_t LeftShift3(uint32_t x) {
    if (x == (1 << 10)) --x;
    x = (x | (x << 16)) & 0b00000011000000000000000011111111;
    x = (x | (x <<  8)) & 0b00000011000000001111000000001111;
    x = (x | (x <<  4)) & 0b00000011000011000011000011000011;
    x = (x | (x <<  2)) & 0b00001001001001001001001001001001;
    return x;
}
```

EncodeMorton3()函数采用3D坐标值，其中每个分量都是0到2^{10}之间的浮点值。 它将这些值转换为整数，然后通过函数LeftShift3()使它们的第i位平移到第3i位来计算Morton码，然后将y位再移位一位，将z位再移位两位，再进行或运算获取结果如下图：

![](https://img-blog.csdnimg.cn/20200629151604749.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5MzAwMjM1,size_16,color_FFFFFF,t_70)

EncodeMorton3()函数代码如下：

``` cpp
inline uint32_t EncodeMorton3(const Vector3f &v) {
    return (LeftShift3(v.z) << 2) | (LeftShift3(v.y) << 1) |
            LeftShift3(v.x);
}
```

计算完莫顿索引后，我们将使用基数排序按Morton码对mortonPrims进行排序。 这里使用基数排序实现比使用系统标准库中的std :: sort()更快(后者是快速排序和插入排序的混合)：

``` cpp
// =基排序图元Morton编码
RadixSort(&mortonPrims);
```

这里基数排序方法是一次对整数值进行排序，从最右边的数字到最左边的数字。 特别是对于二进制值，可以一次对多位进行排序； 这样做减少了访问数据的总次数。 在这里的实现中，bitsPerPass设置每次通过处理的位数； 值为6，我们有5个通道对30位进行排序：

``` cpp
static void RadixSort(std::vector<MortonPrimitive> *v) {
    std::vector<MortonPrimitive> tempVector(v->size());
    constexpr int bitsPerPass = 6;
    constexpr int nBits = 30;
    constexpr int nPasses = nBits / bitsPerPass;
    for (int pass = 0; pass < nPasses; ++pass) {
        // +执行一遍基数排序，对bitsPerPass位进行排序
    }
    // +奇数的情况下交换值
}
```

当前通道将对bitsPerPass位进行排序，从低位开始：

``` cpp
// =执行一遍基数排序，对bitsPerPass位进行排序
int lowBit = pass * bitsPerPass;
// +为基数排序设置in和out的vector指针
// +计算当前基数排序位的数组中零位的数量
// +计算每个存储区在输出数组中的起始索引
// +将排序后的值存储在输出数组中
```

输入和输出引用分别对应于要排序的vector和用于存储排序值的vector。每次通过循环都会在输入向量* v和tempVector之间交替：

// =为基数排序设置in和out的vector指针(避免额外申请空间)
std::vector<MortonPrimitive> &in  = (pass & 1) ? tempVector : *v; 
std::vector<MortonPrimitive> &out = (pass & 1) ? *v : tempVector; 

如果我们每次通过排序n位，则每个值可能有2^{n}个存储桶。我们首先计算每个桶中将有多少个值，这之后能使我们确定在输出数组中排序值的位置。 为了计算当前值的桶索引，我们需要对Morton索引(码)进行移位，以使索引lowBit位于0，掩盖低位的通道位：

``` cpp
// =计算当前基数排序位的数组中零位的数量
constexpr int nBuckets = 1 << bitsPerPass;//2^n
int bucketCount[nBuckets] = { 0 };
constexpr int bitMask = (1 << bitsPerPass) - 1;//这里是111111
for (const MortonPrimitive &mp : in) {
    int bucket = (mp.mortonCode >> lowBit) & bitMask;//右移lowBit后计算索引
    ++bucketCount[bucket];
```

给定每个存储区中有多少值计数，我们可以计算每个存储区值开始处的输出数组中的偏移量； 这只是前面的存储桶中有多少值的总和：

``` cpp
// =计算每个桶在输出数组中的起始索引
int outIndex[nBuckets];
outIndex[0] = 0;
for (int i = 1; i < nBuckets; ++i)
    outIndex[i] = outIndex[i - 1] + bucketCount[i - 1];
```

现在我们知道了从哪里开始存储每个桶的值，我们可以对图元进行再一次遍历，以重新计算每个Morton索引(码)所在的存储桶并将其MortonPrimitives存储在输出数组中。 这样就完成了当前位组的排序过程：

``` cpp
// =将排序后的值存储在输出数组中
for (const MortonPrimitive &mp : in) {
    int bucket = (mp.mortonCode >> lowBit) & bitMask;
    out[outIndex[bucket]++] = mp;//由于Morton码从小到大的顺序正好决定了图元空间从低到高的顺序，因此每次递增后都是有序的
}
```

排序完成后，如果执行了奇数次的基数排序，则需要将tempVector和*v交换。从而完成了仅用两个vector对所有通道的排序：

``` cpp
// =奇数的情况下交换值
if (nPasses & 1)
    std::swap(*v, tempVector);
```

当对所有通道完成遍历时，所有的图元质心就是有序的了。获得图元质心的排序数组后，我们可以找到给定质心附近的图元集群，然后在每个集群中的图元上创建LBVH。 这一步是一个很好的并行化步骤，因为通常有很多集群，并且每个集群都可以独立处理：
```cpp
// =在BVH底部创建LBVH子树
// +查找每个子树的图元间隔
// +并行为子树创建LBVH
```
每个图元群集由LBVHTreelet表示：

``` cpp
struct LBVHTreelet {
   int startIndex, nPrimitives;
   BVHBuildNode *buildNodes;//其相应的根结点
};
```

它对簇中第一个图元的mortonPrims数组中的索引以及后续图元的数量进行编码。如下图：

![](https://img-blog.csdnimg.cn/20200629193141825.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5MzAwMjM1,size_16,color_FFFFFF,t_70)

 图元质心聚集在统一的网格中。该4*4的网格内所有图元我们将其归结为簇。在该簇中，图元的30位莫顿代码的高12位具有相同的值。通过对mortonPrims数组进行线性遍历并找到高12位中的任何变化的偏移量来找到簇。这对应于在每个维度中具有个单元的总网格单元的规则网格中的聚类图元。实际上，尽管我们仍希望在此处找到许多独立的群集，但许多网格单元还是空的。

``` cpp
// =查找每个子树的图元间隔
std::vector<LBVHTreelet> treeletsToBuild;
for (int start = 0, end = 1; end <= (int)mortonPrims.size(); ++end) {
    uint32_t mask = 0b00111111111111000000000000000000;
    if (end == (int)mortonPrims.size() ||//分离所有簇：当图元高12位不相等，则添加子树。
        ((mortonPrims[start].mortonCode & mask) !=
         (mortonPrims[end].mortonCode & mask))) {
        // +将子树添加到treeletsToBuild
        start = end;
    }
}
```

当为树形图找到了一组图元时，将立即为其分配BVHBuildNodes。这里一个重要的细节是传递给MemoryArena :: Alloc()的false值，它指示不应执行所分配的基础对象的构造函数。 这里如果运行BVHBuildNode构造函数将引入了大量开销，并显着降低了整体HLBVH构造性能。 因为BVHBuildNode的所有成员都将在随后的代码中初始化，所以在任何情况下都不需要构造函数执行的初始化：

``` cpp
// =将子树添加到treeletsToBuild
int nPrimitives = end - start;
int maxBVHNodes = 2 * nPrimitives - 1;//树结点总数为2n-1
BVHBuildNode *nodes = arena.Alloc<BVHBuildNode>(maxBVHNodes, false);
treeletsToBuild.push_back({start, nPrimitives, nodes});
```

一旦确定了每个子树的图元，我们就可以为它们并行创建LBVH。 构造完成后，每个LBVHTreelet的buildNodes指针将指向相应LBVH的根。buildNodes保存的值与LBVH子树的前序遍历一致(在后面的buildNodes++体现)。

建立LBVH的工作线程必须在两个地方相互协调。 首先，需要计算所有LBVH中的节点总数，并通过传递到HLBVHBuild()的totalNodes指针返回该总数。 其次，当为LBVH创建叶节点时，需要orderedPrims数组的连续段来记录叶节点中图元的索引。 我们的实现使用了两个原子变量(可以在多线程中避免争用问题)，即atomicTotal用来跟踪节点数，而orderedPrimsOffset则用于orderedPrims中下一个可用条目的索引。

``` cpp
// =并行为子树创建LBVH
std::atomic<int> atomicTotal(0), orderedPrimsOffset(0);
orderedPrims.resize(primitives.size());
ParallelFor(
    [&](int i) {
        // +创建第i个子树
    }, treeletsToBuild.size());
*totalNodes = atomicTotal;
```

通过emitLBVH()来完成构建子树的工作，emitLBVH()会在空间的某些区域中获取具有质心的图元，并依次使用分裂平面将它们进行划分，这些分裂平面以固定的轴(x, y, z其中之一)将空间分成两半。使用emitLBVH()对每个树更新一次atomicTotal，与使用原子变量指针totalNodes指向atomicTotal相比，前者能提供明显更好的性能：

``` cpp
// =创建第i个子树
int nodesCreated = 0;
const int firstBitIndex = 29 - 12;
LBVHTreelet &tr = treeletsToBuild[i];//利用分离好的簇创建子树
tr.buildNodes = emitLBVH(tr.buildNodes, primitiveInfo, &mortonPrims[tr.startIndex],
             tr.nPrimitives, &nodesCreated, orderedPrims,
             &orderedPrimsOffset, firstBitIndex);
atomicTotal += nodesCreated;
```

由于采用了Morton编码，因此无需在emitLBVH()中明确表示当前的空间区域：传入的排序后的MortonPrims具有一定数量的匹配高位(前12位)，这就是对应了相应的空间范围。 对于Morton代码中其余位，此函数都尝试沿对应位bitIndex的平面拆分图元，然后递归调用自身。 尝试分割的下一位的索引作为该函数的最后一个参数传递：最初是29-12，因为29是第30位的索引(从零开始)，我们以前使用的高12位Morton编码的值来聚类图元。我们写出函数emitLBVH()的代码：

``` cpp
BVHBuildNode *BVHAccel::emitLBVH(BVHBuildNode *&buildNodes,
        const std::vector<BVHPrimitiveInfo> &primitiveInfo,
        MortonPrimitive *mortonPrims, int nPrimitives, int *totalNodes,
        std::vector<std::shared_ptr<Primitive>> &orderedPrims,
        std::atomic<int> *orderedPrimsOffset, int bitIndex) const {
    if (bitIndex == -1 || nPrimitives < maxPrimsInNode) {
        // +创建并返回LBVH子树的叶结点
    } else {
        int mask = 1 << bitIndex;
        // +如果此位没有LBVH拆分，则前进到下一个子树级别
        // +找到LBVH分割点
        // +创建并返回内部LBVH节点
    }
}
```

在emitLBVH()用最后的低位对图元进行分区之后，将无法再进行拆分并创建叶节点。 另外，如果节点数量较少，它也会停止并创建一个叶节点。

回想一下orderedPrimsOffset是orderedPrims数组中下一个可用元素的偏移量。 在这里，fetch_add()为原子操作，其调用将nPrimitives的值添加到orderedPrimsOffset，并在添加之前返回其旧值。 由于这些操作是原子操作，因此多个LBVH构造线程可以同时在orderedPrims数组中分配空间，而无需进行数据竞争。 给定数组中的空间，叶的构造与之前的类似：

``` cpp
// =创建并返回LBVH子树的叶结点
(*totalNodes)++;
BVHBuildNode *node = buildNodes++;
Bounds3f bounds;
int firstPrimOffset = orderedPrimsOffset->fetch_add(nPrimitives);
for (int i = 0; i < nPrimitives; ++i) {
    int primitiveIndex = mortonPrims[i].primitiveIndex;
    orderedPrims[firstPrimOffset + i] = primitives[primitiveIndex];
    bounds = Union(bounds, primitiveInfo[primitiveIndex].bounds);
}
node->InitLeaf(firstPrimOffset, nPrimitives, bounds);
return node;
```

可能所有的图元都位于分割平面的同一侧。由于基元按其Morton索引排序，因此可以通过查看范围内的第一个和最后一个基元在该平面上是否具有相同的位值来有效地检查这种情况。 在这种情况下，emitLBVH()会前进到下一位，而不必创建节点:

``` cpp
// =如果此位没有LBVH拆分，则前进到下一个子树级别
if ((mortonPrims[0].mortonCode & mask) ==
    (mortonPrims[nPrimitives - 1].mortonCode & mask))
    return emitLBVH(buildNodes, primitiveInfo, mortonPrims, nPrimitives,
                    totalNodes, orderedPrims, orderedPrimsOffset,
                    bitIndex - 1);
```

如果在拆分平面的两侧都有图元，则二分查找可以有效地找到当前图元集中bitIndexth位从0变为1的分割点:

``` cpp
// =找到LBVH分割点
int searchStart = 0, searchEnd = nPrimitives - 1;
while (searchStart + 1 != searchEnd) {
    int mid = (searchStart + searchEnd) / 2;//二分查找
    if ((mortonPrims[searchStart].mortonCode & mask) ==
        (mortonPrims[mid].mortonCode & mask))
        searchStart = mid;
    else
        searchEnd = mid;
}
int splitOffset = searchEnd;//分割点
```

一旦创建了所有LBVH子树，就可以使用buildUpperSAH()创建所有子树的BVH。 由于通常只有数十或数百个(不超过4096)，因此此步骤只需要很少的时间。

``` cpp
// =从LBVH树中创建并返回SAH的BVH 
std::vector<BVHBuildNode *> finishedTreelets;
for (LBVHTreelet &treelet : treeletsToBuild)
    finishedTreelets.push_back(treelet.buildNodes);
return buildUpperSAH(arena, finishedTreelets, 0,
                     finishedTreelets.size(), totalNodes);
```

buildUpperSAH()函数的实现与基于SAH的BVH构造几乎是一致的，只是在树的根节点而不是场景图元上，因此就不详细介绍了，直接贴出代码：

``` cpp
BVHBuildNode *BVHAccel::buildUpperSAH(MemoryArena &arena,
                                      std::vector<BVHBuildNode *> &treeletRoots,
                                      int start, int end,
                                      int *totalNodes) const {
    int nNodes = end - start;
    if (nNodes == 1) return treeletRoots[start];
    (*totalNodes)++;
    BVHBuildNode *node = arena.Alloc<BVHBuildNode>();
 
    // 计算此HLBVH结点下的所有图元边界框
    Bounds3f bounds;
    for (int i = start; i < end; ++i)
        bounds = Union(bounds, treeletRoots[i]->bounds);
 
    // 计算此HLBVH结点下的所有图元质心的边界框，并选择分割轴dim
    Bounds3f centroidBounds;
    for (int i = start; i < end; ++i) {
        Point3f centroid =
            (treeletRoots[i]->bounds.pMin + treeletRoots[i]->bounds.pMax) *
            0.5f;
        centroidBounds = Union(centroidBounds, centroid);
    }
    int dim = centroidBounds.MaximumExtent();
 
    // 为HLBVH的SAH分区的桶初始化信息_BucketInfo_
    PBRT_CONSTEXPR int nBuckets = 12;
    struct BucketInfo {
        int count = 0;
        Bounds3f bounds;
    };
    BucketInfo buckets[nBuckets];
    for (int i = start; i < end; ++i) {
        Float centroid = (treeletRoots[i]->bounds.pMin[dim] +
                          treeletRoots[i]->bounds.pMax[dim]) *
                         0.5f;
        int b =
            nBuckets * ((centroid - centroidBounds.pMin[dim]) /
                        (centroidBounds.pMax[dim] - centroidBounds.pMin[dim]));
        if (b == nBuckets) b = nBuckets - 1;
        buckets[b].count++;
        buckets[b].bounds = Union(buckets[b].bounds, treeletRoots[i]->bounds);
    }
 
    // 计算每个桶的分区花费(划分点在桶位置后)
    Float cost[nBuckets - 1];
    for (int i = 0; i < nBuckets - 1; ++i) {
        Bounds3f b0, b1;
        int count0 = 0, count1 = 0;
        for (int j = 0; j <= i; ++j) {
            b0 = Union(b0, buckets[j].bounds);
            count0 += buckets[j].count;
        }
        for (int j = i + 1; j < nBuckets; ++j) {
            b1 = Union(b1, buckets[j].bounds);
            count1 += buckets[j].count;
        }
        cost[i] = .125f +
                  (count0 * b0.SurfaceArea() + count1 * b1.SurfaceArea()) /
                      bounds.SurfaceArea();
    }
 
    // 寻找最小的SAH花费
    Float minCost = cost[0];
    int minCostSplitBucket = 0;
    for (int i = 1; i < nBuckets - 1; ++i) {
        if (cost[i] < minCost) {
            minCost = cost[i];
            minCostSplitBucket = i;
        }
    }
 
    // 在选定的SAH存储桶中创建叶子结点或拆分图元集
    BVHBuildNode **pmid = std::partition(
        &treeletRoots[start], &treeletRoots[end - 1] + 1,
        [=](const BVHBuildNode *node) {
            Float centroid =
                (node->bounds.pMin[dim] + node->bounds.pMax[dim]) * 0.5f;
            int b = nBuckets *
                    ((centroid - centroidBounds.pMin[dim]) /
                     (centroidBounds.pMax[dim] - centroidBounds.pMin[dim]));
            if (b == nBuckets) b = nBuckets - 1;
            return b <= minCostSplitBucket;
        });
    int mid = pmid - &treeletRoots[0];
    node->InitInterior(
        dim, this->buildUpperSAH(arena, treeletRoots, start, mid, totalNodes),
        this->buildUpperSAH(arena, treeletRoots, mid, end, totalNodes));
    return node;
}
```

本篇核心思想是利用morton技术对图元分割出LBVH子树，利用线性分割后的子树采用SAH计算出最佳分割方案，如下图：

![](https://img-blog.csdnimg.cn/20200701142016661.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5MzAwMjM1,size_16,color_FFFFFF,t_70)
