# 编译 Flutter Engine

## 环境准备

- Linux, Mac OS X, Windows
  - Windows 不支持 Android 和 iOS 交叉编译
  - Linux 不支持 iOS 交叉编译
- IDE，建议使用 Android Studio 和 CLion（用于编写/查看 C++ 代码）
- depot_tools，用来编译管理 engine 代码
- ant，用来管理/编译 Java 依赖
- Python, JDK, git, ninja

## 安装 depot_tools

```shell
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

并将 depot_tools 导出到环境变量：

```shell
export PATH=$PATH:/path/of/depot_tools
```

## 下载引擎代码

- 首先 fork `https://github.com/flutter/engine` 到你的 github 仓库
- 创建 `engine` 空文件夹用来拷贝 flutter 引擎的代码，**不要改成其他名字**：有些工具会默认使用这个名字，你改成其他名字也可以，但是可能需要你配置，有额外的麻烦
- 在 `engine` 文件夹下创建 `.gclient` 文件，将 `<your_name_here>` 替换成你的 github 账号名

```shell
solutions = [
  {
    "managed": False,
    "name": "src/flutter",
    "url": "git@github.com:<your_name_here>/engine.git",
    "custom_deps": {},
    "deps_file": "DEPS",
    "safesync_url": "",
  },
]
```

- 执行 `gclient sync` 命令将代码同步下来
- 切换到 `src/flutter` 目录，设置 upstream 到官方仓库

```shell
git remote add upstream git@github.com:flutter/engine.git
```

此时你本地仓库关联的远程仓库应该有 flutter 官方的地址：

```shell
git remote -v
# 应该输出：
# origin	git@github.com:<YOUR NAME>/engine.git (fetch)
# origin	git@github.com:<YOUR NAME>/engine.git (push)
# upstream	git@github.com:flutter/engine.git (fetch)
# upstream	git@github.com:flutter/engine.git (push)
```

## 编译

首先切换到 `src` 目录

- 编译 host

执行以下命令：

```shell
./flutter/tools/gn --unoptimized
ninja -C out/host_debug_unopt
```

- 编译 iOS

```shell
./flutter/tools/gn --ios --unoptimized
```

- 编译 Android

```shell
./flutter/tools/gn --android --unoptimized
```

~~对于 Apple Silicon，你会惊喜的得到这个报错：~~

```shell
ERROR at //build/config/android/config.gni:55:5: Assertion failed.
    assert(false, "Need Android toolchain support for your build CPU arch.")
    ^-----
```

~~原因是目前还不支持在苹果新芯片上交叉编译 Android。(TODO)~~

更新（2022.7.30）：

从 `upstream` 更新最新代码后已经支持在苹果新芯片上交叉编译 Android，没有查证啥时候支持了。

参考以下选项来编译不同架构的目标：

```
  --target-os {android,ios,mac,linux,fuchsia,wasm,win}
  --android
  --android-cpu {arm,x64,x86,arm64}
  --ios
  --ios-cpu {arm,arm64}
  --mac
  --mac-cpu {x64,arm64}
  --simulator
  --linux
  --fuchsia
  --wasm
  --windows
  --linux-cpu {x64,x86,arm64,arm}
  --fuchsia-cpu {x64,arm64}
  --windows-cpu {x64,arm64,x86}
  --simulator-cpu {x64,arm64}
```

更多参数可参考 `./flutter/tools/gn --help`。

## 编译 Web Engine

编译 Web Engine 使用 `felt`（Flutter Engine Local Tester 的缩写）工具，首先导出 `felt` 到 PATH：

```shell
export PATH=$PATH:<ENGINE_ROOT>/src/flutter/lib/web_ui/dev
```

ENGINE_ROOT 为引擎根目录。`felt` 的使用形式为：`felt <command>`，其中 `command` 可以是：

- `build`，编译引擎
- `test`，测试引擎
- `help`，获取帮助

要获取子命令的使用帮助，执行 `felt help <command>`。

运行测试项目：

```shell
flutter run -d chrome --local-engine-src-path <ENGINER_ROOT>/src\
--local-engine=<ENGINER_ROOT>/src/out/host_debug_unopt\
--web-renderer canvaskit
```

其中，`--web-renderer` 可为：

- `canvaskit`，使用 CanvasKit 渲染器
- `html`，使用 HTML 渲染器

## 编译 Skia

`Cavaskit` 将 `skia` 编译成 wasm，web engine 默认使用预编译好的 `Canvaskit` 渲染器（release 模式，没有 dwarf 符号，无法调试），如果想要调试 `Canvaskit`，需要手动编译。

单独编译 skia 库：

```shell
cd <ENGINE_ROOT>/src/third_party/skia
python3 tools/git-sync-deps
./bin/gn gen out/Debug
ninja -C out/Debug
```

编译 CanvasKit:

```shell
cd <ENGINE_ROOT>/src/third_party/skia/modules/canvaskit
./compile.sh debug_build
```

`debug_build` 编译为 Debug 包，方便查看生成的 js 代码，如果需要查看 CanvasKit 对 js 的接口，这非常有用。

执行以下命令，把生成的 wasm 和 js 文件拷贝到 build 目录下，编译 example：

```shell
# The following installs all npm dependencies and only needs to be when setting up
# or if our npm dependencies have changed (rarely).
npm ci

make release  # make debug is much faster and has better error messages
make local-example
```

- [how to build skia](https://skia.org/docs/user/build/)
- [build canvaskit](https://github.com/google/skia/tree/main/modules/canvaskit) 

## 在 IDE 中查看引擎代码

在 host 编译完成之后，执行以下命令：

```shell
cd src/flutter && ln ../out/host_debug_unopt/compile_commands.json compile_commands.json
```

此时使用 CLion 打开 src/flutter 就能愉快的修改/查看引擎代码了（当然你也可以直接打开，但是没有代码提示和跳转功能）。

关于 `compile_commands.json`，可以参考 [Compilation database](https://clion.jetbrains.com/help/c/external-tools/compile-commands.html) 和 [JSON Compilation Database Format Specification](https://clang.llvm.org/docs/JSONCompilationDatabase.html)。

VSCode 也支持 `compilation database` 功能，直接打开工程目录即可。

对于 Web engine 代码，直接使用 Android Studio 打开 `<ENGINE_ROOT>/src/flutter/lib/web_ui` 即可。

## 参考

- [Contributing to the Flutter engine](https://chromium.googlesource.com/external/github.com/flutter/engine/+/b7358b33dbd61e124720165dd939fa49cbd0ecb6/CONTRIBUTING.md)
- [Setting up the Engine development environment](https://github.com/flutter/flutter/wiki/Setting-up-the-Engine-development-environment)
- [Compiling the engine](https://github.com/flutter/flutter/wiki/Compiling-the-engine)
- [Flutter web engine](https://github.com/flutter/engine/blob/main/lib/web_ui/README.md)

## 编译 dart-sdk

```shell
./tools/build.py --no-goma --mode debug --arch x64 --export-compile-commands create_sdk
```

**`--export-compile-commands` 将导出 [Compilation database](https://clion.jetbrains.com/help/c/external-tools/compile-commands.html) 到 compile_commands.json 文件，没有这个参数查看 dart vm 的代码比较费劲。** 参考[这个 PR](https://groups.google.com/a/dartlang.org/g/reviews/c/fFImE0AQ6z8)。
