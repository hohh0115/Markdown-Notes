## 自訂的可呼叫型態 p.151
- Python函式是物件，但任何Pyhon物件都可以做成擁有函式般的行為，唯一要做的就是寫__call__實例方法
- 如果想要建立類似函式的物件，而且在它裡面有一些必須每次的呼叫之間保存的內部狀態，你可以在類別中實作__call__。其中一個例子就是修飾器。
  - 使用closure來建立具有內部狀態的函式，是一種完全不同的函式建立方式。
```python
import random
class BingoCase
    def __init__(self):
        pass

    def pick(self):
        pass

    def __call__(self):
        return self.pick()
```

## 函式自我檢查 p.151
- 如同一般自訂類別的實例，函式會用__dict__屬性來儲存它的屬性
- 一般情況下，將任意的屬性指派給函式並不是常見的作法，但Django是使用這種做法的其中一個框架。
- 接下來我們來看函式專用，且不會出現在一般Python自訂物件中的屬性。
```python
class C: pass
obj = C()
def func(): pass
sorted(set(dir(func)) - set(dir(obj)))
```

## 從定位參數到只限使用關鍵字的參數 p.153
pass

## 取得參數的資訊 p.154
- 在函式物件中，__defaults__會保存一個tuple，裡面有定位與關鍵字引數的預設值。關鍵字引數的預設值會被存在__kwdefaults__，但是，引數的名稱會被放在__code__裡面
- 但是有很多缺陷(p.156)，所以，有更好的inspect模組可以使用，使用範例: p.157
- inspect.Signature物件有一個bind方法會取用任何數量的引數並將它們指派給簽章的參數，套用一般規則來將實際的引數匹配給形式上的參數。框架可以使用它，在實際呼叫函式前，先驗證引數。使用範例： p.158

## 函式註釋 type hint
```python
def clip(text:str, max_len:'int > 0'=80) -> str:
    pass
```
- 註釋使用：
  - 用 : 加上註釋表示式
  - 如果引數有預設值，註釋要放在引數名稱與 = 符號中間
  - 註釋回傳值，要在 ) 與函式宣告式最後面的 : 之間加入 -> 與另外一個表示式，表示式可以使用任何型態。
  - 註釋最常見型態是類別，如str或int，或字串，例如 'int > 0'
- 註釋不會處理任何工作。它只會被存在函式的__annotations__屬性裡面
- inspect.signature()知道如何擷取註釋，p.160

## 函式編程套件 functional programming package
- 拜operator與functools等套件，你也可善用函式編程風格
- operator模組提供相當於數十種算術運算子的函式，如階層

```python
from functools import reduce
from operator import mul
# with operator.mul
reduce(mul, range(1, n+1))
# normal
reduce(lambda a, b: a*b, range(1, n+1))
```

- operator還可以取代一些功能簡單的lambda，itemgetter與attrgetter建構了自訂函式來做這些事:
  - 從序列中挑取項目: itemgetter
  - 從物件中讀取屬性的函式: attrgetter
