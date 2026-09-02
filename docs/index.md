# TechDocs サンプル: Admonitions 内の HTML テーブル

このサイトは **Backstage TechDocs**（`mkdocs-techdocs-core`）のサンプルであり、
Markdown の Admonitions（`!!! note` などのブロック）の中に
**HTML のテーブルタグ（`<table>` / `<tr>` / `<td>` …）を含められるか** を検証するために作成しました。

## 結論

!!! success "含められます"

    Admonition の本文を **4 スペース（またはタブ 1 つ）でインデント** していれば、
    生の HTML テーブルはそのまま Admonition の枠内に描画されます。
    `colspan` / `rowspan` など Markdown のテーブル記法では書けない表現も使えます。
    折りたたみ（`??? note`）やネストした Admonition の中でも同様に動きます。

!!! warning "ただし、インデント形式には 3 つの落とし穴があります"

    インデントされた HTML は「HTML ブロック」ではなく **段落内のインライン HTML** として
    扱われます（生成 HTML は `<p><table>…</table></p>`）。そのため:

    1. **`<table>` の途中に空行を入れるとテーブルが壊れます。** 段落が分断され、
       `<tbody>` が別の段落に切り離されます。
    2. **セル内の Markdown は `markdown="1"` なしでも解釈されます。**
       逆に `*` や `` ` `` を文字として出したい場合はエスケープが必要です。
       `markdown="1"` は不要で、書くと属性が出力にそのまま残ります。
    3. **インデントが 4 スペース未満になるとテーブルが枠の外に出ます。**
       整形ツールがインデントを削るケースに注意してください。

!!! tip "複雑なテーブルは div 形式が確実です"

    HTML が長い場合や空行を挟みたい場合は、Admonition 自体を HTML で書き
    `markdown="block"` を付けてください。インデント不要・空行 OK・Markdown 混在 OK になります。

    ```markdown
    <div class="admonition note" markdown="block">
    <p class="admonition-title">タイトル</p>

    <table>
      <thead><tr><th rowspan="2">環境</th><th colspan="2">URL</th></tr></thead>

      <tbody><tr><td>dev</td><td>a</td><td>b</td></tr></tbody>
    </table>
    </div>
    ```

実際にレンダリングされたパターン集は [Admonitions × HTML テーブル](admonitions-html-table.md)、
検証手順・生成 HTML・判断根拠は [検証結果まとめ](findings.md) を参照してください。

## このサンプルの構成

| ファイル | 役割 |
| --- | --- |
| `mkdocs.yml` | MkDocs 設定。`plugins: - techdocs-core` のみを指定 |
| `catalog-info.yaml` | Backstage カタログ登録用（`backstage.io/techdocs-ref: dir:.`） |
| `docs/index.md` | このページ |
| `docs/admonitions-html-table.md` | 検証パターン集（実際にレンダリングされます） |
| `docs/findings.md` | 検証環境・手順・生成 HTML・所見 |

## ローカルでのビルド方法

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
