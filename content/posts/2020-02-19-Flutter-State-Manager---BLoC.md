---
template: post
title: 'Flutter 状态管理 --- BLoC'
slug: Flutter-Bloc
draft: false
date: 2020-02-18T08:08:20.019Z
description: BLoC 模式是由 Google 的工程师 Paolo Soares 和 Cong Hui 在DartConf 2018上首次提出([视频地址](https://www.youtube.com/watch?v=PLHln7wHgPE))的状态管理模式. 主要方便我们管理状态并集中访问数据.
category: Flutter
tags:
  - flutter
  - bloc
---

## 什么是BLoC
**BLoC 模式**是由 Google 的工程师 *Paolo Soares* 和 *Cong Hui* 在DartConf 2018上首次提出([视频地址](https://www.youtube.com/watch?v=PLHln7wHgPE))的状态管理模式. 主要方便我们管理状态并集中访问数据.
其实在我看来这本身并不是什么新颖的模式, 跟我们之前遇到的分层模式 `MVP`、`MVVM`一样, `BLoC`只是将业务逻辑单独抽离出来对这层职责和边界做了明确规范而已. 甚至于个人学习的时候都直接把 `BLoC` 类比 `MVVM` 中的 `ViewModel`. 
`BLoC` 模式将应用划分为三层(当然如果你愿意可以再对其中的层级做细分):
- 数据层(Data)
 - Data Provider
 - Repository 
 
- 业务逻辑层(Business Logic)

- 表示层(Presentation)

![BLoC](https://i.niupic.com/images/2020/02/18/6zWM.png)

**Data Layer**负责从一个或多个来源检索/处理数据.该层是应用程序的最低级别，主要是做一些与数据库、网络请求和其他异步数据源交互的工作。一般这还可以细分出 `Data Provider` 和 `Repository`, 前者负责提供一些原始数据(比如对数据库的CURD或者获取网络数据等), 后者负责对原始数据进行包装处理. 
**Bloc (Business Logic) Layer** 负责对表示层的事件做出响应, 主要行为是接受事件并根据需要从数据层获取数据, 最后给出反馈的状态.在我看来它就是一个状态机.
**Presentation Layer**的责任是找出如何基于一个或多个BLoC返回的state来呈现自己。此外，它还应该具有处理用户输入和应用程序生命周期事件的能力。

为了更好地使用该模式, 我们还需要了解一些关键的概念.
### 核心概念
**Event** 是 `BLoC` 的输入. 通常是为了响应用户交互, 比如用户输入、点击等行为事件, 当然程序生命周期事件也包含其中, 比如页面的页面加载.
针对不同业务的复杂度一般有 `enum` 或 `class` 两种定义事件的方式, 前者用于简单的不需要提供数据的事件传递, 后者可以提供必要的数据传递给 `BLoC`.

```dart
/*枚举方式*/
enum ArticleStarEvent { star, unStar}
/*class 方式*/
abstract class ArticlesEvent {}
class ArticlesEventLoadData extends ArticlesEvent {
  final bool isFirstPage;
  ArticlesEventLoadData({this.isFirstPage = true});
}
``` 

**State** 是`BLoC`的输出，表示应用程序状态的一部分. `BLoC`可以通知UI组件状态，并根据当前状态重新绘制或部分绘制.

**Transition** 是指从一种状态到另一种状态的变化. 它由当前状态、事件和下一个状态组成. 
比如star一篇文章由以下三部分组成.
```json
{
  "currentState": ArticleState.normal,
  "event": "ArticleEvent.star",
  "nextState": ArticleState.stared
}
```

**Stream** 是一种异步数据的序列.
`BLoC` 是在 `RxDart` 之上构建的，但是它抽象了具体的实现细节。所以为了更好的使用`Bloc`，对`Streams`及其工作原理要有一定的了解.
如果你之前使用过其他Reactive的框架的话相信对这一概念并不陌生. 如果你不熟悉,就想一想有水流经的管道, 管道是 `Stream`, 水是异步数据.
```dart
Stream<int> countStream(int max) async* {
    for (int i = 0; i < max; i++) {
        yield i;
    }
}
```

**Bloc** 区别于上面的模式这里指的是具体的业务逻辑组件，它将传入事件流转换为一个输出状态流。
需要注意的是 `flutter` 中 `Bloc` 的默认实现会忽略重复状态. 如果一个Bloc中map后的状态与之前的状态一样, 就不会向 `Stream` 中发送状态.
默认情况下，事件将始终按照添加的顺序进行处理，并对任何新添加的事件进行查询。一旦mapEventTo State完成执行，事件就被认为是处理完成。

**BlocDelegate** flutter 中提供的统一监控 `Transitions` 的类.
```dart
void main() {
    BlocSupervisor.delegate = SimpleBlocDelegate();
}
class SimpleBlocDelegate extends BlocDelegate {
  @override
  void onEvent(Bloc bloc, Object event) {
    super.onEvent(bloc, event);
    debugPrint("$event");
  }

  @override
  void onTransition(Bloc bloc, Transition transition) {
    super.onTransition(bloc, transition);
    debugPrint("${transition.toString()}");
  }

  @override
  void onError(Bloc bloc, Object error, StackTrace stacktrace) {
    super.onError(bloc, error, stacktrace);
    debugPrint('$error, $stacktrace');
  }
}
```

### 为什么用BLoC
就像上面提到的做为分层架构的通用优势. BLoC模式将业务层独立出来, 使得我们表示层不再耦合具体的业务逻辑, 从而使得业务逻辑的复用最大化、也方便迁移, 同时BLoC层不在依赖于表示层也更易于测试. 此外Reactive的编程模式也有利于快速开发, 颗粒度更细的状态观察模式也能提供更好的性能.

> 这里我并没有提及Widget组件的复用优势, 因为实际应用下来Widget都是依赖与具体Bloc的, 这种强耦合没有太明显的复用/迁移的优势. 我们知道flutter还有跨端的能力, 毕竟除了客户端还有web, 当然设计者可能也是考虑到这种跨端的业务逻辑互用才刻意设计成Bloc层的隔离.毕竟各端的特性不一样, 表示层的样式也会有区别, 但业务逻辑都是一样的.


## 使用BLoC
逼逼了这么多, 还是上手Coding下咯. 下面我来用BLoC模式实现一个简单demo. 主要功能是获取文章列表, 以及一个star的功能.

### Data Layer
首先要做这个功能我们需要获取文章列表数据. 我们通过`ArticlesProvider`异步获取文章列表, 并对接口结果做了简单处理.
```dart
class ArticlesRepository {
  Future<ArticlesResult> fetchArticles(int page) async {
    // 获取文章列表
    final response = await ArticlesProvider.articles(page);
    
    if (!response['success']) {
      throw Exception("error to fetch article");
    }
    // 原始数据处理
    final data = response['data']['articles'] as List;
    final hasNext = response['data']['hasNext'] as bool;
    final articles = data.map((raw) {
      final article = ArticleModel(
          id: "${raw['id']}", title: raw['title'], desc: raw['desc']);
      return article;
    }).toList();

    return ArticlesResult(hasNext: hasNext, articles: articles);
  }
}
```

### BLoC Layer
- 首先我们需要一个`Event`来获取文章列表.

    ```dart
    class ArticlesEventLoadData extends ArticlesEvent {
      //是第一页还是下一页
      final bool isFirstPage;
      ArticlesEventLoadData({this.isFirstPage = true});
    }
    ```
- 然后对于该业务定义了四种状态. 分别是`Uninitialized(未初始化)`、`Loading(加载中)`、`FetchError`和`FetchSuccess`.
  ![state](https://i.niupic.com/images/2020/02/18/6zX7.png)

    ```dart
    abstract class ArticlesState {}
    class ArticlesUninitialized extends ArticlesState {}
    class ArticlesLoading extends ArticlesState {}
    class ArticlesFetchError extends ArticlesState {}
    class ArticlesFetchSuccess extends ArticlesState {
      final bool hasNext;
      ArticlesFetchCompletion({this.hasNext = true});
    }
    ```
- 继承Bloc,重写 initialState 和 mapEventToState.
```dart
    class ArticlesBloc extends Bloc<ArticlesEvent, ArticlesState> {
      var _articles = <ArticleModel>[];
      var _currPage = 0;
      
      get articles => UnmodifiableListView<ArticleModel>(_articles);
    
      @override
      ArticlesState get initialState => ArticlesUninitialized();
    
    
      @override
      Stream<ArticlesState> mapEventToState(ArticlesEvent event) async* {
        debugPrint("event change");
        if (event is ArticlesEventLoadData) {
          // 首页重置页码并清除数据
          if (event.isFirstPage) {
            _currPage = 0;
          }
          if (_currPage == 0) {
            _articles.clear();
          }
          // 获取文章列表
          final articlesRepo = ArticlesRepository();
          yield ArticlesLoading();
          try {
            final articleResult = await articlesRepo.fetchArticles(_currPage);
            _articles.addAll(articleResult.articles ?? []);
            yield ArticlesFetchSuccess(hasNext: true);
            _currPage++;
          } catch (error) {
            debugPrint("fetch error: $error");
            yield ArticlesFetchError();
          }
        }
      }
    }
```

### Presentation Layer
有了Bloc, 我们需要UI界面来展示给用户并响应用户对行为. 
这里我们用了`flutter_bloc`框架中的两个`widget`来连接业务逻辑和视图.
- BlocProvider
    它通过 BlocProvider.of(context)为其子女提供了一个bloc。 它用作依赖注入（DI）小部件, 这样子树中的多个小部件就可以使用一个bloc实例了。
- BlocBuilder
    BlocBuilder处理构建窗口小部件以响应新state。 事实上BlocBuilder是对StreamBuilder的封装，但它有一个更简单的API来减少所需的样板代码量。

    ```dart
    class ArticlesList extends StatelessWidget {
      @override
      Widget build(BuildContext context) {
        BlocProvider.of<ArticlesBloc>(context).add(ArticlesEventLoadData());
        return BlocBuilder<ArticlesBloc, ArticlesState>(
          builder: _blocBuilder,
        );
      }
    
      Widget _blocBuilder(BuildContext context, ArticlesState state) {
        final articles = BlocProvider.of<ArticlesBloc>(context).articles;
        final scrollController = ScrollController();
        scrollController.addListener(() {
          if (scrollController.position.pixels ==
              scrollController.position.maxScrollExtent) {
            // 加载下一页
            BlocProvider.of<ArticlesBloc>(context)
                .add(ArticlesEventLoadData(isFirstPage: false));
          }
        });
    
        return RefreshIndicator(
          displacement: 40,
          onRefresh: () async {
            //下拉加载
            BlocProvider.of<ArticlesBloc>(context).add(ArticlesEventLoadData());
          },
          child: ListView.builder(
          controller: scrollController,
            itemBuilder: _buildItem,
            itemCount: articles.length,
          ),
        );
      }
    
      Widget _buildItem(BuildContext context, int index) {
        final articles = BlocProvider.of<ArticlesBloc>(context).articles;
        final article = articles[index];
        return ArticleWidget(
          article: article,
        );
      }
    }
```

### 如何测试
上面也提到了, Bloc 将业务逻辑独立出来更易于我们测试. `bloc_test` 库也提供了方便测试的封装. 比如测试刚才的获取逻辑可以在 `test` 目录下编写测试用例如下, 执行 `flutter test`就行了.
```dart
void main() {
  group("ArticlesBloc", () {
    ArticlesBloc articlesBloc;
    setUp(() {
      articlesBloc = ArticlesBloc();
    });

    blocTest(
      "test fetch articles",
      build: () => articlesBloc,
      act: (bloc) {
        return bloc.add(ArticlesEventLoadData());
      },
      expect: [ArticlesUninitialized(), ArticlesLoading(), ArticlesFetchCompletion()],
    );

    tearDown(() {
      articlesBloc.close();
    });

  });
}
```

## 其他
### 获取Bloc的时机
由于 `BlocProvider.of<T>` 是通过 `context.getElementForInheritedWidgetOfExactType` 获取上下文中的 Bloc 对象. 所以要特别注意 `context` 的正确性, 必须确认你使用的 `context` 是 `BlocProvider<T>` 子节点Widget 所拥有的 BuildContext.
具体的可以参考[Flutter | 深入理解BuildContext](https://juejin.im/post/5c665cb651882562914ec153).

### 副作用
`BlocBuilder<T,S>` 会根据 Bloc 返回的 `State` 变化来决定是否多次执行 builder 函数.所以要特别注意不要在它的 builder 函数中添加 `Event`. 
```dart
BlocBuilder<ArticlesBloc, ArticlesState>(
      builder: (context, state) {
        // 导致state变化死循环 
        BlocProvider.of<ArticlesBloc>(context).add(ArticlesEventLoadData());
        return ...
      },
    );
```
一个好的建议是将这种初始化数据的行为交给 `Bloc` 的initialState 或者 `BlocProvider`所包含的顶层Widget.
### 问题
在写这个demo的时候我试图通过`BlocBuilder`的 condition 来针对不同 `State` 做界面的刷新, 从而节省开销提升性能. 虽然_listViewBuilder都执行了并获得了正确的数据打印, 但二次刷新的时候`ListView`中的item并不会刷新. 而我试着将ListView替换成其他 Widget 比如Text后又是好的, 这点使我困惑.如果有知道原因的朋友还请不吝告知.
```dart
BlocBuilder<ArticlesBloc, ArticlesState>(
        // ⚠️：不能添加， 否则item不会刷新？？？
//        condition: (preState, state) {
//          debugPrint("condition:::::$state");
//          return !(state is ArticlesLoading);
//        },
        builder: _listViewBuilder,
      )
```
**本篇文章所有代码**
- [flutter_bloc_sample](https://github.com/lyn-euler/flutter_bloc_sample)

**参考**

- [Bloc](https://bloclibrary.dev/#/gettingstarted)
- [Reactive Programming - Streams - BLoC - Practical Use Cases](https://www.didierboelens.com/2018/12/reactive-programming---streams---bloc---practical-use-cases/#4-form-validation)