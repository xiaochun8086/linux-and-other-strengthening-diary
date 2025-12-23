# Linuxとか強まり日記
### 2025-12-11
- 『Linuxの新しい教科書』Chapter 06まで
    - 検索のコマンドはうるおぼえが多かったから、もう一回やる。
- Udemy『一週間で身につくC/C++言語』繰り返し処理まで
    - 大変重要なことを知る。「void main()」という書き方はいかん。
        - 「何も返さない」ということなので、エラー報告すらできない。
- 『やさしい中学数学』中学２年の分野に突入
    - だけど図形は全体的にやり直したほうがいいと思う
## 今日の気づき
-昔Excelで、セルにセル番地を入れてそれを関数で読み込めたら便利なのになあ、とか考えたことがあって、Cのポインタってつまりはそういうことじゃないの？と考えたことを思い出し、Geminiに聞いてみたら大絶賛してくれた。


### 2025-12-12
-  X240のWindowsパーティションを縮小してUbuntuパーティションを拡大する  
    -  実際Windowsは使ってないし、Officeを何とかし人並みの2倍以上使えるようにテキストも用意したが、結局やらないだろうし「Webで使い方探せばいいじゃん」という結論になり、**とりあえず動けばいい**という基本方針でWindowsの縮小に乗り出す。
    - しかしBIOSとかUEFIとかの違いで大失敗。Windowsが起動しなくなる。この辺はもう一度理由をおさらいして把握しとかないといけない。  
    - デュアルブートに必要な「/boot/efi」というパーティションを論理パーティションに配置したのがよくなかったらしい。明日もう一度検証してみて、これでダメならWindowは捨てることにする。  
    - 「脱Windows」はPCを使いはじめたころから掲げてた目標。今は主にMacだし、X240は元々Linux用に用意したものだし、いろいろ学習目的で準備してきたところにこの事態だから、**「もう潮時かな…」**というところ。これも運命みたいなもん。

### 2025-12-13  

## **Ubuntu導入始末記**  

- インストールしたPC  
    - Lenovo ThinkPad X240(HDDをSSD256GBに換装済み)  
    
- 下準備  
    - BIOS(UEFI)の設定  
        1. 電源ONでF1かF12連打
        2. BootModeはUEFI
        3. Secure BootはDisabled
        4. F10でsave&exit

- インストール  
    - パーティション  
        1: 512MB    /boot/efi  
        2: 8GB      swap  
        3: 50GB     /  
        4: 192GB    /home  

- 初期設定--
  - aptのみでパッケージ管理
        コマンドでupdate＆upgradeやってるとGUIのツールが出てきてうざい  
        `sudo vi /etc/apt/apt.conf.d/20auto-upgrades`で以下の部分

