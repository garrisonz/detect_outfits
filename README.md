# Detect Outfits

- 基于 yolov5 训练检测模型，AP@.5 达 0.525
- 发现 pycocotools 的一个 bug，已提交 PR

### 训练

- 验证集占比 0.2

- 数据格式
  
  > 原始数据集为 coco 格式，需要和 yolov5 的数据格式，互做转换

### 评估

- 用 cocoeval 脚本评估模型效果，AP 指标
  
  >  cocoeval 有一个 bug。
  > 
  > 我的修复：[fix imgIds mismatch problem by garrisonz · Pull Request #595 · cocodataset/cocoapi · GitHub](https://github.com/cocodataset/cocoapi/pull/595) 
  > 
  > bug 的描述：
  > 
  > 当 img_id $\in$ [int.MAX + 1, uint.MAX] 时，img_ids 作为 numpy.aray 自动转为 float，则
  > 
  > $\to$ 数值精度会丢失，img_id 出现不匹配的情况
  > 
  > $\to$ 大量真实图像的标注数据无法读取
  > 
  > $\to$  AP 计算虚高

- 本地跑 AP 指标
  
  > 预测结果，转换为 coco 格式后，在平台服务器评估模型，无法得到正确的 AP 指标。
  > 
  > 因此，在本地，对验证集，用修复后 cocoeval 跑 AP 指标

- 标注本身不完整，AP 存在误差
  
  > 分析腰带的预测时，发现标注不完整。
  > 
  > 腰带的验证集有 4 张图像
  > 
  > 一个版本的模型，预测到 3 张图片，其中 2 张确实包含腰带。
  > 
  > 但是，这两张图片，并没有标注，导致腰带的 AP 为 0.
  > 
  > 因此，数据集的 AP 评分本身，存在误差。对于小样本的类别，AP 误差明显。

### 训练环境

- colab GPU

- PyTorch

竞赛：[CowBoy Outfits Detection | Kaggle](https://www.kaggle.com/c/cowboyoutfits/)

参考：

- [[Training]cowboy_object_detection_YOLOv5🤠 | Kaggle](https://www.kaggle.com/code/sheepwang/training-cowboy-object-detection-yolov5)

- [[Rank 5th]Challenge with imbalanced training data](https://www.kaggle.com/code/zanghf163com/rank-5th-challenge-with-imbalanced-training-data)

### 坑

- 数据不平衡
  
  > | 类别         | 标注框个数 |
  > | ---------- | ----- |
  > | belt       | 25    |
  > | sunglasses | 2330  |
  > | cowboy_hat | 595   |
  > | boot       | 449   |
  > | jacket     | 2195  |
  > | 总          | 5594  |
  > 
  > 腰带的样本数很少，尝试增加腰带样本的数量，令腰带的样本数期望达到 500 个，但是模型的效果没有提升，反而有所下降。
  > 
  > 无重采样，检测到的腰带正确率 8 / 11
  > 
  > 重采样，检测到的腰带正确率 2 / 3
  > 
  > 可见，无重采样，效果反而更好。
  > 
  > 原因猜测：是因为重新采样，导致每张腰带图像，都复制了 20+ 张，较多的重复图像，导致模型偏差严重，影响效果。再进一步的分析，暂时没有思路。
