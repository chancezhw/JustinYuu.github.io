---
layout: post
title: "Daily Paper 28: Perfect Match"
description: "Notes"
categories: [MMML-Self_Supervised]
tags: [Paper]
redirect_from:
  - /2019/11/06/
---

# Daily Paper 28 - Perfect Match: Improved Cross-Modal Embeddings For Audio-Visual Synchronisation  

## Introduction  

今天的这篇paper是延世大学和Naver联合发表的，来头就韩国来说还是不小的。这篇paper主要研究了跨模态的自监督视听对齐任务，并将其应用在跨模态检索的实现上。这篇paper的主要贡献有以下三个：1.作者提出了一个新的学习策略，使模型可以通过一个多路对应问题而不是大家都采用的二分对应问题来学习embeddings；2.作者说明了该模型的表现能够远超当今时序对齐任务的表现；3.作者使用自监督方法训练视觉语音识别的embeddings，并且得到了与券监督方式段对端学习的表示相似的性能。  

自监督学习一直是大家研究的重点，其便利性和普适性使得自监督学习的网络可以应用网上海量的数据进行数据处理，不需要人类的标注使得整个网络的训练成本和复杂度大大降低，因此这一领域的前景相当广阔。而自监督学习在跨模态方面也有了广泛的应用，比如昨天看的SyncNet，能够学习人脸图像和对应的音频的联合embeddings，从而应用到说话人检测和唇语识别等方面。但是之前我们看的自监督学习的视听跨模态任务主要是用的二元分类或者对比损失，但是这些方法是否能够应用到跨模态识别和检索等下游任务中还是未知的（其实我感觉效果挺好的），所以作者提出了一个新的跨模态学习的训练策略，网络主要为一个多相匹配任务训练，在使用交叉熵损失函数的同时避免了二元分类。  

这里并没有说二元分类的弱点是什么，只是讲了作者的新方法效果很好，能够学习到联合的跨模态表示，从而很好的应用到下游任务中，并且学到的embeddings在其他任务的表现优于二元分类学到的embeddings的表现。  

## System  

作者将其网络用于音频-视频对应任务，训练之后将其结果和当前表现最好的AVE-Net和SyncNet等模型进行对比，我们首先来看网络架构。  

整个网络训练的流程和SyncNet相同，也是分为两个stream(音频和视频)，音频流的输入是13维的MFCC特征，维数也是13×20，网络基于VGG-M的CNN网络；视频流同样基于VGG-M，只不过conv1适应了5帧的输入，从7×7改成了5×7×7，总体来讲和SyncNet一模一样，只不过最后没有使用对比损失函数。  

作者采用了两个baseline：SyncNet、AVE-Net，SyncNet使用对比损失，而AVE-Net计算两个模态之间的欧氏距离后导入FC层和softmax层，之后使用2路交叉熵函数输出结果。而这里提出的Multi-way classification不仅仅考虑视听对之间的距离，还考虑了序列状数据之间的相关信息。作者给定的学习标准使得网络从视频流里输入一个特征，但是从音频流里输入多个特征，这就类似N路特征匹配任务，计算视频特征和每个音频特征之间的欧氏距离，从而得到N个距离，接着通过softmax层后将距离的倒数导入交叉熵损失函数，使得距离远的分数更少，距离近的分数更高。  


## Experiments  

### Audio-to-video synchronisation  

第一个任务是视听同步的音频选择，给定一系列音频，找出与给定视频对应的音频。具体方法是计算视频特征和一系列音频特征的距离，当特征之间的距离最小的时候我们认定两个流是同步的。此外由于短帧数的时候可能没有明显的信息出现，有可能无法完成对齐任务，所以作者根据帧数的不同分不同的组进行测试，最终将结果平均得到最终的结果。  

Dataset使用LRS2数据集，作者测试了不同的参选音频数N对准确率的影响，结果显示N为40时准确率最高。最后的结果显示作者的网络性能在任何frame数量下都远超过两个Baseline，帧数为15的时候准确率甚至能达到98.1%，在帧数为5的默认条件下准确率也能达到89.1%，远超目前的最佳性能75.8%。  

### Visual speech recognition  

作者使用LRW数据集进行训练，采用top-1准确率作为评价指标，网络架构由前端的视觉网络和后端的分类网络组成，结果显示作者提出的TC-5架构下的预训练模型在无调优的情况下表现显著优于其他架构下的端到端模型和该架构下的SyncNet和AVE-Net，表现和基于TC-5架构的端到端模型性能相当。  

## Conclusion  

总结一下，作者介绍了一个跨模态匹配和查找的新策略，使得网络可以在没有显性标注类的前提下自监督学习，并能够享受交叉熵损失带来的良好学习特征。该模型在两个下游任务视听同步的音频选择和视频演讲识别任务中均取得了很好的表现。作者认为该模型还能应用于更多的任务中，应用前景十分广泛。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  