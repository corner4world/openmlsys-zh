## 计算图的设计背景和作用

![基于计算图的架构](../img/ch03/dag.svg)
:width:`800px`
:label:`dag`

早期的机器学习框架主要为了支持基于卷积神经网络的图像分类问题。这些神经网络的拓扑结构简单（神经网络层往往通过串行构建），他们的拓扑结构可以用简单的配置文件来表达（例如Caffe中基于Protocol
Buffer格式的模型定义）。随着机器学习的进一步发展，模型的拓扑日益复杂（包括混合专家，生成对抗网络，多注意力模型）。这些模型的复杂拓扑（例如说，分枝结构，带有条件的if-else结构）。而复杂的拓扑会影响模型算子的执行，自动化计算梯度（一般称为自动微分）和训练参数的自动化判断。为此，我们需要一个更加通用的技术来执行任意机器学习模型。因此，计算图应运而生。综合来看，计算图对于一个机器学习框架提供了以下几个关键作用：

-   **对于输入数据，算子和算子执行顺序的统一表达。**
    机器学习框架用户可以用多种高层次编程语言（Python，Julia和C++）来编写训练程序。这些高层次程序需要统一的表达成框架底层C和C++算子的执行。因此，计算图的第一个核心作用是可以作为一个统一的数据结构来表达用户用不同语言编写的训练程序。这个数据结构可以准确表述用户的输入数据，模型所带有的多个算子，以及算子之间的执行顺序。

-   **定义中间状态和模型状态。**
    在一个用户训练程序中，用户会生成中间变量（神经网络层之间传递的激活值和梯度）来完成复杂的训练过程。而这其中，只有模型参数需要最后持久化，从而为后续的模型推理做准备。通过计算图，机器学习框架可以准确分析出中间状态的生命周期（一个中间变量何时生成，以及何时销毁），从而帮助框架更好的管理内存。

-   **自动化计算梯度。**
    用户给定的训练程序仅仅包含了一个机器学习模型如何将用户输入（一般为训练数据）转化为输出（一般为损失函数）的过程。而为了训练这个模型，机器学习框架需要分析任意机器学习模型和其中的算子，找出自动化计算梯度的方法。计算图的出现让自动化分析模型定义和自动化计算梯度成为可能。

-   **高效程序执行。**
    用户给定的模型程序往往是"串行化"地连接起来多个神经网络层。通过利用计算图来分析模型中算子的执行关系，机器学习框架可以更好地发现将算子进行异步执行的机会，从而以更快的速度完成模型程序的执行。