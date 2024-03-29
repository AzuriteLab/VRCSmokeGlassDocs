# 曇りガラスを一から導入する

ここでは曇りガラスシェーダを使い1からセットアップする方法を紹介します。

1枚だけワールドに配置する場合は、セットアップ済みの`prefab`を`Hierarchy`にドロップすることで簡単につかえますが、
2枚目以降を作成する場合は`Render Texture`など自分で新たに作成する必要があります（もちろん2枚つかうことは重くなりますので非推奨ではあります）
この解説はその方向けのため、最初に配置する方は [曇りガラスをprefabから導入する](setup_with_prefab.md) を参照してください。

1. [RenderTextureを作成する](#1RenderTextureを作成する)
2. [平行投影カメラを作成し、RenderTextureを割り当てる](#2平行投影カメラを作成しRenderTextureを割り当てる) 
3. [CustomRenderTexture用マテリアルを作成する](#3CustomRenderTexture用マテリアルを作成する)
4. [CustomRenderTextureを作成し、マテリアルを割り当てる](#4CustomRenderTextureを作成しマテリアルを割り当てる)
5. [表示用のシェーダのマテリアルを作成する](#5表示用のシェーダのマテリアルを作成する)
6. [表示用の板をカメラと同じサイズで用意、位置合わせをする](#6表示用の板をカメラと同じサイズで用意位置合わせをする)

また、無料配布のunitypackageにはprefabが一つ含まれています。
それをHierarchyにドロップすることで初めから使える状態になります。

細かいガラスの調整に関しては以下の逆引きリファレンスを読んでください。

* [平行投影カメラとTolerance、表示面の調整](tips.md#平行投影カメラとTolerance、表示面の調整)
* [ガラスのマスクによる縦横比の調整](tips.md#ガラスのマスクによる縦横比の調整)
* [拭った状態から戻るスピードの調整](tips.md#拭った状態から戻るスピードの調整)
* [ガラスの模様と光沢表現](tips.md#ガラスの模様と光沢表現)

---

## 1.RenderTextureを作成する

まず、深度情報（ガラスからどれだけ近いのか）を取得するために `Render Texture` を作成します。

`Projectウィンドウ`で右クリックし、メニューを開き、`Create` -> `Render Texture` の順番で選択して `Render Texure` を作成してください。

![render_texture](images/create_rendertex.png)

そして、作成された `Render Texture` を選択し、Inspectorから `Color Format` を `Depth` に変更します。<br>
このとき、`Size` の項目もデフォルトの `256x256` では小さすぎるため、`1024x1024` や `2048x2048` あたりに設定すると良いでしょう。
（もちろんサイズを増やすと負荷が増えます。一応推奨値は `1024x1024` です）

![render_texture_setup](images/setup_rendertex.png)

## 2.平行投影カメラを作成し、RenderTextureを割り当てる

次に深度情報を写すためのカメラを作成します。

`Hierarchy` の空いているスペースで右クリックし、`Camera` を選択してカメラを作成します。

![create_camera](images/create_camera.png)

次に作成されたカメラを選択し、`Hierarchy` に表示された設定を行っていくのですが、ここは項目が多いです。

* `Clear Flags` を `Solid Color` にして `Background` を真っ黒の状態にします
  * 深度は赤で書き込まれるので赤の情報が入っていなければなんでもいいです
* `Projection` を `Orthographic` にします
  * これにより平行投影カメラになります。`Size`についてはガラスの大きさに直結するため、[平行投影カメラとTolerance、表示面の調整](tips.md#平行投影カメラとTolerance、表示面の調整)で解説しています。
* `Clipping Planes` の `Far` を `5` に設定します。
  * `5` はこちらで設定しているデフォルトの値です。
  * シーン上でカメラの箱状のサイズが変わるかと思います。この範囲内に入ったオブジェクトの深度情報を取ることができます。
* `Target Texture` に先ほど作成した `Render Texture` をドロップして設定します。
* `Allow HDR` のチェックボックスを外します。
* `Allow MSAA` のチェックボックスを外します。

![setup_camera](images/setup_camera.png)

この画像のようになっていれば大丈夫なはずです。

## 3.CustomRenderTexture用マテリアルを作成する

お次は作成した平行投影カメラから得た深度情報をガラスの曇り情報へと変換するための `CustomRenderTexture` を作成するのですが、
この `Custom Render Texture` には曇り情報へと変換するためのシェーダを割り当てたマテリアルを作成する必要があります。

`Projectエリア` の空いている部分を右クリックし、 `Create` -> `Material` を選択し、 `Material` を作成します。

![create_material](images/create_material.png)

次に作成した `Material` を選択し、 `Inspector` から設定を行います。
まずはシェーダを選択します。`VRCSmokeGlass` -> `Writer` -> `SGHitBufferWriter` の順で選択してください。<br>
これは `Smoke Glass Hit Buffer Writer` の略で、`曇りガラス当たり判定バッファ書き込み機` の意です。

![set_writershader](images/set_writershader.png)

次にこのシェーダの設定を行っていきます。

* `Render Texture` に先ほど作成した `Render Txture` を指定します。
* `Tolerance` は基本的に規定の値です（specified value）。変更しないでください。
  * これは先ほど指定した `5` mを基準に、カメラ範囲のどれぐらいの割合を当たり判定から除外（マスク）するか、のパーセンテージです。
  * デフォルトであれば、`0.99`。 即ち99%の範囲は除外されるため、 カメラから `0.05` メートル範囲内しか認識されません。
  * この0.05メートル内にガラスをうまいこと配置することで当たったように見せかけるのがこの曇りガラスの仕組みです。
  * 詳しい設定は、[平行投影カメラとTolerance、表示面の調整](tips.md#平行投影カメラとTolerance、表示面の調整) を参照してください。
* `Using blur` は曇っている表現をリアルにしたい場合はonにしてください。ガラスにお絵かきしたい場合はoffがお勧めです。
* `Reset` は曇り状態リセット用です。 `Enable Restoration` をオフにしているときはこれを即座にon/offするボタンを作ってあげると喜ばれます。
* `Enable Restoration` は基本的にonにしておきましょう。これを有効にすることで拭った場所が徐々に曇った状態へ戻ります。

![setup_writer](images/setup_writer.png) 

補間方法など詳しいパラメータの説明は [SGHitBufferWriterシェーダのパラメータ解説](SGBufferWriter.md) を参照してください。

## 4.CustomRenderTextureを作成し、マテリアルを割り当てる

`Render Texture` の時と同じように、 `Projectウィンドウ` の空いている部分を右クリックし、 `Create` -> `Custom Render Texture` の順に選択し、`Custom Render Texture` を作成します。

![create_customrendertex](images/create_customrendertex.png)

そして、作成された `Custom Render Texture` を選択し、`Inspector` にて設定を行います。

* `Size` は `Render Texture`と同じサイズが良いでしょう。（下の画像では `1024x1024` に変更しています）
* `Aniso Level` は `0` に設定します
* `Material` には [3.CustomRenderTexture用マテリアルを作成する](#3CustomRenderTexture用マテリアルを作成する) で作成したマテリアルを割り当てます
  * `Shader Pass` は `UPDATE` にします
* `Initialization Mode` は `OnLoad` に
  * `Source` は `Texture and Color`に、そしてここの `Texture` には先ほど作成した `Render Texture` を指定しましょう
* `Update Mode` は `Realtime` に
  * `Double Buffered` にチェックを入れます
* `Update Dozne` の `+` ボタンを押して一つ追加しておきます

![setup_customrendertex](images/setup_customrendertex.png)

## 5.表示用のシェーダのマテリアルを作成する

これでカメラから深度情報を受け取り、それを曇り情報に変換し、シェーダによってガラスの質感へと変換させる準備ができました！

最後にガラス面に適用するためのシェーダを設定すべくマテリアルを作成します。

同じように `Projectウィンドウ`で右クリックし、メニューを開き、`Create` -> `Material` の順番で選択して `Material` を作成してください。

![smokeglassmat](images/create_smokeglassmat.png)

そして、作成された `Material` を `Inspector` からシェーダの変更とパラメータの設定を行います。

* シェーダを `Standard` から `VRCSmokeGlass` -> `SmokeGlassStandard` に変更
  * 4種類ありますが、今回は裏面表示も行わない通常のStandardシェーダの法則に則ったシェーダを選択します
  * 詳細は [SmokeGlass.md](SmokeGlass.md) のリストを参照してください
* 

## 6.表示用の板をカメラと同じサイズで用意、位置合わせをする

表示のための `Quad` を作成し、前項で作成したマテリアルを割り当てます。
`Quad`を作成するとき、カメラの縦横サイズと同じサイズすることを忘れないでください。

そして、カメラの縦横位置と `Quad` の位置を完全に合わせた上で `Quad` をカメラ範囲の直前に配置します。
当たり判定は先ほど設定した `0.05m` 範囲内しかおこなわれないため、この範囲内に上手いこと設置します。
（何度か試してうまい書き心地になるまでトライしてみてください。CubeやSphereで当たり判定を試して上手く拭えても実際にVR上で確認すると触れていないのに反応してしまったり、微妙だなぁとなることが多いです）

これで曇りガラスの設置は完了となります。
まずは動作確認として、何か適当なCubeやSphereを作成し、ガラス面に当ててみてください。その場所が拭えていれば成功です。
あとはVRChatにログインしつつ納得がいくまで調整を重ねてください。
