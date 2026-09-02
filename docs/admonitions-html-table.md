# Admonitions × HTML テーブル 検証パターン集

このページは実際に `mkdocs-techdocs-core` でレンダリングされます。
各パターンの「表示結果」が期待どおりかを目で確認できます。

判定の凡例: **OK** = 意図どおり描画される / **注意** = 描画はされるが副作用あり / **NG** = 壊れる

---

## パターン 1 (OK): 基本形 — `!!! note` + 生の HTML テーブル

Admonition の本文を **4 スペースでインデント** するだけで、HTML テーブルが枠の中に描画されます。

### 表示結果

!!! note "サポート対象バージョン"

    <table>
      <thead>
        <tr><th>コンポーネント</th><th>バージョン</th><th>状態</th></tr>
      </thead>
      <tbody>
        <tr><td>Backstage</td><td>1.30 以降</td><td>サポート</td></tr>
        <tr><td>mkdocs-techdocs-core</td><td>1.7.x</td><td>サポート</td></tr>
        <tr><td>Node.js</td><td>18</td><td>非推奨</td></tr>
      </tbody>
    </table>

### ソース

```markdown
!!! note "サポート対象バージョン"

    <table>
      <thead>
        <tr><th>コンポーネント</th><th>バージョン</th><th>状態</th></tr>
      </thead>
      <tbody>
        <tr><td>Backstage</td><td>1.30 以降</td><td>サポート</td></tr>
      </tbody>
    </table>
```

!!! abstract "生成される HTML"

    実は `<table>` は「HTML ブロック」ではなく **段落内のインライン HTML** として扱われ、
    `<p><table>…</table></p>` という形で出力されます。
    ブラウザ側で `<p>` が自動的に閉じられるため見た目は正常ですが、
    この性質がパターン 3・6 の挙動につながります。

---

## パターン 2 (OK): `colspan` / `rowspan` — Markdown 記法では書けない表現

HTML を使う最大の動機がこれです。Markdown のテーブル記法ではセル結合ができません。

### 表示結果

!!! info "環境別のエンドポイント"

    <table>
      <thead>
        <tr>
          <th rowspan="2">環境</th>
          <th colspan="2">エンドポイント</th>
        </tr>
        <tr>
          <th>API</th>
          <th>TechDocs</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>dev</td>
          <td><code>https://api.dev.example.com</code></td>
          <td><code>https://docs.dev.example.com</code></td>
        </tr>
        <tr>
          <td>prod</td>
          <td><code>https://api.example.com</code></td>
          <td><code>https://docs.example.com</code></td>
        </tr>
      </tbody>
    </table>

`rowspan="2"` / `colspan="2"` はビルド後の HTML にもそのまま残ります。

### ソース

```markdown
!!! info "環境別のエンドポイント"

    <table>
      <thead>
        <tr>
          <th rowspan="2">環境</th>
          <th colspan="2">エンドポイント</th>
        </tr>
        <tr><th>API</th><th>TechDocs</th></tr>
      </thead>
      <tbody>
        <tr>
          <td>dev</td>
          <td><code>https://api.dev.example.com</code></td>
          <td><code>https://docs.dev.example.com</code></td>
        </tr>
      </tbody>
    </table>
```

---

## パターン 3 (注意): セル内の Markdown — `markdown="1"` は不要

インデント形式の Admonition の中では、`<td>` の中身は **放っておいても Markdown として解釈されます**。
`md_in_html` の `markdown="1"` を書く必要はなく、書くと **属性が出力 HTML にそのまま残ってしまいます**。

### 表示結果（`markdown="1"` なし）

!!! tip "設定値の一覧"

    <table>
      <thead>
        <tr><th>キー</th><th>説明</th><th>既定値</th></tr>
      </thead>
      <tbody>
        <tr>
          <td>`app.baseUrl`</td>
          <td>**必須**。Backstage の公開 URL</td>
          <td>*なし*</td>
        </tr>
        <tr>
          <td>`techdocs.builder`</td>
          <td>`local` または `external`</td>
          <td>`local`</td>
        </tr>
      </tbody>
    </table>

`markdown="1"` を一切書いていませんが、バッククォートは `<code>`、`**…**` は `<strong>` に変換されています。

!!! warning "裏返すと、意図しない変換が起きます"

    セル内にアスタリスクやアンダースコア、バッククォートを **文字として** 表示したい場合は、
    `&#42;` `&#95;` `&#96;` のように HTML 実体参照でエスケープしてください。

### ソース

```markdown
!!! tip "設定値の一覧"

    <table>
      <tbody>
        <tr>
          <td>`app.baseUrl`</td>
          <td>**必須**。Backstage の公開 URL</td>
        </tr>
      </tbody>
    </table>
```

---

## パターン 4 (OK): 折りたたみ Admonition（`???`）の中の HTML テーブル

`pymdownx.details` による折りたたみブロックでも同様に動作します。

### 表示結果

??? warning "廃止予定の API（クリックで展開）"

    <table>
      <thead>
        <tr><th>API</th><th>廃止バージョン</th><th>代替</th></tr>
      </thead>
      <tbody>
        <tr><td><code>/v1/docs</code></td><td>1.25</td><td><code>/v2/docs</code></td></tr>
        <tr><td><code>/v1/search</code></td><td>1.28</td><td><code>/v2/search</code></td></tr>
      </tbody>
    </table>

### ソース

```markdown
??? warning "廃止予定の API（クリックで展開）"

    <table>
      <thead>
        <tr><th>API</th><th>廃止バージョン</th><th>代替</th></tr>
      </thead>
      <tbody>
        <tr><td><code>/v1/docs</code></td><td>1.25</td><td><code>/v2/docs</code></td></tr>
      </tbody>
    </table>
```