``` 
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

   "1"→"0"に書き換え。
    Ubuntuは勝手にネットに繋いで更新を探しに行くことをやめ、GUIの「アップデートがあります」というポップアップも出なくなる。

 - systemdタイマーの無効化。
    更新チェックのタイマーを完全に切っておく。
   serviceとついてるものは「mask」としないと止められない。

``` 
sudo systemctl stop apt-daily.timer
sudo systemctl disable apt-daily.timer
sudo systemctl mask apt-daily.service
sudo systemctl stop apt-daily-upgrade.timer
sudo systemctl disable apt-daily-upgrade.timer
sudo systemctl mask apt-daily-upgrade.service
```

  - 1passwordのインストール--
     ソフトウェアセンターを利用せず、GPGkeyを取得して、それをシステムに登録して、`apt`コマンドで管理するようにする（このほうが早いらしい）

   1. 鍵を取得するために`curl`を準備

``` bash
sudo apt update
sudo apt install curl
```

   2. GPG Key（署名鍵）の追加

``` bash
curl -sS https://downloads.1password.com/linux/keys/1password.asc | \
sudo gpg --dearmor --output /usr/share/keyrings/1password-archive-keyring.gpg
```
   3. リポジトリの追加

``` bash
echo 'deb [arch=amd64 signed-by=/usr/share/keyrings/1password-archive-keyring.gpg] https://downloads.1password.com/linux/debian/amd64 stable main' | \
sudo tee /etc/apt/sources.list.d/1password.list
```
   4. インストール

``` bash
sudo apt update
sudo apt install 1password
```
   5. ついでに1password-cliもインストール

``` bash
sudo apt install 1password-cli
```

  - Braveのインストール

   1. GPG Keyのダウンロードと登録

```
sudo curl -fsSLo /usr/share/keyrings/brave-browser-archive-keyring.gpg https://brave-browser-apt-release.s3.brave.com/brave-browser-archive-keyring.gpg
```

   2. リポジトリの追加

```
echo "deb [signed-by=/usr/share/keyrings/brave-browser-archive-keyring.gpg] https://brave-browser-apt-release.s3.brave.com/ stable main"|sudo tee /etc/apt/sources.list.d/brave-browser-release.list
```

   3. インストール

```
sudo apt update
sudo apt install brave-browser
```

   - SnapやGUIインストーラーを使用しなかった狙い  
     - 起動速度の確保  
     - システム管理の一元化（aptによる一括update&upgrade)  
   - 使用した技術要素  
     - `curl` : 鍵の取得  
     - `gpg --dearmor` : 鍵をaptが読める形式に変換  
     - `tee` : 管理者権限が必要な場所へのファイル書き込み  
     - `/etc/apt/sources.list.d` : 追加リポジトリのリスト置き場

  - **ダウンロード元をJAISTに変更する**
  
    1. `/etc/apt/sources.list.d/ubuntu.sources`を書き換える
    2. セキュリティーアップデートについては特に変える必要はなし

  - **GitHubの準備**

    1. 必要な道具を入れる
    ```
    sudo apt update
    sudo apt install git gh
    ```

    2. PCとGitHubの紐付け
    ```
    gh auth login
    ```
     `What account do you want to log into?` → **GitHub.com**
     `what is your preferred protocol...?` → **HTTPS**
     `Authenticate Git with you GitHub credentials?` →**Yes**
     `How would you like to authenticate?` →**Login with a web browser**

    3. クローン(複製)する
    ```
    gh repo clone <user name>/<repository>
    ```

    4. 名札をつける
    ```
    git config --global user.name "<user name>"
    git config --global user.email "<user email account>"
    ```

    5. テスト
    ```
    cd <repository>
    vim README.md
    git add .
    git commit -m "<txt>"
    git push
    ```

  - **Ubuntu Proの導入**
    1. トークンの取得
       URL:https://ubuntu.com/pro/dashboard

    2. コマンドで接続
    ```
    sudo pro attach [token]
    ```

    3. 状態の確認
    ```
    pro status
    ```
    ここで表示される中で以下の３つが「Enable」になってることを確認
     - `esm-apps` : Universeリポジトリの保護
     - `esm-infra` : OS基盤の保護
     - `livepatch` : 再起動なしのカーネル更新

     4. Livepatchを確実に有効化する
     ```
     sudo pro enable livepatch
     ```

  - **gcc周りのインストール**
    ```
    sudo apt update
    sudo apt install build-essential
    ```
      - これでインストールされるもの
        - `gcc` : C言語コンパイラ
        - `g++` : C++コンパイラ
        - `make` : ビルド自動化ツール
        - `libc6-dev` : 標準Cライブラリ開発用ファイル

    インストールされたか確認
    ```
    gcc --version
    ```
    gcc (Ubuntu 11.x.x...)みたいなのが出ればOK。

### 2025-12-21

## ブラウザの変更
- iMacでBraveを使ってたけど、なんだか挙動がしんどそうなのでいろいろ変更。
    - Apple製の端末（iMac、iPhone、iPad）：Safari
    - Linux：Chrome（実は相性がいいらしい）

## ThinkPadのタッチパッド設定変更

- 逆スクロールになってたので変更

- `/etc/X11/xorg.conf.d/` にファイルを作成

```
sudo vim /etc/X11/xorg.conf.d/30-touchpad.conf
```

- 以下の内容を記述

```
Section "InputClass"
    Identifier "touchpad"
    Driver "libinput"
    MatchIsTouchpad "on"
    Option "NaturalScrolling" "true"
    Option "Tapping" "on"
