# 目标检测网络结构
- ## backbone 主干网络
  从输入图像中提取特征信息，一般直接使用其他backbone模型上训练好的模型参数（pre-trained），进行微调即可。（自己训练需要训练很久）  
  常用的backbone模型有：  
  1. 大型网络（针对使用GPU的大型平台）：VGG-16、ResNet50、ResNet101、ResNet+DCN、darknet19、darknet53、CSPNet等
  2. 轻量型网络（针对于低性能的移动端平台）：MobileNet、ShuffleNet
- ## neck  
   由于backbone网络毕竟是从图像分类任务迁移过来的，其提取特征的模式可能不太适合与detection。因此，我们需要对它们做一些处理，以便更好的得到目标的类别（classification）信息和位置（location）信息。
- ## detection head
