これは[ML Advent Calendar 2017](https://adventar.org/calendars/2142)の21日目の記事です。

OCamlで書いている自作言語の中間表現の紹介をしようと思ったのですが[中途半端なところまでしか作れなかった](https://github.com/yutopp/rill_ir)ので、今回はOCamlで書かれたErlangのBEAM FormatとErlang External Term Formatのパーサである[Obeam(御-BEAM)](https://github.com/yutopp/obeam)の紹介をしたいと思います。

人間がErlangを書く際、[Dialyzer](http://erlang.org/doc/man/dialyzer.html)という静的解析ツールにお世話になることが多いと思います。主に事前に静的に型検査を行える便利なツールで、動的型付き言語に後から静的型検査を追加したということで既存の型指定がないコードもなるべく解析できるような型システムを積んでいます (オススメ記事→
 [Dialyzer の型推論アルゴリズムについて](https://qiita.com/amutake/items/a9a8c37568ec3cb9c028))。
これが非常に重たくてツラいというのがあります。OTP20でかなり改善された気がするのですが、それでもデカいアプリケーションを食わせると時間がかかります。

そこで、「とりあえずErlangではなくOCamlで書いてみたら速くなるんでは？」という雑な会話をした結果、「とりあえずOCamlでErlang関係のファイルフォーマットがいい感じに読めるようにしておくか」と作り始めたのがObeamです。OCaml本当にこういうコード書くのに向いてて好きです。

# Obeam [(repo)](https://github.com/yutopp/obeam)

OCamlで書かれており、Erlang/OTP18〜20の[BEAM Format](http://rnyingma.synrc.com/publications/cat/Functional%20Languages/Erlang/BEAM.pdf)と[Erlang External Term Format](http://erlang.org/doc/apps/erts/erl_ext_dist.html)、[Abstract Format](http://erlang.org/doc/apps/erts/absform.html)をパースできることを目指して作っています。読みはオービームです。

ビルドツールに[jbuilder](https://github.com/janestreet/jbuilder)を使うようにして(さっきしました)、バイナリのパターンマッチには[ppx_bitstring](https://github.com/xguerin/ppx_bitstring)を使って自然に書けるようにしています。ppx_bitstringはいつの間にはjbuilderで使えるようになっているようですね…。

このようなErlangコードを`erlc +debug_info test01.erl`でビルドしたBEAMを与えると、

```erlang:test01.erl
-module(test01).

-export([g/0, h/1]).

f() ->
    0.

g() ->
    10.

-spec h(integer()) -> string().
h(_) ->
    "abcdefg".
```

以下のようなAbsFormまで返してくれます。

```ocaml
(Abstract_format.AbstractCode
   (Abstract_format.ModDecl
      [(Abstract_format.AttrFile (1, "test/test01.erl", 1));
        (Abstract_format.AttrMod (1, "test01"));
        (Abstract_format.AttrExport (3, [("g", 0); ("h", 1)]));
        (Abstract_format.DeclFun (5, "f", 0,
           [(Abstract_format.ClsFun (5, [], None,
               (Abstract_format.ExprBody
                  [(Abstract_format.ExprLit
                      (Abstract_format.LitInteger (6, 0)))
                    ])
               ))
             ]
           ));
        (Abstract_format.DeclFun (8, "g", 0,
           [(Abstract_format.ClsFun (8, [], None,
               (Abstract_format.ExprBody
                  [(Abstract_format.ExprLit
                      (Abstract_format.LitInteger (9, 10)))
                    ])
               ))
             ]
           ));
        (Abstract_format.SpecFun (11, None, "h", 1,
           [(Abstract_format.TyFun (11,
               (Abstract_format.TyProduct (11,
                  [(Abstract_format.TyPredef (11, "integer", []))])),
               (Abstract_format.TyPredef (11, "string", []))))
             ]
           ));
        (Abstract_format.DeclFun (12, "h", 1,
           [(Abstract_format.ClsFun (12, [(Abstract_format.PatUniversal 12)],
               None,
               (Abstract_format.ExprBody
                  [(Abstract_format.ExprLit
                      (Abstract_format.LitString (13, "abcdefg")))
                    ])
               ))
             ]
           ));
        Abstract_format.FormEof]))
```

いまこの記事を書きながら足りなすぎる実装を足しているので年末くらいにはいい感じに色々パースできると思います(ゆるしてください)。

# Tylang [(repo)](https://github.com/yutopp/tylang)

ついでに載せておきます。こちらはフロントはそのままErlangで書かれており、生成したBEAMファイルをObeamを使った型検査器に食わせるためのレイヤを作ろうとした途中のものです。

Erlang/OTPのパーサのコードのみを公式から引っ張ってきて、specではなくclauseごとに細かく型指定を書けるような構文の拡張を入れてあり、それ以外はErlang標準機能を使ってBEAMにしています。多少Obeamの構文の拡張が必要なことを除けば単純なものです(そのかわりParseTransformなどいろいろ犠牲になっていますが…)。

仕組みとしては[alpaca-lang](https://github.com/alpaca-lang/alpaca)というErlangVM上で動作するML風Erlangを参考にしました。

推論を頑張らせるとしんどいので、関数のシグネチャの型は明示的に書かせることで、ある程度軽量に細かい型検査が出来るのではないかなと雑に思ったまま作っていく気持ちです。

# おわりに

ML成分が薄くてとても申し訳ない気持ちでいっぱいなのですが、とりあえずErlangのBEAMファイルをパースできるライブラリをOCamlで作っているので、日々Erlangを書いておりDialyzerが多少つらいという方、ぜひ型検査器やAltErlangを…何卒… :pray:
