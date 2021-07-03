# Pythonの概念

Pythonとその構文については基本的なレベルで理解していると思いますが、ここではPython言語をより深く理解するための基礎的な概念について説明します。

**Pythonのすべてのものはオブジェクトです。**

関数、リスト、ディクショナリ、クラス、モジュール、実行中の関数（関数定義のインスタンス）など、すべてを含みます。CPythonでは、各オブジェクトについて構造体変数があることを意味します。

Pythonの現在の実行コンテキストでは、すべての変数はディクショナリに格納されています。文字列とオブジェクトのマッピングですね。現在のコンテキストで、関数とfloat変数が定義されている場合、内部的には以下のように処理されます。

```python
>>> float_number=42.0
>>> def foo_func():
...     pass
...

# 変数名が文字列であり、ディクショナリに格納されていることに注意してください。
>>> locals()
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'float_number': 42.0, 'foo_func': <function foo_func at 0x1055847a0>}
```

## 関数

関数もオブジェクトなので、次のようにして関数に含まれるすべての属性を見ることができます。

```python
>>> def hello(name):
...     print(f"Hello, {name}!")
...
>>> dir(hello)
['__annotations__', '__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__',
'__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__get__', '__getattribute__', '__globals__', '__gt__',
'__hash__', '__init__', '__init_subclass__', '__kwdefaults__', '__le__', '__lt__', '__module__', '__name__',
'__ne__', '__new__', '__qualname__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__',
'__subclasshook__']
```

たくさんありますが、興味深いものをいくつか見てみましょう。

#### __globals__

この属性はその名の通り、グローバル変数の参照を持ちます。この関数のスコープ内にあるすべてのグローバル変数を知ることができます。関数が新たなグローバル変数を参照し始める様子を見てみましょう。

```python
>>> hello.__globals__
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'hello': <function hello at 0x7fe4e82554c0>}

# 新しいグローバル変数の追加
>>> GLOBAL="g_val"
>>> hello.__globals__
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'hello': <function hello at 0x7fe4e82554c0>, 'GLOBAL': 'g_val'}
```

### __code__

これは面白いです！Pythonではすべてがオブジェクトなので、バイトコードも同様です。コンパイルされた PythonバイトコードはPythonのコードオブジェクトです。これは`__code__`属性でアクセスできます。関数には、いくつかの興味深い情報を持ったコードオブジェクトが関連付けられています。

```python
# 関数が定義されているファイルを出力
# インタプリタで実行されるので、stdinとなる
>>> hello.__code__.co_filename
'<stdin>'

# 関数が受け取る引数の数
>>> hello.__code__.co_argcount
1

# ローカル変数名
>>> hello.__code__.co_varnames
('name',)

# 関数コードのコンパイル済みバイトコード
>>> hello.__code__.co_code
b't\x00d\x01|\x00\x9b\x00d\x02\x9d\x03\x83\x01\x01\x00d\x00S\x00'
```

他にもコードの属性はありますが、それは`>> dir(hello.__code__)` でリストアップできます。

## デコレータ

関数に関連して、Pythonにはデコレータと呼ばれる機能があります。「全てはオブジェクトである」ということを念頭に置いて、どのように機能するか見てみましょう。

デコレータのサンプルを紹介します。

```python
>>> def deco(func):
...     def inner():
...         print("before")
...         func()
...         print("after")
...     return inner
...
>>> @deco
... def hello_world():
...     print("hello world")
...
>>>
>>> hello_world()
before
hello world
after
```

ここでは`@deco`構文を使って`hello_world`関数を装飾しています。基本的には以下のようにするのと同じです。

```python
>>> def hello_world():
...     print("hello world")
...
>>> hello_world = deco(hello_world)
```

`deco`関数の中身は複雑に見えるかもしれません。それを解明してみましょう。

1. 関数`hello_world`が作られる
2. `hello_world`関数が `deco` 関数に渡される
3. `deco`は新しい関数を作成する
    1. この新しい関数は`hello_world`関数と呼ばれる
    2. 他にもいくつかのことをする
4. `deco`は新しく作られた関数を返す
5. 元の`hello_world`は上記の関数に置き換えられる

理解を深めるために視覚化してみましょう。

```
       BEFORE                   function_object (ID: 100)

       "hello_world"            +--------------------+
               +                |print("hello_world")|
               |                |                    |
               +--------------> |                    |
                                |                    |
                                +--------------------+


       WHAT DECORATOR DOES

       creates a new function (ID: 101)
       +---------------------------------+
       |input arg: function with id: 100 |
       |                                 |
       |print("before")                  |
       |call function object with id 100 |
       |print("after")                   |
       |                                 |
       +---------------------------^-----+
                                   |
                                   |
       AFTER                       |
                                   |
                                   |
       "hello_world" +-------------+
```

`hello_world`という名前は新しい関数オブジェクトを指していますが、その新しい関数オブジェクトは元の関数の参照（ID）を知っていることに注意してください。

## いくつかの落とし穴

- Pythonでプロトタイプを作るのはとても簡単で、利用可能なライブラリも大量にありますが、コードベースが複雑になると、型エラーがより頻繁に発生するようになり、対処が難しくなります。(Pythonの型アノテーションのような問題の解決策があります。[mypy](http://mypy-lang.org/)をご覧ください。
- Pythonは動的型付け言語なので、すべての型が実行時に決定されることになります。そのため、他の静的型付けされた言語に比べて、Pythonの動作は非常に遅くなっています。
- Python には[GIL](https://www.dabeaz.com/python/UnderstandingGIL.pdf)（global interpreter lock）と呼ばれるものがあり、これが並列計算のために複数のCPIコアを利用する際の制限要因となっています。
- Pythonの奇妙な点のまとめ: https://github.com/satwikkansal/wtfpython
