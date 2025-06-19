---
layout: post

tags: tips
---

会社で [Cline](https://github.com/cline/cline) を使い始めて、もう毎日 Cline 先生がいないと、ほかの仕事もある中コードが書けません。

でも、 Cline ってわりとスクリーンリーダーで読まないところがあります。しかも、ちょちょちょっと直したら治りそうな絶妙な読まなさ加減です。

これはもったいない、プルリクチャンスということで、今日はとりあえず環境構築するところまでを殴り書きにします。

とはいっても、 npm install して、そのディレクトリを Visual Studio Code で開いて、 F5 を押すだけです。

でも、ちょっとここで罠があります。

これはたぶん設定にもよるのですが、 vscode の default terminal が powershell になっているときに、実行ポリシーの関係でビルドスクリプトが拒否られて立ち上がらないことがあります。自分はそうでした。

解決方法としては、 default terminal を cmd にすればいいです。なのですが、全盲的に結構大変だったのが、じゃあどこから cmd にできるのかを探すところでして。

結論から書くと、コマンドパレット (ctrl+shift+p) から Terminal: Select default profile を実行して、 cmd を選べばいけます。普通にggると、ターミナルを開いてそこの上のほうにあるアイコンをクリックしてどうちゃらとか言われるのですが、我々からすると画面の部品が多すぎる上にタブでフォーカスしない項目もあるので、探すのが大変すぎました。なので、さっさとコマンドパレットからやってしまいましょう。

気が向いたら、なにをどう修正したかみたいな記事も書こうと思いますが、とりあえず眠くなってきたので今日はこれで終わります。

とりあえず今出してる PR だけ貼っておきます。それではおやすみなさい！

[Fix: Make some of the buttons on the task header accessible to screen readers #4246](https://github.com/shrijayan/TWCline-open-source/commit/484a19377393e5a89c277ffc982d7e2f60306e3e)

[Fix: Add role and aria-checked attributes to plan / act mode switch so that screen readers can tell its state ( Resolves #4244 ) #4263](https://github.com/cline/cline/pull/4263)
