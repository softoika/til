## 6章ユーザーのデータモデルの作成  
ユーザ情報を保存する場所、保存するためのデータ構造をまずは作成する必要がある  
MVCのM(モデル)にあたる  
モデルのデータを長期保存する手段としてデータベースが用いられる  
RailsライブラリにあるActive Recordを用いることでSQLを直接操作することなしにデータベースを操作できる  
また、マイグレーション(Migration)の機能によりデータの定義をRuby上で行うことができる  
  
### データモデルの生成
モデルの作成にあたり、まずはデータモデルの定義を設計しておくとよい。  
次に```rails generate```コマンドでモデルを生成する  
```
$ rails generate model User name:string email:string
```
```rails generate```の最初の引数は```model```であり、これはモデルを生成することを示している  
コントローラを生成したい場合には```model```の代わりに```controller```が入る  
その次の引数```User```はモデル名を表す  
それ以降の引数はモデルの持つ各属性を表し、```【属性名】:【データ型】```という書式を取る  
主キーとなるIDは省略できる(書かなくても生成される)    
モデル名は単数形でコントローラ名は複数形、  データベースのテーブル名も複数形    
    
### マイグレーション  
```rails genarate```により、マイグレーションと呼ばれるファイルも生成される(```db/migrate/[timestamp]_create_users.rb```)  
マイグレーションによりデータ構造の変更が容易になっている  
マイグレーション自体はデータベースへ与える変更を定義した```change```メソッドの集まり  
```change```メソッドでは```create_table```というRailsのメソッドを用いてデータベースのテーブルを作成している  
マイグレーションの実行（マイグレーションの適用という)は次のコマンドで行う  
```
$ rails db:migrate
```
これでデータベースが生成される  
マイグレーションはロールバック(rollback)により元に戻すことが可能である  
```
$ rails db:rollback
```
  
### ユーザオブジェクトの生成
モデルからオブジェクトを生成するにはモデルの```new```メソッドを使う  
```
>> user = User.new
```
オブジェクトとしての有効性は```valid?```メソッドで確認できる  
生成されたオブジェクトはデータベースにまだ格納されていないので```save```メソッドで登録する  
```new```と```save```を使う以外にも```create```メソッドを使って、モデルの生成と保存をまとめて行うこともできる  
使い方は```new```メソッドと同じ  
そして```destroy```メソッドは```create```メソッドの逆の処理を施す  
```
>> user = User.create(name: "hoge", email: "fuga@piyo.com") # データの生成
>> user.destroy                                             # データの削除
```
ただし、```destroy```を実行してもuserのオブジェクトが削除されるわけではない  
  
### ユーザオブジェクトの検索  
生成されたオブジェクトの検索には```find```メソッドを用いる  
```
>> User.find(1)
```
この場合,引数にはIDが入る
```new```メソッドでデータを生成していても```save```していなければ```find```で見つけることはできないし、  
```destroy```していても```find```で見つけることはできない  
また、```find_by```メソッドにより、ID以外のカラムでの検索もできる  
```
>> User.find_by(email: "fuga@piyo.com")
```
他に```User.first```や```User.all```によるデータの取得もできる  
  
### ユーザオブジェクトの更新  
1つは属性を個別に代入するという方法  
```
>> user.email = "bbb@ccc.com"
>> user.save
```
```save```を呼び出す前に```user.reload```を呼び出すと変更を取り消すことができる  
もう一つの方法は```update_attributes```を使う場合  
```
>> user.update_attributes(name:"aaa", email:"bbb@ccc.com")
```
このメソッドもcreateと同様に保存を一括して行う  
  
　  
### ユーザーの検証  
ユーザ名は空であってはいけない、メールは重複してはいけないのようにそれぞれの属性が条件に合致しているか確認する必要がある  
Active Recordでは検証(Validation)という機能により制約を課すことができる  
検証においてよく使われるケース→存在性の検証、長さの検証、フォーマットの検証、一意性の検証。
  
