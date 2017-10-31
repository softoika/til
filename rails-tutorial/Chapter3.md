## 静的ページの生成  
2章のscaffold生成と同じように```rails generate```コマンドを使用する  
チュートリアルでは以下のようにコントローラを生成している
```rails generate controller StaticPages home help```  
StaticPagesがコントローラで、homeとhelpがアクション  

```rails generate```で生成したファイルや変更は```rails destroy```コマンドで削除することができる  
例：```rails destroy controller StaticPages home help```  
  
## テストの作成  
```rails generate controller```で作成されたコードに関するテストはtest/controllers/で自動生成時に作成されている  
以下は生成されたテストコードの例  
```rb
test "should get home" do
  get static_pages_home_url
  assert_response :success
end
```  
GETリクエストをhomeアクションに対して発行し、```assert_response :success```でHTTPのステータスコード(200OK)で成功を表す  
テストの実行は```rails test```コマンド  
  
テスト駆動開発など、テストを先に書くべきとき（例えば実装する機能の使用がはっきりとわかっているときなど）には、  
「RED・GREEN・REFACTOR」のステップを踏むといい  
「RED・GREEN・REFACTOR」とは  
1. まずは正常に動作するコードを書かなければパスできない(REDになる)テストを書く。正常に動作していないことを確認する(RED)  
2. エラーを元にテストをパスするコードを書きテストを成功させる(GREEN)  
3. コード重複を解決するなどしてリファクタリングを行う(REFACTOR)  
  
セレクタと呼ばれるアサーションメソッドassert_select  
```rb
assert_select "title", "Home | Ruby on Rails Tutorial Sample App"
```  
<title>タグ内に「Home | Ruby on Rails Tutorial Sample App」という文字列があるかチェックする  
  
  
Guardによってテストを自動実行する```bundle exec guard```  
これをするための設定や準備はチュートリアルを参照  

