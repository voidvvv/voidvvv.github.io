---
title: LIBGDX 游戏引擎
date: 2023-12-18 22:22:22
top: 0
categories: 
- java
- libgdx
tags: 
- [java]
- [game]
- [libgdx]
---

# LIBGDX 游戏引擎
{% cq %} 一款使用OpenGL 的 Java 游戏引擎 {% endcq %}
![logo](libgdx_logo.svg)
<!-- more -->



## 介绍
><b>libGDX</b> is a cross-platform Java game development framework based on OpenGL (ES), designed for Windows, Linux, macOS, Android, web browsers, and iOS. It provides a robust and well-established environment for rapid prototyping and iterative development. Unlike other frameworks, libGDX does not impose a specific design or coding style, allowing you the freedom to create games according to your preferences.<br><br>
github： https://github.com/libgdx/libgdx
<br>

---
## Hello World
### 准备工作
1. JDK： 建议 11-18. 我个人使用JDK11，感觉非常良好
2. 开发工具，正常的java开发集成工具都可以，我个人使用JetBrain的Idea
3. 依赖管理工具，libgdx使用的是gradle进行包管理，所以最好提前安装gradle。我本地安装的是7.4.2. 如果不提前手动下载，后面导入可能会麻烦一些。
4. 初始化项目：
    * 下载官方的初始化工具，目前地址： https://libgdx.com/wiki/start/project-generation （最新版）
    * 或者可以下载 1.10 版，因为我就是用这个版本开发的。https://github.com/libgdx/libgdx/releases 可以在这里找到历史版本，选择1.10.0即可。

### Init!
好了，现在我们可以尝试开始我们的第一个项目了。<br> 
<br>
首先，打开我们刚刚下载好的初始化工具： <br>
![init01](Libgdx_Init01.png)
<br>
关于第6点打包模式，个人建议勾选desktop，方便桌面调试，或者直接开发桌面应用。andriod也推荐勾选，因为这个引擎起始主要是为了andriod游戏而做的，大部分资源文件都会放在andriod项目下。不管你开发的是什么应用。<br>
至于更多的第三方依赖，暂时用不到，可以不需要管。然后，我们直接点击generate生成即可.<br>
点击生成后，会自动在目标文件夹下生成初始化的代码，并且会尝试使用gradle导入依赖，但是此时极有可能因为本地JAVA_HOME等原因导入失败，这个失败我们可以先不去管，忽略它，直接使用我们的开发工具（我使用的IDEA）打开该项目即可。<br>
在使用我们的开发工具打开项目后，首先建议修改默认的gradle配置，更改为自己下载好的版本。
![init02](Libgdx_Init02.png)<br>
这样，导入依赖会更快更方便，不然的话，我本地实测会比较慢。<br>
然后接下来就让gradle自动构建就可以了。

### Hello World
构建好之后，项目看起来应该是这个样子的:
![init03](Libgdx_Init03.png)<br>
其中，desktop目录中，存在一个名为DesktopLauncher的启动类，我们可以直接运行这个启动类。应该会看到一个应用程序启动，如下图：<br>
![init04](Libgdx_Init04.png)<br>
如果看到这个画面，恭喜你，我们已经成功的初始化了一个游戏！

---

现在我们看到的仅仅是一个libgdx logo的图片以及红色背景。这显然不是我们最终想要的，我们至少想要一个自己的**hello world**<br>
我们就需要认识一下core文件夹，下面有一个初始化自带的一个Class，MyGdxGame，它长这个样子：
``` java
package com.mygdx.game;

import com.badlogic.gdx.ApplicationAdapter;
import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.utils.ScreenUtils;

public class MyGdxGame extends ApplicationAdapter {
	SpriteBatch batch; // 画笔，或者说专门画图片(Sprite)的画笔
	Texture img; // 图片，我们看到的libgdx logo 就是它
	
	@Override
	public void create () { // 当前类初始化的方法
		batch = new SpriteBatch(); // 初始化画笔
		img = new Texture("badlogic.jpg"); // 初始化 加载 图片
	}

	@Override
	public void render () { // 渲染函数，可以理解为程序的每一帧需要做什么，这里也是程序的主逻辑
		ScreenUtils.clear(1, 0, 0, 1); // 设置屏幕背景色
		batch.begin();
		batch.draw(img, 0, 0); //  使用画笔绘制图片，图片的位置指定在 （0，0） 处
		batch.end();
	}
	
	@Override
	public void dispose () { // 销毁方法，程序运行中的许多资源（Disposable）都需要释放
		batch.dispose();
		img.dispose();
	}
}

```
正是上面这段代码的render函数绘制出了我们的图片，这个其实也是整个程序的主逻辑。
如果我们想要更改渲染的东西，那么我们要做的工作就是想办法在这里渲染我们想要的东西