**有効性の検証**  
具体的なテスト方法としては、まず有効なモデルのオブジェクトを作成し、その属性のうち１つを有効でない属性に変更する  
そしてバリデーションで失敗するか確かめる  
念のため最初に作成時の状態でもテストして有効かどうか確認する。こうするとバリデーションのテストが失敗したとき、バリデーションの実装に問題があったかオブジェクトに問題が合ったか確認できる  
　　
モデルのテストなので```rails generate```で作成した```test/models/user_test.rb```に書く  
```rb
require 'test_helper'

class UserTest < ActiveSupport::TestCase

  def setup
    @user = User.new(name: "Example User", email: "user@example.com")
  end

  test "should be valid" do
    assert @user.valid?
  end
end
```
```setup```という特殊なメソッドを使ってUserオブジェクトを作成する。```setup```はテストの実行前に実行される  
インスタンス変数```@user```を使うことでUserオブジェクトに対しテストすることができる  
assertメソッドは返ってくる値がtrueであれば成功し、falseだと失敗する  
  
モデルのみのテストは  
```
$ rails test:models
```
で実行できる  
```"should be valid"```のテストはUserモデルにバリデーションを定義していないのでテストは通る  
  
  
### 存在性の検証 
渡された属性が存在するかを検証する  
例えばユーザにnameとemailの両方が存在するか  
  
まず、name属性のバリデーションに対するテストを書く  
例えば```user_test.rb```に次のようなテストを追加する  
```rb
test "name should be present" do
  @user.name = "   "
  assert_not @user.valid?
end
```
```assert_not```は```assert```と逆で、```false```を返すときに成功する  
この場合、```"   "```の名前が有効でない(```false```)と返ってきたら成功になる  
次に```user.rb```のモデルにバリデーションを記述する  
```rb
class User < ApplicationRecord
  validates :name, presence: true
end
```
モデルクラスに```validates```メソッドで属性のルールを決定し、そのルールに適合しているかどうかを```valid?```メソッドが返す  
データの有効性を検証するにはvalidatesでルールを定義し、テストクラスでルールに反するデータを作成し、valid?に対してassertでテストすればよい    
    
ユーザオブジェクトを生成すると、ユーザオブジェクトから```valid?```メソッドを呼び出して有効かどうか調べることができる  
また```errors```オブジェクトでエラーメッセージを確認することもできる  
```
>> user.errors.full_messages
=> ["Name cant be blank"]
```
有効でない状態でデータベースに保存しようとしても失敗する→```save```メソッド  
  
emailに関する存在性の検証も同様に実装する  
  
　  
### 長さの検証  
nameとemailに関して最大文字数のテストをする(```user_test.rb```)  
```rb
  test "name should not be too long" do
    @user.name = "a" * 51
    assert_not @user.valid?
  end

  test "email should not be too long" do
    @user.email = "a" * 244 + "@example.com" # 256文字
    assert_not @user.valid?
  end
```
文字数を制限する処理を書いていないので```@user.valid?```はtrueを返す(テスト失敗)  
次のように```user.rb```の```validates```メソッドを変更する  
```rb
  validates :name,  presence: true, length: { maximum: 50 }
  validates :email, presence: true, length: { maximum: 255 }
```
```presence:```以降がオプションハッシュになっていて、```length:```の値もハッシュとなっている。
  
　  
### フォーマットの検証  
emailのフォーマットがuser@example.comの形式を取っているか検証する  
メールアドレスのバリデーションは扱いが難しく、エラーが発生しやすい  
よって有効なメールアドレスと無効なメールアドレスをいくつか用意して、バリデーション内のエラーを検知していく  
まずは有効なメールアドレスのテストとして、```user_test.rb```に次のようなテストを追加する  
```rb
  test "email validation should accept valid addresses" do

  valid_addresses = %w[user@example.com USER@foo.COM A_US-ER@foo.bar.org
                         first.last@foo.jp alice+bob@baz.cn]

  valid_addresses.each do |valid_address|
    @user.email = valid_address
    assert @user.valid?, "#{valid_address.inspect} should be valid"
  end

end
```
```valid_addresses```にメールアドレスの文字列リストを作成し、それぞれのメールアドレスに対して検証をする  
assertメソッドの第二引数は検証に失敗したときのメッセージを文字列として渡している  
文字列に埋め込まれている```valid_address.inspect```は文字列オブジェクトとしてのvalid_addressの文字列表現が入る  
  
次に無効なメールアドレスに関してもテストを作成する  
```rb
  test "email validation should reject invalid addresses" do
    invalid_addresses = %w[user@example,com user_at_foo.org user.name@example.
                           foo@bar_baz.com foo@bar+baz.com]
    invalid_addresses.each do |invalid_address|
      @user.email = invalid_address
      assert_not @user.valid?, "#{invalid_address.inspect} should be invalid"
    end
  end
```
この時点でテストをして、失敗することを確認する。1つ目のvalid_addressesに関するテストは成功するがinvalid_addressesの方は失敗する  
  
