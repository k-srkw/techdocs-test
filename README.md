# techdocs-test

Backstage TechDocs のサンプル。**Markdown の Admonitions（`!!! note` など）の中に
HTML のテーブルタグを含められるか** を検証したものです。

## 結論

| 書き方 | 結果 |
| --- | --- |
| `!!! note` + 4 スペースインデントの `<table>` | **含められる**（`colspan` / `rowspan` も可） |
| 同上で `<table>` の途中に空行 | **壊れる**（段落が分断され `<tbody>` が切り離される） |
| 同上でセル内の Markdown | `markdown="1"` なしでも解釈される（属性は不要） |
| `<div class="admonition …" markdown="block">` | **推奨**。空行 OK・Markdown 混在 OK |
| インデントなしの `<table>` | Admonition の外に出る |

インデントされた HTML は「HTML ブロック」ではなく **段落内のインライン HTML** として
扱われる（生成 HTML は `<p><table>…</table></p>`）ことが原因です。
詳細は `docs/findings.md` を参照してください。

## 構成

```
mkdocs.yml                       MkDocs 設定（plugins: - techdocs-core のみ）
catalog-info.yaml                Backstage カタログ登録用
docs/index.md                    概要と結論
docs/admonitions-html-table.md   検証パターン集（実際にレンダリングされる）
docs/findings.md                 検証環境・手順・生成 HTML・所見
```

## ビルド

```bash
python3 -m venv venv
./venv/bin/pip install mkdocs-techdocs-core
./venv/bin/mkdocs build --strict   # site/ に出力
./venv/bin/mkdocs serve            # http://127.0.0.1:8000
```

Backstage CLI を使う場合:

```bash
npx @techdocs/cli generate --no-docker --source-dir . --output-dir ./site
```

## Backstage への登録

`catalog-info.yaml` を Backstage のカタログに登録すると、
`backstage.io/techdocs-ref: dir:.` によって同リポジトリの `mkdocs.yml` が使われます。
