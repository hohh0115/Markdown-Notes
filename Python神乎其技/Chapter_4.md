## 4-1 物件比較： == 與 is
- 相等(equal)與相同(identical)的差別
- ==是以「相等」的概念來比較：如果貓是Python物件，當使用==進行比較，結果將會是「兩隻貓是相等的」
- is則是以「相同」的概念來比較：若以is比較貓物件，結果會是「兩隻不同的貓」

結論：
- 若兩個變數指向的物件是相等(equal)的(擁有一樣的內容)，==運算式的結果是True
- 若兩個變數指向同一個(相同identical)的物件，is的運算式結果會是True(看id()的結果)

## 4-2 字串轉換（每個類別都需要__repr__）
- 轉換成字串：print(your_object)
    - 預設會得到的字串，僅含有類別名稱與物件實體的id(在CPython中，這會是物件的記憶體位址)
- 檢視物件：直接 my_object
- 增加兩支前後雙底線方法 __str__ 與 __repr__ 到類別裡，這才是控制Python如何在各種情況把物件轉成字串的正確做法
    - __str__ 是Python dunder之一，每當你透過各種可行方式想要把物件轉成字串時，該方法就會被呼叫
    - __repr__   

```python
class Car:
    def __init__(self, color, mileage):
        self.color = color
        self.mileage = mileage
        
    def __str__(self):
        return f'a {self.color} car'

>>> my_car = Car('red', 37821)

>>> print(my_car) 
>>> str(my_car)
>>> '{}'.format(my_car)
'a red car'
```
