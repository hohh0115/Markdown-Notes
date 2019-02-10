## 3-1 Python函式是頭等物件
- Python的函式是頭等物件(first-class object)，可以指派給物件、存在資料結構中、作為參數傳給其他函式、甚至可作為其他函式的回傳值
```python
def yell(text):
    return text.upper() + '!'

>>> yell('hello')
'HELLO!'
```
#### 函式是物件
- Python程式的所有資料，都是以物件或物件之間的關係來表示，如字串、串列、模組、以及函式，通通都是物件

yell函式是個物件，所以可以指派到另一個變數，如同其他物件
```python
bark = yell

>>> bark('woof')
'WOOF!'
```
這一行程式碼不會呼叫函式，其作用式拿到yell指向的函式物件，建立第二個名稱bark並指向同一個函式物件
- 函式物件與其名稱(def 這個名稱()，如yell)，兩者是分開獨立的關係
```python
>>> del yell

>>> yell('hello?')
NameError

# yell這個名稱被刪除，但其指向的函式物件並沒有被刪除
>>> bark('hey')
'HEY!'

# 不過你要先建立另外一個變數並指向該函式物件，不然你要怎麼叫這個函式物件?
```
- 指向函式的變數與函式本身是兩個各自獨立的東西

順帶一提，Python在建立函式物件時，會把字串識別字附加上去，作為除錯之用，經由屬性項__name__便可存取此一內部識別字
> *自3.3開始，還有個類似的屬性項，叫做__qualname__，提供完整的全名字串，藉以區分函式與類別名稱*

#### 函式可被放在資料結構裡
```python
>>> funcs = [bark, str.lower, str.capitalize]
>>> for f in funcs:
...     print(f, f('hey there'))

<function yell at 0x0000023668E767B8> HEY THERE!
<method 'lower' of 'str' objects> hey there
<method 'capitalize' of 'str' objects> Hey there
```
甚至可以直接呼叫串列裡存放的函式物件，不必先指派給變數
```python
>>> funcs[0]['heyho']
'HEYHO!'
```
#### 函式可作為參數傳給其他函式
```python
def greet(func):
    greeting = func('Hi, I am a Python program')
    print(greeting)

>>> greet(bark)
'HI, I AM A PYTHON PROGRAM!'

def whisper(text):
    return text.lower() + '...'

>>> greet(whisper)
'hi, i am a python program...'
```
可以把函式當作參數傳入其他函式，非常強大，讓我們能把程式裡面的行為予以抽象化，包裝之後到處傳來傳去。

上面的例子中，greet函式維持不變，但你能傳入代表不同歡迎行為的函式物件，影響其輸出結果。

能夠接受其他函式作為參數的函式，也被稱為高階函式(high-order function)，對於函數式程式設計(functional programming?)風格來說不可或缺

Python內建的map是高階函式的經典範例
#### 函式可形成巢狀結構
- 把函式定義在別的函式之內，稱之為nested function或inner function
- 在父函式內部的內部函式，也可以由父函式回傳給外界
#### 函式可捉住本地端狀態
- 函式不僅能回傳其他函式，這些內部函式還能「捕捉並帶著父函式的部分狀態」
``` python
def get_speak_func(text, volume):
    def whisper():
        return text.lower() + '...'
    def yell():
        return text.upper() + '!'
    if volume > 0.5:
        return yell
    else:
        return whisper

>>> get_speak_func('Hello, World', 0.7)()
'HELLO, WORLD!'
```
內部函式whisper與yell，被父函式傳出後它們不再有text參數，但仍然能夠存取定義在父函式裡的text，似乎能捕捉並且記住該參數的值。

能做到這點的函式，稱之為**語彙閉包(lexical closure)**，或者簡稱為閉包。

閉包會記住外圍語彙範圍裡的變數值，即便程式流程不再處於該範圍之內。

從實務面來說，意思是函式不僅能回傳行為，還能預先設定那些行為的組態
```python
def make_adder(n):
    def add(x):
        return x + n
    return add

>>> plus_3 = make_adder(3)
>>> plus_5 = make_adder(5)
>>> plus_3(4)
7
>>> plus_5(4)
9
```

#### 物件可擁有像函式一樣的行為
- 函式通通都是物件，物件則不是函式，但能讓物件成為可被呼叫者(callable)，如同函式那般
- 如果物件是callable，代表你可以使用圓括號的函式呼叫語法，甚至還可傳入參數，以上種種都是由__call__方法提供

```python
class Adder:
    def __init__(self, n):
        self.n = n

    def __call__(self, x):
        return self.n + x

>>> plus_3 = Adder(3)
>>> plus_3(4)
7
```

- Python提供內建函式callable()用來檢查某物件是否可被呼叫
> The built-in callable() method checks if the argument is either of the two:
>
> - An instance of a class with a \_\_call__ method.
>
> - Is of a type that has a which indicates callability such as in functions, methods etc. or has a non null tp_call (c struct) member

## 3-2 lambda是含有單一運算式的函式
```python
# 原本這樣寫
def add(x, y):
    return x + y

>>> add(5, 3)
8

>>> add = lambda x, y: x + y
>>> add(5, 3)
8

# function expression
>>> (lambda x, y: x + y)(5, 3)
8
```
- lambda以及def的主要區別在於：
  - 使用lambda的話，不必在呼叫之前把函式物件綁定到某名稱；只需寫出我想要計算的運算式、作為lambda的一部分，然後如同一般函式，立即呼叫lambda算式執行

- lambda跟def還有個語法上的差異：
  - lambda限制只能含有單一條運算式，意思是說，lambda函式裡不能有語句或附加提式，即便是return也不行。
  - lambda函式執行時，運算式執行結果會自動成為回傳值，所以一定會有個「隱性」的回傳語句。

#### lambda的使用時機
- 我最常使用lambda的情況，是在排序可迭代者時，撰寫短小精悍的鍵函式，提供另一種key參數
```python
>>> tuples = [(1, 'd'), (2, 'b'), (4, 'a'), (3, 'c')]
# 根據每個元組的第二個值來排序串列
>>> sorted(tuples, key=lambda x: x[1])
[(4, 'a'), (2, 'b'), (3, 'c'), (1, 'd')]
```
- lambda如同一般巢狀函式，lambda也可以是語彙閉包(lexical closure)
```python
def make_adder(n):
    return lambda x: x + n

>>> plus_3 = make_adder(3)
>>> plus_5 = make_adder(5)
>>> plus_3(4)
7
>>> plus_5(4)
9
```

#### 或許你不該用...
- 應該盡量少用lambda，即使要用也必須要極度小心
- 應該追求的是乾淨整潔、易讀易懂的程式

## 3-3 裝飾器的威力
- 從本質上來說，裝飾器(decorator)能夠讓你延伸修改可被呼叫者callable的行為，諸如函式、方法、類別，而且不必永久地修改可被呼叫者本身
- 任何可加諸在既有類別或函式的行為、稱得上泛用型的功能，都是適合運用裝飾器的場合，包括下列：
  - 訊息紀錄
  - 強制管控存取權與身分認證
  - 控制與時序函式
  - 比例限定
  - 快取，以及其他
- 學習裝飾器時，關於「頭等函式」最重要的部分是：
  - 函式是物件，可被指派給變數...等
  - 函式可被定義在另一支函式裡頭，並且，子函式能捕捉父函式的區域狀態(語彙閉包)

#### Python裝飾器基礎知識
