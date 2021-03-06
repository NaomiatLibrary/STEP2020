# Q5:pythonのリストの挙動を確認する
## プログラム
```
def foo(b):
    print("bのid:\t%#08x"%id(b))
    print("bの中身のid:\t",[id(i) for i in b])
    b.append(2)
    print("bのid(append(2)後):\t%#08x"%id(b))
    print("bの中身のid:\t",[id(i) for i in b])
    b = b + [3]
    print("bのid(+[3]後):\t%#08x"%id(b))
    print("bの中身のid:\t",[id(i) for i in b])
    b.append(4)
    print("bのid(append(4)後):\t%#08x"%id(b))
    print("bの中身のid:\t",[id(i) for i in b])
    print('b:', b)
a = [1]
print("aのid(定義後):\t%#08x"%id(a))
print("aの中身のid:\t",[id(i) for i in a])
foo(a)
print("aのid(関数foo後):\t%#08x"%id(a))
print("aの中身のid:\t",[id(i) for i in a])
print('a:', a)
```
## 結果
```
aのid(定義後):   0x104251f88
aの中身のid:     [4362902704]
bのid:  　　　　　0x104251f88
bの中身のid:     [4362902704]
bのid(append(2)後):     0x104251f88
bの中身のid:     [4362902704, 4362902736]
bのid(+[3]後):  0x104485c48
bの中身のid:     [4362902704, 4362902736, 4362902768]
bのid(append(4)後):     0x104485c48
bの中身のid:     [4362902704, 4362902736, 4362902768, 4362902800]
b: [1, 2, 3, 4]
aのid(関数foo後):       0x104251f88
aの中身のid:     [4362902704, 4362902736]
a: [1, 2]
```

aのidは`0x104251f88`のまま変化していない。

bのidは、関数の引数としてaを渡された直後はaと同じ`0x104251f88`であり、appendで要素を増やしても変化しないが、
`b=b+[3]`で要素を増やしたときは`0x104485c48`に変化している。

中の要素のidについて、先頭の二要素のidを取り出すと、aとb共通で`[4362902704, 4362902736]`で変化せず、
`b=b+[3]`によりbのidが先述のように変化した後もこのidは変わっていない。

## 考察
pythonの変数には、オブジェクト（数字や文字列などの値のことである）そのものではなく、それを参照するリファレンス（id）が格納されるようになっている。（私はidはオブジェクトの同一性を示すものでありアドレスそのものではないと思っていたのですが、とりあえずCpythonでは実際にメモリ上のアドレスを返すようです:[ドキュメント](https://docs.python.org/ja/3/reference/datamodel.html)。Cに慣れているので今後の文章中ではアドレスと同一視して話を展開することがあります。）


今回のQ5プログラムの動作は、pythonのリストのデータ構造が関係している。
リストである変数aのidが指すアドレスから連続した領域に、それぞれの要素を示すidが記録されている。さらにその記録されたidの指し示す領域に、リストの中のミュータブルな値である1,2,3,4などが記録されている。

pythonは関数の引数として指定したとき、その変数に代入されたidをコピーするため、今回はaを渡したときにaのidである`0x104251f88`がbのidとなった。
listのappend(x)メソッドは、リストのidが指し示すアドレスから連続した領域の末尾に新しいidを追加し、そのidの示すメモリ領域にxを記録する働きを持つ。aとbのidが同じ状態でb.append(x)を行ってしまうと、aとbが共同で参照している「あるidのリスト」の末尾にxのidを追加することになるので、aから参照したときもxが追加されたという影響が及んでしまう。（a=\[1,2\]となる理由）
一方で、+演算子は演算の結果を新たなオブジェクトとして生成するため、当然そのidも新しく生成される。よって`b=b+[3]`を行うと新しいリストのオブジェクトのidが生成されてbに代入されるため、結果で見たようにaとbのidが異なってくる。この状態でbにappendを行なっても、影響があるのはbのidが指し示すリストオブジェクトのみであり、aは異なるidにより異なるリストを指し示しているためにaには影響しない。

先頭の二要素1と2のidは、aとbそれ自体のidが別になった後でも、aとb共通で`[4362902704, 4362902736]`になっているが、もし`b[0]=100`などとしてもaの要素は変更されない。これは、イミュータブルな値（数字とか文字列とか）を代入したときも、変数にはその値自身でなくそれを指し示すidが新しく生成され格納されているから、イミュータブルな値を代入するたびに、その変数が持つidは変わるためである。よって、bの一番目の要素のidが変わるだけで、aの一番目の要素のidやそれが指し示す値には何の影響もない。

Q5と同じように引数としてイミュータブルな値を渡すと、ミュータブルなリストの場合とは違って関数の外に影響を及ぼさない。例えば引数としてイミュータブルな値a=10をbに渡して、関数のなかでb=20としても、bのidが新しく作られた20のオブジェクトの識別子となるだけなので、元のaのidと中身(10)は変化しない。(なお文字列はリストと似ている気もするが、pythonではイミュータブル扱いのようだ)