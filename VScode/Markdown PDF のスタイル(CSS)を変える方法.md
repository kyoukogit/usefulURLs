https://h-s-hige.hateblo.jp/entry/20190405/1554467885
https://mikansei-lab.net/markdown-pdf/
# Markdown PDFを入手

# Visual Studio Code の ワークスペース を作る
Visual Studio Code では、ファイルを開く ことで、ファイルを扱えます。
また、フォルダを開く ことで、複数のファイルを扱うこともできます。
ワークスペースでは、1つ以上のフォルダをまとめて扱うことができます。
ワークスペースは、フォルダを開いたあと メニューの File -> Save Workspace As... から作ることができます。

# ワークスペース の設定をする
ワークスペースを作ると、ワークスペースの設定ができるようになります。
設定は、 Settings (Ctrl+, ) 、または、ワークスペースとして保存した code-workspace ファイルを開くことでできます。
```
{
    "folders": [],
    "settings": {
        "markdown-pdf.styles": [
            "./markdown.css"
        ]
    }
}
```

# CSS を作る
```
img {
    width: 80%;
    margin-left: 10%;
}

h1 {
    padding-bottom: 0.3em;
    line-height: 1.2;
    border-bottom: 2px solid #2f51b4;
    position: relative;
    padding-left: 18px;
}

h1:before {
    background: #2f51b4;
    content: "";
    height: 28px;
    width: 8px;
    left: 0;
    position: absolute;
    top: 3px;
}

h2 {
    padding-bottom: 0.3em;
    line-height: 1.2;
    /* border-bottom: 1px solid #2f51b4; */
    position: relative;
    padding-left: 18px;
    /*margin-left: 16px;*/
}

h2:before {
    background: #2f51b4;
    content: "";
    height: 20px;
    width: 5px;
    left: 0px;
    position: absolute;
    top: 3px;
}

h3 {
    text-decoration: underline;
}
```

# PDF を出力する
あとは、Markdown PDF で PDF に変換するだけです。