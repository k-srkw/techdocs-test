# 検証結果まとめ

## 検証環境

| 項目 | 値 |
| --- | --- |
| Python | 3.11.15 |
| mkdocs | 1.6.1 |
| mkdocs-techdocs-core | 1.7.1 |
| mkdocs-material | 9.7.7 |
| Markdown | 3.10.3 |
| pymdown-extensions | 11.0.1 |
| 検証日 | 2026-09-02 |

## 検証手順

```bash
python3 -m venv venv
./venv/bin/pip install mkdocs-techdocs-core
./venv/bin/mkdocs build --strict
# site/admonitions-html-table/index.html を BeautifulSoup で解析し、
# 各 admonition の中に <table> が入っているかを確認
```

## 前提: techdocs-core が有効にしている拡張

`mkdocs-techdocs-core` は `markdown_extensions` に以下を自動追加します
（`mkdocs.yml` には `plugins: - techdocs-core` と書くだけ）。

- `admonition` … `!!! note` 記法
- `pymdownx.details` … `??? note` 記法（折りたたみ）
- `pymdownx.extra` … この中に **`markdown.extensions.md_in_html`** と
  `markdown.extensions.tables`、`markdown.extensions.attr_list` が含まれる
- `pymdownx.superfences` / `pymdownx.highlight`（`linenums: true`）ほか

つまり **`md_in_html` は追加設定なしで使える** 状態です。

## 結果一覧

| # | パターン | 結果 | 備考 |
| --- | --- | --- | --- |
| 1 | `!!! note` + 4 スペースインデントの `<table>` | **OK** | 枠内に描画される |
| 2 | 同上 + `colspan` / `rowspan` | **OK** | 属性はそのまま出力に残る |
| 3 | セル内の Markdown | **OK（注意）** | `markdown="1"` なしでも変換される |
| 4 | `??? warning`（折りたたみ）+ `<table>` | **OK** | `<details>` 内に描画される |
| 5 | ネストした Admonition（8 スペース）+ `<table>` | **OK** | 内側の `div.admonition` 内に描画 |
| 6 | `<table>` の途中に空行 | **NG** | テーブルが分断されて壊れる |
| 7 | Markdown テーブル記法 | **OK** | セル結合以外はこちらが安全 |
| 8 | `<div class="admonition" markdown="block">` | **OK（推奨）** | 空行・Markdown 混在すべて OK |
| 9 | インデントなしの `<table>` | **NG** | Admonition の外に出る |

## 詳細な所見

### 1. インデント形式の Admonition 内では `<table>` は「インライン HTML」になる

これが今回いちばん重要な発見です。生成された HTML はこうなります。

```html
<div class="admonition note">
<p class="admonition-title">サポート対象バージョン</p>
<p><table>
  <thead>...</thead>
  <tbody>...</tbody>
</table></p>
</div>
```

`<table>` が `<p>` に包まれています。HTML としては不正な入れ子ですが、
ブラウザ（および DOMParser ベースのサニタイザ）が `<p>` を自動で閉じるため、
**表示上は問題なく枠内のテーブルとして描画されます**。

なぜこうなるかというと、Python-Markdown の HTML ブロック処理（`md_in_html` の
`HTMLExtractor` プリプロセッサ）は 4 スペースインデントされた行を HTML ブロックとして
拾わないためです。結果として `<table>…</table>` は Admonition 本文の中の
「素のテキスト」として残り、インライン段階で生 HTML として通過します。

この性質から次の 2 点が導かれます。

### 2. `markdown="1"` は不要で、書くと属性がゴミとして残る

インライン扱いなので、セル内のテキストは通常どおり Markdown のインライン処理を受けます。

```markdown
!!! tip

    <table>
      <tbody><tr><td>`app.baseUrl`</td><td>**必須**</td></tr></tbody>
    </table>
```

出力:

```html
<td><code>app.baseUrl</code></td><td><strong>必須</strong></td>
```

一方 `markdown="1"` を付けると `md_in_html` は発動せず、属性だけが残ります。

```html
<table markdown="1">
  <tbody><tr><td markdown="1"><code>code</code></td></tr></tbody>
</table>
```

<div class="admonition warning" markdown="block">
<p class="admonition-title">副作用</p>

