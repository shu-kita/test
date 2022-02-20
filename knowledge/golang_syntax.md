# Go 基本文法

## 変数
Goでは使用していない変数があれば、エラーになるので注意

var 変数名 型　で定義する
```
var number int
```

= 値　とすることで定義と代入を同時に行う
```
var number int = 100
```

同一の型の変数を一度に複数定義する
```
var t, f bool               //定義のみ
var t, f bool = true, false //同時に代入
```

異なる型の変数を一度に複数定義する
```
var (
  num int = 100
  str string = "Hello World"
)
```

暗黙的な変数定義
```
num := 100
str := "Hello World"
```

## 定数
定数は関数外のグローバルの領域で定義することが多い  
大文字小文字でグローバル・プライベートを区別できる  

const 変数名 = 値　で定義する
```
const Pi = 3.14
```

一度に複数の定数を定義
```
const(
  mail = "abcd@a.com"
  name = "kita"
)
```

値に何も設定しない場合
```
const (
  A = 1
  B
  C = "str"
  D
)
```
Bには1、Dには"str"が設定される  
1つの定数で値を指定しない場合はエラーになる

iota使用した連番の定義
const (
  c0 = iota
  c1
  c2
)
c0には0、c1には1、c2には2が設定される





## 基本型

### int
整数を扱う型  
以下のような種類があり、最小値・最大値が異なる

|  型  |  最小値  |  最大値  |
| ---- | ---- |  ----  |
|  int8  |  -128  |  127  |
|  int16  |  -32768  |  32767  |
|  int32  |  -2147483648  |  2147483647  |
|  int64  |  -9223372036854775808  |  9223372036854775807  |

以下のように定義した場合、整数値が-128 ~ 127の範囲を出た際エラーになる
```
var num int8 = 整数値
```

intと定義すると環境のbit数に合わせ、int32かint64と同じ最小値・最大値になる  
※int64で定義した変数とintで定義してint64扱いの変数を足し算するとエラーになる  
　以下のようにキャストすることでエラーを回避できる
```
var (
  i1 int = 100
  i2 int64 = 150
)

fmt.Println(i1 + int(i2)) //250
```

### float
浮動小数点数を扱う型
以下のような種類がある

|  型  |  有効数字  |
| ---- | ---- |
|  float32  |  6桁  |
|  float64  |  14桁  |

暗黙的な型定義の場合、float64として定義される  
型変換は以下のように行える
```
float32(float64の値・変数)
float64(float32の値・変数)
```

無限大とNaNがある  
正のfloatの数値を0.0で割ると、正の無限大  
負のfloatの数値を0.0で割ると、負の無限大  
0.0を0.0で割ると、NaN  
```
pinf := 1.0 / 0.0
fmt.Println(pinf) //+Inf

ninf := -1.0 / 0.0
fmt.Println(ninf) //-Inf

nan := 0.0 / 0.0
fmt.Println(nan) //NaN
```

### string
文字列を扱う型  
以下のように定義する
```
var str1 string = "Hello World"
var str2 string = "300"
```
変数[index]で文字列中の指定した位置の文字を取得できる  
取り出した値はbyte型なので、string()でキャストする
```
fmt.Println(str1[0])         //72
fmt.Println(string(str1[0])) //H
```

複数行の文字列は「``」で囲う
```
var str3 string = `
    Hello
    world
`
```

### 配列
複数の要素を同時に扱える型  
要素の数は不変  
以下のように定義すると、1, 2, 3という3つの数値を持つ配列を定義できる
```
var array1 [3]int = [3]int{1,2,3}
```
型は[3]intで、[4]intの配列を代入しようとするとエラーになる

以下のように定義すると、要素の数を自動でカウントする  
暗黙的な定義でなければエラーになる
```
array2 := [...]string{"a", "b"}
```
型は[2]string

変数名[index]で値を取り出し、  
変数名[index] = 値で代入する
```
fmt.Println(array1[0]) //1
fmt.Println(array2[1]) //a

array2[1] = "xxx"
fmt.Println(array2[1]) //xxx
```

要素の数はlen()で取り出す
```
fmt.Println(len(array1)) //3
```

### interface
様々な型と互換がある型  
初期値はnil  
以下のように定義する
```
var x interface{}
fmt.Println(x) //<nil>
```

様々な型の値を代入可能
```
x = 1
fmt.Println(x) //1
x = "a"
fmt.Println(x) //a
x = [2]int{1, 2}
fmt.Println(x) //[1 2]
```

型特有の計算はできない  
例：intの値を代入したinterface型の変数とint型の変数を足すことはできない
```
x = 1
fmt.Println(x) //1

var i int = 2
fmt.Println(x + i) //エラーになる
```
型そのものが変化するわけではない

## 型変換

### int ⇔ float
int型とfloat型の変換には、int()やfloat64()を使う  

* int ⇒ float
  ```
  i1 := 100
  f1 := float64(i1)

  fmt.Printf("%T\n", i1) //int
  fmt.Printf("%T\n", f1) //float64
  ```
* float ⇒ int  
小数点以下は切り捨てられる
  ```
  f2 := 10.5
  i2 := int(f2)

  fmt.Println(i2) //10
  fmt.Printf("%T", i2) //int
  ```

### int ⇔ string
int型とstring型の変換には、strconvのAtoi(), Itoa()を使う

* int ⇒ string
  ```
  var i1 int = 500
  fmt.Printf("%T\n", i1) //int
  s1 := strconv.Itoa(i1)
  fmt.Printf("%T\n", s1) //string
  ```

* string ⇒ int
  ```
  var s2 string = "100"
  fmt.Printf("%T\n", s2)    //string
  i2, _ := strconv.Atoi(s2) //変数_ にすることで2つ目の変数を破棄(使わなくてもよい)
  fmt.Printf("%T\n", i2)    //int
  ```

### string ⇔ byte
string型とbyte型の変換には、string()や[ ]byte()を使う
* string ⇒ byte
  ```
  var s1 string = "kita"
  b1 := []byte(s1)
  fmt.Println(b1)        //[107 105 116 97]
  fmt.Printf("%T\n", b1) //[]unit8
  ```
* byte ⇒ string
変数b1をこちらでも使う
  ```
  s2 := string(b1)
  fmt.Println(s2) //kita
  ```


### 演算子

* 算術演算子  

  |  記号  |  動作  |
  |  ----  |  ----  |
  |  +  |  数値の加算、文字列の結合  |
  |  -  |  減算  |
  |  *  |  乗算  |
  |  /  |  除算  |
  |  %  |  余剰  |

  インクリメントとデクリメントはそれぞれ「++」と「--」で行える

* 比較演算子

  |  記号  |  動作  |
  |  ----  |  ----  |
  |  ==  |  等しい  |
  |  < , >  |  ○○を超える、○○未満  |
  |  <= , >=  |  ○○以上、○○以下  |

* 論理演算子  
||でor条件、&&でand条件、!で否定ができる