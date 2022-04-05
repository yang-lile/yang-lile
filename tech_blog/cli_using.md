# Dart 命令行应用开发

## 前提

懒，在多年手写命令行黑窗之后，决定这次偷懒了，使用 Dart 库里面的现存的 DCli 库来开发。

## Isolate 实现加载

为了实现并行加载数据和输出加载进度条，需要单开一个 `Isolate` 来处理。

定义一个只有一个参数的函数：
```Dart
static void _isolateLoading(List<String> charPics) {
  int count = 0;

  var terminal = term.Terminal();
  while (true) {
    terminal.overwriteLine(charPics[count]);
    count++;
    count %= charPics.length;
    term.sleep(100, interval: term.Interval.milliseconds);
  }
}
```

参数通过后面的 `arg` 来传入开启 `Isolate`：
```Dart
Isolate.spawn(_isolateLoading, ['\\', '-', '/', '|']);
```

最后定义一个函数来杀死 `Isolate`，也可以挂起暂停，如果之后还有耗时操作，还能继续使用。

```Dart
void finishLoading() {
  var terminal = term.Terminal();
  terminal.clearLine();
  isolate.kill();
}
```

## 目录使用

```Dart
void showHomeMenu() {
  // 显示目录本身
  const List<HomeMenuItem> homeMenuItems = [
    HomeMenuItem('展示所有事件', HomeMenuItemEnum.show),
    HomeMenuItem('添加事件', HomeMenuItemEnum.add),
    HomeMenuItem('处理事件', HomeMenuItemEnum.process),
    HomeMenuItem('退出', HomeMenuItemEnum.quit),
  ];
  var selectValue = menu<HomeMenuItem>(
    prompt: '【首页】选择您的操作',
    options: homeMenuItems,
  );

  // 回执选项
  print(green('You chose ${selectValue.prompt}'));

  // 使用 switch 根据选项操作
  switch (selectValue.homeMenuItemEnum) {
    case HomeMenuItemEnum.show:
      ShowMessage.showThings();
      break;
    case HomeMenuItemEnum.add:
      showAddThingPage();
      break;
    case HomeMenuItemEnum.process:
      showProcessPage();
      break;
    case HomeMenuItemEnum.quit:
      GetIt.I.get<ThingRepository>().close();
      exit(0);
  }
}
```

非常清晰的三步走，DCli 解决了大部分目录选项问题。




