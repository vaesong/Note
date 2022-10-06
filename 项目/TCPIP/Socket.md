# Linux

在vscode下，stoi 和 to_string 函数说是未声明

解决方法：编译时加上 

```C++
-std=c++11
```

其实是g++默认不支持c++11标准， 编译时在编译命令上加上 -std=c++11 就好咯~

# Windows

首先是构建项目的时候需要连接到静态库，类似头文件一样加入到开头

```C++
#pragma comment(lib,“ws2_32.lib”)
```

此外，还需要在 task.json 的 args 参数列表里面加入

```JSON
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: g++.exe 生成活动文件",
            "command": "D:\\ProfessionalSoftware\\Environment\\C++\\MinGW\\mingw32\\bin\\g++.exe",
            "args": [
                "-fdiagnostics-color=always",
                "-g",
                "${file}",
                //下面这行是 链接库 的，平时不需要，可以删掉
                "-l", "ws2_32",
                "-o",
                "${fileDirname}\\${fileBasenameNoExtension}.exe"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}
```





