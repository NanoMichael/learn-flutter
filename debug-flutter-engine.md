# 调试 Flutter 引擎

参考：

- [编译 flutter 引擎](compile-flutter-engine.md)
- [Flutter 引擎开发](flutter-engine-dev.md)

以下都以实验项目 `hello_flutter` 为例.

## 调试 MacOS

可以看到最终的打包产物位于 `<test-project>/build/macos/Build/Products/Debug/hello_flutter.app/Contents/MacOS/` 目录下，入口可执行文件为 `hello_flutter`。

### 使用命令行

直接在终端使用 lldb 调试（命令行 lldb 是真的难用，又没有类似 cgdb 这样的替代品）：

```shell
lldb <test-project>/build/macos/Build/Products/Debug/hello_flutter.app/Contents/MacOS/hello_flutter
```

以下是一些常用命令：

| 命令 | 说明                                              |
| ---- |-------------------------------------------------|
| b | 设置断点，例：`b canvas.cc:120`，在 canvas.cc 文件的 120 行打断点 |
| r | 启动程序，run                                        |
| bt | 查看调用栈，backtrace                                 |
| c | 继续执行，continue                                   |
| n | 执行下一步，next                                      |
| s | 执行下一步，step                                      |

更多命令可参考 [lldb](https://lldb.llvm.org/)。

当然也可以进入到 GUI 模式（在 lldb session 中输入 `gui` 进入，就是个半成品，最近更新时间还是 18 年，而且没有文档），如果对 Vim 熟悉的话，建议安装 Vim lldb 插件，调试还是很好用的。你也可以使用 [voltron](https://github.com/snare/voltron) 来查看内存、寄存器等调试信息。如何使用可参考[这个文档](https://lldb.llvm.org/use/map.html)。

### 配置 VSCode

VSCode 也支持 [JSON Compilation Database Format Specification](https://clang.llvm.org/docs/JSONCompilationDatabase.html) 项目，打开 engine 所在的目录即将引擎的工程导入到 VSCode 的 workspace 中。

需要安装的插件：

- [C/C++ for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
- [CodeLLDB](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb)

配置 `CodeLLDB` launch，在 `.vscode` 目录下新建 `launch.json` 文件，添加以下内容：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "local_lldb_debug",
      "type": "lldb",
      "request": "launch",
      "program": "<test-project>/build/macos/Build/Products/Debug/hello_flutter.app/Contents/MacOS/hello_flutter"
    }
  ]
}
```

参数解释：

- name: 启动配置的名称
- type: 启动类型，必须为 `lldb`
- request: 启动请求，这里为 `launch` 表示直接启动程序，关于远程调试在之后介绍。
- program: 程序路径

然后在需要调试的地方添加断点，启动调试即可。如图：

![img.png](imgs/debug-host-vscode.png)

关于 `CodeLLDB` 更多使用方式，参考[文档](https://github.com/vadimcn/vscode-lldb/blob/v1.7.0/MANUAL.md)。

### 配置 CLion

对于喜欢使用 CLion 的开发者：

- Attach 到已运行项目，**Run > Attach to Process**，将 debugger attach 到指定进程，如图：

![img.png](imgs/debug-clion.png)

- 配置 Run configuration，打开 **Edit Configurations** 选择 **Custom Build Application**，如图：

![img.png](imgs/debug-host-clion-custom.png)

选择编译好的实验项目，如图：

![img.png](imgs/debug-host-clion-custom2.png)

然后调试即可，非常简单方便。

## 调试 Android

## 调试 iOS

首选 xcode，配置非常简单，打开实验项目 `ios/Runner.xcworkspace`，保证 build configuration 为 debug，然后通过 **Debug > Breakpoints > Create Symbolic Breakpoint** 添加断点，如图：

![img.png](imgs/debug-ios-xcode.png)

编译运行即可。
