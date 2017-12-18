# 9章　発展的ログイン機構
永続クッキーを使って長期のログイン維持を実現する
## 9.1 Remember me機能
### 9.1.1 記憶トークンと暗号化
記憶トークンと呼ばれるパスワードを生成し、そのパスワードを暗号化してデータベースに保存する  
最初に、Userモデルにremember_digest属性を追加する  
```
$ rails generate migration add_remember_digest_to_users remember_digest:string
$ rails db:migrate
```

|属性名|型|
|-----|--|
|id   |integer|
|name   |string   |
|...   |...   |
|password_digest   |string   |
|remember_digest   |string   |

記憶トークンの生成にはランダムな文字列の生成をするものを用いる  
```rb
$ rails console
>> SecureRandom.urlsafe_base64
=> "q5lt38hQDc_959PVoo6b7A"
```
urlsafe\_base64は英数字と-と_で構成される  
これをユーザーモデルのクラスメソッドとして定義する  
またよりRubyらしい書き方としてクラスメソッドを以下のように書く  
```rb
class User < ApplicationRecord
  ...

  # 渡された文字列のハッシュ値を返す
  def self.digest(string)
    cost = ActiveModel::SecurePassword.min_cost ? BCrypt::Engine::MIN_COST :
                                                  BCrypt::Engine.cost
    BCrypt::Password.create(string, cost: cost)
  end

  # ランダムなトークンを返す
  def self.new_token
    SecureRandom.urlsafe_base64
  end
end
```

記憶トークンをユーザーに関連付け、トークンに対応する記憶ダイジェストをデータベースに保存するrememberメソッドを作成する  
さらにremember_token属性が追加できない(セキュリティの都合のため)のでremember_tokenメソッドを使ってトークンにアクセスできるようにする  
```rb
attr_accessor :remember_token
```
最終的にdigestメソッドを流用してトークンを暗号化してデータベースに保存する  
app/models/user.rb  
```rb
class User < ApplicationRecord
  attr_accessor :remember_token
  ...
  # 渡された文字列のハッシュ値を返す
  def self.digest(string)
    cost = ActiveModel::SecurePassword.min_cost ? BCrypt::Engine::MIN_COST :
                                                  BCrypt::Engine.cost
    BCrypt::Password.create(string, cost: cost)
  end

  # ランダムなトークンを返す
  def self.new_token
    SecureRandom.urlsafe_base64
  end

  # 永続セッションのためにユーザーをデータベースに記憶する
  def remember
    self.remember_token = User.new_token
    update_attribute(:remember_digest, User.digest(remember_token))
  end
end
```

### 9.1.2 ログイン状態の保持
cookiesメソッドを用いてユーザーIDと記憶トークンを長期保持する  
app/helpers/sessions_helper.rb  
```rb
module SessionsHelper
  ...
  # ユーザーのセッションを永続的にする
  def remember(user)
    user.remember
    cookies.permanent.signed[:user_id] = user.id
    cookies.permanent[:remember_token] = user.remember_token
  end
  ...
end
```
1行目でremember_tokenを生成し、データベースにremember_digestを保存する  
2行目では、そのままユーザーIDを保持するのはセキュリティに問題が生じるので、signedメソッドで署名付きクッキーを用いる  
2,3行目ではクッキーを長期保持したいのでpermanentメソッドを用いている(20年間クッキーを保持できる)  

これらの保持はログイン時に実行する  
app/controllers/sessions_controller.rb  
```rb
class SessionsController < ApplicationController
...
  def create
    user = User.find_by(email: params[:session][:email].downcase)
    if user && user.authenticate(params[:session][:password])
      log_in user
      remember user ###
      redirect_to user
    else
      flash.now[:danger] = 'Invalid email/password combination'
      render 'new'
    end
  end
...
end
```

一方でビューからログイン状態を確認するときに用いられているcurrent_userメソッドは現状ではセッションにしか対応していない  
```rb
@current_user ||= User.find_by(id: session[:user_id])
```
これをクッキーを用いた永続セッションにユーザーIDがあればクッキーから探すというふうに変更する  
app/helpers/sessions_helper.rb  
```rb
module SessionsHelper
...
  # 記憶トークンcookieに対応するユーザーを返す
  def current_user
    if (user_id = session[:user_id])
      @current_user ||= User.find_by(id: user_id)
    elsif (user_id = cookies.signed[:user_id])
      user = User.find_by(id: user_id)
      if user && user.authenticated?(cookies[:remember_token])
        log_in user
        @current_user = user
      end
    end
  end
...
end
```
クッキーから探す場合、記憶トークンが暗号化されて保存されている記憶ダイジェストと同じものかユーザー認証のときと同じように確認する必要がある  
```rb
if user && user.authenticated?(cookies[:remember_token])
```
app/models/user.rb  
```rb
class User < ApplicationRecord
...
  # 渡されたトークンがダイジェストと一致したらtrueを返す
  def authenticated?(remember_token)
    BCrypt::Password.new(remember_digest).is_password?(remember_token)
  end
end
```
このメソッドではBCryptのパスワードとして見たremember_digestがremember_tokenを暗号化したものかどうかを返している
