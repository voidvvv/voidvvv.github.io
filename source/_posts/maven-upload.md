---
title: 向Maven仓库发布自己的项目所遇到的一些坑
date: 2025-05-17 20:50:45
---

> 在尝试将自己的项目推送到maven仓库时，遇到并解决了很多坑点，特此来记一下笔记

需要提前声明一下，推送maven仓库，官方有很多中方法，比如使用build插件（maven plugin， gradle plugin），API等等，我这次是直接手手动在页面上上传的。

<!-- more -->

## 问题：
maven publish 官网：[maven](https://central.sonatype.org/)

## 前提
首先，需要登陆这个官网。可以直接用 **github** 账号，这样后面会省一些事情（namespace）



### 必要准备：
   1. 你的项目build后的jar
   2. 你的项目build后的源码（source）jar
   3. 你的项目的java doc
   4. 你的项目maven build后的pom，并且pom中需要尽可能详细的填写一些个人信息以及描述
   5. 以上每一份文件，都需要附带对应的一份md5，sha1，gpg加密文件
   6. 将以上所有文件打包在一个zip中。文件结构需要与 groupId -> artifactId -> version 的结构相吻合

### 步骤
1. 登陆[maven官网](https://central.sonatype.org/) （建议使用github账号直登）
2. 打开右上角个人信息里面的 **View Deploy** ![View Deploy](unify\maven_upload\image.png)
3. 此时会看到页面上有两个框，一个NameSpace，一个Deployment。其中，Namespace是类似域名的东西，相当于你需要有一个自己的专属域名。如果你是使用github账号登陆的话，那么这里会自动给你一个validate的github的io域名，可以直接使用。如果没有，那需要自己申请域名然后来这里验证
   ![alt text](unify\maven_upload\image-1.png)
4. 在Deployments页面，就可以看到你当前所有的deploy了
5. 点击右上角的Publish Deployment
   ![alt text](unify\maven_upload\image-2.png)
   会出现一个form表单。填写完标题，description，再加上刚才打包的zip提交上去即可完成。
6. deployment会经过validating的阶段。等validate完成后，就可以对当前的deployment进行发布（publish）了

## 一些坑点
主要坑点其实都集中在打zip包中。
1. zip包中文件需要按照groupId -> artifactId -> version 的结构来放置。
    比如我的项目groupId是 com.baidu, artifactId 是 search_demo, version 是 0.1.0，那么，首先需要建立一个com文件夹，下面再来一个baidu文件夹，然后再来一个search_demo文件夹，最后再来个0.1.0文件夹，最后把之前提到的所有东西放在这个version文件夹中。然后，对最外面的com文件夹进行打zip包，最后可能得到一个``com.zip``,

2. 关于md5  sha1加密文件，没什么好说的，我是自己写了段代码来执行的：
   ```java
    public static void main(String[] args) throws NoSuchAlgorithmException, IOException {
        encryptFile("path");
    }

    public static void encryptFile(String fileName) throws NoSuchAlgorithmException, IOException {
        // Encrypt file
        MessageDigest MD5 = MessageDigest.getInstance("MD5");
        MessageDigest SHA_1 = MessageDigest.getInstance("SHA-1");


        File file = new File(fileName);
        if (file.isDirectory()) {
            File[] files = file.listFiles();
            for (File f : files) {
                if (f.isFile()) {
                    String name = f.getName();
                    byte[] bytes = Files.readAllBytes(f.toPath());
                    MD5.update(bytes);

                    byte[] digest = MD5.digest();
                    String newFileName = fileName + File.separator + "md5" + File.separator + name + ".md5";
                    String md5Content = byteArrToHexStr(digest);
                    Files.write(new File(newFileName).toPath(), md5Content.getBytes(StandardCharsets.UTF_8), StandardOpenOption.CREATE, StandardOpenOption.WRITE, StandardOpenOption.TRUNCATE_EXISTING);

                    SHA_1.update(bytes);
                    digest = SHA_1.digest();
                    newFileName = fileName + File.separator + "sha1" + File.separator + name + ".sha1";
                    String sha1Content = byteArrToHexStr(digest);
                    Files.write(new File(newFileName).toPath(), sha1Content.getBytes(StandardCharsets.UTF_8), StandardOpenOption.CREATE, StandardOpenOption.WRITE, StandardOpenOption.TRUNCATE_EXISTING);
                }
            }
        }

    }

    private static String byteArrToHexStr(byte[] digest) {
        StringBuilder sb = new StringBuilder();
        for (byte b : digest) {
            sb.append(String.format("%02x", b));
        }
        return sb.toString();
    }

   ```

3. **最大坑点，gpg加密**
   首先说一句，如果是在windows系统操作，不要用下载的gpg图形化操作界面。这个界面虽然很直观，但是加密出来的文件后缀是GPG格式的，然而maven那边要求的是ASC格式的。因为我对于这个加密不太了解，所以不太清楚区别以及如何互相转化，反证用图形化界面弄出来的maven不会识别的
   关于GPG，可以在[这里](https://gnupg.org/download/index.html#sec-1-2)下载。
   下载后，进入我们存放上传文件的文件夹，然后打开命令行，执行下列命令

```shell
gpg --version # 查看版本

gpg --gen-key # 生成一个key，这一步需要填写自己的名字，email，并且需要设置一个密码（passphrase）

gpg --list-keys # 查看当前所有的key

#加密文件
gpg -ab myfile.java

# 这里，参数 -a 表示生成ASCII的输出，即asc文件。 -b表示生成一个文件存放签名信息

```

然后，我们就会得到一个gpg签名过的文件： myfile.java.asc。把这个文件放在zip包与源码同级目录下即可。
至此，我们的打包工作完成，zip包中所有的东西均已准备完成。
但还有一件事，就是我们刚才生成的密钥对，目前是只有我们知道pub公钥的，别人没有的，因此，别人拿到你的签名文件也无法识别，无法解密，这会导致你上传的project在maven验证失败。
因此，我们还需要进行一步操作，那就是上传我们自己的``公钥(publicKey)``
```shell
# 首先，查询我们密钥的公钥
gpg --list-keys

我们会得到类似下面的输出：
pub   xxxxx 2025-03-30 [SC] [expires: 2027-03-30]
      AAA80322283FDD52B363D36E4F9B3F67BEDB9576
uid           xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
sub   xxxx 2025-03-30 [E] [expires: 2027-03-30]

其中我没有屏蔽的AAA80322283FDD52B363D36E4F9B3F67BEDB9576就是公钥，我们是可以把这个分享给别人的。
gpg还有一些公共服务器来分享这些公钥，并且maven就是在这里获取公钥进而来识别你签名文件的。

# 上传公钥操作：
 gpg --keyserver keyserver.ubuntu.com --send-keys ${你自己的公钥}

```
上传完毕公钥完毕后，我们就可以去maven那边来上传我们的zip并且publish我们的项目了。
最后，看到这个状态即表示上传成功，okkk
![alt text](unify\maven_upload\image-3.png)
