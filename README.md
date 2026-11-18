# Japanese-History-contents-data

日本歴史改方(https://rekikata.jp)の歴史地図帳・記事のデータを保管するリポジトリ。
サイト(jhr-site-astro)の `content/` に Git サブモジュールとして取り込んで読み込む。**データはここで編集する。**

```
map/
├── categories.json   カテゴリ(遺跡・城・街道・宿場)と、その中のテーマの並び順
└── themes/           テーマごとに1ファイル(ファイル名がテーマの id = URL の /map/<id>/)
    ├── kaizuka.json
    └── kofun.json ...
article/
├── genres.json       歴史記事のジャンルと、その並び順
└── <ジャンルの id>/<記事の id>/index.md   記事(1記事1フォルダ。URL は /article/<ジャンル>/<記事>/)
```

## map/categories.json

配列。並び順がそのまま TOP・メニューの順になる。

| 項目        | 内容                                             |
| ----------- | ------------------------------------------------ |
| id          | カテゴリの英字名                                 |
| name        | カテゴリ名                                       |
| description | TOP でカテゴリ名の下に出す説明(1文)              |
| themes      | 入れるテーマの id の配列(並び順がそのまま表示順) |

## map/themes/<id>.json

| 項目        | 内容                                                                       |
| ----------- | -------------------------------------------------------------------------- |
| category    | 入れるカテゴリの id                                                        |
| name        | テーマ名                                                                   |
| description | テーマの説明(1〜2文)                                                       |
| unit        | 件数の単位(省略すると「か所」)。例: 「城」「宿」                           |
| numbered    | `true` にすると番号付きのテーマ。史跡に `no` を書き、番号順に並べる        |
| route       | `true` にすると地図上でピンを番号順に点線でつなぐ(`numbered` と一緒に使う) |
| spots       | 史跡の配列。**空のテーマは「準備中」**になり、ページは作られない           |

### spots の項目

| 項目        | 内容                                                                                 |
| ----------- | ------------------------------------------------------------------------------------ |
| id          | URL(`?spot=ID`)に使う英字名                                                          |
| name        | 史跡名                                                                               |
| no          | テーマの中での番号(`numbered` のテーマだけ)                                          |
| era         | 時代名。サイトの `src/data/eras.ts` の name と同じにすると時代ページへのリンクが付く |
| period      | 築かれた・使われた時期(表示用)                                                       |
| address     | 所在地                                                                               |
| description | 説明                                                                                 |
| lat / lng   | 緯度・経度(Google マップで場所を右クリックすると表示される値)                        |

同じ史跡を複数のテーマに載せるときは、それぞれのテーマのファイルに書く。

## article/genres.json

配列。並び順がそのまま一覧・メニューの順になる。ここにないジャンルのフォルダに記事を置くと、サイトのビルドがエラーで止まる。

| 項目        | 内容                               |
| ----------- | ---------------------------------- |
| id          | ジャンルの英字名(フォルダ名と同じ) |
| name        | ジャンル名                         |
| description | ジャンルの説明(1文)                |

## article/<ジャンルの id>/<記事の id>/index.md

記事の本文(Markdown)。画像は同じフォルダに置き、本文から `![説明](./photo.jpg)` で参照する。先頭に次の項目を書く。

```md
---
title: 12月14日 赤穂浪士の討ち入り
description: 一覧と検索結果に出す説明
publishedAt: 2026-12-14
era: 江戸時代
tags: [事件]
draft: true
---
```

| 項目        | 必須 | 内容                                                                                 |
| ----------- | ---- | ------------------------------------------------------------------------------------ |
| title       | ○    | タイトル                                                                             |
| description | ○    | 説明                                                                                 |
| publishedAt | ○    | 公開日。一覧はこの日付の新しい順                                                     |
| updatedAt   |      | 更新日                                                                               |
| era         |      | 時代名。サイトの `src/data/eras.ts` の name と同じにすると、時代年表へのリンクが付く |
| tags        |      | 分野(政治・外交・経済・文化・合戦・事件・災害)。分野の年表へのリンクになる           |
| draft       |      | `true` のあいだはサイトに出さない(サイトの開発サーバーでは表示される)                |

## データをサイトに反映する手順

データ(このリポジトリ)を変えたら、次の順に行います。サイト側のコマンドは、サイトのフォルダ `jhr-site-astro/` で実行します。

### 1. このリポジトリで変更を push する

```
cd "/Users/masafumi/Documents/03_Web制作ファイル/01_sites/日本歴史改方/Japanese-History-contents-data"
git add .
git commit -m "[update] 〇〇を追加"
git push
```

### 2. サイトの `content/` を最新にする

```
cd "/Users/masafumi/Documents/03_Web制作ファイル/01_sites/日本歴史改方/jhr-site-astro"
git submodule update --remote content
```

### 3. サイト側で取り込んだ版を記録して push する

```
git add content
git commit -m "[update] 歴史地図帳のデータを更新"
git push
```

`main` に push すると、Cloudflare が再ビルドして公開されます。

## よく使うコマンド(サイト側)

| 目的                                                       | コマンド                                                    |
| ---------------------------------------------------------- | ----------------------------------------------------------- |
| 最新のデータを取り込む                                     | `git submodule update --remote content`                     |
| 記録された版に戻す(pull のあと、`content/` が空・古いとき) | `git submodule update --init`                               |
| 取り込んでいる版を確認する                                 | `git submodule status`                                      |
| サイトを新しく clone する                                  | `git clone --recurse-submodules <サイトのリポジトリの URL>` |

`git submodule status` の先頭に `+` が付いているときは、記録された版と `content/` の中身がずれています。上の手順3で記録し直すか、`git submodule update --init` で戻します。

## 注意

- `content/` の中のファイルは直接編集しない。直すのはこのリポジトリで行う。
- データを push しただけでは、サイトは変わらない。手順2・3まで行ったあと、サイト側の `main` に反映されて初めて公開される。
