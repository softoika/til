RubyもしくはRailsに標準で用意されている関数を組み込みヘルパー、それに対して新しく作った関数をカスタムヘルパーと呼ぶ  
  
Pythonと違って関数やif、elseの行末に「:」はいらない。どの構文もendで終える  
メソッド名に記号を使うことができる 「?」など→Rubyにはbool値を返すメソッド名の末尾に?を付ける習慣がある isEmptyではくempty?    
文字列メソッドempty?は空文字でもtrueになるみたい。provideが実行されてなくてもtrue  
rubyではあらゆるものがオブジェクト。nilもオブジェクトなのでメソッドを持つ(nil.to_sで空文字列を返す)
  
Rails Console(Rails用のインタラクティブ環境)には"development"と"test"、"production"の3つの環境がある  
デフォルトではdevelopment環境が選択される  
これに関しては7章で詳しく説明される  
  
RubyのAPIリファレンスとしては[Rubyリファレンスマニュアル (通称: るりま))](https://docs.ruby-lang.org/ja/)や[るりまサーチ]は(https://docs.ruby-lang.org/ja/search/)が便利  
  
モジュールはメソッドをまとめておくもの```application_helper.rb```のように  
```rb
module ApplicationHelper
end
```
のようにして囲む。  
include文を使用するとモジュールを取り込むことができる(ミックスイン[mixed in])  
Rubyでは明示的にincludeしないと使用できないが、Railsでは自動的にヘルパーモジュールをインクルードしているので  
includeを書かなくてもいい  
  
### 演算子  
Pythonと同じく「**」でべき乗を表す```3**2```は3の二乗で9  
リストを初期化する際には```a = ["a", "b", "c"]```でもできるが、  
```a = %w[a b c]```のように文字列配列を定義するのに便利な書き方もある。  
  
### 文字列  
文字列の結合は「+」  
Rubyには文字列内で式展開ができる  
```rb
first_name = "Michael"
"#{first_name} Hart" # -> "Michael Hart"
```
シングルクオートによる文字列は式展開をしない  
文字列メソッドにはempty?以外にnil?やinclude?がある。include?は文字列を引数に取る(contain?ではない)  
文字列メソッドsplitの初期引数は' '
  
  
### 標準出力
rubyにおける標準出力はputsを使う
```rb
puts "hoge" # -> nil
```
putsはnil(他の言語でいうnull)を返す  
  
rubyにもprint関数はあるが、putsが行末に改行が入るのに対し、printは末尾に改行が入らない

### 制御構文  
条件分岐はif、else、そしてelsif  
else ifでもelifでもない
論理演算子には&&,||,!のほかに(Pythonのように)and, or, notが使える
  
rubyには「後続if」という特殊な書き方がある  
```rb
if !x.empty?
  puts "x is not empty"
end
```
という書き方を  
```rb
puts "x is not empty" if !x.empty?
```
と書き換えることができる  
　　
オブジェクト + 論理演算子という書き方は論理値を必ず返すので、二重否定をすれば(```!!obj```と書く)必ずそのオブジェクトの論理値を返す。
  
### 関数  
Pythonと同じく引数に初期値の設定ができる  
関数の呼び出し時には「(」「)」を省略できる  
  
Rubyの関数には暗黙の戻り値がある。  
暗黙の戻り値は最後に評価された式の値が返される  
```rb
def string_message(str='')
  if str.empty?
    "It's an empty string!"
  else
    "The string is nonempty."
  end
end
```
この場合returnが書かれていないにも関わらず最後に評価された"It's an empty string!"か"The string is nonempty."のどちらかが返される。  
  
上の関数は次のような書き方もできる
```rb
def string_message(str='')
  return "It's an empty string!" if str.empty?
  return "The string is nonempty."
end
```
最後のreturnは省略できる。  
  
### 配列  
添字はPythonと同じように-1で末尾を指定できる。例：```a[-1]```
配列メソッドでも特定の要素を参照できる。```a.first```は```a[0]```に等しい。```a.last```は```a[-1]```に等しい。```a.second```もある。  
  
配列の長さはlengthメソッドで取得できる  
また、sort!メソッドで配列をソートできる。Rubyでは末尾に!を付けたメソッドを破壊的メソッドと呼び、  
sort!のように配列の内容を変更するメソッドが破壊的メソッドと定義される。  
pushメソッドで引数に指定した要素を配列の末尾に加えることができる  
また、「<<」演算子でもpushメソッドと同じ.例:```a << 6```は```a.push 6```と同じ。```a << 6 << 9```のような使い方もできる。
  
範囲(range)と呼ばれる書き方もある。```0..9```と書いて0から9を表す。  
配列に変換するときはto_aメソッドを使う.例:```(0..9).to_a```。カッコは省略できない。```('a'..'e').to_a```のように連続する文字にも対応している。  
範囲は添字として使うことができる。例：```a[0..2]```でa[0],a[1],[2]の範囲の配列を取得できる。  
  
  
### ハッシュとシンボル  
ハッシュ=辞書=連想配列  
ハッシュの定義の仕方その１  
```rb
user = {} # 空のハッシュ
user["first_name"] = "Michael"
user["last_name"] = "Hartl"
```
  
ハッシュの定義の仕方その２（ハッシュロケット```=>```を使った定義)
```rb
user = {"first_name" => "Michael", "last_name" => "Hartl"}
```
  
シンボル...クォートで囲む代わりにコロンが前に置かれる書き方```"name"```は```:name```とも書ける  
```:foo-bar```や```:2foo```のような書き方はできない(基本的に変数の命名規則と同じ)  
ハッシュではシンボルをキーで使うことが一般的  
```rb
h1 = { :name => "Michael Hartl", :email => "michael@example.com" }
```
  