セル内で `*` `_` `` ` `` を **文字として** 見せたい場合はエスケープが必要です。
`&#42;` `&#95;` `&#96;` などの実体参照を使ってください。
</div>

### 3. `<table>` の途中の空行はテーブルを壊す

インライン HTML は段落の一部なので、空行が来ると段落が切れます。

```markdown
!!! example

    <table>
      <thead><tr><th>手順</th></tr></thead>

      <tbody><tr><td>ビルド</td></tr></tbody>
    </table>
```

出力（`<tbody>` が別段落に切り離される）:

```html
<div class="admonition example">
<p class="admonition-title">...</p>
<p><table><thead>...</thead></table></p>
<p><tbody><tr><td>ビルド</td></tr></tbody></p>
</div>
```

ブラウザは孤立した `<tbody>` タグを捨てるため、行がテーブル外の素のテキストとして
表示されます。**インデント形式では HTML テーブル内に空行を入れないでください。**

### 4. 複雑なテーブルは div 形式 + `markdown="block"` が確実

```markdown
<div class="admonition success" markdown="block">
<p class="admonition-title">タイトル</p>

**Markdown** も効きます。

<table>
  <thead><tr><th rowspan="2">手順</th><th colspan="2">コマンド</th></tr></thead>

  <tbody><tr><td>ビルド</td><td>a</td><td>b</td></tr></tbody>
</table>
</div>
```

こちらはインデントされていないため `md_in_html` が正しく HTML ブロックとして扱い、
`<p>` に包まれず、空行を挟んでも 1 つの `<table>` として出力されます。
Markdown のテーブル記法との混在も可能です。

`markdown="1"` でも概ね動きますが、**生 HTML ブロックの後ろに置いた Markdown 段落の
インライン処理が行われない** ケースを確認したため、`markdown="block"` を推奨します。
折りたたみは `<details class="warning" markdown="block">` + `<summary>` で同じ形にできます。

### 5. （副次的な発見）日本語文中の `**強調**` が効かないことがある

`techdocs-core` は `pymdownx.betterem` を `smart_enable: all` で有効にしています。
スマート強調は開始の `**` の直前・終了の `**` の直後が「単語構成文字でないこと」を要求するため、
**日本語のように単語間に空白がない文では強調が解除されずリテラルの `**` が残ります**。

```markdown
本文を **4 スペースでインデント**するだけで   <!-- NG: 閉じ ** の直後が「す」 -->
本文を **4 スペースでインデント** するだけで  <!-- OK: 空白を入れる -->
テーブルを**枠内**に置く                      <!-- NG: 両側とも日本語 -->
テーブルを <strong>枠内</strong> に置く        <!-- OK: HTML タグを使う -->
```

`。` `、` `（` などの句読点が隣接する場合は問題ありません。
Admonition の内外を問わず起きる挙動なので、日本語の TechDocs では
**強調の前後に半角スペースを入れる** か `<strong>` タグを使ってください。

## 推奨ガイドライン

<div class="admonition tip" markdown="block">
<p class="admonition-title">使い分け</p>

| やりたいこと | 推奨する書き方 |
| --- | --- |
| セル結合なしの単純な表 | Admonition + **Markdown テーブル記法** |
| `colspan` / `rowspan` があり、HTML が短い | Admonition + 4 スペースインデントの `<table>`（**空行なし**） |
| HTML が長い・整形ツールが入る・Markdown を混ぜたい | **`<div class="admonition …" markdown="block">`** |

</div>

## 未検証の項目

<div class="admonition question" markdown="block">
<p class="admonition-title">Backstage 実機での確認は別途必要</p>

本検証は **MkDocs のビルド出力（HTML 生成）まで** を対象としています。
Backstage の TechDocs リーダーは、生成された HTML を DOMPurify ベースのサニタイザに
通してから表示します。テーブル系のタグ・属性は一般に許可リストに含まれますが、
`style` 属性やインライン CSS などは環境によって落ちる可能性があります。
実際の Backstage インスタンスでの表示確認をあわせて行ってください。

また `techdocs-core` の `use_pymdownx_blocks: true`（新しい `/// note` 記法）を
有効にした場合の挙動は本検証の対象外です。
</div>
