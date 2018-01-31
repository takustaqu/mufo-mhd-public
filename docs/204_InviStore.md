# MuFo Document : InviStore仕様

## 概要

InviStoreは招待（MuFoにさらわれる機能）に関連するデータを管理します。

## プロパティ

ストアのプロパティは全て参照専用です。
参照型(オブジェクト)の中身を変更してもデータは反映されません。

### {InviModel} lastInvi 直近の招待

最後に受け取って、かつ回答していない招待のデータです。
詳細はInviModelの定義を参照してください。

## アクション

### INVI_ENABLE 招待機能を有効にする

招待を受け付ける状態にします。このアクションを実行するには、ログイン済みでなくてはなりません。
このアクションを実行する前に発行された招待がある場合、このアクションを実行すると`INVI_INVITED`イベントで保留されていた招待がすぐに通知されます。
したがって、このアクションは招待を受け取れる状態になった時に初めて実行するようにしてください。

**引数**

|  名前  |  型  |  必須?  |  説明  |
| --- | --- | --- | --- |


### INVI_DISABLE 招待機能を無効にする

招待を受け付けない状態にします。このアクションはいつでも実行できます。
無効にした後で招待が発行された場合、その招待は保留され、次回招待機能が有効になった際に通知されます。

**引数**

|  名前  |  型  |  必須?  |  説明  |
| --- | --- | --- | --- |


### INVI_REQUEST 招待を要求する

招待を要求します。招待機能が有効である必要があります。
招待を要求すると、ランダムなMuFoが招待を発行し、`INVI_INVITED`イベントが発行されます

**引数**

|  名前  |  型  |  必須?  |  説明  |
| --- | --- | --- | --- |

### INVI_ANSWER 招待に応答する

招待に回答します。回答しない場合、次回招待機能を有効にした際に同じ招待が再度通知されてしまうため、必ず何かしらの回答を行うようにしてください。

**引数**

|  名前  |  型  |  必須?  |  説明  |
| --- | --- | --- | --- |
|  inviid  |  string  |  Y  |  回答する招待のID。InviModel.inviid で取得できます |
|  answer  |  string  |  Y  |  `accept`（受け入れる） `deny`（拒否する） `cancel`（アプリ側の判断でユーザに聞かずに招待をキャンセルする） のいずれかを設定します。  |


## イベント

### INVI_INVITED 招待が通知された

招待を受け取った際に発行されるイベントです。

**コールバックの引数**

|  名前  |  型  |  説明  |
| --- | --- | --- |
| invi | InviModel | 招待の内容 |

**例**

```javascript
App.on(App.events.INVI_INVITED,(invi)=>{
  console.log('招待ID = ', invi.inviid);
  console.log('招待先MuFo = ', invi.mufodata);
});
```


## 補足：招待の発行基準 **★要調整★**

招待は以下の基準で発行されます

### 発行タイミングと招待先MuFo・招待人数

+ 新規にMuFoが作成された時：作成されたMuFo・最大5人
+ 新規に曲が追加された時：曲の追加されたMuFo・最大3人
+ 招待が要求された時：ランダムなMuFo・1人

### 招待される人の選定基準

+ 招待が要求された場合、単純に要求したユーザのみに宛てて招待を発行します
+ それ以外の場合、アプリを利用中のユーザから招待の最大人数の3倍の人数を選択し、そのユーザから下記の要件に当てはまるものだけを最大人数以内で招待します
  + 前回の招待発行からの経過時間が60秒以上であること（連続して招待することを避ける）
  + 前回招待したMuFoと今回招待しようとするMuFoが異なること（同じMuFoに連続して招待することを避ける）
+ 対象となるユーザが一人も見つけられなかった場合、招待は発行されません