# Learning Spatiotemporal Features with 3D Convolutional Networks
* ？: iDT, HOG, etc. , Flow, semantic, spatial stream,temporal stream, LRCN, LSTM, We extract C3D features: prob, fc7, fc6, pool5 for each clip, ROC, AUC, RBF
-------------------------------------------------------------------------------------------------------------------------------------
# 摘要
    我们提出一种简单，有效的方法，使用深度3D ConvNets训练大规模监督视频数据集的时空特征学习。 我们的发现：1）3D ConvNets比2D ConvNets更适合时空特征学习; 2）3×3×3卷积核的3D ConvNets表现最佳; 3）我们的学习特征，即C3D，结合简单的线性分类器在4个不同的基准上胜过最先进的方法，并且与其他2个基准的当前最佳方法相当。 
#    1. 介绍
    有效的video descriptor有四个属性：（i）通用，可代表不同类型的视频，同时有区别。 例如，互联网视频可以是风景，自然场景，运动，电视节目，电影，宠物，食物等; （ii）紧凑：我们面对数百万视频，紧凑的描述符帮助processing, storing, and retrieving任务更加可扩展; （iii）efficient to compute，每分钟处理成千上万的视频; （iv）易于实现，用简单模型（例如线性分类器，代替使用复杂的特征编码方法和分类器）。
    3D卷积深度网络能同时对 appearance and motion进行良好特征学习建模。
#    2. 相关工作
#    3. Learning Features with 3D ConvNets
##    3.1 3D convolution and pooling
    3D ConvNet对时空特征学习较好。 相比2D ConvNet，由于3D convolution和3D pooling，3D ConvNet能够更好地对时间建模。 在3D ConvNets中，卷积和池操作在空间上执行，而在2D ConvNets中，它们仅在空间上完成。 下图说明了差异，应用于图像的2D卷积将输出图像，应用于多个图像（多通道）的2D卷积也产生图像。 因此，2D ConvNets每次卷积后立即丢失输入信号的时间信息。 只有3D卷积保留输入信号的时间信息。


    我们首先使用UCF101（中等规模的dataset）实验，寻找最佳的架构。 根据2D ConvNet中的发现，3×3卷积核的较小感受野与更深的架构产生最佳结果。 因此，我们将receptive field 固定为3×3，仅改变3D卷积核的temporal depth。
    符号：尺寸c×l×h×w的视频clips，c通道数，l是帧长，h和w是帧高和帧宽。 3D convolution and pooling kernel大小d×k×k，d时间深度，k是空间大小。
    Common network settings: 网络设置为以video clips为输入并预测属于101个不同动作class label。 所有视频frame大小128×171。这大约是UCF101帧的一半。 视频被分割成非重叠的16帧clip，然后用作网络的输入。 输入尺寸为3×16×128×171。We also use jittering by using random crops with a size of 3 × 16 × 112 × 112 of the input clips during training。 网络具有5个convolution层和5个pooling层，2个全连接层和softmax loss层以预测动作label。 5个卷积层滤波器数量分别为64,128,256,256,256。 所有卷积核都有时间深度d（我们将改变这些层的d以搜索最好的3D架构）。所有卷积层被padding（空间和时间），stride是1，因此从这些convolution layers的输入到输出的size不变。 所有pooling层是具有步长1的kernel size为2×2×2（除了第一层）的max pooling，这表示输出信号大小减小了8倍。 第一pooling layerkernel size是1×2×2，目的是不过早地merge时间信号，并且还满足16帧的剪辑长度（例如，我们可以在完全折叠时间信号之前在时间上与因子2合并至多4次）。 两个全连接层有2048个输出。 我们使用30个剪辑的小批量train the networks from scratch，initial learning rate为0.003。 每4个epochs后，learning rate除以10。 训练在16个epochs后停止。
    Varying network architectures: 如何通过深度网络聚合时间信息。为寻找一个好3D ConvNet架构，只改变卷积层的时间深度di，同时保持所有其他如上所述通用设置固定。我们实验两种类型架构: 1） homogeneous temporal depth：所有卷积层具有相同的时间深度; 2) varying temporal depth：时间深度在层间变化。对于homogeneous设置，我们对d等于1,3,5,7的时间深度的4个网络进行实验。我们将这些网络命名为depth-d。注意，depth-1等效于在单帧上2D卷积。对于varying temporal depth设置，我们实验两个网络，时间depth分别为3-3-5-5-7和7-5-5-3-3。我们注意到所有这些网络在最后pooling层有相同大小的输出信号，因此全连接层中他们有相同数量的参数。由于不同kernel的时间depth，他们的参数数量仅在卷积层不同。与全连接的层中的数百万个参数相比，这些差异相当微小。因此，网络的学习能力是可比的，参数数量的差异不应当影响我们的架构搜索的结果。
