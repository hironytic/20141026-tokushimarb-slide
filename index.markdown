# tokushima.rb 3 slide

### Author

* か (ka)
* [kaosfield](http://www.kaosfield.net)


## はじめに

このスライド自体のリポジトリは [kaosf/20141026-tokushimarb-slide](https://github.com/kaosf/20141026-tokushimarb-slide) にあります

間違いを発見したら Issue あるいは PullRequest を下さい


## 前回までのあらすじ

* オブジェクト指向
* Rubyのセットアップ出来た(進んだ人はRailsも)


## 今回話すこと

```
method_missing
```

メソッドが無いときに呼ばれるメソッド


## 試しましょう

```ruby
class A
  def initialize(x)
    @x = x
  end

  def m
    @x
  end
end

a = A.new 1

puts a.m #=> 1
```


## まずは普通に

`A` という名前のクラスを作成

コンストラクタで引数を渡してそれがメンバ変数として保存される

`m` という名前のメソッドでそのメンバ変数が取得出来る


## ありえないメソッドを呼んでみる

```ruby
a.n
```

まったく同じ数値にはならないと思うが

````
<main>': undefined method `n' for
#<A:0x007ffa70909878 @x=1> (NoMethodError)
```

のような `n` が無いですというエラーが出るはず


## `method_missing` 追加

```ruby
class A
  # ...
  def method_missing(name, *args)
  end
end
```

ここでおもむろに上記のようなメソッドを追加してみる


## エラーが出ない

先ほどの

```
a.n
```

を実行してみるとエラーが出なくなった

何故だろう


## 答え

元々存在した `method_missing` メソッドが  
オーバライドされたから


## 公式ドキュメントは読もう

[method_missing](http://docs.ruby-lang.org/ja/2.0.0/method/BasicObject/i/method_missing.html)

※余談ですが公式のドキュメントは常に参照しましょう

※意外と知らないことが書いてあったりします


## 引数について

ドキュメントによると  
第一引数がメソッド名のシンボルで  
第二引数がメソッドに与えられた引数


## 確認してみる

```ruby
class A
  # ...
  def method_missing(name, *args)
    p name
    p args
  end
end

a.n 1, 2, nil, 'a'
```

結果

```
:n
[1, 2, nil, "a"]
```

確かに


## この性質は面白い

…と思って欲しい

ありもしないメソッドが呼ばれたときに  
その事実を捕捉出来ると共に  
与えられた情報(メソッド名と全引数)  
も捕捉することが出来る


## 何が出来るか

昔のRails(3まで)には動的ファインダというものがあった

※今でもあるが非推奨になっている

```ruby
Blog.find_by_title('foo bar')
# SQL で言うところの
# SELECT * FROM blogs WHERE title = 'foo bar';
```

※`Blog` というモデルがあり `title` というフィールドを持っていると想定


## 注意して欲しい

`Blog` というモデルのクラスには毎度毎度

`find_by_title` などというメソッドは定義されていない

にも関わらず呼び出すことが出来た


## 簡単に説明すると

```ruby
class Blog
  def self.method_missing(name, *args)
    if name.to_s =~ /find_by_(.*)/
      #where(['? = ?', $1, args.first)
      p $1
      p args.first
    else
      super.method_missing name, args
    end
  end
end
```

実際は全然違うがそれはこの際無視する

`where` は実行出来ないので  
ここでは情報を表示してみるだけ


先ほどの例で

```ruby
Blog.find_by_title 'foo bar'
```

だと `"title"` と `"foo bar"` が出る

```ruby
Blog.foo_method
```

だと基底クラスの `method_missing` が呼ばれるため  
`NoMethodError` が発生する


## 用法用量を守ろう

お察しの通りコードの読み通しがクソになる

意味もなく使うのやめよう

`respond_to_missing?` も正しくオーバライドしよう  
`respond_to?` がちゃんと機能するように。


すごいメタプログラミング楽しく学ぼう

ちなみにメソッドを動的に追加したいだけなら  
`define_method` というメソッドがあります

参考: [define_method](http://docs.ruby-lang.org/ja/2.0.0/method/Module/i/define_method.html)


## 今回のコード

スライドからコピーしてもいいですが  
一応 [kaosf/20141026-tokushimarb-codes](https://github.com/kaosf/20141026-tokushimarb-codes) にも置いておきます


おしまい
