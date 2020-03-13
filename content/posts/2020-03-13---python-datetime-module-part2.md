---
title: Pythonで時間を扱う時に私が使っているモジュールたちパート2
date: "2020-03-13T21:00:00+09:00"
template: "post"
draft: false
slag: "python-datetime-module-part2"
category: "Python"
tags:
    - "Python"
    - "datetime"
    - "dateutil"
    - "timezone"
    - "ISO"
    - "Tech"
    - "pytz"
description: "前回に引き続き、私の中で定着してきた時間を扱う時のPythonモジュールたちを紹介しようと思います"
---

　今回は時間をオブジェクトから文字列、その逆の文字列からオブジェクトの変換について書こうと思います。

## 文字列出力の際はisoformatメソッドを使ってます！
　isoformatメソッドはバージョン3.6から追加された機能なんですが凄い便利です。そもそもdatetimeオブジェクトから文字列を出力する時はどうしてますか？単純にprint文にオブジェクトを渡してますか？そんな事はないですよね。今まではstrftime()メソッドを使っていたかと思います。この表記方法は仕方なく使っていましたが、ぶっちゃけ凄い見辛いし、分かりにくいところもありました。では、isofromat()メソッドで同じ出力をする場合はどうなのか？
```
>>> from datetime import datetime
>>> from pytz import timezone
>>> jst_now = datetime.now(timezone("Asia/Tokyo"))
>>> jst_now.strftime("%Y-%m-%d %H:%M:%S.%f%z") タイムゾーンあり
'2020-03-13 20:16:15.863140+0900'
>>> jst_now.strftime("%Y-%m-%d %H:%M:%S.%f") タイムゾーンなし
'2020-03-13 20:16:15.863140'
```

まず、何も引数を与えない場合だと、日付(date)と時刻(time)のセパレートが"T"として出力されます。なので、引数でセパレータを空白にすれば上記と全く同じ出力となります。凄い分かりやすくなっているかと思います。
```
>>> jst_now.isoformat()
'2020-03-13T20:16:15.863140+09:00'
>>> jst_now.isoformat(' ')
'2020-03-13 20:16:15.863140+09:00'
```

現在はこのisoformatを使って、以下のように使い分けています。
```
>>> jst_now.isoformat(' ', 'seconds')
'2020-03-13 20:16:15+09:00'
>>> jst_now.isoformat(' ', 'minutes')
'2020-03-13 20:16+09:00'
>>> jst_now.isoformat(' ', 'hours')
'2020-03-13 20+09:00'
```

　オブジェクトから文字列を出力することができるのなら、その逆も同じで今まではstrptime()メソッドをstrftime()と同じように使っていたかと思います。バージョン3.7からはfromisoformat()というメソッドが追加されまして、私はそれを使っておりました。
```
>>> str_datetime = "2020-03-01T10:00:00.000000+09:00"
>>> str2obj = datetime.strptime(str_datetime, "%Y-%m-%dT%H:%M:%S.%f%z")
>>> str2obj
datetime.datetime(2020, 3, 1, 10, 0, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400)))
```

strptime()の使い辛い所ってやっぱり柔軟性のなさだと思うんですよね。この表記に"%Y-%m-%dT%H:%M:%S.%f%z"合致してないとダメだから以下のパターンの文字列は全部エラーを吐くんだよね〜。
```
>>> str_datetime = "2020-03-01T10:00:00.000000"
>>> str2obj = datetime.strptime(str_datetime, "%Y-%m-%dT%H:%M:%S.%f%z")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Users/tamash/.pyenv/versions/3.7.6/lib/python3.7/_strptime.py", line 577, in _strptime_datetime
    tt, fraction, gmtoff_fraction = _strptime(data_string, format)
  File "/Users/tamash/.pyenv/versions/3.7.6/lib/python3.7/_strptime.py", line 359, in _strptime
    (data_string, format))
ValueError: time data '2020-03-01T10:00:00.000000' does not match format '%Y-%m-%dT%H:%M:%S.%f%z'
```
- "2020-03-01T10:00:00.000000"
- "2020-03-01T10:00:00+09:00"
- "2020-03-01T10:00+09:00"
- "2020-03-01T10+09:00"

文字列のバリデーションをするのにものすごく面倒くさいのがstrptime()だったのですが、fromisoformat()の追加で大分救われましたｗ
```
>>> str_datetime = "2020-03-01T10:00:00.000000+09:00"
>>> str2obj = datetime.fromisoformat(str_datetime)
>>> str2obj
datetime.datetime(2020, 3, 1, 10, 0, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400)))
>>> str_datetime = "2020-03-01T10:00:00.000000"
>>> str2obj = datetime.fromisoformat(str_datetime)
>>> str2obj
datetime.datetime(2020, 3, 1, 10, 0)
>>> str_datetime = "2020-03-01T10:00:00+09:00"
>>> str2obj = datetime.fromisoformat(str_datetime)
>>> str2obj
datetime.datetime(2020, 3, 1, 10, 0, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400)))
>>> str_datetime = "2020-03-01T10:00+09:00"
>>> str2obj = datetime.fromisoformat(str_datetime)
>>> str2obj
datetime.datetime(2020, 3, 1, 10, 0, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400)))
>>> str_datetime = "2020-03-01T10+09:00"
>>> str2obj = datetime.fromisoformat(str_datetime)
>>> str2obj
datetime.datetime(2020, 3, 1, 10, 0, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400)))
```

このfromisoformat()は強力だとは思うんですが、この記事を考えている時に見つけたドキュメントでは以下のように書いてありました。あくまでisoformat()の逆操作を簡単に行えるものを用意しただけで、厳密にISO 8601形式を構文解析している訳ではないとのこと。

> Caution: This does not support parsing arbitrary ISO 8601 strings - it is only intended as the inverse operation of datetime.isoformat(). A more full-featured ISO 8601 parser, dateutil.parser.isoparse is available in the third-party package dateutil.

例えば、UTCを表す以下のような文字列ではfromisoformat()はエラーを吐きます。
```
>>> str_datetime = "2020-03-01T10:00:00.000000Z"
>>> str2obj = datetime.fromisoformat(str_datetime)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: Invalid isoformat string: '2020-03-01T10:00:00.000000Z'
```

ということで、現在はこのドキュメントに書いてあるサードパーティのdateutilモジュールを使ってます。実際にUTC表記でも対応してくれるので大変助かってます！
```
>>> from dateutil import parser
>>> str_datetime = "2020-03-01T10:00:00.000000Z"
>>> str2obj = parser.isoparse(str_datetime)
>>> str2obj
datetime.datetime(2020, 3, 1, 10, 0, tzinfo=tzutc())
```

ということで、私の使うモジュールたちを紹介しました。次は何を書こうかな〜（さいなら〜）