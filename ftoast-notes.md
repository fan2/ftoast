
ftoast 是 [Fliggy-Mobile](https://github.com/Fliggy-Mobile) 系列组件，需传入 BuildContext，支持Basic,SubMsg,Image,Custom等样式。  

## 代码结构分析

只有一个实现代码文件：`ftoast.dart`。

## 功能实现分析

### ToastData

**_ToastData** 记录了 BuildContext、OverlayEntry 以及显示状态 showed，提供了 dispose 移除 overlayEntry 关闭 toast。

```Dart
class _ToastData {
  OverlayEntry entry;
  int duration;
  BuildContext context;
  bool showed = false;

  dispose() {
    if (showed) entry?.remove();
    entry = null;
    context = null;
  }
}
```

### FToast

`_entryQueue` 为 _ToastData 队列，`_current` 节点记录当前正在显示的 toast，一般为队头元素。

```Dart
class FToast {
  static Queue<_ToastData> _entryQueue;
  static _ToastData _current;

  static toast(
    final BuildContext context, {
    final Widget toast,
    final String msg = "",

  }) {

    Widget buildToast() {
      // ...
    }

    if (_entryQueue == null) _entryQueue = Queue();
    OverlayEntry entry = OverlayEntry(builder: (context) {
      return IgnorePointer(
        child: Material(
          color: Colors.transparent,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              toast ?? buildToast(),
            ],
          ),
        ),
      );
    });
    _ToastData toastData = _ToastData()
      ..context = context
      ..entry = entry
      ..duration = duration;
    _entryQueue.addLast(toastData);
    _show();
  }

  static _show() {

  }

}
```

1. `toast` 内部的函数 *buildToast* 创建返回 Widget；  
2. 构建 `OverlayEntry`：IgnorePointer-Material-Column，children 为传入的 toast 或 buildToast 生成；  
3. 传入 OverlayEntry 和 BuildContext 构建 `_ToastData` 并加入 _entryQueue 队尾；  
4. 调用 `_show` 函数，从 _entryQueue 弹出队头元素：

    - 如果 first.context 非空且 findRenderObject attached，则将 overlayEntry 插入 overlayState 显示，并起定时器 dispose、迭代下一个 _show；  
    - 否则，调用 _ToastData.dispose 函数关闭 toast。  

## 其他参考

[熟悉味道，FToast](https://www.jianshu.com/p/816a14e82050)  
[由你做主，FLoading](https://segmentfault.com/a/1190000023478752)  

> 队列管理方面可参考 [flutter_oktoast](https://github.com/fan2/flutter_oktoast)。
