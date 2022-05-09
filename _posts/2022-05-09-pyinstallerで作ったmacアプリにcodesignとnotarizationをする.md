---
layout: post
tags: tips
---

# 前書き

大変だった記録を去年書き残していたようですが、なんか途中で力尽きてファイルがほっぽり投げられているのを見つけました。Screaming Strike 2 をバージョンアップしなければいけない事情が出てきたので、思い出すついでに体裁を整えてアップしようと思います。2022 年 5 月 9 日！

# 大変だった記録です

いやぁ、大変でした。ScreamingStrike2 に新しいモードを作ったので、それを新バージョンとしてリリースしたかったんですが、最新の Mac ではコード署名(code signing)と公証(notarization)をしないと起動しないらしくて、それを付ける作業がそれはもう大変でした。

もう１回野郎としたら絶対忘れるので、書き残しておきます。

xcode を使うとポチポチやるだけでできるらしいんですが、なんせ pyinstaller を使っているので、xcode プロジェクトなんてないから使えません。ということで、全部コマンドラインでやっていきます。

# 前提

使用したのは MacOS 11.6 です。xcode のバージョンは 13.1。terminal で作業するので、xcode commandline tool をインストールしておきます。やりかたは、なんかころころ変わりそうなので、ググってください。ググったら出ます。

OS は最新じゃないですが、最新にしてぶっ壊れたらいやなので、とりあえず消極的に 11.6 で止めておきます。俺はとりあえずリリースしたいんじゃ。あとのことはリリースしてから考えるんじゃ。

# step1: entitlements.plist を作る

entitlements.plist という xml みたいなファイルを作る必要があります。テキストファイルで書いてもいいと思うんですが、それだと、 entitlements.plist にヘッダー情報がねえ!!!!!!!!みたいに怒られます。ただのテキストファイルやんって感じなんですが、よくわからんので、xcode で生成するのが楽です。

プロジェクトを作らなくても、新規->entitlements みたいな感じでメニューをたどっていけば生成できます。entitlements.plist というファイル名にして、メインの python ファイルが入っているのと同じ場所に置いておきます。

で、中身をこんな感じにします。copyright とかは、xcode で生成した時点でよしなに入ってたような気もするし、自分で入れたような気もする。まぁ、そこは適当に。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<!--
   entitlements.plist


   Created by Yukio Nozawa on 2021/10/20.
   Copyright (c) 2021 Nyanchan Games. All rights reserved.
-->
<plist version="1.0">
  <dict>
    <key>com.apple.security.cs.allow-unsigned-executable-memory</key>
    <true/>
    <key>com.apple.security.cs.disable-library-validation</key>
    <true/>
  </dict>
