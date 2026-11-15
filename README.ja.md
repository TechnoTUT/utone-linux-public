# utone-linux

[English](README.md) | 日本語

このリポジトリは、学内DJイベント "The Utopia Tone" のエッジデバイスを容易に動作させるためのDebian Live環境を提供します。[moip-toolkit](https://github.com/TechnoTUT/moip-toolkit)(NDI送受信とデジタルサイネージ)と[booth-display](https://github.com/TechnoTUT/booth-display)(ブースディスプレイのコントローラ)を含みます。  

## 使い方
ビルド済みのイメージは配布していません(ビルドしたイメージにはNDI SDKのライセンスが適用されるNDIランタイムが含まれるため。[ライセンス](#ライセンス)を参照)。[ビルド方法](#ビルド方法)の手順でISOイメージをビルドし、DVDまたはUSBメモリに書き込みます。USBメモリに書き込む場合は、[Rufus](https://rufus.ie/)等のツールを使用します。  
再起動しても設定が保持されるようにするには、Rufusの設定で保存領域のサイズを指定します。1GB程度で十分です。NDI送信やサイネージの設定を行う場合には、保存領域の設定を推奨します。

完成したDVDまたはUSBメモリをPCに挿入し、電源を投入するとLive環境が起動します。

### NDI受信
NDI信号を受信するには、以下のコマンドを実行します:  
```bash
nrx
```

接続先を設定して自動起動を有効にするには、ホームディレクトリの `.config/systemd/user/nrx.service` を編集します: 

```bash
vim ~/.config/systemd/user/nrx.service
systemctl --user enable --now nrx.service
```

`nrx` は初期設定でKMS/DRMを使用し、デスクトップのコンポジタを経由せずにディスプレイへ直接出力して遅延を最小にします。デスクトップセッション上に表示する場合は、コメントアウトされている `--video-driver auto` 付きの `ExecStart` を使用します。ソースを指定して手動で実行するには:  
```bash
nrx -s "<NDIソース名>"
```
- 対象のディスプレイでデスクトップセッションが動作していない必要があります(コンソール / TTY の状態)。`sudo systemctl set-default multi-user.target` でGUIなしで起動するようになります。
- ユーザーを `video` と `input` グループに追加します: `sudo usermod -aG video,input $USER`
- 出力は常に全画面になります。デスクトップセッションが画面を使っている場合は、デスクトップ上のウィンドウ表示に切り替わります。

### NDI送信
キャプチャーボードからNDI信号を送信するには、以下のコマンドを実行します:  
```bash
ntx
```
NDI信号を `<HOSTNAME> (TX)` として送信します。

送信名や解像度、フレームレートなどの引数を変更し自動起動を有効にする場合は、ホームディレクトリの `.config/systemd/user/ntx.service` を編集し、サービスを起動します。引数の一覧は、`ntx --help`で確認できます。  
```bash
vim ~/.config/systemd/user/ntx.service
systemctl --user enable --now ntx.service
```
初期設定のホスト名は `debian` です。ホスト名を変更するには、`sudo hostnamectl set-hostname <新しいホスト名>` と入力します。

### サイネージ
サイネージ機能は moip-toolkit が提供します。Web UIのサービスを起動し、ブラウザから画像をアップロードすると、接続されたディスプレイに全画面で表示されます。  
```bash
systemctl --user enable --now nweb.service
```
ブラウザで `http://<IPアドレス>:8000/` を開きます(IPアドレスはログイン時に表示されます)。再起動後も設定を保持するには、保存領域を使用します。  

コマンドラインから直接画像を表示することもできます。  
```bash
moip signage /path/to/image.png
```
`--framebuffer` を付けると、GPUを使わずにLinuxフレームバッファ(`/dev/fb0`)に直接描画します(Liveユーザーは `video` グループに所属しています)。  
詳細は [moip-toolkit の README](https://github.com/TechnoTUT/moip-toolkit) を参照してください。

### ブースディスプレイ
[booth-display](https://github.com/TechnoTUT/booth-display) のコントローラは、テストパターン、動画ファイル、NDIソースをタイル状に並べたブースディスプレイへ配信します。以下のコマンドで起動します:  
```bash
bth
```
自動起動を有効にするには、サービスを有効化します:  
```bash
systemctl --user enable --now booth.service
```
ブラウザで `http://<IPアドレス>:8080/` を開くとWeb GUIを利用できます。各ディスプレイの宛先やクロップ位置は `/opt/booth-display/config.yaml` で設定します。  

NDI入力には、公式NDI SDK for LinuxのNDIランタイム(`libndi.so.6`)を使用します。ISOのビルド時にSDKをダウンロードし、ライセンスに自動で同意して `/usr/local/lib` にランタイムをインストールします。ビルドする前にNDI SDKのライセンスを確認してください。  
詳細は [booth-display の README](https://github.com/TechnoTUT/booth-display) を参照してください。

## ビルド方法
### Dev Containerを使用する場合 (推奨)
VS Code と Dev Containers 拡張機能を使用することで、ホスト環境を汚さずにビルド環境を構築できます（Docker または Podman が必要です）。

1. VS Code でこのリポジトリを開きます。
2. ポップアップまたはコマンドパレットからコンテナ内で再度開きます。
3. コンテナ内ターミナルで以下を実行します:
   ```bash
   make build
   ```
   クリーンアップする場合:
   ```bash
   make clean
   ```

### 通常のDebian / WSL2環境を使用する場合
ビルドにはDebian環境が必要です。Windowsを使用している場合は、WSL2を利用することで簡単にDebian環境を構築できます。  
```pwsh
wsl --update
wsl --list --online
wsl --install -d Debian
```

Debian環境で`live-build`と`make`をインストールします。
```bash
sudo apt update
sudo apt install live-build make
```
このリポジトリを`--recursive`オプションを付けてクローンします。
```bash
git clone https://github.com/TechnoTUT/utone-linux-public.git --recursive
cd utone-linux-public
```
`make` を使って ISO イメージをビルドします。
```bash
make build
```
ビルドが完了すると、`work` ディレクトリ内に `utone-linux-amd64.hybrid.iso` が生成されます。  
ビルドをやり直す場合やクリーンアップを行う場合は、`make clean` を実行します。
```bash
make clean
```

## ライセンス
このリポジトリのファイルは[MITライセンス](LICENSE)で公開しています。[moip-toolkit](https://github.com/TechnoTUT/moip-toolkit)と[booth-display](https://github.com/TechnoTUT/booth-display)もMITライセンスです。  

ビルドしたISOイメージには、ビルド時にダウンロードされるソフトウェアも含まれ、それぞれのライセンスが適用されます。NDI SDK for LinuxのNDIランタイムにはNDI SDKのライセンスが適用されるため、ビルドしたイメージを再配布しないでください。NDI®はVizrt NDI ABの登録商標です。  
