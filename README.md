# GR-LYCHEEのカメラ画像をPCのアプリで表示するためのプログラム

## GR-LYCHEEでのオフライン開発環境を作る(チームメンバー向け)
参考文献(https://developer.mbed.org/users/dkato/notebook/offline-development-lychee-langja/)  
このプログラムのコンパイルがローカルのコンパイラ(Windows環境のみ)でしか通らなかったので，Windowsでの開発環境の手順を記載します．
追記(9/18) 
Webコンパイラでもgithubからのインポートでコンパイルが通りました．しかしながら，Webコンパイラにopencvが対応していないそう()なので開発環境としてはmbedのオフライン環境を作っておいてください．

必要なもの(すべて環境変数のPATHを通して再起動する)
- Python(Anacondaが入っていればOK)
- Git(https://git-scm.com/download)
- Mercurial(https://www.mercurial-scm.org/)
- GCC(https://launchpad.net/gcc-arm-embedded/4.9/4.9-2015-q3-update 詳しくは引用を参考に)
- mbed-cli(pipを使ってインストール)

Git,MercurialのPATHが通っているかの確認は以下
```sh
$ git --version
$ hg --version
```
Anacondaを利用している場合，以下のコマンドを使ってPython2.7の仮想環境を作る必要があります．すでにpython2.7の仮想環境が存在する場合は不要です．
```sh
$ conda create -n py27 python=2.7 anaconda
```
仮想環境の確認は次のコマンド．
```sh
$ conda info -e
```
py27環境をactivateした状態でpipでmebd-cliをインストールします．
```sh
$ source activate py27
$ pip install mbed-cli 
```
もとのpython3に戻したい場合は以下のコマンド．今後，mbedのプログラムをコンパイルする場合はpy27をactivateします．
```sh
$ source deactivate  
```

mbedファイルをダウンロードする場合は以下のコマンド．githubからもインポートできます．
```sh
$ mbed import URL
```
git clone コマンドでcloneしてくる場合はdeployコマンドでライブラリをダウンロードする．
```sh
$ git clone URL
$ mbed deploy
```

## テストプログラム(Lチカ)をコンパイルしてみる．
Lチカプログラムをダウンロードしてから，コンパイルするまで．まずはダウンロード．
```sh
$ mbed import https://github.com/d-kato/blinky-for-lychee
```
次にコンパイル．-tオプションはツールチェインの指定でGCC_ARM，-mオプションはボードの指定でGR_LYCHEE．
```sh
$ mbed compile -m GR_LYCHEE -t GCC_ARM --profile debug
```
コンパイルがとおると，./BUILD/GR_LYCHEE/GCC_ARM_DEBUG内に.binファイルができる．あとは，mbedボードにこのファイルをコピーするだけ．


## ここにおいてあるプログラムの概要
カメラ画像をWindows/Mac用PCアプリ**DisplayApp**に表示させるサンプルです。  
偉大なるプログラム引用元(https://github.com/d-kato/GR-Boards_Camera_LCD_sample)．

### カメラとLCDの設定
``mbed_app.json``ファイルを変更することでLCD表示をONにできます。
```json
{
    "config": {
        "camera":{
            "help": "0:disable 1:enable",
            "value": "1"
        },
        "lcd":{
            "help": "0:disable 1:enable",
            "value": "1"
        }
    }
}
```

### Windows用PCアプリで表示する
``main.cpp``の``JPEG_SEND``に``1``を設定すると、カメラ画像をPCアプリに表示する機能が有効になります。  
カメラ画像はJPEGに変換され、USBファンクションのCDCクラス通信でPCに送信します。  
```cpp
/**** User Selection *********/
/** JPEG out setting **/
#define JPEG_SEND              (1)                 /* Select  0(JPEG images are not output to PC) or 1(JPEG images are output to PC on USB(CDC) for focusing the camera) */
/*****************************/
```
PC用アプリは以下よりダウンロードできます。  
[DisplayApp](https://developer.mbed.org/users/dkato/code/DisplayApp/)  

### （追記）Mac環境でのコンパイルエラーについて
Mac環境でコンパイルを試すと以下のエラーが吐き出されました．
```
Compile ...
Compile [  3.3%]: mcu_interrupts.c
Compile [  3.4%]: locking.c
[Error] ioif_aio.h@53,21: conflicting types for 'pthread_attr_t'
[ERROR] In file included from ./mbed-gr-libs/R_BSP/RenesasBSP/drv_inc/TARGET_RZ_A1XX/dma_if.h:46:0,
                 from ./mbed-gr-libs/GraphicsFramework/ospl/porting/locking.c:41:
./mbed-gr-libs/R_BSP/RenesasBSP/drv_inc/ioif_aio.h:53:21: error: conflicting types for 'pthread_attr_t'
 typedef void *      pthread_attr_t;     /**< unsupported */
                     ^~~~~~~~~~~~~~
In file included from /usr/local/Cellar/gcc-arm-none-eabi/20171218/arm-none-eabi/include/sys/types.h:239:0,
                 from /usr/local/Cellar/gcc-arm-none-eabi/20171218/arm-none-eabi/include/stdio.h:61,
                 from ./mbed-gr-libs/R_BSP/RenesasBSP/drv_inc/r_errno.h:20,
                 from ./mbed-gr-libs/R_BSP/RenesasBSP/drv_inc/TARGET_RZ_A1XX/dma_if.h:44,
                 from ./mbed-gr-libs/GraphicsFramework/ospl/porting/locking.c:41:
/usr/local/Cellar/gcc-arm-none-eabi/20171218/arm-none-eabi/include/sys/_pthreadtypes.h:75:3: note: previous declaration of 'pthread_attr_t' was here
 } pthread_attr_t;
   ^~~~~~~~~~~~~~

[mbed] ERROR: "/Users/uchida/.pyenv/versions/2.7.12/bin/python2.7" returned error.
       Code: 1
       Path: "/Users/uchida/progra/HSL_IoT2018/python2_GR/GR-Boards_Camera_LCD_sample"
       Command: "/Users/uchida/.pyenv/versions/2.7.12/bin/python2.7 -u /Users/uchida/progra/HSL_IoT2018/python2_GR/GR-Boards_Camera_LCD_sample/mbed-os/tools/make.py -t GCC_ARM -m GR_LYCHEE --profile debug --source . --build ./BUILD/GR_LYCHEE/GCC_ARM-DEBUG"
       Tip: You could retry the last command with "-v" flag for verbose output
---
```
とても長いですね．この中の，
```
./mbed-gr-libs/R_BSP/RenesasBSP/drv_inc/ioif_aio.h:53:21: error: conflicting types for 'pthread_attr_t'
 typedef void *      pthread_attr_t;     /**< unsupported */
                     ^~~~~~~~~~~~~~
```
の部分が肝のようで，

``./mbed-gr-libs/R_BSP/RenesasBSP/drv_inc/ioif_aio.h``の53行目21文字目から宣言されている変数``pthread_attr_t``が，他の場所で既に宣言されていることが原因（二重宣言）のようでした．

というわけで，下に示している``ioif_aio.h``の53行目をコメントアウトすることで，エラーは生じなくなりました．

```
typedef void *      pthread_attr_t;     /**< unsupported */ 
```
解決の方法としてはこれが最適かどうかはわかりませんが，とりあえずこれで，DisplayAppをコンパイルすることができました．
