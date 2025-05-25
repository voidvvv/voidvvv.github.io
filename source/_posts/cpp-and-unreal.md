---
title: c++与unreal学习
date: 2023-12-18 19:42:40
categories:
- c++
tags:
- c++
---
# C++基础：

> 仅仅作为本人java开发，自学c++的学习笔记。因为有java基础，所以基础内容记录不会很详细

<!-- more -->
## 基础

编译器(也是开发工具)：

visual studio 2017. 下载地址：https://visualstudio.microsoft.com/zh-hans/vs/community/

### visual studio 基本操作：

ctrl+k，然后 ctrl+c ，行注释一段代码。 格式：// 与java相同

ctrl+k，然后ctrl+u，取消一段代码的注释。

菜鸟基础学习地址：https://www.runoob.com/cplusplus/cpp-tutorial.html

### helloworld步骤：

1. 使用visual studio新建一个空白项目，取名helloworld

2. 在源文件处，新建项目。选择cpp文件。取名helloWorld

3. 编写如下代码：

   ```c++
   // <>表示程序将会首先且只会去你的系统类库目录查找你所想引入的类或者包;
   // ""表示程序会首先从你的当前目录查找你所想引入的类或者包,如果没有找到,将去系统类库目录找.
   
   #include "iostream"
   using namespace std;
   
   
   int main() {
   	cout << "hello world" << endl;
   
   	system("pause");
   	return 0;
   
   }
   ```

4. 点击本地windows调试，可以看到输出 hello world

### 常量

定义常量两种方式：

1. #define 宏常量。 #define  name  value，例如：#define dayOfMonth  7
2. 在变量前加 **const**关键字。

常量是不可以更改的。一旦修改就会报错。类似于java的final

### 基本数据类型

1. 整型：**int**。8个字节。与java相同。并且具有构造方法

2. 实型（浮点型）： 对应java中的**float，double**。并与java含义相同

3. 字符型：**char**,与java用法相同。字符的ascii码

4. 转义字符：参见正则

5. 字符串类型：因为沿用于c语言，可以直接将char数组定义为字符串。例如：char str[] = "hello world"

   同时也有自己的对象风格，string str = "hello world"。需要注意，若要在代码中定义sting类型数据，需要引入头文件 #include "string"

6. 布尔类型：**bool**.基本与java相同。不同的在于，在输出bool时，其实质就是一个字节。表示为1（真），或0（假）。

### 标准输入输出流

```c++
// 在引入头文件后：
#include "iostream"
using namespace std;

// 可以在代码中使用如下cout左移来输出数据
cout << "aaa" <<endl;
// 可以使用cin右移运算符来输入数据
int a;
cin >> a;
// 若定义的数据为int，输入的是字母，则会赋值为0
```

### 变量

用法与java大致相同。不过c++具有指针，引用的概念，可以更自由的访问内存。

> 需要注意的是，当c++中， type a = b ; 的时候，会创建一个变量b的副本给a的引用。此时，a，b是两个独立的变量。各自分别改变值，对另一个没有影响。

### 数组

定义与java相同。每个数据类型都相同。并且需要一块连续的内存空间

声明方式：

1.  int arr[2] = {1,2}; 即在声明变量后加[]中括号，并且规定大小，等于号右边放入数组初始值。初始值个数不能超过规定大小，否则会报错
2. int arr[2]; 声明大小，不给初始值，会默认用0填充。对象的话，会用空参数对象填充
3.  int arr[] = {1,2},自动推导数组长度。

需要注意的是：**c++中，数组的名称的本质，其实是指向第一个元素的一个指针。所以可以用指针解引用的方式取出第一个元素。又由于数组内存空间是连续的，所以可以使用名称++，来获取下一个元素的指针.获取数组长度的方法，可以用 sizeof函数，sizeof(数组)/sizeof(数组[0]).**

### 指针与引用：

