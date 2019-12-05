---
layout: post
title: "Daily Paper 39: Synthesizing Obama"
description: "Notes"
categories: [MMML-DeepFake]
tags: [Paper]
redirect_from:
  - /2019/11/25/
---

# Daily Paper 39 - Synethesizing Obama: Learning Lip Sync from Audio  

## Introduction  

今天的这篇paper是华盛顿大学的三位大牛发表在巨牛期刊TOG 2017上的，作者主要分析了奥巴马的嘴型和其所说的话语，用一个RNN学习了嘴型与语言的对应关系，从而能够通过给定的语句生成对应的嘴型。  

作者选奥巴马进行分析有三个原因：首先奥巴马的说话录像很好找，因为他每周都会进行总统讲话；其次他的讲话都是在公开场合的正式讲话，所以能够很好的适应学术研究和发表；最后他的录像相当清晰，并且嘴部的运动占据了整个屏幕的一大块（...），并且他演讲的主题和语调，以及脸部在屏幕的位置几乎是不变的，这更方便了对其的研究。我觉得作者选奥巴马还有一个原因，就是奥巴马比较出名，paper比较好发，不然他怎么不随便选个气象播报员或者新闻主持人呢？人家天天都演讲，奥巴马每星期才演讲一次。  

作者的主要做法是先将音频输入法转化为一个时序变化的稀疏嘴型图，然后基于生成的嘴型图，再生成真实图像下的嘴部图片，并经过时序调整糅合在原始的视频图像中。整个过程在现在看来非常简单和基础，因为17年这篇paper主要是一个开创性的工作。  

作者的创新点在于其RNN模型相比其他的方式（如当时已经发布的Face2Face）速度非常快，但是结果的准确度也很好，总之有效的提升了效率。作者的RNN用的是LSTM，也是之后的Deepfake研究中广泛使用的范式。此外作者的模型还能生成相当细化的结果，包括嘴唇、牙齿的细节和随时间改变的脸颊上的酒窝和嘴部的皱纹。  

## System  

### Audio Feature to Sparce Mouth Shape  

接下来根据上面处理的方式来看作者的模型。首先是从音频到稀疏的嘴型的对应过程。作者使用MFCC系数进行音频特征的提取，主要方式如下：1.使用ffmpeg对初始音频进行基于RMS的归一化。2.以25ms的滑动窗口和10ms的采样间隔对音频进行离散傅里叶变换。3.在傅立叶功率谱上应用40个三角形梅尔标度滤波器，对结果取对数。4.使用离散余弦变换降维到13-D。  

经过上述4步，就可以将初始的音频转换为13-D的音频特征，再加上音量的log mean energy和他们的一阶时间导数，组成了28-D的最终输出。此外为了计算嘴部形状的表示，作者还对奥巴马的脸进行了标注，主要标注嘴上，一共有18个标注点。使用PCA进行降维，将音频和视频的采样率对齐。  

接下来的任务便是学习MFCC音频特征和PCA的嘴部形状特征的对应关系了。这一对应关系需要具有一定记忆功能的网络实现，因为说话是一个动态的过程，嘴巴在说第一个词的时候就已经张开了，之后只是嘴唇形态的不断变化，所以需要网络综合分析之前的状态信息得出最终结论。这里作者使用了LSTM作为具体的RNN网络。有的时候嘴巴在说话之前就长开了，比如某些单词需要张开嘴发声，那么这个时候只考虑之前的音频输入还是不够的，所以作者使用了一个双向LSTM来利用未来的文本内容处理信息。不过这就会出现一个问题，双向LSTM所需要的复杂度太高，而且模型需要的并不是很靠后的信息，而只是稍微delay几秒甚至数十毫秒的信息，所以作者采取了另外一种方法来代替双向LSTM：单纯的将网络输入按照时间线向前平移，也就是将输出后移。在最初的LSTM中，第一个时间步下就会有输出，在这里可能到第二个时间步才会有输出，网络多接收几个时间步的输入信息时才会产生输出的信息，并且每一个时间步的信息都是同步落后的，这显著的提高了模型的表现。  

### Sparce Mouth Shape to Facial Texture Synthesis  

下一步就是从嘴部的稀疏点状形态转变为面部的细化特征。这里作者主要针对的是脸下半部分的一些特征，比如嘴、脸颊、下巴以及鼻子和嘴巴周围的一些区域，而对于其他的一些部位，比如眼睛等都是用的原始视频中的一些背景。作者希望生成的面部满足两个要求：准确真实的面部特征，以及面部特征能够随时间平滑变化。  

作者使用的方法是将每一个给定的嘴部形状单独处理。对于每一个嘴部的PCA形状，都选择特定数量的最满足给定形状的目标视频帧，然后应用加权中位数在这些候选帧上生成一个中值纹理。之后再从目标视频中选择牙齿的代理帧，然后从代理帧中将高频率的牙齿细节转移到媒体纹理中的牙齿区域中，从而完成对于牙齿的模拟。  

### Video Re-timing for Natural Head Motion  