---

## パターン 5 (OK): ネストした Admonition の中の HTML テーブル

ネスト 1 段ごとに 4 スペース追加インデントします（内側は 8 スペース）。

### 表示結果

!!! danger "移行時の注意"

    移行前に以下を必ず確認してください。

    !!! note "確認項目"

        <table>
          <tbody>
            <tr><td>1</td><td>バックアップ取得</td></tr>
            <tr><td>2</td><td>設定ファイルの差分確認</td></tr>
          </tbody>
        </table>

### ソース

```markdown
!!! danger "移行時の注意"

    移行前に以下を必ず確認してください。

    !!! note "確認項目"

        <table>
          <tbody>
            <tr><td>1</td><td>バックアップ取得</td></tr>
          </tbody>
        </table>
```

---

## パターン 6 (NG): HTML テーブルの途中に空行を入れる

インデント形式の Admonition では `<table>` が段落内インライン HTML として扱われるため、
**途中の空行が段落を分断し、テーブルが壊れます**。

### 表示結果（意図的な NG 例）

!!! example "空行を挟んだ HTML テーブル（壊れます）"

    <table>
      <thead>
        <tr><th>手順</th><th>コマンド</th></tr>
      </thead>

      <tbody>
        <tr><td>ビルド</td><td><code>mkdocs build</code></td></tr>
        <tr><td>プレビュー</td><td><code>mkdocs serve</code></td></tr>
      </tbody>
    </table>

上のヘッダー行だけがテーブルとして描画され、`<tbody>` 側は別の段落に切り離されて
表の体裁を失っています（生成 HTML は `<p><table><thead>…</thead></table></p><p><tbody>…</tbody></p>`）。

### 対処

**HTML テーブルの中には空行を入れない。** 空行を入れたい／入ってしまう場合はパターン 8 を使ってください。

---

## パターン 7 (比較): Markdown テーブル記法

セル結合が不要なら、Markdown のテーブル記法のほうが簡潔で安全です。

### 表示結果

!!! quote "記法の比較"

    | 記法 | セル結合 | 途中の空行 | 記述量 |
    | --- | --- | --- | --- |
    | Markdown テーブル | 不可 | 影響なし | 少ない |
    | HTML テーブル（インデント形式） | 可 | 壊れる | 多い |
    | HTML テーブル（div 形式・パターン 8） | 可 | 影響なし | 多い |

---

## パターン 8 (推奨・OK): `<div class="admonition">` + `markdown="block"`

複雑な HTML テーブルを入れたい場合は、Admonition 自体を HTML で書き、
`markdown="block"` を付けるのが最も安全です。
`md_in_html` が働くので、**インデント不要・空行 OK・Markdown と HTML の混在 OK** になります。

### 表示結果

<div class="admonition success" markdown="block">
<p class="admonition-title">div 形式なら空行を挟んでも壊れません</p>

**Markdown の装飾** も `インラインコード` も [リンク](https://backstage.io/docs/features/techdocs/) も効きます。

<table>
  <thead>
    <tr><th rowspan="2">手順</th><th colspan="2">コマンド</th></tr>
    <tr><th>ローカル</th><th>CI</th></tr>
  </thead>

  <tbody>
    <tr><td>ビルド</td><td><code>mkdocs build</code></td><td><code>techdocs-cli generate</code></td></tr>
    <tr><td>公開</td><td>—</td><td><code>techdocs-cli publish</code></td></tr>
  </tbody>
</table>

同じ枠の中に Markdown のテーブルも置けます。

| A | B |
| --- | --- |
| 1 | 2 |
</div>

### ソース

```markdown
<div class="admonition success" markdown="block">
<p class="admonition-title">div 形式なら空行を挟んでも壊れません</p>

**Markdown の装飾** も `インラインコード` も効きます。

<table>
  <thead>
    <tr><th rowspan="2">手順</th><th colspan="2">コマンド</th></tr>
    <tr><th>ローカル</th><th>CI</th></tr>
  </thead>

  <tbody>
    <tr><td>ビルド</td><td><code>mkdocs build</code></td><td><code>techdocs-cli generate</code></td></tr>
  </tbody>
</table>
</div>
```

!!! note "block 指定と 1 指定の違い"

    `markdown="1"` でも動きますが、**生の HTML ブロックの後ろに置いた Markdown が
    処理されなくなる** ケースを確認しました。
    div 形式では `markdown="block"` を明示するのが確実です。

    折りたたみにしたい場合は `<details class="warning" markdown="block">` +
    `<summary>` で同じことができます。

---

## NG パターン: インデント不足

Admonition の本文は 4 スペースインデントが必須です。
インデントを付けないと、テーブルは Admonition の **外** に描画されます。

### 表示結果（意図的な NG 例）

!!! failure "この枠の中にテーブルは入りません"

    枠内のテキストはここまで。以下の HTML はインデントしていないため枠外に出ます。

<table>
  <tbody>
    <tr><td>枠の外にはみ出したテーブル</td></tr>
  </tbody>
</table>

### ソース

```markdown
!!! failure "この枠の中にテーブルは入りません"

    枠内のテキストはここまで。

<table>
  <tbody>
    <tr><td>枠の外にはみ出したテーブル</td></tr>
  </tbody>
</table>
```

!!! bug "よくある落とし穴"

    エディタの自動整形や Prettier が HTML ブロックのインデントを削ってしまうと、
    このパターンに陥ります。整形後は必ずビルド結果を確認してください。
