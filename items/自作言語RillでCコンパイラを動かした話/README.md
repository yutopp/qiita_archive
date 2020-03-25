この記事は [第2のドワンゴ Advent Calendar 2016 21日目](http://qiita.com/advent-calendar/2016/dwango2) の記事です。
おはようございます。[@yutopp](https://twitter.com/yutopp) といいます。毎日Erlangを書いたり書かなかったりしています。

今回は、自作言語でCコンパイラを動かした話を書きます。

# 自作言語Rillとは
まず自作言語の紹介をします。自作言語は`Rill`という名前で、別名`文鳥言語`といいます。リポジトリは以下です。
[GitHub - yutopp/rill: a programming language for 文鳥](https://github.com/yutopp/rill)

OCamlとLLVMを用いて開発されており、C++やD言語、Rustを参考にしつつ、安全かついい感じに楽しくプログラミングをするための言語を目指しています。コンパイル時メタプログラミングがしたいがために色々な機能を盛り込んだり、C++14からOCamlに全書き換えなどしていた結果、未だ安定版がリリースできてない状態です。(つらい)

開発は非常に活発(自称)で、3日経つと構文も意味論も変わるレベルで試行錯誤が行われています。バグが大量にあったり、コンパイルエラーが分かりにくかったりと荒削りさしかない状態ですが、そのうちいい感じに使えるように安定させていきたいと考えています。(はやく自作言語で全てを書きたい気持ちがあります)

リポジトリの`test/compilable`と`examples`ディレクトリの下にコード片があります。
また、インストール方法などは`README.md`に記載してあります。基本、リポジトリの下で`opam`を叩くだけで入ります。

# Cコンパイラを動かすトリック

「Cコンパイラを動かした」の時点で **あっ・・・(察し)** と思われた方も多いと思いますが、今回 8cc と ELVM を利用させて頂きました。

[GitHub - shinh/elvm: EsoLangVM Compiler Infrastructure](https://github.com/shinh/elvm)
[GitHub - rui314/8cc: A Small C Compiler](https://github.com/rui314/8cc)

ELVM は`ELVM IR`と呼ばれる中間形式から色々な()バックエンドに変換を行うコンパイラ基盤です。この中間形式が非常にシンプルで、バックエンドの追加も簡単に行えるのがとても面白いです。すでにelispやvim scriptやtexバックエンドなど*「ん？」*となるものが沢山あります。

8cc はセルフホスト可能なミニマルなCコンパイラです。ELVMではELVM IRを出力できる改造版8ccが同梱されています。

ELVM/8cc の記事のおすすめを紹介します。

- [ELVM Compiler Infrastructure - 兼雑記](http://shinh.hatenablog.com/entry/2016/10/18/095437)
- [ELVM で C コンパイラをポーティングしてみよう（Vim script 編） - はやくプログラムになりたい](http://rhysd.hatenablog.com/entry/2016/12/02/030511)
- [ELVM Compiler Infrastructureバックエンド作成のすゝめ - hak7a3が書き残す何か](http://hak7a3.hatenablog.com/entry/2016/11/13/153418)

# 今回何をしたのか

以上の通り、今回は ELVM に 自作言語Rill用のバックエンドを追加しました。これにより、任意のELVMコードを自作言語のコードに変換することができるようになりました。世界が広がる！
![graph_1.png](https://qiita-image-store.s3.amazonaws.com/0/2746/ea076b12-0a90-a2a0-28f8-fb38a46439b2.png)

Rillバックエンドを追加したELVMのforkは以下になります。
[GitHub - yutopp/elvm](https://github.com/yutopp/elvm/tree/feature/rill)
リポジトリの下で`make rill`をする必要があります。

また、最終成果物もこちらに置いておきます。
[GitHub - yutopp/8cc.rill](https://github.com/yutopp/8cc.rill)

## 実際に動かしてみる

以下のCのソースコードを自作言語製のCコンパイラでビルドするのが目的です。

```c:hello.c
int putchar(int c);
void print_str(const char* p);
int puts(const char* p);

int main() {
    puts("hello Rill world!");
    return 0;
}

void print_str(const char* p) {
  for (; *p; p++)
    putchar(*p);
}

int puts(const char* p) {
    print_str(p);
    putchar('\n');
}
```

### 単純なテスト

それでは最初に、ELVMのRillバックエンドの生成物をRillでビルドして実行できることを確かめてみましょう。ここで用いる`8cc/elc`は通常のCソースコードから生成されたものです。

```bash:ビルド(ELVM側)
./out/8cc -S -I. -Ilibc -o hello.eir hello.c
./out/elc -rill hello.eir > hello.rill
```
これで、CソースコードからRillソースコードへの変換が行われ、いい感じに4000行程度のRillのソースコードが生成されます。Rillでビルドと実行もしてみます。

```bash:ビルド(Rill側)
rillc hello.rill 
./hello
```

```shell-session:実行結果
hello Rill world!
```

実行できました！

### 8ccとelcをRillに移植

8ccとelcはセルフホストできる素晴らしい処理系なので、8ccとelc自体を8ccとelcでビルドしてRillに移植してしまいましょう。
![graph_2.png](https://qiita-image-store.s3.amazonaws.com/0/2746/c4f9e548-5632-5294-f041-9da6401b0e33.png)

変換の流れは先程と同じです。以下のコマンドで、`8cc.rill`と`elc.rill`を作ります。

```bash:ビルド(ELVM側)
./out/8cc -S -I. -Ilibc -I8cc/include -o 8cc.eir out/8cc.c
./out/elc -rill 8cc.eir > 8cc.rill
./out/8cc -S -I. -Ilibc -I8cc/include -o elc.eir out/elc.c
./out/elc -rill elc.eir > elc.rill
```

それでは、ビルドしていきます。

```bash:ビルド(Rill側)
rillc 8cc.rill -o 8cc-rill
rillc elc.rill -o elc-rill
```

2〜3分でビルドすることができます。実行が終わると、`8cc-rill`と`elc-rill`が生成されているはずです。
ついに、自作言語でビルドしたCコンパイラが出来ました！

### Rill製8ccとelcでCソースコードをビルド

それでは、最終的に自作言語処理系で生成したCコンパイラでCソースコードをコンパイルしてみます。
![graph_3.png](https://qiita-image-store.s3.amazonaws.com/0/2746/08548b92-51e3-398b-10e8-e070c6093ad3.png)

```bash:ビルド(Rill側)
cat hello.c | ./8cc-rill > hello_rill.eir
(echo x86 && cat hello_rill.eir) | ./elc-rill > hello_rill.out
chmod +x hello_rill.out
```
`実行結果`
![Screenshot_2016-12-20_16-53-53.png](https://qiita-image-store.s3.amazonaws.com/0/2746/49664d6f-b5f5-700a-f57b-5ed95dce77e9.png)

キャーイ！ ビルドも実行もできました！嬉しいですね。

## 苦労した点

今回苦労したのはコンパイラの処理速度でした。ELVMバックエンドで生成したソースコードは非常に大きくなる場合があり、最大で13万行程度になるものも存在しました。一方で自作言語で用いてきたテストケースは非常に小さいものばかりで、コンパイラにこの大きいコードを食わせると案の定処理が遅く使い物にならなくなってしまいました。OCamlで開発しているのにかかわらず、perfやgdbをゴリっと使ってボトルネックを見つけては潰すのを繰り返したり、内部実装のテンプレートのインスタンス化の部分をひたすらキャッシュするように変更して、まあまあな速さでビルドできるようにはなりましたが、まだ難ありという感じです…。

また、今回ビルトインの関数が足りないケースが無限に発生したために、ELVMのバックエンドのテストが落ちる→自作言語の標準ライブラリを拡張する→テストを走らせる、という感じで自作言語側とバックエンド側の開発を行き来するのも大変でした。結構アドホックなコードを入れてしまったのでこれからじわじわ直していきます。

## これからやりたいこと

この自作言語の特徴はコンパイル時実行なのですが、バグまみれのため今回は動かすまでにたどり着かなかったです…　はやくこの言語でもコンパイル時Cコンパイルやりたいですね。

# まとめ

今回はELVM/8ccに自作言語用のバックエンドを追加して、自作言語で書かれたCコンパイラを生成してみました。
とにかく楽しかったです(小並感) 言語自作もめちゃくちゃ楽しいのでみなさんも言語開発して、どうぞ。

以上です。ありがとうございました。

### おまけ
この記事用に書いた言語のコードの一部は京都で書かれました。はやく本物の特別になりたい…
![kyoto.jpg](https://qiita-image-store.s3.amazonaws.com/0/2746/b1920118-c3a2-024c-6437-da7e4571776d.jpeg)
[響け！ユーフォニアム2 [最新話無料] - ニコニコチャンネル:アニメ](http://ch.nicovideo.jp/anime-eupho02)

