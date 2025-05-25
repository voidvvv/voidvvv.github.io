---
title: OpenGL 纹理
date: 2024-03-05T20:10:01+08:00
categories: 
- c++
- OpenGL
tags: 
- [c++]
- [OpenGL]
---


![2024-03-02T220734](2024-03-02T220734.png)
为了更好的自学游戏开发，所以自学OpenGL
<!-- more -->
- [纹理](#纹理)
  - [纹理环绕方式](#纹理环绕方式)
  - [纹理过滤](#纹理过滤)
  - [多级渐远纹理](#多级渐远纹理)
  - [加载与创建纹理](#加载与创建纹理)
    - [stb\_image.h](#stb_imageh)
  - [生成纹理](#生成纹理)
  - [应用纹理](#应用纹理)
  - [纹理单元](#纹理单元)


# 纹理

> 纹理设置参数方法： `glTexParameter`

我们已经了解到，我们可以为每个顶点添加颜色来增加图形的细节，从而创建出有趣的图像。但是，如果想让图形看起来更真实，我们就必须有足够多的顶点，从而指定足够多的颜色。这将会产生很多额外开销，因为每个模型都会需求更多的顶点，每个顶点又需求一个颜色属性。

艺术家和程序员更喜欢使用纹理(Texture)。纹理是一个2D图片（甚至也有1D和3D的纹理），它可以用来添加物体的细节；你可以想象纹理是一张绘有砖块的纸，无缝折叠贴合到你的3D的房子上，这样你的房子看起来就像有砖墙外表了。因为我们可以在一张图片上插入非常多的细节，这样就可以让物体非常精细而不用指定额外的顶点。

<div class="note info">简单地说，texture纹理就是把图片以指定形式展示出来(甚至可以贴在我们的物体上)</div>

我们可以为text纹理指定坐标，就像之前指定我们三角形（或者别的形状）指定坐标一样,只需要继续追加坐标即可。
```c++
float vertex[] = {
		// position			  color					texCoords
		-0.5f,-0.5f,0.f,	0.5f,0.f,0.f,		-0.5f,-0.5f,
		-0.5f,0.5f,0.f,		0.0f,0.5f,0.f,		-0.5f,0.5f,
		0.5f,0.5f,0.f,		0.0f,0.f,0.5f,		0.5f,0.5f,
		0.5f,-0.5f,0.f,		0.5f,0.5f,0.5f,		0.5f,-0.5f,

	};
```

## 纹理环绕方式
纹理坐标的范围通常是从(0, 0)到(1, 1)，那如果我们把纹理坐标设置在范围之外会发生什么？OpenGL默认的行为是重复这个纹理图像（我们基本上忽略浮点纹理坐标的整数部分），但OpenGL提供了更多的选择：

|环绕方式	|描述   |
|-------- | -----|
|`GL_REPEAT`|	对纹理的默认行为。重复纹理图像。|
|`GL_MIRRORED_REPEAT`|	和GL_REPEAT一样，但每次重复图片是镜像放置的。|
|`GL_CLAMP_TO_EDGE`|纹理坐标会被约束在0到1之间，超出的部分会重复纹理坐标的边缘，产生一种边缘被拉伸的效果。|
|`GL_CLAMP_TO_BORDER`|	超出的坐标为用户指定的边缘颜色。|

![2024-03-05T203029](2024-03-05T203029.png)
前面提到的每个选项都可以使用`glTexParameter*`函数对单独的一个坐标轴设置（s、t（如果是使用3D纹理那么还有一个r）它们和x、y、z是等价的）：
```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT);
```
第一个参数指定了纹理目标；我们使用的是2D纹理，因此纹理目标是`GL_TEXTURE_2D`。第二个参数需要我们指定设置的选项与应用的纹理轴。我们打算配置的是WRAP选项，并且指定S和T轴。最后一个参数需要我们传递一个环绕方式(Wrapping)，在这个例子中OpenGL会给当前激活的纹理设定纹理环绕方式为`GL_MIRRORED_REPEAT`。
如果我们选择`GL_CLAMP_TO_BORDER`选项，我们还需要指定一个边缘的颜色。这需要使用glTexParameter函数的fv后缀形式，用`GL_TEXTURE_BORDER_COLOR`作为它的选项，并且传递一个float数组作为边缘的颜色值：
```c++
float borderColor[] = { 1.0f, 1.0f, 0.0f, 1.0f };
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);
```

## 纹理过滤
纹理坐标不依赖于分辨率(Resolution)，它可以是任意浮点值，所以OpenGL需要知道怎样将纹理像素(Texture Pixel，也叫Texel，译注1)映射到纹理坐标。**当你有一个很大的物体但是纹理的分辨率很低的时候这就变得很重要了**。你可能已经猜到了，OpenGL也有对于纹理过滤(Texture Filtering)的选项。纹理过滤有很多个选项，但是现在我们只讨论最重要的两种：`GL_NEAREST``和GL_LINEAR。`
{% note danger no-icon %}
Texture Pixel也叫Texel，你可以想象你打开一张`.jpg`格式图片，不断放大你会发现它是由无数像素点组成的，这个点就是纹理像素；注意不要和纹理坐标搞混，纹理坐标是你给模型顶点设置的那个数组，OpenGL以这个顶点的纹理坐标数据去查找纹理图像上的像素，然后进行采样提取纹理像素的颜色。
{% endnote %}

`GL_NEAREST`（也叫邻近过滤，Nearest Neighbor Filtering）是OpenGL默认的纹理过滤方式。当设置为GL_NEAREST的时候，OpenGL会选择中心点最接近纹理坐标的那个像素。下图中你可以看到四个像素，加号代表纹理坐标。左上角那个纹理像素的中心距离纹理坐标最近，所以它会被选择为样本颜色：
![2024-03-05T210625](2024-03-05T210625.png)

`GL_LINEAR`（也叫线性过滤，(Bi)linear Filtering）它会基于纹理坐标附近的纹理像素，计算出一个插值，近似出这些纹理像素之间的颜色。一个纹理像素的中心距离纹理坐标越近，那么这个纹理像素的颜色对最终的样本颜色的贡献越大。下图中你可以看到返回的颜色是邻近像素的混合色：
![2024-03-05T210640](2024-03-05T210640.png)

那么这两种纹理过滤方式有怎样的视觉效果呢？让我们看看在一个很大的物体上应用一张低分辨率的纹理会发生什么吧（纹理被放大了，每个纹理像素都能看到）：
![2024-03-05T210658](2024-03-05T210658.png)

`GL_NEAREST`产生了颗粒状的图案，我们能够清晰看到组成纹理的像素，而`GL_LINEAR`能够产生更平滑的图案，很难看出单个的纹理像素。`GL_LINEAR`可以产生更真实的输出，但有些开发者更喜欢8-bit风格，所以他们会用`GL_NEAREST`选项。

当进行放大(Magnify)和缩小(Minify)操作的时候可以设置纹理过滤的选项，比如你可以在纹理被缩小的时候使用邻近过滤，被放大时使用线性过滤。我们需要使用`glTexParameter*`函数为放大和缩小指定过滤方式。这段代码看起来会和纹理环绕方式的设置很相似：
```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

## 多级渐远纹理
想象一下，假设我们有一个包含着上千物体的大房间，每个物体上都有纹理。有些物体会很远，但其纹理会拥有与近处物体同样高的分辨率。由于远处的物体可能只产生很少的片段，OpenGL从高分辨率纹理中为这些片段获取正确的颜色值就很困难，因为它需要对一个跨过纹理很大部分的片段只拾取一个纹理颜色。在小物体上这会产生不真实的感觉，更不用说对它们使用高分辨率纹理浪费内存的问题了。

OpenGL使用一种叫做**多级渐远纹理**(Mipmap)的概念来解决这个问题，它简单来说就是一系列的纹理图像，后一个纹理图像是前一个的二分之一。多级渐远纹理背后的理念很简单：距观察者的距离超过一定的阈值，OpenGL会使用不同的多级渐远纹理，即最适合物体的距离的那个。由于距离远，解析度不高也不会被用户注意到。同时，多级渐远纹理另一加分之处是它的性能非常好。让我们看一下多级渐远纹理是什么样子的：

![2024-03-05T210946](2024-03-05T210946.png)

手工为每个纹理图像创建一系列多级渐远纹理很麻烦，幸好OpenGL有一个`glGenerateMipmaps`函数，在创建完一个纹理后调用它OpenGL就会承担接下来的所有工作了。后面的教程中你会看到该如何使用它。

在渲染中切换多级渐远纹理级别(Level)时，OpenGL在两个不同级别的多级渐远纹理层之间会产生不真实的生硬边界。就像普通的纹理过滤一样，切换多级渐远纹理级别时你也可以在两个不同多级渐远纹理级别之间使用NEAREST和LINEAR过滤。为了指定不同多级渐远纹理级别之间的过滤方式，你可以使用下面四个选项中的一个代替原有的过滤方式：
|过滤方式|描述|
|------|------|
|`GL_NEAREST_MIPMAP_NEAREST`|使用最邻近的多级渐远纹理来匹配像素大小，并使用邻近插值进行纹理采样|
|`GL_LINEAR_MIPMAP_NEAREST`|使用最邻近的多级渐远纹理级别，并使用线性插值进行采样|
|`GL_NEAREST_MIPMAP_LINEAR`|在两个最匹配像素大小的多级渐远纹理之间进行线性插值，使用邻近插值进行采样|
|`GL_LINEAR_MIPMAP_LINEAR`|在两个邻近的多级渐远纹理之间使用线性插值，并使用线性插值进行采样|

```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

注意，一个常见的错误是，将**放大过滤**的选项设置为多级渐远纹理过滤选项之一。这样没有任何效果，因为多级渐远纹理主要是使用在纹理被缩小的情况下的：纹理放大不会使用多级渐远纹理，为放大过滤设置多级渐远纹理的选项会产生一个`GL_INVALID_ENUM`错误代码。

## 加载与创建纹理
使用纹理之前要做的第一件事是把它们加载到我们的应用中。纹理图像可能被储存为各种各样的格式，每种都有自己的数据结构和排列，所以我们如何才能把这些图像加载到应用中呢？一个解决方案是选一个需要的文件格式，比如`.PNG`，然后自己写一个图像加载器，把图像转化为字节序列。写自己的图像加载器虽然不难，但仍然挺麻烦的，而且如果要支持更多文件格式呢？你就不得不为每种你希望支持的格式写加载器了。
> 另一个解决方案也许是一种更好的选择，使用一个支持多种流行格式的图像加载库来为我们解决这个问题。比如说我们要用的`stb_image.h`库。

### stb_image.h
`stb_image.h`是[Sean Barrett](https://github.com/nothings)的一个非常流行的单头文件图像加载库，它能够加载大部分流行的文件格式，并且能够很简单得整合到你的工程之中。stb_image.h可以在这里[下载](https://github.com/nothings/stb/blob/master/stb_image.h)。下载这一个头文件，将它以stb_image.h的名字加入你的工程，并另创建一个新的C++文件，输入以下代码：

```c++
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"
```
通过定义`STB_IMAGE_IMPLEMENTATION`，预处理器会修改头文件，让其只包含相关的函数定义源码，等于是将这个头文件变为一个 .cpp 文件了。现在只需要在你的程序中包含stb_image.h并编译就可以了。
现在，可以随手挑一张图片。要使用`stb_image.h`加载图片，我们需要使用它的`stbi_load`函数：
```c++
int width, height, nrChannels;
stbi_set_flip_vertically_on_load(true);
unsigned char *data = stbi_load("container.jpg", &width, &height, &nrChannels, 0);
```
这个函数首先接受一个图像文件的位置作为输入。接下来它需要三个`int`作为它的第二、第三和第四个参数，`stb_image.h`将会用图像的**宽度、高度和颜色通道**的个数填充这三个变量。我们之后生成纹理的时候会用到的图像的宽度和高度的。
但是，在导入之前，需要先调用一下 `stbi_set_flip_vertically_on_load`函数，来将图片放正。因为默认的图片的0坐标位置是在最上面。跟我们gl中的相反。

## 生成纹理
和之前生成的OpenGL对象一样，纹理也是使用ID引用的。让我们来创建一个：
```c++
unsigned int texture;
glGenTextures(1, &texture);
```
再然后，在操作纹理前绑定：
```c++
glBindTexture(GL_TEXTURE_2D, texture);
```
现在纹理已经绑定了，我们可以使用前面载入的图片数据生成一个纹理了。纹理可以通过glTexImage2D来生成：
```c++
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB,GL_UNSIGNED_BYTE,data);
glGenerateMipmap(GL_TEXTURE_2D);
```
函数很长，参数也不少，所以我们一个一个地讲解：
- 第一个参数指定了纹理目标(`Target`)。设置为GL_TEXTURE_2D意味着会生成与当前绑定的纹理对象在同一个目标上的纹理（任何绑定到GL_TEXTURE_1D和GL_TEXTURE_3D的纹理不会受到影响）。
- 第二个参数为纹理指定多级渐远纹理的级别，如果你希望单独手动设置每个多级渐远纹理的级别的话。这里我们填0，也就是基本级别。
- 第三个参数告诉OpenGL我们希望把纹理储存为何种格式。我们的图像只有RGB值，因此我们也把纹理储存为RGB值。
- 第四个和第五个参数设置最终的纹理的宽度和高度。我们之前加载图像的时候储存了它们，所以我们使用对应的变量。
- 下个参数应该总是被设为0（历史遗留的问题）。
- 第七第八个参数定义了源图的格式和数据类型。我们使用RGB值加载这个图像，并把它们储存为char(byte)数组，我们将会传入对应值。
- 最后一个参数是真正的图像数据。

当调用`glTexImage2D`时，当前绑定的纹理对象就会被附加上纹理图像。然而，目前只有基本级别(Base-level)的纹理图像被加载了，如果要使用多级渐远纹理，我们必须手动设置所有不同的图像（不断递增第二个参数）。或者，直接在生成纹理之后调用`glGenerateMipmap`。这会为当前绑定的纹理自动生成所有需要的多级渐远纹理。
生成了纹理和相应的多级渐远纹理后，释放图像的内存是一个很好的习惯。
```c++
stbi_image_free(data);
```
生成一个纹理的过程应该看起来像这样：
```c++
unsigned int texture;
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);
// 为当前绑定的纹理对象设置环绕、过滤方式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);   
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
// 加载并生成纹理
int width, height, nrChannels;
unsigned char *data = stbi_load("container.jpg", &width, &height, &nrChannels, 0);
if (data)
{
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
    glGenerateMipmap(GL_TEXTURE_2D);
}
else
{
    std::cout << "Failed to load texture" << std::endl;
}
stbi_image_free(data);
```

## 应用纹理
后面的这部分我们会使用`glDrawElements`绘制「你好，三角形」教程最后一部分的矩形。我们需要告知OpenGL如何采样纹理，所以我们必须使用纹理坐标更新顶点数据：
```c++
	float vertex[] = {
		// position			  color					texCoords
		-0.5f,-0.5f,0.f,	0.5f,0.f,0.f,		0.f,0.f,
		-0.5f,0.5f,0.f,		0.0f,0.5f,0.f,		0.f,1.f,
		0.5f,0.5f,0.f,		0.0f,0.f,0.5f,		1.f,1.f,
		0.5f,-0.5f,0.f,		0.5f,0.5f,0.5f,		1.f,0.f,

	};
```
{% note danger no-icon %}
这里需要讲一下，纹理图片的坐标范围是 0-1 （这点跟gl的-1 - 1 不一样），横坐标 0-1代表一个图片整个宽度，纵坐标 0-1 代表整个图片的高度。
这里设置属性的意思代表：在GLSL刷新到 （-0.5，-0.5）点的时候，对应的颜色是 (0.5,0,0),对应的纹理坐标为（0，0）. 然后四个坐标依次类推。整个遍历过程是根据定义的EBO中的indices来决定，然后GL依次序来遍历所有的坐标点（**范围内的所有，比如当前这里就是一个（-0.5，-0.5）到(0.5,0.5) 的矩形**）。然后每到一个点，都会根据比例计算出相应的color坐标，textCoords坐标来决定传入的color属性是什么，以及传入的texCoords是什么。
这里就是在整个GL渲染范围内，正好要渲染整张图片。
![2024-03-05T223546](2024-03-05T223546.png)
但如果是下面这样：
```c++
	float vertex[] = {
		// position			  color					texCoords
		-0.5f,-0.5f,0.f,	0.5f,0.f,0.f,		0.f,0.f,
		-0.5f,0.5f,0.f,		0.0f,0.5f,0.f,		0.f,0.5f,
		0.5f,0.5f,0.f,		0.0f,0.f,0.5f,		0.5f,0.5f,
		0.5f,-0.5f,0.f,		0.5f,0.5f,0.5f,		0.5f,0.f,

	};
```
这样的画图片纹理的范围就只有一半，也就是说，在整个gl遍历的过程中，图片会以一半的形式来占据整个遍历范围。

![2024-03-05T223446](2024-03-05T223446.png)
{% endnote %}


然后，添加新的输入属性:
```c++
glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void*)(6 * sizeof(float)));
glEnableVertexAttribArray(2);
```
接着我们需要调整顶点着色器使其能够接受顶点坐标为一个顶点属性，并把坐标传给片段着色器：

```glsl
#version 330 core
uniform float colorSin;

layout (location = 0) in vec3 aPos;   // 位置变量的属性位置值为 0 
layout (location = 1) in vec3 aColor; // 颜色变量的属性位置值为 1
layout (location = 2) in vec2 aTexCoord; // 纹理坐标


out vec3 ourColor; // 向片段着色器输出一个颜色
out vec2 texCoord;

void main()
{
    gl_Position = vec4(aPos, 1.0);
    ourColor = aColor; // 将ourColor设置为我们从顶点数据那里得到的输入颜色
    texCoord = aTexCoord;
}
```

片段着色器应该接下来会把输出变量TexCoord作为输入变量。

片段着色器也应该能访问纹理对象，但是我们怎样能把纹理对象传给片段着色器呢？GLSL有一个供纹理对象使用的内建数据类型，叫做采样器(Sampler)，它以纹理类型作为后缀，比如sampler1D、sampler3D，或在我们的例子中的sampler2D。我们可以简单声明一个uniform sampler2D把一个纹理添加到片段着色器中，稍后我们会把纹理赋值给这个uniform。

```glsl
#version 330 core
out vec4 FragColor;

in vec3 ourColor;
in vec2 TexCoord;

uniform sampler2D ourTexture;

void main()
{
    FragColor = texture(ourTexture, TexCoord);
}

```
我们使用GLSL内建的texture函数来采样纹理的颜色，它第一个参数是纹理采样器，第二个参数是对应的纹理坐标。texture函数会使用之前设置的纹理参数对相应的颜色值进行采样。这个片段着色器的输出就是纹理的（插值）纹理坐标上的(过滤后的)颜色。

现在只剩下在调用glDrawElements之前绑定纹理了，它会自动把纹理赋值给片段着色器的采样器：

```c++
glBindTexture(GL_TEXTURE_2D, texture);
glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

我们还可以把得到的纹理颜色与顶点颜色混合，来获得更有趣的效果。我们只需把纹理颜色与顶点颜色在片段着色器中相乘来混合二者的颜色：
```glsl
FragColor = texture(ourTexture, TexCoord) * vec4(ourColor, 1.0);
```
最终的效果应该是顶点颜色和纹理颜色的混合色.
![2024-03-05T223955](2024-03-05T223955.png)

## 纹理单元
你可能会奇怪为什么sampler2D变量是个uniform，我们却不用glUniform给它赋值。使用glUniform1i，我们可以给纹理采样器分配一个位置值，这样的话我们能够在一个片段着色器中设置多个纹理。一个纹理的位置值通常称为一个纹理单元(Texture Unit)。一个纹理的默认纹理单元是0，它是默认的激活纹理单元，所以教程前面部分我们没有分配一个位置值。

纹理单元的主要目的是让我们在着色器中可以使用多于一个的纹理。通过把纹理单元赋值给采样器，我们可以一次绑定多个纹理，只要我们首先激活对应的纹理单元。就像glBindTexture一样，我们可以使用glActiveTexture激活纹理单元，传入我们需要使用的纹理单元：
```c++
glActiveTexture(GL_TEXTURE0); // 在绑定纹理之前先激活纹理单元
glBindTexture(GL_TEXTURE_2D, texture);
```
激活纹理单元之后，接下来的glBindTexture函数调用会绑定这个纹理到当前激活的纹理单元，**纹理单元GL_TEXTURE0默认总是被激活，所以我们在前面的例子里当我们使用glBindTexture的时候，无需激活任何纹理单元**。
<div class="note info">OpenGL至少保证有16个纹理单元供你使用，也就是说你可以激活从GL_TEXTURE0到GL_TEXTRUE15。它们都是按顺序定义的，所以我们也可以通过GL_TEXTURE0 + 8的方式获得GL_TEXTURE8，这在当我们需要循环一些纹理单元的时候会很有用。</div>

我们仍然需要编辑片段着色器来接收另一个采样器。这应该相对来说非常直接了：
```glsl
#version 330 core
...

uniform sampler2D texture1;
uniform sampler2D texture2;

void main()
{
    FragColor = mix(texture(texture1, TexCoord), texture(texture2, TexCoord), 0.2);
}
```
最终输出颜色现在是两个纹理的结合。GLSL内建的mix函数需要接受两个值作为参数，并对它们根据第三个参数进行线性插值。如果第三个值是0.0，它会返回第一个输入；如果是1.0，会返回第二个输入值。0.2会返回80%的第一个输入颜色和20%的第二个输入颜色，即返回两个纹理的混合色。

那么选择第二张图片开始混合把。
为了使用第二个纹理（以及第一个），我们必须改变一点渲染流程，先绑定两个纹理到对应的纹理单元，然后定义哪个uniform采样器对应哪个纹理单元：
```c++
// 使用当前的shader program
glActiveTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_2D, texture1);
glActiveTexture(GL_TEXTURE1);
glBindTexture(GL_TEXTURE_2D, texture2);

glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
````
上面的意思是依次激活一个纹理单元，然后紧跟着将刚才激活的单元放入一个纹理.

我们还要通过使用`glUniform1i`设置每个采样器的方式告诉OpenGL每个着色器采样器属于哪个纹理单元。我们只需要设置一次即可，所以这个会放在渲染循环的前面：
```c++
ourShader.use(); // 不要忘记在设置uniform变量之前激活着色器程序！
glUniform1i(glGetUniformLocation(ourShader.ID, "texture1"), 0); // 手动设置
ourShader.setInt("texture2", 1); // 或者使用着色器类设置

while(...) 
{
    [...]
}
```
这个步骤是一次性的，我们定义一次即可，所以可以放在渲染循环外面。意思就是对应的变量名，去对应的纹理单元中取得纹理即可.因为这里的纹理单元对应我们之前激活的数个纹理单元，所以这里应该是uniform的int
通过使用glUniform1i设置采样器，我们保证了每个uniform采样器对应着正确的纹理单元。

最后，你应该就可以见到一个混合着两幅图的画面了。



