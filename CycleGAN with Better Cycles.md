**CycleGAN with Better Cycles**

### 问题：

1. 这篇论文主要指出CycleGAN有什么问题？
2. 这篇论文提出了什么方法来解决这些对应的问题？
3. 这篇论文题目中所说的“Better Cycles”是什么？

### 笔记：

#### Effects of cycle consistency

##### Guide training

在指导模型训练方面一致性损失对颜色方面更加敏感,只需要几个epoch就可以把颜色方面的信息学习的很真实。

![img](https://cdn.nlark.com/yuque/0/2024/png/22747598/1732003942344-641a5f6a-4945-455b-adc8-29b8bf83718b.png)

##### Regularize

作者认为循环一致损失可以减少生成器产生excessive hallucinations and mode collapse， 这两者都会导致不必要的信息丢失并增加循环一致性损失。  

##### Unrealistic artifacts

原文“It assumes a one-to-one mapping between the two image domains and no information loss during translation even when loss is necessary.”

论文中提到，在图像转换的过程中必不可少地会有很多信息丢失，但是循环一致性损失会强制它们不丢失，这就导致了一些比较尴尬的情况，比如说从斑马转换成马的时候应该把条纹隐去，但是循环一致损失会强制模型不丢失任何信息，所以它的条纹还会以一种比较隐晦的形式存在，包括简笔画和油画的转换中也有这种情况。

##### 问题总结

循环一致损失是假定图片转换过程中不会丢失任何信息，但是实际上这是不可能的，这种损失限制了生成器的灵活性（其实太灵活也不行，需要有一个定向生成的方向），同时，因为强制保存了一些不必要的信息，会导致虚假信息的生成（我们的假阳性的一部分原因可能是这个）

#### 论文的解决方法

##### Cycle consistency on discriminator CNN feature level

**翻译过程中几乎总会有信息丢失**。与其要求CycleGAN恢复原始图像的精确像素，不如仅要求其恢复总体结构。例如，在“斑马到马”的转换中，只要重建的斑马图像具有逼真的斑马条纹（无论是水平还是垂直的，是否与原始一致），就可以认为是循环一致的。  

 为了实现这种较弱的循环一致性，作者在相应判别器的CNN特征层上引入了L1损失。具体来说，就是给像素级的循环一致损失加一个权重来削弱，然后使用判别器来对图像进行特征提取，并进行![img](https://cdn.nlark.com/yuque/__latex/8774782e75650651e5d3a5ded5c77e44.svg)损失计算，计算公式如下所示：

![img](https://cdn.nlark.com/yuque/__latex/121e9176f582f0f69bc2036ec46fd678.svg)

![img](https://cdn.nlark.com/yuque/__latex/22131a91f772703b7b3adfcbcacc4dfd.svg) 是判别器的最后一层特征提取器。

![img](https://cdn.nlark.com/yuque/__latex/108a7b89c1a28ca92a6ecf5ebb5cc93b.svg) 表示CNN特征级和像素级损失之间的比率。

作者认为![img](https://cdn.nlark.com/yuque/__latex/72487d7ab2173aef1c0defd2c5fce685.svg)应该随着轮数的增大而增大，因为前期的判别器并不可靠，并且认为最大不能等于1，因为这会导致引入过多的虚假信息。

##### Cycle consistency weight decay

把循环一致损失的权重随着训练轮数逐渐下降（指的是整个的循环一致损失，包括像素级和特征级的）但是不会到0。



### 解答：