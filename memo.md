[toc]



## ブロック要素

### 段落と改行

段落は単に1行以上の連続したテキストです。マークダウンソースコードでは、段落は2つ以上の空白行で区切られています。



見出し

見出しは先頭行に1〜6のハッシュ(#)文字を使用。

```markdown
# 見出し1
## 見出し2
### 見出し3
```

### ブロッククォート

電子メールスタイル

> ブロッククォート
>
> 。

### リスト

* 赤
* 青

1. 赤
2. 青

### タスクリスト

- [ ] チェック1
- [ ] チェック2

### フェンス

```ruby
require 'redcarpet'
markdown = Redcarped.new("Hello World!")
puts markdown.to_html
```

