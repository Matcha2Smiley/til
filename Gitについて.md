# 初めてローカルからGitHubにPushしたので備忘録

## ssh接続の設定

GitHubなどのリモートリポジトリに接続する際にはHTTPSとSSHの二種類の接続方式がある。今回、SSH認証を選択した理由は認証が楽そうという漠然としたイメージがあったから。 調べてみると、GitHubが推奨する認証方式はHTTPSなんだそう。ただ接続する際に一々、ユーザー名とPAT(Personal Access Token ※時限付きのパスワードの代わりになるToken)を発行して入力する必要があるみたいで使い勝手的にはSSHの方が楽そうな印象を受けた。これについては今度ちゃんと調べようと思う。

### SSH接続の手順

#### 1. SSHキーの作成
何はともあれ、まずはSSHキーを作成していきます。
といってもコマンドでポチッとするだけなんですが

```
% ssh-keygen -t ed25519 -N "" -f ~/.ssh/github
```
* -t
    * 暗号化形式　RSAよりed25519の方が強いらしい
* -N
    * 新しくパスフレーズを作成
* -f
    * ファイルを指定

これで、SSH keyが完成

#### 2. SSHキーの登録
さっき作ったSSH keyをGitHubに登録する

```
% pbcopy < ~/.ssh/github.pub
```
これで公開鍵をクリップボードにコピーしてくれる

そしたら[GitHub](https://github.com/settings/keys)に行って

Setting > SSH and GPG Keys > New SSH key

* Title
    * 適当でok（どのPCかわかるようにするといいかも）
* Key
    * ここにさっきクリップボードにコピーしたssh keyをペースト

できたら"Add SSH key"をクリックして完了

#### 3.Configの編集
```
vi ~/.ssh/config
```

今回は色々な兼ね合いでこんな感じに設定しました
メインアカウントとサブアカウントで分けるテストも兼ねて

```
#メインアカウント
Host github
        HostName github.com
        IdentityFile ~/.ssh/github
        User git

#プライベートアカウント
Host github-sub
        HostName github.com
        IdentityFile ~/.ssh/github-sub
        User git

```

下記の二つだけ固定であとは適宜やって下さい

```
HostName github.com
User git
```

#### 4.接続確認
```
ssh -T git@[設定したHost名]
```
これで下記の内容が返ってきたら接続成功
```
Hi [@User名]! You've successfully authenticated, but GitHub does not provide shell access.
```

## GitHubへのpush

さて、GitHubにSSH接続できるようになったので
いよいよヴァージョン管理を体験していく

### GitHubでバージョン管理するまでの手順

#### 1.GitHubにリポジトリを新規作成する

手抜きですが、自分用のメモなのでご愛嬌ということで
[リポジトリ作成方法:公式ドキュメント](https://docs.github.com/ja/repositories/creating-and-managing-repositories/quickstart-for-repositories)


#### 2.ローカルリポジトリを作成する

[リポジトリとは](https://docs.github.com/ja/enterprise-cloud@latest/repositories/creating-and-managing-repositories/about-repositories)

僕の場合

```
% mkdir ~/workspace/private/
```
ここでprivateディレクトリとしているのは後々に
パブリックに使うリポジトリと個人で使うリポジトリを分けるため
リポジトリごとにユーザーを自動で変えられるように設定できるのが楽なので
これについては後述します

・デフォルトブランチの変更(mainにする)
```
% git config --global init.defaultBranch main
```
[ブランチとは](https://docs.github.com/ja/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-branches)

・リポジトリ作成
```
% git init til
```

#### 3.リモートリポジトリと対話する準備

・リポジトリに移動
```
% git remote add origin git@github:USERNAME/til.git
```

ここで'git@'の後に指定している'github'はsshの設定の時にconfigで設定したHost名
今回はメインアカウントを使っているがサブアカウントなら
```
% git remote add origin git@github-sub:USERNAME/til.git
```
これでリモートリポジトリとの対話はできるようになった

#### 4.ようやくpushをする

・まずはpushするファイルを用意

```
% touch pushtest.txt
```

・次に`git add`する
```
% git add pushtest.txt
```
[git addとは](https://docs.github.com/ja/get-started/using-git/about-git)
>git add は変更をステージします。 Git は開発者のコードベースへの変更を追跡しますが、プロジェクトの履歴に含めるためには、変更をステージしてスナップショットを作成する必要がありま>す。 このコマンドは、その 2 段階のプロセスの最初の部分であるステージングを実行します。 ステージされた変更は、次のスナップショットの一部となり、プロジェクトの履歴の一部となります。 ステージングとコミットを別々に行うことで、開発者はコーディングや作業の方法を変えることなく、プロジェクトの履歴を完全に管理することができます。


・そしてコミット
```
% git commit -m "何したのかここに書く"
```

・ついにpush
```
% git push origin main
```
remote先として使いした`origin`の`main`ブランチに`push`

Let's gooooooooo