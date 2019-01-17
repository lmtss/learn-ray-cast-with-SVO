# 仅为学习
人工翻译英伟达的一篇[论文](https://www.nvidia.com/docs/IO/88889/laine2010i3d_paper.pdf)

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
