# ip62_opamp_handson
ISHI-kai OPAMP Hands-On (2026/08/15-16)
# ターゲットとなる設計指標の決定
決めるのは以下の指標
* GB積 $GBW$
* スルーレート $SR$
* 外部負荷容量 $C_L = 40pF$

これらのうち、外部負荷容量については自前の測定環境依存による寄生容量値 40pF として考える。
## GB積
GB積はGainとBandwidthをかけた指標です。

計算上はAmplitude(単位V/V, またはスカラー)とFrequency(単位Hz)です。(GBW = A × F)

例えば $GBW = 1 MHz$ として考えると…

  $$f = 1kHz\longrightarrow A=1000V/V\longrightarrow G=60dB$$
  
  $$f = 100kHz\longrightarrow A=10V/V\longrightarrow G=20dB$$

みたいなゲイン特性を持っていて、AC解析上のボード線図でも上記のような周波数特性が得られるイメージ。

## スルーレート
インパルスなどの急峻な入力した際に出力される電圧波形の最大傾きで単位はV/s。

スルーレートについてはGBWに対して最低値が計算できる

ミラー補償による２段オペアンプのユニティゲイン角周波数は

$$\omega_{unity} \simeq \frac{g_{m1}} {C_C}$$

より、

$$GBW = f_{unity} \simeq \frac{g_{m1}} {2\pi C_C}$$

スルーレートはミラー補償容量 $C_C$を充放電する電流、つまりテール電流 $I_5$に律速される

$$SR \simeq \frac{I_5} {C_C}$$

入力トランジスタM1, M2に流れる電流がテール電流の半分、 $I_5/2$ と考えると

$$g_{m1} \simeq \frac{I_5} {V_{OV1}} \simeq \frac{SRC_C} {V_{OV1}}$$

またGBWとgm1の式から

$$g_{m1} \simeq 2\pi C_CGBW$$

以上より

$$SR \simeq 2\pi \cdot GBW \cdot V_{OV1}$$

