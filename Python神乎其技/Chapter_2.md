## 2-1 運用斷言(assert)
- assert是個能輔助除厝的工具
- 若斷言條件為真，代表正常無事；若結果為假，就會引發AssertionError異常，且可帶有錯誤訊息

**斷言的舉例說明**
```python
def apply_discount(product, discount):
    price = int(product['price']) * (1.0 - discount))
    assert 0 <= price <= product['price']
    return price
```
- assert能確保這支函式根據折扣計算出來的價錢決不會低於0元，也絕不會高於產品的原定價

### 何不使用正規的異常寫法

- 為何不使用if和異常呢?
- 斷言的適當用途，是讓開發人員知曉程式執行時發生了無法回復的錯誤，斷言不該拿來指出可預期的錯誤狀況，例如「找不到檔案」，出現這種狀況時，使用者可採取修正動作，或是乾脆再試一遍
- 斷言功能的設計用意應該作為程式的內部自我檢查，用法式宣告你的程式不可能會發生的情況，如果這些條件其中之一無法為真，就是代表程式裡有bug
- 若程式毫無瑕疵，永遠都不會出現錯誤狀況，但若真的發生了，程式就會因為斷言而斷掉
- 斷言的用途是輔助除錯，不是處理執行期間錯誤的機制，運用斷言的目標是讓開發人員更快找出bug的根源；除非你程式有bug，否則又不該引發斷言錯誤

### Python斷言語法

- 開始使用某項Python語言功能之前，最好能先研究該功能實際上是如何實作的
- https://docs.python.org/3.6/reference/simple_stmts.html?highlight=assert#the-assert-statement
```python
assert_stmt ::=  "assert" expression ["," expression]
# 大略等於
if __debug__:
    if not expression1: raise AssertionError(expression2)

# 簡化
assert <condition>, <error message>
```
- 檢查斷言條件之前，還有個全域變數__debug__的檢查，這是個內建的bool，一般情況下會為真，但若啟動最佳化模式則會為False，則不會進行斷言檢查(陷阱1)

### Python斷言的常見陷阱

- 第一個：關於程式裡的安全性風險與臭蟲
- 第二個：語法的小細節

#### 陷阱1：別用斷言來驗證資料
使用Python斷言時，最大的陷阱在於可以整個功能被關閉，只要使用命令列參數 -O 或者 -OO 即可(關閉__debug__)
https://docs.python.org/3.6/library/constants.html?highlight=__debug__#__debug__
這是有意為之的設計，其他程式語言也有類似的開關。
但如此一來會有副作用，若使用斷言語句作為快速且簡單的輸入資料驗證方式，就變得極端危險
例如：用斷言來檢查函式參數是否為「錯誤」或預期之外的值，都會造成安全性漏洞
```python
# 這樣寫非常危險，__debug__為False的狀況下，assert的兩個檢查都會直接被跳過
# 這種「可預期的錯誤狀況」應該改用if來撰寫
def delete_product(prod_id, user):
    assert user.is_admin(), 'Must be admin'
    assert store.has_produce(prod_id), 'Unknown product'
    store.get_product(prod_id).delete()
```

#### 陷阱2：從不失敗的斷言
當你把tuple作為第一個參數傳給assert語句時，該斷言條件一定為真，從不失敗
```python
assert (1==2, '應該要失敗')
```
原因：Python語言裡的非空tuple物件若用於條件判斷，一律代表bool的True

## 2-2
編輯list, dict or set，需要新增或移除裡面的項目時，**每個項目各一行且每一行都用逗點做結尾**
```python
# 不好
names = ['1', '2', '3']
# 好
name = [
    '1',
    '2',
    '3',
]
```
- 把項目分散到多行的好處：在版控中，顯示前後版本差異都是以「行」為基礎，因此可以清楚看出每一次的變動
- 若項目後面沒有逗號，則項目會合併，這是因為string literal concatenation
- https://docs.python.org/3.6/reference/lexical_analysis.html?#string-literal-concatenation
```python
name = [
    '1',
    '2' # 沒逗點
    '3',
]
>>> names
['1', '23']
```

