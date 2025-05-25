---
title: java包获取路径
date: 2023-12-18 19:24:53
categories:
- java
tags:
- java
---

相关问题总结;
在获取根目录时：
<!-- more -->>
```java

    public static void main(String[] args) throws IOException {

        URL resource = MyTest.class.getResource("/");
        URL resource2 = MyTest.class.getResource("");
        URL resource3 = MyTest.class.getClassLoader().getResource("");
        URL resource4 = MyTest.class.getClassLoader().getResource("/");

        URL resource5 = MyTest.class.getClassLoader().getClass().getResource("");
        URL resource6 = MyTest.class.getClassLoader().getClass().getResource("/");

        String path = MyTest.class.getProtectionDomain().getCodeSource().getLocation().getPath();

        System.out.println("codesource:"+path); // jar包和idea环境均获取相同值，为当前项目路径
        File f = new File(path);
        System.out.println("exist？"+f.exists());

        String path2 = System.getProperty("java.class.path");
        System.out.println("path2::"+path2); // 获取环境变量,jar 中就是获取当前jar包名称。

        System.out.println("class / :"+resource); // idea中获取当前项目地址,jar中为null
        System.out.println("class 空 :"+resource2); // idea 中获取当前类所在位置。 jar中也是，但是不是file，是jar格式
        System.out.println("classloader / :"+resource4); // idea null  jar  null
        System.out.println("classloader 空 :"+resource3); // idea 当前项目地址  jar null

        System.out.println("loader+class/:"+resource6); // idea 当前项目地址 jar 空
        System.out.println("loader+class 空:"+resource5); // idea null  jar null

        // 找到jar包内，properties指定位置路径下所有指定类型的文件
        if (path.endsWith(".jar")){ // 若是jar包，则需要这样做来获取文件
            System.out.println("jar execute");
            JarFile jarFile = new JarFile(path);
            Enumeration<JarEntry> entries = jarFile.entries();
            while (entries.hasMoreElements()){
                JarEntry jarEntry = entries.nextElement();
                String name = jarEntry.getName(); // 全类名(/cn/text/A.class),以及包名
//                System.out.println(name);
                if (name.startsWith("fxml/")&&name.endsWith(".txt")){
                    System.out.println("zhjao");
                    BufferedReader b = new BufferedReader(new BufferedReader(new InputStreamReader(MyTest.class.getResourceAsStream("/"+name))));
                    String s = b.readLine();
                    System.out.println(s);
                }
            }
        }

        InputStream resourceAsStream = MyTest.class.getResourceAsStream("/config.properties"); // idea与jar均相同，获取jar项目内文件输入流
        Properties p = new Properties();
        p.load(resourceAsStream);
        System.out.println(p);


    }
```

在获取文件情况下：

```java
    public void testtt() throws IOException {

        URL resource = MyTest.class.getResource("/config.properties");
        URL resource2 = MyTest.class.getResource("config.properties");
        URL resource3 = MyTest.class.getClassLoader().getResource("config.properties");
        URL resource4 = MyTest.class.getClassLoader().getResource("/config.properties");

        URL resource5 = MyTest.class.getClassLoader().getClass().getResource("config.properties");
        URL resource6 = MyTest.class.getClassLoader().getClass().getResource("/config.properties");

        String path = MyTest.class.getProtectionDomain().getCodeSource().getLocation().getPath();

        System.out.println("codesource:"+path); // jar包和idea环境均获取相同值，为当前项目路径
        File f = new File(path);
        System.out.println("exist？"+f.exists());

        String path2 = System.getProperty("java.class.path");
        System.out.println("path2::"+path2); // 获取环境变量,jar 中就是获取当前jar包名称。

        System.out.println("class / :"+resource); // idea jar中均可以取到,jar 中是jar模式
        System.out.println("class 空 :"+resource2); // idea jar 均为空
        System.out.println("classloader / :"+resource4); // idea null  jar  null
        System.out.println("classloader 空 :"+resource3); // idea 可以取到资源  jar 可以取到 jar模式

        System.out.println("loader+class/:"+resource6); // idea 可以取到资源  jar 可以取到 jar模式
        System.out.println("loader+class 空:"+resource5); // idea null  jar null

        // 找到jar包内，properties指定位置路径下所有指定类型的文件
        if (path.endsWith(".jar")){ // 若是jar包，则需要这样做来获取文件
            System.out.println("jar execute");
            JarFile jarFile = new JarFile(path);
            Enumeration<JarEntry> entries = jarFile.entries();
            while (entries.hasMoreElements()){
                JarEntry jarEntry = entries.nextElement();
                String name = jarEntry.getName(); // 全类名(/cn/text/A.class),以及包名
//                System.out.println(name);
                if (name.startsWith("fxml/")&&name.endsWith(".txt")){
                    System.out.println("zhjao");
                    BufferedReader b = new BufferedReader(new BufferedReader(new InputStreamReader(MyTest.class.getClassLoader().getResourceAsStream(name))));
                    String s = b.readLine();
                    System.out.println(s);
                }
            }
        }

        InputStream resourceAsStream = MyTest.class.getResourceAsStream("/config.properties"); // idea与jar均相同，获取jar项目内文件输入流
        Properties p = new Properties();
        p.load(resourceAsStream);
        System.out.println(p);

    }
```
