[←目次](index.md)


# 自作プログラミング言語講座 第1章 - 逆ポーランド記法計算機の作成 - 

## 今回作るもの
* **逆ポーランド記法**の計算機
  - プログラミング言語にまず要る要素は「計算」
  - 四則演算( + - * / ) が出来るようにする

* 文字列で与えられるソースコード(計算式)を解く
  - 読み込み -> 解釈 -> 実行、までを作る

---

## 逆ポーランド記法とは
### 計算式記法のひとつ
* 四則演算は、「計算対象が2つ」と「演算子」から成っている。ここでは、それの記述方法のこと。

### 逆ポーランド記法
* 人間の書く計算式では、「 `2 + 3` 」 のように、「`対象1 演算子 対象2`」 という順番に書く
  - 「中置記法」とも言う
* 逆ポーランド記法では、「 `2 3 +` 」 のように、「`対象1 対象2 演算子`」 という順番に書く
  - 「後置記法」とも言う
---

## Q. なぜ逆ポーランド記法が良いのか?
### A. コンピュータが解釈しやすいから
  - **スタック**と併用することで**前から順番に**解釈できる


## スタックとは
* データを「積み上げていく」データ構造
* イメージは「積み木」
  - 上に向かって積み上げていき、積み木を取るときは上から取る
  - LIFO (Last In First Out) とも言う

## スタックの例
* 初期状態: ` [ ] `
* 1を積む: ` [1 ] `
* 2を積む: ` [1 2 ] `
* 3を積む: ` [1 2 3 ] `
* 1個データを取り出す: `[1 2]`
  - 取り出したデータは、最後に入れた「`3`」
* 1個データを取り出す: `[1 ]`
  - 取り出したデータは、最後から2番目に入れた「`2`」
* 4を積む: ` [1 4 ] `

---

## スタック + 逆ポーランド記法の例①
例えば、「` 2 + 3` 」の場合、
  -  逆ポーランド記法では「` 2 3 + ` 」

* 「`2`」: 2をスタックに積む
  - スタックは `[2 ] `
* 「`3`」: 3をスタックに積む
  - スタックは `[2 3 ] `
* 「`+`」: スタックの最上部2つを取り出して、足し算し、結果をスタックに積む
  - スタックは `[5 ] `


## スタック + 逆ポーランド記法の例②
複雑な計算式でも、必ず、前から順番に評価できる

 例えば、「` ( 1 + 2 ) * ( 2 + 3 ) ` 」の場合、
  - 逆ポーランド記法では「` ( 1 2 + ) ( 2 3 + ) * ` 」
    -  コンピュータでの実行時には、括弧は無視する。「 ` 1 2 + 2 3 + * ` 」のように記述する
    
* 「`1`」: 1をスタックに積む
  - スタックは `[1 ] `
* 「`2`」: 2をスタックに積む
  - スタックは `[1 2 ] `
* 「`+`」: スタックの最上部2つを取り出して、足し算し、結果をスタックに積む
  -  スタックは `[3 ] `

* 「`2`」: 1をスタックに積む
  - スタックは `[3 2 ] `
* 「`3`」: 2をスタックに積む
  - スタックは `[3 2 3 ] `
* 「`+`」: スタックの最上部2つを取り出して、足し算し、結果をスタックに積む
  -  スタックは `[3 5 ] `

* 「`*`」: スタックの最上部2つを取り出して、掛け算し、結果をスタックに積む
  -  スタックは `[15 ] `

----

## プログラムの作成
  - 以下、Hot Soup Processor 3.6 を使用して実装

### ソースコード
* 今回は、単語をスペースで区切った文字列で与える
  ```hsp
  sdim src, 65536
  src = "2 3 +"
  ```

----
## スタックの実装
* スタック構造を一番簡単に実現出来るのは、「**リスト**」を利用する方法。( Pythonなどではこれでよい )
  - ただし、HSPには**リストは存在しない**


* なので、「配列変数」と、「スタックの最上段を示す矢印」 を併用して実現する
    |    | 0  | 1  | 2  | 3  |
    | -- | -- | -- | -- | -- |
    | スタックのデータ | 1 | 2 | 0 | 0 |
    | 矢印 |  |  | ^ |  |


  ```hsp
  dim stack, 32
  dim stack_index
  stack_index = 0

  // スタックにデータをひとつ入れるとき。スタックの矢印の位置にデータを入れ、矢印をひとつ右にずらす。
  stack(stack_index) = DATA_TO_PUT
  stack_index += 1

  //スタックからデータをひとつ取り出すとき。矢印は最上段のデータの1つ上を指すので、(矢印-1)の位置からデータを取り出し、矢印をひとつ左にずらす。
  DATA_TO_GET = stack(stack_index - 1)
  stack_index -= 1
  ```


----

## 単語分割
* HSPで文字列を区切り文字で分割するには、` split ` 命令を使う

     >split p1,"string",p2...
     >
     >p1=変数 : 元の文字列が代入された変数  
     >"string" : 区切り用文字列  
     >p2=変数 : 分割された要素が代入される変数  
     >
     > ...
     >
     >指定された変数の数よりも、もともとの要素の数が少ない場合は、残りの変数に空の文字列("")が代入されます。  
     >**指定された変数の数よりも、分割された要素が多い場合は、指定された変数の配列に代入されていきます。**
     >
     > ...
     >
     >**実行後に、システム変数statに分割できた数が代入されます。**

     ```hsp
     sdim src, 65536
     sdim elem,256,16 
     dim elem_count
     src = "2 3 +"
 
     split src, " ", elem
     elem_count = stat
 
     // elem(0) = "2", elem(1) = "3", elem(2) = "+"
     // elem_count = 3
     ```