> c++中的一个难点。c++指针在32位系统下，占4个字节。在64位系统下，占8个字节，即为一个int（不管指针内存的数据类型）

指针就是指指向一块内存区域的地址，下面的代码展示指针的相关用法:

```c++
#include "iostream"
using namespace std;
#include "string"

#define dayOfMonth  7

int main() {
	int a = 20;
	// 这里的&a为把a所代表的内存空间拿出来，来让p指针来指向该空间。
	int *p = &a;
	cout << p << endl;
    
    // 下面这一步是对指针解引用。就是将指针所指向内存空间的值（或对象）拿出来。这里就拿val来接受
    int val = *p;

	system("pause");
	return 0;

}
```

在变量前加 * 号，可以将其定义为指针。指针 = 引用，代表指针指向某一块引用。上面int * p ,就是定义了一个类型为int的指针。指针指向变量a所在的内存空间。这里&a就是a变量所在内存空间的引用。因为指针需要指向引用，所以等号右边就需要是一个引用；

关于引用，就是一块内存，指针就是内存地址，而上面代码中的a，就是该内存当前的名字。详细可以看下面的代码：

```c++

int main() {
	// 声明一个int型的变量a，初始值为20；
	int a = 20;
	// 声明一个int型的变量b，初始值为30；
	int b = a;
	// 声明一个int类型的变量c，其引用的是a的地址；
	int &c = a;

	a++;
	cout << "a++后：" << "a: " << a << "b: " << b << "c: " << c << endl;
	b++;
	cout << "b++后：" << "a: " << a << "b: " << b << "c: " << c << endl;
	c++;
	cout << "c++后：" << "a: " << a << "b: " << b << "c: " << c << endl;

	system("pause");
	return 0;

}
```

可以看到，c用的是a的引用，即c所用的内存空间，就是a所在的内存空间。所以当a改变时，c也会跟着改变。c改变时，a也会跟着改变。但是b就跟a和c没关系。只是用a的值做了个克隆构造函数，在另一块内存创建了一个新对象。

#### 空指针：

内存条中，由操作系统所占用，我们的程序无法访问的内存区域。一般位0-255号内存。即0-255内存若要访问，就会导致空指针异常（npe）

#### 野指针：

指向非法内存空间的指针。具体是值我们程序没有申请的内存空间，若要解引用，则会抛异常

#### 指针取值

在有当前指针的情况下，一般是需要解引用 *p解引用来获取指针指向内存的数据。但是如果目标区域是对象，则可以用 ->来获取对象参数或者执行方法.



### 流程控制：for循环，while循环，if判断，三目运算法

与java一样

### c++流程控制：goto关键字

可以在代码中直接跳到某个位置。实际开发中，由于代码可读性差，可以被break和continue代替等原因，不会使用。这也是java中goto作为未实现的保留关键字的原因

### 函数（java中的方法）

我们上面写的main函数，就是函数的一种。具有返回值，函数名，入参。并且c++中main方法需要无惨，并且返回int

#### 函数值传递：

```c++
#include "iostream"
using namespace std;
#include "string"

void say(int a) {
	a += 50;
	cout << "say方法中的a："<<a << endl;
}

int main(){
	int a = 10;
	say(a);
	cout << "say方法外的a：" << a << endl;

	system("pause");
	return 0;
}
// 上面代码可以看到方法内和方法外打印的a的值是不一样的，这个也是java对于基本类型的处理，传入一个值得副本，对值得改变，无法对原值产生影响。
// 值传递对于对象，也是拷贝一个副本作为方法参数，所以即使在方法内修改对象的属性，外面的对象属性也不会发生变化。
```

#### 函数的引用传递：

```c++
void say2(int &a) {
	a += 50;
	cout << "say方法中的a：" << a << endl;
}

int main(){
	int a = 10;
	say2(a);
	cout << "say方法外的a：" << a << endl;

	system("pause");
	return 0;
}
// 则会发现，方法内外的a的值都发生了变化。这是因为传入方法内的是a的引用，方法参数a与外面的a代表同一块内存空间。修改的也是同一块内存空间。java对象采用的即此传递
```

