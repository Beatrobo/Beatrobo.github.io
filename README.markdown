# tech.beatrobo.com

## 概要
- [Octopress](http://octopress.org/) を利用
- BitBucket のプライベートリポジトリでソースコード管理、GitHub Pages でページ公開（パブリックリポジトリ）

## 環境構築
```
$ git clone git@git-infra.plugair.com:beatrobo-infra/beatrobo.github.io.git Beatrobo.github.io
$ cd Beatrobo.github.io
$ gem install bundler
$ rbenv rehash
$ bundle install --path bundle
```


## ブログの書き方

### authorの登録
このブログでは、各ポストでauthorをロボットで表示します。

そのため最初に_config.ymlにauthor情報の追加とロボット画像の追加が必要です。

#### _config.ymlにauthor情報の追加
`_config.yml` の一番下にauthor情報を追加します。

`new_user_name` のキーと、その中の name, email, fb, tw のキーを情報を追記してね。以下例です。

```
# authors info
authors:
  hide:
    name: Hideyuki Takei
    email: hide@beatrobo.com
    fb: https://www.facebook.com/hidep
    tw: https://twitter.com/HideyukiTakei
  new_user_name:
    name: New User Full Name
    email: newuser@beatrobo.com
    fb: https://www.facebook.com/new_user_fb_name
    tw: https://twitter.com/new_user_tw_name
```

#### ロボット画像の追加
上記のauthor情報 `new_user_name` で登録したあとは、ロボットの画像を登録してね。ファイル名は `new_user_name` で指定したものにしよう。画像の場所は `source/images/robo/***.png` です。


### 記事作成
```
$ bundle exec rake new_post["ブログの英語タイトル(URLに使われる)"]
```

### プレビュー&編集
```
$ bundle exec rake preview
Starting to watch source with Jekyll and Compass. Starting Rack on port 4000
Configuration from /Users/hide/beatrobo/src/Beatrobo.github.io/_config.yml
[2014-05-04 01:59:29] INFO  WEBrick 1.3.1
[2014-05-04 01:59:29] INFO  ruby 2.1.1 (2014-02-24) [x86_64-darwin13.0]
[2014-05-04 01:59:29] INFO  WEBrick::HTTPServer#start: pid=49818 port=4000
Auto-regenerating enabled: source -> public
[2014-05-04 01:59:29] regeneration: 94 files changed
>>> Change detected at 01:59:30 to: screen.scss
identical public/stylesheets/screen.css 

Dear developers making use of FSSM in your projects,
FSSM is essentially dead at this point. Further development will
be taking place in the new shared guard/listen project. Please
let us know if you need help transitioning! ^_^b
- Travis Tilley

>>> Compass is watching for changes. Press Ctrl-C to Stop.
```

そして `http://localhost:4000/` にアクセス！

ソースコードを変更したら自動的にコンパイルが走ります。

### GitHub Pageにデプロイ
```
$ bundle exec rake gen_deploy
```

### ソースコードをBitBucketにPush
```
$ git add .
$ git commit -m "unko"
$ git push -u bitbucket source
```

### TIPS

#### 非公開で保存
`published: true` を記事の頭に追加。記事として表示されなくなる。

#### 記事中に「続きを読む」をつける
「続きを読む」を付けたいところに `<!-- more -->` をつけましょう。

#### 画像の置き場所
`source/images/201405` のような感じで、月ごとにディレクトリを作って画像を格納。すると `/images/201405/irkit.jpg` という感じの相対URLで参照できるようになる。

そして、markdownでは以下のように指定して画像を貼ることができる。

```
{% img /images/201405/irkit.jpg 500 %}
```

### よく使うタグ
#### Gistタグ

Syntax

```
{% gist gist_id [filename] %}
```

Example

```
{% gist d64bbee03f6da64a42e9 air.coffee %}
```

[Gist Tag](http://octopress.org/docs/plugins/gist-tag/)


#### コードを貼るタグ

Syntax
 
 ```
 {% codeblock [lang:language] [title] [url] [link text] [start:#] [mark:#,#-#] [linenos:false] %}
code snippet
{% endcodeblock %}
 ```
 
 Example
 
 ```
 {% codeblock run.sh %}
#!/bin/bash
 
export HUBOT_HIPCHAT_JID="01234_56789@chat.hipchat.com"
export HUBOT_HIPCHAT_PASSWORD="passworddayo"
export HUBOT_HIPCHAT_ROOMS="01234_android@conf.hipchat.com" # ROOM JID
 
bin/hubot --adapter hipchat
{% endcodeblock %}
 ```
 
 [Codeblock](http://octopress.org/docs/plugins/codeblock/)

#### 画像タグ
Syntax

```
{% img [class names] /path/to/image [width] [height] [title text [alt text]] %}
```

Example 

```
{% img http://placekitten.com/890/280 %}
{% img left http://placekitten.com/320/250 Place Kitten #2 %}
{% img right http://placekitten.com/300/500 150 250 Place Kitten #3 %}
{% img right http://placekitten.com/300/500 150 250 'Place Kitten #4' 'An image of a very cute kitten' %}
```

[Image Tag](http://octopress.org/docs/plugins/image-tag/)


### Markdownの書き方
- [GitHub Flavored Markdown](https://help.github.com/articles/github-flavored-markdown)


