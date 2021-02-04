# CMakeを使用したビルド方法
以下、Windows環境下でのCMakeによる mosquitto のビルド方法を説明します。

## はじめに
### 概要
あらかじめ簡単に説明すると、最終目標を mosquitto.exe または mosquitto.dll の出力とした場合、作業は以下の流れになります。

1. CMake を使用することで VisualStudio のソリューションファイルを生成する
1. 生成された VisualStudio のソリューションファイルを使用して、VC++のコンパイラで mosquitto.exe および mosquitto.dll を出力する

### 事前に必要な環境
* mosquitto のソースファイルを用意
	* [こちら](https://github.com/eclipse/mosquitto)からリポジトリのクローンまたはダウンロードができます
* VisualStudio2013以降
	* 詳細は不明ですが、VisualStudio2010ではビルド失敗しました（`stdbool.h` が存在しないためみたいです）
	* 参考までに、私自身は VisualStudio2017 でビルド成功しました
* OpenSSL(32bit版)
    * もしかしたら 64bit 版でもいいかもしれません
    * 以下の階層に配置しないといけないみたいです
        * `C:\OpenSSL-Win32`
    * また、以下を環境変数の Path に追加する必要があるみたいです
        * `C:\OpenSSL-Win32\bin`
* CMake
	* [こちら](https://cmake.org/download/)などから CMake をダウンロードします。（`Binary distributions` がインストーラです）
	* 64bit版でも、mosquitto を 32bit でビルドできるので、x86かx64どちらでもいいと思います
	* ある程度新しいバージョンでないと、VisualStudio2017 等を選択できないので注意してください
	* 参考までに、私自身は Ver3.19.4 で生成実績があります
* pthreads
    * インストール方法は失念してしまいました（後日加筆します）
    * 後追いで記事を書いているので、このフォルダはもしかしたら自動生成されるのかもしれません。その場合はすみません

## 作業
### 1. CMake で VisualStudio のソリューションファイルを生成する
CMake を起動して、以下の通り入力（ブラウズでフォルダーを選択）します。

* 「source code」 に mosquitto のリポジトリフォルダを指定（`config.mk` のあるフォルダ）
* 「build the binaries」 は任意（私は `source code` と同じところを指定しました）

![](.\images\screenshot_23.png)

次に、「Configure」ボタンを押して、生成するVSソリューションファイルのバージョンを指定します。  
前述したとおり、VS2013以上を選択しないと最終的に mosquitto 出力に成功しないみたいです。  
私は VS2017 を指定しました。  
選択したら「Finish」を押します。

![](.\images\screenshot_24.png)

すると、CMake が動作し始めます。  
出力ウィンドウに `Configuring Done` が表示されたら、とりあえず Confiture は成功みたいです。

![](.\images\screenshot_25.png)

<**重要**>  
`WITH_THREADNING` にチェックが入っていることを確認してください。  
これがないと、mosquitto.dll において `mosquitto_loop_start` などの関数が使用できなくなります。  
ただし、この関数自体がレガシーなので、使用目的によっては不要かもしれません。

![](.\images\screenshot_26.png)

以上で、「Generate」ボタンを押します。  
出力ウィンドウに `Generating Done` が表示されたら完了です。

![](.\images\screenshot_27.png)

### 2. VisualStudio から mosquitto をビルドする
CMake の 「build the binaries」で指定したフォルダに `mosquitto.sln` が出力されているので、それを起動します。

複数のプロジェクトがありますが、「ALL_BUILD」を選択してビルドしてみてください。  
これが通れば、とりあえず全てが出力成功になります。

![](.\images\screenshot_28.png)

#### 補足
私の環境では、ビルドすると `client_shared.c` の L1198 あたりでエラーになりました。どうやら `∞` というマルチバイト文字が悪さをしている？っぽいのが原因みたいです。  
client で使用する箇所みたいなので、使用することはないかなと思い、とりあえず if 文自体を消去するとビルドは通りました。

#### 出力場所
* mosquitto.dll は、`lib\(Debug|Release)` フォルダに出力されるようです
* mosquitto.exe は、`src\(Debug|Release)` フォルダに出力されるようです