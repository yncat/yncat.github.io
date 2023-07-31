---
layout: post
tags: vscode tips
---

# はやりには追従していきます

[公式ドキュメント](https://docs.github.com/ja/copilot/github-copilot-chat/using-github-copilot-chat) の通りにやれば動くんですが、私のブログなので、スクリーンリーダーでの操作方法です。

ちなみに、たぶんまだ個人には降ってきてないと思います。 organization または enterprise 単位で有効化してくれている組織に属していれば使うことができます。なので、有効化ステップの説明は省略します。

## まずは Extension を入れる

vscode に GitHub Copilot の拡張機能を入れていましたが、どうやら別の拡張機能が必要らしいです。ということで、 ctrl+shift+x で拡張機能の画面を出して、 copilot chat と検索して、出てきた GitHub Copilot Chat という拡張機能を入れます。拡張機能を入れたら、 vscode をリロードまたは再起動しておきます。

私は以前に GitHub Copilot の拡張機能を入れていたので、チャットの方の拡張機能を入れたときに GitHub の認証をすっ飛ばしてしまいました。もし、チャットから入れた場合は、なんか認証画面が出るのかもしれません。

## チャット画面を開く

公式サイトに

> Visual Studio Code のアクティビティ バーで、 [GitHub Copilot Chat] アイコンをクリックして、[GitHub Copilot Chat] チャット ウィンドウを開いてください。

とか書いてあるのですが、スクリーンリーダーユーザーからしたら、なんのこっちゃwwwという感じです。これの答えを説明します。

最短ルートは、 ctrl+alt+i を押すことです。

公式の説明をキーボードで再現するのであればこうです。

- F6 を押していく
- Explorer (Ctrl+Shift+E) とかいうタブコントロールがあったらそこでストップ
- 下矢印を押していく
- Chat (Ctrl+Alt+I) というのがあったらここでエンター

## 使い方その1: 質問を入力する

例えば、チャット画面に、

Write a python program which gets cat photos from the internet!

とか入力します。入力したらエンターを押します。

しばらく考えた後、答えが返ってきます。答えが返ってくると自動で読み上げられます。今回はこんな答えを返してきました。

Sure, here's an example Python program that uses the requests library to download cat photos from the internet:

``````
import requests

# Define the URL of the cat photo API
url = 'https://api.thecatapi.com/v1/images/search'

# Define the number of cat photos to download
num_photos = 5

# Loop over the number of photos and download each one
for i in range(num_photos):
    # Send a GET request to the cat photo API
    response = requests.get(url)

    # Extract the URL of the cat photo from the response JSON
    photo_url = response.json()[0]['url']

    # Download the cat photo and save it to a file
    response = requests.get(photo_url)
    with open(f'cat{i+1}.jpg', 'wb') as f:
        f.write(response.content)
``````

In this example, we first import the requests library, which provides a simple way to send HTTP requests in Python. We then define the URL of the cat photo API and the number of cat photos to download.
We then loop over the number of photos and send a GET request to the cat photo API using the requests.get method. We extract the URL of the cat photo from the response JSON using the response.json() method, and then download the photo using another requests.get method call. Finally, we save the photo to a file using the open function and the write method.
This program will download the specified number of cat photos and save them to files named cat1.jpg, cat2.jpg, etc. in the current working directory.

今回はあまりにも唐突な質問なのでちょっとあれな答えが返ってきていますが、まぁまだベータ版なので、こんなものでしょう。

## 答えの確認方法

ctrl+上矢印で、返ってきた答えにカーソルを移動します。逆に、ctrl+下矢印で、自分の入力画面にカーソルを移動します。

コードを提案してきた場合、 F9 でコードブロックにカーソルを移動します。

## 使い方その2: コードを見せて質問する

現在エディターでアクティブにしているファイルの情報を共有できます。例えば、エディターにこんなプログラムを書いておきます。

```
import sys

target = sys.argv[1]

for i in range(target):
    current = i + 1
    if i%5 == 0:
        print("fizz")
    elif i%3 == 0:
        print("buzz")
    elif i%3 == 0 and i%5 == 0:
        print("fizzbuzz")

```

これは微妙に間違っているのですが、あえて聞いてみます。
チャット画面に

This is a fizzbuzz program. Is this correct?

と入力します。

そうすると、こうなります。

The code you provided has a few issues that would prevent it from working correctly as a FizzBuzz program. Here are the issues I see:
1. 
The range function expects an integer argument, but target is a string obtained from the command line arguments. You need to convert target to an integer using the int function.
2. 
The order of the if statements needs to be changed so that the "fizzbuzz" case is checked first, since it is the most specific.
3. 
The conditions in the if statements are incorrect. The "fizz" case should check for multiples of 3, the "buzz" case should check for multiples of 5, and the "fizzbuzz" case should check for multiples of both 3 and 5.
Here's the corrected code:

``````
import sys

target = int(sys.argv[1])

for i in range(target):
    current = i + 1
    if i%3 == 0 and i%5 == 0:
        print("fizzbuzz")
    elif i%5 == 0:
        print("buzz")
    elif i%3 == 0:
        print("fizz")
    else:
        print(current)
``````

In this version of the code, we first convert target to an integer using the int function. We then check the "fizzbuzz" case first, followed by the "buzz" and "fizz" cases. Finally, if none of these cases apply, we print the current number.
```

## 使い方その3: スラッシュコマンド

/ を入力すると、利用可能なコマンドの一覧が表示されます。それぞれのコマンドがなにをするのかは、 Copilot 自身に聞くと教えてくれます。ただ、 /ext というのがあるのですが、これだけはまだ知らないようです。公式サイトにも書いてないので、なんのコマンドか、執筆時点ではわかりません。

さっき fizzbuzz のコードを文章で指示して指摘してもらいましたが、コードを書いた状態で /fix コマンドを使うと同じことができます。

/tests コマンドを使うと、今書いてあるコードに対してテストを書いてくれます。このコードは、ぱっと見そのままは使えない気がしますが、まだベータ版ですから、こんなものでしょう。

Sure, here's an example set of unit tests for the FizzBuzz code you provided:

```
import unittest
from io import StringIO
import sys
from fizzbuzz import fizzbuzz

class TestFizzBuzz(unittest.TestCase):
    def test_fizzbuzz(self):
        # Test that the function produces the correct output for a range of inputs
        expected_output = "1\n2\nfizz\n4\nbuzz\nfizz\n7\n8\nfizz\nbuzz\n11\nfizz\n13\n14\nfizzbuzz\n"
        with StringIO() as buffer:
            sys.stdout = buffer
            fizzbuzz(15)
            output = buffer.getvalue()
        self.assertEqual(output, expected_output)

    def test_fizz(self):
        # Test that the function correctly identifies multiples of 3
        expected_output = "1\n2\nfizz\n4\n"
        with StringIO() as buffer:
            sys.stdout = buffer
            fizzbuzz(5)
            output = buffer.getvalue()
        self.assertEqual(output, expected_output)

    def test_buzz(self):
        # Test that the function correctly identifies multiples of 5
        expected_output = "1\n2\nfizz\n4\nbuzz\n"
        with StringIO() as buffer:
            sys.stdout = buffer
            fizzbuzz(5)
            output = buffer.getvalue()
        self.assertEqual(output, expected_output)

    def test_fizzbuzz(self):
        # Test that the function correctly identifies multiples of both 3 and 5
        expected_output = "1\n2\nfizz\n4\nbuzz\nfizz\n7\n8\nfizz\nbuzz\n11\nfizz\n13\n14\nfizzbuzz\n"
        with StringIO() as buffer:
            sys.stdout = buffer
            fizzbuzz(15)
            output = buffer.getvalue()
        self.assertEqual(output, expected_output)

if __name__ == '__main__':
    unittest.main()
``````

In this example, we define a TestFizzBuzz class that inherits from unittest.TestCase. We then define four test methods:
• test_fizzbuzz: This tests that the fizzbuzz function produces the correct output for a range of inputs.
• test_fizz: This tests that the fizzbuzz function correctly identifies multiples of 3.
• test_buzz: This tests that the fizzbuzz function correctly identifies multiples of 5.
• test_fizzbuzz: This tests that the fizzbuzz function correctly identifies multiples of both 3 and 5.
Each test method uses the StringIO class to capture the output of the fizzbuzz function, and then compares it to the expected output using the assertEqual method. Finally, we use the unittest.main() function to run the tests.

# まあこんな感じです

ちょっとまだ実用的ではないかな。
