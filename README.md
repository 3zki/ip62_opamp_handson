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


$$I_5 = SR\cdot C_C$$ 

より、

$$I_5 = 2\times 10^{6}(V/s) \cdot 10\times 10^{-12}(F) = 20 \mu A$$

以下、余談

0.22 という値は 2.2 という値と 10 という値からきています。意味はそれぞれ、

* $2.2\longrightarrow$ : $z \simeq 10GB$かつ $PM \simeq 60deg$ の条件
* $10\longrightarrow$ : RHP零点をGB積(=ユニティゲイン周波数)の10倍のところに置くと仮定したため ($z \simeq 10GB$)

この2.2という値について

位相余裕とは次の極と零点により定義できます

$$PM = 180^\circ - tan^{-1}\left( \frac{GBW}{|p1|} \right) - tan^{-1}\left( \frac{GBW}{|p2|} \right) - tan^{-1}\left( \frac{GBW}{|z|} \right)$$

ここでp1は低周波極でありユニティゲイン周波数から遠い場所にあるとすると $tan^{-1}\left( \frac{GB}{|p1|} \right) = 90^\circ$とみなせるため、

$$PM = 90^\circ - tan^{-1}\left( \frac{GBW}{|p2|} \right) - tan^{-1}\left( \frac{GBW}{|z|} \right)$$

RHP zeroとなる零点は GB積(=ユニティゲイン周波数)の10倍のところに置くと仮定します。 ($z \simeq 10GB$)

すると零点の位相は

$$tan^{-1}\left( \frac{GBW}{|z|} \right) = tan^{-1}(0.1) \simeq 5.71^\circ$$

また、位相余裕を60度と設定したので、 $PM=60^\circ$です。以上から

$$tan^{-1}\left( \frac{GBW}{|p2|} \right) = 90^\circ -60^\circ -5.71^\circ = 24.29^\circ$$

$$\frac{GBW}{|p2|} = tan(24.29^\circ )$$

$$ |p2| = \frac{GBW}{tan(24.29^\circ)} \simeq 2.2GB$$

余談おわり

## 2. 初段OTA: 入力段 M1, M2のサイズ決定

入力トランジスタ M1, M2は

$$gm_{1} = 2\pi \cdot GBW \cdot C_C$$

より、

$$gm_{1} = 2\pi \cdot 1.0\times 10^6 \cdot 10\times 10^{-12} = 62.83\mu S$$

よって、

$$\left( \frac{W}{L} \right) _{1,2} = \frac{{gm_1}^2}{\mu _pC_{ox}\cdot I_5} = 12.17 \longrightarrow 16$$

## 3. 初段OTA: カレントミラー M3,M4のサイズ決定
M3, M4は入力電圧範囲により決定される ($V_{th_1,2}$は正の値 (+0.7V) として式の符号を決定していることに注意)

$$\left( \frac{W}{L} \right) _{3,4} = \frac{I_5}{\mu _nC_{ox}\cdot (V_{in,min} - V_{SS} - V_{th1,2} + V_{th3,4})^2}$$

より、

$$\left( \frac{W}{L} \right) _{3,4} = \frac{20\times 10^{-6}}{61.931\times 10^{-6}\cdot (0.1 - 0 - 0.73047 + 0.794425)^2} = 12.013 \longrightarrow 16$$

第3極はM3,M4のゲート容量Cgs3,4により発生するため悪さをしないか確認する。GB積に対して

$$\left| p_3\right|= \left| \frac{-gm_3}{2C_{gs3}}\right| > 10GBW$$

を満たすか確認する。CgsはW/L, mを設定してOP解析を実施して取得する。負の容量値が出た場合は符号をとって正の値、絶対値とする。

$$gm_3 = \sqrt{2\mu _nC_{ox}\cdot \left( \frac{W}{L} \right) _{3,4} \cdot I_{3,4} } = \sqrt{2\cdot 61.931\times 10^{-6}\cdot 16\cdot 10\times10^{-6}} = 140.78\mu S$$

$$\left| p_3\right| = \frac{140.78\times 10^{-6}}{2\cdot 3.9\times 10^{-13}} = 1.8\times 10^8 > 10^7 ;  Okay!$$


ここでM1~M4のサイズバランスが明らかに悪い(W/Lが1000を超えたり、1を割ったなど)場合や、第3極の条件に違反した場合はスルーレートやGBWの設定を見直す。自分ならスルーレートを違う値にしてみる。

1. ~ 3. の手順は何回か反復してみる。

## 4. 初段OTA: テール電流　M5 のサイズ決定
M5は飽和領域で動作するのに必要なドレインソース間電圧 Vds5(sat) に律速される。

$$\left( \frac{W}{L} \right) _{5} = \frac{2I_5}{\mu _pC_{ox}\cdot {V_{ds5(sat)}}^2}$$

Vds5satは最大入力電圧により決定. $V_{th5}$は正の値 (+0.7V) として式の符号を決定していることに注意

$$V_{ds5(sat)} = V_{DD}-V_{in(max)}+\sqrt{\frac{I_5}{\left( \frac{W}{L} \right) _{1,2}\mu _nC_{ox}}}-V_{th5}$$

$V_{in(max)}$は適当でもよいがVovを用いた計算で求めた. $V_{th5}$は正の値 (+0.7V) として式の符号を決定していることに注意

$$V_{in(max)} = V_{DD}-V_{th5}-V_{ov5} = 5.0 - 0.73047 - 0.1 = 4.16 V$$

これを代入して

$$V_{ds5(sat)} = 5.0 - 4.16 + \sqrt{\frac{20\times 10^{-6}}{16\cdot 16.213\times 10^{-6}}} - 0.73047 = 0.377 V$$

よって、

$$\left( \frac{W}{L} \right) _{5} = \frac{2\cdot 20\times 10^{-6}}{16.213\times 10^{-6}\cdot {0.377}^2} = 17.29 \longrightarrow 16$$

M5のサイズに関して、Vds5satに余裕を持たせるのであれば計算値以下の値を採用する。

計算値以上の値とするとVin(max)が上昇するが、これはVds5satが下がる (=Vov5が小さくなる!)ので設計が厳しくなります。　

以上とするか以下とするかは自由ということで。

## 5. ソース接地増幅段: 入力段M7の決定
M7のサイズはgm7をどうするかで決まる。

$$\left( \frac{W}{L} \right) _{7} = \left( \frac{W}{L} \right) _{3,4} \cdot \frac{gm_7}{gm_3} $$

入力段M7に関しては零点をGB積の10倍のところに置くことにしたので、

$$z = \frac{gm_7}{C_c} = 10\frac{gm_1}{C_C} = 10GBW$$

よって、

$$ gm_7 = 10gm_1 = 628.32\mu S$$

がz=10GBWの目安。 よって

$$\left( \frac{W}{L} \right) _{7} =16 \cdot \frac{628.32\mu S}{140.78 \mu S} = 71.41 \longrightarrow 80 $$


