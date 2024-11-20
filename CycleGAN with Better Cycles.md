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

##### Weight cycle consistency by quality of generated image

作者认为，在早期使用循环一致损失意义不大，因为早期训练的时候生成的图像质量非常差。例如，在图7中，两个生成器并没有尝试生成逼真的图像，而是学习了颜色反转映射，从而可以共同降低循环一致性损失。为了解决这个问题，作者建议根据生成图像的质量对循环一致性损失加权，这个质量分数通过判别器的输出获得。将这一改进应用到公式5后，新的循环一致性损失如下：
$$
\begin{equation}
\tilde{L}_{\text{cyc}}(G, F, D_X, X, \gamma) = \mathbb{E}_{x \sim p_{\text{data}}(x)} 
\left[ D_X(x) \left( \gamma \| f_{D_X}(F(G(x))) - f_{D_X}(x) \|_1 
+ (1 - \gamma) \| F(G(x)) - x \|_1 \right) \right]
\end{equation}
$$


##### Full objective

完整公式如下：
$$
\begin{equation}
L(G, F, D_X, D_Y, t) = L_{\text{GAN}}(G, D_Y, X, Y) + L_{\text{GAN}}(F, D_X, Y, X) \\
+ \lambda_t \tilde{L}_{\text{cyc}}(G, F, D_X, X, \gamma_t) 
+ \lambda_t \tilde{L}_{\text{cyc}}(F, G, D_Y, Y, \gamma_t),
\end{equation}
$$

作者声称，使用了新的损失函数以后可以得到更好的结果，会比原来的CycleGAN有更少的artifacts，但是使用判别器结果来作为加权的结果可能不太合适，对结果影响不大，作者认为这可能是因为判别器和生成器是共同训练的，如果能用预训练的判别器，就可以得到更好的结果。（换句话说就是，训练早期，判别器太菜了，得到的结果很难作为加权的依据，如果用了预训练的判别器，效果就会比较好）

#### Future

##### 参数调整

不同的参数组合会产生不同的结果，因为时间原因，所以很难找到更好的组合，作者接下来会继续做这个方向的内容

##### 判别器的预训练和微调

使用预训练的判别器来作为权重的依据，应该可以比较好地提升效果。由于 CycleGAN 使用的是最小二乘 GAN，理论上可以对判别器进行过度训练，而无需担心梯度消失问题。

##### 一对多映射与随机输入

这部分内容主要讨论了一个潜在的改进方向：**如何实现一对多映射（One-to-Many Mapping）**，即通过随机输入扩展生成器的灵活性，使其能够在目标域中生成多样化的图像。这种方法试图解决 CycleGAN 传统方法中**一对一映射的限制**，即相同的输入始终映射到相同的输出。

在图片转换中，一张图应该可以对应多张的图，比方说“夏天到冬天的转换”，一张夏天的图像应该可以对应多张冬天图像（有雪，大雪，少雪等），但是传统的CycleGAN并不能做到这部分的内容。所以作者通过 **添加一个噪声通道，并且鼓励生成器学习这个噪声，来引入随机性**， 但是到论文发表为止，效果不是很好。

##### Generators with latent variables

作者认为 CycleGAN 翻译任务中的两个图像域视为**共享一个潜变量空间（latent space）**，并将每个生成器视为首先映射到潜变量空间，然后再映射到目标域。为将这一想法引入 CycleGAN 框架中，作者选择生成器架构中的某一层输出潜变量，并将两个生成器视为一对“相互编码器/解码器”（mutual encoders/decoders），即在特定图像域和潜变量空间之间的编码和解码分别由两个不同的生成器完成。

##### Single discriminator for both directions

作者认为使用一个优秀的分类器来作为判别器，其实就可以来判断当前的图像属于哪一类，就已经可以得到比较好的结果。