## 2-3 文脈管理器(context manager/資源管理器)與with語法
- 善加運用with的話，可簡化相同的資源管理模式
- 藉由把資源管理功能的邏輯予以抽象化，然後重構提取出來、重複使用
        - 例如把try...finally裡面做資源管理的邏輯部分，用with語句代替

**open()正是絕妙使用案例**
```python
with open('hello.text', 'w') as f:
    f.write('hello, world!')
```
- 開啟檔案時，會建議使用with語句，因為在程式執行流程離開with語句的範圍之後，可確保一定會自動關閉檔案
- 上面的例子在Python直譯器會被轉成類似的樣子：
```python
f = open('hello.text', 'w')
try:
    f.write('hello, world!')
finally:
    f.close()
```
- try finally不可或缺，若光寫成下面的樣子，還無法達到目的：若f.write()發生異常，檔案還是會處於一直被開啟的狀況
```python
f = open('...')
f.write('...')
f.close()
```
**另一個使用案例: threading.Lock**
```python
some_lock = threading.Lock()

# not good
some_lock.acquire()
try:
    # do stuffs
finally:
    some_lock.release()

# better
    with some_lock:
    # do stuffs
```

### 讓自己的物件支援with
**實作context manager介面**
- open()和threading.Lock能與with語句搭配使用，並不奇特，你自己的類別與函式也可以擁有相同的能力，只要實作所謂的context manager即可
- context manager是個簡單的「協定」(或叫做介面)，你的物件若想支援with語句，就必須遵守它的規範，基本上你要做的就是把__enter__和__exit__方法加入物件，就能具備context manager的能力。Python在資源管理的週期裡，會在適當的時刻呼叫那兩支方法
```python
class ManagedFile:
    def __init__(self, name):
        self.name = name

    def __enter__(self):
        self.file = open(self.name, 'w')
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
```
這個ManagedFile類別，遵守context manager協定，於是能夠支援with語句，用法如同先前的open()
```python
with ManagedFile('hello.txt') as f:
    f.write('...')
    f.write('bye')
```
Python在進入with語句，需要取得資源的時候，就會進入__enter__
當執行流程要離開with，Python就會呼叫__exit__，釋放掉資源

**contextlib模組**
- 撰寫類別支援context manager介面，並非支援with語句的唯一方式，標準函式庫裡的contextlib模組，以基本的context manager介面為基礎，在那之上建立了一些抽象層，若你的情況吻合contextlib提供的東西，就可善加利用
- 例如，可使用contextlib.contextmanager decorator來定義以generator為基礎的Factory Function，讓資源自動支援with語句
```python
from contextlib import contextmanager

@contextmanager
def managed_file(name):
    try:
        f = open(name, 'w')
        yield f
    finally:
        f.close()

with managed_file('hello.txt') as f:
    f.write('...')
```
其中，managed_file()是個generator，首先取得資源，然後暫時停止它自己的執行流程，產出(yield)資源，以便交給呼叫者使用。當呼叫者離開with語句，generator繼續執行，任何剩餘的清理步驟都會開始執行，於是會釋放資源、還給系統

### 運用context manager撰寫優質API
- 若善用with，可謂模組和類別定義出方便存取的API

例如想要管理的「資源」是某種報表產生程式的文字縮排層級，要得到這樣的結果
```python
hi!
    hello
        bonjour
hey
```
會撰寫類似下面的程式；
```python
with Indenter() as indent:
    indent.print('hi')
    with indent:
        indent.print('hello')
        with indent:
            indent.print('bonjour')
    indent.print('hey')
```
看起來幾乎像是縮排文字的特定領域語言(Domain-Specific Language, DSL)
同時也請注意，上面程式會多次進出同樣的context manager，藉以改變縮排層級
那麼該如何實作context manager呢?
```python
class Indenter:
    def __init__(self):
        self.level = 0

    def __enter__(self):
        self.level += 1
        return self

    def __exit__(sefl, exc_type, exc_val, exc_tb):
        self.level -= 1

    def print(self, text):
        print('    ' * self.level + text)
```
**重點整理**
- with語句能把try...finally的標準寫法封裝在所謂的context manager裡面，簡化異常的處理
- 就大部分的情況而言，with語句被用來管理系統資源的取得與釋放動作，當程式流程進入with就會取得資源(__enter__)，離開時則自動釋放(__exit__)