#### 函数指针传递

```c++
void say3(int *p) {
	*p += 50;
	int a = *p;
	cout << "say方法中的a：" << a << endl;
	cout << "say方法中的p：" << p << endl;

}


int main(){
	int a = 10;
	int *p = &a;

	say3(p);

	cout << "say方法外的a：" << a << endl;
	cout << "say方法外的p：" << p << endl;

	system("pause");
	return 0;
}
// 上面代码可以看到，若直接解引用然后赋值，会影响外面的数据。若int a = *p； 然后a++，则不会影响外部数据
```

#### 函数重载：

与java一样

#### 默认参数:

c++中，函数的入参可以有默认值。若调用者不传对应参数，则方法会自动使用默认值。

需要注意，在声明函数的时候若使用了默认值，则在实现的时候不能有默认值。并且默认参数的函数本质是函数重载，所以有任何相同调用方法的别的函数存在，编译就会报错

#### 占位参数

区别于java的一个点，函数的入参可以只写一个类型，而不写名称。称之为站位参数。占位参数在调用时，需要传入对应类型的站位参数来调用函数。应该是用于函数重载的

```c++
void say(int a) {
	cout << "aaa" << endl;
}

void say(int a,int) {
	cout << a << endl;
}

int main(){
	say(10,10);

	system("pause");
	return 0;
}
```





#### 函数的声明

与函数的定义相对，在函数声明前，是无法使用的。编译器会找不到函数。若先进行声明，（无函数体，类似于java的抽象方法），下面就可以用这个方法了。然后，编译器在执行的时候，会找该方法的实现。方法声明可以多次，但是实现只能是一次。若有多次实现，会报错。

### c++关键字：const

被const修饰的变量为常量，我们上面已经知道,可以作为变量。但是对于const修饰指针，以及对象，则另有说法。

#### const修饰指针

````c++
int main(){
	// 基准
	int a = 20;
	// 与a引用不同，值不同
	int b = 30;
	// 与a引用不同，值相同
	int c = 20;
	// 与a引用相同
	int &d = a;

	// 常量指针，指针可以改指向，但是指针指向的内存空间不可以改
	const int* p1 = &a;
	// 指针常量  指针指向不可以改，但是指针内的内存空间可以改
	int* const p2 = &a;
	// 全都不能改
	const int*  const p3 = &a;


	cout << *p1 << endl;
	//*p1 = b;// 修改p1的值，报错
	p1 = &c;// 修改p1的指向为c，不报错
	p1 = &b;// 修改p1的指向为b，不报错
	cout << *p1 << endl;

	//p2 = &b;// 试图修改p2的指向，报错
	*p2 = b;// 修改p2的值，不报错

	*p3 = b;// 修改p3的值，报错
	p3 = &c; // 修改p3的指向，报错 

	system("pause");
	return 0;
}
````

#### const修饰对象：

该对象及其内部属性全部不能改

#### 常量引用：

一般情况，int &a =10,即将一个引用等于一个常量是非法的。但是可以在前面加const，来使其合法。该引用变为常量引用，无法修改内存中的值

## 核心

### 面向对象

c++中，定义类有两种方式，1. 定义结构体struct，2. 定义class。两种方式几乎没区别。在网上看到的区别为：struct默认权限类型为public，class默认权限类型为private。

#### c++类

```c++
class MyStruct
{
private:
	string name;
	int age;
protected:
    string lastName;
public:
	MyStruct(int age,string name) {
		this->age = age;
		this->name = name;
	}
};
// 权限修饰符的写法与java有差异。定义一个权限区，然后将该权限的属性或函数写进去即可。权限定义也跟java一致。private私有，public公共，protected保护，子类可见
	MyStruct(int age,string name) {
		this->age = age;
		this->name = name;
	}

	MyStruct( MyStruct& origin) {
		// 拷贝构造
	}

	~MyStruct() {
		// 析构函数
	}
```

