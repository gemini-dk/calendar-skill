---
name: upload-calendar-data
description: 作成したカレンダーデータをサーバにアップロード・投入する方法
---

# 必要なデータ
campus-calendarのSKILL.mdに基づいて作成した下記ファイル
- calendar.yaml
- output.csv

いずれも状況によってファイル名が異なる場合があります。

# 概要
1. calendar.yamlの内容をJSONパラメータに変換し、APIを呼び出してカレンダーを作成します。
2. 1.の戻り値でカレンダーIDが取得できるので、データ登録APIを用いてCSVデータをアップロードします。

# 詳細.1 カレンダーの作成方法
calendar.yamlの内容を下に下記APIを実行します。
<host>は、"https://college-calendar-admin.vercel.app/" です。

```
curl -X POST https://<host>/api/calendars/create \
  -H "Content-Type: application/json" \
  -d '{
    "name": "〇〇大学 2026年度",
    "universityCode": "K12345",
    "fiscalYear": 2026,
    "terms": [
      { "name": "前期", shortName: "前", "order": 1, "holidayFlag": 2 },
      { "name": "後期", shortName: "後", "order": 2, "holidayFlag": 2 }
    ],
    "inputInformation": "inputUrlの内容",
    "runAgent": false
  }'
```
この返却値は下記フォーマットです。idの値が次のリクエストに必要になります。
{ ok: true, id: <id>, agentTriggered: false } 

# 詳細.2 カレンダーデータ(CSV)のアップロード

```
curl -X POST https://<host>/api/calendars/import-csv \
  -H "Content-Type: application/json" \
  -d '{
    "calendarId": "<create APIで取得したID>",
    "csv": "date,type,term,classWeekday,description,notificationReasons\n2026-04-01,授業日,前期,3,初回ガイダンス,\n2026-04-02,授業日,前期,4,,補講予定\n"
  }'
```

csvの値に、作成したCSVファイルのデータを一行に変換して設定します。