## 2-4 底線、雙底線
雙底線字元(double underscour，又稱作dunder)
用在 變數名稱或函式、類別屬性項(包含class attribute或class method) 上面，會有什麼效果?
1. **前單底線 _var** ：命名慣例，代表提醒該變數僅供內部使用的名稱，一般來說，Python直譯器不管此種名稱(除了星號匯入)，只是作為給程式設計師的提示，沒有任何強制性
    - 有著前單底線的屬性，並不會阻擋我們從外部存取
    - 若某一模組/檔案中有一個前單底線開頭的函式 -> def _func()，當外部使用「星號匯入」從該模組匯入所有名稱，Python不會匯入前頭有底線的名稱(除非該模組裡的__all__串列含有該名稱，蓋過上述行為)
2. **後單底線 var_** ：命名慣例，用來避免與Python關鍵字起名稱衝突
    - 如class是Python關鍵字之一，因此若要在程式中使用class這個名稱，會改用class_
3. **前雙底線 __var** ；用在類別屬性項(屬性或方法)上，會啟動名稱修飾(name mangling)，藉以避免與子類別發生名稱衝突，這是Python直譯器強迫執行的行為
4. **前後雙底線 __var__** ：代表由Python語言定義的特別方法，你自己的屬性項應避免採用此種名稱
5. **單底線 _** ：作為暫時性或不重要的變數(不用管它)，另外在Python REPL交談模式下，代表上一道運算式的執行結果
    - 也可用於Upacking Expression
        - `car = ('red', 'auto', 12, 3812.4)`
        - `color, _, _, mileage = car`

詳細說明：
- 應該避免使用「星號匯入」，因為會導致不清楚有什麼名稱會出現在命名空間裡面
- 前雙底線的名稱修飾的說明：
```python
class Test:
    def __init__(self):
    self.foo = 11
    self._bar = 23
    self.__baz = 23

>>> t = Test()
>>> dir(t)
# 沒有__baz這個同名的變數，反而有一個屬性項叫做_Test__baz，這就是名稱修飾，防止變數被子類別覆寫

class ExtendedTest(Test):
    def __init__(self):
        super().__init__()
        self.foo = 'overridden'
        self._bar = 'overridden'
        self.__baz = 'overridden'

>>> t2 = ExtendedTest()
>>> dir(t2)
['_ExtendedTest__baz',
 '_Test__baz',
 ... ]
# t2的__baz屬性叫做_ExtendedTest__baz，但原先的_Test__baz仍然存在。而沒有名稱修飾的其他屬性項會依照寫法而被覆蓋
>>> t2._ExtendedTest__baz
'overridden'
>>> t2._Test__baz
23

# 名稱修飾過的屬性項，可以如此提取...
class ManglingTest:
    def __init__(self):
        self.__mangled = 'hello'

    def get_mangled(self):
        return self.__mangled

    def __method(self):
        return 42

    def call_it(self):
        return self.__method()

>>> ManglingTest().get_mangled()
'hello'
>>> ManglingTest().__mangled
AttributeError
>>> ManglingTest().__method()
AttributeError
>>> ManglingTest().call_it()
42

# 一個特別的例子...
_MangledGlobal__mangled = 23

class MangledGlobal:
    def test(self):
        return self.__mangled

>>> MangledGlobal().test()
23
# 因為名稱修飾的作用，在test()方法裡便能夠以__mangled之名來存取全域變數_MangledGlobal__mangled
```

讚讚讚