##    3.2 Exploring kernel temporal depth
    我们在the train split 1 of UCF101上训练这些网络。下图给出了其在不同结构的clip accuracy。左图显示了具有homogeneous temporal depth的网络结果，右图显示了changing kernel temporal depth的网络结果。depth-3在同类网络中表现最好。注意depth-1明显比我们认为是由于缺乏运动模型的其他网更差。与varying temporal depth网络相比，depth-3是最好的执行者，but the gap is smaller。我们还尝试更大的spatial receptive field（例如5×5）和/或全输入分辨率（240×320帧输入），并且仍然观察到类似的行为。这表明3×3×3是3D ConvNets（根据我们的实验子集）的最佳kernel选择，3D ConvNets始终优于2D ConvNets视频分类。我们还验证了3D ConvNet在大型内部数据集（即I380K）上始终比2D ConvNet性能更好。


##    3.3 Spatiotemporal feature learning
    Network architecture：上节表明，卷积kernel为3×3×3的homogeneous设置是3D ConvNets的最佳选择。这个发现也与2D ConvNets中的类似发现一致。我们设计的3D ConvNet有8个卷积层，5个pooling层，2个全连接层和1个softmax输出层。网络架构如下图所示。称之为C3D网络。所有3D卷积filter的size是3×3×3，其stride为1×1×1。所有3D pooling kernel size 为2×2×2，其stride是2×2×2，不包括pool1，pool1的kernel size是1×2×2，stride是1×2×2，其目的是保持早期的时间信息。每个全连接层有4096个输出单元。

    Dataset：为学习spatiotemproal features，我们在Sports-1M数据集上训练C3D，这是目前最大的视频分类benchmark。 数据集包括110万个运动视频。 每个视频属于487个运动categories之一。 与UCF101相比，Sports-1M拥有5倍的类别数量和100倍的视频数量。
    Training：训练是在Sports-1M train split上进行的。 Sports-1M有许多长视频，我们从每个训练视频中随机提取5个2秒长的clips。 clips的frame size调整为128×171。训练时，我们随机地crop输入的clips成为16×112×112的crops for spatial and temporal jittering。 我们也horizontally  flip them with 50％的概率。 Training由SGD完成with minibatch size of 30 examples。 Initial lr为0.003，每150K次迭代除以2。 在1.9M迭代（约13 epochs）后停止优化。 除了train from scratch的C3D网络外，我们还使用在I380K上pre-trained的模型对C3D网络进行了fine-tuning。

    Sports-1M分类结果：下表显示了C3D与DeepVideo和Convolution pooling相比结果。我们对每个clip只使用center crop，并将其传递给网络进行clip prediction。我们平均从视频中随机提取的10个clips的clip predictions。比较方法之间存在一些设置差异。 DeepVideo和C3D使用短clips，而Convolution pooling使用更长的clips。 DeepVideo使用更多crops：每个clip有4个crops，每个video有80个crops，而C3D分别是1和10。Trained from scratch的C3D产生84.4％的accuracy，而从I380K pre-trained模型fine-tuned的C3D有85.5％的精度at video top-5 accuracy。两个C3D的性能优于DeepVideo。 C3D仍然比文献[29]的方法低5.6％。然而，该方法使用convolution pooling of deep image features on long clips of 120 frames，因此不能直接与在much shorter clips上操作的C3D和DeepVideo相比。在实践中，convolution pooling or more sophisticated aggregation schemes可以应用在C3D特征之上，以改善视频命中表现。

    C3D video descriptor：训练后，C3D可用于特征提取。为了提取C3D特征，将视频分成16 frame的clips，两个连续clips有8 frame重叠。 这些clips传到C3D网络提取fc6 activations。 将这些clips的fc6 activations平均以形成4096-dim的video descriptor，然后进行L2归一化。 我们将这种表示称为C3D video descriptor/feature。
   What does C3D learn?：使用反卷积理解C3D内部学习。 C3D starts by关注前几帧中的appearance并跟踪后续帧中的显着运动。 下图显示了具有最高激活的两个C3D conv5b特征图deconv投影回图像空间。 在第一示例中，特征集中于整个人，然后在其余帧跟踪撑杆跳的动作。 类似地，在第二示例中，它首先聚焦于眼睛，然后跟踪在施加化妆时在眼睛周围发生的运动。 因此，C3D与标准2D ConvNets的不同之处在于它选择性地关注motion and appearance。

 
