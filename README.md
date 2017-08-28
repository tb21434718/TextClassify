# TextClassify
文本分类---从原理到实现

## 1、文本分类简介
  文本分类是指在给定分类体系下，根据文本内容自动确定文本类别的过程。比如：一些新闻网站含有大量的政治、军事、科技、体育等不同类别的文章，如何基于这些文章内容，自动的按题材进行分类。再比如，在京东淘宝等电子商务网站，用户会对交易商品进行评价。商家需要对用户的评价划分为正面和负面，以进行反馈统计。类似的，邮箱垃圾邮件的过滤，垃圾广告的处理等等，都是需要用到文本分类技术处理的场合。
  当下主流的文本分类的方式，是基于机器学习、深度学习的方法。主要以统计理论为基础，进行有监督的学习，通常对已知的训练数据做统计分析，从而获得一些规律，再运用规律对未知的数据做预测。
  文本分类的常见流程，具体过程如下：
#### 1、数据准备：
    要得到一个准确率高、效果好的模型，大量的数据是必须的。只有数据的量达到一定程度，模型的正确率才能达到一个可以接受的水平。数据的量和模型复杂度，以及置信率的关系，可以参见vc维理论。
#### 2、数据清洗：
    海量的数据中，必然有不靠谱的错误的数据，我们可以先对数据进行适当的清洗。
#### 3、特征提取：
    使用jieba对文本进行分词，将文本表示成向量的形式，对于文本分类问题来说，数据的特征可以是：词频特征、TF-IDF特征，由于词袋的规模非常的大，我们可以使用Hashing Trick技巧来进行降维操作。
#### 4、训练模型：
    将提取的特征喂给分类器，比如bayes分类器，训练bayes分类器，最后使用bayes分类器对未知的文本进行分类。
   而当下文本分类任务，主流的方法是基于深度学习的方式，benchmark则为卷积神经网络在文本分类中的应用。基于卷积神经网络的文本分类，最经典的论文就是Kim Yoon的Convolutional Neural Networks for Sentence Classification，该论文的分类模型在诸如情感分类任务中获得了良好的性能，且目前这篇论文称为了文本分类的baseline。
  详细介绍可以参考 :  
  #### text_classify_details.pdf。

## 2、运行环境
  python 2.7.13  + tensorflow 1.1 + genism(可选,在词向量使用word2vec时使用)

## 3、数据集获取
 ### 英文数据集：
  http://www.cs.cornell.edu/people/pabo/movie-review-data/，使用和Kim Yoon论文中也使用的数据集：sentence polarity dataset    v1.0 (includes 5331 positive and 5331 negative processed sentences / snippets.) 
  MR电影评论数据，其中包含10662条评论，正面和负面评论各一半。共包含18765个单词（vocab_size），最长的评论有59个单词，数据集保存在data目录下的rt-  polarity.neg和rt-poliarity.pos文件中。
 ### 中文数据集（sogou新闻）

## 4、数据预处理
  MR电影评论数据，数据集相对来说还是比较的小，在数据处理阶段，首先我们需要适当的对数据集作清洗，以及对数据集进行训练集、验证测试集划分。这里取10%的数据作为验证测试集，用于模型验证和参数调节。其次，将数据中非英文字母数字用re.sub函数空格替换，对于一些标点符号，加入空格，用于后续分词。然后，如果基于one-hot编码方式，我们需要构建词汇索引表，将每个单词映射到 0 ~ 18765 之间（18765是词汇量大小），那么每个句子就变成了一个整数的向量。对于句子长度不一致的问题，我们可以通过padding的方式来解决，即填充标记至最大句子长度。如果基于word2vec方式，我们需要加载word2vec对数据进行处理，这部分后续我会补充提到。此外，由于文本分类，被作为一个有监督学习来对待，所以，训练数据，需要构建为数据+标签的形式，这里需要对数据集按照pos(1), neg(0)的方式对句子评论标签化处理。最后，再利用数据训练CNN模型时，我们通常对数据批量化进行处理，也就是设定batch_size, 批训练的引入最大好处是针对非凸损失函数来做的, 毕竟非凸的情况下，全样本就算工程上算的动，也会卡在局部优上，而批量化处理相当于对全样本的部分抽样，修正梯度上的采样噪声，使其更有可能搜索最优值。
  ### 详细代码见：data_preprocess.py
  ### 详细介绍可以参考 :  text_classify_details.pdf。
  
