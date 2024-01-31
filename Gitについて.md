# 初めてローカルからGitHubにPushしたので備忘録
***
## ssh接続の設定
***
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
メインアカウントとサブアカウントで分けたかった

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