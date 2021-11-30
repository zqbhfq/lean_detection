# R-CNN 系列
[参考文章](https://blog.csdn.net/lsj944830401/article/details/103448746)

## R-CNN
- 架构：  
![image](https://user-images.githubusercontent.com/82703975/143685732-c4a67068-eca7-4a31-a5a9-430bb8b73228.png)
- 流程：  
1. 通过 selective search 算法在输入原图中找出大约 2000 个框框
2. 对找出的框框进行缩放操作（因为 CNN 网络包含有全连接层，输出的特征图尺寸需要一样（论文中是 227×227），同时在论文中进行缩放之前会将边框扩大 p=16 个像素），
3. 用预训练模型（在 Imagenet 训练的模型）进行特征提取操作（得到 4096 维度的特征），提取的特征保存在磁盘中用于下一步骤。
4. 分类和回归
- selective search
![image](https://user-images.githubusercontent.com/82703975/143686749-e5688e9c-24a6-46e5-841b-8ef1b777cd05.png)  
- NMS非极大值抑制
1. 原理：对于Bounding Box的列表B及其对应的置信度S,采用下面的计算方式.选择具有最大score的检测框M,将其从B集合中移除并加入到最终的检测结果D中.
通常将B中剩余检测框中与M的IoU大于阈值Nt的框从B中移除.重复这个过程,直到B为空.  
2. NMS在计算机视觉领域有着非常重要的应用，如视频目标跟踪、数据挖掘、3D重建、目标识别以及纹理分析等。  
- 优点
4. 使用 selective search 方法大大提高候选区域的筛选速度（对比传统算法）。  
5. 使用预训练参数，解决了在目标检测训练过程中标注数据不足的问题。  
6. 通过线性回归模型对边框进行校准，减少图像中的背景空白，得到更精确的定位  
- 缺点
7. R-CNN 存在冗余计算， 因为 R-CNN 的方法是先生成候选区域，再对区域进行卷积，其中候选区域会有一定程度的重叠，造成后续 CNN 提取特征时出现重复运算  
8. 保存特征需要大的存储空间
9. 对候选框缩放容易造成目标变形

## SPP-NET
- 架构    
![image](https://user-images.githubusercontent.com/82703975/143686468-059e81f4-3180-4398-9ab9-e48972139e8d.png)
- SPP    
1. SPP:空间金字塔池化，操作流程如下。
![image](https://user-images.githubusercontent.com/82703975/143686492-44a24c9e-ce3c-4f03-b118-6010ba75eb7b.png)  
2.SPP 其实就是一系列最大池化操作，那么池化操作就必须有两个参数，窗口的大小 win 和步长 stride。win = a/n 向上取整，stride=a/n 向下取整.a为feature map大小 , n为金字塔层级的 bins 大小。  
- 优点  
1. 计算整幅图像的 the shared feature map，然后根据 object proposal 在 shared feature map上映射到对应的 feature vector  
2. 可以接受更多尺度的输入，其速度也比较 R-CNN 快 24-102 倍
- 缺点
1. 和 R-CNN 一样，训练是多阶段的，速度慢，同时，特征保存对存储要求高

## Fast R-CNN
- 架构  
![image](https://user-images.githubusercontent.com/82703975/143686618-e5543c0d-ea40-475f-aeb2-cdeb01eecca2.png)
- 流程  
1. 对图片输入进行特征提取，得到特征图（整个提取的过程就这一步）
2. 也是根据 selective search 算法提取候选框，然后得到的候选框映射到上面提取的特征图上。（怎么映射？）
3. 由于每个候选框的大小不一样，使用 RoI Pooling操作，得到固定的维度特征，通过两个全连接层，分别使用 softmax 和回归模型进行检测。
- RoI pooling  
![image](https://user-images.githubusercontent.com/82703975/143686708-2541f154-22a1-4759-9d82-f69043d47432.png)  
RoI 的好处是可以得到固定大小的特征，加快了处理的速度  
- 优点  
1. 实现了一次卷积处处可用，类似于积分图的思想，从而大大降低了计算量。  
2. 用 RoI pooling 层替换最后一层的 max pooling 层，同时引入建议框数据，提取相应建议框特征  
3. 同时它的训练和测试不再分多步，不再需要额外的硬盘来存储中间层的特征，梯度也能够通过 RoI Pooling 层直接传播。  
4. Fast R-CNN 还使用 SVD 分解全连接层的参数矩阵，压缩为两个规模小很多的全连接层。  
- 缺点  
没有解决sleative search 造成的计算冗余的问题
## Faster R-CNN  
Faster R-CNN 采用与 Fast R-CNN 相同的设计，只是它用内部深层网络代替了候选区域方法。新的候选区域网络（RPN）在生成 ROI 时效率更高  
- 架构  
![image](https://user-images.githubusercontent.com/82703975/143687039-56b240f9-72bf-48ec-a7d8-1cefe29ae5ce.png)
- 流程  
1. 特征提取：与 Fast R-CNN 相同，Faster R-CNN 把整张图片输入神经网络中，利用 CNN 处理图片得到 feature map；
2. 区域提名：在上一步骤得到的 feature map 上学习 proposal 的提取；
3. 分类与回归：对每个 Anchor Box 对应的区域进行二分类，判断这个区域内是否有物体，然后对候选框位置和大小进行微调，分类。
- RPN候选区域网络      
1. 以一张任意大小的图片作为输入，输出一批候选区域，每一个区域都会对应目标的分数和位置信息。实际上就是在最终的卷积特征层上，在每个点利用滑窗生成 k 个不同的矩形框来提取区域，k 一般取为 9。
2. K 个不同的矩形框被称为 anchor，具有不同尺度和比例。用分类器来判断 anchor 覆盖的图像是前景还是背景。对于每一个 anchor，还需要使用一个回归模型来回归框的精细位置。 
3. **RPN结构**  
![image](https://user-images.githubusercontent.com/82703975/144000784-99b9fe3a-ff4a-4380-8fc4-e4cbcda936e5.png)
 
- 优点    
4. 与 selective search 方法相比，RPN 网络将候选区域的选择从图像中移到了 feature map 中，因为 feature map 的大小远远小于原始的图像，此时的滑动窗口的计算量呈数量级的降低。  
5. RPN 和 RoI Pooling 还共用了基础的网络，更是大大地减少了参数量和预测时间。  
6. 由于是在特征空间进行候选框生成，可以学到更加高层语义的抽象特征，生成的候选区域的可靠程度也得到了大大提高。  





