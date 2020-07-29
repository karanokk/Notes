# Stream

> **“dart:async/stream.dart”**

> [异步编程：使用 stream](https://dart.cn/tutorials/language/streams)

> [在 Dart 里使用 Stream](https://dart.cn/articles/libraries/creating-streams)

## Stream 类型

1. "**Single-subscription**" streams
   
   - 整个生命周期只允许一个订阅者。
   
   - 不允许订阅两次，即使第一次订阅被取消了。
   
   - 直到有订阅者订阅时才会**产生**事件，即使源依然能够提供更多事件，一旦取消订阅就会停止**发送**事件。
   
   - 通常用于流式传输较大的连续数据，例如文件I /O。

2. "**broadcast**" streams
   
   - 允许任意数量的订阅者，无论是否被订阅都会发送事件。
   
   - 订阅者接收的事件取决于订阅的时间。
   
   - 当触发了 `done` 事件，订阅者在接受到这个事件前会被取消订阅， `done`此事件发出后，这个 `stream` 就没有订阅者了。此时依然可以订阅，不过只会立即接收到一个新的`done` 事件。
   
   - 使用 `[asBroadcastStream]`转换 `Single-subscription`。

`isBroadcast` 默认为 `false`，broadcast stream 继承 Stream 需要重写成 `true`。

## StreamSubscription

- 可用于替换 `Stream.listen`的回调。

    ```dart
    void onData(void handleData(T data)?);
    void onError(Function? handleError);
    void onDone(void handleDone()?);
    ```

- `void pause([Future<void>? resumeSignal])`
  
  - 对于 `non-broadcast stream`，会通知源暂停生成事件。
  
  - 对于 `broadcast stream`，源发出的事件会被缓存直到订阅恢复，如果要避免事件被缓存，最好是取消订阅，然后重新订阅。
  
  - `resumeSignal` 完成后会取消暂停，如果 `resumeSignal` 产生了错误，则不会取消暂停。
  
  - 与 `void resume()` 配合使用