メールアドレスのフォーマットを検証するには  
```rb
validates :email, format: { with: /<regular expression>/ }
```
のように正規表現を含んだ```format:```を引数に与える必要がある  
正規表現の動作を確認するには[Rebular](http://www.rubular.com/)というサイトがおすすめ。  
正規表現のルールに対しどんな文字列がマッチするか対話的に確認することができる  
   
実際のメールアドレスの正規表現は実戦で堅牢性が保証されているものを流用する  
```user.rb```にフォーマットのバリデーションを追加する  
```rb
VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX }
```
  
　  
### 一意性の検証  
validatesメソッドの:uniqueオプションを使うことで可能。  
ここではメールアドレスをユーザ名として使うために他のユーザと被らないよう一意性を検証する  
  
一意性を検証するためには、データのオブジェクトをnewメソッドで生成してテストする方法では検証できない。  
そのためレコードをデータベースに登録する必要がある  
```user_test.rb```に次のテストメソッドを追加する  
```rb
  test "email addresses should be unique" do
    duplicate_user = @user.dup
    @user.save
    assert_not duplicate_user.valid?
  end
```
```@user.dup```というメソッドを使っているが、これは同じ属性を持つデータを複製するためのメソッド。  
@userが生成され、```save```メソッドで保存された後では、複製されたユーザが既存の(データベースに保存された)ユーザと同じメールアドレスを持つため、複製された```duplicate_user```の作成は無効になるはず(```duplicate_user.valid?```)
  
このテストをパスするために、emailのバリデーションに```uniqueness: true```のオプションを追加する  
```rb
validates :email, presence: true, length: {maximum: 255},
    format: { with: VALID_EMAIL_REGEX },
    uniqueness: true
```
これでテストはパスするが、メールアドレスには大文字小文字が区別されないという特徴がある。  
そのため、foo@bar.comとFoo@BAr.comのような場合にも重複していると判断しなければならない。  
先程の```test "email addresses should be unique"```のテストメソッドを次のように変更する  
```rb
  test "email addresses should be unique" do
    duplicate_user = @user.dup
    duplicate_user.email = @user.email.upcase
    @user.save
    assert_not duplicate_user.valid?
  end
```
upcaseで@userの"user@example.com"は"USER@EXAMPLE.COM"となり(duplicate_userのメールアドレスは小文字のまま)、これが無効と判断されればテストが通る  
:uniquenessでは:case_sensitiveのオプションをfalseにすることで大文字小文字を区別しないようにできる  
```rb
validates :email, presence: true, length: {maximum: 255},
    format: { with: VALID_EMAIL_REGEX },
    uniqueness: {case_sensitive: false}
```

**Webサイトのトラフィック上で起こるデータの重複**  
トラフィックが多いとき送信ボタンを2回連続で押すなどした場合に、最初のリスエストの保存が終わる前に次のリクエストの同じメールアドレスによるユーザの作成ができてしまい、重複したユーザーレコードの生成が可能になってしまうことがある。  
これを解決するにはemailのカラムにインデックスを追加し、そのインデックスが一意であるようにすればいい  
なぜなら、重複しているか調べるためには１つ１つのレコードのemail属性を探索する必要があり非効率（全表スキャン(Full-table Scan))  
emailカラムにインデックスを追加することで索引から探すことができるようになり、テーブルの全てのレコードを探す必要がなくなる  
今回は既に存在するモデルに構造を追加するのでmigrationジェネレーターを使用して直接マイグレーションを作成する必要がある    
```
$ rails g migration add_index_to_users_email
```
次に生成されたマイグレーションファイルのchangeメソッドにインデックス作成の定義を追加する(```db/migrate/[timestamp]_add_index_to_users_email.rb```)  
```rb
  def change
    add_index :users, :email, unique: true
  end
```
このメソッド内ではusersテーブルのemailカラムにインデックスを追加するためのadd_indexというメソッドを使っている  
オプションで```unique: true```とすることでインデックスの一意性を強制できる  
最後にデータベースをマイグレートする  
```
$ rails db:migrate
```
この時点では、テスト用のDBにサンプルデータが含まれ、サンプルデータに一意性が保たれていないため、テストは失敗する(サンプルデータは```test/fixtures/users.yml```に定義されている)  
よってusers.ymlの中のサンプルデータの記述をすべて削除する必要がある(削除するとテストは通る)  
  