EndSection
```

- logout&login、または再起動


## zshとStarshipの導入

- 作業の効率アップのため、Alacrittyを十分に活かすために導入

1. zshのインストール

```bash
sudo apt update
sudo apt install zsh
```

2. デフォルトシェルの切り替え

```bash
chsh -s $(which zsh)
```

3. Starshipのインストール

 公式のインストールスクリプト
```bash
curl -sS https://starship.rs/install.sh | sh
```

4. 連携設定（.zshrc）

zshが起動した瞬間にStarshipを読み込む設定

```bash
vim ~/.zshrc
```

以下の内容を追加

```.zshrc
eval "$(starship init zsh)"
```
5. logout&login

## i3wmでもCapsLockをCtrlキーにする

- 圧倒的にそっちのほうが使いやすい。Gnomeはあっさりできたけど、i3wmではファイルで設定

1. configファイルへ追記
``` zsh
vim ~/.config/i3/config
```

以下の内容を追記

```text
# CapsLockをControlにする（ハッカーの配列）
exec_always --no-startup-id setxkbmap -option ctrl:nocaps
```

2. 保存してリロード。またはログアウトとか

- システム全体で変えてしまう場合

1. ファイル編集
```zsh
sudo vi /etc/default/keyboard
```

2. 書き換え
`XKBOPTIONS=""`の部分を書き換え

```
XKBOPTIONS="ctrl:nocaps"
```

3. 反映

再起動させるか

`sudo udevadm trigger --subsystem-match=input --action=change`

## コマンドの履歴を残す

- zshで入れただけだとコマンドの履歴が残らないので設定

1. .zshrcに追記
```
vim ~/.zshrc
```

から以下を追記

```
# ---ヒストリー（履歴）設定---
HISTFILE=~/.zsh_history

# メモリ上に保存する履歴の数
HISTSIZE=10000

# ファイルに保存する履歴の数
SAVEHIST=10000

# 履歴を追記モードで保存する（上書きしない）
setopt append_history

# 複数のターミナル間で履歴をリアルタイム共有する
setopt share_history

# 直前と同じコマンドは履歴に残さない（連打したときに汚れない）
setopt hist_ignore_dups

# スペースで始まるコマンドは履歴に残さない
# （パスワードなどを打つ時のセキュリティー対策）
setopt hist_ignore_space
```

2. 反映

`source .zshrc`

### 2025-12-22

## GitHubの手順

- GitHubの基本的な手順がまだ身についてないので備忘録を作る

**1. 認証ツールの導入**

```zsh
sudo apt update
sudo apt install gh
gh auth login
```
ログインすると質問される

- `What account do you want to log into?` -> **GitHub.com**
- `What is your preferred protocol for Git operations?` -> **HTTPS**
- `Authenticate Git with your GitHub creentials?` -> **Yes**
- `How would you like to authenticate?` -> **Login with a web browser**

画面に8桁のコードが表示されるのでコピー。Enterを押すとブラウザが開くのでコードをペースト。 
「Authorize」を押すとGitHubの接続完了。

### 2025-12-23

## C言語学習用のディレクトリでのStarshipの表示

使ってるコンパイラをプロンプトで表示してくれるが、gccでなくclangが表示。

1. 設定ファイルに書き込む

```
vim ~/.config/starship.toml
```

2. 以下の内容を書き込む

```
# C言語の設定を上書き
[c]
# デフォルトの cc ではなく、gcc をバージョン検出に使う
commands = [ ["gcc", "--version"] ]

```

3. 一応Alacrittyを再起動

Starshipを起動させれば自動的に設定ファイルが作られると思ってた。
これはStarshipの設計思想らしく、

** デフォルトでも充分使えるんや **

という自信の表れだと。


