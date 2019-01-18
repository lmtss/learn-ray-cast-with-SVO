# 仅为学习
人工翻译一篇[论文](https://www.nvidia.com/docs/IO/88889/laine2010i3d_paper.pdf)  
缓慢更新

## 3Voxel Data Structure
## 3体素数据结构
Voxels may be further subdivided into smaller ones, in which case both the parent voxel and its children are included in the octree.
The data structure has been designed to minimize the memory footprint while supporting efficient ray casts.  
这种数据结构被设计在高效光线投射中用来最小化内存占用  
Sometimes both can be achieved at the same time, because more compact data layout also reduces the memory bandwidth requirements.  
有时两者能被同时实现， 因为更紧密的数据布局能减少带宽需求  

。。。

On the highest level, our octree data is divided into blocks.Blocks are contiguous areas of 
memory that store the octree topology along with voxel geometry and shading attributes for 
localized portions of the octree.
在最顶层， 八叉树被被为若干块。块是连续的内存空间，被用来存储八叉树拓扑结构、体素几何体以及八叉树局部的着色属性  
。。。

。。。

。。。

### 3.1Child Descriptors
### 3.1子描述符
We encode the topology of the octree using 64-bit child descriptors, each corresponding to a single non-leaf voxel.  
我们使用64位描述符编码八叉树的拓扑结构， 每个描述符代表一个非叶子体素。  
Leaf voxels do not require a descriptor of their own, as they are described by their parents.  
叶子节点不需要自己的描述符， 他们被自己的父节点描述。  
As illustrated in Figure 2, the child descriptors are divided into two 32-bit parts.  
如图2， 子描述符被分为两个32位部分。
The first part describes the set of child voxels, while the second part is related to contours(Section 3.2).  
第一个部分描述子体素， 另一部分与轮廓有关(3.2节)。

Each voxel is subdivided spatially into 8 child slots of equal size.  
每个体素在空间上被细分为8个相同大小的子空间。  
The child descriptor contains two bitmasks, each storing one bit per child slot.  
子描述符包含了两个位掩码， 每个都按照一个子节点一bit的方式存储。  
valid mask tells whether each of the child slots actually contains a voxel, while leaf mask further specifies whether each of
these voxels is a leaf.  
有/无效掩码(valid mask)表述是否子空间确实有一个体素, 叶子掩码(leaf mask)表述是否体素是叶子。
Based on the bitmasks, the status of a child slot can be interpreted as follows:
* Neither bit is set: the slot is not intersected by a surface.
* The bit in valid mask is set: the slot contains a non-leaf voxel.
* Both bits are set: the slot contains a leaf voxel.
基于位掩码，子空间的状态能被如下表示:
* 两个位都为0：
* 有/无效位为1：子空间有一个非叶子体素
* 两个位都为1：子空间有一个叶子体素

If the voxel contains any non-leaf children, we store an unsigned 15-bit child pointer in order to reference their data.  
如果体素包含任何一个非叶子孩子，我们存储一个无符号15位子指针来指向他们的数据。  
These children, in turn, store their own child descriptors at consecutive memory addresses, and the child pointer points to first one of them as illustrated in Figure 3.  
这些孩子，依次在连续内存地址存储他们自己的子描述符，子指针指向第一个子描述符，如图3所示。  
This way, we can find a particular child by incrementing the pointer based on the bitmasks.  
通过这种方法，我们能找到特定的孩子通过基于位掩码增加指针值  
The children can reside either in the same block or in a different one, making it possible to traverse the octree without having to consideer block boundaries.  
孩子能被存储于同一个块而不是不同的块， 使遍历八叉树时无需考虑块边界  

。。。

。。。

### 3.2Contours
### 3.2轮廓
The most straightforward way to visualize voxel data is to approximate the geometry contained within each voxel as a cube  
最直接的显示体素数据的方法是近似于每个体素一个正方体  
The resulting visual quality is acceptable as long as the data is oversampled, i.e. the projection of each voxel on the screen is smaller than a pixel.  
只要数据过采样，显示质量是可以接受的，也就是说每个体素投射在屏幕上的结果要小于像素。  
However, if voxels are considerably larger due to undersampling, the approximation produces very noticeable artifacts near the silhouettes of objects.  
然而，如果体素由于欠采样而相当的大，近似结果会在物体轮廓附近产生非常显著的artifacts  
。。。
。。。

To reduce the approximation error, we constrain the spatial extent of each voxel by intersecting it with a pair of parallel planes matching the orientation of approximated surface.  
为了减少近似错误，我们通过使用一对与近似表面符合的平行面横截体素来限制体素空间  
We refer to such a pair of planes as a contour  
The result is a collection of oriented slabs that define a tight bounding volume for the surface, as illustrated in the bottom row of Figure 4.  
结果是一堆给表面定义了一个紧密边界体积的定向的片， 如图4  
For flat and relatively smooth surfaces, the planes can be oriented with the average surface normal to obtain a good fit.  
对于平坦或相对光滑的表面， 平面通过平均表面法向来定向以获得更好的结果  
For curved and undersampled surfaces, the planes can still be used to reduce the approximation error, as can be seen in Figure 5.  
对于弯曲和欠采样的表面， 平面仍能被用来减少近似错误， 可以在图5看到  
。。。

We use 32 bits to store the contour corresponding to one voxel.  
我们使用32位来存储一个体素的轮廓  
The value is divided into five components: three 6-bit integers to define the normal vector of the two planes, and two 7-bit integers to define their positions within the voxel.  
值被分为5个部分：三个6位整形来定义两个平面的法向量，两个7位整形来定义在体素中的位置。  

The mapping between voxels and their corresponding contours is established by two fields in the child descriptor(Figure 2).  
体素和他们相对应的轮廓间的映射是建立在子描述符中的两个域里(图 2).  
contour mask is an 8-bit mask telling whether the voxel in each child slot has an associated contour.  
轮廓掩码是一个8位的掩码 描述子空间中的体素有一个相关的轮廓  
Storing a separate bitmask allows omitting contours in voxels where they do not significantly reduce the approximation error.  
存储一个分开的位掩码允许允许省略部分体素的轮廓，这些体素不会显著地减少近似错误  
Similar to the child pointer, the unsigned 24-bit contour pointer reference a list of consecutive contour values, one for each bit in contour mask is set.  
与子指针相似， 这个无符号24位轮廓指针指向一个连续的轮廓值列表， 。。。

### 3.3 Cooperation Between Contours
### 3.3 轮廓之间的协同

While using only one contour per leaf voxel would be enough to represent smooth surfaces, it would also introduce distracting artifacts near sharp edges of objects.  
当仅仅为一个叶子体素使用一个轮廓时能足够的表述顺滑的表面， 而在物体的锐利边缘附近则会产生wtf的渲染不正确。  
This is because the orientation of the surface varies a lot within voxels containing such an edge,  and no single orientation can be chosen as a good representative for all the points on the surface.  
这是因为当体素包含这样一个边时表面的方向变化太多， 没有一个方向能被选为一个好的代表对表面上的所有点来说  

Fortunately, we can utilize the fact that we are storing a full hierarchy of overlapping voxels.  
幸运地， 我们能利用我们存储了一个完全层次化的体素结构这件事  
To enable cooperation between multiple contours, we define the final shape of a voxel as the intersection of its cube with all the contours of its ancestors.  
为了让多个体素协作， 我们定义了一个体素的最终形状作为 ...

### 3.4 Construction of Contours
### 3.4 轮廓的结构

### 3.5 Shading Attributes
### 3.5 着色属性

## 4 Rendering
The regularity of the octree data structure is the key factor in enabling efficient ray casts.  
八叉树数据结构的规律性是高效光线投射起效的关键因素。  
As most of the data associated with a voxel is actually stored within its parent, we need to express the current voxel using its parent voxel parent and a child slot index idx ranging from 0 to 7.  
因为大多数与一个体素相关的数据实际上存储在他的父亲处， 我们需要使用他的父亲体素parent和一个子空间索引idx(0 to 7)表示当前体素。  
Since we do not store information about the spatial location of voxels, we also need to maintain a cube corresponding to the current voxel.  
既然我们没有存储关于体素位置的信息， 我们也需要维护一个与当前体素一致的cube  
The cube is expressed using position vector pos ranging from 0 to 1 in each dimension, and a non-negative integer scale that defines the extent of the cube as exp2(scale − smax).  
这个cube被使用每一维一个0到1的位置向量， 一个表示大小程度的非负整数来描述  
The entire octree is contained within a cube of scale smax positioned at the origin.  
整个八叉树被包含在一个最大尺度的在origin的cube中  
<strong>Basics.</strong>  
<strong>基本的.</strong>  
<strong>Hierarchy traversal.</strong>We will now extend the idea of incremental traversal to a hierarchy of voxels.  
<strong>层次遍历.</strong>我们现在要对一个层次化的体素结构延展增量遍历的想法  
This is necessary since our octree data structure is sparse in the sense that we do not include the subtrees corresponding to empty space.  
这是必要的既然我们的八叉树数据结构在场景中是稀疏的，我们不包括表示空的子树  
Doing the traversal in a hierarchical fashion also has the benefit of being able to improve the performance by using contours as bounding volumes of their corresponding subtrees.  
以层次的风格遍历也有好处，改进以轮廓作为他们对应子树的包围盒的表现。  

Our algorithm traverses the set of voxels intersected by the ray in depth-first order.  
我们的算法深度优先遍历被光线横截的体素集合  
In each iteration, there are three distinct cases for selecting the next voxel:  
在每一次迭代， 有三种限制情况来选择下一个体素：  
∙ PUSH: Proceed to the child voxel that the ray enters first.处理光线第一个进入的子体素  
∙ ADVANCE: Proceed to the next sibling voxel.处理下一个兄弟体素  
∙ POP: Otherwise, proceed to the next sibling of the highest ancestor that the ray exits.否则，处理此光线存在的最远祖先的下一个兄弟  

The algorithm incorporates a stack of parent voxels and contour t values associated with the ancestors of the current voxel.  
算法维护了一个父体素栈。。。  
The depth of the stack is smax, making it possible to address its entries directly using cube scale values.  
这个栈的深度为smax， 使他能直接用cube的比例值来寻址  
Whenever the algorithm descends the hierarchy by executing PUSH, it potentially stores the previous parent into the stack at scale based on a conservative check.  
无论算法何时按照层次下降通过执行PUSH， 它都潜在地存储过去的父亲。。。  
When the ray exits the current parent voxel, the algorithm ascends the hierarchy by executing POP.  
当光线离开当前父体素， 算法上升层次通过执行POP 
It first uses the current position pos to determine the new pos′, scale′, and idx′ as described below. It then reads the stack at scale′ to restore the previous parent.  
它先按照上面描述的方法用当前位置pos决定新的pos′, scale′, idx′。然后读取在scale′处的栈元素来恢复过去的父亲  

