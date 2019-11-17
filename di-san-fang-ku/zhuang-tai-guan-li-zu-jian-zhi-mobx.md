---
description: 任何源自应用状态的东西都应该自动地获得
---

# 状态管理组件之mobx

#### 简介

MobX只是一个状态管理库，可以很容易地将应用程序的被动数据与UI连接起来.

#### 使用

参照mobx官网创建一个demo示例

#### 1.在项目中引入mobx <a id="&#x5728;&#x9879;&#x76EE;&#x4E2D;&#x8BBE;&#x7F6E;jsonserializable"></a>

我们需要一个常规和两个**开发依赖**项。简而言之，**开发依赖项**是不包含在我们的应用程序源代码中的依赖项，它是开发过程中的一些辅助工具、脚本，和node中的开发依赖项相似。

**pubspec.yaml**

```yaml
dependencies:
  flutter:
    sdk: flutter
  mobx: ^0.3.7
  flutter_mobx: ^0.3.2

dev_dependencies:
  flutter_test:
    sdk: flutter
  mobx_codegen: ^0.3.10
  build_runner: ^1.3.1
```

注：请核对包的版本，高版本和低版本创建class时略有不同，请参照官网[mobx中demo的yaml文件](https://github.com/mobxjs/mobx.dart/blob/master/mobx_examples/pubspec.yaml)

#### 2. 创建一个store类

在lib下任意位置新建counter.dart文件，请注意，使用 **with Store**，而不是**implements Store**

在需要被观察的数据增加@observable注解，需要执行操作的方法增加@action注解,

```dart
import 'package:mobx/mobx.dart';

part 'counter.g.dart';

class Counter = _Counter with _$Counter;

abstract class _Counter with Store {
  @observable
  int value = 0;

  @action
  void increment() {
    value++;
  }
}
```

对于dart版本的mobx，是通过生成新的类来实现双向绑定的效果，所以需要在store里面加上生成类的一些定义

```dart
part 'counter.g.dart';

class Counter = CounterBase with _$Counter;
```

执行`flutter packages pub run build_runner build`就会在新建的文件旁生成对应的`counter.g.dart`文件

注：

1. 如果之前有使用过类似的生成库，可能会报文件冲突错误，可以尝试下面的clean命令，在命令窗也会有对应的提示:`flutter packages pub run build_runner watch --delete-conflicting-outputs`
2. 如果需要实时跟踪store的变化从而实时改变新生成的类，需要执行一个命令:

   `flutter packages pub run build_runner watch` ,

#### 3. 在widget中使用

在需要观察数据变化的widget套上一层Observer widget

```dart
import 'package:flutter/material.dart';
import 'package:flutter_mobx/flutter_mobx.dart';
import 'package:mobx_examples/counter/counter.dart';

class CounterExample extends StatefulWidget {
  const CounterExample();

  @override
  CounterExampleState createState() => CounterExampleState();
}

class CounterExampleState extends State<CounterExample> {
  final Counter counter = Counter();

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(
          backgroundColor: Colors.blue,
          title: const Text('MobX Counter'),
        ),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              const Text(
                'You have pushed the button this many times:',
              ),
              Observer(
                  builder: (_) => Text(
                        '${counter.value}',
                        style: const TextStyle(fontSize: 40),
                      )),
            ],
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: counter.increment,
          tooltip: 'Increment',
          child: const Icon(Icons.add),
        ),
      );
}
```

这样一个简单的mobx计数器demo就完成了，功能与原本demo一致，点击右下角的+号，屏幕中央显示的计数对应增加。