----

## 解釈処理
* 分割された各要素に対して、前から順番に解釈処理をしていく
    ```hsp
    // 省略...

    repeat elem_count
        mes "" + cnt + "番目の要素は" + elem(cnt) + "です。"
    loop

    /*
    「0番目の要素は2です。
      1番目の要素は3です。
      2番目の要素は+です。」
    */
    ```

* 各要素において、
  - 1文字目が数字なら「数」なので、スタックに積む  
  - それ以外なら「演算子」なので、スタックの上2つで演算する

  ```hsp
  // 省略...

  repeat elem_count
      if peek(elem(cnt), 0) >= '0' & peek(elem(cnt), 0) <= '9' {
          // その要素は数、なのでスタックに積む
      }else{
          // その要素は数以外
      }
  loop
  ```
  >val = peek(p1,p2)
  >
  >p1=変数 : 内容を読み出す元の変数名  
  >p2=0～ : バッファのインデックス(Byte単位)

### 数だったとき
* 今見ている要素の中身は、「数」。スタックに代入する
  ```hsp
  // 省略...

  if peek(elem(cnt), 0) >= '0' & peek(elem(cnt), 0) <= '9' {
      // その要素は数、なのでスタックに積む
      stack(stack_index) = int(elem(cnt)) 
      stack_index += 1
  }else{
      
  // 省略...
  ```

### 数以外だったとき
* 現状は「 + - * / 」 のどれか。それぞれ`if`で分岐する。
  ```hsp
  // 省略...
  if peek(elem(cnt), 0) >= '0' & peek(elem(cnt), 0) <= '9' {
      // 省略...

  }else{
      if elem(cnt)="+"{
          // その要素は「+」
      }
      else:if elem(cnt)="-"{
          // その要素は「-」
      }
      else:if elem(cnt)="*"{
          // その要素は「*」
      }
      else:if elem(cnt)="/"{
          // その要素は「/」
      }
  }
  // 省略...
  ```


* それぞれの演算子は、「スタックの上2つの数を取り出して、計算し、スタックに戻す」という手順を踏む
  ```hsp
  //ソースの先頭、変数宣言の部分に以下を追加

  //calc_a, calc_b: 計算対象を取り出して一時格納する変数
  dim calc_a
  dim calc_b
  //calc_result: 計算結果を一時格納する変数
  dim calc_result

  // 省略...

  if elem(cnt)="+"{
      // その要素は「+」
      calc_a = stack(stack_index - 1)
      stack_index -= 1

      calc_b = stack(stack_index - 1)
      stack_index -= 1

      calc_result = calc_a + calc_b

      stack(stack_index) = calc_result
      stack_index += 1

  }

  // 省略...
  ```

----
## 完成品
* 以下はこれらを組み合わせたもの
* デバッグのため、実行終了時にスタックの一番上に格納されている数字 (たいていは計算結果) を表示するように
```hsp
//スタック
dim stack, 32
dim stack_index
stack_index = 0

//ソースコードと、その解釈用
sdim src, 65536
sdim elem,256,16 
dim elem_count
src = "2 3 +"

//スペースで分割する
split src, " ", elem
elem_count = stat

//各要素に対して、ループで解釈する
repeat elem_count
	if peek(elem(cnt), 0) >= '0' & peek(elem(cnt), 0) <= '9' {
		// その要素は数、なのでスタックに積む
		stack(stack_index) = int(elem(cnt)) 
		stack_index += 1
	}else{
		// その要素は数以外
		if elem(cnt)="+"{
			// その要素は「+」
			calc_a = stack(stack_index - 1)
			stack_index -= 1
			
			calc_b = stack(stack_index - 1)
			stack_index -= 1
			
			calc_result = calc_a + calc_b
			
			stack(stack_index) = calc_result
			stack_index += 1
			
		}
		else:if elem(cnt)="-"{
			// その要素は「-」
			calc_a = stack(stack_index - 1)
			stack_index -= 1
			
			calc_b = stack(stack_index - 1)
			stack_index -= 1
			
			calc_result = calc_a - calc_b
			
			stack(stack_index) = calc_result
			stack_index += 1
			
		}
		else:if elem(cnt)="*"{
			// その要素は「*」
			calc_a = stack(stack_index - 1)
			stack_index -= 1
			
			calc_b = stack(stack_index - 1)
			stack_index -= 1
			
			calc_result = calc_a * calc_b
			
			stack(stack_index) = calc_result
			stack_index += 1
			
		}
		else:if elem(cnt)="/"{
			// その要素は「/」
			calc_a = stack(stack_index - 1)
			stack_index -= 1
			
			calc_b = stack(stack_index - 1)
			stack_index -= 1
			
			calc_result = calc_a / calc_b
			
			stack(stack_index) = calc_result
			stack_index += 1
			
		}
	}
loop

mes "実行終了"
mes "いま、スタックの最上段は " + stack(stack_index - 1)
stop
```

* 空行・コメントを除いて55行
* 「スタック + 逆ポーランド記法の例② 」 で挙げた計算式も正しく処理できる
  - 「 `15` 」が出力されます