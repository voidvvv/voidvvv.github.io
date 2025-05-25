---
title: openGL 实现自己的Camera类
date: 2024-03-08T17:55:18+08:00
categories: 
- c++
- OpenGL
tags: 
- [c++]
- [OpenGL]
---

![2024-03-02T220734](2024-03-02T220734.png)
一个小实战
<!-- more -->


写一个perspective cameraa作为小实战
# 头文件
```c++
typedef glm::vec3 VEC3;
class MyPerspectCamera
{
public:
	VEC3 Position ;
	VEC3 Front;
	VEC3 Up ;
	VEC3 WorldUp;
	VEC3 Right;

	// euler Angles
	float Yaw;
	float Pitch;
	// camera options
	float MovementSpeed;
	float MouseSensitivity;
	float Zoom;

	glm::mat4 Projection;

	MyPerspectCamera(VEC3 position = VEC3(0,0,0), VEC3 front = VEC3(0, 0, 0), VEC3 worldUP = VEC3(0, 0, 0));

	void update();

	glm::mat4 vieMatrix();

	glm::mat4 projection();

	void move(CAMERA_DIRECTION , float deltaTime);

	void rotate(float xoffset, float yoffset, GLboolean constrainPitch = true);

	void justZoom(float yoffset);
};

```
根据之前的学习，我们知道，想要渲染一个3d物体，我们需要几个矩阵：model，view，projection。
其中，model矩阵是物体本身的属性，我们相机不负责。剩下的两个，分别是view 和projection是我们的相机可以控制的

## view
为了构造view矩阵，我们需要知道：当前相机的位置，相机所对的方向，相机的上方（定位）：
```c++
	VEC3 Position ;
	VEC3 Front;
	VEC3 Up ;
```
构造方法中，传入这三个参数。

## method
我们的摄像机类，还需要几个方法。
### move
因为我们的相机想要移动，所以定义一个move方法，方法参数两个，一个是移动方向，另一个是移动持续时间（我们这里一般是当前帧与上一帧的时间差，保证不同渲染性能下表现是一样的）。我们这里同时定义一个移动方向枚举：
```c++
enum CAMERA_DIRECTION
{
	CAMERA_UP, CAMERA_DOWN, CAMERA_LEFT, CAMERA_RIGHT
};

	void move(CAMERA_DIRECTION , float deltaTime);
```
### rotate
我们的相机也需要能够改变视角。类似于一个fps游戏中的第一人称视角控制那种。
转动需要知道在x轴上以及y轴上转动的角度。为什么不管z轴呢？因为我们还不想做歪头的那种效果。就先管x,y暂时够了
```c++
	void rotate(float xoffset, float yoffset, GLboolean constrainPitch = true);
```
在这里我们有另外一个参数，表示是否要限制当前的角度。试想一下，正常的fps游戏中玩家的视角转动肯定是有一定限制的，尤其是在y轴上，就是说抬头，最多抬到90°。这里的这个参数就是用来限制的。

## projection
view相关的操作就先这么多。我们还有另外一个矩阵，projection。
在projection中，我们只想控制相机的`聚焦`，就是 `glm::perspect`方法的第一个参数。并且这个参数是用鼠标滚轮来控制的，所以我们还需要监控系统鼠标滚轮的变化.
```c++
	void justZoom(float yoffset);
```

## other
最后，我们要有一个update方法来更新我们的矩阵。还有另外两个get方法把view和projection矩阵获取到

## 实现
```c++
#include "MyPerspectCamera.h"


#define SCREEN_WIDTH 800
#define SCREEN_HEIGH 600

extern enum CAMERA_DIRECTION;

MyPerspectCamera::MyPerspectCamera(VEC3 position, VEC3 front, VEC3 worldUP):
Yaw(YAW),Pitch(PITCH),MovementSpeed(SPEED),MouseSensitivity(SENSITIVITY), Front(glm::vec3(0.0f, 0.0f, -1.0f)),Zoom(ZOOM)
{
	Position = position;
	Front = front;
    WorldUp = worldUP;
    Projection = glm::perspective(glm::radians(Zoom), (float)SCREEN_WIDTH / (float)SCREEN_HEIGH, 0.1f,100.f);
	update();
}

// view矩阵是用lookat方法获取，用到的就是我们之前定义的属性
glm::mat4 MyPerspectCamera::vieMatrix() {
    return glm::lookAt(Position,Position + Front, Up);
};
// projection 矩阵则是直接返回我们对象持有的当前projection
glm::mat4 MyPerspectCamera::projection() {
    return Projection;
};

void MyPerspectCamera::move(CAMERA_DIRECTION dir, float deltaTime) {
    if (dir == CAMERA_DIRECTION::CAMERA_UP) {
        Position += Front * deltaTime;
    }
    else if (dir == CAMERA_DIRECTION::CAMERA_DOWN) {
        Position -= Front * deltaTime;
    }

    if (dir == CAMERA_DIRECTION::CAMERA_LEFT) {
        Position -= Right * deltaTime;
    }
    else if (dir == CAMERA_DIRECTION::CAMERA_RIGHT) {
        Position += Right * deltaTime;
    }
}

void MyPerspectCamera::rotate(float xoffset, float yoffset, GLboolean constrainPitch) {
    xoffset *= MouseSensitivity;
    yoffset *= MouseSensitivity;

    Yaw += xoffset;
    Pitch += yoffset;

    // make sure that when pitch is out of bounds, screen doesn't get flipped
    if (constrainPitch)
    {
        if (Pitch > 89.0f)
            Pitch = 89.0f;
        if (Pitch < -89.0f)
            Pitch = -89.0f;
    }

    // update Front, Right and Up Vectors using the updated Euler angles
    update();

}

void MyPerspectCamera::justZoom(float yoffset)
{
    Zoom -= (float)yoffset;
    if (Zoom < 1.0f)
        Zoom = 1.0f;
    if (Zoom > 45.0f)
        Zoom = 45.0f;
    Projection = glm::perspective(glm::radians(Zoom), (float)SCREEN_WIDTH / (float)SCREEN_HEIGH, 0.1f, 100.f);
}

void MyPerspectCamera::update()
{
    // calculate the new Front vector
    glm::vec3 front;
    front.x = cos(glm::radians(Yaw)) * cos(glm::radians(Pitch));
    front.y = sin(glm::radians(Pitch));
    front.z = sin(glm::radians(Yaw)) * cos(glm::radians(Pitch));
    Front = glm::normalize(front);
    // also re-calculate the Right and Up vector
    Right = glm::normalize(glm::cross(Front, WorldUp));  // normalize the vectors, because their length gets closer to 0 the more you look up or down which results in slower movement.
    Up = glm::normalize(glm::cross(Right, Front));
}
```

