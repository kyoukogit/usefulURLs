https://b1tblog.com/2019/10/08/vscode-markdown-toc/

# 前提
Markdown Preview Enhancedという拡張機能を用います。

# 目次（TOC）の挿入方法
コマンドパレット（F1 または Ctrl + Shift + P）を開き、「Markdown Preview Enhanced: Create TOC」を実行します。

実行すると次のようなコードが追加されます。

# 対象外の設定
目次に追加したくない項目は次のように {ignore=true} を設定します。
```
## 目次 {ignore=true}
```
# 目次範囲の設定
目次に追加するタグの範囲は、Markdown Preview Enhanced: Create TOC によって追加されたコードのdepthFrom、depthToで設定します。

上のサンプルでは、h2タグとh3タグを目次に追加するように設定しています。

```
<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=3 orderedList=false} -->
```

# 項番の設定
上のサンプルではただリストですが、項番（順序あり）リストに変更することが可能です。

Markdown Preview Enhanced: Create TOC によって追加されたコードのorderdListをtrue設定します。

```
<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=3 orderedList=true} -->
```
