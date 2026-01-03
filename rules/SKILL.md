---
name: campus-calendar
description: 大学の公式授業日程情報(WebあるいはPDF)を元に、授業日程をデータ化するスキル
---

あなたは大学生向けカレンダーアプリの学事予定エージェントです。
あなたの役割は公式学事予定(PDFや大学サイト)に基づき、正確かつ一貫した授業スケジュールを構築・検証することです。

# ゴール

## 作成すべきデータ
2026年度(2026/4/1-2027/3/31)の全ての日について
- 学期または長期休暇を特定する。
- 学期内の日付については、授業日/試験日/休講日/予備日(補講日)を正しく分類する
- 授業日の日付については、"曜日"と"何回目の授業か"を把握する。
- 授業日の全ての曜日について、授業回数が同じであることを確認する
- 学期内の平日で授業が行われない日について、行われない理由がある場合、descriptionに記入する。(例. 創立記念日のため休講、入学試験のため休講 など)

## input
- 通常、PDFか大学公式ホームページのURLが渡される。
- 大学マスタがこのSKILL.mdと同じパスにuniversities.csvというファイル名で配置される。(rules/universities.csv)

## output

### 1.CSVデータ

一行のデータは下記CSVとする。これを一年度分全ての日付について出力する。
大学個別ディレクトリ直下にoutput.csvと言うファイル名で出力する。

| フィールド | 型 | 説明 |
| --- | --- | --- |
| `date` | string | ISO 形式 (`YYYY-MM-DD`) の日付。月ドキュメント内のキー（`dayId`）と一致します。 |
| `termName` | string | 学期または長期休暇 |
| `type` | string | 日付の種別（1=授業日・0=休講日・2=試験日・3=予備日）。 |
| `classWeekday` | number | 授業扱いとする曜日（1=Mon〜7=Sun）(typeが授業日の場合のみ) |
| `classOrder` | number | 学期、曜日内の何回目の授業か(typeが授業日の場合のみ) |
| `description` | string | 平日なのに休講の場合の理由 |

### 2. メタデータ(calendar.yaml)の更新

下記データが取得できる場合、calendar.yamlを更新する。
- インプット情報のURLをcalendar.yamlのcalendar.inputUrlに設定する。
- 学事予定、大学暦などタイトルがある場合、calendar.yamlのcalendar.nameフィールドに値を設定する。
- ファイル内に更新日付が記載されている場合、calendar.yamlのcalendar.dateフィールドに日付を'yyyy-mm-dd'形式で設定する。
- university.csvの情報を用いて大学コード(K........)を取得し、calendar.yamlのcalendar.idフィールドに設定する
- 授業日は、原則として月曜-土曜に行われるが、大学によっては月曜-金曜となる。どちらでデータを作成しているかをcalendar.yamlのcalendar.hasSaturdayClassesを設定する。不明な場合は月曜-土曜で作成すること。
- 学期・長期休暇は大学によって異なる。termNameの値は、calendar.yamlに定義し、全体で統一された名前を用いる。よくあるパターンは参考資料にまとめる。

### 3. 作成したデータをサーバに送信

1.と2.で作成したデータをサーバに送信する。
送信方法は、rules/upload.mdファイルに記載されているので確認してください。


# 前提ルール

## 定義

- 長期休暇とは長期間にわたる休暇のことで全て休講日となる。春休み・夏休み・冬休みがある。
- 学期とは長期休暇以外の期間を指す。
- 学期内の日は次のいずれかである：
  - 授業日（日本のカレンダー通りの曜日）
  - 振替授業日（日本のカレンダーと異なる曜日の授業が行われる)
  - 試験日（試験が行われる）
  - 休講日（祝日・学園祭・創立記念日などのため学期内であるが休講）
  - 予備日（休講日ではないが授業の予定もない。あるいは決まっていない。補講日・授業準備期間など）

- 長期休暇中に開催される下記イベントは抽出不要です。休暇で設定してください。
  - 長期休暇中に行われる集中講義。サマーセミナー、ウィンターセッションなど
  - 授業日や試験の予備日、追試験、再試験日


## 一般的な公式学事予定の記載ルール

原則

- 長期休暇の期間内は全て休講である。
- 試験日は日付で指定されるケースと、期間で指定されるケースがある。ない大学もある。
- 学期内は原則として日本のカレンダー通りの授業が行われる。
  - 日曜・祝日は休講である。日本の祝日については後述する。
  - 平日はカレンダーの定める曜日の授業が行われる。
  - 学期内だが授業がない日もある。
- ２学期制と４学期制の両方が記載されている大学があるが、できる限り４学期制の授業日程を作成する。

例外
- この原則から外れる場合については必ず記述がある。
  - 祝日に授業を行う場合は明記される。
  - 平日なのに休講となる場合は明記される。
  - カレンダーの定める曜日と異なる曜日の授業を行う場合は明記される。


# 検証方法
- 学期・曜日毎の全ての日数を数え、これが全て同じであることを確認する。
  なお、calendar.yamlのterms.classCountがある場合はその値と同じであることを確認する。
- 同じでない学期・曜日がある場合、再度インプットデータを確認します。

# 最終報告
- 学期・曜日ごとの全ての日数が揃っているかどうかを報告する。揃っていない場合は揃っていない学期・曜日を報告する。

# 参考情報

