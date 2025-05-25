---
title: Windows使用OpenAL+FreeAlut排坑记录
date: 2024-04-11T10:33:04+08:00
categories: 
- c++
- OpenAL
tags: 
- [c++]
- [OpenAL]
---
为了给[游戏](https://github.com/voidvvv/LinkA)添加一些炫酷的音效，所以我就又开始了搜索c++的一些音乐库，最后选择了同是open系列的OpenAL，但是对这个库的使用着实费了一些功夫,因为OPENAL只能负责如何播放音频，但是不负责加载音频，所以我们还需要FREEAlut这个工具来帮助我们，但是这个库的使用并不简单。

<!-- more -->

# OpenAL
因为想要给游戏添加音效，所以就搜到了[OpenAL](https://www.openal.org/).
由于之前对音乐解析这一部分没有太多的了解，仅仅是在LIBGDX中使用过封装好的工具，所以我理所当然的以为这一步会很简单，直接调用工具库然后使用（至少hello world）。
首先来到[OPENAL官网](https://www.openal.org/),自然而然的看到上面目录的download字样，自然而然的选择了下载。
![2024-04-11T104427](2024-04-11T104427.png)
然后我选择了下载CORE SDK：
![2024-04-11T104534](2024-04-11T104534.png)
下载下来发现是一个内部只有一个安装程序的压缩包，解压出来安装后：
![2024-04-11T104652](2024-04-11T104652.png)
可以看出来，直接是编译打包好的二进制文件，并且redist文件夹中还有一个可执行程序，会把openAL的动态链接库放到你操作系统依赖库(C:\Windows\SysWOW64\OpenAL32.dll 和 C:\Windows\System32\OpenAL32.dll)中去。等于是免去了编译的到过程
然后我就以为万事大吉了，准备使用了。按照[官方文档](https://www.openal.org/documentation/OpenAL_Programmers_Guide.pdf)，我便开始了一步步的hello world。
前几步都还ok，但是加载音频却出现了问题，因为OPENAL没有加载音频功能，于是我就去搜索了相关的资料，发现不同格式的音频还需要不同的decoder来解析成binary才能在openAL中使用，于是我便开始了新的一轮探索.

# ALUT
ALUT 其实就是 OpenAL Utils，顾名思义，就是OpenAL的一些工具包，可以在其[开源仓库](https://github.com/vancegroup/freealut)查看详情.
将其clone到本地后，我像之前一样使用cmake将其构建为mingw项目，然后编译，结果报错了。因为这freealut 中的CMakeLIST其实是适配linux的，报错的是这一段:
```cmakelist
# Dependencies
find_package(OpenAL REQUIRED)
```
这里其实该项目的readme也说了，使用cmake的时候需要指定cmake寻找project的prefix.
然后我去查了一下，发现这里其实就是需要指定openAL的include目录以及依赖文件，于是我找到了第一步中，直接下载好的include以及lib库替代了这里的配置:
```
# Dependencies
# find_package(OpenAL REQUIRED)
set(OPENAL_INCLUDE_DIR C:/myWareHouse/dev/cpp/include)
set(OPENAL_LIBRARY C:/myWareHouse/dev/cpp/vscode_lib/OpenAL32.lib)
```
但是编译还是无法通过，显示某些方法无法找到reference。应该就是lib库文件或者dll库文件有问题了。
然后我就在这一步被困了好久，头大了很久.
不过最后，还是让我找到了编译的方法

# 成功编译
## 手动编译OpenAL
其实在第一步下载的OpenAL就有些问题，如果想要使用freealut，需要手动编译OpenAL源码。
源码需要在OpenAL主页点击download 旁边的 Links
![2024-04-11T111505](2024-04-11T111505.png)
然后点击Open AL soft 进入 [soft主页](https://www.openal-soft.org/)
在这里可以下载源码或者下载该源码编译好的OpenAl.我们这里选择 [源码下载](https://www.openal-soft.org/openal-releases/openal-soft-1.23.1.tar.bz2)

## 将源码构建为Visual studio 项目。
下载后，使用cmake，将其构建为Visual Studio项目。这一步及其重要，一定要构建为Visual Studio项目，具体原因放在后面.
构建完毕后，进入build文件夹，使用Visual studio将其打开，然后编译打包.
如果一切顺利，会在Debug文件夹下看到这些：
![2024-04-11T112016](2024-04-11T112016.png)
然后，我们的OpenAL才算是正式能用了，可以看到，我们编译好的文件，其中 OpenAL32.dll 大小是 3519KB，跟之前redist给我们放的 20KB完全不一样，然后，我们可以使用这个dll来覆盖之前的(C:\Windows\SysWOW64\OpenAL32.dll 和 C:\Windows\System32\OpenAL32.dll)。

## 再次编译FreeAlut
将我们刚才编译出的lib文件放入我们freealut所依赖的库目录中，头文件也如此做。
还是打开之前的freealut项目，将其CMAKEList仍然改为：
```
# Dependencies
# find_package(OpenAL REQUIRED)
set(OPENAL_INCLUDE_DIR C:/myWareHouse/dev/cpp/include)
set(OPENAL_LIBRARY C:/myWareHouse/dev/cpp/vscode_lib/OpenAL32.lib)
```
然后保存，再次Cmake build，发现可以成功编译，并且生成了 libalut.dll 以及其 导入库： libalut.dll.a
我们自己的项目只需要将dll复制到我们执行程序目录，然后源码中引入libalut.dll.a，同时还需要引入 OpenAL32.lib，
我们自己项目中的CMAKELIST 引入依赖大概是这个样子:
```
target_link_libraries(${PROJECT_NAME} libglfw3.a  libalut.dll.a OpenAL32.lib)

```
最后，从freealut项目中，粘贴一段demo代码过来，我们试着执行一下：
```c++

#include <iostream>
#include <AL/alut.h>

// #include <bass.h>


void sayHu(){
      ALuint helloBuffer, helloSource;
  alutInit (NULL, NULL);
  helloBuffer = alutCreateBufferHelloWorld ();
  alGenSources (1, &helloSource);
  alSourcei (helloSource, AL_BUFFER, helloBuffer);
  alSourcePlay (helloSource);
  alutSleep (1);
  alutExit ();
  return ;
};

int main()
{
    sayHu();

}
```
会听到一段hello world。至此，OpenAL+ freeALUT的集成完毕.我们可以在游戏中添加音效了.

## 为何必须打包成Visual Studio
这里，仅仅是我自己的一个推测。
当我打包成mingw程序时，编译无法通过，查看源码发现，出问题的是在alfstream.h 以及其cpp文件中。
![2024-04-11T113206](2024-04-11T113206.png)
这里可以看到openAL自定义的ifstream类继承了SDK中自带的ifstream类，并且调用了其构造方法。其中有一个是wchar_t参数的构造方法。
在mingw中，我们可以看到，std::ifstream其实就是模板类型为char的basic_ifstream, 
模板类型为wchar的是另外一个类型：std:wistream.
![2024-04-11T113436](2024-04-11T113436.png)
既然模板类型都不一样，那自然无法编译通过。

但是在visual studio中，我们可以看到，basic_ifstream本身其实是有wchar作为参数的构造方法的，所以在这里自然能够编译通过。
![2024-04-11T113557](2024-04-11T113557.png)
所以我们在使用cmake构建openAL时，必须将其构建为visual studio。
但是如果你的mingw版本不一样，sdk中ifstream已经支持这种方法，那么我觉得也是可以构建为mingw项目的。
我当前使用的mingw版本如下：
![2024-04-11T113852](2024-04-11T113852.png)

完毕，可以愉快的使用OpenAL了。
