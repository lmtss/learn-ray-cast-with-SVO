# 仅为学习
人工翻译英伟达的一篇[论文](https://www.nvidia.com/docs/IO/88889/laine2010i3d_paper.pdf)  
缓慢更新

## Voxel Data Structure
## 体素数据结构
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

### Child Descriptors
### 子描述符
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
