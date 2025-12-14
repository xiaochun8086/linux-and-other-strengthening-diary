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

``` 
sudo systemctl stop apt-daily.timer
sudo systemctl disable apt-daily.timer
sudo systemctl mask apt-daily.service
sudo systemctl stop apt-daily-upgrade.timer
sudo systemctl disable apt-daily-upgrade.timer
sudo systemctl mask apt-daily-upgrade.service
```
   serviceとついてるものは「mask」としないと止められない。

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
