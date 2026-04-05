# 調達・購買デイリーメール自動化プロジェクト

## 概要
企業の調達・購買に関するニュース・記事を毎日自動収集し、GmailとSlack DMで配信するデイリーダイジェストシステム。

## プロジェクト構成
- `config.json` - 検索キーワード、RSSフィード、配信先の設定
- `SKILL.md` - `/daily-mail` スキル定義（手動実行用）

## 重要なルール
- 各記事には必ず**URLリンク**を貼ること
- 各記事に**サマリー（200〜400字）**、**要点（100字）**、**InProc影響度（★★★/★★/☆）**を付与すること
- InProc影響度は、株式会社InProcの調達・購買コンサル、SCM最適化、IT調達最適化コンサル、AIプロダクト事業への影響で判定
- Gmail配信先: hamu9811@gmail.com, tetsuhiro.tsuchiya@inproc.org
- Slack DM: U0A7TP2MECT
