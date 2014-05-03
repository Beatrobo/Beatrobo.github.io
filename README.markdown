# tech.beatrobo.com

## 概要
- [Octopress](http://octopress.org/)を利用
- BitBucketのプライベートリポジトリでソースコード管理、GitHub Pagesでページ公開（パブリックリポジトリ）

## 環境構築
```
$ git clone git@git-infra.plugair.com:beatrobo-infra/beatrobo.github.io.git Beatrobo.github.io
$ cd Beatrobo.github.io
$ gem install bundler
$ rbenv rehash
$ bundle install --path bundle
$ bundle exec rake install
```