**データベースのアダプタが大文字小文字区別するインデックスを使っているとは限らない問題**  
例えばFoo@ExAMPle.Comとfoo@example.comが別々の文字と解釈するデータベースがある  
これを避けるために、データベースに保存する前にすべて小文字に変換しておくという方法をとる。  
実装にはActiveRecordのコールバックメソッドを使う。  
オブジェクトがsaveされる前に処理を追加したいので```before_save```というコールバックを使う  
```user.rb```の次の一行を追加する  
```rb
class User < ApplicationRecord
  before_save {self.email = email.downcase}
  .
  .
  .
end
```
ここではbefore_saveコールバックにブロックを渡してメールアドレスの設定を行っている。
ちなみに```self.email.downcase```のselfは省略できるが```self.email```は省略できない(後ほど詳しく解説される)  
  
### ハッシュ化されたパスワード  
```
has_secure_password
```
このメソッド呼び出しをUsersモデルのクラスに書くだけでハッシュにより暗号化されたパスワードを使うことができる  
さらにパスワード登録時の２回書く確認や存在性と２回の値が一致するかのバリデーションもできるようになる  
authenticateという引数の文字がパスワードと一致するとUsersオブジェクトを間違っていると判定するメソッドも使えるようになる  
  
ただし、```has_secure_password```を使うにはモデルに```password_digest```という属性を持っている必要がある  
すなわち、Usersを次のようなデータモデルにする必要がある  
  
| 属性                 | 型         |  
|:--------------------|:-----------|  
| id                  | integer    |  
| name                | string     |  
| email               | string     |  
| created_at          | datetime   |  
| updated_at          | datetime   |  
| **password_digest** | **string** |  
  
  
この属性を追加するために以下のコマンドを実行する  
```
$ rails generate migration add_password_digest_to_users password_digest:string
```
add_password_digest_to_usersはマイグレーション名  
to_usersを追加することによってusersテーブルにカラムを追加するマイグレーションが自動生成される  
```
$ rails db:migrate
```
これでマイグレーションが反映される  
　　  
また、パスワードをハッシュ化するために最先端のハッシュ関数である```bcrypt```が必要  
次のようにGemfileに追記する  
```rb
gem 'bcrypt',         '3.1.11'
```
```
$ bundle install
```
has_secure_passwordは使えるようになったがこの状態のままではテストに失敗する  
テストで使用しているUsersオブジェクトにpasswordとpassword_confirmationが追加されていなく、バリデーションが効いているからだ  
なのでUsersTest.rbのsetupメソッドを次のように変更する  
```rb
  def setup
    @user = User.new(name: "Example User", email: "user@example.com",
                     password: "foobar", password_confirmation: "foobar")
  end
```  
  
### パスワードの最小文字数
まずパスワードの最小文字数が６文字以上でパスするテストを書く  
```rb   
test "password should be present (nonblank)" do
  @user.password = @user.password_confirmation = " " * 6
  assert_not @user.valid?
end

test "password should have a minimum length" do
  @user.password = @user.password_confirmation = "a" * 5
  assert_not @user.valid?
end
```
ここでは多重代入という書き方を使っている  
```rb
@user.password_confirmation = "a" * 5
@user.password = @user_password_confirmation
```
という書き方が  
```rb
@user.password = @user.password_confirmation = "a" * 5
```
と書ける  
最小文字数のバリデーションは次のように追加する  
```rb
validates :password, length: { minimum: 6 }
```
他の属性と同様に```presence: true```を渡すことで存在性の検証も通る  
```rb
validates :password, presence: true, length: { minimum: 6 }
```
  
### ユーザーの作成と認証  
rails consoleでユーザーの作成および認証を試してみる  
```
$ rails console 
```
```rb
>> User.create(name: "Michael Hartl", email: "mhartl@example.com",
?>             password: "foobar", password_confirmation: "foobar")
```
```rb
>> user = User.find_by(email: "mhartl@example.com")
>> user.password_digest
```
password_digestはハッシュの文字列で表現していて、bcryptで暗号化されているため、元のパスワードを算出することは困難  
そこでauthenticateメソッドを用いることでパスワードの認証をすることができる  
```rb
>> user.authenticate("not_the_right_password")
false
>> user.authenticate("foobar")
【userオブジェクトが返ってくる】
>> !!user.authenticate("foobar")
true
```
