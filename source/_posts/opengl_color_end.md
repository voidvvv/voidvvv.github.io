---
title: opengl 光照(2)
date: 2024-03-23T16:16:40+08:00
categories: 
- c++
- OpenGL
tags: 
- [c++]
- [OpenGL]
---
![2024-03-02T220734](2024-03-02T220734.png)
opengl 颜色，光照学习
<!-- more -->

# 投光物(光源)
我们目前使用的光照都来自于空间中的一个点。它能给我们不错的效果，但现实世界中，我们有很多种类的光照，每种的表现都不同。将光投射(Cast)到物体的光源叫做投光物(Light Caster)。在这一节中，我们将会讨论几种不同类型的投光物。学会模拟不同种类的光源是又一个能够进一步丰富场景的工具。

我们首先将会讨论定向光(Directional Light)，接下来是点光源(Point Light)，它是我们之前学习的光源的拓展，最后我们将会讨论聚光(Spotlight)。在下一节中我们将讨论如何将这些不同种类的光照类型整合到一个场景之中。

## 平行光
当一个光源处于很远的地方时，来自光源的每条光线就会近似于互相平行。不论物体和/或者观察者的位置，看起来好像所有的光都来自于同一个方向。当我们使用一个假设光源处于无限远处的模型时，它就被称为定向光，因为它的所有光线都有着相同的方向，它与光源的位置是没有关系的。

定向光非常好的一个例子就是太阳。太阳距离我们并不是无限远，但它已经远到在光照计算中可以把它视为无限远了。所以来自太阳的所有光线将被模拟为平行光线，我们可以在下图看到：

![2024-03-23T161923](2024-03-23T161923.png)

因为平行光假定是跟距离没有关系的，所以平行光唯一的一个常量就是方向。

```glsl
struct Light {
    // vec3 position; // 使用定向光就不再需要了
    vec3 direction;

    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};
...
void main()
{
  vec3 lightDir = normalize(-light.direction);
  ...
}
```
同样的，我们的平行光也是一种光，所以也有三种分量：环境分量，漫反射分量，以及镜面反射分量。
这里需要注意，我们定义Light中的 direction，一般是光发散出去的方向。但是我们计算各个分量的时候，需要用到由物体到光的方向，所以我们这里需要对光方向取反.

这里，如果我们在场景中尝试渲染多个不同的立方体，那么我们会看到冲着光方向的那面会变的更亮，反之更暗：
![2024-03-23T163256](2024-03-23T163256.png)

## 点光源
我们之前介绍光源的额时候，使用的都是一个模拟的点光源，我们平时生活中的灯泡也可而已算作点光源。点光源对一个物体照亮的程度是不一样的，离点光源越近的地方会更亮一些：
![2024-03-23T165605](2024-03-23T165605.png)
在之前的教程中，我们一直都在使用一个（简化的）点光源。我们在给定位置有一个光源，它会从它的光源位置开始朝着所有方向散射光线。然而，我们定义的光源模拟的是永远不会衰减的光线，这看起来像是光源亮度非常的强。在大部分的3D模拟中，我们都希望模拟的光源仅照亮光源附近的区域而不是整个场景。

如果你将10个箱子加入到上一节光照场景中，你会注意到在最后面的箱子和在灯面前的箱子都以相同的强度被照亮，并没有定义一个公式来将光随距离衰减。我们希望在后排的箱子与前排的箱子相比仅仅是被轻微地照亮。

### 衰减
随着光线传播距离的增长逐渐削减光的强度通常叫做衰减(Attenuation)。随距离减少光强度的一种方式是使用一个线性方程。这样的方程能够随着距离的增长线性地减少光的强度，从而让远处的物体更暗。然而，这样的线性方程通常会看起来比较假。在现实世界中，灯在近处通常会非常亮，但随着距离的增加光源的亮度一开始会下降非常快，但在远处时剩余的光强度就会下降的非常缓慢了。所以，我们需要一个不同的公式来减少光的强度。

