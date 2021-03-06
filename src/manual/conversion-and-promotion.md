`[](# [Conversion and Promotion](@id conversion-and-promotion))
# [変換と昇格](@id conversion-and-promotion)

```@raw html
<!--
Julia has a system for promoting arguments of mathematical operators to a common type, which has
been mentioned in various other sections, including [Integers and Floating-Point Numbers](@ref),
[Mathematical Operations and Elementary Functions](@ref), [Types](@ref man-types), and [Methods](@ref).
In this section, we explain how this promotion system works, as well as how to extend it to new
types and apply it to functions besides built-in mathematical operators. Traditionally, programming
languages fall into two camps with respect to promotion of arithmetic arguments:
-->
```
[整数と浮動小数点数](@ref)、[算術処理と基本的な関数](@ref)、[型](@ref man-types)、[メソッド](@ref)、その他のさまざまなセクションに記述しているように、Juliaには算術演算子の引数を共通の型に昇格するシステムが備わっています。
このセクションでは、この昇格システムが動作するしくみや、昇格システムを新しい型に拡張して、組込みの算術演算子や関数でも動作させる方法について説明します。
従来、プログラミング言語は算術演算の引数がどのように昇格するかによって、2つの陣営に分類されています。


```@raw html
<!--
  * **Automatic promotion for built-in arithmetic types and operators.** In most languages, built-in
    numeric types, when used as operands to arithmetic operators with infix syntax, such as `+`,
    `-`, `*`, and `/`, are automatically promoted to a common type to produce the expected results.
    C, Java, Perl, and Python, to name a few, all correctly compute the sum `1 + 1.5` as the floating-point
    value `2.5`, even though one of the operands to `+` is an integer. These systems are convenient
    and designed carefully enough that they are generally all-but-invisible to the programmer: hardly
    anyone consciously thinks of this promotion taking place when writing such an expression, but
    compilers and interpreters must perform conversion before addition since integers and floating-point
    values cannot be added as-is. Complex rules for such automatic conversions are thus inevitably
    part of specifications and implementations for such languages.
-->
```
  *  **組込みの数値型や算術演算子の自動昇格** 
  ほとんどの言語では、組込みの数値型が、`+`、 `-`、`*`、`/`などの中置記法の算術演算の被演算子として使われるときは、自動的に共通の型に昇格してから、計算結果が算出されます。
  少し例を挙げると、 C、Java、Perl、Pythonなどはすべて、`1 + 1.5`の合計を、浮動小数点値の`2.5`として正しく計算することができますが、`+`の被演算子の一方は整数です。
  こういうシステムは便利であり、通常はプログラマには、ほとんど見えないように慎重に設計されています。
  このような式を書くときに昇格が起こっていると意識する人はほとんどいませんが、コンパイラやインタプリタは、足し算を行う前に変換を必ずおこないます。
  整数と浮動小数点数はそのままでは足せないからです。このような自動変換の複雑な規則は、必然的にこの陣営の言語では仕様や実装の一部となります。

```@raw html
<!--
  * **No automatic promotion.** This camp includes Ada and ML -- very "strict" statically typed languages.
    In these languages, every conversion must be explicitly specified by the programmer. Thus, the
    example expression `1 + 1.5` would be a compilation error in both Ada and ML. Instead one must
    write `real(1) + 1.5`, explicitly converting the integer `1` to a floating-point value before
    performing addition. Explicit conversion everywhere is so inconvenient, however, that even Ada
    has some degree of automatic conversion: integer literals are promoted to the expected integer
    type automatically, and floating-point literals are similarly promoted to appropriate floating-point
    types.