#    4. Action recognition
    Dataset：我们在UCF101数据集上评价C3D特征。 数据集包括101个人类行为类别的13320个视频。 我们使用此数据集提供的三个split setting。
    分类模型：我们提取C3D特征并将其输入到用于训练模型的多类线性SVM。 我们在3个不同的网络使用C3D descriptor进行实验：C3D trained on I380K, C3D trained on Sports-1M, C3D trained on I380K and fine-tuned on Sports-1M。 在网设置中，我们使用L2标准化C3D descriptors。
    Baselines：我们比较C3D特征和几个baselines，当前最好的hand-crafted features即iDT，和广泛使用的deep image features即Imagenet，使用Caffe的Imagenet预训练模型。 对于iDT，we use the bag-of-word representation with a codebook size of 5000 for each feature channel of iDT which are trajectories, HOG, HOF, MBHx, and MBHy。 我们使用L1-norm对每个通道的histogram进行normalize，并且连接这些normalized histograms以形成用于视频的25K特征向量。 对于Imagenet baseline，类似于C3D，我们为每个frame提取Imagenet fc6特征，平均这些帧特征来make video descriptor。 多类线性SVM用于这两个baseline以进行公平比较。

    Results：下表显示了C3D与两个baseline和当前最佳方法相比的动作识别精度。上部显示了两个基线的结果。中间部分给出了仅使用RGB帧作为输入的方法。下部显示使用所有可能的特征组合（如光流，iDT）的当前最佳方法。
    在前面描述的三个C3D网络中，C3D fine-tuned net表现最好。然而，这三个net之间的性能gap很小（1％）。从现在起，我们将C3D fine-tuned net称为C3D。C3D使用一个仅有4096-dim的网络获得了82.3％的accuracy。有3个网络的C3D将精度提高到85.2％，尺寸增加到12,288。当与iDT组合时，C3D进一步将精度提高到90.4％，而当与Imagenet组合时，我们仅观察到0.6％的改进。这表示C3D可以很好地捕appearance and motion信息，因此与appearance based deep feature的Imagenet组合没有益处。 另一方面，将C3D与iDT组合是有利的，因为它们彼此高度互补。 事实上，iDT是hand-crafted features which are based on optical flow tracking and histograms of low-level gradients，而C3D captures high level abstract/semantic information。

    具有3个net的C3D达到85.2％，分别比iDT和Imagenet baseline好9％和16.4％。在仅有的RGB输入设置上，与基于CNN的方法相比，我们的C3D分别优于deep networks和spatial stream network 19.8％和12.6％。deep networks和spatial stream network都使用AlexNet架构。虽然deep networks网络是从他们的模型预训练的Sports-1M中fine-tuned，spatial stream network从Imagenet预训练模型fine-tuned。在网络架构和基本操作方面，我们的C3D不同于这些基于CNN的方法。此外，C3D是在Sports-1M上训练的，并且在没有任何细微调整的情况下使用。与基于循RNN的方法相比，C3D优于LRCN和LSTM复合模型分别为14.1％和9.4％。 只有RGB输入情况下，C3D仍然优于这两种基于RNN的方法，当他们使用光流和RGB以及时间流网络。
    然而，C3D需要与iDT相结合以优于two-stream网络，其他基于iDT的方法[31,25]以及侧重于长期建模的方法[29]。 除了promising numbers，与其他方法相比，C3D还具有简单的优势。

    C3D is compact：为了评估C3D特征的紧凑性，我们使用PCA将特征投影到较低的维度，并使用线性SVM在UCF101上调查投影特征的分类精度。 我们使用iDT以及Imagenet特征应用相同的过程，并比较如下图中。在仅有10个维度的极限设置下，C3D精度为52.8％，在50和100维时，C3D获得72.6％和75.6％的精度，在使用500维时，C3D能够达到79.4％的精度。 这表明我们的特征既compact又discriminative。 这对于低存储成本和快速检索的大规模检索应用非常有用。我们定性评估我们学习的C3D功能，以验证它是否是视频的一个好的通用功能，通过可视化学习的功能嵌入在另一个数据集。 我们从UCF101随机选择100K的clips，然后对这些clips从Imagenet和C3D中分别提取fc6 features。 然后使用t-SNE将其投影到二维空间。 如下图。 值得注意的是，我们没有做任何finetuning，因为我们想要验证这些features是否在数据集之间显示出良好的泛化能力。 我们定量地观察到C3D优于Imagenet。

