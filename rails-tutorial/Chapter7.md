## 7.1 ユーザを表示する  
  
### 7.1.1 デバッグとRails環境  
動的なユーザページを作るということでデバッグ表示を出せるようにする  
application.html.erb
```erb
<!DOCTYPE html>
<html>
  .
  .
  .
  <body>
    <%= render 'layouts/header' %>
    <div class="container">
      <%= yield %>
      <%= render 'layouts/footer' %>
      <%= debug(params) if Rails.env.development? %>
    </div>
  </body>
</html>
```
```erb
<%= debug(params) if Rails.env.development? %>
```
これを追記する  
本番環境やテスト環境ではデバッグ表示したくないので```if Rails.env.development?```の記述をしている(開発環境のみ)  
ビルトインのdebugメソッドとparams変数を用いている  

デバッグ表示の整形のためにcssを設定する  
```scss
@import "bootstrap-sprockets";
@import "bootstrap";

/* mixins, variables, etc. */

$gray-medium-light: #eaeaea;

@mixin box_sizing {
  -moz-box-sizing:    border-box;
  -webkit-box-sizing: border-box;
  box-sizing:         border-box;
}
.
.
.
/* miscellaneous */

.debug_dump {
  clear: both;
  float: left;
  width: 100%;
  margin-top: 45px;
  @include box_sizing;
}
```
Sassのミックスイン機能を用いている  
これによりCSSのグループをパッケージ化して複数の要素に適用することができる  
@mixinで定義したものが@includeしたところで適用されるので関数みたいなもの?　　
　　
### 7.1.2Usersリソース  
Userモデルの中のデータ表示を行う
RESTアーキテクチャの習慣に従う リレーショナルデータベース操作のPOST/GET/UPDATE/DELETEと
HTTPリクエストのPOST/GET/PATCH/DELETE両方に対応している  
REST原則に従う場合、リソース名とユニークIDを用いる。例えば/users/1のように  
RailsのREST機能が有効になっているとGETリクエストはshowアクションとして扱われる  
/users/1へのURLを有効するためにconfig/routes.rbに以下の１行を追加する  
```rb
resources: users
```
/users/1が有効になるだけでなく、RESTfulなUsersリソースで必要となるすべてのアクションが利用できるようになる
名前付きルートも使える  
[resources:usersで使えるURLとそれに対応するアクション、名前付きルート](https://railstutorial.jp/chapters/sign_up?version=5.0#table-RESTful_users)
　　
現状ではルーティングが有効になっているがまだページは表示されない  
表示のためにビューが必要なのでユーザ情報を表示するための仮のビューを作成する  
app/views/users/show.html.erb  
```rb
<%= @user.name %>, <%= @user.email %>
```
この記述は@userが定義されていることが前提であるため、showアクションで@userの定義を行う必要がある 
app/controllers/users_controller.rb  
```rb
class UsersController < ApplicationController

  def show
    @user = User.find(params[:id])
  end

  def new
  end
end
```
idの指定にparamsを使用した。Usersコントローラにリクエストが正常に送信されると、params[:id]の部分はユーザーidの1に置き換わる  
param[:id]は文字列のidを返すが、findメソッドが自動的に数値として解釈する  
  
### 7.1.3 debuggerメソッド
byebug gemによるdebuggerメソッドを用いると簡単にデバッグプリントを挟むことができる  
app/controllers/user_controller.rb
```rb
class UsersController < ApplicationController
def show
    @user = User.find(params[:id])
    debugger
  end
def new
  end
end
```
/users/1にアクセスしてターミナルを見るとdebuggerが呼び出される瞬間の状態を確認できる  
確認し終わったらdebuggerの行を削除する  
  
### Gravatar画像とサイドバー
Gravatarをプロフィールに導入する  
>Gravatarは無料のサービスで、プロフィール写真をアップロードして、指定したメールアドレスと関連付けることができます。その結果、 Gravatarはプロフィール写真をアップロードするときの面倒な作業や写真が欠けるトラブル、また、画像の置き場所の悩みを解決します。というのも、ユーザーのメールアドレスを組み込んだGravatar専用の画像パスを構成するだけで、対応するGravatarの画像が自動的に表示されるからです   
  
gravatar_forヘルパーメソッドを使用して表示を行う  
app/viewers/users/show.html.erb
```rb
<% provide(:title, @user.name) %>
<h1>
  <%= gravatar_for @user %>
  <%= @user.name %>
</h1>
```
デフォルトではヘルパーファイルで定義されているメソッドは自動的にすべてのビューで使用できる  
[Gravatarのホームページ](http://ja.gravatar.com/site/implement/hash/)にも書かれているようにGravatarのURLはユーザーのMD5という仕組みでハッシュ化されている。  
DigestライブラリのhexdigestメソッドでMD5のハッシュ化ができる  
```rb
>> email = "MHARTL@example.COM"
>> Digest::MD5::hexdigest(email.downcase)
=> "1fda4469bcbec3badf5418269ffc5968"
```
MD5ではメールアドレスの小文字大文字は区別されるのでdowncaseメソッドで小文字に統一する必要がある  
gravatar_forヘルパーメソッドを定義する  
app/helpers/users_helper.rb
```rb
module UsersHelper
# 引数で与えられたユーザーのGravatar画像を返す
  def gravatar_for(user)
    gravatar_id = Digest::MD5::hexdigest(user.email.downcase)
    gravatar_url = "https://secure.gravatar.com/avatar/#{gravatar_id}"
    image_tag(gravatar_url, alt: user.name, class: "gravatar")
  end
end
```
```rb
image_tag(gravatar_url, alt: user.name, class: "gravatar")
```
はgravatarクラスとユーザー名のaltテキストを追加したもの。  
この状態でusers/1にアクセスするとGravatarのデフォルト画像が表示される(user@example.comが本当のメールアドレスではないため)
アプリケーションでGravatarを利用できるようにするため、updata_attributesメソッドを使ってユーザ情報(メールアドレスを更新してみる)  
```rb
$ rails console
>> user = User.first
>> user.update_attributes(name: "Example User",
?>                        email: "example@railstutorial.org",
?>                        password: "foobar",
?>                        password_confirmation: "foobar")
=> true
```
  
ユーザーのサイドバーの表示を整える  
asideタグを使用し、rowクラスとcol-md-4クラスも追加する  
app/views/users/show.html.erb
```rb
<% provide(:title, @user.name) %>
<div class="row">
  <aside class="col-md-4">
    <section class="user_info">
      <h1>
        <%= gravatar_for @user %>
        <%= @user.name %>
      </h1>
    </section>
  </aside>
</div>
```
HTML要素とCSSクラスを配置したことによりSCSSで整えることができるようになった  
app/assets/stylesheets/custom.scss
```scss
.
.
.
/* sidebar */
aside {
  section.user_info {
    margin-top: 20px;
  }
  section {
    padding: 10px 0;
    margin-top: 20px;
    &:first-child {
      border: 0;
      padding-top: 0;
    }
    span {
      display: block;
      margin-bottom: 3px;
      line-height: 1;
    }
    h1 {
      font-size: 1.4em;
      text-align: left;
      letter-spacing: -1px;
      margin-bottom: 3px;
      margin-top: 0px;
    }
  }
}
.gravatar {
  float: left;
  margin-right: 10px;
}
.gravatar_edit {
  margin-top: 15px;
}
```
### 7.2 ユーザ登録フォーム  
ユーザ登録用のページを作る  
メールアドレスとユーザ名とパスワード、パスワードの確認欄を設ける  
  
#### 7.2.1 form_for
入力欄として用いるフォーム用にRailsのform_forヘルパーメソッドを用いる  
form_forではActive Recordのオブジェクトを取り込み、そのオブジェクトの属性をつかってフォームを構築するようにする。  
  
ところでユーザ登録のルーティングはnewアクションに紐付けられていた  
app/config/routes.rb  
```rb
  get '/signup' to: 'users#new'
```
form_forで必要となるuserオブジェクトの作成をnewアクションでする  
app/controllers/user_controller.rb  
```rb
  def new
    @user = User.new
  end
```
  
formそのものはviewに書く  
app/views/users/new.html.erb  
```erb
<% provide(:title, 'Sign up') %>
<h1>Sign up</h1>
<div class="row">
  <div class="col-md-6 col-md-offset-3">
    <%= form_for(@user) do |f| %>
    <%= f.label :name %>
    <%= f.text_field :name %>
    <%= f.label :email %>
    <%= f.email_field :email %>
    <%= f.label :password %>
    <%= f.password_field :password %>
    <%= f.label :password_confirmation, "Confirmation" %>
    <%= f.password_field :password_confirmation %>
    <%= f.submit "Create my account", class: "btn btn-primary" %>
    <% end %>
  </div>
</div>
```
```form_for(@user) do |f|```のfオブジェクトはHTMLフォーム要素 (テキストフィールド、ラジオボタン、パスワードフィールドなど) に対応するメソッドが呼び出されると、@userの属性を設定するために特別に設計されたHTMLを返す。  
つまり、  
```erb
<%= f.label :email %>
<%= f.email_field :email %>
```
この記述はUserモデルのname属性を設定する、ラベル付きテキストフィールド要素を作成するのに必要なHTMLを作成する(生成されたページのページソースを見てみる)  
```erb
<%= f.label :name %>
<%= f.text_field :name%>
```
は  
```html
<label for="user_name">Name</label>
<input id="user_name" name="user[name]" type="text" />
```
を生成し、
```erb
<%= f.label :email %>
<%= f.email_field :email %>
```
は  
```html
<label for="user_email">Email</label>
<input id="user_email" name="user[email]" type="email" />
```
を生成する。  
```erb
<%= f.label :password %>
<%= f.password_field :password %>
```
は  
```html
<label for="user_password">Password</label>
<input id="user_password" name="user[password]" type="password" />
```
を生成する。  
```type=```の部分に注目すると```f.text_field```では```type="text"```となり、通常のテキストフィールドを生成する。  
```f.password_field```では```type="password"```となり、入力文字が隠されたテキストフィールドとなる  
```f.email_field```の```type="email"```は一見通常のテキストフィールドと変わらないように見えるが、モバイル環境ではメールアドレス入力に適したキーボードが出るようになる。  
  
さらに、SCSSでフォームの整形をしておく  
```scss
.
.
.
/* forms */
input, textarea, select, .uneditable-input {
  border: 1px solid #bbb;
  width: 100%;
  margin-bottom: 15px;
  @include box_sizing;
}
input {
  height: auto !important;
}
```

### 7.3 ユーザー登録失敗
#### 7.3.1 正しいフォーム
/usersへのPOSTリクエストはcreateアクションに送られる(routes.rbに```resources :users```を追加したため)
createアクションでフォーム送信を受取り、User.newを使用してユーザオブジェクトを作成し、ユーザーを保存し、再度の送信用のユーザ登録ページを表示するという機能を作成する。  
ユーザ登録フォームのソースを見てみる  
```html
<form action="/users" class="new_user" id="new_user" method="post">
```
```action="/users"```と```method="post"```でPOSTリクエストを/usersに送っている  
まずはUsersControllerのcreateアクションに以下の変更を加える(※実装はまだ途中)  
app/controllers/user_controller.rb
```rb
  def create
    @user = User.new(params[:user])    # 実装は終わっていないことに注意!
    if @user.save
      # 保存の成功をここで扱う。
    else
      render 'new'
    end
  end
```
保存が成功したかどうかは@user.saveの戻り値(trueかfalse)で判定する  
5章のパーシャルの説明でも使用したrenderメソッドを再度使いまわしている  
この状態で無効なユーザデータでユーザ登録データを送信してみる。
デバッグ情報には以下のようになっている  
```
{"utf8"=>"✓",
 "authenticity_token"=>"Fz3IrSiR1RxnvCeBOPAfTnPUJESLKleVzyTODtAlerOzZ6ZEgapQDYGiAluDLkpHynmrkMUiMXm17DOseks/bw==",
 "user"=>{"name"=>"Foo Bar", "email"=>"foo@invalid", "password"=>"[FILTERED]", "password_confirmation"=>"[FILTERED]"},
 "commit"=>"Create my account"}
```
このうちパラメータハッシュのuserの部分を見てみる  
```
"user"=>{"name"=>"Foo Bar", "email"=>"foo@invalid", "password"=>"[FILTERED]", "password_confirmation"=>"[FILTERED]"}
```
この部分はUserコントローラにparamsとして渡される  
フォーム送信の結果が送信された値に対応する属性とともにuserハッシュに格納される  
このハッシュキーがinputタグにあったname属性になる  
```html
<input id="user_email" name="user[email]" type="email" />
```
```user[email]```という値はuserハッシュの:emailキーの値と一致する  
ハッシュのキーはデバッグ情報では文字列となっているが、Railsは文字列ではなく、params[:user]のように「シンボル」としてUsersコントローラに渡している  
この性質により、このハッシュはUser.newの引数で必要となるデータと完全に一致する  
次のようなコード(マストアサインとよぶ)は  
```rb
@user = User.new(params[:user])
```
実際にはこのコードとほぼ同じ  
```rb
@user = User.new(name: "Foo Bar", email: "foo@invalid",
                 password: "foo", password_confirmation: "bar")
```

#### 7.3.2 Strong Parameter
下記のようにマスアサインメント(引数にハッシュを用いてメソッド呼び出し)を利用して簡単に設定可能だが  
```rb
@user = User.new(params[:user])
```
不正なリクエストによりparamsに想定外なカラムを指定していた場合、想定していないデータベースの更新をユーザに許すことになる。(例えば管理者権限のカラムが存在する場合など)  
上記の```params[:user]```はparamsハッシュ全体で初期化していることになっている。  

そういった脆弱性をマスアサインメント脆弱性という。  
これの対策方法としてStrong Parameterというものがある。  
Strong Parameterを用いることによって、必須のパラメータと許可されたパラメータを指定することができる。  
  
今回の場合```:user```を必須とし、名前、メールアドレス、パスワード、パスワードの確認の属性を必須としたい。  
次のような記述による実現できる  
```rb
params.require(:user).permit(:name, :email, :password, :password_confirmation)
```
このコードの戻り値は、paramsハッシュのバージョンと、許可された属性です (:user属性がない場合はエラーになります)。  
これらのパラメータを使いやすくするために、user_paramsという外部メソッドを用意して使用するのが慣習になっています。  
app/controllers/user_controller.rb  
```rb
class UsersController < ApplicationController
  .
  .
  .
  def create
    @user = User.new(user_params)
    if @user.save
      # 保存の成功をここで扱う。
    else
      render 'new'
    end
  end
  private
    def user_params
        params.require(:user).permit(:name, :email, :password,
                                   :password_confirmation)
    end
end
```
privateキーワード以降がprivateメソッドであることを強調するためにprivate以降のメソッドのインデントを１つ下げている  

#### 7.3.3 エラーメッセージ
有効でないユーザモデルを保存しようとするとエラーメッセージが文字列リストに格納される  
app/models/user.rbにvalidatesを設定しているため。  
```rb
$ rails console
>> user = User.new(name: "Foo Bar", email: "foo@invalid",
?>                 password: "dude", password_confirmation: "dude")
>> user.save
=> false
>> user.errors.full_messages
=> ["Email is invalid", "Password is too short (minimum is 6 characters)"]
```
このメッセージをブラウザで表示するには、ユーザーのnewページでエラーメッセージのパーシャル (partial) を出力します。このとき、form-controlというCSSクラスも一緒に追加することで、Bootstrapがうまく取り扱ってくれるようになります  
app/views/users/new.html.erb  
```erb
<% provide(:title, 'Sign up') %>
<h1>Sign up</h1>
<div class="row">
  <div class="col-md-6 col-md-offset-3">
    <%= form_for(@user) do |f| %>
     <%= render 'shared/error_messages' %>
     <%= f.label :name %>
      <%= f.text_field :name, class: 'form-control' %>
     <%= f.label :email %>
      <%= f.email_field :email, class: 'form-control' %>
     <%= f.label :password %>
      <%= f.password_field :password, class: 'form-control' %>
     <%= f.label :password_confirmation, "Confirmation" %>
      <%= f.password_field :password_confirmation, class: 'form-control' %>
    <%= f.submit "Create my account", class: "btn btn-primary" %>
    <% end %>
  </div>
</div>
```
’shared/error_messages’というパーシャルをrender (レンダリング) している点に注目してください。  
複数のコントローラに渡るビューの場合、sharedディレクトリを使用する  
```
$ mkdir app/views/shared
```
app/views/shared/error_messages.html.erb  
```erb
<% if @user.errors.any? %>
  <div id="error_explanation">
    <div class="alert alert-danger">
      The form contains <%= pluralize(@user.errors.count, "error") %>.
    </div>
    <ul>
    <% @user.errors.full_messages.each do |msg| %>
      <li><%= msg %></li>
    <% end %>
    </ul>
  </div>
<% end %>
```
any?メソッドはempty?メソッドと互いに補完し合う  
１つでも存在すればtrueとなる  
pluralizeという英語テキスト専用ヘルパーメソッドを使用している  
```rb
>> helper.pluralize(1, "error")
=> "1 error"
>> helper.pluralize(5, "error")
=> "5 errors"
```
また、エラーメッセージにスタイルを与えるためのCSS id error_explanationも含まれていることに注目してください  
さらに、Railsは無効な内容が送信されて元のページに戻されるとdivで囲まれたエラー用cssクラス、field_with_errorsを返す  
```scss
.
.
.
/* forms */
.
.
.
#error_explanation {
  color: red;
  ul {
    color: red;
    margin: 0 0 30px 0;
  }
}
.field_with_errors {
  @extend .has-error;
  .form-control {
    color: $state-danger-text;
  }
}
```
ここでは@extend関数を使ってBootstrapのhas-errorというCSSクラスを適用している  

#### 7.3.4 失敗時のテスト  
Railsはフォーム用のテストを書くことができ、それを自動化できる  
まずは新規登録用の統合テストを書く  
```
rails generate integration_test users_signup
```
リソース名は複数形で書く慣習があるのでusers_signupとしている。  
このテストでは無効なユーザ情報で登録ボタンを押したときにユーザが作成されないことを検証する  
確かめる方法としてcountメソッドを用いる  
```
$ rails console
>> User.count
=> 1
```
まずは、getメソッドを使ってsignupページにアクセスできるか確かめる  
次にassert_selectメソッドを用いてHTML要素を検証する  
ただ、フォーム送信をテストするためにはusers_pathに対してPOSTリスエストを送る必要がある  
次のようにpost関数を使って実現する  
```rb
assert_no_difference 'User.count' do
  post users_path, params: {user: {name: "",
    email: "user@invalid",
    password: "foo",
    password_confirmation: "bar"}}
end
```
createアクションのUser.newで期待されるデータをparams[:user]というハッシュにまとめている  
assert_no_differenceメソッドのブロック内でpostを使い、メソッドの引数に'User.count'と書いている  
これはブロック内を実行する前後でUser.countの値が変わらないことを確かめている  
最終的にこのテストは次のコードになる  
test/integration/users_signup_test.rb  
```rb
require 'test_helper'
class UsersSignupTest < ActionDispatch::IntegrationTest
  test "invalid signup information" do
    get signup_path
    assert_no_difference 'User.count' do
      post users_path, params: { user: { name:  "",
                                         email: "user@invalid",
                                         password:              "foo",
                                         password_confirmation: "bar" } }
    end
    assert_template 'users/new'
  end
end
```
送信に失敗したときにnewアクションが再描画されるはずなので、assert_templateメソッドを使ったテストも含まれている(assert_templateはページの表示をテストする)  
  
### 7.4 ユーザ登録成功  
新規登録ユーザをデータベースに登録できるようにする  
→ユーザプロフィールの表示  
#### 7.4.1 登録フォームの完成  
現状では正しいユーザ情報を入力してもエラーが発生する  
createアクションに対するビューのテンプレートが存在しないため。  
テンプレートを作成する以外の方法で、別ページにリダイレクトするという選択肢もある(Railsではこっちの方が一般的)  
  
保存とリダイレクトを行うuserのcreateアクション  
app/controllers/users_controller.rb
```rb
class UsersController < ApplicationController
  .
  .
  .
  def create
    @user = User.new(user_params)
    if @user.save
      redirect_to @user
    else
      render 'new'
    end
  end
private
def user_params
  .
  .
  .
```
redirect_to @userという一文は```redirect_to user_url(@user)```に等しい

user_urlは5章で登場した名前付きルートの書き方である

この場合user_urlは/users/[ユーザID]のURLを返す。

参考：[_pathメソッドと_urlメソッドの使い分け](http://qiita.com/higeaaa/items/df8feaa5b6f12e13fb6f)  
  
#### 7.4.2 flash  
登録時にリダイレクトしたプロフィールページにはウェルカムメッセージを表示し、以降のプロフィールページへのアクセス時には表示をしない  
railsではflashという特殊な変数を用いて実現する  
app/controllers/users_controller.rb
```rb
class UsersController < ApplicationController
  .
  .
  .
  def create
    @user = User.new(user_params)
    if @user.save
        flash[:success] = "Welcome to the Sample App!"
      redirect_to @user
    else
      render 'new'
    end
  end
  .
  .
  .
```  
この変数はハッシュのように扱い、railsの慣習に習って:successというキーに成功メッセージを代入している  
  
今回はflashに登録してあるキーを全て調べ表示するようにしてみる  
ビューにはこのように書く  
```erb
<% flash.each do |message_type, message| %>
  <div class="alert alert-<%= message_type %>"><%= message %></div>
<% end %>
```
erbとhtmlの書き方が混ざっているがそれを改善することも可能(後述)  
alert-<%=message_type%>というクラスがあるが、埋め込み部分にはflashのキーが入るのでalert-successなどのクラスが指定される  
なお、Bootstrap CSSではflashのクラス用に4つのスタイル(success, danger, info, warning)を持っている  
上記のerbから、　　
```rb
flash[:success] = "Welcome to the sample App!"
```
このコードは　　
```html
<div class="alert alert-success">Welcome to the sample App!</div>
```
となる。最終的な埋め込みRubyのレイアウトはこのようになる  
app/view/layouts/application.html.erb  
```erb
<!DOCTYPE html>
<html>
  .
  .
  .
  <body>
    <%= render 'layouts/header' %>
    <div class="container">
      <% flash.each do |message_type, message| %>
        <div class="alert alert-<%= message_type %>"><%= message %></div>
      <% end %>
      <%= yield %>
      <%= render 'layouts/footer' %>
      <%= debug(params) if Rails.env.development? %>
    </div>
    .
    .
    .
  </body>
</html>
```
  
#### 7.4.3 実際のユーザ登録  
次のコマンドで一旦データベースをリセットする  
```
$ rails db:migrate:reset
```
次に最初のユーザ登録をしてみる  
Name: Rails Tutorial  
Email: example@railstutorial.org  
登録すると7.4.2で用意したメッセージが表示される。ページを更新されるとメッセージは消える  

#### 7.4.4 成功時のテスト  
有効な送信に対するテストを書く  
今回は有効な送信をして、ユーザ数が変更された(増えた)ことを確認するテストを書く  
よってassert_differenceというメソッドを使って書く  
```
assert_difference 'User.count', 1 do
  post users_path, …
end
```
assert_no_differenceと同様に第一引数には'User.count'の文字列で書かれたメソッド呼び出し、そして第二引数には比較した結果の差異が入る  
最終的に次のテストを実装する  
test/integration/users_signup_test.rb
```rb
require 'test_helper'
class UsersSignupTest < ActionDispatch::IntegrationTest
  .
  .
  .
  test "valid signup information" do
    get signup_path
    assert_difference 'User.count', 1 do
      post users_path, params: { user: { name:  "Example User",
                                         email: "user@example.com",
                                         password:              "password",
                                         password_confirmation: "password" } }
    end
    follow_redirect!
    assert_template 'users/show'
  end
end
```
postリクエストの後にfollow_redirect!というメソッドを呼び出している  
これはpostリクエストを送信した結果を見て指定されたリダイレクト先に移動するメソッド  
したがってこの直後ではusers/showが表示されているはず。asset_templateでそれをテストしている。  
  
content_tagを用いてレイアウトの中にflashを埋め込む  
app/views/layouts/application.html.erb
```erb
<!DOCTYPE html>
<html>
      .
      .
      .
      <% flash.each do |message_type, message| %>
        <%= content_tag(:div, message, class: "alert alert-#{message_type}") %>
      <% end %>
      .
      .
      .
</html>
```
  
### 7.5 プロのデプロイ  
下準備にこれまでの変更をmasterブランチにマージしておく  
```
$ git add -A
$ git commit -m "Finish user signup"
$ git checkout master
$ git merge sign-up
```
#### 7.5.1 本番環境でのSSL
本番環境ではフォーム送信をネットワーク越しに流すことになり、これらのデータは途中で補足できてしまうというセキュリティ上の欠陥がある。  
これを修正するためにSecure Socket Layer(SSL)を使う。  
これはローカルのサーバからネットワークに流れる前に大事な情報を暗号化する技術  
  
SSLを有効化するにはproduction.rbという本番環境用の設定ファイルの１行を修正するだけで済む  
config/environments/production.rb  
```rb
Rails.application.configure do
  .
  .
  .
  # Force all access to the app over SSL, use Strict-Transport-Security,
  # and use secure cookies.
  config.force_ssl = true
  .
  .
  .
end
```
本番用のWebサイトでSSLを有効にするためにはSSL証明書をドメインごとに購入する必要がある  
ただし、Heroku上でアプリケーションを動かしHerokuのSSL証明書に便乗する方法だと購入する必要はない  
  
#### 7.5.2 本番環境用のWebサーバ  
HerokuではデフォルトではWeBrickというサーバを使っている(セットアップが簡単だが多くのトラフィックに弱い)  
今回はPumaを使う  
基本的に[HerokuのPumaドキュメント](https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server)に従って設定していけば良い  
ドキュメントの通りに以下の変更を加える  
config/puma.rb  
```rb
workers Integer(ENV['WEB_CONCURRENCY'] || 2)
threads_count = Integer(ENV['RAILS_MAX_THREADS'] || 5)
threads threads_count, threads_count
preload_app!
rackup      DefaultRackup
port        ENV['PORT']     || 3000
environment ENV['RACK_ENV'] || 'development'
on_worker_boot do
  # Worker specific setup for Rails 4.1+
  # See: https://devcenter.heroku.com/articles/
  # deploying-rails-applications-with-the-puma-web-server#on-worker-boot
  ActiveRecord::Base.establish_connection
end
```
最後にProcfileと呼ばれるHeroku上でPumaのプロセスを走らせる設定ファイルを作成する  
./Procfile  
```
web: bundle exec puma -C config/puma.rb
```
  
#### 7.5.3 本番環境へのデプロイ  
```
$ rails test
$ git add -A
$ git commit -m "Use SSL and the Puma webserver in production"
$ git push
$ git push heroku
$ heroku run rails db:migrate
```
