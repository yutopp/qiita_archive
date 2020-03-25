[Why doesn't ADL find function templates? - StackOverflow](http://stackoverflow.com/questions/2953684/why-doesnt-adl-find-function-templates)

oh...知らなかった...
とりあえず，VC11で書いたコードをg++にかけたとき引っかかったのでメモ．
実際のコードを単純にしすぎたために，意図が分からないサンプルコードになってます・・・．

# 間違ってる感じのコード
下のコードはコンパイルエラーになります．

```c++
#include <iostream>

namespace test
{
  namespace ns_a
  {
    struct A {};

    template<typename T>
    T func( A* )
    {
      std::cout << "A" << std::endl;
      return T();
    }
  } // namespace ns_a

  struct Chikuwa {};

  template<typename T>
  Chikuwa call()
  {
    return func<Chikuwa>( static_cast<T*>(0) );
  }
} // namespace test

int main()
{
  test::call<test::ns_a::A>();
}
```


次に，下のコードのように，VC11では何故かvoid*を取る関数を同じ名前空間に入れておくとコンパイルが通り，かつ意図した通りに動作します．(g++ではコンパイルエラー)

```c++
#include <iostream>

namespace test
{
  namespace ns_a
  {
    struct A {};

    template<typename T>
    T func( A* )
    {
      std::cout << "A" << std::endl;
      return T();
    }
  } // namespace ns_a

  struct Chikuwa {};

  template<typename T>
  Chikuwa call()
  {
    return func<Chikuwa>( static_cast<T*>(0) );
  }

  template<typename T>
  void func( void* ) { /*dummy*/}
} // namespace test

int main()
{
  test::call<test::ns_a::A>();
}
```


# 正解？
StackOverflowに書いてあるように，usingで可視化するとどちらのコンパイラでも上手くいきます．

```c++
#include <iostream>

namespace test
{
  namespace ns_a
  {
    struct A {};

    template<typename T>
    T func( A* )
    {
      std::cout << "A" << std::endl;
      return T();
    }
  } // namespace ns_a

  class Chikuwa {};

  template<typename T>
  Chikuwa call()
  {
    using ns_a::func;
    return func<Chikuwa>( static_cast<T*>(0) );
  }
} // namespace test

int main()
{
  test::call<test::ns_a::A>();
}
```


でも一々using使うのも大変なので，こんなので使うことにしました．どちらのコンパイラでも通ります．(C++11)

```c++
#include <type_traits> 
#include <iostream>

namespace test
{
  namespace ns_a
  {
    struct A {};

    template<typename T>
    T func( T*, A* )
    {
      std::cout << "A" << std::endl;
      return T();
    }
  } // namespace ns_a

  struct Chikuwa {};

  template<typename T>
  Chikuwa call()
  {
    typedef std::add_pointer<Chikuwa>::type cp;
    return func( static_cast<cp>(0), static_cast<T*>(0) );
  }
} // namespace test

int main()
{
  test::call<test::ns_a::A>();
}
```

# で
実際こんなコードを書くのはどうなのだろうか・・・と悩む最近でございました．
(Qiitaに一度投稿してみたかったノリで書きました．)