また、```hoge:```のようにコロンを後ろに置くと次のような書き方でハッシュを書くことができる  
```rb
h1 = {name: "Micael Hartl", email: "micahel@example.com"}
```
  
ハッシュはネストする形でも定義することができる  
```rb
params = {}
params[:user] = { name: "Michael Hartl", email: "mhartl@example.com" }
```
  
配列や範囲オブジェクトと同様にハッシュもeachメソッドを持つ  
```rb
flash = { success: "It worked!", danger: "It failed." }
flash.each do |key, value|
  puts "Key #{key.inspect} has value #{value.inspect}"
end
```
  
inspectメソッド...配列などのオブジェクトに対しRailsコンソールで返すような形式で出力する。　　
すなわち、オブジェクトを人が読める形にした文字列を返すもの。
```rb
puts (1..5).to_a.inspect # 出力結果->[1,2,3,4,5]
puts "hoge".inspect      # 出力結果->"hoge"
puts nil.inspect         #        ->"nil"
```
  
inspectメソッドを使う代わりにp関数を使うことでも同様の出力が可能  
  
ハッシュのmergeメソッドでキーが重複する場合、ブロックでの指定がなければ引数の値で更新される  
```rb
{ "a" => 100, "b" => 200 }.merge({ "b" => 300 }) # {"a" => 100, "b" => 300}
```
  
関数呼び出しにおいて、最後の引数がハッシュの場合には波括弧を省略できる  
```rb
# 最後の引数がハッシュの場合、波かっこは省略可能。
stylesheet_link_tag 'application', { media: 'all',
                                     'data-turbolinks-track': 'reload' }
stylesheet_link_tag 'application', media: 'all',
                                   'data-turbolinks-track': 'reload'
```
  
  
### ブロック  
Rubyにはブロックという範囲や配列に関する機能がある(Pythonでいうリストの内包表記みたいなもの？)  
```rb
(1..5).each{ |i| puts i * 2 }
```
のような使い方をする。  
doやendを使った書き方もできる
```rb
(1..5).each do |i|
  puts i * 2
end
```
Rubyの慣習としては短い１行を書く場合は波括弧を使い、長い１行や複数行書く場合にはdo endを使う。
ブロックのそれぞれの要素に対する演算をそれぞれ配列に格納して返すにはmapを使う  
```rb
a = %w[a b c].map{ |c| c.upcase }
```
また上記の書き方は省略記法を使ってこう書くことができる(symbol-to-proc)  
```rb
a = %w[a b c].map(&:upcase)
```
波括弧でなく丸括弧なことに注意
  
testの書き方も実はブロック。テスト名がtestの引数でdo-endでブロックを囲っている  
```rb
test "test hogehoge" do
  #テスト内容
end
```
  
  
### Rubyにおけるクラス  
文字列を使うと文字列のインスタンスが自動的に作成される(リテラルコンストラクタ)  
オブジェクトからclassメソッドを呼び出すと元のクラスを確認できる  
```rb
s = "foobar"
s.class      # => String
```
newメソッドを使うことで明示的なインスタンス生成ができる  
```rb
s = String.new("foobar") # 文字列の名前付きコンストラクタ
```
配列の場合  
```rb
a = Array.new([1,2,3])
```
ハッシュの場合  
```rb
h = Hash.new #配列のように引数を取ることはできない
```
  
クラスに対し、superclassメソッドを使うことで親クラスを調べることができる
```rb
"".class.superclass # =>Object
```
  
**継承の書き方**  
自身の文字列の回文を調べるクラスの例  
ちなみにself.reverseのような例ではselfを省略できる
```rb
class Word < String
  def palindrome?
    self == self.reverse
  end 
end
```
Stringを継承しているので引数に文字列を指定してインスタンスを生成できる  
```rb
w = Word.new("hoge")
w.palindrome?
```
  
組み込みクラスにメソッドを拡張することもできる  
```rb
class String
  def palindrome?
    self == reverse
  end
end
```
  
**コントローラクラス**  
Railsのコントローラ(クラス)のアクション(メソッド)には戻り値が存在しない  
アクションはWebページを表示するものであるため。  
  
**アクセサー**
属性(attribute)に対してgetterとsetterを定義   
```rb
class User
  attr_accessor :name, :email
end
```
  
**インスタンス変数のinitializeメソッド**
initializeメソッドはnewメソッドでインスタンスが生成されるときに呼び出される  
```rb
class User
  attr_accessor :name, :email
  
  def initialize(attributes = {})
    @name = attributes[:name]
    @email = attributes[:email]
  end
end
```
このinitializeメソッドは空のハッシュを１つ引数に取っている  
引数のハッシュに何も入っていなければ@nameにも@emailにもnilが入ることになる  
このようにアトリビュートアクセサを設定することで```インスタンス名.name```のように  
外部からインスタンス変数の参照や変更ができるようになる
  
クラスをコンソールなどで読み込みたい場合はrequireを使う  
カレントディレクトリはアプリケーションのルートディレクトリとなる  
```rb
require './example_user' # example_user.rbの中のクラスを読み込みたい場合
```
  
実際にインスタンス生成時にinitializeの引数にハッシュを与えてみる  
```rb
user = User.new(name:"hoge", email:"hoge@example.com")
```
最後の引数がハッシュなら波括弧を省略できる  
このようにハッシュ引数を使ってオブジェクトを初期化する方法をマスアサイメント(mass assignment)と呼ぶ。 