## 5、模型构建
  CNN用于文本分类最基本的5层架构：包括embedding layer、convolutional layer、max-pooling layer 、fully-connected layer 、softmax-layer。
  第一层网络将词向量嵌入到一个低维的向量中。第二层网络就是利用多个卷积核在前一层网络上进行卷积操作。比如，每次滑动3个，4个或者5个单词。第三层网络是一个max-pool层，从而得到一个长向量，并且添加上 dropout 正则项。第四层是全连接层，最后一层使用softmax函数对进行分类。由于就是对文本进行正负面二分类。
构建深度神经网络，最主要的两部分就是网络结构和网络参数的问题。这里网络结构，我们采用的是基本的CNN网络结构，针对这种网络结构，结合输入数据，我们需要设置一些基本参数，并对这些参数进行初始化。假设我们现在已经构建好了网络，那么应该向这个网络里传入什么参数尼？
### 参数
    sequence_length
    输入句子的长度（max_length = 59）
    num_classes
    输出的类别（正负面二分类）
    vocabulary_size
    词典大小（这里是18765）
    embedding_size
    词向量的维度（这里是128）
    kernel_size
    卷积核大小（纵向宽度，这里是3，4，5）
    num_filters
    卷积核个数（这里是128）
    l2_reg
    正则化强度（这里是0）
     stride
    卷积核的步长，默认为1
    batch_size
    批处理：一个batch中句子的数量
    learning_rate
    学习率：优化算法更新网络权重幅度大小
    dropout_keep_prob
    训练时候为0.5，测试的时候置为1
   ### 具体代码参考：build_model.py
  ### 详细介绍可以参考 :  text_classify_details.pdf。

## 6、训练过程
   既然神经网络模型已经搭建完毕，那么接下来就是传入数据，开始一步步的训练了。这部分除了需要初始化数据处理阶段和模型构建阶段需要定义的参数之外，还需要相应的配置模型的训练参数，主要包括：迭代次数、保存模型的频率、评估模型的频率、优化损失过程的学习率等。训练过程也主要就是围绕上述三方面的参数来展开。
  Train的过程其实就是向模型里喂入数据，启动计算，不断优化损失函数，更新网络参数，直到几乎收敛，得到最佳参数的过程。不过除此之外，在这里我们还要使用批处理进行操作，也就是对数据进行批次迭代处理。
  由于整个训练过程，包括模型的构建，都是基于tensorflow深度学习框架展开的，所以，必须搞清楚tensorflow最基本的概念和用法，这里再回顾下。        Tensorflow是一个图计算编程系统，使用图来表示计算任务。对于图我们自然会想到节点和边，而这里的节点被称之为op(operation的缩写)，一个op获得0个或多个tensor（这里表示数据，是一个类型化的多维数组，），并返回0个或多个tensor。而边在这里就是一种连接，数据在这些边上流动，比如两个op建立了联系，那么边上，就会有数据的传递，也就是tensor的flow。整个过程的状态，需要变量来维护。而整个图需要在被称之为会话（Session）的上下文中执行。每个会话执行一个单一的图。如果你在创建变量和操作时，没有明确地使用一个会话，那么TensorFlow会创建一个当前默认会话。你可以在程序中使用多个图，但是大多数程序都只需要一个图。所以，整体来说tensorflow程序通常被组织成两个阶段（构建阶段和执行阶段），通常在构建阶段创建一个图来表示和训练神经网络（op的执行步骤被描述为一个图），然后在执行阶段反复执行图中的训练op。
 ### 整个训练过程具体流程如下：text_classify_details.pdf。
 ### 具体代码参考train_model.py
 
## 7、测试过程
   在已经训练好模型或有pre-trained模型的情况下，对现有测试集进行测试。启动session，restore训练好的模型, 预测即可。
 
## 8、其他相关实现
      https://github.com/yoonkim/CNN_sentence
      https://github.com/dennybritz/cnn-text-classification-tf
  
## 9、整体详细过程见：
    text_classify_details.pdf。



