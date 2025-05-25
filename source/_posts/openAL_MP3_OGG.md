---
title: OpenAL 解析MP3 + OGG 音频
date: 2024-04-12T23:25:41+08:00
categories: 
- c++
- OpenAL
tags: 
- [c++]
- [OpenAL]
---
在前面已经成功的给游戏引入了OpenAL作为音频播放工具，并且使用了FreeAlut来解析音频。
但是这个freeALUT有一个问题，就是目前只支持Wav格式的文件。我们这次来给我们的系统添加对MP3和ogg的支持

<!-- more -->
# FreeALUT
首先先看下freeALUT的源码：
```c++
  fileName = _alutInputStreamGetFileName (stream);
  if (fileName != NULL && hasSuffixIgnoringCase (fileName, ".raw"))
    {
      return loadRawFile (stream);
    }

  /* For other file formats, read the quasi-standard four byte magic number */
  if (!_alutInputStreamReadInt32BE (stream, &magic))
    {
      return AL_FALSE;
    }

  /* Magic number 'RIFF' == Microsoft '.wav' format */
  if (magic == 0x52494646)
    {
      return loadWavFile (stream);
    }

  /* Magic number '.snd' == Sun & Next's '.au' format */
  if (magic == 0x2E736E64)
    {
      return loadAUFile (stream);
    }
```
可以看到这里本身确实只支持了raw，wav文件。

