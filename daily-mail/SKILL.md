---
name: daily-mail
description: 企業の調達・購買に関するニュース・記事を収集し、デイリーダイジェストをGmailとSlack DMで配信する
command: daily-mail
user-invocable: true
---

# 調達・購買デイリーメール

企業の調達・購買に関する最新ニュース・記事を自動収集し、デイリーダイジェストとしてGmailとSlack DMに配信するスキルです。

## 実行手順

### Step 1: 設定ファイルの読み込み

`config.json` を読み込み、検索キーワード、RSSフィード、配信先情報を取得する。

### Step 2: ニュース収集

以下の3つのソースから並行して記事を収集する。

#### 2a. WebSearch による検索

`config.json` の `keywords` セクションの各カテゴリのキーワードで WebSearch を実行する。

- 各カテゴリから最大2-3キーワードを選んで検索（全24キーワードを検索すると時間がかかるため、カテゴリごとに代表的なものを選ぶ）
- 検索クエリには現在の年を含めて最新ニュースを優先する
- SNS検索: キーワードに `site:x.com` や `site:linkedin.com` を付与して別途検索する

**検索の優先順位（上から順に実行、時間が許す限り）:**
1. 「企業 調達 購買 ニュース 最新」
2. 「サプライチェーン 最新動向 日本」
3. 「調達DX デジタル化 企業」
4. 「クラウド調達 IaaS SaaS PaaS」
5. 「ITAM SAM HAM CMDB」
6. 「CIO CDO CDIO 交代 新任 着任」
7. 「procurement supply chain news Japan」
8. 残りのキーワードを順次検索

#### 2b. RSSフィード取得

WebFetch で各RSSフィードURLを取得し、調達・購買に関連する記事をフィルタリングする。

- ITmedia エンタープライズ: `https://rss.itmedia.co.jp/rss/2.0/enterprise.xml`
- 日経クロステック: `https://xtech.nikkei.com/rss/index.rdf`
- ZDNet Japan: `https://japan.zdnet.com/rss/index.rdf`
- MONOist: `https://rss.itmedia.co.jp/rss/2.0/monoist.xml`

RSSの記事タイトル・概要に、config.jsonのキーワードに関連する内容が含まれているもののみを抽出する。

#### 2c. SNS検索

WebSearch で以下を検索:
- 主要キーワード + `site:x.com` （X/Twitter）
- 主要キーワード + `site:linkedin.com` （LinkedIn）

### Step 3: 記事の整理・分類

収集した全記事を以下の基準で整理する:

1. **重複排除**: 同じURLの記事を除外
2. **カテゴリ分類**: 以下のカテゴリに分類
   - 調達・購買全般
   - サプライチェーン
   - 調達DX
   - IT調達・管理
   - IT資産管理
   - ベンダー管理
   - 組織・人事
   - その他
3. **URLリンク**: 各記事に必ずURLリンクを付与する
4. **要点**: 各記事が最も何を伝えたいかを100字程度で記載
5. **サマリー**: 各記事を200字〜400字程度でまとめる
6. **InProc影響度**: 株式会社InProcの事業（調達・購買コンサル、SCM最適化コンサル、IT調達最適化コンサル、AIプロダクト事業）への影響度を判定
   - 高(★★★): InProcの事業に直接影響（新規案件創出、サービス展開に直結）
   - 中(★★): 間接的な影響（市場トレンド、競合動向）
   - 低(☆): 参考情報レベル
7. **記事数制限**: カテゴリあたり最大5件、全体で最大30件

### Step 4: ダイジェスト生成

#### Gmail用（HTML形式）

以下のフォーマットでHTML形式のメール本文を生成する:

```
件名: [調達・購買ニュース] YYYY年MM月DD日のダイジェスト

本文:
# 調達・購買デイリーダイジェスト - YYYY年MM月DD日

## 本日のハイライト
- 最も重要な記事3件の要約

## カテゴリ別ニュース

### 調達・購買全般
- **[記事タイトル](URL)** ★★★
  要点: ○○が△△に影響。InProcの□□事業に直結。
  サマリー: 200〜400字の要約...

（各カテゴリ同様のフォーマット）

## SNS注目投稿
- **[投稿元](URL)** ★★
  要点: ...
  サマリー: ...

---
このメールは Claude Code のデイリーメール自動化により生成されました。
```

#### Slack DM用（Markdown形式）

```
*調達・購買デイリーダイジェスト - YYYY年MM月DD日*

:star: *本日のハイライト*
• 最も重要な記事3件の要約

*カテゴリ別ニュース*
> *調達・購買全般*
> • <URL|記事タイトル> - 1行サマリー

（各カテゴリ、コンパクトに）

_Claude Code 自動配信_
```

### Step 5: 配信

1. **Gmail**: `gmail_create_draft` でドラフトを作成する
   - To: `hamu9811@gmail.com, tetsuhiro.tsuchiya@inproc.org`（config.jsonのgmail_to）
   - Subject: `[調達・購買ニュース] YYYY年MM月DD日のダイジェスト`
   - Body: HTML形式のダイジェスト
   - contentType: `text/html`
   - ドラフト作成後、ユーザーに確認を求める（手動実行時のみ）

2. **Slack DM**: `slack_send_message` で自分宛てDMを送信する
   - channel_id: `U0A7TP2MECT`（config.jsonのslack_dm_user_id）
   - message: Markdown形式のダイジェスト

### Step 6: 完了報告

配信結果をユーザーに報告する:
- 収集記事数
- カテゴリ別の内訳
- Gmailドラフト作成の成否
- Slack DM送信の成否
