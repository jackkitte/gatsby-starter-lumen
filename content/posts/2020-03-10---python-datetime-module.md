---
title: Pythonで時間を扱う時に私が使っているモジュールたち
date: "2020-03-10T21:00:00+09:00"
template: "post"
draft: false
slag: "python-datetime-module"
category: "Python"
tags:
    - "Python"
    - "datetime"
    - "timezone"
    - "ISO"
    - "Tech"
    - "pytz"
description: "Python開発で時刻を扱う時に色々なモジュールを使ったやり方があるが、私の中で定着してきたモジュールたちがいるのでそれを紹介しようかと思います"
---

　クラウドとかで開発をしていると時間を扱うときにタイムゾーンを気にする必要が多々あるかと思います。また、時間の表記はどれにしたらいいんだろうかということもよくある話です。そんな中、私の中でPythonを使って時間を扱うときのある程度のパターンみたいなのが出来つつあるのでそれを記事にしました。

## 定番だがpythonでタイムゾーンを扱うときはpytzモジュールを使ってます！
　Pythonでタイムゾーンを扱うときって、例えば以下のようにすればUTCのタイムゾーンを持ったdatetimeオブジェクトを生成できます。
```
from datetime import datetime
from datetime import timezone
from datetime import timedelta

>>> now = datetime.now(timezone.utc)
>>> now
datetime.datetime(2020, 3, 10, 10, 41, 19, 426067, tzinfo=datetime.timezone.utc)
```

UTCならまだいいですが、JSTの場合だと以下のようになります。どうでしょうか？長ったらしいし、32400秒とか分かりにくくないですかｗ  
また、設定の部分を見て下さい。``timezone(timedelta(hours=+9))``こいつ、割と面倒くさくないですか？
```
>>> now_jst = datetime.now(timezone(timedelta(hours=+9)))
>>> now_jst
datetime.datetime(2020, 3, 10, 19, 44, 2, 527212, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400)))
```

このタイムゾーン設定をpytzを使うと以下のようになります。どうですか？timedeltaとか+9とかやらずに'Asia/Tokyo'と指定するだけでJSTが設定できます。しかも、datetimeオブジェクトの中身も分かりやすくないですか。
```
from pytz import timezone as timezone_pytz

>>> now_jst_pytz = datetime.now(timezone_pytz('Asia/Tokyo'))
>>> now_jst_pytz
datetime.datetime(2020, 3, 10, 19, 44, 51, 654476, tzinfo=<DstTzInfo 'Asia/Tokyo' JST+9:00:00 STD>)
```

まぁ、現状はJSTとUTCしか使わないから、無理してpytz（サードパーティ製なのでpip installが必要です）を使うこともないけど（+9時間ってのを覚えておけば標準ライブラリだけで書ける）、こんなん見せられてもパッと見では分からないかと思います。
```
datetime.datetime(2020, 3, 10, 19, 44, 2, 527212, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400)))
```
それよりはこっちのがパッと見で何となくだけど、日本時間のことなんだなぁと分かるので、後々役に立ってくると思うんですよね〜。（デバッグやらログ分析などで）
```
datetime.datetime(2020, 3, 10, 19, 44, 51, 654476, tzinfo=<DstTzInfo 'Asia/Tokyo' JST+9:00:00 STD>)
```

他にも書くことあったんですけど、そいつを含めると長くなっちゃうので次の記事に持ち越そうかと思います。ほなねー！