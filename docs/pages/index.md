# SkyWay Unity SDK

SkyWay Unity SDK(以下、Unity SDK)は Unity デバイス用のアプリケーションから SkyWay を利用するための SDK です。  
Unity のネイティブなアプリケーションに SkyWay を組み込むことで、デバイス同士やブラウザとのリアルタイム通信を実現できます。

\tableofcontents

## インストール方法

プラットフォームによって異なります。
各クイックスタートをご参照ください。

- [クイックスタート(Windows)](https://skyway.ntt.com/ja/docs/user-guide/unity-sdk/quickstart-windows/)
- [クイックスタート(Android)](https://skyway.ntt.com/ja/docs/user-guide/unity-sdk/quickstart-android/)

## 用語の解説

SkyWayを利用する際に登場する用語について解説します。

### Room

Roomは通話を行うグループの単位であり、共通の Room に参加したユーザー同士で通話を行うことができます。  
RoomにはP2P Room と SFU Room の 2 種類の Room があり、どちらかを選択して利用できます。  
P2P Room は少人数(4人程度)向けで、SFU Room は多人数向けです。  

### Member  

Room に参加しているユーザーのことを Member と呼びます。  

### Stream  

Room 上で送受信できるメディアのことを Stream といいます。  
音声、映像、データの3種類の Stream を利用できます。  

### Publish  

Member が Stream を Room に公開することを Publish といいます。  
Stream を Publish すると Room 上に Stream に対応する Publication というリソースが作成されます。  

### Subscribe  

Member が Room 上の Publication を受信することを Subscribe といいます。  
Subscribe をすると Room 上に Subscription というリソースが作成されます。  
Publication を Subscribe した Member は Subscription を通じて Stream にアクセスし映像や音声を受信できます。

## 基本機能の解説

Unity SDKの基本的な機能について解説します。

### SWContext

SkyWay利用の開始や終了を行うためのクラスです。
ログレベルなど、アプリケーション全体に影響するオプションの設定も担っています。

#### SkyWay利用の開始

```cs
await SWContext.Setup(this, authToken);
```

事前に SkyWay Auth Token の取得が必要になります。

#### Auth Tokenの更新

Auth Tokenには期限を設定する必要があり、期限が切れるとSkyWayを利用できなくなります。  
そのため、Auth Tokenが失効する前に更新してください。

```cs
// Auth Tokenの更新が必要なタイミングを検知
SWContext.onTokenRefreshingNeededHandler = () => {
    // 新しいAuth Tokenに更新
    SWContext.UpdateAuthToken("");
};
```

#### SkyWay利用の終了

```cs
SWContext.Dispose();
```

### Room

通話グループである Room の作成/取得および操作を行うことができます。
Room には P2PRoom と SFURoom の 2 種類が存在し、API は基本的に共通しています。

#### Roomの作成・取得

Roomの作成時にnameを指定することができます。
他のクライアントからは同じnameを指定してRoomを取得することができます。

```cs
// Roomの作成
var room = await SWP2PRoom.Create(roomName);

// Roomの取得
var room = await SWP2PRoom.Find(roomName /* または id = roomId */);

// Roomの取得を試み、存在しなければ作成
var room = await SWP2PRoom.FindOrCreate(roomName);
```

#### Metadata の更新

Room の Metadata を更新することができます

```cs
// metadataの取得
var metadata = room.Metadata;

// metadataの更新
room.UpdateMetadata("metadata");
```

#### Member の Room への参加

Room に参加すると LocalRoomMember インスタンスを取得できます。  
`SWRoomMemberOptions`には`name`と`metadata`の設定が可能です。  

```cs
var localRoomMember = room.Join(new SWRoomMemberOptions());
```

### LocalRoomMember

Memberの操作を行うことができます。
具体的には、Metadataの更新、Stream の Publish/Subscribe を行うことが出来ます。

#### Metadata の更新

Member の Metadata を更新することができます

```cs
// metadataの取得
var metadata = member.Metadata;

// metadataの更新
member.UpdateMetadata("metadata");
```

#### Stream の Publish

Room に Stream を Publish することができます。  
また、Room 上の Stream の Publication を Unpublish することができます。  
このとき、対象のPublicationに対する Subscription が自動的に Unsubscribe されます。

```cs
// Publish
var publication = member.Publish(stream);

// Unpublish
member.Unpublish(publication);
```

#### Publication の Subscribe

Room 上の Publication を Subscribe することができます。

```cs
// Subscribe
var subscription = member.Subscribe(publication);

// Unsubscribe
member.Unsubscribe(subscription);
```

### Source/Stream
Sourceは、SDKから各種メディアへアクセスするAPIを提供します。
また、Publish可能なStream を作成することが出来ます。

#### SourceからStreamの作成

```cs
// マイク音声のソース
var audioSource = new SWAudioSource();
var stream = audioSource.CreateStream();

// カメラテクスチャのソース(Androidのみ対応)
var videoSource = new SWVideoSource(sendingTexture);
var stream = videoSource.CreateStream();

// データソース
var dataSource = new SWDataSource();
var stream = dataSource.CreateStream();
```

#### AudioStreamの再生

受信した音声を再生することができます。

```cs
stream.SetAudioSource(audioSource);
audioSource.Play();
```

#### VideoStreamの再生

受信した映像を再生することができます。

```cs
stream.onTextureHandler = (texture) => {
    // 受信した映像テクスチャをRaw Imageに描画します
    renderingImage.texture = texture;
};
```

#### DataStreamによるデータの送受信

任意のデータの送信ができます。（SFU Room は非対応です）

```cs
// データの送信
stream.Write("hello")

// データの受信
stream.OnDataHandler = (data) => {
    Debug.Log("data: " + data);
};
```

### RoomPublication

Publication の情報の参照と操作ができます。

#### Metadata の取得・更新

PublicationのMetadata を取得・更新することができます。

```cs
// metadataの取得
var metadata = publication.Metadata

// metadataの更新
publication.UpdateMetadata("metadata")
```

#### ミュート

Publication に紐ついた映像や音声などの配信を一時停止（ミュート）することができます

```cs
// ミュート
publication.Disable()

// ミュートの解除
publication.Enable()
```

### RoomSubscription

Subscription の情報の参照と Subscription の操作ができます

#### Stream の参照

Subscription から映像/音声/データの Stream を参照できます。

```cs
if (stream is SWRemoteVideoStream videoStream) {
    videoStream.onTextureHandler = (texture) => {
        renderingImage.texture = texture;
    };
} else if (stream is SWRemoteAudioStream audioStream) {
    audioStream.SetAudioSource(audioSource);
    audioSource.Play();
} else if (stream is SWRemoteDataStream dataStream) {
    dataStream.OnDataHandler = (data) => {
        Debug.Log("data: " + data);
    };
}
```