> 注意：c++类定义完后，需要加分号 ;

#### 构造函数，析构函数：

一个类，默认会有无参构造，这个跟java一致。并且自定义有参构造，会默认覆盖无惨构造。还有一个拷贝构造入参为自己本类的类型，使用浅克隆将属性赋值到本对象.

还有一个析构函数，在对象销毁时会调用，一般用来释放内存;

#### 初始化列表：

```c++
// 是跟在构造方法后，初始化类属性的方法
// 使用上面的MyStruct类
	MyStruct():name("zhangsan"),age(20) {
	}
// 上面是个空参构造，但是在调用时会初始化name和age。值就是括号中的值
// 或者有参构造，
	MyStruct(int age,string name):name(name),age(age) {
		
	}

```

#### 静态成员变量与函数

属性前加static。规则与java差不多。

调用方式:  类名::成员名称

```c++
class People{
    static int i;
    
    static void test(){
        cout<< i <<endl;
        
    }
    
}

int main(){
    People::test();
    return 0;
}
```



#### this指针

与java的this类似。不过这里的this不是对象，而是指针。需要解引用或者使用 ->

#### 空指针相关：

c++中，空指针是可以调用成员函数的。但是前提是需要函数中没有用到this。若用到了this，则会报异常。否则成功运行

#### const

const修饰函数，函数变为常函数。常函数只能调用常量（const修饰的）或者mutable修饰的变量。是为了保护对象内部的属性不被乱改

const修饰对象名称，则该对象只可以调用常函数

#### 友元

关键字：friend. 可以声明一个类或者函数作为友元。该类或者该函数可以访问本类私有属性及函数

```c++
#include "iostream"
using namespace std;
#include <string>

class A {
    // 类做友元
	friend class B;
    // 成员函数做友元
    friend void B::visit(A a);
    // 全局函数也可以做友元
private:
	int a_i;
	string a_m;
	void privateTest() {
		cout << "A private" << endl;
	}
};

class B {
private:
	int b_i;
	string b_m;
public:
	void visit(A a) {
		a.privateTest();
	}
};
int main() {
	B b;
	A a;
	b.visit(a);

	system("pause");
	return 0;
}
```

### 继承

类似于java继承，但是c++允许多继承。

```c++
// 继承语法：
class A{
    // 定义类A
}

class B:public A{
    // 定义类B，继承类A。
}

```

继承也有限定。public：将父类属性及方法按照原有访问权限继承。protected：将父类public的属性以及函数改为protected继承到自身。private：将父类属性及函数继承过来，但是权限全变为private；

在单继承后，子类在不重写父类方法的情况下，调用函数是父类的函数。若重写了函数，则调用的是自己的。

若想调用父类的函数，需要 对象名.父类类名::方法名（参数）;例如：

```c++
class B {
private:
	int b_i;
	string b_m;

public:

	void visit(A a) {
		a.privateTest();
	}


};

class D {
private:
	int b_i;
	string b_m;

public:

	void visit(A a) {
		cout << "d visit" << endl;
	}


};

class C:public B,public D  {


public:


};
int main() {
	B b;
	A a;
	C c;
	b.visit(a);
// 调用父类方法
	c.B::visit(a);

	system("pause");
	return 0;
}
```

#### 菱形继承-virtual关键字

多继承，难免会有A,B两个父类有相同的成员或者方法。并且该情况会引出父类方法实现无用的结果。

则在继承时加入virtual关键字虚继承。则在编译时不会生成父类，在执行时才会。这样，相同的数据只会有一份，按照子类为准。

virtual也可以修饰方法，称为虚方法，在父类引用指向子类实例时，c++中，调用父类引用的方法，会执行父类实现。若方法为虚方法，则类中会存储虚基类指针，调用父类引用子类实现的对象的方法，会调用子类函数。