## 主要组件
### **Game Screen**
com.badlogic.gdx.ApplicationListener 可以算作整个游戏的入口，我们需要手动实现它。它里面的render方法就是我们的主逻辑。但是但看这个接口，是有点抽象的，因为我们还不太了解具体该怎么做。幸好，libgdx给了我们一个比较好的实现类：**com.badlogic.gdx.Game**
我们现在已经看到Game类是实现了ApplicationListener接口的，那么我们可以试着把系统自动给我们生成的MyGdxGame类改成继承Game类来试一下.更改后：
```java
public class MyGdxGame extends Game { // 只有父类变成了Game
	SpriteBatch batch;
	Texture img;
	
	@Override
	public void create () {
		batch = new SpriteBatch();
		img = new Texture("badlogic.jpg");
	}

	@Override
	public void render () {
		ScreenUtils.clear(1, 0, 0, 1);
		batch.begin();
		batch.draw(img, 0, 0);
		batch.end();
	}

	@Override
	public void dispose () {
		batch.dispose();
		img.dispose();
	}
}
```
然后再次运行，发现没有变化，那么就证明我们更换正确了。我们现在开始就可以使用这个Game当作游戏的起点了！
还有另外一个组件 **Screen**
**Screen** 是一个接口，代码如下： 
{% codeblock  Screen.java lang:java %}
public interface Screen {
	
	/** Called when this screen becomes the current screen for a {@link Game}. */
	public void show ();
	
	/** Called when the screen should render itself.
	 * @param delta The time in seconds since the last render. */
	public void render (float delta);

	/** @see ApplicationListener#resize(int, int) */
	public void resize (int width, int height);

	/** @see ApplicationListener#pause() */
	public void pause ();

	/** @see ApplicationListener#resume() */
	public void resume ();

	/** Called when this screen is no longer the current screen for a {@link Game}. */
	public void hide ();

	/** Called when this screen should release all resources. */
	public void dispose ();
}
{% endcodeblock   %}

这里的render方法可以用了来**更好的**实现我们的逻辑，因为参数附加了当前帧与上一帧相差的秒数（float），可以更好的更新我们的逻辑。

