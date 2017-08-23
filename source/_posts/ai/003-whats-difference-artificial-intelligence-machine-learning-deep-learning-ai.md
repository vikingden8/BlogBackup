---
title: What’s the Difference Between Artificial Intelligence, Machine Learning, and Deep Learning?
date: 2017-08-23 23:36:26
categories: "人工智能"
---

转载自：[What’s the Difference Between Artificial Intelligence, Machine Learning, and Deep Learning?](https://blogs.nvidia.com/blog/2016/07/29/whats-difference-artificial-intelligence-machine-learning-deep-learning-ai/)

_This is the first of a multi-part series explaining the fundamentals of deep learning by long-time tech journalist Michael Copeland_.

这是第一个讲解机器学习基础的多系列文章，来自于拥有多年科技经验的记者 **Michael Copeland**。

Artificial intelligence is the future. Artificial intelligence is science fiction. Artificial intelligence is already part of our everyday lives. All those statements are true, it just depends on what flavor of AI you are referring to.

有人说，人工智能（AI）是未来，人工智能是科幻，人工智能已是我们日常生活中的一部分。这些评价可以说都是正确的，就看你指的是哪一种人工智能。

For example, when Google DeepMind’s AlphaGo program defeated South Korean Master Lee Se-dol in the board game Go earlier this year, the terms AI, machine learning, and deep learning were used in the media to describe how DeepMind won. And all three are part of the reason why AlphaGo trounced Lee Se-Dol. But they are not the same things.

例如，今年早些时候，Google DeepMind的AlphaGo打败了韩国的围棋大师李世乭九段。在媒体描述DeepMind胜利的时候，将人工智能（AI）、机器学习（machine learning）和深度学习（deep learning）都用上了。这三者在AlphaGo击败李世乭的过程中都起了作用，但它们并不是同样的事物。

The easiest way to think of their relationship is to visualize them as concentric circles with AI — the idea that came first — the largest, then machine learning — which blossomed later, and finally deep learning — which is driving today’s AI explosion —  fitting inside both.

今天我们就用最简单的方法——同心圆，可视化地展现出它们三者的关系和应用。人工智能是最早出现的，也是最大、最外侧的同心圆；其次是机器学习，稍晚一点；最内侧，是深度学习，当今人工智能大爆炸的核心驱动。

![](/images/categories/ai/003/01.png)

五十年代，人工智能曾一度被极为看好。之后，人工智能的一些较小的子集发展了起来。先是机器学习，然后是深度学习。深度学习又是机器学习的子集。深度学习造成了前所未有的巨大的影响。

<!--more-->

### From Bust to Boom 从概念的提出到走向繁荣

AI has been part of our imaginations and simmering in research labs since a handful of computer scientists rallied around the term at the Dartmouth Conferences in 1956 and birthed the field of AI. In the decades since, AI has alternately been heralded as the key to our civilization’s brightest future, and tossed on technology’s trash heap as a harebrained notion of over-reaching propellerheads. Frankly, until 2012, it was a bit of both.

1956年，几个计算机科学家相聚在达特茅斯会议（Dartmouth Conferences），提出了“人工智能”的概念。其后，人工智能就一直萦绕于人们的脑海之中，并在科研实验室中慢慢孵化。之后的几十年，人工智能一直在两极反转，或被称作人类文明耀眼未来的预言；或者被当成技术疯子的狂想扔到垃圾堆里。坦白说，直到2012年之前，这两种声音还在同时存在。

Over the past few years AI has exploded, and especially since 2015. Much of that has to do with the wide availability of GPUs that make parallel processing ever faster, cheaper, and more powerful. It also has to do with the simultaneous one-two punch of practically infinite storage and a flood of data of every stripe (that whole Big Data movement) – images, text, transactions, mapping data, you name it.

过去几年，尤其是2015年以来，人工智能开始大爆发。很大一部分是由于GPU的广泛应用，使得并行计算变得更快、更便宜、更强大。当然，无限拓展的存储能力和骤然爆发的数据洪流（大数据）的组合拳，也使得图像数据、文本数据、交易数据、映射数据全面海量爆发。

Let’s walk through how computer scientists have moved from something of a bust — until 2012 — to a boom that has unleashed applications used by hundreds of millions of people every day.

让我们慢慢梳理一下计算机科学家们是如何将人工智能从最早的一点点苗头，发展到能够支撑那些每天被数亿用户使用的应用的。

#### Artificial Intelligence  —  Human Intelligence Exhibited by Machines 人工智能——为机器赋予人的智能

![](/images/categories/ai/003/02.jpg)

King Me : 能下国际跳棋的程序是早期人工智能的一个典型应用，在二十世纪五十年代曾掀起一阵风潮。

Back in that summer of ’56 conference the dream of those AI pioneers was to construct complex machines — enabled by emerging computers — that possessed the same characteristics of human intelligence. This is the concept we think of as “General AI” —  fabulous machines that have all our senses (maybe even more), all our reason, and think just like we do. You’ve seen these machines endlessly in movies as friend —  C-3PO —  and foe —  The Terminator. General AI machines have remained in the movies and science fiction novels for good reason; we can’t pull it off, at least not yet.

早在1956年夏天那次会议，人工智能的先驱们就梦想着用当时刚刚出现的计算机来构造复杂的、拥有与人类智慧同样本质特性的机器。这就是我们现在所说的“强人工智能”（General AI）。这个无所不能的机器，它有着我们所有的感知（甚至比人更多），我们所有的理性，可以像我们一样思考。人们在电影里也总是看到这样的机器：友好的，像星球大战中的C-3PO；邪恶的，如终结者。强人工智能现在还只存在于电影和科幻小说中，原因不难理解，我们还没法实现它们，至少目前还不行。

What we can do falls into the concept of “Narrow AI.” Technologies that are able to perform specific tasks as well as, or better than, we humans can. Examples of narrow AI are things such as image classification on a service like Pinterest and face recognition on Facebook.

我们目前能实现的，一般被称为“弱人工智能”（Narrow AI）。弱人工智能是能够与人一样，甚至比人更好地执行特定任务的技术。例如，Pinterest上的图像分类；或者Facebook的人脸识别。

Those are examples of Narrow AI in practice. These technologies exhibit some facets of human intelligence. But how? Where does that intelligence come from? That get us to the next circle, Machine Learning.

这些是弱人工智能在实践中的例子。这些技术实现的是人类智能的一些具体的局部。但它们是如何实现的？这种智能是从何而来？这就带我们来到同心圆的里面一层，机器学习。

#### Machine Learning —  An Approach to Achieve Artificial Intelligence 机器学习—— 一种实现人工智能的方法

![](/images/categories/ai/003/03.jpg)

Spam free diet: machine learning helps keep your inbox (relatively) free of spam.

Spam free diet: 机器学习能帮助你过滤你邮箱中的垃圾电子邮件。

Machine Learning at its most basic is the practice of using algorithms to parse data, learn from it, and then make a determination or prediction about something in the world. So rather than hand-coding software routines with a specific set of instructions to accomplish a particular task, the machine is “trained” using large amounts of data and algorithms that give it the ability to learn how to perform the task.

机器学习最基本的做法，是使用算法来解析数据、从中学习，然后对未来真实世界中的事件做出决策和预测。与传统的为解决特定任务、硬编码的软件程序不同，机器学习是用大量的数据来“训练”，通过各种算法从数据中学习如何完成任务。

Machine learning came directly from minds of the early AI crowd, and the algorithmic approaches over the years included decision tree learning, inductive logic programming. clustering, reinforcement learning, and Bayesian networks among others. As we know, none achieved the ultimate goal of General AI, and even Narrow AI was mostly out of reach with early machine learning approaches.

机器学习直接来源于早期的人工智能领域。传统算法包括决策树学习、推导逻辑规划、聚类、强化学习和贝叶斯网络等等。众所周知，我们还没有实现强人工智能。早期机器学习方法甚至都无法实现弱人工智能。

As it turned out, one of the very best application areas for machine learning for many years was computer vision, though it still required a great deal of hand-coding to get the job done. People would go in and write hand-coded classifiers like edge detection filters so the program could identify where an object started and stopped; shape detection to determine if it had eight sides; a classifier to recognize the letters “S-T-O-P.” From all those hand-coded classifiers they would develop algorithms to make sense of the image and “learn” to determine whether it was a stop sign.

机器学习最成功的应用领域是计算机视觉，虽然也还是需要大量的手工编码来完成工作。人们需要手工编写分类器、边缘检测滤波器，以便让程序能识别物体从哪里开始，到哪里结束；写形状检测程序来判断检测对象是不是有八条边；写分类器来识别字母“S-T-O-P”。使用以上这些手工编写的分类器，人们总算可以开发算法来感知图像，判断图像是不是一个停止标志牌。

Good, but not mind-bendingly great. Especially on a foggy day when the sign isn’t perfectly visible, or a tree obscures part of it. There’s a reason computer vision and image detection didn’t come close to rivaling humans until very recently, it was too brittle and too prone to error.

这个结果还算不错，但并不是那种能让人为之一振的成功。特别是遇到云雾天，标志牌变得不是那么清晰可见，又或者被树遮挡一部分，算法就难以成功了。这就是为什么前一段时间，计算机视觉的性能一直无法接近到人的能力。它太僵化，太容易受环境条件的干扰。

Time, and the right learning algorithms made all the difference.

随着时间的推进，学习算法的发展改变了一切。

#### Deep Learning — A Technique for Implementing Machine Learning 深度学习——一种实现机器学习的技术

![](/images/categories/ai/003/04.jpg)

Herding cats: Picking images of cats out of YouTube videos was one of the first breakthrough demonstrations of deep learning.

Herding cats：从YoutTube视频里面寻找猫的图片是深度学习的首次重大突破的展现。

Another algorithmic approach from the early machine-learning crowd, Artificial Neural Networks, came and mostly went over the decades. Neural Networks are inspired by our understanding of the biology of our brains – all those interconnections between the neurons. But, unlike a biological brain where any neuron can connect to any other neuron within a certain physical distance, these artificial neural networks have discrete layers, connections, and directions of data propagation.

人工神经网络（Artificial Neural Networks）是早期机器学习中的一个重要的算法，历经数十年风风雨雨。神经网络的原理是受我们大脑的生理结构——互相交叉相连的神经元启发。但与大脑中一个神经元可以连接一定距离内的任意神经元不同，人工神经网络具有离散的层、连接和数据传播的方向。

You might, for example, take an image, chop it up into a bunch of tiles that are inputted into the first layer of the neural network. In the first layer individual neurons, then passes the data to a second layer. The second layer of neurons does its task, and so on, until the final layer and the final output is produced.

例如，我们可以把一幅图像切分成图像块，输入到神经网络的第一层。在第一层的每一个神经元都把数据传递到第二层。第二层的神经元也是完成类似的工作，把数据传递到第三层，以此类推，直到最后一层，然后生成结果。

Each neuron assigns a weighting to its input —  how correct or incorrect it is relative to the task being performed.  The final output is then determined by the total of those weightings. So think of our stop sign example. Attributes of a stop sign image are chopped up and “examined” by the neurons —  its octogonal shape, its fire-engine red color, its distinctive letters, its traffic-sign size, and its motion or lack thereof. The neural network’s task is to conclude whether this is a stop sign or not. It comes up with a “probability vector,” really a highly educated guess,  based on the weighting. In our example the system might be 86% confident the image is a stop sign, 7% confident it’s a speed limit sign, and 5% it’s a kite stuck in a tree ,and so on — and the network architecture then tells the neural network whether it is right or not.

每一个神经元都为它的输入分配权重，这个权重的正确与否与其执行的任务直接相关。最终的输出由这些权重加总来决定。我们仍以停止（Stop）标志牌为例。将一个停止标志牌图像的所有元素都打碎，然后用神经元进行“检查”：八边形的外形、救火车般的红颜色、鲜明突出的字母、交通标志的典型尺寸和静止不动运动特性等等。神经网络的任务就是给出结论，它到底是不是一个停止标志牌。神经网络会根据所有权重，给出一个经过深思熟虑的猜测——“概率向量”。这个例子里，系统可能会给出这样的结果：86%可能是一个停止标志牌；7%的可能是一个限速标志牌；5%的可能是一个风筝挂在树上等等。然后网络结构告知神经网络，它的结论是否正确。

Even this example is getting ahead of itself, because until recently neural networks were all but shunned by the AI research community. They had been around since the earliest days of AI, and had produced very little in the way of “intelligence.” The problem was even the most basic neural networks were very computationally intensive, it just wasn’t a practical approach. Still, a small heretical research group led by Geoffrey Hinton at the University of Toronto kept at it, finally parallelizing the algorithms for supercomputers to run and proving the concept, but it wasn’t until GPUs were deployed in the effort that the promise was realized.

即使是这个例子，也算是比较超前了。直到前不久，神经网络也还是为人工智能圈所淡忘。其实在人工智能出现的早期，神经网络就已经存在了，但神经网络对于“智能”的贡献微乎其微。主要问题是，即使是最基本的神经网络，也需要大量的运算。神经网络算法的运算需求难以得到满足。不过，还是有一些虔诚的研究团队，以多伦多大学的Geoffrey Hinton为代表，坚持研究，实现了以超算为目标的并行算法的运行与概念证明。但也直到GPU得到广泛应用，这些努力才见到成效。

If we go back again to our stop sign example, chances are very good that as the network is getting tuned or “trained” it’s coming up with wrong answers —  a lot. What it needs is training. It needs to see hundreds of thousands, even millions of images, until the weightings of the neuron inputs are tuned so precisely that it gets the answer right practically every time — fog or no fog, sun or rain. It’s at that point that the neural network has taught itself what a stop sign looks like; or your mother’s face in the case of Facebook; or a cat, which is what Andrew Ng did in 2012 at Google.

我们回过头来看这个停止标志识别的例子。神经网络是调制、训练出来的，时不时还是很容易出错的。它最需要的，就是训练。需要成百上千甚至几百万张图像来训练，直到神经元的输入的权值都被调制得十分精确，无论是否有雾，晴天还是雨天，每次都能得到正确的结果。只有这个时候，我们才可以说神经网络成功地自学习到一个停止标志的样子；或者在Facebook的应用里，神经网络自学习了你妈妈的脸；又或者是2012年吴恩达（Andrew Ng）教授在Google实现了神经网络学习到猫的样子等等。

Ng’s breakthrough was to take these neural networks, and essentially make them huge, increase the layers and the neurons, and then run massive amounts of data through the system to train it. In Ng’s case it was images from 10 million YouTube videos. Ng put the “deep” in deep learning, which describes all the layers in these neural networks.

吴教授的突破在于，把这些神经网络从基础上显著地增大了。层数非常多，神经元也非常多，然后给系统输入海量的数据，来训练网络。在吴教授这里，数据是一千万YouTube视频中的图像。吴教授为深度学习（deep learning）加入了“深度”（deep）。这里的“深度”就是说神经网络中众多的层。

Today, image recognition by machines trained via deep learning in some scenarios is better than humans, and that ranges from cats to identifying indicators for cancer in blood and tumors in MRI scans. Google’s AlphaGo learned the game, and trained for its Go match —  it tuned its neural network —  by playing against itself over and over and over.

现在，经过深度学习训练的图像识别，在一些场景中甚至可以比人做得更好：从识别猫，到辨别血液中癌症的早期成分，到识别核磁共振成像中的肿瘤。Google的AlphaGo先是学会了如何下围棋，然后与它自己下棋训练。它训练自己神经网络的方法，就是不断地与自己下棋，反复地下，永不停歇。

#### Thanks to Deep Learning, AI Has a Bright Future 深度学习，给人工智能以璀璨的未来

Deep Learning has enabled many practical applications of Machine Learning and by extension the overall field of AI. Deep Learning breaks down tasks in ways that makes all kinds of machine assists seem possible, even likely. Driverless cars, better preventive healthcare, even better movie recommendations, are all here today or on the horizon. AI is the present and the future. With Deep Learning’s help, AI may even get to that science fiction state we’ve so long imagined. You have a C-3PO, I’ll take it. You can keep your Terminator.

深度学习使得机器学习能够实现众多的应用，并拓展了人工智能的领域范围。深度学习摧枯拉朽般地实现了各种任务，使得似乎所有的机器辅助功能都变为可能。无人驾驶汽车，预防性医疗保健，甚至是更好的电影推荐，都近在眼前，或者即将实现。人工智能就在现在，就在明天。有了深度学习，人工智能甚至可以达到我们畅想的科幻小说一般。你的C-3PO我拿走了，你有你的终结者就好了。
