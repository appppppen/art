![](https://pic4.zhimg.com/80/v2-5bff610798835ab95d23a5a591e47ccb_1440w.jpg)

## 引言

为什么要引入BVH（Bounding Volume Hierarchies 层次包围盒）？

在之前，我们每发出一条射线就要进行许多次的求交运算（和渲染的物体数量成正比，也就是说时间复杂度 O(n)）；求交命中了还好，要是求交点不中，就相当于之前所作的运算全白费了（大概率是命中不了）。在这种情况下渲染时间就会变得超长。

在此背景下引入了一系列的算法优化，其中一个就是BVH。

具体来说，BVH的核心思想就是用体积略大而几何特征简单的包围盒来近似描述复杂的几何对象，并且这种包围盒是嵌套的，我们只需要对包围盒进行进一步的相交测试，就可以越来越逼近实际对象（很明显这个功能需要用到树形的层次结构）。

## BVH

bvh的思路其实还是比较容易理解，最naive的理解方式是 - 用盒子把物体包起来，检测光是否和盒子相撞，相撞 - 近一步检测盒子里的物体，否则丢弃盒子。

![](https://pic4.zhimg.com/80/v2-5400517a23094c159f941b21fee4a4c7_1440w.jpg)

### ray - box intersection

首先看2d平面的射线和盒子，这个盒子是AABB盒子（axis aligned bounding box)，跟坐标轴平行：
![](https://pic4.zhimg.com/80/v2-8f519274bfc26dff24576d83e46b6037_720w.jpg)

射线 p(t) = e + td, 可能的交点可以通过式子 ![](https://www.zhihu.com/equation?tex=x_e+%2B+t+x_d+%3D+x_%7Bmin%7D) 来计算，所以四个可能的交点
![](https://www.zhihu.com/equation?tex=t_%7Bxmax%7D+%3D+%28x_%7Bmax%7D+-+x_e%29+%2F+x_d)

![](https://www.zhihu.com/equation?tex=t_%7Bxmax%7D+%3D+%28x_%7Bmax%7D+-+x_e%29+%2F+x_d)

![](https://www.zhihu.com/equation?tex=t_%7Bymin%7D+%3D+%28y_%7Bmin%7D+-+y_e%29+%2F+y_d)
![](https://www.zhihu.com/equation?tex=t_%7Bymax%7D+%3D+%28y_%7Bmax%7D+-+y_e%29+%2F+y_d)

有两种可能不想交的情况：

![](https://pic1.zhimg.com/80/v2-211a98fe9ea61c2ce78d12e1720a9910_720w.jpg)

即：

* ![](https://www.zhihu.com/equation?tex=t_%7Bymax%7D+%3C+t_%7Bxmin%7D)

* ![](https://www.zhihu.com/equation?tex=t_%7Bxmax%7D+%3C+t_%7Bymin%7D)

也可以这样看，那就是 (tymin, tymax) 和 (txmin, txmax)有重叠 overlap的时候相交：

![](https://pic1.zhimg.com/80/v2-b19d416b3601f5a3558bac55281b2004_720w.jpg)

![](https://pic1.zhimg.com/80/v2-50f9aac69513959d214476a2ada7d490_720w.jpg)

这个状况我们可以推广到3维空间，但是还需要解决一些问题：

光线平行于x轴或者y轴， 除0
光线方向是负x或者负y方向，txmin和tymin更大
感谢compiler 帮忙处理一些 IEEE floating point 除0的问题，比如：

cout << 1.9 / 0.0 << endl; 
// inf

cout << 1.9 / -0.0 << endl; 
// -inf
然后对于光线的方向，我们可以交换tmin和tmax，所以代码看起来也很清晰：

``` 

class aabb {
public:
  aabb () {}
  aabb(const vec3& a, const vec3& b) { _min = a, _max = b; }

  vec3 min() const { return _min; }
  vec3 max() const { return _max; }

  bool hit(const ray& r, float tmin, float tmax) const{
    for (int a = 0; a < 3; a++) {
      float invD = 1.0f / r.direction()[a];
      float t0 = (min()[a] - r.origin()[a]) * invD;
      float t1 = (max()[a] - r.origin()[a]) * invD;
      if (invD < 0.0f) std::swap(t0, t1);
      tmin = t0 > tmin ? t0 : tmin;
      tmax = t1 < tmax ? t1 : tmax;
      if (tmax <= tmin) return false;
    }
    return true;
  }

  vec3 _min, _max;
};
```

### bounding box

然后我们需要把物体用盒子包起来，首先清楚有些物体可以有bounding box - sphere, moving_sphere, box本身，但是无穷大的平面比如 z = 1 就不可能有bounding box，所以我们的 hitable 中添加函数来看物体是否可以被bounding：

``` 

class hitable {
public:
  virtual bool hit(const ray &r, float t_min, float t_max, hit_record& rec) const = 0;
  // bool stands for whether this object can have a bounding box
  virtual bool bounding_box(float t0, float t1, aabb box) const = 0;
};
```

球的bounding box很容易：

``` 

bool sphere::bounding_box(float t0, float t1, aabb box) const{
  box = aabb(center - vec3(radius, radius, radius), center + vec3(radius, radius, radius));
  return true;
}
```

移动的球也不难， 我们算出 t0 时候的 bounding box，t1 时候的 bounding box ，然后把这两个bounding box 能bound 住的bounding box就是我们求得结果:

``` 

bool moving_sphere::bounding_box(float t0, float t1, aabb& box) const {
  aabb b0 = aabb(center0 - vec3(radius, radius, radius), center0 + vec3(radius, radius, radius));
  aabb b1 = aabb(center1 - vec3(radius, radius, radius), center1 + vec3(radius, radius, radius));
  box = surrounding_box(b0, b1);
  return true;
}
```

包围两个盒子的盒子：

``` 

aabb surrounding_box(aabb box0, aabb box1){
  vec3 small(std::fmin(box0.min().x(),box1.min().x()),
             std::fmin(box0.min().y(),box1.min().y()),
             std::fmin(box0.min().z(),box1.min().z()));

  vec3 big(std::fmax(box0.max().x(),box1.max().x()),
           std::fmax(box0.max().y(),box1.max().y()),
           std::fmax(box0.max().z(),box1.max().z()));

  return aabb(small, big);
}
```

### BVH 数据结构

这个数据结构最难的部分应当是建立它：

// bvh 'data structure' build
bvh_node::bvh_node(hitable **l, int n, float time0, float time1){
  int axis = int(3*drand48()); 
  if (axis == 0)  {

    // we sort by x axis
    // the hitable list will be ordered by box_x_compare function
    std::qsort(l, n, sizeof(hitable *), box_x_compare);

  } else if (axis == 1){

    std::qsort(l, n, sizeof(hitable *), box_y_compare);

  } else{

    std::qsort(l, n, sizeof(hitable *), box_z_compare);

  }
  if (n == 1) {

    left = right = l[0];

  } else if (n==2){

    left = l[0];
    right = l[1];

  } else {

    left = new bvh_node(l, n/2, time0, time1);
    right = new bvh_node(l + n/2, n - n/2, time0, time1);

  }
  aabb box_left, box_right; 
  if (!left->bounding_box(time0, time1, box_left) ||

      !right->bounding_box(time0, time1, box_right)) {
    std::cerr << "no bounding box in bvh_node constructor" << '\n';

  }
  box = surrounding_box(box_left, box_right); 
}
通过这样递归的方式被建立起来，比较的时候也可以递归的比较，先root是否被hit，然后依次检查左右：

// bvh 'data structure' build
bvh_node::bvh_node(hitable **l, int n, float time0, float time1){
  int axis = int(3*drand48()); 
  if (axis == 0)  {

    // we sort by x axis
    // the hitable list will be ordered by box_x_compare function
    std::qsort(l, n, sizeof(hitable *), box_x_compare);

  } else if (axis == 1){

    std::qsort(l, n, sizeof(hitable *), box_y_compare);

  } else{

    std::qsort(l, n, sizeof(hitable *), box_z_compare);

  }
  if (n == 1) {

    left = right = l[0];

  } else if (n==2){

    left = l[0];
    right = l[1];

  } else {

    left = new bvh_node(l, n/2, time0, time1);
    right = new bvh_node(l + n/2, n - n/2, time0, time1);

  }
  aabb box_left, box_right; 
  if (!left->bounding_box(time0, time1, box_left) ||

      !right->bounding_box(time0, time1, box_right)) {
    std::cerr << "no bounding box in bvh_node constructor" << '\n';

  }
  box = surrounding_box(box_left, box_right); 
}
其实最后需要在main.cpp 中修改的只有很小的部分，把 random_scene 返回由 hitable_list 变成 bvh_node.

时间对比， 渲染 10*10 个小球：
![](https://www.zhihu.com/equation?tex=%5Cbegin%7Barray%7D%7Bc%7Cc%7D+%5Ctext%7Bdepth%7D+%26+%5Ctext%7Btime%7D+%5C%5C+%5Chline+hitable%5C_list%26++920+%28sec%29%5C%5C+bvh%5C_node+%26+460+%28sec%29%5C%5C+%5Cend%7Barray%7D%5C%5C)

## 求交

首先我们需要将场景中的物体全都容纳在一个大的包围盒里面。然后再进一步细分。我们不妨设置一个简单的长方体包围盒（2D）。

![](https://pic4.zhimg.com/80/v2-674f6c54f83aa2d4beb772a374dba277_1440w.jpg)
假设此时存在一条射线，我们怎么去判断这条射线和包围盒有没有相交？（当然是用眼睛看=_=）我想下图已经很明白了：拿出一个包围盒，延长它的边线，得到一个像“＃”一样的图形。如果射线与包围盒相交，那么对应的t值区间是有重叠（overlap）的，如果不相交，则没有重叠。（当然还有别种方法检测是否重叠）

![](https://pic2.zhimg.com/80/v2-d9bc00cf6ba035c175af79afbe55b83d_1440w.jpg)

给出代码：

bool hit(const ray& r, float tmin, float tmax) const
{

    for(int a = 0; a < 2; a++)  //two axis
    {
        float invD = 1.0f / r.direction()[a];  //note: operator[] is overloaded
        //according to the formula: P(t) = origin + direction * t
        float t0 = (bounding_box.min()[a] - r.origin()[a]) * invD;  //multiplication is                                                                     //faster 
        float t1 = (bounding_box.max()[a] - r.origin()[a]) * invD;
        //t0 must less than t1
        if (invD < 0.0f) std::swap(t0, t1);

        tmin = (tmin > t0) ? tmin : t0;
        tmax = (tmax < t1) ? tmax : t1;

        if (tmax <= tmin)  //"=" :if wiped the edge, it is defined as missed
            return false;
    }
    return true;

}

倘若不明白，可以跟着代码走一遍。

上面代码还有需要注意的地方，也就是当r.direction平行于轴的时候：

t0和t1都会同时为正无穷或者负无穷（因为有+0和-0），我们通过设置最大最小值来解决这个问题。这也就是一开始需要传入tmin和tmax的原因。
在r.direction平行于轴的前提下，如果origin位于包围盒边缘，即:
bounding_box.min()[a] == r.origin()[a]; 
//or
bounding_box.max()[a] == r.origin()[a]; 
我们会得到NaN（not a number)，这种情况下，由于任何与NaN的比较都会返回false，因此tmin和tmax的比较也会返回false，此时我们当作它hit成功。

## 创建Bounding box

这个比较简单，直接给出源码：

对于sphere类：

bool bounding_box(float t0, float t1, aabb& box) const
{

	box = aabb(center - vec3(radius, radius, radius), center + vec3(radius, radius, radius));

}
//t0, t1 in there is not used, just because of pure virtual function in the class
//of hittable
对于moving sphere类：

bool bounding_box(float t0, float t1, aabb& box) const
{

    //center(t) is a function that the center of the sphere change over time
    aabb box0(center(t0) - vec3(radius, radius, radius),
             center(t0)+ vec3(radius, radius, radius));
    aabb box1(center(t1) - vec3(radius, radius, radius),
		center(t1) + vec3(radius, radius, radius));
    box = surrounding_box(box0, box1);

}
包围两个盒子的盒子：

inline float ffmin(float a, float b) {a < b ? a : b}; 
inline float ffmax(float a, float b) {a > b ? a : b}; 
aabb surrounding_box(aabb box0, aabb box1)
{

	vec3 small(ffmin(box0.min().x(), box1.min().x()),
		ffmin(box0.min().y(), box1.min.y()),
		ffmin(box0.min().z(), box1.min.z()));
	vec3 big(ffmax(box0.max().x(), box1.max().x()),
		ffmax(box0.max().y(), box1.max().y()),
		ffmax(box0.max().z(), box1.max.z()));

	return aabb(small, big);

}
构建bounding box列表（嵌套box）

bool bounding_box(float t0, float t1, aabb& box)    {
if(list_size < 1)

       return false;

   aabb temp_box; 
   bool first_true = list[0]->bounding_box(t0, t1, temp_box); //Polymorphism, it will                                    //call the sphere or moving_sphere's bounding_box

   if (!first_true) 

       return false;

   else

       box = temp_box;

   for(int i = 1; i < list_size; i++)
   {

       if(list[i]->bounding_box(t0, t1, temp_box))
           box = surrounding_box(box, temp_box);
       else
           return false;

   }
   return true; 
}   
 //this function will be called at BVH construction

## 创建BVH树

首先给出BVH类的整体实现：

class bvh_node : public hittable
{
public:

    bvh_node(){};
    bvh_node(hittable **l, int n, float t0, float t1);

    virtual bool hit(const ray&r, float tmin, float tmax, hit_record& rec) const;
    virtual bool bounding_box(float t0, float t1, aabb& b) const;

    hittable* left;
    hittable* right;
    aabb box;

}
//return itself
bool bvh_node::bounding_box(float t0, float t1, aabb& b) const
{

    b = box;
    return true;

}
我们需要注意一下child指针left和right，它们的指针类型是hittable ，即通用指针，这就意味着它们可以指向其他的 bvh_node，或者sphere类，又或者其他继承自hittable*类的子类型。这有利于多态实现。

码接上文，先给出bvh_node构造函数的实现：

bvh_node::bvh_node(hittable **l, int n, float t0, float t1)
{

    int axis = int(3 * random_double()); //random_double will give a number between 0 & 1
    //sort
    if (axis == 0) // x-axis
        qsort(l, n, sizeof(hittable*), box_x_compare);
    else if (axis == 1) //y -axis
        qsort(l, n, sizeof(hittable*), box_y_compare);
    else
        qsort(l, n, sizeof(hittable*), box_z_compare);

    if (n == 1)
        left = right = l[0];
    else if (n == 2)
    {
        left = l[0];
        right = l[1];
    }
    else
    {
        left = new bvh_node(l, n / 2, t0, t1);
        right = new bvh_node(l + n / 2, n - n / 2, t0, t1);
    }

    aabb box_left, box_right;

    if (!left->bounding_box(t0, t1, box_left) ||
        !right->bounding_box(t0, t1, box_right))
        std::err << "no bounding box in bvh_node constructor" << std::endl;

    box = surrounding_box(box_left, box_right);

}

int box_x_compare (const void * a, const void * b) {

    aabb box_left, box_right;

    hittable *ah = *(hittable**)a; 
    hittable *bh = *(hittable**)b; 

    if (!ah->bounding_box(0,0, box_left) || !bh->bounding_box(0,0, box_right))
        std::cerr << "no bounding box in bvh_node constructor\n";

    if (box_left.min().x() - box_right.min().x() < 0.0)
        return -1;
    else
        return 1;

}
//box_y_compare and box_z_compare are just like box_x_compare, so I won't give them.
兴许这里需要详细解释一下：

一开始我们将hittable l 即所有物体进行快速排序，排序依据随机（x y z）；

之后开始构建BVH树：

对于只有一个物体的hittable list ，将左子树和右子树都赋值为这个物体，之后分别调用bounding_box，将左右子树的物体（同一个）外面都用包围盒围起来（实质上是构建一个虚拟盒子），再然后调用surrounding_box将左右子树物体包成一个box。此时左右子树为左右叶子节点。
对于两个物体的hittable list，将左子树赋值为第一个物体，右子树赋值为第二个物体，之后分别调用 bounding_box，将两个物体外面都用包围盒围起来，再然后调用surrounding_box将左右子树物体包成一个box。此时左右子树为左右叶子节点。
对于超过两个物体的hittable list，左子树为新构建的BVH节点，右子树也为新构建的BVH节点，递归下去，直到 **n = 2**的时候终止递归（记住在每一个节点中都是先递归左子树再递归右子树）。
终止递归之后，对于 **n = 2** ，将当前节点的左右叶子节点（左右物体）用bounding_box包围起来，然后包成一个box；执行完之后返回到上一层节点 **n = 4**。

返回到节点 **n = 4**之后，会开始构建右子树的叶子节点，同样的用bounding_box包围，然后包成一个box；

当节点 **n = 4**的右子树也构建完了之后，就会依照运行顺序继续往下运行，也即：调用bounding_box分别将左右子树的两个物体再包一遍（这里调用的bounding_box是bvh_node的，也即返回自身的那个），然后将左右子树的物体包成一个box；

一直这样子持续下去，直到最后变成一个最大的box。

值得一提的是，每次递归，都会导致hittable l 的局部排序方式改变。

构建完了BVH树之后，接下来就是hit函数的判定了，同样先给出代码：

``` cpp
bool bvh_node::hit(const ray& r, float tmin, float tmax, hit_record& rec) const 
{
  //box.hit is the most large aabb's hit--yes, we gave it at the beginning of the article

    if(box.hit(r, tmin, tmax))
    {
        hit_record left_rec, right_rec;
        bool hit_left = left->hit(r, tmin, tmax, left_rec);
        bool hit_right = right->hit(r, tmin, tmax, right_rec);

        if (hit_left && hit_right)
        {
            if (left_rec.t < right_rec.t)
                rec = left_rec;
            else
                rec = right_rec;
            return true;
        }
        else if (hit_left)
        {
            rec = left_rec;
            return true;
        }
        else if (hit_right)
        {
            rec = right_rec;
            return true;
        }
        else
            return false;
    }
    else 
        return false;

}
```

bvh_node::hit函数一开始就调用aabb类的hit函数（用的是最大的box，我们知道当bvh树对象创建好了之后构造函数就会将整个box构建好），当我们hit到了最大的box之后就会继续往下（如果连最大的box都没有hit到的话后面的也不用继续往下判定了，直接返回false就好）；

往下，开始判定左子树是否有hit到东西：如果左子树是一个叶子节点类型，那么就会调用aabb类的hit函数，如果左子树是一个bvh_node节点，那么就会递归调用bvh_node::hit（递归的box显然会随着调用对象的改变而改变，这里是左子树对应的box，而不是root对应的box）。

递归调用直到出现子叶节点，此时对应节点的box调用的hit是aabb类的，hit到最后的子叶节点之后，就会调用具体物体的hit函数了（比如说sphere::hit或者moving_sphere::hit），并且将最后的hit结果保存起来。再之后就开始返回（true or false）到上一层节点，进行右子树的判定，重复这个过程。

后面就简单了，返回hit到的物体（如果两个物体都hit到（即重叠），那么就取最近的那个），对于一条射线来说，一次hit到的物体最多有一个，因此我们不需要存储所有hit_record，只需要一个就行。这就是为啥没用上list的缘故。

## 总结

多态的应用以及递归函数具体的行为