幸运的是一些聪明的人已经帮我们解决了这个问题。下面这个公式根据片段距光源的距离计算了衰减值，之后我们会将它乘以光的强度向量：
![2024-03-23T165733](2024-03-23T165733.png)
![2024-03-23T165742](2024-03-23T165742.png)
![2024-03-23T165745](2024-03-23T165745.png)
你可以看到光在近距离的时候有着最高的强度，但随着距离增长，它的强度明显减弱，并缓慢地在距离大约100的时候强度接近0。这正是我们想要的。

**选择正确的值**
但是，该对这三个项设置什么值呢？正确地设定它们的值取决于很多因素：环境、希望光覆盖的距离、光的类型等。在大多数情况下，这都是经验的问题，以及适量的调整。下面这个表格显示了模拟一个（大概）真实的，覆盖特定半径（距离）的光源时，这些项可能取的一些值。第一列指定的是在给定的三项时光所能覆盖的距离。这些值是大多数光源很好的起始点，它们由Ogre3D的Wiki所提供：
![2024-03-23T170103](2024-03-23T170103.png)

这里建议直接参考这个表抄数值即可。
接下来在glsl中定义一个点光源类型：
```glsl
struct Light {
    vec3 position;  

    vec3 ambient;
    vec3 diffuse;
    vec3 specular;

    float constant;
    float linear;
    float quadratic;
};
```
c++中需要把对应的属性赋值:

```c++
lightingShader.setFloat("light.constant",  1.0f);
lightingShader.setFloat("light.linear",    0.09f);
lightingShader.setFloat("light.quadratic", 0.032f);
```
glsl中添加计算衰减的方法：
```glsl
float distance    = length(light.position - FragPos);
float attenuation = 1.0 / (light.constant + light.linear * distance +  light.quadratic * (distance * distance));
```

最后把结果的三个分量乘这个强度:
```glsl
ambient  *= attenuation; 
diffuse  *= attenuation;
specular *= attenuation;
```
现在，我们的点光源就完成了

## 聚光
我们要讨论的最后一种类型的光是聚光(Spotlight)。聚光是位于环境中某个位置的光源，它只朝一个特定方向而不是所有方向照射光线。这样的结果就是只有在聚光方向的特定半径内的物体才会被照亮，其它的物体都会保持黑暗。聚光很好的例子就是路灯或手电筒。

OpenGL中聚光是用一个世界空间位置、一个方向和一个切光角(Cutoff Angle)来表示的，切光角指定了聚光的半径（译注：是圆锥的半径不是距光源距离那个半径）。对于每个片段，我们会计算片段是否位于聚光的切光方向之间（也就是在锥形内），如果是的话，我们就会相应地照亮片段。下面这张图会让你明白聚光是如何工作的：
![2024-03-23T170859](2024-03-23T170859.png)

我们这里需要比较两个角度，一个是

手电筒就是一个典型的聚光灯

### 手电筒
手电筒(Flashlight)是一个位于观察者位置的聚光，通常它都会瞄准玩家视角的正前方。基本上说，手电筒就是普通的聚光，但它的位置和方向会随着玩家的位置和朝向不断更新。

所以，在片段着色器中我们需要的值有聚光的位置向量（来计算光的方向向量）、聚光的方向向量和一个切光角。我们可以将它们储存在Light结构体中：

```glsl
struct Light {
    vec3  position;
    vec3  direction;
    float cutOff;
    ...
};
```
接下来我们将合适的值传到着色器中：
```c++
lightingShader.setVec3("light.position",  camera.Position);
lightingShader.setVec3("light.direction", camera.Front);
lightingShader.setFloat("light.cutOff",   glm::cos(glm::radians(12.5f)));
```
需要注意，我们这里传入的并不是角度，而是角度的cos。因为我们上面可以看到，我们需要比较片段-光源与光源方向指夹角。但是在glsl中，我们有dot叉乘向量的操作，这个操作得到的就是向量间的cos。所以我们这里传入cos的话可以直接放在glsl中比较叉乘结果即可。