Game 中负责Screen的调度，我们可以通过 {% label primary@com.badlogic.gdx.Game#setScreen %}  方法来设置当前需要渲染以及更新的screen

### **Batch**
Batch 是我们整个程序的画笔，这个画笔有很多类型，并且由于每种batch都没有一个统一的抽象规范，目前是没有所谓的batch基类的，我们只能找到一些各种实现类。比如我们上面见到的  {% label primary@com.badlogic.gdx.graphics.g2d.SpriteBatch %} ，就是专门用于绘制纹理图片的，还有可以绘制各种形状的 {% label primary@com.badlogic.gdx.graphics.glutils.ShapeRenderer%},这个叫做render的类也是起到batch的作用。甚至还有可以绘制3D模型的 {% label primary@com.badlogic.gdx.graphics.g3d.ModelBatch %}
batch底层使用的起始都是opengl。这是一个计算机图形统一接口。

### **Camera**
Camera顾名思义，就是我们整个游戏中的{% label @相机%}, 它的作用其实是处理我们游戏中的各种坐标变换，游戏视角的切换等等
Camera的底层其实是一个 {% label @四维矩阵%},这一部分也是OpenGL的知识.


### **Input**
Input是游戏中不可缺少的，负责处理用户输入，有了用户输入才能有交互，有了交互才能称之为游戏。
在LibGDX中，Input 相关的交互全在  {% label @Gdx.input %} 这个全局静态引用中。 Input 最基础的用法是：
{% codeblock  Input lang:java %}
		// 获取当前点击屏幕的坐标
		if (Gdx.input.isTouched()) {
			int x = Gdx.input.getX();
			int y = Gdx.input.getY();
		}
{% endcodeblock   %}
除此之外，还有很多使用方法，直接调用这个引用即可。
需要注意：
1. Gdx.input 这个全局静态引用的实现是基于你当前的系统来自动决定的。
2. 上面这个获取坐标的方法获取到的其实是我们的屏幕坐标，也就是当前指针指向的像素点的坐标。这个坐标还需要使用我们的 {% label @Camera %} 换算才能被使用。Camera坐标换算可能需要在其他文章中仔细讲解.
除了上面的这种最基本的控制，LibGDX还给我们提供了一种更加方便的封装，那就是 {% label success@com.badlogic.gdx.Input#setInputProcessor %}. 这个方法的参数 InputProcessor 是libgdx给我们封装好的一个输入处理接口，里面有几乎我们能用到的所有输入处理。我们只需要实现对应的输入处理方法即可。
同时，libgdx还内置了一个InputMultiplexer来同时处理多个Input，源码比较简单，就是内置了一个集合遍历，故不在此详述。


### **Asset**
Asset,游戏资源，关于我们如何加载游戏资源，我们可以使用 {% label success@Gdx.files %} 这个全局静态引用来获取我们所需要的文件。
获取到的文件是一个 {% label @com.badlogic.gdx.files.FileHandle %}, 我们可以使用这个handle来生成我们想要的各种游戏内素材，比如：

{% codeblock  Asset lang:java %}
        Texture pic = new Texture(Gdx.files.internal("图片位置")); // 获取图片
        Music music = Gdx.audio.newMusic(Gdx.files.internal("音乐文件位置")) ;// 获取长音乐;
        Sound sound = Gdx.audio.newSound(Gdx.files.internal("音乐文件位置")) ;// 获取短音效;
{% endcodeblock   %}
这里文件位置使用的是internal。internal代表是从我们项目内部来获取资源的。
![asset01](Libgdx_asset01.png)
这也是我们初始代码获取默认图片的方式

关于几种file的获取方式，可以参考chatgpt的回答：
```
Classpath (类路径)：

路径类型： classpath:/path/to/file
说明： 这是相对于类路径的文件位置。这意味着文件应该位于你的类路径（例如，src 目录）下。
示例： 如果你有一个位于 "assets/images/myimage.png" 的文件，你可以通过 Gdx.files.classpath("assets/images/myimage.png") 获取它。
Internal (内部文件)：

路径类型： internal:/path/to/file
说明： 这是相对于应用程序的根目录的文件位置。这通常用于存储在应用程序打包时一起分发的资源。
示例： 如果你的应用程序包含 "data/config.txt" 文件，你可以通过 Gdx.files.internal("data/config.txt") 获取它。
External (外部文件)：

路径类型： external:/path/to/file
说明： 这是外部存储设备上的文件位置，例如 SD 卡。这允许你访问设备上的外部文件系统。
示例： 如果你希望访问 SD 卡上的 "MyApp/data/config.txt" 文件，你可以通过 Gdx.files.external("MyApp/data/config.txt") 获取它。
Absolute (绝对路径)：

路径类型： absolute:/path/to/file
说明： 这是一个绝对路径，允许你指定文件系统中的完整路径。
示例： 如果你有一个绝对路径为 "/home/user/documents/file.txt" 的文件，你可以通过 Gdx.files.absolute("/home/user/documents/file.txt") 获取它。
```
总之，我目前最常用的就是internal

## LibGDX 内置 Enviroment
libgdx 内置了一些可以用来在程序运行中获取资源或者修改程序参数的功能类。这些enviroment被以单例模式放在了 com.badlogic.gdx.Gdx 类中。
其中每一项都会根据当前运行的系统来分别init。
{% codeblock  Gdx lang:java %}
public class Gdx {
	public static Application app;
	public static Graphics graphics;
	public static Audio audio;
	public static Input input;
	public static Files files;
	public static Net net;

	public static GL20 gl;
	public static GL20 gl20;
	public static GL30 gl30;
}
{% endcodeblock %}

比如，我们可以使用其中的 audio 来管理音频，可以使用files来管理文件资源，可以使用input来处理输入。

## 参考
[官方文档](https://libgdx.com/wiki/start/project-generation)
[官方simple教程](https://libgdx.com/wiki/start/a-simple-game)