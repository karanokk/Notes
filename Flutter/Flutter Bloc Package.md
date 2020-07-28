# [Flutter Bloc Package](https://medium.com/flutter-community/flutter-bloc-package-295b53e95c5c)

![Image for post](https://miro.medium.com/max/537/1*lP-1aF6Rg8jo459f87l3zg.png)

After having worked with Flutter for a bit, I decided to create a package to help with something that I have used quite frequently: the BLoC pattern.

For those not familiar with the BLoC pattern, it is a design pattern which helps separate the presentation layer from the business logic. You learn more about it [here](https://www.youtube.com/watch?v=fahC3ky_zW0).

While using the BLoC pattern can prove to be challenging due to the setup as well as an understanding of Streams and Reactive Programming, at it’s core a BLoC is pretty simple:

*A BLoC takes a stream of events as input and transforms them into a stream of states as output.*

![Image for post](https://miro.medium.com/max/60/1*_EgLk67LpREOEMCSIhc_4Q.png?q=20)

![Image for post](https://miro.medium.com/max/1589/1*_EgLk67LpREOEMCSIhc_4Q.png)

High-Level BLoC Architecture

We can now use this powerful design pattern with the help of the [bloc package](https://pub.dartlang.org/packages/bloc).

---

This package abstracts reactive aspects of the pattern allowing developers to focus on converting events into states.

Let’s start by defining those terms…

# Glossary

> # 术语

**Events** are the input to a Bloc. They are commonly UI events such as button presses. `Events` are `dispatched` and converted to `States`.

> **Events** 是 Bloc 的 input。比如按下按钮，通常为 UI 事件， `Events` 被 `发送` 并且转换成 `States`.

**States** are the output of a Bloc. Presentation components can listen to the stream of states and redraw portions of themselves based on the given state (see `BlocBuilder` for more details).

> **States** 是 Bloc 的 output。 Presentation 组件能够监听 states 流，并且基于给定的 state 部分重绘自身 有关详细信息，请参阅 `BlocBuilder`)。

**Transitions** occur when an `Event` is `dispatched` after `mapEventToState` has been called but before the bloc’s state has been updated. A `Transition` consists of the `currentState`, the `event` which was dispatched, and the `nextState`.

> **Transitions** 发生在事件发送时，在`mapEventToState`方法调用后，bloc 的 state 更新之前。`Transition` 由 `currentState`, 发送的 `event` 以及 `nextState`组成。

Now that we understand events and states we can take a look at the Bloc API.

> 现在我们了解了 events 和 states，让我们来看看 Bloc API。

# Bloc API

**mapEventToState** is a method that must be implemented when a class extends `Bloc`. The function takes the incoming event as an argument. `mapEventToState` is called whenever an event is `dispatched` by the presentation layer. `mapEventToState` must convert that event into a new state and return the new state in the form of a `Stream` which is consumed by the presentation layer.

> **mapEventToState** 是类继承 Bloc 时必须实现的方法。该函数将传入的事件作为参数。每当 presentation layer发送事件时，都会调用 `mapEventToState`。`mapEventToState` 必须将该事件转换为新状态，并以 presentation layer 使用的流的形式返回新状态。

**dispatch** is a method that takes an `event` and triggers `mapEventToState`. `dispatch` may be called from the presentation layer or from within the Bloc (see examples) and notifies the Bloc of a new `event`.

> **dispatch** 是一个接受事件并触发 `mapEventToState `的方法。`dispatch` 可能由 presentation layer 或 Bloc 内部(参见示例)调用，并通知 Bloc 一个新事件。

**initialState** is the state before any events have been processed (before `mapEventToState` has ever been called). `initialState` is an optional getter. If unimplemented, initialState will be `null`.

> **initialState** 是任何事件被处理前的状态(在 `mapEventToState` 被调用之前)。initialState是一个可选的 getter 。如果未实现，initialState将为 null。

**transform** is a method that can be overridden to transform the `Stream<Event>` before `mapEventToState` is called. This allows for operations like `distinct()` and `debounce()` to be used.

> 在调用 mapEventToState之前，**transform** 是一个可以被重写以转换`Stream<Event>` 的方法。这就允许你使用一些像 `distinct()` 和 `debounce()` 这样的操作。

**onTransition** is a method that can be overridden to handle whenever a `Transition` occurs. A `Transition` occurs when a new `Event` is dispatched and `mapEventToState` is called. `onTransition` is called before a bloc’s state has been updated. **It is a great place to add bloc-specific logging/analytics**.

> **onTransition** 可以被重写以处理任何发生 `Transition` 的方法。当发送新 event 并调用`mapEventToState` 时，就会发生 `Transition`。`onTransition `在更新 bloc 的 state 之前调用。

Let’s create a counter bloc!

```dart
enum CounterEvent { increment, decrement }

class CounterBloc extends Bloc<CounterEvent, int> {
  @override
  int get initialState => 0;

  @override
  Stream<int> mapEventToState(CounterEvent event) async* {
    switch (event) {
      case CounterEvent.decrement:
        yield currentState - 1;
        break;
      case CounterEvent.increment:
        yield currentState + 1;
        break;
    }
  }
}
```

Counter Bloc Implementation

In order to create a Bloc, all we **need** to do is:

- Define our events and states
- Extend Bloc
- Override `initialState` and `mapEventToState`.

In this case our events are `CounterEvents` and our states are `integers`.

Our `CounterBloc` converts `CounterEvents` to integers.

We can notify out `CounterBloc` of events by calling `dispatch` like so:

```dart
void main() {
  final counterBloc = CounterBloc();

  counterBloc.dispatch(CounterEvent.increment);
  counterBloc.dispatch(CounterEvent.decrement);
}
```

In order to observe state changes (`Transitions`) we can override `onTransition`.

```dart
enum CounterEvent { increment, decrement }

class CounterBloc extends Bloc<CounterEvent, int> {
  @override
  int get initialState => 0;

  @override
  void onTransition(Transition<CounterEvent, int> transition) {
    print(transition);
  }

  @override
  Stream<int> mapEventToState(CounterEvent event) async* {
    switch (event) {
      case CounterEvent.decrement:
        yield currentState - 1;
        break;
      case CounterEvent.increment:
        yield currentState + 1;
        break;
    }
  }
}
```

Now every time we dispatch a `CounterEvent` our Bloc will respond with a new `integer` state and we will see a transition logged to the console.

Now let’s build a UI using Flutter and hook up the presentation layer to our `CounterBloc` using the [flutter_bloc](https://pub.dartlang.org/packages/flutter_bloc) package.

---

The [flutter_bloc](https://pub.dartlang.org/packages/flutter_bloc) package provides two widgets to make interacting with Blocs easy:

> [flutter_bloc](https://pub.dartlang.org/packages/flutter_bloc) 提供了2个 widgets 让你与 Blocs 交互更加轻松:

# BlocBuilder

`BlocBuilder` is a Flutter widget which requires a `Bloc` and a `builder` function. `BlocBuilder` handles building a widget in response to new states. `BlocBuilder` is very similar to `StreamBuilder` but has a more simple API to reduce the amount of boilerplate code needed.

> `BlocBuilder` 是一个 Flutter widget，它需要一个 `Bloc` 和 `builder` 方法。`BlocBuilder` 处理构建 widget 以响应新状态。`BlocBuilder` 与 `StreamBuilder` 非常类似，但是它有一个更简单的API来减少所需的样板代码数量。

# BlocProvider

`BlocProvider` is a Flutter widget which provides a bloc to its children via `BlocProvider.of(context)`. It is used as a dependency injection (DI) widget so that a single instance of a bloc can be provided to multiple widgets within a subtree.

> BlocProvider是一个 Flutter widget，它通过 `BlocProvider.of(context)` 为它的子组件提供一个 bloc。它被用作依赖注入(DI) widget，这样可以将一个单个实例的 bloc 提供给一个子树中的多个 widget。

Now let’s build our Counter App!

```dart
class App extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => _AppState();
}

class _AppState extends State<App> {
  final CounterBloc _counterBloc = CounterBloc();

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      home: BlocProvider<CounterBloc>(
        bloc: _counterBloc,
        child: CounterPage(),
      ),
    );
  }

  @override
  void dispose() {
    _counterBloc.dispose();
    super.dispose();
  }
}

class CounterPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final CounterBloc _counterBloc = BlocProvider.of<CounterBloc>(context);

    return Scaffold(
      appBar: AppBar(title: Text('Counter')),
      body: BlocBuilder<CounterEvent, int>(
        bloc: _counterBloc,
        builder: (BuildContext context, int count) {
          return Center(
            child: Text(
              '$count',
              style: TextStyle(fontSize: 24.0),
            ),
          );
        },
      ),
      floatingActionButton: Column(
        crossAxisAlignment: CrossAxisAlignment.end,
        mainAxisAlignment: MainAxisAlignment.end,
        children: <Widget>[
          Padding(
            padding: EdgeInsets.symmetric(vertical: 5.0),
            child: FloatingActionButton(
              child: Icon(Icons.add),
              onPressed: () {
                _counterBloc.dispatch(CounterEvent.increment);
              },
            ),
          ),
          Padding(
            padding: EdgeInsets.symmetric(vertical: 5.0),
            child: FloatingActionButton(
              child: Icon(Icons.remove),
              onPressed: () {
                _counterBloc.dispatch(CounterEvent.decrement);
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

Our `App` widget is a `StatefulWidget` which is responsible for creating and disposing a `CounterBloc`. It makes the `CounterBloc` available to the `CounterPage` widget using the `BlocProvider` widget we mentioned above.

Our `CounterPage` widget is a `StatelessWidget` which uses `BlocBuilder` to rebuild the UI in response to state changes from our `CounterBloc`.

At this point we have successfully separated our presentational layer from our business logic layer. Notice that the `CounterPage` widget knows nothing about what happens when a user taps the buttons. The widget simply tells the `CounterBloc` that the user has pressed either the increment or decrement button.

That’s all there is to it!

For more examples and detailed documentation check out the [official bloc documentation](https://felangel.github.io/bloc).

If you like the bloc library, you can support me by ⭐️the [repository](https://github.com/felangel/bloc), or 👏 for this story.
