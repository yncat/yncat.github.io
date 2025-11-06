---
layout: post

tags: tips
---

Hiss の 64-bit バージョンを作る必要があり、内部で OpenSSL の static library 自前ビルドしたやつを使っているので、 64-bit バージョンの OpenSSL が必要になりました。

試したバージョンは OpenSSL 3.6.0。

[Strawberry Perl](https://strawberryperl.com/) をインストール。ビルドスクリプトで使っているらしい。

[nasm](https://www.nasm.us/pub/nasm/releasebuilds/3.01/win64/) をインストール。とりあえず書いている時点では以下のバージョン。

インストーラー版を使うと、自動で perl と nasm にはパスが通るはず。

x64 native tools command prompt を開く。これは visual studio community に付いてくる。

ソースディレクトリに移動して、以下のオマジナイを唱える。

`perl Configure VC-WIN64A`

これで設定が行われるので、 `nmake` でビルド。依存関係のビルドを省略することができるらしいが、なにがどう省略されるかはよくわからんのでとりあえず full build でいく。

あとはひたすらコンパイルだのなんだのが走っていくのを見守るだけです。簡単ですね。

