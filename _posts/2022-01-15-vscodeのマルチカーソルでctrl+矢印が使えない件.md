---
layout: post
tags: vscode tips
---

# キー割り当ての条件をいじるひつようがあります

スクリーンリーダーを使っているとき、複数のカーソルを置いている状態でctrl+右矢印などを押すと、全てのセカンダリーカーソルが消えてしまいます。これは今は仕様です。スクリーンリーダーでも使えるようにするには、キー割り当ての条件を変更する必要があります。ただし、副作用として、ワード単位で移動したときのスクリーンリーダーの読み上げが信頼できなくなります。

# 変更方法

- ctrl+shift+pまたはf1でコマンドパレットを表示
- keyboard shortcuts を実行
- shift+tabを押していって、 open key bindings JSON でエンターする
- JSONが開くので、いかを追記する。

```

{ "key": "ctrl+right",       "command": "cursorWordEndRight",       "when": "textInputFocus" },
{ "key": "ctrl+shift+right", "command": "cursorWordEndRightSelect", "when": "textInputFocus" },
{ "key": "ctrl+left",        "command": "cursorWordLeft",           "when": "textInputFocus" },
{ "key": "ctrl+shift+left",  "command": "cursorWordLeftSelect",     "when": "textInputFocus" },

```

# 詳細

デフォルトの状態では、 ctrl+left/right のショートカットには、 `cursorWordEndRight, Control+RightArrow, DefaulttextInputFocus && !accessibilityModeEnabled` のような発動条件が設定されています。これを見ると分かるように、 ctrl+矢印は、アクセシビリティーモードがオフ、つまり、スクリーンリーダーを使っていないときのみ発動するようになっています。つまり、ctrl+矢印は、スクリーンリーダーを使っているときには Chromium の実装に任せられているということになります。ただ、 Chromium の実装はマルチカーソルを考慮していないため、セットしていたセカンダリーカーソルは全て消えてしまうようです。

なので、ctrl+矢印を、本来vscodeが持っている実装にキー割り当てすることで、マルチカーソルが消えなくなります。やり方として、JSONにさっきのを記述して、アクセシビリティーモードの条件を外しています。

ただ、これをするときに、いくつか注意があります。

まず、移動の単位がChromiumの実装と微妙に違うことです。これは、まあそういうものです。

次に、移動後にNVDAの読み上げる場所と、実際に移動した場所が一致しないことです。これは、vscode側のカスタム実装によってカーソルが制御されていて、NVDAが移動先を誤認識するためと思われます。これは、今はしょうがないので、点字ディスプレイで確認するか、移動する位置を事前に予測して使う必要があります。

詳しい内容は、以下のissueに書いてあります。

[https://github.com/microsoft/vscode/issues/140008](https://github.com/microsoft/vscode/issues/140008)
