---
title: libcurl 编译使用
date: 2019-07-19 16:20:30tags:
- libcurl
categories: 
- 环境配置
copyright: true
---

工作的原因，需要使用libcurl库，这里整理一下过程，方便日后食用。（很大一部分来自网络资源，不喜勿喷）

#### 一、引言

cURL、libcurl 还有 curl，他们究竟是什么？

也就是说，我们需要去了解下 libcurl 及其相关的概念。

#### 二、cURL、libcurl 还有 curl 傻傻分不清楚 T_T

可能对于新人来说，就连 cURL、libcurl 和 curl 的概念都是分不清楚的。这不怪我们，确实关于这一点，官方网站都没有说的很清楚，但是在源代码中的 FAQ 文档中却说的非常明白。

```text
What is cURL? 
cURL is the name of the project. The name is a play on ‘Client for URLs’, originally with URL spelled in uppercase to make it obvious it deals with URLs. The fact ti can also be pronounced ‘see URL’ alse helped, it works as an abbreviation for “Client URL Request Library” or why not the recursive version: “Curl URL Request Library”.
```

简而言之，cURL 是一个项目的名称。是 Client for URLs、see URL、Client URL Request Library 或者 Curl URL Request Library 的缩写，也就是一个客户端 URL 请求库的项目。

那么什么是 libcurl 呢？

```text
The cURL project produces two products: 
libcurl 
A free and easy-to-use client-side URL transfer library. 
… 
curl 
A command line tool for getting or sending files using URL syntax.
```

上面这段话很清晰的表现出了 cURL 与 libcurl 以及 curl 的关系，也就是说：

cURL 这个项目包含了 libcurl 和 curl 两个产品。 
其中，libcurl 是一个客户端的 URL 支持库；而 curl 就是一个使用了 libcurl 库写出来的命令行工具，其可以使用 URL 标识来请求或者发送文件。

也就是说，如果我们想要编写代码来控制有关网络的行为的话，我们就需要使用到 libcurl 库而不是 curl 命令行工具；而如果我们想要直接调用 curl 命令行工具来完成一些操作，比如将其嵌入到脚本代码中去，那么这个时候，我们才会用到 curl 命令行工具。

现在搞清楚了一些必要的概念，以及一些有趣的题外话，现在让我们来看看，我们要在 Unix 环境和 Windows 环境下编译和使用 libcurl，我们需要哪些东西，以及我们能在哪些地方获取到这些东西。

#### 三、资料的获取

我们想要了解 libcurl 这个库，最直接的资源获取来源当然是官方网站： 

