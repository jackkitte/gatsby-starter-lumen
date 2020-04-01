---
title: Google AnalyticsのアクセスデータをElasticsearchに投入してみる！
date: "2017-12-29T21:30:00+09:00"
template: "post"
draft: false
slag: "google-analytics-to-elasticsearch"
category: "elasticsearch"
tags:
    - "Elasticsearch"
    - "Google Analytics"
    - "Tech"
description: "今回は、GAの生データをGoogle Analytics APIを使ってPythonで取得し、そいつをElasticsearchに突っ込んで可視化してみました！"
---
　どうも。縁あって某グループ会社のGoogle Analytics(以下GAと呼びます)にアクセスすることができましたので、GAを使ったアクセス解析をやらせて貰ってました。

流石Googleさん！色々なデータが入っており、機能も充実していてとても面白かったのですが、もっと柔軟なこともやりたいなとも思いました。そこで、GAの生データを取得できないかなぁと調べていたらあるじゃないですか〜！しかも、その生データをPythonを使って取得することもできるなんて最高かよっ！

ということで、今回は、GAの生データをGoogle Analytics APIを使ってPythonで取得し、そいつをElasticsearchに突っ込んで可視化してみました！

## 導入環境

- OS : Ubuntu 16.04 LTS / 64bit
- Python : 3.6
- Elasticsearch : 6.0.0

## 事前準備
GAのAPIを使うために、Google Developers ConsoleというWebサイトで様々な情報の登録が必要となってきます。ここでは、詳細は省きますので、登録方法については以下のページの目次0〜3までを参照して下さい。

