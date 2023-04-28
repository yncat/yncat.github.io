---
layout: post
tags: tips
---

# 入れただけだと使い方が直感的にはわからないのでメモ

GitHub Copilotを使うと、コードを書いているときに、AIのペアプログラマーが手伝ってくれるようになります。私も、今日使い始めました。

[GitHub Copilotの公式ヘルプ](https://docs.github.com/ja/copilot/getting-started-with-github-copilot)

ただ、これをスクリーンリーダーで使おうとすると、使い方がよくわからんと思うので、メモります。

## セットアップするまで

まず普通にGitHub Copilotの拡張機能を入れます。ctrl+shift+xで拡張機能の画面を出して、 copilot とか入れて、タブを押して、それっぽいやつでエンターみたいな感じです。

インストールすると、GitHubにサインインしてアクセスを認可しろと言われます。これは通知画面に出るので、ちょっと見つけるのが面倒です。以下の手順で操作します。

- f6でステータスバーまで行く
- 上下矢印で通知ボタンまで行ってエンター
- 「GitHubにサインインしてください」みたいな通知を選んで、近くにあるサインインボタンを押す
- ブラウザが開くので、後は普通にアクセスを認可してあげる

## 実際に使うとき

たとえば、 公式の例にあるように、TypeScriptの関数を書かせてみます。

test.ts を作って、 function calculateDay とか入れます。

そうすると、たぶん今まで聞いたことがない音が鳴ります。コマンドパレット(ctrl+shift+p)から"help: List audio cues"で音を確認すると、 inline suggestion の音だとわかります。

この inline suggestion は、現在のところ自動では読んでくれません。なので、ここで提案の内容をすぐ確認することは不可能です。

代わりに、提案が出たところで ctrl+enter を押します。そうすると、copilotが提案した内容を1つずつプレビュー画面で見ることができます。たとえばこんな感じ。

```
Synthesizing 2/10 solutions

=======
Suggestion 1

function calculateDaysBetweenDates(startDate, endDate) {
    var start = Date.parse(startDate);
    var end = Date.parse(endDate);
    var millisecondsPerDay = 86400000;
    return (end - start) / millisecondsPerDay;
}

=======
Suggestion 2

function calculateDaysBetweenDates(start, end) {

```

これを読んで、よし採用だと思ったら、ctrl+slashを押します。そうすると、今カーソルがある提案がファイルに書かれます！

今回はなにも回りのコードがないので提案がしょぼいですが、きっとうまく使えばいい感じになることでしょう！

## 余談

以前まではctrl+slashで提案の採用ができなかったらしいです。丁度最近その機能が追加されてました。最高かよ。 [喜びのコメントをしておきました](https://github.com/orgs/community/discussions/7139) 。

