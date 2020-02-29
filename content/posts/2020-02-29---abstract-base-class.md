---
title: Pythonにおける抽象クラスのメリット
date: "2020-02-29T17:00:00+09:00"
template: "post"
draft: false
slag: "abstract-base-class"
category: "Python"
tags:
    - "Python"
    - "object oriented programming"
    - "abstract base class"
    - "Tech"
    - "Interface"
description: "Python開発でオブジェクト指向によるリファクタリングを行ってた際に出会った抽象クラスのメリットとは？"
---

　私は機械学習系に興味があったのでPythonをよく使っておりました。そのおかけで現在はPythonでの開発を主に行ってます。ここ最近ではPythonで書いたコードが大規模になってきたので、オブジェクト指向のリファクタリングを行っております。そんな時にPythonのabcモジュールという抽象クラスを扱うものがあるのですが、こいつって何のメリットがあるの？っと思ったので記事にしました。

## 「サブクラスの満たすべき要件を規定できる」
　こちらの記事に私の求めていた答えが全部載ってました！！
[抽象クラスを使うメリット](https://qiita.com/bluepost59/items/eef6f48fdd322b0b9791)(Qiita記事)  
要は「サブクラスの満たすべき要件を定められること」であり、抽象クラスの持つメンバは必ずサブクラスでオーバーライドしなければいけません。以下のようにして抽象クラスを作った場合、
```
from abc import ABCMeta
from abc import abstractmethod

class AbstractHello(metaclass=ABCMeta):
    @abstractmethod
    def hello(self):
        pass
```
継承先のサブクラス内でhello関数を定義しないとエラーが発生します。
```
class WrongHello(AbstractHello):
    pass
w = WrongHello()


# エラー
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-14-70ed5a2d6030> in <module>()
      1 class WrongHello(AbstractHello):
      2     pass
----> 3 w = WrongHello()

TypeError: Can't instantiate abstract class WrongHello with abstract methods hello
```
このようなメリットはありますが、別に抽象クラスを使わなくてもclassの継承を使った開発は行なえます。私も最初は抽象クラスを使わずに親クラスで共通プロパティを用意して、子クラスで別のプロパティを追加したり、メソッドのオーバーライドをしたりとPythonの柔軟な機能でリファクタリングを行ってました。

ですが、私も抽象クラスを使いたいなぁと思う場面に出会ってしまったのです。それは、チーム開発における機能の統一でした。コードが大規模化し、リファクタリングをせざる得なくなりオブジェクト指向プログラミングでのリファクタリングを行った時に、親クラスを継承する時に特定のメソッドを子クラスで必ずオーバーライドさせるようにしたいと思ったのです。親クラスのdocstringに書くのもいいですが、抽象クラスを使えば他のチームメンバーで私の意図が伝わってなくても、この親クラスを継承して間違った実装をしてもエラーを吐いてくれるので安心できます。

ただ、Pythonにおける抽象クラスを使う際の注意点が[こちらの記事](https://qiita.com/baikichiz/items/7c3fdb721bb72644f638#%E6%B3%A8%E6%84%8F%E7%82%B9)に書いてありますが、動的型付け言語なので厳密な型チェックが行える訳ではないので、それらをしっかりと把握した上で開発を行いましょうってことですね。