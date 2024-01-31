---
title: '[Go][備忘録] sliceをmake()する方法による挙動の違い'
tags:
  - Go
private: false
updated_at: '2023-02-11T22:52:22+09:00'
id: 9076ef2c7824fd656f13
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

Go初心者です。
学習していて、今後つまずきそうと思ったのでメモしておきます。

# make()方法による挙動の違い

`make()`を使用してsliceを作成する場合、第2引数に`length`、第3引数に`capacity`を渡すようにしないと、`int`の初期値である`0`で埋められたsliceが作成されてしまう。

`make()`にsizeだけ渡す場合

```go
package main

import "fmt"

func main() {
	var s []int
	s = make([]int, 5)

	for i := 0; i < 5; i++ {
		s = append(s, i)
		fmt.Println(s)
	}
	fmt.Println(s)
    // -> [0 0 0 0 0 0 1 2 3 4]
}
```

`make()`にsizeとcapcityどちらも渡す場合

```diff_go
package main

import "fmt"

func main() {
	var s []int
-	s = make([]int, 5)
+   s = make([]int, 0, 5)

	for i := 0; i < 5; i++ {
		s = append(s, i)
		fmt.Println(s)
	}
	fmt.Println(s)
    // -> [0 1 2 3 4]
}
```

まあ、`make()`関数の説明にハッキリ明言されてるし大丈夫か...?

>Slice: The size specifies the length. The capacity of the slice is
equal to its length. A second integer argument may be provided to
specify a different capacity; it must be no smaller than the
length.
