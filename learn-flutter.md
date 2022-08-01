# Flutter 学习笔记

学习 flutter 过程中的一些碎碎念

## dart

[A tour of the Dart language](https://dart.dev/guides/language/language-tour)

有一些有趣的东西。

- 判断两个对象相同，`identical`
- 两个 `const` 对象，在运行时是 `identical` 的
- `factory` 关键字定义一个工厂函数，语法层面上支持了工厂模式
- `external` 关键字修饰函数，表示该函数的定义与实现是分开的，通常用来针对不同平台做具体实现，类似与 Kotlin 的 `expect` 还有 `actual`，没有细究。参考：Kotlin [Connect to platform-specific APIs](https://kotlinlang.org/docs/multiplatform-connect-to-apis.html)

### [Runes and grapheme clusters](https://dart.dev/guides/language/language-tour#runes-and-grapheme-clusters)

简单来说，`Runes` 计算字符串的 Unicode 码点。有个词很有意思：`user-perceived characters`，**用户可感知的字符**，又叫 [Unicode (extended) grapheme cluster](https://unicode.org/reports/tr29/#Grapheme_Cluster_Boundaries)，Unicode 字形（叫字素可能更合适）簇。Dart 使用 UTF-16 编码，因此对于码点超过 2 字节的字符，需要使用[这个东西](https://pub.dev/packages/characters)。

为啥不用 UTF-8？可能是因为计算单个码点有点复杂，尤其在遍历时，无论是往前还是往后，都需要根据当前 byte 的值来计算前一个或后一个码点；并且不能简单从 byte sequence 的某一处向前或向后遍历（因为当前 byte 大概率并不是码点的开始，因此若要从某一处遍历必须遍历完这一处之前的所有码点）。

既然如此，又为啥不用 UTF-32？完全不需要计算码点。UTF-32 空间浪费严重，绝大多数码点不会超过 16 位，而超过 16 位的码点通过 `surrogate pair` 来解决，这比起遍历 UTF-8 要简单得多，使用 UTF-16 平衡了这两点。但是使用 UTF-16 也有很多问题，比如最典型的 `surrogate pair` 倒序，没验证过 dart 有没有处理 `surrogate pair` 的异常情况，有时间再搞。

Unicode 字符会影响文字排版，比如：`bidi`，会改变文字的排版方向（常见的阿拉伯文，希伯来文）；比如竖排文本（蒙古语等，参考[这里](https://github.com/flutter/flutter/issues/35994))；比如零宽连接符（👩 + 👦 = 👩‍👦）；总之 Unicode 字符集的细节太多，其中很多字符又与渲染和排版耦合（主要原因是本来这些语言字素和书写是耦合的，比如著名的阿拉伯-希伯来文，印地语等），毕竟要统一所有语言是一件很复杂的事情（这让我想起来通天塔的故事），需要各个语言领域的专家做这些这些事情。目前 Flutter 上用的 shaping engine 是 Harfbuzz。

相对来说，中文和英文的排版就简单得多。中文就更简单了，因为都是方块字，每个字素的大小、metrics（ascent，descent，width）都是确定的；而英文还需要处理诸如连字（ligature）、间距微调（kerning）、Font features（手写体，old-styled numbers...）等等相关特性。

参考：

- [Grapheme Cluster Boundaries](https://unicode.org/reports/tr29/#Grapheme_Cluster_Boundaries)
- [Flutter 竖排文本](https://github.com/flutter/flutter/issues/35994)
- [Zero-width joiner](https://en.wikipedia.org/wiki/Zero-width_joiner)
- [UTF-8 转 UTF-32](https://github.com/NanoMichael/MicroTeX/blob/da9ed490c5f21a10914a3b222099b7a3b102478f/lib/utils/utf.cpp#L53)
- [Recommended Emoji ZWJ Sequences](https://unicode.org/emoji/charts/emoji-zwj-sequences.html)
- [Harfbuzz](https://harfbuzz.github.io/index.html)

TODO:

- [ ] 整理字符编码相关知识

### [Functions](https://dart.dev/guides/language/language-tour#functions)

新的东西：

- 命名参数，`{}` 包裹，必须传入的参数用 `required` 关键字
- 位置参数，`[]` 包裹
- 函数作为 type 声明，语法还挺奇怪的（主要原因还是因为 dart 没有类型后置吧），举个 🌰：

```dart
// 可以这么声明，不推荐，会建议使用下一种方式
void dummy(void f(String str)) {
  f('hello world!');
}

void dummy(void Function(String str) f) {
  f('hello world!');
}
```

相比来说 Kotlin 还是好玩点：

```kotlin
fun dummy(f: (String) -> Unit) {
  f("hello world!")
}
```

### [Generics](https://dart.dev/guides/language/language-tour#generics)

dart 泛型不擦除，是协变的，但是它在协变上有缺陷，举个 🌰：

```dart
List<String> strs = [];
// 这里完全木有问题，因为协变，List<String> 是 List<Object> 的子类
// 相当于 List<String> extends List<Object>
List<Object> objs = strs;
// ⚠️ 这里出问题了，因为 objs 的类型是 List<Object>，所以可以往里边添加
// 任何类型的数据，编译没有问题，但实际上 objs 的 runtime type 是
// List<String>，因而运行时会出类型转换的错误
objs.add(1);
```

dart 的逆变需要在 `analysis_options.yaml` 文件中开启编译选项。

```yaml
analyzer:
  enable-experiment:
    - variance
```

另外，`use-site variance` 有 proposal，但是尚未纳入到任何 project（3 年了哇），参考 [Feature: Sound use-site variance](https://github.com/dart-lang/language/issues/753)。

有个东西挺有意思，直接看[原文](https://dart.dev/guides/language/sound-problems#the-covariant-keyword)吧：

> Some (rarely used) coding patterns rely on tightening a type by overriding a parameter’s type with a subtype, which is invalid. In this case, you can use the covariant keyword to tell the analyzer that you are doing this intentionally. This removes the static error and instead checks for an invalid argument type at runtime.

```dart
class Animal {
  void chase(Animal x) { ... }
}

class Mouse extends Animal { ... }

class Cat extends Animal {
  @override
  void chase(covariant Mouse x) { ... }
}
```

作为对比：

```kotlin
// Kotlin 协变
val strs: MutableList<String> = mutableListOf()
// ❌ 这里会编译错误，因为 MutableList 非协变
val objs: MutableList<Any> = strs
// 这么写就没问题了，out 修饰这个 list 只能产出，不能消费，所以 objs
// 不能添加元素（确切的说是没有能接收元素的方法），如果一个泛型容器只
// 产出元素，那么可以直接在声明处用 out 修饰
val objs: MutableList<out Any> = strs

// 如果这么声明，那么以下语句完全没问题，因为 objs 无法添加元素
// List 在 Kotlin 中本来就被声明为协变的
// open class List<out E> ...
val strs: List<String> = mutableListOf()
val objs: List<Any> = strs
```

Kotlin 同时支持声明处型变和使用处型变。对于逆变，一个典型的例子是 `copy`：

```kotlin
// 这里 out 要省略，因为 List 定义为协变的
fun <T> copy(src: List</*out*/ T>, dst: MutableList<T>) {
  src.forEach { dst.add(it) }
}
val ints = arrayListOf(1, 2, 3)
val nums = arrayListOf<Number>()
// 这里会报错，因为我们在声明 copy 时，dst 是型不变的，MutableList<Number>
// 不是 MutableList<Int> 的子类
copy<Int>(src = ints, dst = nums) // 1
// 这里不会报错 😏，因为 Kotlin 的自动推导，src 协变为 List<Number>，
// 所以这里的 T 被推导为 Number
copy(src = ints, dst = nums)
// 为了让 1 处的代码通过，我们需要修改 copy 的函数签名为：
fun <T> copy(src: List</*out*/ T>, dst: MutableList<in T>)
```

Java 跟 Kotlin 基本差不多，唯一区别是使用通配符，且没有声明处型变。

```java
// Java 泛型是型不变的，要实现协变和逆变，需要在使用时使用通配符
List<Integer> ints = new ArrayList<>();
// 通过上界限制只能产出，不能消费
List<? extends Number> a = ints;
// 通过下界限制只能消费，不能产出
List<? super Integer> b = new ArrayList<Number>(); 
```

另外，Java 的数组是协变的：

```java
Integer a[] = {1,2,3};
Number b[] = a;
// 这样是完全合法的，在编译期不会报错
b[0] = 1.0;
```

C++，emmm，它的模板是型不变的（应该说和通常所说的泛型关系不大。。。）

参考：

- [Kotlin 泛型](https://kotlinlang.org/docs/generics.html#variance)

### Generator

Dart 统一了 `Iterable` 和同步 `Generator`，都用 `Iterable` 表示，异步生成器用 `Stream` 来表示。同步生成器函数使用 `sync*` 修饰，异步生成器用 `async*` 修饰。举个 🌰：

- 同步生成器

```dart
Iterable<int> fibonacci() sync* {
  int a = 0, b = 1;
  while (true) {
    yield a;
    final int temp = a;
    a = b;
    b = temp + b;
  }
}
```

- 异步生成器

```dart
Stream<int> produce() async* {
  int i = 0;
  while (true) {
    await Future.delayed(Duration(seconds: 1));
    yield i++;
  }
}
```

### Extension

dart 扩展函数，还是挺有意思，举几个🌰：

```dart
extension IterableExt<T> on Iterable<T> {
  Iterable<R> fmap<R>(Iterable<R> Function(T t) convert) sync* {
    for (final t in this) {
      for (final r in convert(t)) {
        yield r;
      }
    }
  }

  Iterable<B> zip<A, B>(Iterable<A> a, B Function(T x, A y) convert) sync* {
    for (final t in this) {
      yield convert(t, a.first);
      a = a.skip(1);
      if (a.isEmpty) break;
    }
  }
}

extension NullableListExt<T> on List<T>? {
  bool get isNullOrEmpty => this?.isEmpty ?? true;

  bool get isNotNullAndEmpty => !isNullOrEmpty;
}

extension ScopedExt<T> on T {
  Iterable<T> repeat(int n) sync* {
    for (var i = 0; i < n; i++) {
      yield this;
    }
  }

  T also(void Function(T it) block) {
    block(this);
    return this;
  }

  R let<R>(R Function(T it) convert) {
    return convert(this);
  }
}
```

可玩性挺高，可惜的是，dart 不支持 function receiver 语法，相比 Kt 来说还是弱点，虽然有 [proposal](https://github.com/dart-lang/sdk/issues/34362)，但是还是没有实现。

### duck type

[proposal](https://github.com/dart-lang/language/issues/1612) here, NOT IMPLEMENTED.

[Construction of generic types "new T()"](https://github.com/dart-lang/sdk/issues/30074)， 看起来是不准备实现了。

没研究 dart 反射，用反射应该可以实现 duck type。

在 Kotlin 中实现一个简单的 `duck type cast` 😊（总是不自觉跟 Kotlin 对比）:

```kotlin
class DuckTypeProxy(private val delegator: Any) : InvocationHandler {
  override fun invoke(proxy: Any, method: Method, args: Array<out Any>?): Any? {
    val (dispatcher, fn) = if (delegator is Class<*>) {
      null to delegator.getMethod(method.name, *method.parameterTypes)
    } else {
      delegator to delegator.javaClass.getMethod(method.name, *method.parameterTypes)
    }
    return if (args == null) fn.invoke(dispatcher) else fn.invoke(dispatcher, *args)
  }
}

inline fun <reified T> Any.duckCast(): T {
  return Proxy.newProxyInstance(javaClass.classLoader, arrayOf(T::class.java), DuckProxy(this)) as T
}

interface Swimmable {
  fun swim()
}

class Duck {
  fun swim() = println("duck swimming")
}

class Goose {
  fun swim() = println("goose swimming")
}

fun main() {
  val duck = Duck()
  val goose = Goose()
  duck.duckCast<Swimmable>().swim()
  goose.duckCast<Swimmable>().swim()
}
```

这非常有用，尤其是当引入第三方库，或者自动生成的代码，又无法扩展库内的功能时，可以使用这个方法。举个喜闻乐见的 🌰，自动生成的 json 序列化代码，它们都有一个静态的工厂方法 `fromJson` 来构造对象，我们可能需要在 Retrofit 中使用，而无法修改这些代码，那么就可以使用这个方法。

```kotlin
class Duck(private val name: String) {
  companion object {
    @JvmStatic
    fun fromJson(json: JSONObject): Duck {
      return Duck(json.getString("name"))
    }
  }
  // ...
}

class Goose(private val name: String) {
  companion object {
    @JvmStatic
    fun fromJson(json: JSONObject): Goose {
      return Goose(json.getString("name"))
    }
  }
  // ...
}

interface FromJson<T> {
  fun fromJson(json: JSONObject): T
}

// FromJsonAdapter 实现 Retrofit 的 Converter 接口，这里简化
object FromJsonAdapter {
  @JvmStatic
  fun <T> new(clazz: Class<T>, json: JSONObject): T {
    return clazz.duckCast<FromJson<T>>().fromJson(json)
  }
}

fun swim(obj: Any) {
  try {
    obj.duckCast<Swimmable>().swim()
  } catch (e: Exception) {
    println("cannot swim")
  }
}

fun main() {
  val json = JSONObject("{\"name\":\"Donald\"}")
  val duck = FromJsonAdapter.new(Duck::class.java, json)
  swim(duck)

  val json2 = JSONObject("{\"name\":\"Golden\"}")
  val goose = FromJsonAdapter.new(Goose::class.java, json2)
  swim(goose)
}
```

### Pattern matching

暂不支持，有望实现，[proposal](https://github.com/dart-lang/language/issues/546)。

参考一些实现：

- [union](https://github.com/rrousselGit/union)
- [match](https://github.com/mrbech/match)

Roadmap 可以看[这里](https://github.com/dart-lang/language/blob/master/working/0546-patterns/goals-and-constraints.md)。

### [dart 协程](https://dart.dev/codelabs/async-await)

没啥好说的，没有 coroutine scope 的概念，所以取消啥的要麻烦些。

先看个 🌰：

```dart
void main(List<String> arguments) async {
  final d1 = await measureTime(() async {
    final a = await delay();
    final b = await delay();
    print('sum1: ${a + b}');
  });
  print('d1: $d1');
  final d2 = await measureTime(() async {
    final a = delay();
    final b = delay();
    print('sum2: ${await a + await b}');
  });
  print('d2: $d2');
  final d3 = await measureTime(() async {
    final r = Future.wait([delay(), delay()]);
    final sum = (await r).fold<int>(0, (p, e) => p + e);
    print('sum3: $sum');
  });
  print('d3: $d3');
}

Future<int> delay() async {
  return Future.delayed(Duration(seconds: 1)).then((value) => 1);
}

Future<Duration> measureTime(Future<void> Function() f) async {
  final watch = Stopwatch()..start();
  await f();
  return watch.elapsed;
}
```

输出：

```
sum1: 2
d1: 0:00:02.011320
sum2: 2
d2: 0:00:01.001897
sum3: 2
d3: 0:00:01.004450
```

跟其他协程实现类似，当调用 `async` 函数时会立马执行该函数，所以特别注意一点，如果两个协程无依赖时应该要并发执行。以下代码可以证明：

```dart
void main(List<String> arguments) {
  dummy();
  print('done?');
}

void dummy() {
  Future.delayed(Duration(seconds: 1)).then((value) => print('hello'));
}
```

输出：

```
done?
hello
```

诶？为啥 `done` 都输出了，`main` 函数还未结束？🤔 很容易想到，类似的，跟 JS 一样，dart 是单线程模型，使用事件机制：

![event-loop](https://dart.cn/articles/archive/images/both-queues.png)

参考：

- [dart 事件循环与异步机制](https://juejin.cn/post/6949898044628271140)

> Event Looper 为空时, 是**可以**而不是**一定**要退出, 视场景而定.
 
dart 协程是对称协程，也是无栈协程。dart 中 `async`，`await` 是 `Future` 的语法糖，将 `monad` 转换为 `CPS`（maybe？没有细究），`Future` 和 JS 中的 `Promise` 类似，也可以近似的看作是函数式编程中 `monad` 的概念，有机会讲一下函数式编程。

参考：

- [Coroutine in C](https://www.chiark.greenend.org.uk/~sgtatham/coroutines.html)
- [Revisiting Coroutines](http://www.inf.puc-rio.br/~roberto/docs/MCC15-04.pdf)
- [Functors, Applicatives, And Monads In Pictures](https://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)

#### Structured concurrency

似乎 dart 没有结构化并发的概念，关于啥是 Structured concurrency，直接看 Kotlin 的原文吧：

> Coroutines follow a principle of structured concurrency which means that new coroutines can be only launched in a specific CoroutineScope which delimits the lifetime of the coroutine.
> 
> In a real application, you will be launching a lot of coroutines. Structured concurrency ensures that they are not lost and do not leak. An outer scope cannot complete until all its children coroutines complete. Structured concurrency also ensures that any errors in the code are properly reported and are never lost.

大概意思就是，结构化并发减少了出错的机会，能保证在 coroutine scope 生命周期结束后相应的 coroutine 能得到释放；coroutine 出错能得到更好的处理。

参考：

- [Kotlin coroutine](https://kotlinlang.org/docs/coroutines-basics.html)

TODO：

- [ ] 事件循环基于啥（阻塞队列，epoll？）
- [ ] 为什么事件队列为空时，是**可以**而非**一定**要退出？这些场景是哪些场景？
- [ ] 整理一下各语言协程的实现，横向比较下

## Flutter

### widget 生命周期：

![lifecycle](https://miro.medium.com/max/1400/1*XGgoTyEdU_5wh6vqfvAGFg.png)

参考：

- [Flutter APP lifecycle](https://medium.flutterdevs.com/app-lifecycle-in-flutter-c248d894b830)
 
### [Flutter architecture](https://docs.flutter.dev/resources/architectural-overview)

架构层级

![architectural-layers](https://docs.flutter.dev/assets/images/docs/arch-overview/archdiagram.png)

Layout and rendering

![trees](https://docs.flutter.dev/assets/images/docs/arch-overview/trees.png)

Constraints and size

![constraints](https://docs.flutter.dev/assets/images/docs/arch-overview/constraints-sizes.png)

### 状态管理

- scoped model
- redux
- BloC

#### redux

参考：

- [Flutter Redux: Complete tutorial with examples](https://blog.logrocket.com/flutter-redux-complete-tutorial-with-examples/)
