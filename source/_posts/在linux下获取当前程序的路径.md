---
title: 在linux下获取当前程序的路径
date: 2019-10-17 10:48:24
tags: linux
---

在Windows下获取当前程序的路径可以调用`GetModuleFileName()`函数，在linux下没有直接的api函数来实现此功能，因此需要自己来实现。这个功能挺简单的。

```cpp
    std::string GetCurrentPath(){
        char name[100] = {0};
        char path[PATH_MAX + 1] = {0};
        sprintf(name, "/proc/%d/exe", getpid());
        readlink(name, path, 1024);
        return std::string(path);
    }
```

