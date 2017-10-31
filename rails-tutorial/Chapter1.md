rails newでアプリケーションの作成をする  
```rails <バージョン> new <アプリケーション名>```  
でバージョンを指定してアプリケーションを作成する
例：```rails _5.0.0.1_ new hello_app```
  
Gemfileの設定  
バージョン指定する場合は```gem 'uplifier', '>=1.3.0'```  
もしくは```gem 'coffee-rails', '~> 4.0.0'```で指定する  
前者は1.3.0以上の最新バージョンのgemがインストールされる  
後者は4.0.0 < 4.0.x < 4.1のマイナーバージョン4.0.xのgemがインストールされる  
Gemfileを更新する場合```bundle install```で反映させる  
  
ローカル環境でRailsサーバーを実行するには、アプリケーションのディレクトリに入り、  
```rails server```で実行する
Cloud9のクラウドIDEで実行するには```rails server -b $IP -p $PORT```で実行する  
メニューバー > Window > Share... からapplication:のURLで確認できる