逻辑如下:
```glsl
float theta = dot(lightDir, normalize(-light.direction));

if(theta > light.cutOff) 
{       
  // 执行光照计算
}
else  // 否则，使用环境光，让场景在聚光之外时不至于完全黑暗
  color = vec4(light.ambient * vec3(texture(material.diffuse, TexCoords)), 1.0);
```
由于我们使用的是cos比较，而cos在0-90°区间是递减的，所以我们应当保证当前片段大于光源的cutOff时，来渲染光效。

### 平滑/软化边缘
如果我们上面成功的写出了一个聚光，那么我们会看到聚光的周围有些不真实：
![2024-03-23T172606](2024-03-23T172606.png)
因为我们真实的聚光边缘肯定是要有一定的平滑过渡的。
所以我们要渐进式的对光渲染,除了outoff外，我们还需要一个outcusoff来表示光外围的角度，我们需要对角度进行判断段，如果角度cos大于cutoff，那么全量渲染，如果角度落在了cutoff与outcutoff之间，则需要根据之间的比例来确定不同的光亮强度.

![2024-03-23T172927](2024-03-23T172927.png)

```glsl
float theta     = dot(lightDir, normalize(-light.direction));
float epsilon   = light.cutOff - light.outerCutOff;
float intensity = clamp((theta - light.outerCutOff) / epsilon, 0.0, 1.0);    
...
// 将不对环境光做出影响，让它总是能有一点光
diffuse  *= intensity;
specular *= intensity;
...
```

