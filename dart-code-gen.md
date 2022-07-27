# Dart 代码生成

## 使用 source_gen

`source_gen` 是一个 Dart 库，用于生成 Dart 代码。它封装了 `dart build` 和 `dart analyzer` 等 low-level 的 API，对开发者更加友好。你不一定非得使用 source_gen 来生成代码，`dart build` 提供了更底层、更丰富的 API，可以完成更奇怪的需求。参考 [source_gen](https://pub.dev/packages/source_gen)。

### 一个例子

创建一个 Flutter 工程（当然不必是 Flutter 工程，纯 dart 工程也 OK），我给起名为 test_gen，在这个工程中创建子模块 test_gen_annotations 和 test_gen_generator，分别存放 annotation 和 generator 的代码。其结构为：

```
test_gen
├── lib
├── pubspec.yaml
├── test_gen_annotations
├── test_gen_generator
└── ...
```

test_gen 通过 path 依赖 test_gen_annotations 和 test_gen_generator。

> 在实践中，对工程这样分层也是合理的。

在 test_gen_generator 中引入 `source_gen`，在 `pubspec.yaml` 中添加：

```yaml
dependencies:
  source_gen: '^1.2.2' 
```

如果 generator 不需要发布，可以使用 `dev_dependency`:

```yaml
dev_dependencies:
  source_gen: '^1.2.2' 
```

在 test_gen_annotations 中添加类 `DummyAnnotation`，这个类没有任何意义，仅仅用来测试而已。注意：**Annotation 类构造器必须为 const (why?)**。

```dart
class DummyAnnotation {
  final String comment;
  final int id;

  const DummyAnnotation({required this.comment, required this.id});
}
```

然后在 test_gen 中添加类 `ADummyClass`：

```dart
@DummyAnnotation(
  comment: 'This is a dummy annotation',
  id: 1,
)
class ADummyClass {}
```

我们的目标是根据 annotation `DummyAnnotation` 生成如下代码：

```dart
// desc: $comment
// with a dummy id: $id
const DUMMY_COMMENT_${className} = "$comment";
```

（这自然也没任何意义，仅仅用来测试而已。）

Dart 很奇怪，一个 dart 源文件就是一个 library。

## Aggregate builder

## 收集子库注解

## 调试

![img.png](imgs/debug-build.png)

这里是 build 的入口文件，打开看下：

```dart
final _builders = <_i1.BuilderApplication>[
  _i1.apply(r'source_gen:combining_builder', [_i2.combiningBuilder],
      _i1.toNoneByDefault(),
      hideOutput: false, appliesBuilders: const [r'source_gen:part_cleanup']),
  _i1.apply(r'test_gen_generator:all_files_builder', [_i3.allFilesBuilder],
      _i1.toDependentsOf(r'test_gen_generator'),
      hideOutput: false),
  _i1.apply(r'test_gen_generator:test_builder', [_i4.testBuilder],
      _i1.toDependentsOf(r'test_gen_generator'),
      hideOutput: false),
  _i1.applyPostProcess(r'source_gen:part_cleanup', _i2.partCleanup)
];
void main(List<String> args, [_i5.SendPort? sendPort]) async {
  var result = await _i6.run(args, _builders);
  sendPort?.send(result);
  _i7.exitCode = result;
}
```

里边包含了我们刚写的 builders。

打开 `Edit Configurations`，配置 build：

![img.png](imgs/debug-build-config.png)

正常调试就行了。
