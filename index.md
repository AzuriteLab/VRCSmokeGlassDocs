# VRCSmokeGlass Documantation

ver : 2019.11.25.0

## はじめに

曇りガラスシェーダ『VRCSmokeGlass』のセットアップと使い方を解説していきます。
このシェーダは1台のカメラ、1枚のRenderTexture、1枚のCustomRenderTextureを利用しているため、
セットアップ方法が少々複雑なため、この手のギミックを導入し慣れている玄人向けです。

## 目次

* [このシェーダの構成（はじめに読んでください）](detail.md)
* [曇りガラスをprefabから導入する](setup_with_prefab.md)
* [曇りガラスを一から導入する](setup_ext.md)
* *シェーダリファレンス*
  * [SGHitBufferWriterシェーダ](SGBufferWriter.md)
  * [SmokeGlassシェーダ](SmokeGlass.md)
* [逆引きリファレンス](tips.md)

## 簡単な説明

unityprojectをimportしたら `VRCSmokeGlass` フォルダがAssets直下に作成されます。<br>
`VRCSmokeGlass` フォルダ直下にある `.prefab` がありますので、それをHierarchyにドロップするだけでシーンに曇りガラスが配置されます
（Hierarchyに新しく出来たオブジェクトがカメラで、その下に入っているオブジェクトが表示面です）。

詳細は [曇りガラスをprefabから導入する](setup_with_prefab.md) を。
2枚以上ワールドに配置したい場合（非推奨）は [曇りガラスを一から導入する](setup_ext.md) を参照してください。

ガラスの大きさや比率を変更するには普通の拡大/縮小では行えないため、[逆引きリファレンス](tips.md) を参照してください。

## 諸注意

* シェーダのコード、エディタスクリプトのコードはMITライセンスで提供しています
* このシェーダは確実に全ての環境で動作することは保証しません
* このシェーダはVRChatのみで動作することを想定しています
* Unityの更新やVRChatの更新によって使えなくなる可能性があります
* *玄人向けのシステムであるため、手順ミスによる動作不良などのサポートは致しません*

---

Copylight(c) 2019 Azurite

