# EMUZ80-Z180RAM

![MEZZ180RAM](https://github.com/satoshiokue/EMUZ80-Z180RAM/blob/main/imgs/MEZZ180RAM_1.jpeg)  
Z8S180 16MHz no wait  

![MEZZ180RAM](https://github.com/satoshiokue/EMUZ80-Z180RAM/blob/main/imgs/MEZZ180RAM_2.jpeg)  
試作基板MEZZ180+MEZ80RAMによるテストの様子  

電脳伝説さん(@vintagechips)のEMUZ80が出力するZ80 CPU制御信号をメザニンボードで組み替え、Z180と64kB RAMを動作させることができます。  
RAMの制御信号とIOアクセスのWAIT信号をPICのCLC(Configurable Logic Cell)機能で作成しています。  
電源が入るとPICはZ180を停止させ、ROMデータをRAMに転送します。データ転送完了後、バスの所有権をZ180に渡してリセットを解除します。  

Z8S18020VSCとPIC18F47Q83の組み合わせで動作確認しています。  

動作確認済みCPU  
Z8S18020VSC  
Z8018006VSC  

このソースコードは電脳伝説さんのmain.cを元に改変してGPLライセンスに基づいて公開するものです。

## メザニンボード
https://github.com/satoshiokue/MEZZ180RAM 

## 回路図
https://github.com/satoshiokue/MEZZ180RAM/blob/main/MEZZ180RAM.pdf

## ファームウェア

EMUZ80で配布されているフォルダemuz80.X下のmain.cと置き換えて使用してください。
* emuz80_z180ram.c

## クロック周波数による注意

Z180は外部から入力されるクロック周波数を半分にして動作します。  
Z180を6MHzで動作させるにはファームウェアの84行目を12MHzにします。  

```
#define Z180_CLK 12000000UL 	// Z180 clock frequency(Max 16MHz)
```

クロック周波数を5.6MHz以下にする際はファームウェアの149-150行目を確認してください。
```
//while(!RA0);                // /IORQ <5.6MHz  
  while(!RD7);                // /WAIT >=5.6MHz  
```
Z180がIOアドレスのデータリードを完了するタイミングに適した信号を選択してください。  

両方のwhile文をコメントアウトすると16MHzまで動作するようです。  

## アドレスマップ
```
Memory
RAM   0x0000 - 0xFFFF 64kバイト

I/O
通信レジスタ 0x40
制御レジスタ 0x41
```

## PICプログラムの書き込み
EMUZ80技術資料8ページにしたがってPICに適合するファイルを書き込んでください。  

またはArduino UNOを用いてPICを書き込みます。  
https://github.com/satoshiokue/Arduino-PIC-Programmer

PIC18F47Q43 emuz180RAM_xMHz_Q43.hex 

PIC18F47Q83 emuz180RAM_xMHz_Q8x.hex  
PIC18F47Q84 emuz180RAM_xMHz_Q8x.hex  

6MHzファームウェアはPICから12MHzのクロック周波数を与え、Z180の設定はメモリとI/Oのウェイト0、リフレッシュ無効です。  
16MHzファームウェアはZ8S180/L180で追加されたCPU Control Resister(CCR:1FH) の Clock Divide(Bit7)をXTAL/1として、ファームウェアの150行目をコメントアウトしています。PICから与えた16MHzのクロック周波数で動作します。  

## Z80プログラムの格納
インテルHEXデータを配列データ化して配列rom[]に格納すると0x0000に転送されZ180で実行できます。
ROMデータは最大32kBまで転送できますが、現行のファームウェアは8kバイト(0x0000-0x1FFF)転送しています。

## EMUBASICの変更点
EMUZ80用のEMBASICを次の様に変更しました。  

RAMスタートアドレスを8000Hから2000Hに変更  
通信入出力をメモリマップドI/OからI/O空間へ変更  
Z180設定の追加（配列rom[]のコメント参照してください）  

## 謝辞
思い入れのあるCPUを動かすことのできるシンプルで美しいEMUZ80を開発された電脳伝説さんに感謝いたします。

## 参考）EMUZ80
EUMZ80はZ80CPUとPIC18F47Q43のDIP40ピンIC2つで構成されるシンプルなコンピュータです。

![EMUZ80](https://github.com/satoshiokue/EMUZ80-6502/blob/main/imgs/IMG_Z80.jpeg)

電脳伝説 - EMUZ80が完成  
https://vintagechips.wordpress.com/2022/03/05/emuz80_reference  
EMUZ80専用プリント基板 - オレンジピコショップ  
https://store.shopping.yahoo.co.jp/orangepicoshop/pico-a-051.html
