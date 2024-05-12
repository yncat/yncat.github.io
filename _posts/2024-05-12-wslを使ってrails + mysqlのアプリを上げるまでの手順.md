---
layout: post
tags: tips
---

# メモ用です

自分はいつも雰囲気でやってて環境があっちゃうので、そのまま動かなかったらあとで更新します。

## Step 1: WSL 2 をインストール

1. [スタート]メニューにある検索バーに、 powershell と入力します。
2. Windows Powershell を右クリックして、「管理者として実行」を押します。
3. Powershell のウィンドウで、 wsl --install と入力してエンターを押します。
4. いろいろインストールされるのを待ちます。
5. Enter new unix password と言われたら、適当なパスワードを設定します。
6. ubuntu のコマンド入力待ちになったら完了です。

## Step 2: WSL 仮想マシンの起動と終了

WSL は、 Windows 上に Unix (今回は Ubuntu ) という種類の OS を立ち上げます。

WSL の仮想マシンを立ち上げるには、 [スタート] メニューから、 Ubuntu を検索して実行します。

Ubuntu を操作するには、コマンド入力を使います。

Ubuntu を終了して抜けるには、 exit と入力します。

### Step 3: 必要なパッケージを入れる

以下をコピペで全部インストールしましょう。

```
sudo apt update && sudo apt install -y curl autoconf patch build-essential rustc libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libgmp-dev libncurses5-dev libffi-dev libgdbm6 libgdbm-dev libdb-dev uuid-dev libmysqlclient-dev
```

## Step 4: 必要なファイルを準備する

GitHub から Clone するときの鍵などを、仮想マシン側にコピーする必要があります。

WSL から、 Windows 上のドライブにアクセスできます。この時に決まりがあります。それは、 C ドライブは /mnt/c/ のディレクトリに、 D ドライブは /mnt/d/ のディレクトリに……というように、 /mnt/ドライブレター/ のディレクトリを見ればいいということです。

ということは、Windows 上にあるファイルは、以下のようにして仮想マシンに持って来ることができます。

1. ファイルを選んで、シフトを押しながら右クリックします。
2. 「パスのコピー」を選びます。
3. Ubuntu のウインドウに行きます。
4. cp "/mnt/ と入力してからスペースを1個あけます。
5. mnt/ の / の直後に、スペースがない状態で、貼り付けを実行します。
6. 貼り付けた最初に " の記号が入った場合は取り除きます。
7. ドライブレターの C や D などを小文字にします。
8. そのあとの : を取り除きます。
9. \ の記号を / に置き換えます。
10. 貼り付けた最後の " のあとにスペースを1つ空けて、仮想マシン上のコピー先フォルダを指定します。
11. コマンドを確認したらエンターで実行します。

たとえば、 ssh の鍵が `C:\test\id_ed25519` だったら、実行するコマンドは `cp /"/mnt/c/test/id_ed25519" ~/.ssh` になります。

まだキーを作ってないとか、新しく作っていい場合は、 Ubuntu 上で ssh-keygen コマンドで作成して配置してください。

ここで、 git clone とかで、動かしたいアプリを仮想マシン上にダウンロードしておきましょう。

## step 5: 必要なものをインストールする

ここが一番大変です。頑張って。

### mise

mise は、 Ruby とかを楽にインストールできるコマンドです。

[公式](https://mise.jdx.dev/getting-started.html) に従ってインストールします。

以下のコマンドを実行します。

```
curl https://mise.run | sh
echo 'eval "$(~/.local/bin/mise activate bash)"' >> ~/.bashrc
. .bashrc
```

### Ruby

`mise install ruby バージョン番号`

でインストールします。バージョン番号は、アプリのほうで指定されていると思うのでそれを参照します。

アプリのリポジトリの中に .tool-versions とか .ruby.version とかいうファイルがあれば、そのディレクトリに入った状態で `mise install ruby` とすれば指定のバージョンが入ることがあります。

Ruby のインストールには、数分かかります。逆に、一瞬で終わっていたらそれは失敗しています。

インストールできたら、 `mise use ruby バージョン番号` でとりあえず使うバージョンを初期設定に入れておきましょう。

### Docker

mysql を Docker で立ち上げた方が楽なので、 Docker もインストールします。 [公式](https://docs.docker.com/engine/install/ubuntu/) の通りにします。

```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
```

ここまでやったら、一度 exit で抜けて、もう1回 Ubuntu を起動し直します。

### mysql のコンテナを上げておく

再起動した後、以下のコマンドを実行します。

`docker run --name mysql -e MYSQL_ROOT_PASSWORD=パスワード -d -p 3306:3306 mysql`

パスワードは、 Rails アプリの config/db.yml に書いてある development のパスワードを指定します。たぶん、 development 環境だと、ユーザーとして root を使っているか、 mysql を浸かっていると思います。 mysql を使っていたら、一度 DB に入ってユーザー作成する必要があるかもしれません。

ちなみに、こんな長いコマンドを打つのは最初だけです。1回これをやったら、次からは docker start mysql とか docker stop mysql とか打てばOKになります。

## Step 6:いよいよ Rails を上げる

お疲れ様でした。いよいよ運命の瞬間です。

アプリのディレクトリに入って、

```
bundle install
```

を実行します。

もしここで、 bundle がねえ！！！みたいに言ってきたら、 ruby のインストールがうまくできてないです。

もし、 bundle install の途中で bundler cannot install なんちゃらかんちゃら とか言ってきたら、たぶんここの説明を修正する必要があるのでこっそり教えてください。

`bundle install` できたら、 `bundle ex rake db:reset` を実行します。この意味がよくわからない人は、教科書通りに `bundle ex rake db:create && bundle ex rake db:migrate && bundle ex rake db:seed` としても同じです。なお、これをやるまえに mysql が上がってないといけないので、 `docker start mysql` しておいてください。

## Step 7: いよっしゃああああ

ここまでできたらあとちょっと。

`bundle ex rails s`

でサーバーを立ち上げて、 `localhost:3000` にブラウザでアクセスしましょう。

アプリのページが表示されれば成功です。

ここまでできれば、関連する rake タスクも全部動くと思いますし、デプロイに capistrano とかを使っているなら、 bundle ex cap なんちゃらかんちゃら deploy みたいなコマンドも動くはずです。
