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