# OGG支持
为了支持ogg，我们需要引入额外的依赖库，libogg + libvorbis。
我们可以在[这里](https://xiph.org/downloads/)找到下载的链接，也可以去[GITHUB仓库](https://github.com/xiph/ogg)来clone。
<div class="note danger">需要注意的是，我们这里需要使用两个库，这两个库也是有依赖关系的，即libvorbis 依赖于 libogg ，这点很重要</div>

## LIBOGG
我们需要编译LIBOGG库。
直接打开然后使用cmake生成项目文件，然后clean & rebuild 生成即可，默认会生成.a后缀的静态库，我们这里就使用静态库就好。

## libvorbis
打开libvorbis库，仍然还是使用cmake生成项目文件。
但是这里需要注意一下，libvorbis库需要依赖libOGG库来编译，因为libvorbis本质是对LIBOGG的一个拓展。
这里，我们有两个方法来引入libOGG：
### 更改引用文件
1. 首先打开libvorbis根目录的cmakelist文件
2. 注释掉find_package步骤
3. 添加两个变量，就是我们libOGG打包后的include文件目录，以及生成的依赖文件地址：
   ![2024-04-12T234529](2024-04-12T234529.png) 
4. 进入lib文件夹下，打开这里的cmakelist
5. 在第89行左右的位置（如下图），将所有的include directories设置中，全部加上刚才设置的include文件目录变量：
    ![2024-04-12T234947](2024-04-12T234947.png)
6. 将下面对 vorbis的 target_link_libraries 中，PUBLIC Ogg::ogg 改为 PUBLIC + `生成的依赖文件地址变量`
   ![2024-04-12T235104](2024-04-12T235104.png)
   然后打包编译，

### 方法2 install LIBOGG
在编译libogg库成功后，可以尝试在libOGG库的cmake文件开头加上如下定义:
```cmake
set(CMAKE_INSTALL_PREFIX  C:/myWareHouse/dev/cpp/cmake_install)
```
这个定义就是用来设置cmake的安装目录的。其实cmake是有默认的安装目录的，但是其默认安装目录为C:/Program Files, cmake没有权限去install，所以我们最好自己再指定一个目录。然后执行cmake install命令，这样，我们就会在对应目录看到我们的libOGG已经被安装好了。
这里的安装其实就是把头文件，以及依赖库放在了这里。

然后我们可以在libvorbis根目录的cmakelist文件夹下，加入如下定义:
```cmake
set(CMAKE_INSTALL_PREFIX  C:/myWareHouse/dev/cpp/cmake_install)

set(CMAKE_PREFIX_PATH  C:/myWareHouse/dev/cpp/cmake_install)

```
这里第一个变量含义仍是安装目录，代表现在如果在libvorbis目录下执行install，那么libvorbis也会安装到这里。
第二个变量`CMAKE_PREFIX_PATH`，代表当前项目在使用cmake构建的时候，会去这个目录下查找当前已经安装过的库，并直接从这里读取头文件以及依赖。
这样，我们就不必更改别的地方了，直接执行cmake rebuild，cmake就会去我们到的这个目录下自己去找libOGG，即可成功编译打包，最后也可以将libvorbis 进行一下install操作，这样我们的vorbis也会在我们的cmake库中了。
<div class="note info"> 这个形式其实跟java中的maven有些类似了</div>

### 使用
在编译打包成功后，会生成三个依赖库文件：
 ![2024-04-12T235156](2024-04-12T235156.png)
现在我们如果需要解析ogg音频文件，需要引入这三个依赖。同时还需要引入libOGG依赖。
但是这里有一个坑，<span style='color:red'>libOGG是被这三个所依赖的，在我们自己的应用中，如果要引入这几个依赖，需要先引入libvorbis的三个依赖，然后再引入libOGG，这样我们自己的项目才能成功编译通过，否则是无法使用这个依赖库的。这个是g++本身的设定，其引入库是需要有顺序的，被依赖的库应当放在后面</span>
像下面这样:
```cmake
target_link_libraries(${PROJECT_NAME} 
libvorbisfile.a 
libvorbis.a
libvorbisenc.a
libogg.a
)
```
## demo
最后放上一个使用demo，demo中将该库与openAL进行了整合
```c++
#include <AL/al.h>
#include <AL/alut.h>
#include <al/alc.h>
#include <vorbis/vorbisfile.h>
#include <cstdio>
#include <iostream>
#include <vector>
 
#define BUFFER_SIZE     32768       // 32 KB buffers
 
typedef struct ALCdevice_struct ALCdevice;
typedef struct ALCcontext_struct ALCcontext;
 
using namespace std;
 
bool LoadOGG(char *name, vector<char> &buffer, ALenum &format, ALsizei &freq)
{
    int endian = 0;                         // 0 for Little-Endian, 1 for Big-Endian
    int bitStream;
    long bytes;
    char array[BUFFER_SIZE];                // Local fixed size array
    FILE *f;
 
    f = fopen(name, "rb");
 
    if (f == NULL)
        return false; 
 
    vorbis_info *pInfo;
    OggVorbis_File oggFile;
 
    // Try opening the given file
    if (ov_open(f, &oggFile, NULL, 0) != 0)
        return false; 
 
    pInfo = ov_info(&oggFile, -1);
 
    if (pInfo->channels == 1)
        format = AL_FORMAT_MONO16;
    else
        format = AL_FORMAT_STEREO16;
  
    freq = pInfo->rate;
 
    do
    { 
        bytes = ov_read(&oggFile, array, BUFFER_SIZE, endian, 2, 1, &bitStream);
 
        if (bytes < 0)
            {
            ov_clear(&oggFile);
            cerr << "Error decoding " << "fileName" << "..." << endl;
            exit(-1);
            }
 
        buffer.insert(buffer.end(), array, array + bytes);
    }
    while (bytes > 0);
 
    ov_clear(&oggFile);
    return true; 
}
 
int main(int argc, char *argv[])
{
 
    ALCdevice* pDevice;
    ALCcontext* pContext;
 
    ALint state;                            // The state of the sound source
    ALuint bufferID;                        // The OpenAL sound buffer ID
    ALuint sourceID;                        // The OpenAL sound source
    ALenum format;                          // The sound data format
    ALsizei freq;                           // The frequency of the sound data
    vector<char> bufferData;                // The sound buffer data from file
     
	 ALCdevice *device;
     ALCcontext *context; 
 
       device = alcOpenDevice(0);
       context = alcCreateContext(device,0);
	   ALboolean initStatus = alcMakeContextCurrent(context);    
 
    // Create sound buffer and source
    alGenBuffers(1, &bufferID);
    alGenSources(1, &sourceID);
 
    // Set the source and listener to the same location
    alListener3f(AL_POSITION, 0.0f, 0.0f, 0.0f);
    alSource3f(sourceID, AL_POSITION, 0.0f, 0.0f, 0.0f);
 
    // Load the OGG file into memory
    LoadOGG("./sound/TestBeatMono.ogg", bufferData, format, freq);
 
    // Upload sound data to buffer
    alBufferData(bufferID, format, &bufferData[0], static_cast<ALsizei>(bufferData.size()), freq);
 
    // Attach sound buffer to source
    alSourcei(sourceID, AL_BUFFER, bufferID);
	
	alSourcef (sourceID, AL_GAIN, 1.0 );
 
    // Finally, play the sound!!!
    alSourcePlay(sourceID);
 
    do
    {
        // Query the state of the souce
        alGetSourcei(sourceID, AL_SOURCE_STATE, &state);
    }
    while (state != AL_STOPPED);
 
    // Clean up sound buffer and source
    alDeleteBuffers(1, &bufferID);
    alDeleteSources(1, &sourceID);
 
    alcDestroyContext(context);
	alcCloseDevice(device);   
 
    return 0;
}
```
然后我们就可以欣赏美妙的ogg音乐了。

# MP3
关于MP3，我其实并没有找到特别好用的专门处理MP3的库。但是我找到了一个[全能库](https://github.com/ddiakopoulos/libnyquist)，它不仅能解析MP3，还能解析个各种音频文件。
关于这个库的编译，没什么好说的，直接下下来，编译打包，install即可。
这个库的使用有一点点坑

## 格式
在之前使用openAL播放音频的时候，我们传入的bufferData都是一个char数组，但是这个库它解析出来的数据是float数组。
这里有一个坑，就是如果给openAL绑定Buffer数据的时候，format如果使用默认的格式，则会出现噪音，乱流，甚至杂音，以及完全无法确认的声源。
这时我们需要引入`#include <AL/alext.h>`库，这个库中有专门对float32优化的格式：`AL_FORMAT_STEREO_FLOAT32`

具体代码如下,使用libnyquist跟openAL整合：
```c++

#include <libnyquist/Decoders.h>
#include <libnyquist/Encoders.h>
#include <libnyquist/Common.h>

#include <AL/alext.h>
#include <AL/alut.h>

using namespace nqr;
bool compare_pred(unsigned char a, unsigned char b)
{
    return std::tolower(a) == std::tolower(b);
}
static const uint32_t FRAME_SIZE = 512;
static const int32_t CHANNELS = 2;
static const int32_t BUFFER_LENGTH = FRAME_SIZE * CHANNELS;
int main()
try
{
    const char *s1 = "abc123";
    const char *s2 = "123";
    std::cout << ((s1 + 3) == s2) << " - = - " << std::equal(s1 + 3, s1 + 6, s2) << std::endl;
    std::shared_ptr<AudioData> fileData = std::make_shared<AudioData>();

    NyquistIO loader;

    std::string cli_arg = std::string("./sound/xx.mp3");
    auto memory = ReadFile(cli_arg);
    loader.Load(fileData.get(), "mp3", memory.buffer);

    std::cout << fileData.get()->sourceFormat << std::endl;
    std::cout << fileData.get()->channelCount << std::endl;
    std::cout << fileData.get()->sampleRate << std::endl;
    std::cout << "frameSize: " << fileData.get()->frameSize << std::endl;
    std::cout << "size: " << fileData.get()->samples.size() << std::endl;
    fileData.get()->samples.data();

    alutInit(NULL, NULL);
    ALuint buffer, source;
    alGenBuffers(1, &buffer);
    alGenSources(1, &source);
    ALenum format;
    if (fileData.get()->channelCount == 1)
        format = AL_FORMAT_MONO16;
    else
        format = AL_FORMAT_STEREO16;

    alBufferData(buffer,
                 AL_FORMAT_STEREO_FLOAT32, // 这里，需要注意，不能使用上面openAL原生的格式，需要使用alext给我们提供的格式
                 fileData.get()->samples.data(),
                 fileData.get()->samples.size() * sizeof(float),
                 fileData.get()->sampleRate);
    alSourcei(source, AL_BUFFER, buffer);
    alSourcePlay(source);
    system("pause");
    
    alSourceStop(source);
    alDeleteSources(1, &source);
    alDeleteBuffers(1, &buffer);
}
catch (const UnsupportedExtensionEx &e)
{
    std::cerr << "Caught: " << e.what() << std::endl;
    system("pause");
}
catch (const LoadPathNotImplEx &e)
{
    std::cerr << "Caught: " << e.what() << std::endl;
    system("pause");
}
catch (const LoadBufferNotImplEx &e)
{
    std::cerr << "Caught: " << e.what() << std::endl;
    system("pause");
}
catch (const std::exception &e)
{
    std::cerr << "Caught: " << e.what() << std::endl;
    system("pause");
}
```
然后就可以听到MP3音乐了。

# 整合
具体将三种常见格式(WAV,OGG,MP3)整合到一起，放在项目中使用OpenAL操作，可以看我的[游戏](https://github.com/voidvvv/LinkA)中音频部分的实现。
按照这个思路，其实也可以把更多的音频文件解析给整合进来。