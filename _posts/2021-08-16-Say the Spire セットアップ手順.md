---
layout: post
tags: saythespire slaythespire game

---

# Say the Spire を動かすまでの道のり
分かってればある程度は簡単なんですが、英語の説明しかないので、それを解説します。

# そもそも Say the Spire ってなんやねん

[Slay the Spire](https://store.steampowered.com/app/646570/Slay_the_Spire/?l=japanese) というゲームがあります。楽しいです。どんなゲームかは、公式サイトを見てください。

[Say the Spire](https://steamcommunity.com/sharedfiles/filedetails/?id=2239220106) は、 Say the Spire をスクリーンリーダーで読めるようにする追加のプログラムです。これをインストールすると、画面を見なくても、ゲームをプレイできるようになります。

# セットアップ手順

## 前提条件と、ゲームのインストール

前提として、 Say the Spire を入れるには、 Slay the Spire のゲーム本体を Steam で購入する必要があります。他のプラットフォームから購入しても使えません。さっき書いた Steam のサイトから買って入れてください。 Steam のセットアップ自体がなかなか難しいですが、それは今回は解説しないので、まぁ頑張ってください。

スクリーンリーダーは、 NVDA を使用してください。それ以外の話は知りません。

今のところ、ゲームコントローラーによる操作にのみ対応しています。お勧めは、 xbox のコントローラーです。

## Say the Spire を subscribe

さっき貼った Say the Spire のリンクを踏むと、 Steam Workshop というサイトに行きます。これは、 Steam のコミュニティーサイトで、ゲーム関連のファイルを共有できるところです。ここからコンテンツをもらうには、 subscribe という処理をする必要があります。

さっきのリンクで Say the Spire の詳細ページに飛ぶはずなので、 Steam アカウントにログインしている状態で Subscribe というリンクを押します。ページの下のほうにモーダルが出てきます。関係する mod を一緒に subscribe するかどうか利いてきてるので、 OK 的な感じにします。ここまでやると、 PC のなかで勝手に色々されて、必要なファイルが落ちてきます。

## 起動ファイルを落とす

[Say the Spireの起動ファイル](https://raw.githubusercontent.com/bradjrenshaw/say-the-spire/master/scripts/mts_steam.cmd) を落とします。そのまま押すとたぶんテキストが出てきちゃうので、右クリックメニューからの「リンク先をファイルに保存」的なやつで。

もし、 steam や、 Slay the Spire のインストール先ディレクトリを変更している場合、落とした cmd ファイルに書かれているパスをいじる必要があります。まぁ、わざわざインストール先を買えている人は、そのへんが分かってる人なんだろうから、よしなにしてください。普通はやらなくていいです。

## Java Access Bridge を有効にしておく

コマンドプロンプトで、

```
"C:\Program Files (x86)\Steam\steamapps\common\SlayTheSpire\jre\bin\jabswitch" -enable
```

というオマジナイを唱えてください。よく分からない人は、上の1行を bat という拡張子で保存して、それを実行してください。例によって Slay the Spire の場所を変えている人は、よしなにしてください。

## 落とした cmd ファイルから起動する

起動ファイルからゲームを立ち上げると、 Slay the Spire 本体が立ち上がる前に、 Mod the Spire という画面が出てきます。これは、 Slay the Spire の mod を管理するための存在です。

ゲームを初めて起動するときは、このがめんでタブキーを押していったところの "toggle all mods on / off" というようなボタンを1回押してから、 "play" ボタンを押します。

2回目以降、ゲームを起動するときは、ただ単に "play" ボタンを押します。

この画面を読まない場合、これより1個前のステップで、 Java Access Bridge の有効化が正常にできていない可能性があります。コマンドプロンプトでやっている場合、 "Java Access Bridge has been enabled." という出力が出ることを確認してください。

ここまでできれば、ゲームの画面をしゃべってくれるようになるはずです。

## 日本語環境で問題が起こるのを直す

そのままだと、日本語のカード説明を正しく読まないです。ダメージの数値を読まなかったりとか。

で、このバグは、中の人と協力して直したんですが、まだ正式に反映されていません。 [私がビルドしたやつを貼っておくので、しばらくはこれを使ってください](https://www.nyanchangames.com/yncat_github_io/sayTheSpire.jar) 。

もし、上のリンクがうまく動かなかったら、たぶんもう正式版になったということなので、そのままでいいです。

使い方ですが、普通に落として、元々のファイルに上書きするだけです。だいたいここにあります。

`C:\Program Files (x86)\Steam\steamapps\workshop\content\646570\2239220106\sayTheSpire.jar`

以上です。 enjoy.
