---
template: post
title: 'Flutter 单元测试&覆盖率报告'
slug: Flutter
draft: false
date: 2020-02-18T08:00:20.019Z
description: Dart 本身提供了很好的体系帮助我们构建和测试单元用例. Flutter 又提供了flutter_test 包来降低针对 Widget 的测试门槛. 所以总体来说给 Flutter 应用提供单元测试还是一件相对‘愉悦’的事情.
category: Flutter
tags:
  - flutter
  - unit test
---

无论是团队 KPI 考核, 还是出于对自己代码的质量保障, 我们在构建应用的时候都会有单元测试的诉求. 特别是当前环境下很多公司在移动端人员充足, 业务稳定的时候, 对质量、稳定性要求越来越高了. 所以在用 Flutter 构建的时候自然而然也会有这方面的诉求.

Dart 本身提供了很好的体系帮助我们构建和测试单元用例. Flutter 又提供了`flutter_test` 包来降低针对 Widget 的测试门槛. 所以总体来说给 Flutter 应用提供单元测试还是一件相对‘愉悦’的事情.

## Flutter 单元测试

- `pubspec.yaml`中引入测试包, 
  如果你的库没有依赖flutter 只用引入 `test` 库就够了.

    ```yaml
    dev_dependencies:
      test: ^1.3.0
      flutter_test:
        sdk: flutter
    ```
- 编写测试用例
  在 test 目录下创建测试文件.
  
  ```
  |____test
  | |____widget_test.dart
  ...
  |____pubspec.yaml
  ...
  ```
  **语法**
  1. `group()`函数可用于对测试进行分组
  2. `test()`函数定义测试
  3. `expect()`函数来强制执行断言
  
  ```dart
    group("some_Group_Name", () { 
       test("test_name_1", () { 
          expect(actual, equals(exptected)); 
       });  
       test("test_name_2", () { 
          expect(actual, equals(expected)); 
       }); 
     })

  ```
  
- 执行测试命令, 输出结果
   
    ```shell
    flutter test --converage
    ```
  添加 `--converage` 参数, 该命令会执行 `test` 目录下的用例, 并生成 `coverage/lcov.info`.
  如果只是单独修改了某个文件或几个文件可以利用 `--merge-coverage` 参数, 来对覆盖率信息作增量更新.
  
  #### Dart 覆盖率信息
  针对没有依赖 Flutter 的库, 可以使用 [`coverage`](https://pub.flutter-io.cn/packages/coverage) 来生成覆盖率信息
  
  ```shell
  pub global activate coverage
  # 执行测试用例
  dart --pause-isolates-on-exit --enable-vm-service=NNNN test/*
  # 收集测试结果信息
  collect_coverage -o coverage/coverage.json --uri=${specifies the Observatory URI emitted by the VM} 
  # 格式化成lcov信息
  format_coverage --packages=${the path to the package whose coverage is being collected} -i coverage.json
  
  ```
  
- 可视化覆盖率信息
上面我们已经得到了 lcov 信息, 但是还并不能很直观的看出具体没有覆盖到的代码.
好在现在IDE都提供了非常完善的工具. 比如 VS 可以下载 https://marketplace.visualstudio.com/items?itemName=markis.code-coverage 插件. 当配置覆盖率文件后, VS 会在未覆盖的代码行显示高亮线.
    同时, 如果团结使用持续化集成的话, 我们希望在集成过程中能提供更直观的统计信息, 方便团队跟进和提高覆盖率. 这时可以使用 `lcov` 来输出 html 格式文件. 

    ```shell
    # brew install lcov 谨慎安装 cpu 占用非常高.
    genhtml coverage/lcov.info -o coverage
    ```
其实对于开源项目还有一个更简单的方式就是使用 `coveralls` 查看覆盖率数据. Flutter 项目自己就是这么干的(https://coveralls.io/github/flutter/flutter?branch=master).

## 持续集成

我的测试项目是托管在 Github 上的, 所以我这里就直接使用 `Travis CI` + `Coveralls` 做了集成.
网上也有一些集成方案, 但先对比较老, 像 `language` 我本来是使用 `dart` 的, 但`Travis CI` 会默认执行 `pub get` 就会报错, 这是我测试项目的配置.

```yaml
language: generic
# dart:
#   - stable
os:
  - osx
sudo: false
before_script:
- cd ..
- gem install coveralls-lcov
- git clone https://github.com/flutter/flutter.git -b stable
- export PATH=`pwd`/flutter/bin:`pwd`/flutter/bin/cache/dart-sdk/bin:$PATH
# - flutter upgrade
- flutter doctor

script:
- cd $TRAVIS_BUILD_DIR
- flutter --version
# - flutter upgrade
# - flutter packages get
- flutter test --coverage
- coveralls-lcov coverage/lcov.info

```

这样你在每次提交后 `Travis CI` 就会自动生成覆盖率报告.

![-w492](https://mmbiz.qpic.cn/sz_mmbiz_png/eB0h4dLLiakttibG953uHIibeXB5tMAOzfN8pAMCibPQJ72tuSphK10t7UJYpyVqfYhgEazsWmiacgMpzsNCEWAia97Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

点击 coveralls 项的 Details 还可以查看具体的覆盖率信息.

![-w863](https://mmbiz.qpic.cn/sz_mmbiz_png/eB0h4dLLiakttibG953uHIibeXB5tMAOzfNNu4NAJY6Yk32aKY1HBP8kZiaIWfSqiabSJOAavLrbwqFo274UicUkEEzg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)


  