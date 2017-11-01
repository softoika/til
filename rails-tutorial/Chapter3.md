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
これをするための設定や準備は[チュートリアル](https://railstutorial.jp/chapters/static_pages?version=5.1#sec-guard)を参照  

## 埋め込みRuby
```erb
<% provide(:title, "Home") %>
```
erbファイルはHTML中にrubyコードを埋め込めるテンプレートシステム  
provideメソッドはRailsのメソッドであり、文字列```"Home"```とラベル```:title```を結びつけている  
```<%``` と ```%>``` で囲った中に記述できる  

```erb
<title><%= yield(:title)%> | Rails Sample App</title>
```
```<%= ... %>``` という記述は中で得られた値を実際に出力された値としてHTMLに挿入することができる  
そしてyieldメソッドによって:titleの文字列を取り出してテンプレートに挿入している  
yieldは引数として渡されたブロックを実行する  
```erb
へっだー
<%= yield %>
ふったー
```
他にyieldを使う場面としては、ページのヘッダーやフッターを他のerbファイルに記述して使いまわしたいときに使われる  

### headタグ内に書いておくべき埋め込みRuby
```erb
<%= csrf_meta_tags %>
<%= stylesheet_link_tag  'application', media: 'all', 'data-turbolinks-track': 'reload' %>
<%= javascript_include_tag 'application',  'data-turbolinks-track': 'reload' %>
```
csrf_meta_tagsはWeb攻撃手法のCSRFを防いでくれるもの  
あとはcssファイルとjsファイルの関連付けをやってくれるもの  
