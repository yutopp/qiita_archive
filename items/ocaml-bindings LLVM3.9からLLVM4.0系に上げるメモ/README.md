# 背景

LLVM4.0より、C APIからAttributes関係の関数が消えました。[LLVM 4.0.0 Release Notes](http://releases.llvm.org/4.0.0/docs/ReleaseNotes.html#non-comprehensive-list-of-changes-in-this-release)
それに伴って、LLVMのocaml-bindingsのAttribute関係の定義もごそっと消えてしまいました。そこで、この記事では自分が調べたLLVM3.9からLLVM4.0への書き換え方法のメモを書きます。

# 本題
まず、該当コミットは[これ](https://github.com/llvm-mirror/llvm/commit/18c0ee263804d2d736405b66898309583ee8385e)です。

コレを見るに、今まで`Llvm.Attribute.Alwaysinline`のように指定していた値は、`Llvm.create_enum_attr ctx "alwaysinline" 0L`で代替のものが作れるようです(`ctx: Llvm.llcontext`)。

また、`Llvm.add_function_attr f Llvm.Attribute.Alwaysinline`のように呼び出していた関数は、`Llvm.add_function_attr f (Llvm.create_enum_attr ctx "alwaysinline" 0L) Llvm.AttrIndex.Function`のように置き換えれば良さそうです。

# まとめ

関数への`alwaysinline`指定の方法の変化点をまとめます。

|LLVM3.9|LLVM4.0|変更点|
|:--|:--|:--|
|`Llvm.Attribute.Alwaysinline`|`Llvm.create_enum_attr ctx "alwaysinline" 0L`<br>または<br>`let k = Llvm.enum_attr_kind "alwaysinline" in Llvm.attr_of_repr ctx (Llvm.AttrRepr.Enum(k, 0L))`|今まで属性は定数で定義されていましたが、`alwaysinline`のような`llattrkind型`の属性の種類と、その種類に引数を指定した`llattribute型`の属性を分けて定義できるようになったようです。|
|`Llvm.add_function_attr f Llvm.Attribute.Alwaysinline`|`L.add_function_attr f (Llvm.create_enum_attr ctx "alwaysinline" 0L) Llvm.AttrIndex.Function`|定数ではなく生成した`llattrbute型`の値を渡すようになったことと、`Llvm.AttrIndex.*`によってどの場所に属性が付くか明示的に指定できるようになったようです。|

上記以外にも、属性が明確に`llattribute型`の値として扱われるようになったことで、`is_enum_attr`などの属性に対する関数などが追加されているようです。

