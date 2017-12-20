#### Shinjuku.rb #56 2017年開発厄落としLT大会
- --
## case-when と `===` と gem

---

## 自己紹介
- - -

* なまえ  : おしょー
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* 言語：Ruby、C++、JavaScript
* エディタ：Vim
* ※：Rails はほとんどわからない

---

## [今年自作した gem](https://rubygems.org/profiles/osyo-manga)
- - -

* [iolite](https://github.com/osyo-manga/gem-iolite)
  * メソッドを遅延呼び出しするための gem
* [use_arguments](https://github.com/osyo-manga/gem-use_arguments)
  * ブロック引数を省略出来る gem
* [unmixer](https://github.com/osyo-manga/gem-unmixer)
  * mixin したモジュールを削除する gem
* [proc-unbind](https://github.com/osyo-manga/gem-proc-unbind)
  * `Proc` から `UnboundMethod` を定義する

---

#### アドベントカレンダー
- - -

* [Ruby Advent Calendar 2017](https://qiita.com/advent-calendar/2017/ruby)
  * [Ruby で型チェックを実装してみよう](http://secret-garden.hatenablog.com/entry/2017/12/01/000154)
* [一人 Ruby Advent Calendar 2017](https://qiita.com/advent-calendar/2017/ruby_pink_bangbi)
  * Ruby に関する小ネタみたいなのを書いてます
* [一人 vimrc Advent Calendar 2017](https://qiita.com/advent-calendar/2017/vimrc_pink_bangbi)
  * vimrc に関する小ネタみたいなのを書いてます

---

## 今日話すこと
- - -

## case-when で遊ぶ

---

#### case-when とは
- - -

```ruby
def func n
	case n
	when 1
		"one"
	when 2
		"two"
	when 3
		"three"
	else
		"other"
	end
end

p (0..5).map &method(:func)
# => ["other", "one", "two", "three", "other", "other"]
```

内部で === を使って比較を行っている  <!-- .element: class="fragment" -->

---

#### Regexp オブジェクト
- - -

正規表現で判定

```ruby
def check str
	case str
	when /\d+/
		"number"
	when /\w+/
		"string"
	else
		"other"
	end
end

p check "42"    # => "number"
p check "homu"  # => "string"
p check "+++"   # => "other"
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="3,5"></span>

---

#### Range オブジェクト
- - -

任意の範囲に該当するかどうか判定

```ruby
def level n
    case n
    # 1〜3の範囲にマッチする場合みたいに記述することが出来る
    when (1..3)
        "low"
    when (4..6)
        "middle"
    when (7..9)
        "high"
    else
        "other"
    end
end

p level 3   # => "low"
p level 5   # => "middle"
p level 9   # => high
p level 32  # => other
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="4,6,8"></span>

---

#### Class オブジェクト
- - -

インスタンスが任意のクラスかどうか判定する

```ruby
def twice x
    case x
    # x が各クラスのインスタンスだった場合
    when Integer
        x + x
    when String
        x.to_i + x.to_i
    when Symbol
        x.to_s.to_i + x.to_s.to_i
    else
        "???"
    end
end

p twice 42        # => 84
p twice "1234"    # => 2468
p twice :"5678"   # => 11356
p twice [1, 2, 3] # => "???"
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="4,6,8"></span>

---

#### Proc オブジェクトを渡したり
- - -

`Proc#===` は `Proc#call` を呼び出す

```ruby
def check value
	case value
	# Proc#=== は Proc#call と同じ
	when proc { |it| 0 < it }
		"plus"
	when proc { |it| 0 > it }
		"minus"
	when proc { |it| 0 == it }
		"zero"
	end
end

p check 3   # => "plus"
p check -6  # => "minus"
p check 0   # => "zero"
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="4,6,8"></span>

---

#### Method オブジェクトを渡したり?
- - -

```ruby
def check value
	case value
	when 0.method(:<)
		"plus"
	when 0.method(:>)
		"minus"
	when 0.method(:==)
		"zero"
	end
end

p check 3   # => "plus"
p check -6  # => "minus"
p check 0   # => "zero"
```
<span class="code-presenting-annotation fragment current-only" data-code-focus="3,5,7"></span>

Ruby 本体にパッチを投げて Ruby 2.5 から Method#=== が Method#call として動作するようになりました！  <!-- .element: class="fragment" -->

---

#### Set オブジェクトを渡したり（Ruby 2.5）
- - -

Set#=== は Set#include? のエイリアス

```ruby
case :apple
# Set のいずれかに含まれていれば
when Set[:potato, :carrot] then 'vegetable'
when Set[:apple, :banana]  then 'fruit'
end
#=> "fruit"
```

`===` ブームの流れが来てるぞ…。  <!-- .element: class="fragment" -->


---

## 便利 gem
- - -

`case-when` で使えるような gem をいろいろとつくってみた。

---

## [array-eqq](https://github.com/osyo-manga/gem-array-eqq)
- - -

`Array#===` を定義した gem。

```ruby
require "array/eqq"

using Array::Eqq

p [1, 2] === [1, 2]
# => true

p [Integer, String] === [1, "homu"]
# => true

# 一部の要素が false なら false
p [Integer, String] === [1, :homu]
# => false

# 要素数が違っていても false
p [Integer, String] === [1, "homu", 3]
# => false
```

>>>

#### Array#=== を使って FizzBuzz
- - -

```ruby
def fizzbuzz n
	_ = proc { true }
    # 3 で割った余りと 5 で割った余りを配列として渡す
	case [n % 3, n % 5]
	when [0, 0]		# 両方共 0なら
		"FizzBuzz"
	when [0, _]		# 3で割った余りだけが 0 なら
		"Fizz"
	when [_, 0]		# 5で割った余りだけが 0 なら
		"Buzz"
	else
		n
	end
end

p (1..20).map &method(:fizzbuzz)
# => [1, 2, "Fizz", 4, "Buzz", "Fizz", 7, 8, "Fizz", "Buzz", 11, "Fizz", 13, 14, "FizzBuzz", 16, 17, "Fizz", 19, "Buzz"]
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="4"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="5"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="7"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="9"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="11"></span>

---

## [laurel](https://github.com/osyo-manga/gem-laurel)
- - -

なんかいい感じに `|` や `&` をしてくれるライブラリ

```ruby
require "laurel"

using Laurel::Refine
# (a | b).hoge => a.hoge || b.hoge

# String === rhs || Integer == rhs と同じ
p (String | Integer) === 10         # => true
p (String | Integer) === "homu"     # => true
p (String | Integer) === :homu      # => false

# String === other && /\d+/ === other と同じ
p (String & /\d+/) === "1234" # => false
p (String & /\d+/) === 10     # => false
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="4"></span>

>>>

```ruby
require "laurel"
using Laurel::Refine

def twice value
	case value
	# Integer === value || Float === value
	when Integer | Float
		value + value
	# Sting === value && /\d+/ === value
	when String & /\d+/
		twice value.to_i
	else
		"other"
	end
end

p twice 42       # => 84
p twice 3.14     # => 6.28
p twice "24"     # => 48
p twice "homu"   # => "other"
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="7,17,18"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="10,19"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="12,20"></span>

---

## まとめ
- - -

## case-when 楽しいのでみんなやろう

---

## 今年を振り返って、来年の抱負
- - -

* 楽しい gem をつくることができた
* LT や勉強会などに参加できた
* Ruby 本体にパッチを投げて採用された
* 来年は Rails や Hanami などの生産性のあることをやりたい

---

## ご清聴
## ありがとうございました
