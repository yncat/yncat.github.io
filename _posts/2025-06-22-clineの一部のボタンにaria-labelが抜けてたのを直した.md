---
layout: post

tags: a11y
---

この間出した cline の pull request の話です。

[Fix: Make some of the buttons on the task header accessible to screen readers #4246](https://github.com/shrijayan/TWCline-open-source/commit/484a19377393e5a89c277ffc982d7e2f60306e3e)

タスクの詳細が表示されている画面で、タスクを終了してトップ画面に戻るボタンと、タスクを削除して忘れるボタンが「icon button」と読まれてなにがなんだかわかんねえという致命的な問題に対処するものです。特に、私は最初、どうやったらタスク開いた状態から前の画面に戻れるのかぜんぜんわからなかったので、画面を見てくれる人がいない場合はこれはなかなか初見殺しになってしまうでしょう。

まぁ、変更内容としては、ただ単に aria-label を足しただけですね。

どうやら、 vscode 側が、ある程度の UI 要素は提供してくれているようで、 Cline もそれを使って要るっぽいです。そのコンポーネント自体は aria-label を取れるようになっていて、 Cline もある程度は入れてくれているみたいなのですが、ところどころ忘れちゃうんですね。

Claude Code にざっくり調査してもらった限りでは、あと10か所ぐらいあるらしいのですが、ちょっと自分がそのボタンがどこに表示されるかわかっていなくて、動作確認することができないので、まだほかの個所の aria-label を埋める対応はしていません。こうご期待。

