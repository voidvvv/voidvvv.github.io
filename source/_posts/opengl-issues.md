---
title: OpenGL 学习中遇到的issue
date: 2024-03-04 22:32:16
categories: 
- c++
- OpenGL
- issue
tags: 
- [c++]
- [OpenGL]
---

![2024-03-02T220734](2024-03-02T220734.png)
纯粹为了记录自己的愚蠢
<!-- more -->
- [2024-03-04](#2024-03-04)
- [2024-03-05 凌晨](#2024-03-05-凌晨)
- [03-10 glVertexAttribPointer](#03-10-glvertexattribpointer)
- [2024-03-12 凌晨](#2024-03-12-凌晨)
- [2024-04-07 记录一次动态设置顶点数组的问题排查](#2024-04-07-记录一次动态设置顶点数组的问题排查)


# 2024-03-04
本来想复习一下周末的学习成果的，在没有使用EBO的时候，很开熏，代码一切正常。
直到我使用了EBO并且脑子抽了。代码如下：
```c++
#include <iostream>
#include <glad/glad.h>
#include <GLFW/glfw3.h>
#include "MyShader.h"

#define SCREEN_WIDTH 800
#define SCREEN_HEIGH 600

void framebuffer_size_callback(GLFWwindow* window, int width, int height);
void processInput(GLFWwindow* window);

#define DEFAULT_VERTEX_FILE_PATH "./vertex01.vert"
#define DEFAULT_FRAGMENT_FILE_PATH "./fragment01.frag"

int test_0304() {


    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

    GLFWwindow* window = glfwCreateWindow(SCREEN_WIDTH, SCREEN_HEIGH, "LearnOpenGL", NULL, NULL);
    if (window == NULL)
    {
        std::cout << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }
    // 创建窗口
    glfwMakeContextCurrent(window);
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);//注册回调

    // 检测窗口是否正常启动，传入内置方法
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
    {
        std::cout << "Failed to initialize GLAD" << std::endl;
        return -1;
    }
    float vertices[] = {
        // 位置              // 颜色
         0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,   // 右下
        -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,   // 左下
         0.5f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f ,   // 顶部
         -0.5f, 0.5f, 0.0f,   0.f, 0.f, 0.f,
         //0.5f, -0.5f, 0.0f,
         //-0.5f, -0.5f, 0.0f,
         // 0.0f,  0.5f, 0.0f,
         // -0.5f, 0.5f, 0.0f,
    };

     float indices[] = {
        0,1,2,
        2,3,1
    };

    GLuint VAO = 0, VBO = 0, EBO = 0;
    // 注册三个data
    glGenVertexArrays(1,&VAO);// 两个参数，第一个代表要几个
    glGenBuffers(1, &VBO);//VBO
    glGenBuffers(1, &EBO);
    glBindVertexArray(VAO);// 注册后需要立刻绑定，因为后面的操作要绑定到当前的VAO上面
    glBindBuffer(GL_ARRAY_BUFFER,VBO);
    glBufferData(GL_ARRAY_BUFFER,sizeof(vertices), vertices, GL_STATIC_DRAW);
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
    glVertexAttribPointer(0,3, GL_FLOAT,GL_FALSE,6*sizeof(float),(void*)0);
    glEnableVertexAttribArray(0); // 定义一个参数并启用
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6* sizeof(float), (void*)(3 * sizeof(float)));
    glEnableVertexAttribArray(1); // 定义一个参数并启用




    int success;
    char infoLog[1024];

    // shader
    GLuint shaderProgram = glCreateProgram();
    GLuint vertexShader = glCreateShader(GL_VERTEX_SHADER);
    GLuint fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);

    std::string vertexSourceCoded;
    const char* vertexSourceCodedC;
    std::string fragmentSourceCode;
    const char* fragmentSourceCodeC;

    std::ifstream vShaderFile;
    std::ifstream fShaderFile;

    vShaderFile.open(DEFAULT_VERTEX_FILE_PATH);
    std::stringstream vst;
    vst << vShaderFile.rdbuf();
    vShaderFile.close();
    vertexSourceCoded = vst.str();
    vertexSourceCodedC = vertexSourceCoded.c_str();
    std::stringstream vst2;
    fShaderFile.open(DEFAULT_FRAGMENT_FILE_PATH);
    vst2 << fShaderFile.rdbuf();
    fShaderFile.close();
    fragmentSourceCode = vst2.str();
    std::cout << fragmentSourceCode << std::endl;

    fragmentSourceCodeC = fragmentSourceCode.c_str();

    glShaderSource(vertexShader,1,&vertexSourceCodedC,NULL);
    glCompileShader(vertexShader);
    glGetShaderiv(vertexShader,GL_COMPILE_STATUS,&success);

    if (!success) {
        glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
        return -1;
    }

    glShaderSource(fragmentShader,1,&fragmentSourceCodeC,NULL);
    glCompileShader(fragmentShader);
    glGetShaderiv(fragmentShader,GL_COMPILE_STATUS,&success);
    if (!success) {
        glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::FRAGMENT::COMPILATION_FAILED\n" << infoLog << std::endl;
        return -1;
    }

    glAttachShader(shaderProgram,vertexShader);
    glAttachShader(shaderProgram,fragmentShader);
    glLinkProgram(shaderProgram);

    glGetProgramiv(shaderProgram,GL_LINK_STATUS,&success);
    if (!success) {
        glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" << infoLog << std::endl;
    }
    glDeleteShader(fragmentShader);
    glDeleteShader(vertexShader);

    //主循环
    while (!glfwWindowShouldClose(window)) {
        // do something...
        processInput(window);
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT); // 清屏
        
        glUseProgram(shaderProgram);
        glBindVertexArray(VAO);         
        //glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);
        //glDrawArrays(GL_TRIANGLES,0,3);
        //glDrawElements(GL_TRIANGLES,6, GL_UNSIGNED_INT,0);
        glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);

        glfwSwapBuffers(window);
        glfwPollEvents();
    }
    glDeleteVertexArrays(1, &VAO);
    glDeleteBuffers(1, &VBO);
    glDeleteBuffers(1, &EBO);
    glDeleteProgram(shaderProgram);
    glfwTerminate();
    return 0;
}
```

一开始我还没看出问题，直到运行时，怎么也出不来想要的图像。我就感觉事态不对了。各种翻看昨天的笔记，代码，逐行对照，都没有结果。
**在尝试搜索答案未果，进群找人无人理睬的情况下，我开始了把昨天的代码逐行复制到这里查看异常。最后结果出来了：**
```c++
    // 这里是int, int,int
    // float indices[] = {
    unsigned int indices[] = {
        0,1,2,
        2,3,1
    };
```
我这里定义成了float。。
这里，indices代表参数下标，所以必须是整数int，我不知道脑子抽什么风了把这里写成了float。
以此为戒吧。被自己蠢哭

# 2024-03-05 凌晨
又遇见一个问题。这个问题是不熟悉C++导致的。
当我读取shader源码准备编译时，出现了shader编译报错，代码如下:
```c++
	const char* fragmentSourceCodeC;
	std::stringstream ss2;
	std::ifstream fragmentFileHandle;
	fragmentFileHandle.open(DEFAULT_FRAGMENT_FILE_PATH);

	ss2 << fragmentFileHandle.rdbuf();
	fragmentFileHandle.close();

	fragmentSourceCodeC = ss2.str().c_str();
```
一直编译shader失败。我就打印了`fragmentSourceCodeC`看了下，结果全是乱码汉字。但是这段代码跟之前的唯一区别就是我把生成C字符串省略成一个方法了。
当代码改回来的时候，一切就正常了;
```c++
	std::string fragmentSourceCode;
	const char* fragmentSourceCodeC;
	std::stringstream ss2;
	std::ifstream fragmentFileHandle;
	fragmentFileHandle.open(DEFAULT_FRAGMENT_FILE_PATH);

	ss2 << fragmentFileHandle.rdbuf();
	fragmentFileHandle.close();

	fragmentSourceCode = ss2.str();
	fragmentSourceCodeC = fragmentSourceCode.c_str();
```
暂不明确原因，应该是与c++底层的实现有关.

# 03-10 glVertexAttribPointer
记录一个坑，表现是渲染出的3d物体能够正常显示模型 ，但是无法显示显示纹理材质。
原因是这个方法：
```c++
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 5 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);

    glVertexAttribPointer(1, 2, GL_FLOAT, GL_FALSE, 5 * sizeof(float), (void*)(3 * sizeof(float)));
    glEnableVertexAttribArray(1);
```

<span style='color:red'>
    这里面的最后一个参数，起始位置偏移量需要强转成(void*),并且，<span style='font-weight:bold'>长度是参数个数 × 参数类型长度</span>
    </span>

# 2024-03-12 凌晨
两个低级错误：
1. 学习光照渲染的时候，最后一步渲染出来的界面只有光源cube，没有被照亮物体。将被光源渲染代码去掉发现屏幕只会渲染没有经过矩阵变换的原始vertex顶点，并且颜色为白色。排查了好半天，最后跟教程源码比对发现，片段着色器居然少加了一个in参数，这个参数本应该是从顶点着色器传入给片段着色器的，但是片段着色器没有声明这个参数，导致渲染异常。但是这个异常很诡异。大晚上的着实吓着我了。
2. 对矩阵不熟悉。在c++代码中使用位移向量构造灯源的model矩阵后，为了把灯源位置传入给物体着色器，我在代码中又使用变换后的transform矩阵乘了位移向量然后转成vec3传入，导致最后的结果偏差的有些离谱.

# 2024-04-07 记录一次动态设置顶点数组的问题排查
在写自己的[小游戏](https://github.com/voidvvv/LinkA)的时候，我想要添加一个功能，就是在两个方块相连接消除的时候，显示两个方块相连的路线。
算法方面，我选择了AStar寻路算法，这方面写的头晕眼花，但是问题不大。
最后渲染的时候，却无论如何也渲染不出来。
我是手写了一个basicRender来渲染的，所以理所当然的我就怀疑是我的render写的有问题,下面就是我的render:
```c++
#ifndef __BASICRENDER_H__
#define __BASICRENDER_H__

#include <glad/glad.h>
#include <GLFW/glfw3.h>
#include <iostream>

#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>
#include "Camera.h"

#include "Shader.h"



// render use for rendering basic shape, include line, dot and rectangle
class BasicRender
{
private:
    float vertex[5000];
    int vIndex;
    int vSize;
    GLuint VAO;
    GLuint VBO;
    ShaderProgram *shader; 
    void setVec3(glm::vec3);
    unsigned int vertexCount;

public:
    BasicRender(ShaderProgram *shader);

    void initialData();

    void drawLine(glm::vec3 start,glm::vec3 end, Camera* camera , glm::vec3 color1 = glm::vec3(1.f), glm::vec3 color2 = glm::vec3(1.f));
    void drawLine(glm::vec2 start,glm::vec2 end, Camera* camera , glm::vec3 color1 = glm::vec3(1.f), glm::vec3 color2 = glm::vec3(1.f));

    // void drawFillLine();
};

#endif // __BASICRENDER_H__
```

cpp:
```c++
#include "BasicRender.h"

#define Point GL_POINTS
#define Line GL_LINES
#define Filled GL_TRIANGLES

void BasicRender::setVec3(glm::vec3 v3)
{
    vertex[vIndex++] = v3[0];
    vertex[vIndex++] = v3[1];
    vertex[vIndex++] = v3[2];
    vertexCount++;
}

BasicRender::BasicRender(ShaderProgram *_shader)
    : shader(_shader), VAO(0)
{
    vertexCount = 0;
    vIndex = 0;
}

void BasicRender::initialData()
{

    glGenVertexArrays(1, &VAO);
    glBindVertexArray(VAO);
    glGenBuffers(1, &VBO);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, 600 * sizeof(float), NULL, GL_DYNAMIC_DRAW);
    // postion
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void *)0);
    glEnableVertexAttribArray(0);
    // color
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void *)(3 * sizeof(float)));
    glEnableVertexAttribArray(1);

    // glBindBuffer(GL_ARRAY_BUFFER,0);
    glBindVertexArray(0);
}

void BasicRender::drawLine(glm::vec3 start, glm::vec3 end,

                           Camera *camera, glm::vec3 color1, glm::vec3 color2)
{
    vertexCount = 0;
    vIndex = 0;
    // bind matrix
    shader->use();
    glBindVertexArray(VAO);
    shader->setUniformMat4("projection", camera->getProjectionMatrix());
    // shader->setUniformMat4("projection", glm::mat4(1.f));

    shader->setUniformMat4("view", camera->getViewMatrix());
    // shader->setUniformMat4("view", glm::mat4(1.f));

    shader->setUniformMat4("model", glm::mat4(1.f));

    // position
    setVec3(start);
    // setVec3(glm::vec3(0.f));
    setVec3(color1);

    setVec3(end);
    // setVec3(glm::vec3(1.f));
    setVec3(color2);


    glBufferSubData(GL_ARRAY_BUFFER, 0, vIndex * sizeof(float), vertex);

    glDrawArrays(GL_LINES, 0, 2);

    glBindVertexArray(0);
}

void BasicRender::drawLine(glm::vec2 start, glm::vec2 end, Camera *camera, glm::vec3 color1,
                           glm::vec3 color2)
{
    drawLine(glm::vec3(start, 0.f), glm::vec3(end, 0.f), camera, color1, color2);
}
```
我就怀疑是动态设置顶点数组无法这么`glBufferSubData`设置,就各种搜索，各种改。但是还是没有改变，仍然无法渲染。
最后我开始怀疑不是render的问题了，为了确认我的想法，于是我就把顶点在方法设置中写死，view和projection矩阵设置为标准矩阵（上面的注释里的内容）,然后开始渲染，神奇的是，竟然成功的渲染出了一条线段。
于是乎，我确认了我的render是没有问题的，自然我就开始查找我算法获取到的路径了。
首先，我加了一个标准输出来查看找出的路劲中心点信息：
```c++
void printPath(std::vector<Card*> path){
    std::cout<< "----"<<std::endl;
    for (Card* c:path){
        std::cout<< c->center.x<< " - "<< c->center.y <<std::endl;
    }
        

    std::cout<< "----"<<std::endl;
};
```
然后我在查找路径出来的地方来打印信息:
```c++
                bool b = outer->pathFinder->searchNodePath(extraCard.get(), selecedCard.get(), Game_Heuristic, Game_ShouldStop, outer->linkAPath);
                std::cout << "A: X - [" << extraCard.get()->x << "]  Y - [" << extraCard.get()->y << "]  B: x - [" << selecedCard.get()->x << "]   y- [" << selecedCard.get()->y << "] reseult: " << b << std::endl;

                if (b)
                {
                    std::cout << "outer->linkAPath size: " << outer->linkAPath.size() << std::endl;
                    printPath(outer->linkAPath);
                    outer->showPath = true;
                    events->sendMessaage(_CARD_SUCCESS_MATCH, NULL, extraCard.get(), outer);
                    events->sendMessaage(_CARD_SUCCESS_MATCH, NULL, selecedCard.get(), outer);
                }
```
结果令我大吃一惊，所有节点的中心坐标完全一样，也就是说，之前并不是没有渲染路径，而是路径全部渲染到了一个像素点上，我看不到。
然后我就跑到给card设置中心点的位置看了下:
```C++
        std::shared_ptr<Card> objPtr = objs[i];
        int c_col = i % column;
        int c_row = i / column;

        objPtr.get()->position.x = cardGapx * (c_col + 1) + cardWidth * c_col + cardsOrigin.x;
        objPtr.get()->position.y = cardGapy * (c_row + 1) + c_row * cardHeight + cardsOrigin.y;

        objPtr.get()->size.x = cardWidth;
        objPtr.get()->size.y = cardHeight;

        // 啊 这
        objPtr.get()->center.x = position.x + cardWidth/2;
        objPtr.get()->center.y = position.y + cardHeight/2;
```
这里的position是我Board的position，也就是整个连连看地盘的position，每个card都这么设置，那么每个card自然都是同样的坐标。正确的设置应该是:
```c++
        objPtr.get()->center.x = objPtr.get()->position.x + cardWidth/2;
        objPtr.get()->center.y = objPtr.get()->position.y + cardHeight/2;
```
然后一段困扰我一下午的排查就告一段落了。哎，粗心大意害死人