[Google AnalyticsのデータをPython経由で収集する](https://dev.classmethod.jp/business/business-analytics/getting-google-analytics-data-via-python/#toc-)

## PythonでGoogle APIを使うためのライブラリのインストール
インストール方法は簡単で、以下のようにpipを使うことでインストールできます。

`$ sudo pip install --upgrade google-api-python-client`

## PythonでElasticsearchにアクセスするためのライブラリのインストール
こちらもpipで簡単にインストールできます。

`$ sudo pip install elasticsearch`

## Pythonコード
それでは早速コードを書いていきましょう。まず始めに、認証情報の生成、HTTP通信用のオブジェクト、Google APIの各サービス用(Google Compute EngineやGoogle+など)のオブジェクトの作成を行う関数を定義しています。
credentilasオブジェクトで認証情報を生成します。その次に、GoogleとHTTP通信するためのHTTP通信用のオブジェクトに認証情報を付与して生成します。最後に、Google APIのオブジェクト(今回はGoogle Analyticsを指定)を生成して、そのオブジェクトを返します。

```python3
# -*- coding: utf-8 -*-

from apiclient.discovery import build
from oauth2client.service_account import ServiceAccountCredentials

import httplib2
from oauth2client import client
from oauth2client import file
from oauth2client import tools

SCOPES = ['https://www.googleapis.com/auth/analytics.readonly']
DISCOVERY_URI = ('https://analyticsreporting.googleapis.com/$discovery/rest')
KEY_FILE_LOCATION = '/Keyファイルの保存先/Google-Analytics-python-*.p12'
SERVICE_ACCOUNT_EMAIL = '事前準備で作ったアドレス@*.iam.gserviceaccount.com'
VIEW_ID = 'Google Analyticsでデータを取ってきたいビューのID'

def initialize_analyticsreporting():

  credentials = ServiceAccountCredentials.from_p12_keyfile(
    SERVICE_ACCOUNT_EMAIL, KEY_FILE_LOCATION, scopes=SCOPES)

  http = credentials.authorize(httplib2.Http())

  analytics = build('analytics', 'v4', http=http, discoveryServiceUrl=DISCOVERY_URI)

  return analytics
```

次は、Google APIを通じて実際にデータを引っ張ってくる処理を実行する関数です。

ここまでのPythonコードをGoogleAnalyticsAPIv4.pyとして保存します。

```python3
def get_report(analytics, body):

  return analytics.reports().batchGet(body=body).execute()
```


次は、GAのデータをElasticsearchに格納するプログラムです。import GoogleAnalyticsAPIv4で先ほど作成したPythonプログラムを読み込みます。以下は、GAのパラメータ設定値とElasticsearchのインデックスのマッピング設定値です。設定値の詳細は以下のリンクをそれぞれ参照して下さい。

[Google Analytics API v4 各メソッド](https://developers.google.com/analytics/devguides/reporting/core/v4/rest/v4/reports/batchGet?hl=ja)

[Elasticsearch mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html)

```python3
# -*- coding: utf-8 -*-

import GoogleAnalyticsAPIv4
from elasticsearch import Elasticsearch

# Google Analytics API v4 へアクセスするためのパラメーター
view_id = "6540339"
dateRanges = [{"startDate": "2016-10-01", "endDate": "2017-09-30"}]
dimensions = [{"name": "ga:region"}, {"name": "ga:date"}]
dimensionFilterClauses = [{"operator": "AND",
                           "filters":[{"dimensionName": "ga:country",
                                       "not": "false",
                                       "operator": "REGEXP",
                                       "expressions": ["Japan"],
                                       "caseSensitive": "false",}],
                           }]
metrics = [{"expression": "ga:sessions", "formattingType": "INTEGER"}]
pageSize = 10000
body = {"reportRequests": []}
body["reportRequests"].append({"viewId": view_id,
                                "dateRanges": dateRanges,
                                "dimensions": dimensions,
                                "dimensionFilterClauses": dimensionFilterClauses,
                                "metrics": metrics,
                                "pageSize": pageSize,
                               })

# Elasticssearchのインデックスのマッピング
index = "googleanalytics"
mapping = {"mappings": {"都道府県別セッション": {"properties": {}}}}
ga_date = {"type": "date", "format": "yyyyMMdd"}
mapping["mappings"]["都道府県別セッション"]["properties"]["ga:date"] = ga_date
```

次は、Elasticsearchのインデックスを作成する関数です。deleteで同じインデックス名があった場合に削除を行います。続くcreateで新しいインデックスを作成します。

```python3
sticsearch_index_create(index, mapping):

    es = Elasticsearch()
    es.indices.delete(index=index)
    es.indices.create(index=index, body=mapping)

    return es
```

次は、GAのAPIで取得したデータをインデックスに格納していく関数です。response_listがAPIで取得したデータが格納されているオブジェクトで、JSON形式になっています。

for文を使って、JSON形式の各項目にアクセスしながら目的のデータを見つけて辞書型の変数docに格納します。

最後に、elasticsearchのindex関数にそれぞれの情報と取ってきたデータが格納されているdocを指定することで、インデックスへのデータの格納が完了します。

```python3
def elasticsearch_register(es, response_list):

    doc = {}
    id = 0
    for response in response_list:
        for report in response.get("reports", []):
            columnHeader = report.get("columnHeader", {})
            dimensionHeaders = columnHeader.get("dimensions", [])
            metricHeaders = columnHeader.get("metricHeader", {}).get("metricHeaderEntries", [])
            rows = report.get("data", {}).get("rows", [])

            for row in rows:
                dimensions = row.get("dimensions", [])
                metrics = row.get("metrics", [])

                for header, dimension in zip(dimensionHeaders, dimensions):
                    doc[header] = dimension

                for values in metrics:
                    for metricHeader, value in zip(metricHeaders, values.get("values")):
                        doc[metricHeader.get("name")] = int(value)

                es.index(index="googleanalytics", doc_type="都道府県別セッション", id=id+1, body=doc)
                id += 1
```

最後は、main関数を定義して実際の処理の流れを定義しています。

```python3
def main():

    analytics = GoogleAnalyticsAPIv4.initialize_analyticsreporting()
    response = GoogleAnalyticsAPIv4.get_report(analytics, body)
    response_list = []
    response_list.append(response)

    while response.get("reports", [])[0].get("nextPageToken"):
        body["reportRequests"][0]["pageToken"] = response.get("reports", [])[0].get("nextPageToken")
        response = GoogleAnalyticsAPIv4.get_report(analytics, body)
        response_list.append(response)

    es = elasticsearch_index_create(index, mapping)
    elasticsearch_register(es, response_list)

if __name__ == "__main__":
    main()
```

ここまでのPythonプログラムをGoogleAnalyticsAPI2Elasticsearch.pyとして保存し、実行します。

`python3.6 GoogleAnalyticsAPI2Elasticsearch.py`

## Kibanaでの可視化
実行すると、以下のようにインデックスが作成されていました。

![Kibana_GA_index.png](/google_analytics/Kibana_GA_index.png)

データのほうもしっかりと登録されていました。

![Kibana_GA_visualize.png](/google_analytics/Kibana_GA_visualize.png)

## 可視化までして、いざこれからって時に.........
意外と上手くいけましたので、こいつを使って様々なデータの可視化や、Pythonを使った機械学習を行っていくぞーーーー！.........と思っていたものの、なんやかんやで某グループ会社とのプロジェクトが凍結になりまして、データが使えなくなってしまいましたーーーーorz トホホ(´；ω；｀)

まぁでも、Elasticsearchの勉強にはなりましたのでそれだでも良しとします！！！次回は何にしようかなぁ〜。
