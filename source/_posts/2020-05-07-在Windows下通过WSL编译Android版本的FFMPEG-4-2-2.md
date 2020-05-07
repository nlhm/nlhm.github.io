---
title: 在Windows下通过WSL编译Android版本的FFMPEG 4.2.2
date: 2020-05-07 09:24:58
tags: FFMPEG
---

本文讲述了在Windows下通过WSL编译Android版本使用的FFMPEG库文件。

<!--more-->

[toc]

写在前面：本文是在编译成功之后边回忆边写的，可能有细节上的偏差。

## 配置环境

WSL ：Ubuntu，已安装NASM。

>  NASM在Ubuntu下安装方式:`sudo apt-get install nasm`

NDK : android-ndk-r12b-linux-x86_64

> 最新版的NDK（21）移除了GCC编译并采用了clang，但是FFMPEG中的Configure文件依然采用gcc，所以为了方便使用的是老版本的NDK。NDK不同版本的改动可以参考官网的[NDK 修订历史记录](https://developer.android.google.cn/ndk/downloads/revision_history?hl=zh-cn)

源码：[x264](http://www.videolan.org/developers/x264.html)和[FFMPEG](https://ffmpeg.org/download.html#get-sources)

编译脚本：[链接](https://github.com/RoyGuanyu/build-scripts-of-ffmpeg-x264-for-android-ndk)

## 编译x264

首先配置脚本文件，以`build_android_arm64-v8a.sh`为例。

* NDK：android-ndk-r12b-linux-x86_64的存放位置。注意在WSL下加载Windows盘符的格式。

* --preffix：编译完成之后生成的库文件的存放位置。
* --host：编译生成的库将要运行的平台。

```sh
#!/bin/bash
NDK=/mnt/d/DevelopTools/android-ndk-r12b-linux-x86_64/android-ndk-r12b
PLATFORM=$NDK/platforms/android-21/arch-arm64/
TOOLCHAIN=$NDK/toolchains/aarch64-linux-android-4.9/prebuilt/linux-x86_64
PREFIX=./android/arm64

function build_one
{
  ./configure \
  --prefix=$PREFIX \
  --enable-static \
  --enable-pic \
  --host=aarch64-linux \
  --cross-prefix=$TOOLCHAIN/bin/aarch64-linux-android- \
  --sysroot=$PLATFORM

  make clean
  make
  make install
}

build_one

echo Android ARM64 builds finished

```

接下来在该目录下通过WSL运行脚本就可以了。

## 编译FFMPEG

首先按照上述方式配置FFMPEG的脚本文件。记得按照脚本文件中的位置存放X264的库文件或者自己修改脚本中的路径。

之后要修改FFMPEG自带的configure文件。因为默认编译出的so库是带有版本号的，在安卓工程中是使用不了的。

在Configure文件中找到如下代码：

```cpp
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'
```

然后将其修改为：

```cpp
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
```

然后执行脚本。

## 错误

### 错误一 request for member 's_addr' in something not a structure or union

```c++
libavformat/udp.c: In function 'udp_set_multicast_sources':
libavformat/udp.c:290:28: error: request for member 's_addr' in something not a structure or union
         mreqs.imr_multiaddr.s_addr = ((struct sockaddr_in *)addr)->sin_addr.s_addr;
                            ^
libavformat/udp.c:292:32: error: incompatible types when assigning to type '__be32' from type 'struct in_addr'
             mreqs.imr_interface= ((struct sockaddr_in *)local_addr)->sin_addr;
                                ^
libavformat/udp.c:294:32: error: request for member 's_addr' in something not a structure or union
             mreqs.imr_interface.s_addr= INADDR_ANY;
                                ^
libavformat/udp.c:295:29: error: request for member 's_addr' in something not a structure or union
         mreqs.imr_sourceaddr.s_addr = ((struct sockaddr_in *)&sources[i])->sin_addr.s_addr;
                             ^
ffbuild/common.mak:59: recipe for target 'libavformat/udp.o' failed
make: *** [libavformat/udp.o] Error 1
CC      libavformat/udp.o
libavformat/udp.c: In function 'udp_set_multicast_sources':
libavformat/udp.c:290:28: error: request for member 's_addr' in something not a structure or union
         mreqs.imr_multiaddr.s_addr = ((struct sockaddr_in *)addr)->sin_addr.s_addr;
                            ^
libavformat/udp.c:292:32: error: incompatible types when assigning to type '__be32' from type 'struct in_addr'
             mreqs.imr_interface= ((struct sockaddr_in *)local_addr)->sin_addr;
                                ^
libavformat/udp.c:294:32: error: request for member 's_addr' in something not a structure or union
             mreqs.imr_interface.s_addr= INADDR_ANY;
                                ^
libavformat/udp.c:295:29: error: request for member 's_addr' in something not a structure or union
         mreqs.imr_sourceaddr.s_addr = ((struct sockaddr_in *)&sources[i])->sin_addr.s_addr;
                             ^
ffbuild/common.mak:59: recipe for target 'libavformat/udp.o' failed
make: *** [libavformat/udp.o] Error 1
```

出现上述错误的原因是：在此版本的NDK中，`ip_mreq_source`的定义如下。

```c
struct ip_mreq_source {
/* WARNING: DO NOT EDIT, AUTO-GENERATED CODE - SEE TOP FOR INSTRUCTIONS */
 __be32 imr_multiaddr;
 __be32 imr_interface;
 __be32 imr_sourceaddr;
};
```

为了解决此问题，在报错中提到的函数`udp_set_multicast_sources`前面加上自定义的结构体，并替换出错的结构体。

```c
struct my_ip_mreq_source {
               struct in_addr imr_multiaddr;  /* IP multicast group address */
               struct in_addr imr_interface;  /* IP address of local interface */
               struct in_addr imr_sourceaddr; /* IP address of multicast source */
};
```

```c
#if HAVE_STRUCT_IP_MREQ_SOURCE && defined(IP_BLOCK_SOURCE)
    for (i = 0; i < nb_sources; i++) {
        struct ip_mreq_source mreqs;
----->   ----->    ----->
#if HAVE_STRUCT_IP_MREQ_SOURCE && defined(IP_BLOCK_SOURCE)
    for (i = 0; i < nb_sources; i++) {
        struct my_ip_mreq_source mreqs;
```

### 错误二 error: expected identifier or '(' before numeric constant int B0 = 0, B1 = 0;

```cpp
libavcodec/aaccoder.c: In function 'search_for_ms':
libavcodec/aaccoder.c:803:25: error: expected identifier or '(' before numeric constant
                     int B0 = 0, B1 = 0;
                         ^
libavcodec/aaccoder.c:865:28: error: lvalue required as left operand of assignment
                         B0 += b1+b2;
                            ^
libavcodec/aaccoder.c:866:25: error: 'B1' undeclared (first use in this function)
                         B1 += b3+b4;
                         ^
libavcodec/aaccoder.c:866:25: note: each undeclared identifier is reported only once for each function it appears in
make: *** [libavcodec/aaccoder.o] Error 1
```

编译时出现以上错误。原因是在` /usr/arm-linux-androideabi/include/asm/termbits.h`中已经定义`#define B0 0000000 `，所以在此处重复定义故而报错。将`B0`修改为`b0`即可。按照同样的方法还需处理`libavcodec/hevc_mvs.c`(B0->b0, xB0->xb0, yB0->yb0)和`libavcodec/opus_pvq.c`(B0->b0)。

## 完成

至此，编译通过。



>参考链接：
>
>https://yesimroy.gitbooks.io/android-note/content/compile_ffmpeg_for_android.html
>
>http://alientechlab.com/how-to-build-ffmpeg-for-android/
>
>https://www.jianshu.com/p/e3fb667dacae

