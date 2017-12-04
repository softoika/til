## 8章 基本的なログイン機構  
認証システム(ブラウザがログインしている状態を保持し、ブラウザが閉じられたら破棄する仕組み)  と認可モデル(ログイン済みのユーザのみがアクセスできるページや扱える機能を制御する)  
を作る。  

### 8.1 セッション  
HTTPは状態を持たないプロトコルなので前のページの情報を引き継げない  
セッションは半永続的な接続をコンピュータ間に設定できるプロトコルで、HTTPとは独立している。(プロトコルの階層が異なる)  
cookies(ユーザのブラウザに保存される小さなテキストデータ)を使ってセッションを実装する  
まずは最初にブランチを切っておく  
```
$ git checkout -b basic-login
```

#### 8.1.1 Sessionsコントローラ  
ログイン・ログアウトの要素をRESTアクションに対応付けさせる  
ログインのフォームはnewアクション  
createアクションにPOSTリクエストを送信すると実際にログインする  
destroyアクションにDELETEリクエストを送信するとログアウトする  
([HTTPメソッドとRESTアクションの対応付](https://railstutorial.jp/chapters/sign_up?version=5.0#table-RESTful_users))  

Sessionsコントローラとnewアクションを作成  
```
$ rails generate controller Sessions new
```  
sessions_controllerとnewに対応するビューが生成された  
createやdestroyに対応するビューは必要ないのでnewのみ生成するようにした  

名前付きルーティングを使って、GETリクエストやPOSTリクエストをloginルーティングで、DELETEリクエストをlogoutルーティングで扱う  

リソースを使って標準的なRESTfulアクションをgetできるようにする  
config/routes.rb  
```rb
Rails.application.routes.draw do
  root   'static_pages#home'
  get    '/help',    to: 'static_pages#help'
  get    '/about',   to: 'static_pages#about'
  get    '/contact', to: 'static_pages#contact'
  get    '/signup',  to: 'users#new'
  get    '/login',   to: 'sessions#new'
  post   '/login',   to: 'sessions#create'
  delete '/logout',  to: 'sessions#destroy'
  resources :users
end
```
Sessionsコントローラで名前付きルートを使えるようにする  
test/controllers/sessions_controller_test.rb  
```rb
require 'test_helper'
class SessionsControllerTest < ActionDispatch::IntegrationTest
test "should get new" do
    get login_path
    assert_response :success
  end
end
```

ここまで追加した全ルートを確認する方法  
```
$ rails routes
```

#### 8.1.2 ログインフォーム  
ログインフォームのビューを作る  
必要なフィールドはEmailとPasswordのみ(NameとConfirmは不要)  
入力に誤りがあった場合はログインページをもう一度出してエラーメッセージを出す  
今回扱うセッションはActive Recordではないので、自動的にエラーメッセージを出すことはできない  
その代わりにflashを用いてエラーメッセージを出す  

7章のときは次のようにform_forヘルパーを使い、ユーザのインスタンス変数@userを引数に取っていた  
```erb
<%= form_for(@user) do |f| %>
  .
  .
  .
<% end %>
```
セッションフォームとユーザ登録フォームの違いは、セッションにはSessionモデルが存在していなく、@userなどのインスンタンス変数が存在しないこと。  
Railsでは```form_for(@user)```と書くだけで「フォームのactionは/usersというURLへのPOSTである」と自動判別してくれるが、  
セッションの場合はリソースの名前とそれに対応するURLを具体的に指定する必要がある  
```rb
form_for(:session, url: login_path)
```
最終的なセッションフォームのコードは次のようになる(基本的にはユーザ登録ページに則った書き方)    
app/views/sessions/new.html.erb  
```erb
<% provide(:title, "Log in") %>
<h1>Log in</h1>
<div class="row">
  <div class="col-md-6 col-md-offset-3">
    <%= form_for(:session, url: login_path) do |f| %>
       <%= f.label :email %>
      <%= f.email_field :email, class: 'form-control' %>
      <%= f.label :password %>
      <%= f.password_field :password, class: 'form-control' %>
      <%= f.submit "Log in", class: "btn btn-primary" %>
    <% end %>
<p>New user? <%= link_to "Sign up now!", signup_path %></p>
  </div>
</div>
```

フォームで入力した情報はparamsに引き渡される  
セッションに関するparamsの値は次のようになっている  
```rb
{session: {password: "foobar", email: "user@example.com"}}
```
メールアドレスの取得とパスワードの取得は以下のように書く  
```rb
params[:session][:email]
params[:session][:password]
```

ログインを実装する。  
ログインの処理は、まずデータベースからメールアドレスを持つユーザーを探す。  
ユーザーが見つかり、さらにパスワードによる認証に成功すれば、ユーザー情報のページにリダイレクトする。  
ユーザーが見つからないもしくは認証に失敗したら、ログインページを再描画し、エラーメッセージを表示する。  
app/controllers/sessions_controller.rb
```rb
class SessionsController < ApplicationController
  ...
  def create
    user = User.find_by(email: params[:session][:email].downcase)
    if user && user.authenticate(params[:session][:password])
      # ユーザーログイン後にユーザー情報のページにリダイレクトする
    else
      # エラーメッセージを作成する
      render 'new'
    end
  ...
end
```

エラーメッセージの表示にはflash.nowを使う  
これを使うことで、エラーメッセージの表示後にページ遷移してもエラーメッセージが残るということがなくなる  
flash.nowは現在のリスエストのみで有効なメッセージを表示する  
renderで表示する画面にメッセージを表示したい場合に用いる  
flash[:danger]のような使い方は、次のリクエストまで有効なメッセージが表示される  
redirect_toした先の画面で表示したい場合に用いる  

flash.nowでのエラーメッセージ表示  
```rb
flash.now[:danger] = "Invalid email/password combination"
```

### 8.2 ログイン
セッションを実装するにはapplication_controller.rbにヘルパーモジュールの読み込みを定義することで簡単に実装できる  
app/controllers/application_controller.rb
```rb
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
  include SessionsHelper
end
```

#### log_inメソッド
Sessionsヘルパーにlog_inというメソッドを定義し、sessionメソッドを使ってユーザーIDをcookiesで暗号化されたIDから取り出す。  
sessionメソッドはハッシュのように扱える  

app/helpers/sessions_helper.rb  
```rb
module SessionsHelper

  # 渡されたユーザーでログインする
  def log_in(user)
    session[:user_id] = user.id
  end
end
```
ヘルパーメソッドを定義できたのでsessions_controller.rbでログインして、ユーザーページにリダイレクトする  
app/controllers/sessions_controller.rb   
```rb
class SessionsController < ApplicationController
  ...
  def create
    user = User.find_by(email: params[:session][:email].downcase)
    if user && user.authenticate(params[:session][:password])
      log_in user
      redirect_to user
    else
  ...
end
```

#### 8.2.2 現在のユーザー
ユーザーIDを一時セッションに置く処理を書いたので、次はセッションからユーザーIDを取り出す処理をヘルパーメソッドとして書く  
current_userというメソッドを定義する。  
current_userは次のようにして使うことを目的としている  
```erb
<%= current_user.name %>
```
ユーザーをIDで検索する方法は従来ならUser.find(session[:user_id])とするが、findメソッドではユーザーが存在しない場合に例外が発生してしまう(session[:user_id]はnilを返し、それが例外の元となる)  
そこでfind_byメソッドを使うとnilをとっても例外を出さないことから、  
```rb
User.find_by(id: session[:user_id])
```
のように検索する。  
また、current_userメソッドが１リクエスト内で何度も呼び出されると何度もデータベースに問い合わせが起きてしまうので、インスタンス変数が未定義であれば検索結果をインスタンス変数に入れ、定義済みであればそのまま用いる処理とする(この処理をメモ化(memoization)という)  
```rb
if @current_user.nil?
  @current_user = User.find_by(id: sesssion[:user_id])
else
  @current_user
end
```
また、Rubyではor演算子を用いることでこれを簡略化して書くことができる  
```rb
@current_user = @current_user || User.find_by(id: session[:user_id])
```
Rubyの条件式はnilもfalseと扱われ、nilやfalse以外はtrueとして扱われる性質がある。  
or演算子の場合次のような評価の仕方もできる
```rb
>> @hoge
=> nil
>> "hoge"
=> "hoge"
>> @hoge = @hoge || "hoge"
=> "hoge"
>> @hoge = @hoge || "fuga"
=> "hoge"
```
また、Rubyはor演算子も+=のような短縮表現にすることができる  
```rb
@current_user ||= User.find_by(id: session[:user_id])
```

以上から、セッションヘルパーに以下のようなメソッドが定義される  
app/helpers/session_helper.rb  
```rb
module SessionsHelper
  ...
  # 現在ログイン中のユーザーを返す (いる場合)
  def current_user
    @current_user ||= User.find_by(id: session[:user_id])
  end
end
```

#### 8.2.3 レイアウトリンクを変更する  
ログインしているときとしていないときで異なるメニューを表示したい  
ヘルパーにlogged_in?というメソッドを用意してログイン状態かどうか返せるようにする  
app/helpers/session_helper.rb  
```rb
module SessionsHelper
  ...
  def logged_in?
    !current_user.nil?
  end
end
```
viewでは大まかに以下のようにしてリンクを切り替える  
```erb
<% if logged_in? %>
  # ログインユーザー用のリンク
<% else %>
  # ログインしていないユーザー用のリンク
<% end %>
```
ログイン状態ではユーザーページに飛べるようにしたい  
現在ログインしてるユーザのユーザーページへのリンクは次のように書く  
```erb
<%= link_to "Profile", current_user %>
```
この書き方は次の書き方の省略形となっている  
```erb
<%= link_to "Profile", user_path(current_user) %>
```
ログアウトもメニューからできるようにしたい  
```erb
<%= link_to "Log out", logout_path, method: :delete %>
```
最後のハッシュではHTTPリクエストでDELETEリクエストを送るように指示している  
ドロップダウンメニューの実現にはbootstrapの機能を使っている  
```erb
<li class="dropdown">
  <a href="#" class="dropdown-toggle" data-toggle="dropdown">
    Account <b class="caret"></b>
  </a>
  <ul class="dropdown-menu">
    <li><%= link_to "Profile", current_user %></li>
    <li><%= link_to "Settings", '#' %></li>
    <li class="divider"></li>
    <li>
      <%= link_to "Log out", logout_path, method: :delete %>
    </li>
  </ul>
</li>
```
ドロップダウン機能を使うにはjavascriptでjQueryとBootstrapを有効にすること  
app/assets/javascripts/application.js  
```js
//= require jquery
//= require bootstrap
```