[curl官网](https://curl.haxx.se/)

可能对于新手来说，一点开就会觉得有些迷茫，因为可能分不清楚 curl 和 libcurl 的区别。这也是为什么我一定要在介绍资料来源之前介绍 cURL、libcurl 和 curl 三者的区别的原因。相信在我上一节花了那么大篇幅来介绍这三者的区别之后，你应该不会那么迷茫了。 

对于我们开发者来说，了解 libcurl 应该是最重要的。因为我们是想要使用 libcurl 库来编写代码的，而不是来学习 curl 的使用方法的。

通过多次尝试的我最后发现，下载 Source Archives 也就是源代码的版本，是最好的。原因嘛，当你下载下来之后，解压到本地看看里面的内容你就知道了，因为它，实在是太完善了： 


简单的说几个：

1. docs 文件夹 
这里面有丰富的说明文档以及 libcurl 的运行示例代码。 
有关 cURL、libcurl 以及 curl 的概念的定义，就是在这个文件夹下的 FAQ 文件里面。后面将要讲述的在 Unix 下编译使用 libcurl 库的内容，也是来源于这个文件夹下的 INSTALL.md 文件。除此之外，这个文件夹下还有很多说明文档，有待大家去探索去发现去思考去使用。 
这个文件夹下的 examples 文件夹下，有着丰富的示例代码，其中的 https.c 就是本篇博客的测试运行代码。

2. winbuild 文件夹 
这个文件夹介绍了如何使用 Visual Studio 编译 libcurl 的方法。基于 Windows 环境的编译与使用就是参考的这个文件夹下的 BUILD.WINDOWS.txt 文件中的内容。 
并且这个文件夹下提供了编译的配置信息文件，大大方便了我们在 Windows 下使用 Visual Studio 编译 libcurl 的工作。

3. configure 
这个脚本文件用于在 Unix 下配置 libcurl 的安装信息，用来之后安装 libcurl 环境使用。

等等等等，curl 的源代码文件中，包含了很多很多东西。有很多你可能接触不到，我们可以在学习中在使用中慢慢去发掘去使用。

#### 四、libcurl 在 Unix 环境下的编译

让我们步入正题吧，libcurl 在 Unix 环境下怎么编译与使用呢？

这个问题在 curl-7.61.0\docs 下的 INSTALL.md 文件中讲述的非常清晰：

> A normal Unix installation is made in three or four steps (after you’ve unpacked the source archive): 
> ./configure 
> make 
> make test (optional) 
>
> make install

也就是说，在类 Unix 环境下，我们都可以在源代码文件目录下使用以下四句指令完成 libcurl 库的安装与编译：

```
$ ./configure
$ make
$ make test (optional)
$ make install
```

其中第三步，也就是测试那步是非常耗时间的，为了节约时间可以省略。 
另外第四步，可能会涉及到权限问题，如果出现这个问题，需要切换到 root 权限安装。

简单总结下步骤吧：

1. 使用 WinSCP 以及类似的工具，将下载下来的源代码文件放置到指定目录下，比如我现在将下载下来的 curl-7.61.0.zip 文件放到了 RedHat 7.2 环境中的 /home/wangying/libcurl 文件夹下

2. 解压上一步放置的源代码文件

   ```
   $ unzip curl-7.61.0.zip
   ```


3. 解压完成后进入 curl-7.61.0 文件夹下，运行上述所说的 4 条指令：

   ```
   $ ./configure
   $ make
   $ make test (optional)
   $ make install
   ```

   建议大家省去第三步，因为实在太耗费时间了（T_T）。

4. 执行完了之后，现在直接运行 curl 还是会显示系统默认自带的版本。我们需要进入到默认安装的目录下 `/usr/local/bin` 中使用下列命令运行：

   ```
   $ ./curl --version
   $ ./curl-config --version
   ```

![libcurl_1](libcurl-编译使用\libcurl_1.png)

上述两行代码会输出 curl 以及 curl-config 两个工具的版本信息，与你下载的版本一致即为安装成功。根据我的试验经验，一旦你运行了上述两行代码，系统自动会记录下来这两个工具的路径，不过为了确保系统找得到这两个工具，你可以单独设置下环境变量。

5. libcurl 库的相关头文件与库文件所在地方，可以通过下列命令查看（这会在下一节中的使用中提到）

   ```
   $ curl-config --cflags
   $ curl-config --libs
   ```

![libcurl_2](libcurl-编译使用\libcurl_2.png)

其中 –cflags 输出的我们编写代码时需要包含的头文件路径， –libs 输出的是我们编写代码时需要包含的库文件路径，我们只需要将上述两行简单的添加到编译指令中即可运行我们的示例代码。

#### 五、libcurl 在 Unix 环境下的使用

让我们点开源代码 docs 目录下的 examples 文件夹，其中的 README 文件详细介绍了 libcurl 示例代码的使用方式：

>Most examples should build fine using a command line like this: 
>$ curl-config --cc --cflags --libs -o example example.c 
>Some compilers don’t like having the arguments in this order but instead want you do reorganize them like: 
>
>$ `curl-config --cc` -o example example.c `curl-config --cflags --libs`

也就是说，只要我们在上一步中成功安装了 curl 以及 curl-config 工具，在这一步中，我们只需要简单的运行这行指令即可自动的指定代码的包含头文件以及库文件信息：
>$ `curl-config --cc` -o example example.c `curl-config --cflags --libs`

让我们来尝试下 examples 中的 https.c 文件的编译（因为 https.c 文件可以在源代码中看到，这里就不再详细展示文件内容）：

>$ `curl-config --cc` -o https https.c `curl-config --cflags --libs`

![libcurl_3](libcurl-编译使用\libcurl_3.png)

可见，https.c 的运行是非常成功的，成功返回了获取的 html 信息。

同样的示例我在 Ubuntu 18.04上运行通过，相信在类 Unix 上应该都是没有问题的。

#### 六、libcurl 在 Windows 环境下的编译

libcurl 在 Windows 环境下的编译，在源代码文件夹下的 docs 文件夹下的 INSTALL 文件中有所提及，但是讲述的稍微有些晦涩难懂。欣喜的是源代码根目录下直接提供了一个 winbuilds 文件夹方便我们完成在 Windows 环境下的编译工作。

这里，让我们看看 winbuild 文件夹下的 BUILD.WINDOWS.txt 文件的内容（这个文件中的内容非常详尽，这里不再粘贴出来）。

我简单总结下步骤：

1. 运行 Developer Command Prompt for VS 工具 


2. 进入到源代码文件夹下的 winbuild 目录下

3. 运行下列指令：

   ```
   nmake /f Makefile.vc mode=<static or dll> <options>
   ```

   其中的 mode= 后面填写 static 是编译静态库， dll 是动态库；另外 options 的填写说明如下：

   ```text
   where <options> is one or many of:
     VC=<6,7,8,9,10,11,12,14,15>    - VC versions
     WITH_DEVEL=<path>              - Paths for the development files (SSL, zlib, etc.)
                                      Defaults to sibbling directory deps: ../deps
                                      Libraries can be fetched at http://windows.php.net/downloads/php-sdk/deps/
                                      Uncompress them into the deps folder.
     WITH_SSL=<dll or static>       - Enable OpenSSL support, DLL or static
     WITH_NGHTTP2=<dll or static>   - Enable HTTP/2 support, DLL or static
     WITH_MBEDTLS=<dll or static>   - Enable mbedTLS support, DLL or static
     WITH_CARES=<dll or static>     - Enable c-ares support, DLL or static
     WITH_ZLIB=<dll or static>      - Enable zlib support, DLL or static
     WITH_SSH2=<dll or static>      - Enable libSSH2 support, DLL or static
     ENABLE_SSPI=<yes or no>        - Enable SSPI support, defaults to yes
     ENABLE_IPV6=<yes or no>        - Enable IPv6, defaults to yes
     ENABLE_IDN=<yes or no>         - Enable use of Windows IDN APIs, defaults to yes
                                      Requires Windows Vista or later
     ENABLE_WINSSL=<yes or no>      - Enable native Windows SSL support, defaults to yes
     GEN_PDB=<yes or no>            - Generate Program Database (debug symbols for release build)
     DEBUG=<yes or no>              - Debug builds
     MACHINE=<x86 or x64>           - Target architecture (default is x86)
     CARES_PATH=<path to cares>     - Custom path for c-ares
     MBEDTLS_PATH=<path to mbedTLS> - Custom path for mbedTLS
     NGHTTP2_PATH=<path to HTTP/2>  - Custom path for nghttp2
     SSH2_PATH=<path to libSSH2>    - Custom path for libSSH2
     SSL_PATH=<path to OpenSSL>     - Custom path for OpenSSL
     ZLIB_PATH=<path to zlib>       - Custom path for zlib
   ```

   大家可根据需要填写，这里我填写的命令是：

   ```
   nmake /f Makefile.vc mode=static
   ```

4. 待编译完成，进入源代码文件夹下，其中多出来了一个 builds 文件夹，其中名字长度最短的那个点击进去，可以看到 bin、include 和 lib 子文件夹，其中 bin 中就是编译出来的 curl 命令行工具，include 就是我们在编写代码中需要包含的头文件，lib 就是我们在编写代码中需要包含的静态库文件（不过仍然需要 CRT 静态库的另外链接） 


至此，使用 Visual Studio 2017 编译工具静态编译 libcurl 库的工作就算是完成了。这里值得注意的是，Windows 平台下的编译选项有很多，我们可以根据需要配置自己想要的 libcurl 库。比如静态动态，比如使用 VC 12 13 14 15 版本等等等等。

在这篇博客里，主要展示了使用静态库的方式。接下来，我们来看看如何使用刚刚编译出来的这些文件进行示例代码的运行使用。

#### 七、libcurl 在 Windows 环境下的使用

这里，我简单总结下步骤：

1. 使用 Visual Studio 2017 新建一个 Visual C++ 的空项目，并将示例代码 https.c 拷贝到项目中去

2. 为了使得我们的 https.c 代码运行后控制台不会一闪而过，我们需要修改 https.c 的代码，不多，只需要添加两行即可：

// 添加在 include 的地方
#include <stdlib.h>
// 添加在 return 语句之前
system("pause");

这样，就可以使得控制台窗口运行完代码逻辑之后暂停，不至于关闭掉窗口了。

3. 将上一节中生成的 include 文件夹以及 lib 文件夹拷贝到项目中去 


4. 配置 VC++ 目录：需要让代码能够找到需要包含的头文件和静态库文件 
    包含目录添加：

  ```
  // $(SolutionDir) 是当前的解决方案目录
  $(SolutionDir)include
  ```

  库目录添加：

  ```
  // $(SolutionDir) 是当前的解决方案目录
  $(SolutionDir)lib
  ```

  ![libcurl_4](libcurl-编译使用\libcurl_4.png)


5. 配置需要链接的静态库文件 链接器->输入->附加依赖项

  ```
  libcurl_a.lib
  Ws2_32.lib
  Wldap32.lib
  winmm.lib
  Crypt32.lib
  Normaliz.lib
  ```

  ![libcurl_5](libcurl-编译使用\libcurl_5.png)


6. 修改预处理器定义：

C/C++ -> 预处理器 -> 预处理器定义

```
// 添加 
CURL_STATICLIB;
```


7. 编译运行，顺利的话能够看到结果如下 

   ![libcurl_6](libcurl-编译使用\libcurl_6.png)


至此，完结撒花，^_^

