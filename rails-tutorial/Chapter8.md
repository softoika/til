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