```c++
virtual void say() = 0;
// 称为纯虚函数。类似于java的抽象接口。有纯虚函数的类无法实例化对象。子类若若不实现纯虚函数，则子类也无法实例化对象
```

#### 函数模板 类模板

java中的泛型。其实java的泛型就是从c++的模板变过来的。

```c++
// 语法
template <class T> // 下面跟着类，就是类模板。跟着函数，则是函数模板
class Person<T>{
    
}    
```

基本语法如上。大致上跟java里差不多。

方法模板中，可以根据入参来自动判断模板类型。而类没有自动判断。

类模板可以 class T = int 来指定默认值;若类模板函数在类外实现，需要在实现前加上模板声明

### 运算符重写

运算符：==，>>,<<,() 都可以重写。返回值类型   operator==（入参）{方法体}

```c++
// 重写== （可以类比于java重写equals），在类内
bool operator== (obj o){
    
}
// 若重写左移，可以在全局重写. 第一个参数是调用方，第二个参数是右侧（可视为java的tosting）

ostream operator<<(ostream& cout,obj o){
    cout<< 拼接obj的属性 <<endl;
    return cout;
}
```



### 分文件编写

在前面讲到了，c++可以是先写好方法定义，然后去实现的。平时开发习惯将类声明以及类方法实现放在两个不同文件，声明为.h后缀的头文件，一般放属性，以及函数定义。实现是.cpp后缀的实现，引入对应头文件来编写方法实现。

**需要注意的是，若类模板份文件编写，则要注意类模板函数是在执行的时候生成的。导致引入头文件不会导入函数，编译器会不认得这个函数。cpp实现文件自然也加载不到。**



## STL

类比于java的容器。有list，map，set等。主要是使用模板技术。

stl主要分为：容器，迭代器，算法,仿函数，适配器，空间配置器。

#### 迭代器

大部分容器使用.begin()方法会获取迭代器的开始指针，解引用后可以获得该处对象

### 容器

#### vector容器--单边数组

类比于java的list；

```c++
// 基本使用
void vectortest() {
	vector<int> v;
	for (int x = 0; x < 10; x++) {
		v.push_back(x);

	}
	vector<int>::iterator it=v.begin();
	vector<int>::iterator ite = v.end();
	// 遍历vector
	while
        (it != ite) {
		cout << *it << endl;
		it++;
	}

}
```

##### 常用函数 api：

```c++
// 重写运算符 = 用来初始化一个集合
// assign(begin,end)  用来将指定迭代器中间选中部分的数据复制到本集合中
// assign(n,element)  将n个指定元素复制到本集合中
empty();//容器是否为空
capacity();// 容器的容量
size();// 容器的大小（当前元素个数）
resize(int);// 重新指定容器大小。若当前元素个数超过大小，则删除末尾元素。若不足，则以默认值填充
resize(int，ele);// 重新指定容器大小。若当前元素个数超过大小，则删除末尾元素。若不足，则以ele填充
// 插入与删除
push_back(ele);// 将元素放在集合尾部
pop_back();// 删除最后一个元素
clear(); // 清除所有元素
erase(iterator);// 删除指定迭代器指向的元素
insert(iterator,ele);// 在指定位置插入元素
// 存取
at(index);// 获取指定索引处元素
operator[];// 获取指定id处元素 -- 可以存
front();// 返回最前面的元素
back();// 返回最后一个元素
swap(vec);// 互换元素
reserrve(int);// 预留指定长度空间.预留位置内存不初始化.无法访问
```

#### deque -双端数组.可头插可尾插

常用api:

```c++
push_front(ele);// 数组前段插入元素
push_back(ele);// 后方插入元素
pop_front();// 删除头元素
pop_back();//删除尾部
// deque无容量限制.类似于双向链表.其他api类比于vector
```

