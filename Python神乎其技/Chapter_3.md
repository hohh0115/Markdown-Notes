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
- 裝飾器的作用是「裝飾」或「包裹」另一支函式，讓你能在被包裹的函式執行之前與之後，執行你裝飾器的程式碼
- 用概略的方式形容，裝飾器是個callable，以callable作為輸入、回傳值也是callable
```python
# 最簡單的裝飾器
def null_decorator(func):
    return func

def greet():
    return 'Hello!'

greet = null_decorator(greet)

>>> greet()
'Hello!'

# 也可以使用@語法這個語法糖來裝飾函式
@null_decorator
def greet():
    return 'Hello!'

>>> greet()
'Hello!'
```
請注意，使用@語法的話，會在定義期間就立即裝飾函式，如此一來，若不走不正常管道，很難再存取得到被裝飾的原函式，因此有人會選擇手動裝飾函式(上面的第一種寫法)，便可保留彈性，還能呼叫裝飾前的原始函式

#### 裝飾器能修改行為
```python
def uppercase(func):
    def wrapper():
        original_result = func()
        modified_result = origin_result.upper()
        return modified_result

    return wrapper

@uppercase
def greet():
    return 'Hello!'

>>> greet()
'HELLO!'
```
這支uppercase裝飾器在裏頭定義一支新函式(閉包)，使用它來包裹輸入函式，藉以在呼叫期間修改其行為。
裝飾器透過一層包裹閉包，修改可被呼叫者的行為，因此你不必永久地變更原本的可被呼叫者，它的行為只有在裝飾之後才會變更
經由上述機制，你便可以把能夠重複使用的建構區塊添加到既有的函式與類別。


#### 套用多個裝飾器到函式身上
- 這麼做會累積所有裝飾器的效果
```python
def strong(func):
    def wrapper():
        return '<strong>' + func() + '</strong>'
    return wrapper

def emphasis(func):
    def wrapper():
        return '<em>' + func() + '</em>'
    return wrapper

@strong
@emphasis
def greet():
    return 'Hello!'

>>> greet()
'<strong><em>Hello!</em></strong>'
# decorated_greet = strong(emphasis(greet))
```
清楚地秀出裝飾器的套用順序：__「從下到上」__。
首先函式被@emphasis加以裝飾，其結果(已被裝飾過一回)函式，又再由@strong進行裝飾。
我習慣稱呼為裝飾器堆疊(decorator stacking)，建立堆疊時會從底部開始，然後在頂部加入新東西，逐漸往上成長。
裝飾器的層級若太深，最終將會影響效能表現

#### 裝飾有接受參數的函式
以上的範例都只裝飾簡單的、沒有參數的greet函式，因此你看見的裝飾器不需要處理參數、轉傳給輸入函式。
若你把上述裝飾器套用到接受參數的函式，將無法正確運作，那麼，我們如何裝飾會接受參數個數不定的函式呢?
此時正該由Python的 *args 與 **kwargs 功能接手
```python
# proxy裝飾器
def proxy(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```
這個裝飾器有兩個值得注意的點:
- 在wrapper閉包定義裡使用*與**運算子，收集所有位置型與關鍵字參數，儲存在變數裡(args與kwargs)
- 然後，wrapper閉包利用*與**「拆箱運算子」，把收集後的參數、轉傳給原始的輸入函式

底下的trace裝飾器，會在執行期間紀錄韓式的參數與結果，若有需要的話，可成為絕佳的除厝輔助工具
```python
def trace(func):
    def wrapper(*args, **kwargs):
        print(
            f'TRACE: calling {func.__name__}() '
            f'with {args}, {kwargs}'
        )
        original_result = func(*args, **kwargs)
        print(
            f'TRACE: {func.__name__}() '
            f'returned {original_result!r}'
        )
        retrun original_result
    return wrapper

@trace
def say(name, line):
    return f'{name}: {line}'
```

#### 如何撰寫「可除錯」的裝飾器
舉例而言，被裝飾的原始函式的名稱、文件字串(docstring)、參數清單，都會被包裹閉包藏起來
```python
def greet():
    """Return a friendly greeting."""
    return 'Hello!'
decorated_greet = upppercase(greet)

>>> greet.__name__
'greet'
>>> greet.__doc__
'Return a friendly greeting.'

>>> decorated_greet.__name__
'wrapper'
>>> decorated_greet.__doc__
'None'
```
有個解決辦法：Python標準程式庫的functools.wraps裝飾器
在裝飾器上使用functools.wraps裝飾器，便可把遺失的後設資料，從未裝飾的函式、複製到裝飾器閉包
```python
import functools

def uppercase(func):
    @functools.wraps(func)
    def wrapper():
        return func().upper()
    return wrapper
```

