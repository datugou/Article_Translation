# 创建合成图像数据集--“What”、“Why”、“How”
#### 训练模型的图像不足？教你如何使用图像合成技术将数据集大小扩充数倍

_作者：[**Viraf**](https://medium.com/@virafpatrawala)_  
_原文链接：<https://towardsdatascience.com/create-a-synthetic-image-dataset-the-what-the-why-and-the-how-f820e6b6f718>_  
_2020 年 5 月 12 日_  
`Data Science` `标签2`

---
<div align=center><img src="https://miro.medium.com/max/2400/1*8whwkDAHXUVV9ueRScBYkA.jpeg"></div>
<div align=center><h6>把你的狗送上月球！图像合成技术的示例</h6></div>

众所周知，用于训练模型的数据集的质量和数量将对模型的性能产生直接影响。
一个广泛的，通用的数据集能够帮助你快速达成目标，你可以跳到机器学习/深度学习流水线的下一步。
但往往你可能最终会遇到这样的情况：你所需要的数据集并不容易获得。
或者你所拥有的数据不足以训练一个大型的模型。
现在该怎么办呢？

> 有这么多资源可以帮助你创建一个新的自定义数据集，你需要的任何对象类。
但如果我必须用一个词来描述建立数据集的过程，我会称它为“繁琐”、“耗时”和“令人沮丧”，而且----我知道我用了不止一个词，但你明白我的意思！

尤其是在画分割蒙版的时候，我实在是没办法再多注释几张图片了！
别误会，我知道数据集的创建是非常非常重要的一步，但是难道就没有别办法了吗？

*数据增强*是一个很好的方法，可以增加你的模型用到的数据集的多样性，但是同样，它的帮助也是有限的。
我们想再进一步该如何做呢？

## 什么是合成数据集？
“人工”或“程序化”地创建任何一种数据（图像、音频、文本等）都会产生我们所说的合成数据集。
这些数据不是自然或正常收集的，也不是人工标注的，那么用它来训练你的模型安全吗？
它能给你带来好的结果吗？

虽然有很多[论文](http://news.mit.edu/2017/artificial-data-give-same-results-as-real-data-0303)声称，精心创建的合成数据可以提供与自然数据相当的性能，但我建议两者适当地混合使用。
一个精心设计的合成数据集可以将数据增强的概念提升到一个新的水平，并为模型提供更多种类的训练数据。
越多越好吧？

有许多种方法可以创建合成图像数据集，在本教程中，我们来看一个最基本的方法----图像合成。
我们将通过在多个背景图像上合成前景对象来生成新的图像。

在本教程中，我将为输出类别“狗”创建合成图像。
现实中，网上已有很多包含狗类的数据集（比如 [COCO](https://towardsdatascience.com/master-the-coco-dataset-for-semantic-image-segmentation-part-1-of-2-732712631047)），但多创建一些这些大自然可爱礼物的图片又何妨呢。:P

## 1. import
看一下下面的 import 语句。安装还没有的模块。

``` python
import os
import numpy as np
import skimage.io as io
import skimage.transform as transform
import matplotlib.pyplot as plt
from matplotlib.patches import Rectangle
%matplotlib inline
```

## 2. 主体对象----前景
有两种解决方法----
### （A）创建自己的数据集
首先，让我们从互联网上找一些狗的照片。
有几个网站可以免费使用图像，例如 [pexels](https://www.pexels.com/)，[flickr](http://flickr.com/)，[unsplash](http://unsplash.com/) 等。
我将使用下面这个。

<div align=center><img src="https://miro.medium.com/max/640/1*b9pny7Mp3aTkNKrqI9X62Q.jpeg"></div>
<div align=center><h6>Photo by Matheus Bertelli from Pexels</h6></div>

接下来，您将需要花些力气才能将狗与背景分开。
是的，很抱歉----没有免费的午餐。
但是我保证，这是你最后需要费力气的工作，这肯定比创建整个数据集要干的活少。

可以借助这些工具完成这项工作： [Photoshop](https://www.adobe.com/in/products/photoshop.html?promoid=PC1PQQ5T&mv=other)（付费）或 [GIMP](https://www.gimp.org/)（免费）。
或者，你甚至可以使用永远可靠的 Microsoft Paint！
最终输出如下所示。

<div align=center><img src="https://miro.medium.com/max/640/1*BudWTv3NIgIMhB2pKO0cwQ.jpeg"></div>
<div align=center><h6>狗的分割图像</h6></div>

读取图像。
接下来，从该图像中剪切出前景层，并将其余像素设置为零。
编码此步骤的另一种方法是在分割图像时，用相应的工具（GIMP / Paint / Photoshop）添加黑色背景。
虽然你可以用绘图工具做，但编码的方式更加省事，这是在 python 中提取前景对象的方法----简单的像素阈值处理。

``` python
# Read the image
I = io.imread('./dogSeg.jpg')/255.0
# Cut out the foreground layer
foreground = I.copy()
foreground[foreground>=0.9]=0 # Setting surrounding pixels to zero
plt.axis('off')
plt.imshow(foreground)
plt.show()
```

<div align=center><img src="https://miro.medium.com/max/345/1*x6HHturKJ0hxorHaGBHjFA.png"></div>
<div align=center><h6>周边像素设置为零，前景层保持不变。</h6></div>

### (B) 使用现有数据集
如果你不想从头开始，另一个选择是利用已有数据集的图像。

我在这里将使用 COCO 数据集来演示。
由于 COCO 已经有了对象类别“狗”，所以很容易检索到图像及其蒙版。
COCO 数据集有 4385 张（train）和 177 张（val）图像，用于“狗”类别。

你可以在我的 [GitHub 仓库](https://github.com/virafpatrawala/Synthetic-Image-Datasets)中参考本教程的整个过程的代码。
我不会在这篇文章中赘述这一步的代码。

如果你对操作 COCO 图像数据集和创建一个数据生成器来训练你的图像分割模型感兴趣，我这里有一个[通用的教程](https://towardsdatascience.com/master-the-coco-dataset-for-semantic-image-segmentation-part-1-of-2-732712631047)。

## 3. 增强前景
让我们跟这只“狗”玩一下吧。
我在代码中使用了随机旋转、随机缩放、随机转换和随机水平翻转，当然你也可以用其他东西。

``` python
### Apply augmentations on the foreground.

def foregroundAug(foreground):
    # Random rotation, zoom, translation
    angle = np.random.randint(-10,10)*(np.pi/180.0) # Convert to radians
    zoom = np.random.random()*0.4 + 0.8 # Zoom in range [0.8,1.2)
    t_x = np.random.randint(0, int(foreground.shape[1]/3))
    t_y = np.random.randint(0, int(foreground.shape[0]/3))

    tform = transform.AffineTransform(scale=(zoom,zoom),
                                rotation=angle,
                                translation=(t_x, t_y))
    foreground = transform.warp(foreground, tform.inverse)

    # Random horizontal flip with 0.5 probability
    if(np.random.randint(0,100)>=50):
        foreground = foreground[:, ::-1]
        
    return foreground

foreground_new = foregroundAug(foreground)
# Visualize the foreground
plt.imshow(foreground_new)
plt.axis('off')
plt.show()
```

代码的输出如下图所示。
看看它和原图有什么不同吧!

<div align=center><img src="https://miro.medium.com/max/345/1*A4Xf8zG1DrfhlHg9_Pn34g.png"></div>
<div align=center><h6>增强前景图像</h6></div>

在对前景应用所有这些变化之后，让我们从图像中提取分割蒙版。

``` python
# Create a mask for this new foreground object
def getForegroundMask(foreground):
    mask_new = foreground.copy()[:,:,0]
    mask_new[mask_new>0] = 1
    return mask_new

mask_new = getForegroundMask(foreground_new)
plt.imshow(mask_new)
plt.axis('off')
plt.show()
```

<div align=center><img src="https://miro.medium.com/max/345/1*r3guH3h1l_jVniL3-M0kPw.png"></div>
<div align=center><h6>增强前景的分割蒙版</h6></div>

## 4. 背景情况
现在你需要一个新的背景图片。

使用我前面提到的免费网站来获得各种各样的背景。
你可以选择世界上的任何图像（看看我是如何在本文的展示图片中把我的狗送到月球上的），但我建议尽可能保持它的真实性。

我将使用这 4 张图片作为本教程的背景。

<div align=center><img src="https://miro.medium.com/max/445/1*mpMLx15hPqAHDU1bkuvJyQ.jpeg"><img src="https://miro.medium.com/max/556/1*TiQ1FB0vptOquWzpC7KF3w.jpeg"></div>
<div align=center><img src="https://miro.medium.com/max/500/1*mST_Z7tuAw3CMuNY_zs3DA.jpeg"><img src="https://miro.medium.com/max/500/1*3nSep1xlNcrnaEARSP2iwA.jpeg"></div>

我们用随机的方式挑选背景图。

``` python
# Random selection of background from the backgrounds folder
background_fileName = np.random.choice(os.listdir("./backgrounds/"))
background = io.imread('./backgrounds/'+background_fileName)/255.0
```

## 5. 最后：合成前景图和背景图
是时候将你的“狗”添加到背景图中了！

``` python
def compose(foreground, mask, background):
    # resize background
    background = transform.resize(background, foreground.shape[:2])

    # Subtract the foreground area from the background
    background = background*(1 - mask.reshape(foreground.shape[0], foreground.shape[1], 1))

    # Finally, add the foreground
    composed_image = background + foreground
    
    return composed_image

composed_image = compose(foreground_new, mask_new, background)
plt.imshow(composed_image)
plt.axis('off')
plt.show()
```

这个函数给出了下面的输出。
现在，我知道它并不完美，但已经相当不错了。

<div align=center><img src="https://miro.medium.com/max/345/1*GrGuLacfgeztDPo_3jBzGA.png"></div>
<div align=center><h6>最后合成的图像</h6></div>

对于对象定位任务，你也可以从我们生成的新分割蒙版中轻松获得框坐标。

``` python
# Get the smallest & largest non-zero values in each dimension and calculate the bounding box
nz = np.nonzero(mask_new)
bbox = [np.min(nz[0]), np.min(nz[1]), np.max(nz[0]), np.max(nz[1])]

x = bbox[1]
y = bbox[0]
width = bbox[3] - bbox[1]
height = bbox[2] - bbox[0]

# Display the image
plt.imshow(composed_image)

#draw bbox on the image
plt.gca().add_patch(Rectangle((x,y),width,height,linewidth=1,edgecolor='r',facecolor='none'))
plt.axis('off')
plt.show()
```

<div align=center><img src="https://miro.medium.com/max/345/1*sg3_y4S7iybcD70F49DWLg.png"></div>
<div align=center><h6>图像上标记的对象框</h6></div>

而在所有的背景上循环执行这些函数时，我们得到了一组漂亮的图像!

<div align=center><img src="https://miro.medium.com/max/1890/1*vRh7tzLSWC4LIEQG8bTS1Q.png"></div>
<div align=center><h6>一条狗在多个背景上的合成图像</h6></div>

在本教程中，我只使用了 1 个前景与 4 个背景。
如果你随机使用多个前景与多个背景，你可以创建很多很多的图像!
也许你也可以在一个背景上合并两个或多个前景。
发挥创意----根据你的需求，玩转大小、角度、前景和背景，建立一个很棒的合成数据集。
这里也有很多设计上的考虑，但只要稍加努力，你可以做得更多，并创建更精确的数据集。

> 最后，你可以在我的 [GitHub 仓库](https://github.com/virafpatrawala/Synthetic-Image-Datasets)中找到本教程的全部代码。

作为这个主题的下一步，你可以查看这个关于使用 Unity 3D 创建合成图像数据集的[教程](https://blog.stratospark.com/generating-synthetic-data-image-segmentation-unity-pytorch-fastai.html)。

好了，就到这里吧----感谢你读了这么久的文章。
我希望能从中得到一些好的东西。
有什么想法，问题，评论？请在下面的[回复](https://towardsdatascience.com/create-a-synthetic-image-dataset-the-what-the-why-and-the-how-f820e6b6f718)中告诉我吧!

---
[返回目录](https://github.com/datugou/Article_Translation/tree/master/LEARNING_data_science)