到这里，该模型已经能够很好的通过奥巴马的声音生成其讲话的面部团了，但是这里会出现一个问题，音频和视频可能会导致不对齐，这会使模型生成的视频观感大打折扣。特别是当演讲过程中出现短暂的停顿时，奥巴马的嘴巴可能不动了，但是头或者眼珠还在不停的动来动去，这就看上去非常不自然。为了解决这一问题，作者使用了动态规划的方法去对目标视频重新计时。作者寻找了一个从N个合成的嘴部动态帧和M个目标视频帧的最佳映射。具体的动态规划我实在是没看明白，就不讲了。  

### Composite into Target Video  

最后一步就是合成目标视频。作者已经有了脸的下半部分的细化纹理，也有了能够适应沉默和说话状态的重新计时的目标视频，那么合成这一部分的最重要的任务就是创建一个自然，无伪影的下巴运动，并将下脸纹理融合到下颌骨中，说白了就是把造好的嘴插到脸里。  

为了对应好下颌骨，作者专门建立了一个颌骨矫正算法。具体来讲，先计算目标视频帧和作者的嘴部纹理帧间的光流，这个计算得到的光流被一个颚线制作的mask遮蔽后，用来扭曲嘴巴纹理来适应目标视频帧。听起来非常复杂，其实就是将嘴部的形状变成颌骨的形状，方式就是用颌骨的形状对嘴部的光流做一个遮罩。  

最后就是简单的添加合成了，将目标的脸加上脖子、衬衫和嘴巴，就得到了最终的奥巴马人脸。  

## Experiments  

实验环节主要交代一下训练的细节和最终的表现结果。首先是比较重要的时间复杂度，作者在TitanX上进行了RNN计算，在i7-5820K上进行了其他计算，最终得出对于一个66s长度的源音频，在单核CPU上需要45min来处理，产生一个30fps的视频，这个复杂度稍微有点高，但是对于研究来说还算是可以接受。具体的时间是RNN推理需要5s，稀疏嘴型生成需要66s，嘴部纹理需要3.3分钟，最后的合成和渲染需要11.5分钟，retiming的动态规划需要额外的0.2s，光流的插补需要额外的4s来处理每一个重复帧。实际操作中，作者用一个24核的CPU进行处理，将时间从45min降低到了3min，实现了时间复杂度的大大缩短。  

作者还通过实验测试了delay-LSTM的效果，并寻找了表现最好的delay值和LSTM的node数量，结果发现delay的确能够很好的降低loss值，其中LSTMnode为90且delay为200的时候表现最好。以及进行了一系列测试探究其他具体的超参数，这里就不多做介绍了。  

作者还评价了其合成算法和一些经典的合成算法的性能，结果显示其他的方法模糊度都比较高，而作者的方法得出的结果非常清晰。此外作者还将结果和Face2face算法进行了对比，结果显示虽然作者的输入只有原始的音频，并不需要原始视频而face2face需要，但是作者产生的视频中嘴唇和头部的运动更为真实，更像奥巴马。此外作者的算法还可以捕捉到随时间变化的嘴部皱纹和褶皱，而face2face产生的皮肤更偏向于静态。作者算法产生的牙齿也更为实际，反正就是作者造的哪里都好，特别是作者着重注意的部分，那更是差距巨大。  

不过作者还是承认其算法是存在一些缺陷的，主要表现在四点。1.3D几何错误：作者的光流方法在合成脸颊的时候会出现一些不对齐的状况。2.目标视频长度：作者的算法需要在目标视频中看到所有时刻的嘴部，这会导致能够应用的视频种类和长度有一定的限制。3.表情建模：作者的算法其实并没有显示的对表情建模，也没有对奥巴马所说的内容进行语义分析，所以奥巴马的表情看起来都非常的严肃。当然这里演讲的语境就是严肃场合的公开演讲，所以问题不大，但是要继续泛化应用的话还需要进一步的对这方面的研究。4.舌头建模。这里作者假定嘴部的纹理可以完全由由嘴唇的基准点来决定，但是这并不一定是正确的，因为一些发音比如'th'需要舌头的使用，那么在没有对舌头进行建模的情况下可能很难单纯的通过嘴唇来判别出来。  

## Conclusion  

总结一下，作者使用大量奥巴马的演讲视频进行训练，通过设计好的算法，成功的完成了给定音频制作视频的任务。作者强调，在算法中有一个步骤是手动的选择牙齿的代理，这一步作者认为可以通过牙齿检测器来实现自动化。作者还认为处理音频所使用的MFCC特征并不是专门用来实现视觉对齐的，用新的方法直接用原始的音频来代替MFCC特征，可能用一种更加端对端的方法来实现更好的效果，比如van den Oord等人就使用了扩张卷积来代替RNN和MFCC。此外作者还认为研究网络是否能够从音频中预测情感状态，并产生相应的视频也是有前景的研究方向之一。  

对一个非名人进行训练，可能会因为训练数据的不足而显得格外困难，但是作者认为嘴型和声音的联系可能与说话人具体是谁并无关系，那么对于奥巴马的训练结果可以用一种迁移学习的方式应用到其他人身上，变成一种迁移+小样本学习。作者还认为只分析奥巴马的嘴没啥意思，分析更为广阔的空间，比如整张脸，或者整个身体，再加上他的背景，可能会是一个功能更为强大的模型，能够预测奥巴马将要做什么，将要朝那里走等等。  

总之作者提出了一系列可行的研究方向，给了这一领域的研究学者更为广阔的思路和研究空间。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  