# 多光源
多光源就是假设场景中有多个光源，我们要做的就是把他们的光照影响叠加输出。
需要注意，每一种光都同时具有三个分量：环境，漫反射，镜面反射。所以我们可以尝试抽象出来三个方法来分别对其计算。
这里直接贴出我自己写的代码:
片段着色器:
```glsl
#version 330 core
in vec3 FragPos;  
in vec3 Normal;
in vec2 texCoord;

out vec4 FragColor;

struct Material{
    sampler2D diffuse;
    sampler2D specular;
    float shininess;
};

struct DirectionalLight{
    vec3 direction;
    vec3 color;
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};

struct PointLight{
    vec3 position;
    vec3 color;
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;

    float constant;
    float linear;
    float quadratic;
};

struct PotLight {
    vec3  position;
    vec3  direction;
    float cutOff;
    float outerCutOff;

    vec3 color;
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;

    float constant;
    float linear;
    float quadratic;
};

#define NR_POINT_LIGHTS 4

uniform Material material;
uniform vec3 viewPos;

uniform DirectionalLight dirLight;
uniform PointLight pointLights[NR_POINT_LIGHTS];
uniform PotLight potLight ;



vec3 ambientCalc (vec3 textureVec3, vec3 ambient) {
    return textureVec3 * ambient;
}

vec3 diffuseCalc(vec3 textureVec3, vec3 diffuse, vec3 norm, vec3 lightDir){
    float diff = max(dot(norm, lightDir), 0.0);// 点乘，获取夹角的cos值，角度越大，cos越小，光亮效果越小
    return  diffuse *(diff * textureVec3); // 点乘结果跟灯光相乘
}

vec3 specularCalc(vec3 textureVec3, vec3 specular, vec3 viewDir, vec3 reflectDir){
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
    return specular * (spec *  textureVec3 );
}

vec3 dirLightCalc(DirectionalLight dirLight, vec3 basicDiffuse , vec3 basicSpecular,vec3 norm, vec3 viewDir){
    vec3 lightDir = normalize(-dirLight.direction);
    vec3 reflectDir = reflect(-lightDir, norm);


    vec3 ambient = ambientCalc(basicDiffuse, dirLight.ambient);
    vec3 diffuse =  diffuseCalc(basicDiffuse, dirLight.diffuse,norm, lightDir);
    vec3 specular = specularCalc(basicSpecular, dirLight.specular,viewDir, reflectDir);

    ambient*=dirLight.color;
        diffuse*=dirLight.color;

    specular*=dirLight.color;

    return ambient + diffuse + specular;
}

vec3 pointLightCalc(PointLight pointLight, vec3 basicDiffuse , vec3 basicSpecular, vec3 fragPos, vec3 norm, vec3 viewDir ){
    vec3 lightDir = normalize(pointLight.position - fragPos);
    vec3 reflectDir = reflect(-lightDir, norm);

    float distance    = length(pointLight.position - FragPos);
    float attenuation = 1.0 / (pointLight.constant + pointLight.linear * distance + 
                pointLight.quadratic * (distance * distance));

    vec3 ambient = ambientCalc(basicDiffuse, pointLight.ambient);
    vec3 diffuse =  diffuseCalc(basicDiffuse, pointLight.diffuse,norm, lightDir);
    vec3 specular = specularCalc(basicSpecular, pointLight.specular,viewDir, reflectDir);

    ambient*=pointLight.color;
        diffuse*=pointLight.color;

    specular*=pointLight.color;

    ambient *= attenuation;
    diffuse *= attenuation;
    specular *= attenuation;

    return ambient + diffuse + specular;
}

vec3 potLightCalc(PotLight potLight, vec3 basicDiffuse , vec3 basicSpecular,vec3 fragPos, vec3 norm, vec3 viewDir ){
    vec3 lightDir = normalize(potLight.position - fragPos);
    vec3 reflectDir = reflect(-lightDir, norm);


    float distance    = length(potLight.position - FragPos);
    float attenuation = 1.0 / (potLight.constant + potLight.linear * distance + 
                potLight.quadratic * (distance * distance));



    vec3 ambient = ambientCalc(basicDiffuse, potLight.ambient);
    vec3 diffuse =  diffuseCalc(basicDiffuse, potLight.diffuse,norm, lightDir);
    vec3 specular = specularCalc(basicSpecular, potLight.specular,viewDir, reflectDir);

    // spotlight (soft edges)
    float theta = dot(lightDir, normalize(-potLight.direction)); 
    float epsilon = (potLight.cutOff - potLight.outerCutOff);
    float intensity = clamp((theta - potLight.outerCutOff) / epsilon, 0.0, 1.0);
    diffuse  *= intensity;
    specular *= intensity;

    ambient*=potLight.color;
        diffuse*=potLight.color;

    specular*=potLight.color;

    ambient *= attenuation;
    diffuse *= attenuation;
    specular *= attenuation;

    return ambient + diffuse + specular;
}


void main()
{
    vec3 basicDiffuse = vec3(texture(material.diffuse,texCoord)); 
    vec3 basicSpecular = vec3(texture(material.specular,texCoord));

    vec3 norm = normalize(Normal); // 标准化法向量
    vec3 viewDir = normalize(viewPos - FragPos);

    vec3 result = vec3(0);
    result = dirLightCalc(dirLight, basicDiffuse, basicSpecular, norm, viewDir);

    for(int i = 0; i < 1; i++)
        result += pointLightCalc(pointLights[i], basicDiffuse, basicSpecular, FragPos,norm, viewDir );
    
    result += potLightCalc(potLight, basicDiffuse, basicSpecular, FragPos,norm, viewDir );

    FragColor = vec4(result,1.0);
}
```
顶点着色器:
```glsl
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aNormal;
layout (location = 2) in vec2 aTexCoord;


uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;
uniform mat3 reverseModel;

out vec3 FragPos;  
out vec3 Normal;
out vec2 texCoord;

void main()
{
    gl_Position = projection * view * model * vec4(aPos, 1.0);
    FragPos = vec3(model * vec4(aPos, 1.0));
    Normal = reverseModel * aNormal;
    texCoord = aTexCoord;
}
```

