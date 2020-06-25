---
title: PythonでJSONを扱うとき
date: "2020-06-25T19:20:00+09:00"
template: "post"
draft: false
slag: "JSON"
category: "JSON"
tags:
    - "JSON"
    - "Python"
description: "Pythonには組込みでJSONを扱う「json」というライブラリがありますが、上手く扱えないデータ型あったときの対処法を紹介します。"
---

　仕事でPythonを使ってサーバーレス開発をやっているのですが、サーバーレスになるとどうしてもREST APIを使ったデータのやり取りが多くなってきます。この時に多用するのがJSON形式のデータを変換する「json」ライブラリです。

## jsonライブラリの使い方
```
import json
from datetime import datetime

person = {
    "name": "jackkitte",
    "age": "秘密だぞ〜",
    "created_at": datetime(2020, 6, 25)
}

json.dumps(person)
# TypeError: datetime.datetime(2020, 6, 25, 0, 0) is not JSON serializable
```
　上記のようにjsonライブラリをインポートして、`json.dumps()`関数を使うことでJSON形式の文字列を出力できます。しかしながら上記の場合ですと、dict型のデータの中にdatatime型のオブジェクトがあるためにTypeErrorを起こしています。jsonライブラリでは[こちらのドキュメント](https://docs.python.org/ja/3/library/json.html#py-to-json-table)にある型しか対応しておりません（以下の対応表）。  
そのためPythonの組込み以外のクラスオブジェクトやインスタンスオブジェクトが含まれるとTypeErrorとなるのです。

|Python|JSON|
|:------:|:----:|
|dict|object|
|lilst, tuple|array|
|str|string|
|int, floatとintやfloatの派生列挙型|number|
|True|true|
|False|false|
|None|null|

　それではこの問題をどう解決すればいいのかと言いますと、jsonライブラリのdefault引数に関数を設定し、その関数に対応していない型が含まれていた場合の処理を記述することで、TypeErrorを回避し正常にJSON形式の文字列を出力することができます。
```
def support_json_format(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    raise TypeError(repr(obj) + " is not JSON serializable")

person = {
    "name": "jackkitte",
    "age": "秘密だぞ〜",
    "created_at": datetime(2020, 6, 25)
}

json.dumps(person, default=support_json_format)
# {"created_at": "2020-06-25T00:00:00", "age": "秘密だぞ〜, "name": "jackkitte"}
```

ちなみに、AWSのサービスを使ってる時に遭遇したTypeErrorに、DynamoDBのDecimal型とLambdaのinvokeのレスポンスで返ってくるbotocore.response.StreamingBodyオブジェクトがあったので、以下のように対処しました。
```
from botocore.response import StreamingBody # boto3をpip install
from decimal import Decimal

def support_json_format(obj):
    if isinstance(obj, Decimal):
        return float(obj)
    elif isinstance(obj, datetime):
        return obj.isoformat()
    elif isinstance(obj, StreamingBody):
        data = obj.read()
        return data.decode('utf-8')
    raise TypeError(repr(o) + " is not JSON serializable")
```