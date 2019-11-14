# このシェーダの構成

このシェーダは曇った状態のガラスを再現すべく制作された機構です。
厳密には現実のガラスのように光の屈折を行う処理は含めていませんので、あくまでも平面のみの環境で動作することを想定しています。

このシェーダは

* 1個の平行投影カメラ
* 1枚のRender Texture
* 1枚のCustom Render Texture
* Plane (またはQuad)

から構成され、スクリプトを使用せず、テクスチャをメモリ代わりに使い滑らかな曇りガラス処理を実現しています。

図にすると以下のような構成です。

![sg_detail.png](images/sg_detail.png)

ユーザが弄るパラメータは赤文字で記載されている `SGHitBufferWriter` シェーダと `SmokeGlass` シェーダとなります※。

prefabを使って導入する場合、それぞれを設定してあるマテリアルは、

|SGHitBufferWriter|`Assets/SGSmokeGlass/Material/SGHitBufferWriterMat.mat`|
|SmokeGlass|`Assets/SGSmokeGlass/Material/SmokeGlassMat.mat`|

となります。

※：<br>
`SmokeGlass`は無料版でもついてくるものです。<br>
Boothで購入された方は `SGFrostStandard` など霜のついた曇りガラスのマテリアルなどもあります。