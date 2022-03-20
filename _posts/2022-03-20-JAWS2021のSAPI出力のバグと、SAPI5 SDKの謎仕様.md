---
layout: post
tags: tips
---

# HISSをデバッグしてたらJAWSのバグを見つけた

HISSのSAPI拡張をJAWS2021日本語版で使うと、音声速度の設定がなんかへんな挙動をすると言われました。私はJAWSを持ってないからデバッグできないよーんごめんにーとか言っていたのですが、さすがにずっとそんなことを言い続けているわけにもいかないので、SAPIの出力と各種イベントのパラメータを全部ダンプするロガーを入れてデバッグしました。そこで分かったことをメモしておきます。

# 前提知識

SAPI5のインターフェイスは、 [ISpVoice](https://documentation.help/SAPI-5/ISpVoice.htm) です。これを経由してSAPIを操作できます。たとえば、しゃべらせるときには [::Speak](https://documentation.help/SAPI-5/ISpVoice_Speak.htm) を使います。

SAPIの音声速度は、-10から10の範囲で指定することになっています。それ以上、またはそれ以下の値を指定したとき、SAPI5のレイヤーではその値を丸め込みませんが、範囲外をサポートするかどうかはエンジンに寄ります、たぶんそんなあたいはサポートしてないから丸められますよ、かってにしてね、という(投げやりなスタンスのようです)[https://documentation.help/SAPI-5/Engine_Characteristics.htm]。SAPIの音声速度を変える方法は2通りあります。

1つめは、 [::SetRate](https://documentation.help/SAPI-5/ISpVoice_SetRate.htm) メソッドを呼び出して設定値を与える方法です。これは、インスタンスにより保持される状態なので、これ以降の全ての音声発生に対して速度変更が作用します。

2つめは、speakメソッドでしゃべらせる内容にxmlタグを埋め込んで、 `<rate absspeed="-5">speak slowly</rate>` のようにマークアップした文字列を送りつける方法です。これは、タグで囲んだ範囲の中だけが速度変更の対象になります。

上記のどちらも設定されていないとき、SAPIの音声速度は、Windowsのコントロールパネルで設定した速度が使われます。

# JAWSのバグ

JAWSの音声をSAPI5出力に設定しているとき、以下の手順で再現します。

1. Windowsのコントロールパネルで、SAPIの音声速度を最高速度に設定する。
2. JAWSの音声出力の設定で、音声速度を最低速度に設定する。
3. JAWSを再起動する。

期待する結果: JAWSで設定した速度が優先されて、ゆっくりした音声が流れる。

実際の結果: JAWSが起動直後にSetRateを呼び忘れていると思われ、最高速度の音声が流れる。

# JAWSがrateタグをどうやって送っているか

どうやら、JAWSは、SAPI5の音声速度を、基本的にはSetRateメソッドの法を使って調整しているようです。ただし、rateタグのほうも一緒に送っていることを確認しました。たとえば、音声速度の設定値が5のとき、SetRateで5が設定されているけれども、毎回rateタグで5という値を送っているらしいです。

JAWSには、何かの条件を満たしたときだけ音声を遅くする、みたいな機能があるらしいです。これが発動したときは、SetRateは元の値から変化しないけれども、rateタグのほうが変更後の値として更新して送っているようです。たとえば、元々の設定でrate 5だったとして、何かの条件を満たしたときだけrateを3下げる設定にしていたときは、setRateの状態は5で、rateタグに2が送られてくるそうです。

ここらへんは、JAWSを持ってるactlabのメンバーに確認してもらいました。

# そもそもの謎仕様、実はrateタグには指定方法が2種類ある

[xmlの説明](https://documentation.help/SAPI-5/WP_XML_TTS_Tutorial.htm)に、こう書いてあります。

> The Rate tag has two attributes, Speed and AbsSpeed, one of which must be present. The value of both of these attributes should be an integer between negative ten and ten. Values outside of this range may be truncated by the engine (but are not truncated by SAPI). The AbsSpeed attribute controls the absolute rate of the voice, so a value of ten always corresponds to a value of ten, a value of five always corresponds to a value of five.

(中略)

> The Speed attribute controls the relative rate of the voice. The absolute value is found by adding each Speed to the current absolute value.

これはまあべつにいいのですが。問題は、これを受け取るほうのインターフェイスが謎だということです。

xmlは、SAPIのエンジンに渡される前にパースされて、いい感じのstructに詰めて送ってくれます。具体的には、 [SPVTEXTFRAG](https://documentation.help/SAPI-5/Struct_SPVTEXTFRAG.htm) で受けます。受けるんですけど、よーく見てください。

このstructの中に、

```
SPVSTATE              State;
```

とありますね。 [SPVSTATE](https://documentation.help/SAPI-5/Struct_SPVSTATE.htm) を見てみます。RateAdjというメンバーで取れそうですね。取れそうなんだけど、こういう記述があります。

> RateAdj
> The rate associated with this text. Set using the <Rate> XML tag. This value should be combined with the baseline rate (either the default, or a value set by ISpVoice::SetRate) to yield the final rate value.

……おわかりいただけただろうか。 speed プロパティの値なのか、 absspeed プロパティの値なのか、判断する方法がないのです。

baseline rate(ここではsetSpeedの値のことを言ってます)と組み合わせるべきと書いてあるんだから、speedの値じゃね？absspeedが来たら、speedに変換してるんじゃね？と思うかもしれませんが、さっきのJAWSの例を見ると、明らかにabsspeedのつもりで送られてきてますよね？しかも、speedプロパティが指定されていなかったら、この値は0になります。だから、単純にJAWSがそういう送り方をしてきているからabsspeedとして扱おう、ということはしちゃいけないのです。実際、pc-talkerのSAPI5出力からは、rateタグが送られてこないので、rateAdjは常に0を示すことが分かっています。どないしろっちゅうねん。指定方法が2つあるのに、受け側が1つしかない。あほか。おれ、なんか間違ってる？

# じゃあ他のSAPI音声エンジンはどうしてるのか

当然浮かんでくる疑問です。ということで、調べて見ました。なお、これは音声を耳で聞いて予測した挙動に過ぎないので、厳密にはちょっと違うかもしれません。

## やっぱりというか、speedとabsspeedは区別できてない

とりあえず以下をpythonで実行して見ます。

``` python
import win32com.client

engine = win32com.client.Dispatch("SAPI.SpVoice")
engine.Rate = 10
engine.Speak("これはテストです。最大速度だよ。")
engine.Speak("<rate absspeed=\"-10\">速度をマイナス10に設定して読み上げていますよ。テストテストテストテスト。</rate>")
engine.Speak("<rate speed=\"-10\">速度を10だけ減少させて読み上げていますよ。テストテストテストテスト。</rate>")
```

これをMicrosoft Harukaで実行して見ると、speedでもabsspeedでもまったく同じスピードで読んでいることが分かります。xml tutorialが言うとおりに考えるなら、absspeedなら実際の値は-10、speedなら0になるはずなんだから、その違いを聞き分けられないわけありませんよね？

じゃあ、両方あるときにどうやって使われてるかを見てみましょう。

## Microsoft Haruka

setSpeedで与えられたbaseline rateの値と、rateタグで指定された値を合算して、-10と+10の境界を丸め込んだものを、最終的な値としているようです。

``` python
import win32com.client

engine = win32com.client.Dispatch('SAPI.SpVoice')
engine.Rate = 10
engine.Speak("これが最大速度です。")
engine.Rate = -10
engine.Speak("これが最低速度です。")
engine.Rate = 0
engine.Speak(
    "<rate absspeed=\"10\">速度0、補正10。たぶん、合算ですね。実質は速度10、最大速度でしゃべってると思いますね。テストテストテストテスト。</rate>")
engine.Rate = 10
engine.Speak(
    "<rate absspeed=\"10\">速度10、補正10。実は10よりも早くできるらしい。限界突破じゃん。テストテストテストテスト。</rate>")
engine.Speak(
    "<rate absspeed=\"100\">速度10、補正100。境界を越えたら丸め込んでると思いますね。テストテストテストテスト。</rate>")
```

## みんな大好きDocumentTalker

DocumentTalkerも同じでした。やっぱり合算しとるやん。absspeedとは。

# っていうことはさあ、JAWSの「条件満たしたら遅くする」って…逆に作用してない？

私は持ってないから確かめられないんですけど。さっきデバッグログを見て教えてくれたメンバーのレポートを信じるなら、「あるときだけ遅くする」はずの機能は、今「あるときだけ早くする」機能になってると思うんですよ。だって、そうじゃないですか。速度5、補正2だったとして、JAWSは5から3を引いた値、2を使ってねという意味でabsspeedプロパティに入れてるかもしれないけど、実質はspeedプロパティ(相対)で解釈されるから、5に2を足しちゃって、7に名って読み上げられますよね？

# じゃあなんでHISSはおかしかったのか

最初に戻ると。速度をめっちゃ早くしても早くならない、という問い合わせだったんですが。

普通に合算はしてたんですけど、境界値を丸め込むの忘れてました。で、HISSのdllのほうは、境界を越えたのは丸め込みじゃなくてエラーで返して値を変えないってしてたので、そこらへんでバグってました。

absspeedが実質机上の存在なのはなんかもやっとしますが、まあそういうことなんだと思います。 combine and truncate。ここ、テストに出るからな。
