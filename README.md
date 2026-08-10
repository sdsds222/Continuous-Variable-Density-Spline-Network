# Continuous Variable-Density Spline Neural Network Architecture

The continuous variable-density spline neural network is a novel architecture that transforms traditional discrete matrix calculations into continuous function processing. Its core mechanism utilizes non-uniform B-spline technology to fit input features into a continuous and smooth curve. The network then calculates a set of trainable spacing weights W to generate a series of specific coordinate points and extracts data at corresponding positions on the continuous curve.
The primary advantage of this design is that the network can freely determine where to extract features densely and where to extract them sparsely. This entire processing pipeline remains completely independent of fixed input data dimensions.
Based on how the spacing weights W are generated, the architecture is divided into the following two implementation routes:
1. Static Spacing Architecture
This route trains the spacing weights W as fixed parameters of the network itself. By learning from massive amounts of data, the network solidifies an optimal sampling coordinate distribution rule suitable for the task. When performing actual inference after model training is complete, this set of sampling coordinates remains absolutely fixed.
Requires smooth activation functions. Since the sampling coordinates remain fixed during the inference phase, the feature extraction process within the current layer is mathematically equivalent to pure linear calculation. To prevent linear degradation after stacking multiple network layers, smooth activation functions such as SiLU or GELU must be added between layers to introduce necessary non-linearity. Meanwhile, ReLU must be avoided because its numerical truncation property would destroy the smoothness and continuity required by the entire network.
Possesses extremely high computational efficiency and stability. Fixed sampling positions mean that the underlying complex mapping matrices can be significantly simplified or even cached in advance before deployment. This makes the model extremely fast in practical applications with very low resource consumption.
2. Dynamic Spacing Architecture
This route abandons fixed sampling coordinates and instead incorporates an auxiliary small fully connected layer within the network layer. This auxiliary layer reads the input features of each round and calculates the exclusive spacing weights W for that sample in real-time based on the actual distribution of the current data, thereby achieving real-time tracking of feature distribution positions.
Completely discards traditional activation functions. Because the sampling coordinates for each feature extraction are dynamically determined by the current input data, this mechanism of using input data to alter the computational path inherently possesses strong non-linear processing capabilities. Therefore, this architecture can completely remove traditional non-linear activation functions, relying directly on the dynamic changes of coordinate positions to complete complex feature extraction.
Requires signal maintenance and initialization protection mechanisms. Without activation functions, the weighted average characteristic of the B-spline interpolation algorithm will cause the feature signals to gradually weaken or even be flattened during deep network transmission. Therefore, basic linear normalization operations must be introduced between layers to re-stretch the variance of the signal and maintain signal strength. Furthermore, to prevent generating extremely incorrect sampling coordinates at the very beginning of training which would cause the model to crash directly, the internal auxiliary layer must execute strict zero initialization. This ensures that the network first performs absolutely uniform sampling in the early stages of training, and then gradually learns dynamic position adjustments based on gradients.


# 连续变密度样条神经网络架构概述
连续变密度样条神经网络是一种将传统的离散矩阵计算转化为连续函数处理的新型架构。它的核心机制是利用非均匀B样条技术，将输入特征拟合成一条连续且平滑的曲线。接着，网络通过计算一组可训练的间距权重W，生成一系列具体的坐标点，并在连续曲线上对应的位置提取数据。
这种设计的核心优势在于，网络可以自由决定在哪些区域密集提取特征，在哪些区域稀疏提取特征，并且整个处理过程不受输入数据固定维度的限制。
根据间距权重W的生成方式，该架构分为以下两种实现路线：

一、 静态间距

这种路线将间距权重W作为网络本身固定的参数进行训练。网络通过学习大量数据，固化出一套最适合该任务的采样坐标分布规律。在模型训练完成后进行实际推理时，这组采样坐标将保持绝对固定。
必须使用平滑激活函数。 由于推理阶段的采样坐标固定不变，特征在当前层内的提取过程在数学上等同于纯线性计算。为了防止多层网络堆叠后发生线性退化，层与层之间必须加入SiLU或GELU等平滑激活函数来引入必需的非线性。同时，必须避免使用ReLU，因为它的数值截断特性会破坏整个网络所要求的平滑与连续性。
具备极高的计算效率与稳定性。 采样位置的固定意味着底层复杂的映射矩阵可以被大幅简化甚至在部署前提前缓存，这使得模型在实际应用中的运行速度极快，且资源消耗极低。

二、 动态间距

这种路线放弃了固定的采样坐标，转而在网络层内部加入一个辅助的小型全连接层。该辅助层会读取每一轮的输入特征，并根据当前数据的实际分布，实时计算出该样本专属的间距权重W，从而实现对特征分布位置的实时追踪。
完全舍弃传统激活函数。 因为每一次提取特征的采样坐标都由当前的输入数据动态决定，这种用输入数据去改变计算路径的机制本身就具备极强的非线性处理能力。因此，该架构可以完全去除传统的非线性激活函数，直接依靠坐标位置的动态变化来完成复杂特征的提取。
需要引入信号维持与初始化保护机制。 在没有激活函数的情况下，B样条插值算法的加权平均特性会导致特征信号在深层网络传递中逐渐减弱甚至被抹平。因此，必须在层间引入基础的线性归一化操作来重新拉伸信号的方差，维持信号强度。此外，为了防止训练刚开始时生成极端错误的采样坐标导致模型直接崩溃，内部的辅助层必须执行严格的零初始化，确保网络在训练初期先进行绝对均匀的采样，随后再基于梯度逐步学习动态位置调整。