</plist>
```

見ると分かりますが、この entitlements っていうファイルは、アプリのポリシーみたいなのを定義しているマニフェストファイルなんですね。

指定しているキーは２つあります。

com.apple.security.cs.allow-unsigned-executable-memory は、pyinstaller で固めたアプリを実行するときには指定する必要がある…とネットの誰かが言ってました。[公式の説明](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_security_cs_allow-unsigned-executable-memory)としては、書き込み可能/実行可能なメモリを確保することを許可するかどうか…といったところか。pyinstaller は、python の実行環境を展開した後にバイトコードを実行するので、そこらへんの bootstrap 処理で必要なんでしょう。

com.apple.security.cs.disable-library-validation は、外部ライブラリやフレームワークを読み込むときのコード署名チェックを緩和します。これも[公式の説明](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_security_cs_disable-library-validation)を見ると分かりますが、app bundle の中にあるライブラリが、サードパーティーなど、違う人によってすでにコード署名されている場合に使います。これがないと、python framework がすでに署名されていて、app bundle の中にある実行ファイルは私の証明書で署名されているので、そこが合わなくてアプリが起動しません。そこまでこまかく署名チェックするなんて、りんごさんはきびしいですねえ。

# step2: app bundle を作る

python で作ったすげえやべえアプリがあるとします。これを pyinstaller で app bundle というものに変えます。

app bundle というのは、実態は SugeeYabee.app というような、アプリ名の後ろに「.app」がついたディレクトリです。mac のアプリは、みんなこの app bundle になっています。そして、この app bundle の中にアプリのファイルは全部置かれるので、アプリをコピペで移動したりしても問題ないし、アプリを消せばゴミが残らずに消えるようになってるわけですね。

最新の pyinstaller のバージョンは、２０２１年の１１月ぐらいに見た時は 4.7 だったんですが、4.7 だと、できた app bundle をコード署名するときに invalid app bundle みたいなエラーが出て署名できませんでした。もともと入ってたのが 3.5 だったので、ちょっと古くて心配ですが、とりあえず 3.5 を使っています。脆弱性があるらしいので、できるだけバージョンを上げていきたいですが...

`pip3 install pyinstaller==3.5`

これで 3.5 をインストールします。

そしたら、

```
pyinstaller --windowed --onefile --osx-bundle-identifier "アプリの識別子" エントリポイントのファイル
```

こんな感じのコマンドで app bundle にします。

--windowed は、ウィンドウが表示されることを示します。コマンドラインアプリの場合は必要ないです。
--onefile は、１個のファイルにまとめることを示します。Windows だと、ライブラリファイルとかが散らばらずに巨大な exe ファイルが１個生成されるようになるんですが、mac の場合はこれを外すとどうなるのかよく分かりません。ちなみに、Windows で--onefile オプションを使うと、なんか起動が遅くなるような気がするので、screaming strike2 の Windows 版では使っていません。

アプリの識別子の部分は、自分で識別できればいいので、好きな名前を指定していいです。ただし、英数で。慣習的には、 com.nyanchanGames.screamingStrike2 みたいに、 com.組織.アプリ名 というお作法らしいです。

エントリポイントのファイルは、アプリを実行するときに python3 に渡すメインのファイルを指定します。たとえば、ローカルでアプリをソースから起動するときに python3 hogehoge.py とやって実行しているなら、 hogehoge.py です。

# ここでおわってるぅぅぅぅ

ここまでは覚えてるんじゃあ。このあとがいっちゃん重要なのにかいてないんじゃあ。ぼけーわれー。2022 年ー。なので頑張って記憶で書きます。バージョンアップするときに違ってたら直します。

# ここからさきはお布施だヨーン

ここから先を行うには、Apple Developer への登録が必要です。無料じゃだめです。有料のメンバーシップに加入している必要があります。じゃないと証明書を作れません。金でゲス。

Apple Developer のページで開発用の証明書を作って、Mac のキーチェーンに登録しておきます。やり方はググってください。ここでよく事故るので、頑張ってください。私も盛大に事故りましたが、なんか証明書を入れたり消したりごちゃごちゃやっていたらいつの間にか治ってたのでもう全部忘れました。二度と環境再構築とかしたくありません。きっと、将来、Mac を買い換えた私が期待を込めてこのブログを見たとき、この段落を読んで PC ごとたたき壊したくなるに違いない。でも、しょうがないんや。マジでわかんないんや。

# step3: 実行可能ファイルとライブラリにコード署名を付ける

最新の Mac の app bundle が動くためには、メインの実行ファイルと、その実行ファイルから読み込まれる全てのライブラリ(so とか、dylib とか)が署名されている必要があります。なので、自分で配布するアプリの中にあるファイルの内、実行可能なコードを含むファイルを洗い出して、それらを自分の証明書を使って署名する必要があります。署名するには、以下のコマンドをファイルごとに打ちます。

`codesign -s 証明書の名前 --timestamp -o runtime 対象ファイル名`

証明書の名前は、キーチェーンから探すときに使われる名前らしいです。普通は、自分の名前を英語で入れたらいいはずです。私は Yukio Nozawa で通りました。ダブルクオートがなくてもいけました。

Screaming Strike の場合は、メインの実行ファイルはこの段階で署名していないようです。なので、ライブラリだけやればいいのかもしれません。よく分かりません。

# step4: app bundle にコード署名を付ける

微妙にコマンドが違います。 -f は必要かどうか分かりませんが、とりあえず Screaming Strike ではこれで動いたのでそういうものだと思っています。

`codesign -f -s 証明書の名前 -v --deep --timestamp --entitlements entitlements.plist -o runtime app bundleのパス`

deep オプションで再帰的に署名している感じがするのですが、ライブラリは別で署名しておかないと、このあとの notarization のステップで怒られます。

entitlements.plist、ここで出てきます。

app bundle のパスは、たとえば screamingstrike.app 見たいに、実際に app bundle がある場所を入れます。

# step5: zip ファイルを作る

コード署名によって、app bundle の中身を改ざんしようとする行為を検知できるようになりました。アレな dynamic link library を注入されてぴえんとなったり、実行ファイルの機械語部分にパッチを当てられてぱおんとなったりしないということですね。

だがしかし、Apple はこれでは満足しません。なんと、今度はアプリ自体が安全かどうかを見せろと要求してきます。ということで、俺のアプリは安全だし、ちゃんと署名してるんだからさっさと配らせろ、と言うために、Apple にアプリを送りつける必要があります。これが Notarization です。

Apple にアプリを送りつけるには、まずは app bundle を zip に圧縮しなくてはなりません。以下のコマンドで zip を作ります。

`ditto -c -k -rsrc --sequesterRsrc --keepParent app bundleのパス zipファイルのパス`

zip ファイルのパスには、ちゃんと拡張子まで入れてくださいね。

# step6: zip を Apple に送りつける

zip ができたら、以下のコマンドで Apple に送りつけます。

`xcrun altool --notarize-app -t osx -f zipファイルのパス --primary-bundle-id "アプリの識別子" --asc-provider "プロバイダのshort name" -u "apple IDのメアド" -p "アプリ用パスワード"`

プロバイダとは、複数の開発者ロールを持っている？正式な言い方は分かりませんが、まあとにかく権限が複数あるときに必要です。最初はこのオプションなくてもいいと思います。怒られたら、 `xcrun altool --list-providers ` とかで一覧が見えると思うので、自分のアカウントの short name を入れればいいです。

apple ID のメアドは、開発者登録の時に使ったメアドです。

アプリ用パスワードは、Apple Developer dashboard から生成できる App Specific Password というやつを使う必要があります。なので、まずは生成してきてから、それを使ってください。Apple ID のパスワードでは認証できません。そして、App Specific Password じゃないと認証できないよ、みたいな親切な説明は一切ありません！

これがうまくいくと、アップロードしたぜーみたいなメッセージが出ます。で、しばらく待つと、Apple ID のメアドにメールが送られてきます。このメールに、「Your app has been notarized」みたいなことが書いてあれば成功です。失敗していた場合、メールから詳細に飛んで理由を確認できます。だいたいは、中身のファイルの内、署名できてないものがあるとかです。

ここまでできたら、zip は消していいです。

# step7: Notarization した記録をくっつける

正式には staple と言います。notarization が完了すると、このアプリはちゃんと安全性を確認したぞ、ヨシ！という記録が Apple のサーバーに残ります。この「ヨシ！」の記録を、ホッチキスで留めるみたいに app bundle の中にくっつけることができます。くっつけなくてもいいのですが、くっつけないとアプリを起動するたびに Apple のサーバーに「ねえねえこれ安全性確認してあんの？」って問い合わせに行ってしまうので、起動するのにちょっとばかし時間がかかってしまいます。そんなに難しくないのでちゃっちゃと留めちゃいましょう。

こんな感じのオマジナイを唱えます。

`xcrun stapler staple app bundleのパス`

なんかやたら簡単ですね。これでくっつきます。

# step8: DMG を作る

Mac を使ってる人ならおなじみの DMG。ディスクイメージですね。やっとインストールできる。もうコマンドを打つのやになってきたので、秒で終わらせましょう。これです。

`hdiutil create -volname イメージの名前 -srcfolder app bundleが入ってるディレクトリ -ov -format UDZO DMGのパス`

最初のイメージの名前には拡張子がいらなくて、DMG のパスには拡張子がいります。

イメージの中に app bundle が収まらなきゃいけないので、app bundle の本体ではなくて、app bundle が入ってるディレクトリ(dist/screamingstrike.app だったら、dist)を指定する必要があります。で、たぶん app bundle 以外のゴミが入ってると一緒にイメージに固められてしまうので、事前に app bundle 以外の不要物は消しておきましょう。

よっしゃこれをインターネッツにアップロードしてワールドワイドなエブリワンにダウンロードしてもらってエグゼキュートさせると………怒られます！！！！！！！！！！！！！！！！！！！！！！！！！なんやねん！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！

# step9: DMG にも署名を付ける

さっきのをそのまま開いてインストールしようとすると、なんかやばいのが入ってるかもしれないからインストールさせませんみたいになって、インストールボタンが出現しません。なので、もう 1 回！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！Apple に送りつける必要があります。

DMG にもコード署名を付けろと言うことになっています。べつにアプリがコード署名されてるんだからいいと思うんですが。

`codesign -s 証明書の名前 --timestamp DMGのパス`

とはいえ、ずいぶんとオプションが減りました。

# step10: Apple に送りつける！

codesign させたんだから、分かるでしょ？はい、送りつけて notarization します。

`xcrun altool --notarize-app -t osx -f DMGのパス --primary-bundle-id "アプリの識別子" --asc-provider "プロバイダのshort name" -u "Apple IDのメアド" -p "アプリ用パスワード"`

またメールが来ます。これもちゃんと notarize できている必要があります。

# step11: もう 1 回 staple

notarize させたんだから、分かるでしょ？今度こそ最後です。

`xcrun stapler staple DMGのパス`

# これでようやく終わりです

やっと、やっとやっとやっと、DMG を配れます。

やばくない？

xcode だとこれを自動でやるから問題ないとかって Apple は思ってるのかもしれないけどね。知らんがな。

ということで、Screaming Strike 2 の Mac 版は、ちゃんとお金を払って、正式な手順で公開しているので、気が向いたらねぎらってください。焼き肉とか食べたいです。w