c++代码:
```c++
#include "my_glfw.h"
#include "MyPerspectCamera.h"
#include "MyShader.h"
#include "myTexture.h"
#include "my_camera_input_procecssor.h"
#include "DirectLight.h"
#include "PointLight.h"
#include "SpotLight.h"
#include "MyMaterial.h"

extern MyPerspectCamera myPerspectCamera;
extern GLFWwindow* window;

void debug(int x) {
	if (x % 1000 == 0) {
		std::cout << "========= x ========= start ==========" << std::endl;
		glCheckError();
		std::cout << "========= x ========= end ==========" << std::endl;

	}
}

int test0318() {
	// D:\files\doc\shaders\colortest
	initGlfw();
	const char* vertexPath = "D:\\files\\doc\\shaders\\colortest\\mix\\vertex.glsl";
	const char* fragPath = "D:\\files\\doc\\shaders\\colortest\\mix\\frag.glsl";

	const char* diffuseTexturePath = "C:\\Users\\voidvvv\\Pictures\\test\\container2.png";
	const char* specularTexturePath = "C:\\Users\\voidvvv\\Pictures\\test\\container2_specular.png";

	window = glfwCreateWindow(SCREEN_WIDTH, SCREEN_HEIGH, "hello 0314", NULL, NULL);

	std::cout << window << std::endl;
	glfwMakeContextCurrent(window);
	glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
	int f = gladLoadGLLoader((GLADloadproc)glfwGetProcAddress);

	glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_DISABLED);

	myPerspectCamera = MyPerspectCamera(VEC3(1, 1, -3.f), VEC3(1, 1, 0), VEC3(0, 1, 0));
	regist_my_camera();

	float vertex[] = {
		// positions          // normals           // texture coords
		-0.5f, -0.5f, -0.5f,  0.0f,  0.0f, -1.0f,  0.0f, 0.0f,
		 0.5f, -0.5f, -0.5f,  0.0f,  0.0f, -1.0f,  1.0f, 0.0f,
		 0.5f,  0.5f, -0.5f,  0.0f,  0.0f, -1.0f,  1.0f, 1.0f,
		 0.5f,  0.5f, -0.5f,  0.0f,  0.0f, -1.0f,  1.0f, 1.0f,
		-0.5f,  0.5f, -0.5f,  0.0f,  0.0f, -1.0f,  0.0f, 1.0f,
		-0.5f, -0.5f, -0.5f,  0.0f,  0.0f, -1.0f,  0.0f, 0.0f,

		-0.5f, -0.5f,  0.5f,  0.0f,  0.0f, 1.0f,   0.0f, 0.0f,
		 0.5f, -0.5f,  0.5f,  0.0f,  0.0f, 1.0f,   1.0f, 0.0f,
		 0.5f,  0.5f,  0.5f,  0.0f,  0.0f, 1.0f,   1.0f, 1.0f,
		 0.5f,  0.5f,  0.5f,  0.0f,  0.0f, 1.0f,   1.0f, 1.0f,
		-0.5f,  0.5f,  0.5f,  0.0f,  0.0f, 1.0f,   0.0f, 1.0f,
		-0.5f, -0.5f,  0.5f,  0.0f,  0.0f, 1.0f,   0.0f, 0.0f,

		-0.5f,  0.5f,  0.5f, -1.0f,  0.0f,  0.0f,  1.0f, 0.0f,
		-0.5f,  0.5f, -0.5f, -1.0f,  0.0f,  0.0f,  1.0f, 1.0f,
		-0.5f, -0.5f, -0.5f, -1.0f,  0.0f,  0.0f,  0.0f, 1.0f,
		-0.5f, -0.5f, -0.5f, -1.0f,  0.0f,  0.0f,  0.0f, 1.0f,
		-0.5f, -0.5f,  0.5f, -1.0f,  0.0f,  0.0f,  0.0f, 0.0f,
		-0.5f,  0.5f,  0.5f, -1.0f,  0.0f,  0.0f,  1.0f, 0.0f,

		 0.5f,  0.5f,  0.5f,  1.0f,  0.0f,  0.0f,  1.0f, 0.0f,
		 0.5f,  0.5f, -0.5f,  1.0f,  0.0f,  0.0f,  1.0f, 1.0f,
		 0.5f, -0.5f, -0.5f,  1.0f,  0.0f,  0.0f,  0.0f, 1.0f,
		 0.5f, -0.5f, -0.5f,  1.0f,  0.0f,  0.0f,  0.0f, 1.0f,
		 0.5f, -0.5f,  0.5f,  1.0f,  0.0f,  0.0f,  0.0f, 0.0f,
		 0.5f,  0.5f,  0.5f,  1.0f,  0.0f,  0.0f,  1.0f, 0.0f,

		-0.5f, -0.5f, -0.5f,  0.0f, -1.0f,  0.0f,  0.0f, 1.0f,
		 0.5f, -0.5f, -0.5f,  0.0f, -1.0f,  0.0f,  1.0f, 1.0f,
		 0.5f, -0.5f,  0.5f,  0.0f, -1.0f,  0.0f,  1.0f, 0.0f,
		 0.5f, -0.5f,  0.5f,  0.0f, -1.0f,  0.0f,  1.0f, 0.0f,
		-0.5f, -0.5f,  0.5f,  0.0f, -1.0f,  0.0f,  0.0f, 0.0f,
		-0.5f, -0.5f, -0.5f,  0.0f, -1.0f,  0.0f,  0.0f, 1.0f,

		-0.5f,  0.5f, -0.5f,  0.0f,  1.0f,  0.0f,  0.0f, 1.0f,
		 0.5f,  0.5f, -0.5f,  0.0f,  1.0f,  0.0f,  1.0f, 1.0f,
		 0.5f,  0.5f,  0.5f,  0.0f,  1.0f,  0.0f,  1.0f, 0.0f,
		 0.5f,  0.5f,  0.5f,  0.0f,  1.0f,  0.0f,  1.0f, 0.0f,
		-0.5f,  0.5f,  0.5f,  0.0f,  1.0f,  0.0f,  0.0f, 0.0f,
		-0.5f,  0.5f, -0.5f,  0.0f,  1.0f,  0.0f,  0.0f, 1.0f
	};
	GLuint indices[] = {
		1,
	};

	glm::vec3 cubePositions[] = {
	glm::vec3(0.0f,  0.0f,  0.0f),
	glm::vec3(2.0f,  5.0f, -15.0f),
	glm::vec3(-1.5f, -2.2f, -2.5f),
	glm::vec3(-3.8f, -2.0f, -12.3f),
	glm::vec3(2.4f, -0.4f, -3.5f),
	glm::vec3(-1.7f,  3.0f, -7.5f),
	glm::vec3(1.3f, -2.0f, -2.5f),
	glm::vec3(1.5f,  2.0f, -2.5f),
	glm::vec3(1.5f,  0.2f, -1.5f),
	glm::vec3(-1.3f,  1.0f, -1.5f)
	};

	// VAO;
	GLuint objVAO, VBO;
	glGenVertexArrays(1,&objVAO);
	glBindVertexArray(objVAO);

	glGenBuffers(1,&VBO);
	glBindBuffer(GL_ARRAY_BUFFER,VBO);
	glBufferData(GL_ARRAY_BUFFER,sizeof(vertex),vertex,GL_STATIC_DRAW);

	// ATTRIBUTE
	glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void*)0);
	glEnableVertexAttribArray(0);
	glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void*)(3 * sizeof(float)));
	glEnableVertexAttribArray(1);
	glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void*)(6 * sizeof(float)));
	glEnableVertexAttribArray(2);

	GLuint lightVAO;
	glGenVertexArrays(1,&lightVAO);
	glBindVertexArray(lightVAO);
	glBindBuffer(GL_ARRAY_BUFFER, VBO);
	glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void*)0);
	glEnableVertexAttribArray(0);

	glBindVertexArray(0);// unbind vao

	// shader
	MyShader objShader(vertexPath,fragPath);
	//MyShader lightShader("", "");

	myTexture objDiffuse(diffuseTexturePath, GL_RGBA, GL_RGBA);
	myTexture objSpecular(specularTexturePath, GL_RGBA, GL_RGBA);

	Light baseLight1 = Light(glm::vec3(0.2f,0.5f,0.2f), glm::vec3(0.05f, 0.05f, 0.05f), glm::vec3(0.4f, 0.4f, 0.4f), glm::vec3(0.5f, 0.5f, 0.5f));
	Light baseLight2 = Light(glm::vec3(0.5f), glm::vec3(0.05f, 0.05f, 0.05f), glm::vec3(0.8f, 0.8f, 0.8f), glm::vec3(1.0f, 1.0f, 1.0f));
	Light baseLight3 = Light(glm::vec3(0.5f), glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(1.0f, 1.0f, 1.0f), glm::vec3(1.0f, 1.0f, 1.0f));

	DirectLight dirLight = DirectLight(baseLight1,glm::vec3(-0.2f, -1.0f, -0.3f));

	PointLight pointLights[] = {
		PointLight(0,baseLight1,glm::vec3(0.7f,  0.2f,  2.0f),1.0f,0.09f,0.032f),
	};

	SpotLight spotLight = SpotLight(baseLight3,
		&myPerspectCamera,
		glm::cos(glm::radians(12.5f)), glm::cos(glm::radians(15.f)),
		1,0.09f,0.032f);

	MyMaterial myMaterial(&objDiffuse, &objSpecular,32.f);
	int x = 0;
	
	//model = glm::scale(model,glm::vec3(5));
	_MAIN_LOOP{
		debug(x);
		glClearColor(baseLight1.color[0], baseLight1.color[1], baseLight1.color[2], 1.0f);
		glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
		debug(x);
		glEnable(GL_DEPTH_TEST);
		processInput(window);
		// render obj
		objShader.use();
		debug(x);
		glBindVertexArray(objVAO);
		objShader.setMatrix4("view",myPerspectCamera.vieMatrix());
		objShader.setMatrix4("projection",myPerspectCamera.projection());
		objShader.setFloat3("viewPos", myPerspectCamera.Position);
		debug(x);
		// update model maybe

		debug(x);
		dirLight.setToUniform(&objShader);
		debug(x);
		for (PointLight pt : pointLights) {
			pt.setToUniform(&objShader);
		}
		debug(x);
		spotLight.setToUniform(&objShader);
		myMaterial.setToUniform(&objShader);

		for (int i = 0; i < sizeof(cubePositions)/(sizeof(glm::vec3));i++) {
			glm::mat4 model(1.f);
			model = glm::translate(model, cubePositions[i]);
			float angle = 20.0f * i;
			model = glm::rotate(model, glm::radians(angle), glm::vec3(1.0f, 0.3f, 0.5f));
			glm::mat3 reverseModel = glm::mat3(glm::transpose(glm::inverse(model)));
			objShader.setMatrix4("model", model);
			objShader.setMatrix3("reverseModel", reverseModel);
			glDrawArrays(GL_TRIANGLES, 0, 36);
		}
		debug(x);
		// render light

		glfwSwapBuffers(window);
		glfwPollEvents();
		debug(x);
		x++;
	}

	return 0;
}
```
光照类：
```c++
#include "DirectLight.h"

void DirectLight::setToUniform(MyShader* shader) {
	std::string prefix = "dirLight.";

	std::string tmpPrefix = prefix + "color";
	shader->setFloat3(tmpPrefix.c_str(), this->myLight.color[0], this->myLight.color[1], this->myLight.color[2]);

	tmpPrefix = prefix + "ambient";
	shader->setFloat3(tmpPrefix.c_str(), this->myLight.ambient[0], this->myLight.ambient[1], this->myLight.ambient[2]);

	tmpPrefix = prefix + "diffuse";
	shader->setFloat3(tmpPrefix.c_str(), this->myLight.diffuse[0], this->myLight.diffuse[1], this->myLight.diffuse[2]);

	tmpPrefix = prefix + "specular";
	shader->setFloat3(tmpPrefix.c_str(), this->myLight.specular[0], this->myLight.specular[1], this->myLight.specular[2]);

	tmpPrefix = prefix + "direction";
	shader->setFloat3(tmpPrefix.c_str(), this->direction[0], this->direction[1], this->direction[2]);

}
#include "PointLight.h"

void PointLight::setToUniform(MyShader* shader)
{
	std::string prefix = "pointLights[" + std::to_string(this->indices) + "].";
	// color
	std::string tmpPrefix = prefix + "color";
	shader->setFloat3(tmpPrefix.c_str(), this->myLight.color[0], this->myLight.color[1], this->myLight.color[2]);
	// ambient
	tmpPrefix = prefix + "ambient";
	shader->setFloat3(tmpPrefix.c_str(), this->myLight.ambient[0], this->myLight.ambient[1], this->myLight.ambient[2]);
	// diffuse
	tmpPrefix = prefix + "diffuse";
	shader->setFloat3(tmpPrefix.c_str(), this->myLight.diffuse[0], this->myLight.diffuse[1], this->myLight.diffuse[2]);
	// specular
	tmpPrefix = prefix + "specular";
	shader->setFloat3(tmpPrefix.c_str(), this->myLight.specular[0], this->myLight.specular[1], this->myLight.specular[2]);
	// position
	tmpPrefix = prefix + "position";
	shader->setFloat3(tmpPrefix.c_str(), this->position[0], this->position[1], this->position[2]);


	//float constant;
	tmpPrefix = prefix + "constant";
	shader->setFloat(tmpPrefix.c_str(), this->constant);
	//float linear;
	tmpPrefix = prefix + "linear";
	shader->setFloat(tmpPrefix.c_str(), this->linear);
	//float quadratic;
	tmpPrefix = prefix + "quadratic";
	shader->setFloat(tmpPrefix.c_str(), this->quadratic);
}


#include "SpotLight.h"
#include <string>

void SpotLight::setToUniform  (MyShader* shader) {
	std::string prefix = "potLight.";
	// color
	std::string tmpPrefix = prefix + "color";
	shader->setFloat3(tmpPrefix.c_str(),this->myLight.color[0], this->myLight.color[1], this->myLight.color[2] );
	// ambient
	tmpPrefix = prefix + "ambient";
	shader->setFloat3(tmpPrefix.c_str(), this->myLight.ambient[0], this->myLight.ambient[1], this->myLight.ambient[2]);
	// diffuse
	tmpPrefix = prefix + "diffuse";
	shader->setFloat3(tmpPrefix.c_str(), this->myLight.diffuse[0], this->myLight.diffuse[1], this->myLight.diffuse[2]);
	// specular
	tmpPrefix = prefix + "specular";
	shader->setFloat3(tmpPrefix.c_str(), this->myLight.specular[0], this->myLight.specular[1], this->myLight.specular[2]);
	// position
	tmpPrefix = prefix + "position";
	shader->setFloat3(tmpPrefix.c_str(), this->pos->getPosition()[0], this->pos->getPosition()[1], this->pos->getPosition()[2]);
	// specular
	tmpPrefix = prefix + "direction";
	shader->setFloat3(tmpPrefix.c_str(), this->pos->getDirection()[0], this->pos->getDirection()[1], this->pos->getDirection()[2]);
	tmpPrefix = prefix + "cutOff";
	shader->setFloat(tmpPrefix.c_str(), this->cutOff);
	tmpPrefix = prefix + "outerCutOff";
	shader->setFloat(tmpPrefix.c_str(), this->outerCutOff);

	//float constant;
	tmpPrefix = prefix + "constant";
	shader->setFloat(tmpPrefix.c_str(), this->constant);
	//float linear;
	tmpPrefix = prefix + "linear";
	shader->setFloat(tmpPrefix.c_str(), this->linear);
	//float quadratic;
	tmpPrefix = prefix + "quadratic";
	shader->setFloat(tmpPrefix.c_str(), this->quadratic);
}
```


