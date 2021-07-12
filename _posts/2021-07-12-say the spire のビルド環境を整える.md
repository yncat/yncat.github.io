---
layout: post
tags: java saythespire slaythespire
---

# ビルド環境の準備
バグ調査をしたかったので、自分でビルドできるようにしました。そのときのメモ。

## 下準備
jdk 8 を入れる。

maven を落としてきて、パスを通す。

## 依存ライブラリのビルド

tolk のビルド済みがあればいいのだけど、 appveyor による最後の自動ビルドが消えているので、自前でビルド環境を作る。以下のものが必要。

- pandoc
- visual studio
- Windows SDK

必要な物はこのなかの最小限の部分だろうけど、よくわからんので、まとめて全部入れちゃう。選定するだけ時間の無駄である。

tolk をビルドする。

talk を clone。

tolk のディレクトリにいるときに、

```
call vcvars64.bat
```

これで vs のコンソールが出るので、そしたら

```
nmake /nologo dist
```

コンソールがうまく出せなかったら、以下を path に追加 ( vs2019 community の場合)

```
C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build
```

途中、 win32/jni.h が見つからないと言われたら、ディレクトリをincludeパスに追加する。

```
C:\Program Files\Java\jdk1.8.0_291\include
C:\Program Files\Java\jdk1.8.0_291\include\win32
```

たぶんこのあたり。

7-zip が入ってないと、アーカイブを作る時にこけるかもしれないけど、まあそれはどっちでもいい。 dll さえ手に入ればいいので。

## 本体ビルド準備

ModTheSpire と BaseMod を Say the Spire のリポジトリに持ってくる。とりあえず、 lib に入れろと書いてあるので、 lib ディレクトリを作って入れてみる。 github からリリースを持って来てと書いてあるけど、最新は steam workshop にあるやつらしいので、今回は、 steam のディレクトリから拝借してきた。

steam のディレクトリから、 slay the spire の jar ファイルを持って来て、 lib にコピーする。 desktop-1.0.jar かな。

tolk をビルドしたときにできた tolk.jar も、とりあえず lib に入れてみる。

そしたら、 say the spire の説明に戻って、こうする。

```
mvn install:install-file -Dfile=C:/git/say-the-spire/lib/tolk.jar -DgroupId=com.davykager.tolk -DartifactId=Tolk -Dversion=unknown -Dpackaging=jar
```

tolk をビルドしたときにできた dist/x86 と dist/x64 のディレクトリを、 say the spire の src/main/resources/tolk というディレクトリを掘ってコピーする。 src/main/resources/tolk の中に x86 と x64 がある状態にする。

最後に、 mvn package でビルド!!!!!!!!!!! 全てがうまくいってれば、ビルドできるはず!!!!!!!!!!!信じろ!!!!!!!!!!!!!!!!!!!

# オレオレビルドしたやつを実行する
普通に、 steam workshop でダウンロードされた jar が入ってるところに上書きしたら動きました。

でも、肝心の調べたかった問題は、 mod のせいじゃなくて、ゲーム側の問題っぽかった。