#    5. Action Similarity Labeling
    Dataset：ASLAN数据集包含432个动作类的3,631个视频。 任务是预测给定的一对视频是否属于相同或不同的动作。 我们使用10倍规定的交叉验证with the splits provided with the dataset。 这不同于动作识别，因为任务集中于预测动作相似性而不是实际动作标签，是相当具有挑战性，因为测试集包含“从未见过”动作的视频。
    Features：我们将视频分成16-frame clips with an overlap of 8 frames。 我们提取每个clip的C3D特征：prob，fc7，fc6，pool5。The features for videos are computed by averaging the clip features separately for each type of feature, followed by an L2 normalization。
    Classification model：我们遵循在[21]中使用的相同设置。 给定一对视频，我们计算[21]中提供的12种不同的距离。 有4种类型的特征，我们获得每个video pair的48维（12×4 = 48）特征向量。 由于这48个距离彼此不可比，我们独立地对它们进行归一化，使得每个维度具有zero mean and unit variance。 最后，线性SVM被训练用以在这些48-dim特征向量上将video pairs分类为相同或不同。Beside comparing with current methods, we also compare C3D with a strong baseline using deep image-based features. The baseline has the same setting as our C3D and we replace C3D features with Imagenet features。
    Results：我们的方法使用对视频的C3D特征的简单平均和线性SVM。 







#    6. Scene and Object Recognition
    Datasets：我们在两个benchmark上评价C3D：YUPENN和Maryland。YUPENN consists of 420 videos of 14 scene categories and Maryland has 130 videos of 13 scene categories。
    Classification model：对于这两个数据集，我们使用相同的特征提取和线性SVM设置进行分类，并遵循与这些数据集作者描述的leave-one-out evaluation协议。 对于object dataset，标准评估基于帧。 然而，C3D采用长度为16帧的clip来提取特征。 我们在所有视频上slide一个16 frames的window，以提取C3D featues。 We choose the ground truth label for each clip to be the most frequently occurring label of the clip。 如果剪辑中最频繁的标签出现少于8帧，我们认为它是没有object的clip，并在训练和测试中将其丢弃。 我们使用线性SVM训练和测试C3D特征，并报告对象识别精度。We follow the same split provided in [32]. We also compare C3D with a baseline using Imagenet feature on these 3 benchmarks.
    Results：// TODO