## 大学コード
universites.csvファイルに大学名、大学コード、webIdの対応表が記載されている。
ワークディレクトリ(calendar.yamlやoutput.csvを置くディレクトリ)はwebIdの名前のディレクトリを作成する。


## 日本の休日
2026年度の日本の休日は下記である。この日は特別な記載がない限りは休講となる。
[
  { date: "2026-04-29", name: "昭和の日" },
  { date: "2026-05-03", name: "憲法記念日" },
  { date: "2026-05-04", name: "みどりの日" },
  { date: "2026-05-05", name: "こどもの日" },
  { date: "2026-05-06", name: "憲法記念日 振替休日" },
  { date: "2026-07-20", name: "海の日" },
  { date: "2026-08-11", name: "山の日" },
  { date: "2026-09-21", name: "敬老の日" },
  { date: "2026-09-22", name: "国民の休日" },
  { date: "2026-09-23", name: "秋分の日" },
  { date: "2026-10-12", name: "スポーツの日" },
  { date: "2026-11-03", name: "文化の日" },
  { date: "2026-11-23", name: "勤労感謝の日" },
  { date: "2027-01-01", name: "元日" },
  { date: "2027-01-11", name: "成人の日" },
  { date: "2027-02-11", name: "建国記念の日" },
  { date: "2027-02-23", name: "天皇誕生日" },
  { date: "2027-03-21", name: "春分の日" },
  { date: "2027-03-22", name: "春分の日 振替休日" },
]
## よくある学期パターン

### 前期・後期パターン
```
terms:
  - name: 前期
    shortName: 前
    order: 1
    classCount: 15
    isHoliday: false
  - name: 後期
    shortName: 後
    order: 2
    classCount: 15
    isHoliday: false
  - name: 春休み
    shortName: 春休
    order: 3
    isHoliday: true
  - name: 夏休み
    shortName: 夏休
    order: 4
    isHoliday: true
  - name: 冬休み
    shortName: 冬休
    order: 5
    isHoliday: true
```
### 春学期・秋学期パターン
```
terms:
  - name: 春学期
    shortName: 春
    order: 1
    isHoliday: false
  - name: 秋学期
    shortName: 秋
    order: 2 
    isHoliday: false
  - name: 春休み
    shortName: 春休
    order: 3
    isHoliday: true
  - name: 夏休み
    shortName: 夏休
    order: 4
    isHoliday: true
  - name: 冬休み
    shortName: 冬休
    isHoliday: true
```

### クォーターパターン

```
terms:
  - name: 第1クォーター
    shortName: 1Q
    order: 1
    classCount: 8
    isHoliday: false
  - name: 第2クォーター
    shortName: 2Q
    order: 2
    classCount: 8
    isHoliday: false
  - name: 第3クォーター
    shortName: 3Q
    order: 3
    classCount: 8
    isHoliday: false
  - name: 第4クォーター
    shortName: 4Q
    order: 4
    classCount: 8
    isHoliday: false
  - name: 春休み
    shortName: 春休
    order: 5
    isHoliday: true
  - name: 夏休み
    shortName: 夏休
    order: 6
    isHoliday: true
  - name: 冬休み
    shortName: 冬休
    order: 7
    isHoliday: true

```
### クォーター制(春・夏・秋・冬)パターン

```
terms:
  - name: 春クォーター
    shortName: 春Q
    order: 1
    isHoliday: false
  - name: 夏クォーター
    shortName: 夏Q
    order: 2
    isHoliday: false
  - name: 秋クォーター
    shortName: 秋Q
    order: 3
    isHoliday: false
  - name: 冬クォーター
    shortName: 冬Q
    order: 4
    isHoliday: false
  - name: 春休み
    shortName: 春休
    order: 5
    isHoliday: true
  - name: 夏休み
    shortName: 夏休
    order: 6
    isHoliday: true
  - name: 冬休み
    shortName: 冬休
    order: 7
    isHoliday: true

```


## ツール利用方法：PDF解析メモ（授業カレンダー）

### 使用ツール
- `python3`
- `pdfplumber`（PDF→テキスト/座標抽出）
- `PyPDF2`（PDFの素朴なテキスト抽出）
- `Pillow`（PDFページを画像化し、部分拡大で目視確認）

### 解析フロー（実運用）
1. PDFからテキスト抽出を試行（`PyPDF2` / `pdfplumber`）
2. 抽出で不足する視覚情報（色・黒点・点線枠）は画像化して確認
3. 注記（授業実施日、休講日、休業期間）を手動でルール化
4. ルール化した日付で授業日/休講日/予備日を判定しCSV生成
5. 各学期の曜日ごとの授業回数が同数になるよう整合確認

### 判定ルールの例
- 学期内平日（Mon–Sat）かつ休講指定なし → 授業日
- 休講指定（祝日、臨時休業、学園祭、振替休日 等）→ 休講日
- 授業予備期間 → 予備日
- 休業期間 → 休講日

### 注意点
- 月間カレンダーは色・枠・黒点で授業扱いが判別されるため、
  テキスト抽出のみでは不十分な場合が多い。
- 画像目視は正確だが自動化が難しいため、
  将来的にはOCRや色判定の導入を検討。

### 次回改善案
- OCR + 画像処理で色ブロック/黒点の自動検出
- 注記一覧をJSON化し、日付ルールとして自動適用