-->
```

   * **非自動昇格**この陣営にはAdaとMLなどの非常に「厳密」な静的に型付けされた言語があります。こういった言語では、すべての変換をプログラマが明示的に指定する必要があります。したがって、例に挙げた`1 + 1.5`という式は、AdaやMLでは共にコンパイルエラーが生じます。これを避けるには`real(1) + 1.5`のように書いて、足し算を実行する前に整数`1`を浮動小数点数に明示的に変換する必要があります。しかし、常に明示的に変換しなければならないのは不便なので、Adaでさえもある程度の自動変換が行われます。整数リテラルは想定通り整数型に自動的に昇格され、浮動小数点リテラルも同様に適切な浮動小数点型に昇格されます。



```@raw html
<!--
In a sense, Julia falls into the "no automatic promotion" category: mathematical operators are
just functions with special syntax, and the arguments of functions are never automatically converted.
However, one may observe that applying mathematical operations to a wide variety of mixed argument
types is just an extreme case of polymorphic multiple dispatch -- something which Julia's dispatch
and type systems are particularly well-suited to handle. "Automatic" promotion of mathematical
operands simply emerges as a special application: Julia comes with pre-defined catch-all dispatch
rules for mathematical operators, invoked when no specific implementation exists for some combination
of operand types. These catch-all rules first promote all operands to a common type using user-definable
promotion rules, and then invoke a specialized implementation of the operator in question for
the resulting values, now of the same type. User-defined types can easily participate in this
promotion system by defining methods for conversion to and from other types, and providing a handful
of promotion rules defining what types they should promote to when mixed with other types.
-->
```

ある意味、Juliaは「非自動昇格」の陣営に分類されるでしょう。
Juliaでは、算術演算子は特殊な構文を持つ関数に過ぎず、関数の引数は決して自動的に変換されません。
しかし、様々な型の混合した引数に算術演算を適用することは、多相的な多重ディスパッチの極端な事例に過ぎません。
これは型によるディスパッチをおこなうJuliaのシステムに非常に適しています。
算術演算でおこる被演算子の「自動」昇格は、特殊な適用がおこるだけです。
Juliaには、算術演算子を全捕捉するディスパッチ規則が事前に定義されており、被演算子の型の組み合わせに対して特化した実装が存在しないときに呼び出されます。
この全捕捉規則は、まずユーザーの定義可能な昇格規則を利用してすべての被演算子を共通の型に昇格し、こうして同一となった型に特化した演算子の実装を呼び出し、演算結果を算出します。
ユーザーの定義した型もこの昇格システムに簡単に追加できます。
ユーザー定義型に対して、他の型と相互に変換するメソッドを定義し、他の型が混在する場合にどの型に昇格するかを定義する少数の昇格規則を決めてやればいいのです。


`[](## Conversion)
## 変換

```@raw html
<!--
The standard way to obtain a value of a certain type `T` is to call the type's constructor, `T(x)`.
However, there are cases where it's convenient to convert a value from one type to another
without the programmer asking for it explicitly.
One example is assigning a value into an array: if `A` is a `Vector{Float64}`, the expression
`A[1] = 2` should work by automatically converting the `2` from `Int` to `Float64`, and
storing the result in the array.
This is done via the `convert` function.

The `convert` function generally takes two arguments: the first is a type object and the second is
a value to convert to that type. The returned value is the value converted to an instance of given type.
The simplest way to understand this function is to see it in action:
-->
```

特定の型`T`の値を得る標準的な方法は、型のコンストラクタ`T(x)`を呼び出すことです。
しかし、値をある型から別の型へ、プログラマが明示的に指定しなくても、便利に変換できる場合があります。
例えば、配列に値を代入する場合です。
`A`が`Vector{Float64}`の時、`A[1] = 2`という式は、自動的に値`2`を`Int`から`Float64`に変換し、その結果を配列に格納します。
これは、関数`convert`を通じて行われます。


この`convert`関数は通常、2つの引数をとります。1番目は型オブジェクトで、2番目はその型に変換する値です。
戻り値は指定した型のインスタンスに変換された値です。
この関数を理解する最も簡単な方法は、実際の動作を見ることです。


```jldoctest
julia> x = 12
12

julia> typeof(x)
Int64

julia> convert(UInt8, x)
0x0c

julia> typeof(ans)
UInt8

julia> convert(AbstractFloat, x)
12.0

julia> typeof(ans)
Float64

julia> a = Any[1 2 3; 4 5 6]
2×3 Array{Any,2}:
 1  2  3
 4  5  6

julia> convert(Array{Float64}, a)
2×3 Array{Float64,2}:
 1.0  2.0  3.0
 4.0  5.0  6.0
```

```@raw html
<!--
Conversion isn't always possible, in which case a no method error is thrown indicating that `convert`
doesn't know how to perform the requested conversion:
-->
```
変換は常に可能であるとは限りません。
そんな時は、メソッドがないことを示すエラーが投げられ、`convert`関数が要求された変換の実行方法が分からないことを通知します。


```jldoctest
julia> convert(AbstractFloat, "foo")
ERROR: MethodError: Cannot `convert` an object of type String to an object of type AbstractFloat
[...]
```

```@raw html
<!--
Some languages consider parsing strings as numbers or formatting numbers as strings to be conversions
(many dynamic languages will even perform conversion for you automatically), however Julia does
not: even though some strings can be parsed as numbers, most strings are not valid representations
of numbers, and only a very limited subset of them are. Therefore in Julia the dedicated `parse`
function must be used to perform this operation, making it more explicit.
-->
```

言語の中には文字列を解析して数値とみなしたり、書式付きの数値を変換して文字列とみなしたりするものがありますが、
（多くの動的言語ではさらに自動的に変換まで行います）、Juliaはそうではありません。
文字列の中には数値として解析できるものもありますが、ほとんどの文字列は数値の有効な表現ではなく、
非常に限られた一部が当てはまるだけです。
そのため、Juliaでは、こういった操作は、専用の関数`parse()` を使って明示的に行う必要があります。


`[](### When is `convert` called?)
### `変換`が呼ばれるのはいつ?


```@raw html
<!--
The following language constructs call `convert`:

  * Assigning to an array converts to the array's element type.
  * Assigning to a field of an object converts to the declared type of the field.
  * Constructing an object with `new` converts to the object's declared field types.
  * Assigning to a variable with a declared type (e.g. `local x::T`) converts to that type.
  * A function with a declared return type converts its return value to that type.
  * Passing a value to `ccall` converts it to the corresponding argument type.
-->
```

以下のような言語の構文では`convert`が呼び出されます。

  * 配列を配列の要素の型に変換して代入する。
  * オブジェクトのフィールドを宣言したフィールドの型に変換して代入する。
  * `new`を使ったオブジェクトの生成で宣言したオブジェクトの型に変換する。
  * 変数の代入で型を宣言し(例えば `local x::T`) その型に変換する。
  * 関数で戻り値の型を宣言し、戻り値をその型に変換する。
  * `ccall`に値を渡し、対応する引数の型に変換する。


`[](### Conversion vs. Construction)
### 変換 vs. 生成

```@raw html
<!--
Note that the behavior of `convert(T, x)` appears to be nearly identical to `T(x)`.
Indeed, it usually is.
However, there is a key semantic difference: since `convert` can be called implicitly,
its methods are restricted to cases that are considered "safe" or "unsurprising".
`convert` will only convert between types that represent the same basic kind of thing
(e.g. different representations of numbers, or different string encodings).
It is also usually lossless; converting a value to a different type and back again
should result in the exact same value.

There are four general kinds of cases where constructors differ from `convert`:
-->
```
注意点があります。
`convert(T, x)`の挙動は`T(x)`とほとんど同じで、実際に大抵の場合同じです。
しかし、セマンティック上の重要な違いがあります。
`convert`は暗黙裏に呼び出されるので、このメソッドの使用は、「安全」で「驚かない」場合のみに制限されます。
`convert`は、基本的には同じ種類の表現どうしの間で変換を行います。
（例えば、数字の異なる表現や文字列の異なるエンコーディングなど）
また、通常は損失は起こりません。別の型に変換してから元に戻すと元の値ちょうど等しい値に戻ります。

コンストラクタと`convert`が異なる４種類の一般の場合があります。


`[](#### Constructors for types unrelated to their arguments)
#### 引数と無関係な型のコンストラクタ

```@raw html
<!--
Some constructors don't implement the concept of "conversion".
For example, `Timer(2)` creates a 2-second timer, which is not really a
"conversion" from an integer to a timer.
-->
```
コンストラクタの中には、「変換」という概念を実装していないものがあります。
例えば、`Timer(2)`は２秒のタイマーを生成し、実際に整数をタイマーに「変換する」わけではありません。


`[](#### Mutable collections)
#### 可変コレクション

```@raw html
<!--
`convert(T, x)` is expected to return the original `x` if `x` is already of type `T`.
In contrast, if `T` is a mutable collection type then `T(x)` should always make a new
collection (copying elements from `x`).
-->
```

`convert(T, x)`は`x`が既に型`T`である時、元の`x`を返しますが、
`T`が可変コレクションの場合、`T`は常に新しいコレクションを生成します。（`x`の要素をコピーします）


`[](#### Wrapper types)
#### ラッパー型

```@raw html
<!--
For some types which "wrap" other values, the constructor may wrap its argument inside
a new object even if it is already of the requested type.
For example `Some(x)` wraps `x` to indicate that a value is present (in a context
where the result might be a `Some` or `nothing`).
However, `x` itself might be the object `Some(y)`, in which case the result is
`Some(Some(y))`, with two levels of wrapping.
`convert(Some, x)`, on the other hand, would just return `x` since it is already
a `Some`.
-->
```
他の値を「ラップする」型では、コンストラクタは引数を新しいオブジェクトの中にラップします。
引数が既に要求された型であった場合でさえもです。
例えば、`Some(x)`は`x`をラップし、値が存在することを（結果が`Some`か`nothing`になるという文脈で）示します。
しかし、`x`自体が`Some(y)`出会った場合、結果は`Some(Some(y))`となり、二重にラップされます。
一方`convert(Some, x)`は単なる`x`を返します。`x`が既に`Some`だったからです。



`[](#### Constructors that don't return instances of their own type)
#### 自身と同じ型のインスタンスを返さないコンストラクタ

```@raw html
<!--
In *very rare* cases it might make sense for the constructor `T(x)` to return
an object not of type `T`.
This could happen if a wrapper type is its own inverse (e.g. `Flip(Flip(x)) === x`),
or to support an old calling syntax for backwards compatibility when a library is
restructured.
But `convert(T, x)` should always return a value of type `T`.
-->
```
**非常に稀に** コンストラクタ`T(x)`が型が`T`ではないオブジェクトを正常であっても返す場合があります。
これは、ラッパーの型が自身の逆写像(つまり`Flip(Flip(x)) === x`)だったり、
ライブラリが再構成された時に、後方互換性のために、古い構文が呼び出しに対応したりする場合に起こります。
しかし、`convert(T, x)`は常に型`T`の値を返します。


`[](### Defining New Conversions)
### 新しい変換の定義

```@raw html
<!--
When defining a new type, initially all ways of creating it should be defined as
constructors.
If it becomes clear that implicit conversion would be useful, and that some
constructors meet the above "safety" criteria, then `convert` methods can be added.
These methods are typically quite simple, as they only need to call the appropriate
constructor.
Such a definition might look like this:
-->
```
新しい型を定義する時は、まず、すべての型を生成する方法を、コンストラクタとして定義すべきです。
暗黙の変換が有用だとわかり、コンストラクタが上記の「安全」の条件を満たす時は`convert`メソッドを追加しても構いません。
普通は、メソッドはとても単純です。適切なコンストラクタを呼ぶだけでいいからです。
そういった定義は、こんな感じになります。


```julia
convert(::Type{MyType}, x) = MyType(x)
```

```@raw html
<!--
The type of the first argument of this method is a [singleton type](@ref man-singleton-types),
`Type{MyType}`, the only instance of which is `MyType`. Thus, this method is only invoked
when the first argument is the type value `MyType`. Notice the syntax used for the first
argument: the argument name is omitted prior to the `::` symbol, and only the type is given.
This is the syntax in Julia for a function argument whose type is specified but whose value
does not need to be referenced by name. In this example, since the type is a singleton, we
already know its value without referring to an argument name.
-->
```

このメソッドの1番目の引数の型は[シングルトン型]（@ ref man-singleton-types）の`Type{MyType}`で、この型のインスタンスは`MyType`だけです。
したがって、このメソッドは、最初の引数が型の値を示す`MyType`である場合にのみ呼び出されます。
最初の引数の構文に注目してください。
`::`という記号の前にあるはずの引数名は省略され、型だけが指定されています。
これはJuliaの関数の構文で、引数の型は指定するけれども、引数の値は関数本体でまったく使わない時に用います。
この例では、型はシングルトンであるため、引数の名前で参照しなくてもその値がわかります。

```@raw html
<!--
All instances of some abstract types are by default considered "sufficiently similar"
that a universal `convert` definition is provided in Julia Base.
For example, this definition states that it's valid to `convert` any `Number` type to
any other by calling a 1-argument constructor:
-->
```
ある種の抽象型のインスタンスすべては、デフォルトでは「十分似ている」とみなされて、
JuliaのBaseライブラリの中で普遍的な`convert`が定義されています。
例えば、任意の`Number`型から他の`Number`型への有効な`convert`の定義を、
１引数のコンストラクタを呼び出すことによって行っています。



```julia
convert(::Type{T}, x::Number) where {T<:Number} = T(x)
```

```@raw html
<!--
This means that new `Number` types only need to define constructors, since this
definition will handle `convert` for them.
An identity conversion is also provided to handle the case where the argument is
already of the requested type:
-->
```

これが意味するのは、新しい`Number`型にはコンストラクタを定義するだけ良いということです。
`convert`の定義が扱うのは、コンストラクタだけだからです。
恒等変換も引数がすでに要望される型である時に扱えます。


```julia
convert(::Type{T}, x::T) where {T<:Number} = x
```

```@raw html
<!--
Similar definitions exist for `AbstractString`, `AbstractArray`, and `AbstractDict`.
-->
```
同様の定義が`AbstractString`、`AbstractArray`、`AbstractDict`に存在します。

`[](## Promotion)
## 昇格

```@raw html
<!--
Promotion refers to converting values of mixed types to a single common type. Although it is not
strictly necessary, it is generally implied that the common type to which the values are converted
can faithfully represent all of the original values. In this sense, the term "promotion" is appropriate
since the values are converted to a "greater" type -- i.e. one which can represent all of the
input values in a single common type. It is important, however, not to confuse this with object-oriented
(structural) super-typing, or Julia's notion of abstract super-types: promotion has nothing to
do with the type hierarchy, and everything to do with converting between alternate representations.
For instance, although every [`Int32`](@ref) value can also be represented as a [`Float64`](@ref) value,
`Int32` is not a subtype of `Float64`.
-->
```

昇格は、様々な型の値を単一の共通の型に変換することを指します。
共通の型に変換した値は、元の値を全く忠実に表現していることが、必ずしも必要ありませんが、通常は想定されています。
この意味では、「昇格」という用語は適切で、値は「より大きな」型、
つまり、入力した値のすべてを表せる単一の共通の型に変換されます。
しかし重要な点ですが、昇格をオブジェクト指向の（構造的）スーパータイプやJuliaの抽象型のスーパータイプなどの概念と混同しないでください。
昇格は型の階層とは関連がなく、代替的な表現への変換のみに関連する概念です。
たとえば、すべての [`Int32`](@ref) の値は[`Float64`](@ref)の値として表現できますが、`Int32`は`Float64`のサブタイプではありません。


```@raw html
<!--
Promotion to a common "greater" type is performed in Julia by the `promote` function, which takes
any number of arguments, and returns a tuple of the same number of values, converted to a common
type, or throws an exception if promotion is not possible. The most common use case for promotion
is to convert numeric arguments to a common type:
-->
```

「より大きな」共通の型に昇格するには、Juliaでは`promote`関数を使います。
この関数は、任意個数の引数をとって、共通の型に変換された同じ個数の値のタプルを返し、また昇格が不可能な場合は例外を投げます。
昇格で一番よく使われるのは、引数の数値を共通の型に変換する場合です。


```jldoctest
julia> promote(1, 2.5)
(1.0, 2.5)

julia> promote(1, 2.5, 3)
(1.0, 2.5, 3.0)

julia> promote(2, 3//4)
(2//1, 3//4)

julia> promote(1, 2.5, 3, 3//4)
(1.0, 2.5, 3.0, 0.75)

julia> promote(1.5, im)
(1.5 + 0.0im, 0.0 + 1.0im)

julia> promote(1 + 2im, 3//4)
(1//1 + 2//1*im, 3//4 + 0//1*im)
```

```@raw html
<!--
Floating-point values are promoted to the largest of the floating-point argument types. Integer
values are promoted to the larger of either the native machine word size or the largest integer
argument type. Mixtures of integers and floating-point values are promoted to a floating-point
type big enough to hold all the values. Integers mixed with rationals are promoted to rationals.
Rationals mixed with floats are promoted to floats. Complex values mixed with real values are
promoted to the appropriate kind of complex value.
-->
```

浮動小数点数は、浮動小数点数の引数の型の中で最大のものに昇格します。
整数値は、ネイティブマシンのワードサイズと整数の引数の型の最大のものとのどちらか大きい方に昇格します。
整数と浮動小数点数の混合した場合は、すべての値を保持するのに十分な大きさの浮動小数点型に昇格します。
整数と有理数の混合した場合は有理数に昇格します。
有理数と浮動小数点数の混合した場合は、浮動小数点数に昇格します。
複素数と実数の混合した場合は、複素数の適切な型に昇格します。


```@raw html
<!--
That is really all there is to using promotions. The rest is just a matter of clever application,
the most typical "clever" application being the definition of catch-all methods for numeric operations
like the arithmetic operators `+`, `-`, `*` and `/`. Here are some of the catch-all method definitions
given in [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl):
-->
```

昇格の使い方はこれがすべてです。あとはどううまく使うかだけです。
最も「うまい」使い方は、 `+`、 `-`、 `*` 、`/`のような算術演算子に対する全捕捉メソッドの定義です。
ここで [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl)で定義された
全捕捉メソッドの一部を見てみましょう。


```julia
+(x::Number, y::Number) = +(promote(x,y)...)
-(x::Number, y::Number) = -(promote(x,y)...)
*(x::Number, y::Number) = *(promote(x,y)...)
/(x::Number, y::Number) = /(promote(x,y)...)
```

```@raw html
<!--
These method definitions say that in the absence of more specific rules for adding, subtracting,
multiplying and dividing pairs of numeric values, promote the values to a common type and then
try again. That's all there is to it: nowhere else does one ever need to worry about promotion
to a common numeric type for arithmetic operations -- it just happens automatically. There are
definitions of catch-all promotion methods for a number of other arithmetic and mathematical functions
in [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl), but beyond
that, there are hardly any calls to `promote` required in Julia Base. The most
common usages of `promote` occur in outer constructors methods, provided for convenience, to allow
constructor calls with mixed types to delegate to an inner type with fields promoted to an appropriate
common type. For example, recall that [`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl)
provides the following outer constructor method:
-->
```

これらのメソッド定義では、例えば加算、減算、乗算、除算に対してより特化した規則がない場合、値を共通の型に昇格してから演算を再試行します。
昇格を行うのはここだけです。他の場所では算術演算の共通の数値型への昇格を心配する必要はありません。
自動的に処理されます。
他にいくつもの算術関数や数学関数の全捕捉昇格メソッドの定義が[`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl)にありますが、
ここ以外で、JuliaのBaseライブラリの中で`promote`の呼び出しが必要となることはほとんどありません。
外部コンストラクターメソッドで一番よく見かける`promote`の使い方は、使い勝手がいいように、異なる型が混ざったコンストラクター呼び出しを可能にすることでしょう。
これば、引数を適切な共通型に昇格し、その型をフィールドに持つ内部型に、コンストラクター呼び出しを委譲して実現します。
たとえば [`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl)では、以下のような外部コンストラクタメソッドが利用可能なことを思い出してください。


```julia
Rational(n::Integer, d::Integer) = Rational(promote(n,d)...)
```

```@raw html
<!--
This allows calls like the following to work:
-->
```
これにより、次のような呼び出しが可能になります。


```jldoctest
julia> Rational(Int8(15),Int32(-5))
-3//1

julia> typeof(ans)
Rational{Int32}
```

```@raw html
<!--
For most user-defined types, it is better practice to require programmers to supply the expected
types to constructor functions explicitly, but sometimes, especially for numeric problems, it
can be convenient to do promotion automatically.
-->
```
ユーザー定義型は、プログラマーがコンストラクター関数に対して想定する型を明示的に指定するのが大抵の場合はよいと思いますが、
時には、特に数値問題の場合は、自動昇格を使うと、便利なものです。


`[](### Defining Promotion Rules)
### 昇格規則の定義

```@raw html
<!--
Although one could, in principle, define methods for the `promote` function directly, this would
require many redundant definitions for all possible permutations of argument types. Instead, the
behavior of `promote` is defined in terms of an auxiliary function called `promote_rule`, which
one can provide methods for. The `promote_rule` function takes a pair of type objects and returns
another type object, such that instances of the argument types will be promoted to the returned
type. Thus, by defining the rule:
-->
```
原理的には`promote`関数のメソッドを直接定義することができますが、とりうる引数の型の順列すべてに対する数多くの冗長な定義が必要になります。
その代わりに、`promote`関数の挙動の定義を、`promote_rule`という補助的な関数のメソッドを定義して行うことができます。
この`promote_rule`関数は、型オブジェクトの組を引数に取って、別の型オブジェクトを返しますが
これは引数のインスタンスの型が戻り値の型に昇格することを意味するので、これを使って規則を定義します。



```julia
promote_rule(::Type{Float64}, ::Type{Float32}) = Float64
```

```@raw html
<!--
one declares that when 64-bit and 32-bit floating-point values are promoted together, they should
be promoted to 64-bit floating-point. The promotion type does not need to be one of the argument
types, however; the following promotion rules both occur in Julia Base:
-->
```
64ビット浮動小数点数と32ビット浮動小数点数を一緒に昇格するときは、64ビット浮動小数点数に昇格する必要があります。
しかし昇格後の型は引数の型のどれかである必要はありません。
次の昇格規則は共にJuliaの標準ライブラリにあるものです。



```julia
promote_rule(::Type{UInt8}, ::Type{Int8}) = Int
promote_rule(::Type{BigInt}, ::Type{Int8}) = BigInt
```

```@raw html
<!--
In the latter case, the result type is [`BigInt`](@ref) since `BigInt` is the only type
large enough to hold integers for arbitrary-precision integer arithmetic. Also note that
one does not need to define both `promote_rule(::Type{A}, ::Type{B})` and
`promote_rule(::Type{B}, ::Type{A})` -- the symmetry is implied by the way `promote_rule`
is used in the promotion process.
-->
```

後者の場合、昇格後の型は[`BigInt`](@ref)になります。
というのも`BigInt`だけが、任意の精度の整数演算に対して整数を保持する大きさを持つの唯一の型だからです。
`promote_rule(::Type{A}, ::Type{B})`と `promote_rule(::Type{B}, ::Type{A})`の両方を定義する必要はない点に注意してください。`promote_rule`は、対称性を暗黙の前提として昇格の処理を行います。


```@raw html
<!--
The `promote_rule` function is used as a building block to define a second function called `promote_type`,
which, given any number of type objects, returns the common type to which those values, as arguments
to `promote` should be promoted. Thus, if one wants to know, in absence of actual values, what
type a collection of values of certain types would promote to, one can use `promote_type`:
-->
```
`promote_rule`関数を基にして、`promote_type`という第２の関数を定義します。
`promote_type`関数は、任意の数の型オブジェクトを引数にとり、これらの値の共通の型を返します。
この型が`promote`関数が引数を昇格後にとる型となります。
したがって、実際の値がなくても、`promote_type`を使えば、型の決まった値の集合が昇格するとどんな型になるかを調べることができます。



```jldoctest
julia> promote_type(Int8, Int64)
Int64
```

```@raw html
<!--
Internally, `promote_type` is used inside of `promote` to determine what type argument values
should be converted to for promotion. It can, however, be useful in its own right. The curious
reader can read the code in [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl),
which defines the complete promotion mechanism in about 35 lines.
-->
```

内部的には、`promote_type`は`promote`の内部で、引数値を昇格してどんな型に変換するかを決定するために使用されますが、`promote_type`単体でも有用なことがあります。
興味のある読者は約35行で完全な昇格の仕組みを定義するコードを [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl)で読むことができます。



`[](### Case Study: Rational Promotions)
### 事例研究: 有理数

```@raw html
<!--
Finally, we finish off our ongoing case study of Julia's rational number type, which makes relatively
sophisticated use of the promotion mechanism with the following promotion rules:
-->
```
とうとう、ここまで進めてきたJuliaの有理数型の事例研究が完成します。ここでは、以下の昇格規則による昇格メカニズムを比較的洗練された手法で利用しています。


```julia
promote_rule(::Type{Rational{T}}, ::Type{S}) where {T<:Integer,S<:Integer} = Rational{promote_type(T,S)}
promote_rule(::Type{Rational{T}}, ::Type{Rational{S}}) where {T<:Integer,S<:Integer} = Rational{promote_type(T,S)}
promote_rule(::Type{Rational{T}}, ::Type{S}) where {T<:Integer,S<:AbstractFloat} = promote_type(T,S)
```

```@raw html
<!--
The first rule says that promoting a rational number with any other integer type promotes to a
rational type whose numerator/denominator type is the result of promotion of its numerator/denominator
type with the other integer type. The second rule applies the same logic to two different types
of rational numbers, resulting in a rational of the promotion of their respective numerator/denominator
types. The third and final rule dictates that promoting a rational with a float results in the
same type as promoting the numerator/denominator type with the float.
-->
```
第1の規則は、有理数型と整数型を昇格すると、有理数型に昇格し、その分子/分母の型は元の有理数の分子/分母の型と整数型を昇格した型になることを示しています。
第2の規則は、2つの異なる有理数型を昇格すると、同様の論理を適用して、各有理数型の分子/分母の型を昇格した型を分子/分母の型とするような有理数型に昇格することを示しています。
最後の3つ目の規則は、有理数型と浮動小数点型を昇格すると、浮動小数点型に昇格し、その型は有理数型の分子/分母の型と浮動小数点数型を昇格した結果と同じ型になることを示しています。


```@raw html
<!--
This small handful of promotion rules, together with the type's constructors and the default
`convert` method for numbers, are sufficient to make rational numbers interoperate completely
naturally with all of Julia's other numeric types -- integers, floating-point numbers, and complex
numbers. By providing appropriate conversion methods and promotion rules in the same manner, any
user-defined numeric type can interoperate just as naturally with Julia's predefined numerics.
-->
```

この少数の昇格規則と、[前述の変換メソッド]だけで十分、有理数型をとても自然にJuliaの他の数値型、つまり 整数、浮動小数点数、複素数と一緒に使うことができます。
同様に、適切な変換メソッドと昇格規則を定義すれば、どんなユーザー定義の数値型でも自然に、Juliaで事前定義されている数値型と一緒に使うことができます。

