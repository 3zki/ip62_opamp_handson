# ip62_opamp_handson
ISHI会 OPAMPハンズオン (2026/08/15-16)

提示された資料を全部無視して設計しました

# ターゲットとなる設計指標の決定
決めるのは以下の指標
* GB積 $GBW$
* スルーレート $SR$
* 電源電圧 $VDD = 5V$ (IP62プロセス標準電圧)
* 最小入力電圧 $Vin(min) = 0.1V$ (本当に適当)
* 外部負荷容量 $C_L = 40pF$

これらのうち、外部負荷容量については自前の測定環境依存による寄生容量値 40pF として考える
## GB積
GB積はGainとBandwidthをかけた指標です

計算上はAmplitude(単位V/V, またはスカラー)とFrequency(単位Hz)です。(GBW = A × F)

例えば $GBW = 1 MHz$ として考えると…

  $$f = 1kHz\longrightarrow A=1000V/V\longrightarrow G=60dB$$
  
  $$f = 100kHz\longrightarrow A=10V/V\longrightarrow G=20dB$$

みたいなゲイン特性を持っていて、AC解析上のボード線図でも上記のような周波数特性が得られるイメージ

## スルーレート
インパルスなどの急峻な入力した際に出力される電圧波形の最大傾きで単位はV/s

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

実設計上は上記の式で得られる値よりも十分に高いSRを設定する

$V_{OV1}=0.1V$として置いてもよいが、M1,M2のサイズが大きすぎてM3,M4のサイズが小さすぎるといったアンバランスな設計値になる

M1~M4がいい感じのバランスになるようなSRを後付けで設定してあげるとGood

# トランジスタの特性を調べた

Vds = 2.5V, Vgs = 1.25V , W = 10um, L=5um でOPをとった

uCoxの計算式

$$uC_{ox} = \frac{L}{W} \cdot \frac{gm}{v_{gs}-v_{th}}$$ 

単位: $(A/V^2)$

## PMOS (L=5um)

| W/L | Vgs | gmp | Vthp | upCox |
| ---- | ---- | ---- | ---- | ---- |
| 2 | 1.25 | 16.846 uS | 0.73047 | 16.213 uA/V^2 |

## NMOS (L=5um)
| W/L | Vgs | gmn | Vthn | unCox |
| ---- | ---- | ---- | ---- | ---- |
| 2 | 1.25 | 56.428 uS | 0.794425 | 61.931 uA/V^2 |

# 設計してみた

* GB積 $GBW = 1MHz$ 　（AC解析やった時にGBWがわかりやすいので）
* スルーレート $SR = 2V/us$ (いい感じのSRを後付けで決定した)
* 電源電圧 $VDD = 5V$ (IP62プロセス標準電圧)
* 最小入力電圧 $Vin(min) = 0.1V$ (本当に適当)
* 外部負荷容量 $C_L = 40pF$
* 位相余裕 $PM = 60 deg$

## 1. 位相補償容量とテール電流の決定

60°を得るためには

$$C_C > \frac{2.2}{10} \cdot C_L$$

が必要なので

$$C_C > 8.8pF$$

ここでは $C_C = 10pF$ としました。

* $2.2\longrightarrow PM \simeq 60deg$の条件
* $10\longrightarrow g_{m7} = 10g_{m2}$の条件

これはあとで解説します

$$I_5 = SR\cdot C_C$$ 

より、

$$I_5 = 2\times 10^{6}(V/s) \cdot 10\times 10^{-12}(F) = 20 \mu A$$

## 2. 
