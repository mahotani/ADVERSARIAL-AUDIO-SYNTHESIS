# ADVERSARIAL-AUDIO-SYNTHESIS

## 0.ABSTRACT
この研究は、GANを用いて人が認識しやすい音声を生成することを目的としたものです。  
現状、ほとんどのGANは画像の生成で用いられていましたが、音声の生成では用いられていませんでした。  
この研究では、既存のGANを音に適したものに作り替えて、それに加えて音の特徴を用いて**SpecGAN**と**WaveGAN**の2種類のGANを作りその性能を比較しています。  

## 1.INTRODUCTION
音声生成は、音楽制作や映画のSE音で実用的であると考えられています。  
映画などの制作に携わっている音響監督の方達は、作品内で使用するSE音を選ぶ時、多数ある音の中からその場面に合う一つを見つけなければなりません。とても面倒臭い作業です。  
そこで、音声生成があるとSE音を探したい場面の情報をインプットしただけで適した音を生成してくれるとその作業が楽になるのではないかと考えられます。  
<img width="625" alt="how_to_use_voice_synthesis" src="https://user-images.githubusercontent.com/39772824/71435632-22423f80-272d-11ea-985f-6a55735da5d9.png">  
従来の音声生成には自己回帰トレーニングによるニューラルネットワークモデルがあげられるが、これは出力が出るたびにフィードバックをしなければならないので、とても時間がかかる方法です。  
画像生成で使われているGANを音声生成で使用するには、スペクトログラムに変換して画像として扱うと簡単になると考えられます。  

この論文では、2種類のGANの提案をしています。

---
1つ目はSpecGANと呼ばれるもので、これは入力のオーディオデータをスペクトログラムに直して扱うモデルです。  
<img width="776" alt="SpecGAN" src="https://user-images.githubusercontent.com/39772824/71434596-962e1900-2728-11ea-9a69-93b3c72d03e9.png">  

---
2つ目はWaveGANと呼ばれるもので、これは画像生成に使われているDCGANを音声生成に対応するように作り替えたものです。入力データを別の形に変換せずにそのまま使えるのが特徴です。  
<img width="555" alt="WaveGAN" src="https://user-images.githubusercontent.com/39772824/71434590-91696500-2728-11ea-9958-3d52cec1892f.png">  

## 3.WAVEGAN
WaveGANは画像と音声の違いを見つけ、それらの違いを使い従来の画像の生成に用いていたGANを音声用に作りかえたものです。

### 3.1INTRINSIC DIFFERENCES BETWEEN AUDIO AND IMAGES
<img width="690" alt="principal_audio_images" src="https://user-images.githubusercontent.com/39772824/71436030-e60fde80-272e-11ea-8125-99e92374798f.png">  
上の画像は音声と画像を主成分分析した結果です。
<dl>
  <dt>音声</dt>
  <dd>2次元のデータ構造</dd>
  <dd>エッジや色の強度などの特徴を抽出している</dd>
  <dt>画像</dt>
  <dd>1次元のデータ構造</dd>
  <dd>周期性が強く表れている</dd>
</dl> 
2次元データに対応していた従来のGANを1次元データに対応させるように作りかえれば良いように思えるが、それ以外にも特徴の違いが見られるので他の部分も考慮しつつ作りかえないとうまくいかないと思われる。

### 3.2WAVEGAN ARCHITECTURE
WaveGANはDCGANを元にして作られています。  
画像用のDCGANは画像が2次元データなので、2次元のデータを扱う構造をしているが、音声データを扱うためには1次元のデータを扱う構造に直す必要があります。  
画像を生成する際、stride factorと呼ばれる空白を徐々に増やしていきその空白をすでに形成されているデータと照らし合わしながら埋めていきます。  

画像の場合
---
<img width="555" alt="DCGANinWaveGAN1" src="https://user-images.githubusercontent.com/39772824/71439955-f62fba00-273e-11ea-8cc2-4fbf3e7abd20.png"> 

音声の場合
---
<img width="555" alt="DCGANinWaveGAN2" src="https://user-images.githubusercontent.com/39772824/71439961-faf46e00-273e-11ea-8a06-fc75cd4c054f.png">  

学習方法はWGAN-GPと同じものを使っています。  
また、WaveGANでは、本来のDCGANと違い、バッチ正規化を行っていません。  

### 3.3PHASE SHUFFLE
DCGANは画像生成する時に生成した画像にチェッカーボードと呼ばれるジャギのようなものが発生することが知られています。  
音声の場合は、いずれかの音階が壊れることがあります。  
それを防ぐために以下の図のようなイメージでディスクリミネータ側で生成されたデータをシャッフルしています。  

<img width="405" alt="Phase_shuffle" src="https://user-images.githubusercontent.com/39772824/71440336-2e83c800-2740-11ea-9dbd-602e14314d59.png">

生成されたデータを細かく区切り、いくつかの整数をランダムで用いて、例えば  
- -1の場合は、左に1つずらし、はみ出た部分を空いたとこに戻す。  
- 0の場合は何もしない。  
- 1の場合は、右に1つずらし、はみ出た部分を空いたとこに戻す。  
のような作業を行います。  
WaveGANでは[-2, 2]の範囲の整数を用いて行っています。