## 3-4 *args與**kwargs
```python
# *args與**kwargs允許函式接受選用性參數
def foo(required, *args, **kwargs):
    print(required)
    if args:
        print(args)
    if kwargs:
        print(kwargs)
```
- 上述函式要求至少要有一個叫做required的參數，但也能接受額外的位置型與關鍵字參數
- 呼叫該函式時若傳入更多的參數，args將會收集額外的位置型參數，成為tuple，因為其參數名稱以*開頭
- kwargs將會收集額外的關鍵字參數，成為dict，因為其參數名稱以**開頭
- 如果沒有傳入額外的參數，那麼args與kwargs可以是空的
- 把參數叫做args與kwargs只是命名慣例，實際上僅要求*與**而已

> 總結來說，*與**可放在兩種地方：
> - 用在formal parameter，定義函式參數的時候
>   - 收集額外的位置型參數以及額外的關鍵字參數
>   - 轉傳選用性或關鍵字參數
> - 用在actual argument，傳給函式的時候
>   - 函式參數拆箱

#### 轉傳選用性或關鍵字參數
做法是呼叫另一個函式時，使用拆開參數的運算子*與**，便可轉傳參數，同時，也讓你有機會先修改參數然後再傳入其他函式
```python
def foo(x, *args, **kwargs):
    kwargs['name'] = 'Alice'
    new_args = args + ('extra', )
    bar(x, *new_args, **kwargs)
```
1. 撰寫子類別與包裹函式時，上述技巧將派上用場，

例如，你可以擴充父類別的行為，但又不必在子類別裡，完全照抄父類別建構式的全部參數
```python
class Car:
    def __init__(self, color, mileage):
        self.color = color
        self.mileage = mileage

class AlwaysBlueCar(Car):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        sefl.color = 'blue'

>>> AlwaysBlueCar('green', 48392).color
'blue'
```
AlwaysBlueCar建構式會把所有參數通通傳入父類別，然後改寫一個內部屬性項，意思是說，未來若變更父類別建構式，AlwaysBlueCar仍可照預期正常運作的機率，將會很高。
<br>
__但有個缺點__，AlwaysBlueCar建構式的參數清單變得沒有什麼用，我們不知道它預期收到那些參數，除非往上層查看父類別的定義。

__一般來說，我們在自己的類別繼承架構中，並不會使用此技巧__，
較可能採用的情境，是當你想要修改或覆蓋某些外來的類別行為，而且你又沒辦法控制該類別的時候。<br>
但這樣做總是有危險，務必小心評估。

2. 另一個可能採用此技巧的情境是在撰寫包裹函式時，

例如裝飾器通常也想要接受任意種類與個數的參數時，然後轉傳給被包裹的函式。
```python
def trace(f):
    @functools.wraps(f)
    def decorated_function(*args, **kwargs):
        print(f, args, kwargs)
        result = f(*args, **kwargs)
        print(result)
    return decorated_function

@trace
def greet(greeting, name):
    return '{}, {}!'.format(greeting, name)

>>> greet('Hello', 'Bob')
<function greet at xxx> ('Hello', 'Bob') {}
'Hello, Bob!'
```

## 3-5 函式參數拆箱
以*與**運算子來「拆箱」函式的序列與字典參數。
```python
def print_vector(x, y, z):
    print('<%s, %s, %s>' % (x, y, z))

>>> print_vector(0, 1, 0)
<0, 1, 0>
>>> tuple_vec = (0, 1, 0)
>>> list_vec = [0, 1, 0]
>>> print_vector(
        tuple_vec[0],
        tuple_vec[1],
        tuple_vec[2]
    )
<0, 1, 0>
```
一般函式呼叫，竟然要分開傳入個個參數，令人覺人繁瑣！

如果能夠「爆開」向量物件、分解成3個元素，然後直接傳入print_vector函式，不是更棒嗎？

那就已*運算子進行函式參數拆箱。
```python
>>> print_vector(*tuple_vec)
>>> print_vector(*list_vec)
```
__呼叫函式時，在可迭代者的前頭放上*，將會「解開」可迭代者物件，把其中的元素分別做成個別的位置型參數，傳入函式。__
<br>
這項技巧可用於任何種類的可迭代者，包括產生器運算式(generator expression)，對產生器套用*的話，會從產生器拿出所有的元素，傳入函式。
```python
>>> genexpr = (x * x for x in range(3))
>>> print_vector(genexpr)
```
除了以*運算子來拆解序列型物件，如tuple, list和產生器，拆解成位置型參數；__另外還有**運算子，可拆開字典，拆解成關鍵字參數。__
```python
>>> dict_vec = {'x': 0, 'y': 1, 'z': 0}
>>> print_vector(dict_vec)
```
__若你錯以一個星號*來拆箱字典，那麼結果會是以亂數順序把鍵傳入函式。__
```python
>>> print_vector(*dict_vec)
<y, x, z>
```

#### 3-6
任何函式的結尾，Python都會在暗地裡偷偷加上return None語句，因此，若你沒有明確指定回傳值，那麼預設回傳None。
意思是說，你可以把return None語句置換成return，甚至根本不寫，得到的結果仍相同。
