---
layout: post
title: "IRKitを使ってHipChatからエアコンを制御する"
date: 2014-02-01 01:55:46 +0900
comments: true
categories: [irkit, hipchat, hubot]
---

どうもはじめまして。Beatrobo竹井([@HideyukiTakei](https://twitter.com/HideyukiTakei))です。

これからBeatroboエンジニアたちの技術的な話をここにまとめていこうと思います。

第一弾はIRKitに関して！

![IRKit](/images/201405/irkit.jpg)

[IRKit](http://getirkit.com/)とは、[@maaash](https://twitter.com/maaash)さんが開発しているWiFi機能の付いたオープンソースな赤外線リモコンデバイスです。[Arduinoのソースコードや基板の回路図も公開](https://github.com/irkit/device)されています。また、IRKitをローカルネットワークから制御するための[Device HTTP API](http://getirkit.com/#IRKit-Device-API)や、インターネット越しに制御するための[Internet HTTP API](http://getirkit.com/#IRKit-Internet-API)も用意されています。

今回はこのIRKitを使って[HipChat](https://www.hipchat.com/‎)からエアコンを制御する方法を順を追ってご紹介します！

---

### 1.リモコンON/OFF時の赤外線信号を取得する

Internet HTTP API の[GET /1/messages](http://getirkit.com/#IRKit-Internet-GET-1-messages)を用いることでリモコンの赤外線信号をJSONで取得することができます。

IRKitはBonjourに対応しているようですので、まずIRKitの名前を取得しましょう。

```
$ dns-sd -B _irkit._tcp
Browsing for _irkit._tcp
DATE: ---Sat 01 Feb 2014---
15:41:50.027  ...STARTING...
Timestamp     A/R    Flags  if Domain               Service Type         Instance Name
15:41:50.555  Add        2   5 local.               _irkit._tcp.         iRKitD1D1
^C
```

コマンド結果で表示されている `iRKitD1D1` (仮名)がIRKitの名前です。また、ドメインが `local` なので、`iRKitD1D1 .local` がIRKitの宛先となります。

Internet HTTP API を使用するためには clientkey, deviceid が必要で、これらを取得するためには clienttoken が必要です。clienttoken は Device HTTP API の[POST /1/keys](http://getirkit.com/#IRKit-Device-POST-keys)で取得できます。

```
$ curl -i "http://iRKitD1D1.local/keys" -d ""
{"clienttoken":"00112233445566778899AABBCCDDEEFF"}
```

ここで得られた clienttoken を使って Internet HTTP API の [POST /keys](http://getirkit.com/#IRKit-Internet-POST-1-keys) で clientkey, deviceid を取得します。

```
$curl -i "http://api.getirkit.com/1/keys" -d "clienttoken= 00112233445566778899AABBCCDDEEFF"
HTTP/1.1 200 OK
Server: ngx_openresty
Date: Tue, 07 Jan 2014 08:46:06 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 94
Connection: keep-alive
X-Content-Type-Options: nosniff

{"deviceid":"0123456789ABCDEF0123456789ABCDEF","clientkey":"FEDCBA9876543210FEDCBA9876543210"}
```

これで Internet HTTP API を使う準備は完了です。

次は赤外線信号の取り込みです。 Internet HTTP API の [GET /1/messages](http://getirkit.com/#IRKit-Internet-GET-1-messages) は最も新しい受信した赤外線信号を返してくれるAPIです。
新しい赤外線信号がIRKitデバイスから届いたらただちにレスポンスを返します。

```
$ curl -i "http://api.getirkit.com/1/messages?clientkey= FEDCBA9876543210FEDCBA9876543210&clear=1"
```

ロングポーリングでレスポンスが返ってこない状態になります。

次に、IRKitにリモコンを向けながら、エアコンONボタンを1度だけ押してみましょう。青色LEDが点滅したら赤外線信号受信ができているようです。

```
HTTP/1.1 200 OK
Server: ngx_openresty
Date: Sat, 01 Feb 2014 07:43:49 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 2506
Connection: keep-alive
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: X-Requested-With
ETag: "-1919363029"
X-Content-Type-Options: nosniff

{"message":{"format":"raw","freq":38,"data":[5408,6648,262,6424,322,1366,382,1232,205,1622,280,1275,395,1413,205,1275,539,1037,619,1111,539,1073,663,968,761,968,619,1232,470,1111,558,1111,619,1073,619,968,619,968,735,1073,663,904,735,968,735,968,735,2626,710,904,904,904,735,904,735,904,815,904,710,968,710,968,710,873,873,873,873,873,761,873,873,873,873,873,761,873,873,873,873,873,873,873,873,873,873,873,873,873,761,873,761,873,761,2537,873,873,873,2537,873,2537,873,2537,873,873,873,2537,873]},"hostname":"IRKitD2C7","deviceid":"32658096E6ED4AE3977E4B6BEFCA5493"}
```

この`message`部分がエアコンONの赤外線信号のJSONです。

---

### 2.Internet HTTP APIを使ってエアコンをONする
さきほど得られたmessageのJSONを使って、試しにエアコンをONにしてみましょう。Internet HTTP API の [POST /1/messages](http://getirkit.com/#IRKit-Internet-POST-1-messages) を使います。

```
$ curl -i "http://api.getirkit.com/1/messages" \
-d "clientkey=FEDCBA9876543210FEDCBA9876543210" \
-d "deviceid=0123456789ABCDEF0123456789ABCDEF" \
-d 'message={"format":"raw","freq":38,"data":[5408,6648,262,6424,322,1366,382,1232,205,1622,280,1275,395,1413,205,1275,539,1037,619,1111,539,1073,663,968,761,968,619,1232,470,1111,558,1111,619,1073,619,968,619,968,735,1073,663,904,735,968,735,968,735,2626,710,904,904,904,735,904,735,904,815,904,710,968,710,968,710,873,873,873,873,873,761,873,873,873,873,873,761,873,873,873,873,873,873,873,873,873,873,873,873,873,761,873,761,873,761,2537,873,873,873,2537,873,2537,873,2537,873,873,873,2537,873]}'
HTTP/1.1 100 Continue

HTTP/1.1 200 OK
Server: ngx_openresty
Date: Sat, 01 Feb 2014 07:47:26 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 0
Connection: keep-alive
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: X-Requested-With
X-Content-Type-Options: nosniff
```

これでエアコンがつくはずです！
次にこのAPIをHipChatから叩けるようにしてみましょう。

---

### 3.HUBOTをローカルで動かす

HipChatからIRKitのInternet HTTP APIを使うために、チャットボット[HUBOT](http://hubot.github.com/)とそのHipChatアダプタ[hubot-hipchat](https://github.com/hipchat/hubot-hipchat)を利用します。今回は自分のMac上でHUBOTを動かしてみましょう。

まず、HUBOTをインストールします。

```
$ npm install --global coffee-script hubot@v2.6.4
```

そしてHUBOTのプロジェクトテンプレートを作成。

```
$ hubot --create hubot
$ cd hubot
$ ls
Procfile               README.md              bin/                   external-scripts.json  hubot-scripts.json     package.json           scripts/
```

hubot-hipchat をインストール

```
$ npm install
$ npm install --save hubot-hipchat
```

初期状態だとRedisを使うようになっているのですが、今回は特に使わないので `hubot-scripts.json` を編集して該当スクリプトを無効にします。

{% codeblock hubot-scripts.json %}
["shipit.coffee"]
{% endcodeblock %}

これでHUBOTの準備は整いました。

次にHipChat側でボット用のアカウントを作成しましょう(HipChatはボットにも月$2かかっちゃうのが残念）。アカウント作成はHipChatのWebサイトから行います。

HUBOTからHipChatのルームにアクセスするためには以下の情報が必要です。

- ボットアカウントのJID: XMPP/Jaberのアカウント名
- パスワード: ボットアカウントを作った時のパスワード
- ルームのJID: ルームは[XMPPのMUC](http://xmpp.org/extensions/xep-0045.html)で実装されているようです

ボットアカウントとルームのJIDは、HipChatのWebページの `My account` -> `Account settings` -> `XMPP/Jabber info` に掲載されています。

![HipChat setting](/images/201405/hipchat_settings.png)

ルームのJIDは、「ルームのXMPP/JabberName@ルームのドメイン」となります。上の画像だと、「****_android@conf.hipchat.com」がルームのJIDとなります。 

上記の3つの情報を元にHUBOTを起動してみましょう。起動するためのスクリプト `run.sh` を以下のように書きます。exportしている情報は上記でゲットしたものを入力してください。

{% codeblock run.sh %}
#!/bin/bash
 
export HUBOT_HIPCHAT_JID="01234_56789@chat.hipchat.com"
export HUBOT_HIPCHAT_PASSWORD="passworddayo"
export HUBOT_HIPCHAT_ROOMS="01234_android@conf.hipchat.com" # ROOM JID
 
bin/hubot --adapter hipchat
{% endcodeblock %}

そして実行！

```
$ bash run.sh
[Sun Feb 02 2014 07:45:59 GMT+0900 (JST)] INFO Connecting HipChat adapter...
[Sun Feb 02 2014 07:46:03 GMT+0900 (JST)] INFO Connected to hipchat.com as @hubot
[Sun Feb 02 2014 07:46:03 GMT+0900 (JST)] WARNING The HUBOT_AUTH_ADMIN environment variable not set
[Sun Feb 02 2014 07:46:04 GMT+0900 (JST)] INFO Joining 01234_android@conf.hipchat.com
```

ルームに以下の表示が出ていれば成功です。

```
[HUBOTアカウントのニックネーム] joined the room
```

---

### 4.HUBOTにエアコン機能を付け加える

HUBOT に hubot-hipchat アダプタを追加することで、HUBOTが参加しているルームに流れてくる文字列を読み取り、コマンドに応じた処理を実行することができるようになりました。

どういうコマンドが来たらどういう処理をするか、ということは `scripts` ディレクトリのスクリプトで定義されています。新しいコマンドとその処理の定義は、こちらの `scripts` ディレクトリにスクリプトを追加することで可能となります。

今回は `air` というエアコンコマンドを実装してみましょう。「@hubot air on」でエアコンをON、「@hubot air off」でエアコンをOFFできるようにするスクリプトは以下です。

{% codeblock air.coffee %}
# Description:
#   Air controller
#
# Commands:
#   hubot air (on|off) - switch air
 
module.exports = (robot) ->
  IRKIT_MESSAGE_API = "http://api.getirkit.com/1/messages"
  CLIENT_KEY = "YOUR_CLIENT_KEY"
  DEVICE_ID = "YOUR_DEVICE_ID"
  MESSAGE_ON = '{"format":"raw","freq":38,"data":[6648,3341,843,・・省略・・,843,2537,843]}' # your ON message
  MESSAGE_OFF = '{"format":"raw","freq":38,"data":[6881,3228,904,・・省略・・,935,2451,935]}' # your OFF message
 
  postIRKit = (msg, json, output) ->
    robot
      .http IRKIT_MESSAGE_API
      .query
        clientkey: CLIENT_KEY
        deviceid: DEVICE_ID
        message: json
      .post (err, res, body) ->
        msg.send output
 
  robot.respond /air (on|off)/i, (msg) ->
    sw = msg.match[2]
    if sw is "on"
      postIRKit msg, MESSAGE_ON, "ON"
    else
      postIRKit msg, MESSAGE_OFF, "OFF"
{% endcodeblock %}

HUBOTのスクリプトは、基本的には以下の形式で反応させたいコマンドを正規表現で指定して、内部に処理を書いていきます。

```
  robot.respond /air (on|off)/i, (msg) ->
```

scripts ディレクトリにはたくさんのスクリプトが入っているので、それらを見れば雰囲気はだいたい把握できると思います！

これで再度

```
$ bash run.sh
```

を実行して、HUBOTがいる部屋で `@hubot air on` とつぶやいてみましょう。

![HUBOT AIR](/images/201405/hipchat_air_on.png)

これで快適エアコンライフ！がんばれば温度もコントロールできそうです。

上記のコードは [GitHub](https://github.com/hideyuki/hubot-hipchat-air) に公開しています。

Beatroboではこのhubotちゃんを Heroku で動かしてます。
もしエアコン大好きな方はぜひBeatroboに遊びに来てください。

ではでは！あでゅ！
