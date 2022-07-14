# Flutter 引擎开发

## 前置条件

同 Flutter 引擎编译

- Linux, Mac OS X, Windows
  - Windows 不支持 Android 和 iOS 交叉编译
  - Linux 不支持 iOS 交叉编译
- IDE，建议使用 Android Studio 和 CLion（用于编写/查看 C++ 代码）
- depot_tools，用来编译管理 engine 代码
- ant，用来管理/编译 Java 依赖
- Python, JDK, git, ninja

## 配置 Flutter Framework 开发环境

- fork [Flutter framework](https://github.com/flutter/flutter) 代码到你个人的 github
- clone 刚 fork 的仓库到本地，切换开发分支到 dev
- 设置 upstream 到 flutter 官方仓库（当然这步可以不做，但是为了从官方同步代码，建议设置）
- 导出 fork 下来的 Flutter bin 到环境变量
- 执行 flutter packages upgrade, flutter packages get

参考以下命令:

```shell
git clone <your-flutter-repository>
git remote add upstream https://github.com/flutter/flutter
git checkout origin/dev -b dev
export PATH=<your-local-flutter-repository>/bin:$PATH
cd <your-local-flutter-repository>/packages/flutter
flutter packages upgrade && flutter packages get
```

- 在 Android Studio 中导入 Flutter framework 源码，注意使用 `<your-local-flutter-repository>/packages/flutter`。

## 配置 Flutter engine 开发环境

参考编译 flutter engine，直接在 CLion 中打开 engine 项目即可。

> ⚠️注意如果同时涉及到修改 framework 和 engine 的代码，要保持 framework 和 engine 代码版本一致。
> 一般来说，framework 基于 `main` 分支，engine 基于 `master` 分支。

## 修改 Engine 代码

以 `<your-engine-repository>/src/flutter/lib/ui/painting/canvas.h` 为例，在 `Canvas` 中添加一个 `drawDummy` 方法：

```c++
namespace flutter {
// ...

class Canvas {
  // ...
  void drawDummy();
};
```

在对应的 `canvas.cpp` 文件实现刚加的方法：

```c++
// 注册到 dart 层
#define FOR_EACH_BINDING(V)           \
  V(Canvas, save)                     \
  // ... omitted
  V(Canvas, drawDummy)

// ...
void Canvas::drawDummy() {
  // draw a rectangle for testing
  if (display_list_recorder_) {
    builder()->drawRect(SkRect::MakeXYWH(0, 0, 100, 100));
  } else if (canvas_) {
    canvas_->drawRect(SkRect::MakeXYWH(0, 0, 100, 100), SkPaint());
  }
}
```

修改对应的 dart 文件，`<your-engine-repository>/src/flutter/lib/ui/painting.dart`：

```dart
class Canvas extends NativeFieldWrapperClass1 {
  // ...
  void drawDummy() native 'Canvas_drawDummy';
}
```

重新编译引擎：

```shell
./flutter/tools/gn --unoptimized
ninja -C out/host_debug_unopt
```

⚠️这里仅编译了 host，若要编译 Android 或 iOS，执行对应的命令即可。

## 实验自定义引擎

创建一个新 flutter 项目，假设命名为 `hello_flutter`，设置 flutter engine 为你刚编译的 engine，设置编译环境，修改项目 pubspec：

```yaml
dependency_overrides:
  sky_engine:
    path: <your-engine-repository>/src/out/host_debug_unopt/gen/dart-pkg/sky_engine
```

尝试使用刚添加的 `drawDummy` 方法实现一个自定义 widget，创建文件 `dummy.dart`：

```dart
class Dummy extends LeafRenderObjectWidget {
  @override
  RenderObject createRenderObject(BuildContext context) {
    return RenderDummy();
  }
}

class RenderDummy extends RenderBox {
  @override
  void performLayout() {
    size = const Size(100, 100);
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    context.canvas.drawDummy();
  }
}
```

在 main.dart 中使用 dummy widget，通过以下命令运行，项目将使用本地 engine，以 macOS 为例：

```shell
flutter run -d macOS --local-engine-src-path <your-engine-repository>/src \
  --local-engine=<your-engine-repository>src/out/host_debug_unopt
```

根据官方文档：

> If you do this, you can omit --local-engine-src-path and not bother to set $FLUTTER_ENGINE, as the flutter tool will
> use these paths to determine the engine also! The tool tries really hard to figure out where your local build of the
> engine is if you specify --local-engine.

If you do this 中的 this，指的是配置 `dependency_overrides`，但是发现不管用，所以还是手动指定一下这俩参数。

也可以配置 Android Studio，在 `Edit configuration` 添加上述参数，通过 Run 来运行项目。

会发现报 `_MulticastCanvas` 没有实现 `drawDummy`
方法，找到文件 `<your-flutter-repository>/packages/flutter/lib/src/widgets/widget_inspector.dart` 实现 `drawDummy`
方法，会发现报错 `Canvas` 没这个方法 🤔。

跟我们新建的项目一样，需要修改 flutter framework 使用的引擎，在 pubspec 中添加：

```yaml
dependency_overrides:
  sky_engine:
    path: <your-engine-repository>/src/out/host_debug_unopt/gen/dart-pkg/sky_engine
```

再次更新 packages，实现 `drawDummy`。回到实验项目，编译运行。

## 参考

- [Compiling the engine](https://github.com/flutter/flutter/wiki/Compiling-the-engine)
- [The flutter tool](https://github.com/flutter/flutter/wiki/The-flutter-tool)
