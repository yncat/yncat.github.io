---
layout: post
tags: msvc tips
---

# github actions でビルドしているときにはまった問題

cl でコンパイルして、最後に link でリンクして exe ファイルを作るんですけど、github actions で実行すると、こんなふうになりました。

`link: extra operand keiko.obj`

最初は、link の引数に互換性がないのかと思ったんですが、色々調べると、どうやら unix 計にある link っぽいものをたたいていそうでした。

環境変数をダンプして調べると、github actions に入っている git bash に、link を Windows 用にコンパイルした link.exe が含まれていて、それが優先して実行されていることが分かりました。

環境変数を手っ取り早く持ってくるために、 [ilammy/msvc-dev-cmd](https://github.com/ilammy/msvc-dev-cmd) を使ってるのですが、これで環境変数を prepend(先頭に挿入)した直後でも、workflow step の開始時にさらに上書きされてしまうらしいです。

上記の action の readme を見ると、これが起こるのは shell: bash の設定のときだけと書いてあるのですが、普通に cmd だろうが powershell だろうが起きています。

ということで、ちょっとお行儀が悪いですが、仮想環境なのでまあいいでしょうということで、消し去って差し上げました。コンパイルする前のステップに以下を追加です。

```

      - name: Die, link!!!!!
        run: rm /usr/bin/link.exe
        shell: bash

```

shell: bash を指定するのがポイントです。じゃないと、 /usr/bin が通らないです。べつに他のステップは bash 指定じゃなくてよいです。
