# G医t Data Repository (`g-i-t-data`)

G医t (G-I-T) のマスター設定、監視対象サイトのスナップショット、GitHub Pages 公開用のデータリポジトリです。
監視データはすべてこのリポジトリに集約します（サブモジュールや個別リポジトリは使いません）。

## ディレクトリ構成

```text
g-i-t-data/
├── config.yaml                # 監視対象URLのマスター設定
├── public/                    # GitHub Pages にデプロイされる静的ダッシュボード
│   ├── index.html
│   └── sites/<slug>/          # サイト別履歴 (history.html) と差分HTML (diff_<hash7>.html)
└── sites/<slug>/              # website-stalker.yaml と取得したページ（通常のディレクトリ）
```

## 運用

- 1 サイトの 1 回の実行結果は 1 コミット (`Update <slug>`) になります。
- サイト別の履歴: `git log -- sites/<slug>/`
- 新しい URL は `config.yaml` に追記するだけで、次回の実行時に `sites/<slug>/` が自動作成されます。

## config.yaml の項目

```yaml
sites:
  - url: https://www.naika.or.jp/
    name: "日本内科学会"        # 表示名（日本語・UTF-8）
    slug: "naika"              # ASCII 小文字・数字・- _ のみ。ディレクトリ名と URL になる
    tags: ["学会", "内科"]      # 任意。絞り込みと分類別 Atom フィードに使う
    ignore:                    # 任意。取得したページから消す（毎回変わる）文字列の正規表現
      - ';jsessionid=[0-9A-Fa-f]+'                                   # 消す
      - { pattern: 'token=[0-9a-f]+', replace: 'token=X' }          # 置き換える
```

全サイト共通の規則は、`config.yaml` のトップレベルに `ignore:` として書けます。`file.pdf?1700000001` のような WordPress のキャッシュ回避の数値は、標準で自動的に消えます（効かせたくないサイトは `default_ignore: false`）。

`ignore` は Rust の正規表現（先読み・後方参照は使えません）で、YAML では **シングルクォート**で囲むと `\` をそのまま書けます。
既存サイトに追加した場合は、次回の実行時に `sites/<slug>/website-stalker.yaml` へ追記されます（その回の更新としては 1 回だけ差分が出ます）。
