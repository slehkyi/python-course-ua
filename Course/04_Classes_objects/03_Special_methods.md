[Зміст](../Contents.md) \| [Попередній розділ (4.2. Успадкування)](02_Inheritance.md) \| [Наступний розділ (4.4. Визначення нових винятків)](04_Defining_exceptions.md)

# 4.3 Спеціальні методи

Різні частини поведінки Python можна налаштувати за допомогою спеціальних або так званих «магічних» методів. Цей розділ представляє цю фішку. Крім того, обговорюються динамічний доступ до атрибутів і зв'язні методи.

### Вступ

Класи можуть визначати спеціальні методи. Вони мають особливе значення для інтерпретатора Python. Перед ними завжди стоїть `__`. Наприклад `__init__`.

```python
class Stock(object):
    def __init__(self):
        ...
    def __repr__(self):
        ...
```

Існують десятки спеціальних методів, але ми розглянемо лише кілька конкретних прикладів.

### Спеціальні методи для перетворення рядків

Об’єкти мають два представлення рядків.

```python
>>> from datetime import date
>>> d = date(2012, 12, 21)
>>> print(d)
2012-12-21
>>> d
datetime.date(2012, 12, 21)
>>>
```
Функція `str()` використовується для створення гарного виводу для друку:
```python
>>> str(d)
'2012-12-21'
>>>
```
Функція `repr()` використовується для створення більш детального представлення для програмістів.
```python
>>> repr(d)
'datetime.date(2012, 12, 21)'
>>>
```
Ці функції, `str()` і `repr()`, використовують пару спеціальних методів у класі для створення рядка для відображення.
```python
class Date(object):
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    # Використовується з `str()`
    def __str__(self):
        return f'{self.year}-{self.month}-{self.day}'

    # Використовується з `repr()`
    def __repr__(self):
        return f'Date({self.year},{self.month},{self.day})'
```

*Примітка. Конвенція для `__repr__()` полягає в тому, щоб повертати рядок, який, передаючи в `eval()`, відтворить базовий об’єкт. Якщо це неможливо, замість цього використовується якесь легко читане представлення.*

### Спеціальні методи з математики

Математичні оператори включають виклики наступних методів.
```python
a + b       a.__add__(b)
a - b       a.__sub__(b)
a * b       a.__mul__(b)
a / b       a.__truediv__(b)
a // b      a.__floordiv__(b)
a % b       a.__mod__(b)
a << b      a.__lshift__(b)
a >> b      a.__rshift__(b)
a & b       a.__and__(b)
a | b       a.__or__(b)
a ^ b       a.__xor__(b)
a ** b      a.__pow__(b)
-a          a.__neg__()
~a          a.__invert__()
abs(a)      a.__abs__()
```

### Спеціальні методи доступу до елементів

Це методи для реалізації контейнерів.
```python
len(x)      x.__len__()
x[a]        x.__getitem__(a)
x[a] = v    x.__setitem__(a,v)
del x[a]    x.__delitem__(a)
```

Ви можете використовувати їх на своїх уроках.
```python
class Sequence:
    def __len__(self):
        ...
    def __getitem__(self,a):
        ...
    def __setitem__(self,a,v):
        ...
    def __delitem__(self,a):
        ...
```

### Виклик методу

Виклик методу — це двоетапний процес.

1. Пошук: Оператор `.`
2. Виклик методу: оператор `()`

```python
>>> s = Stock('GOOG',100,490.10)
>>> c = s.cost  # Lookup
>>> c
<bound method Stock.cost of <Stock object at 0x590d0>>
>>> c()         # Method call
49010.0
>>>
```

### Зв'язні методи

Метод, який ще не був викликаний оператором виклику функції `()`, відомий як *прив’язаний метод*. Він діє на тому екземплярі, де він виник.

```python
>>> s = Stock('GOOG', 100, 490.10)
>>> s
<Stock object at 0x590d0>
>>> c = s.cost
>>> c
<bound method Stock.cost of <Stock object at 0x590d0>>
>>> c()
49010.0
>>>
```

Зв'язні методи часто є джерелом необережних неочевидних помилок. Наприклад:

```python
>>> s = Stock('GOOG', 100, 490.10)
>>> print('Cost : %0.2f' % s.cost)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: float argument required
>>>
```

Або підступна поведінка, яку важко усунути.

```python
f = open(filename, 'w')
...
f.close     # Ой, взагалі нічого не сталося. `f` все ще відкрито.
```

В обох цих випадках причиною помилки є те, що ви забули додати кінцеві дужки. Наприклад, `s.cost()` або `f.close()`.

### Доступ до атрибутів

Існує альтернативний спосіб доступу до атрибутів, маніпулювання ними та керування ними.

```python
getattr(obj, 'name')          # Те ж що і obj.name
setattr(obj, 'name', value)   # Те ж що і obj.name = value
delattr(obj, 'name')          # Те ж що і del obj.name
hasattr(obj, 'name')          # Перевіряє наявність атрибута
```

EПриклад:

```python
if hasattr(obj, 'x'):
    x = getattr(obj, 'x'):
else:
    x = None
```

*Примітка: `getattr()` також має корисне значення за умовчанням *arg*.

```python
x = getattr(obj, 'x', None)
```

## Вправи

### Вправа 4.9: Кращий вихід для друку об’єктів

Змініть об’єкт `Stock`, який ви визначили в `stock.py`, щоб метод `__repr__()` створював корисніші результати. Наприклад:

```python
>>> goog = Stock('GOOG', 100, 490.1)
>>> goog
Stock('GOOG', 100, 490.1)
>>>
```

Подивіться, що відбувається, коли ви читаєте портфель акцій і переглядаєте отриманий список після внесення цих змін. Наприклад:

```
>>> import report
>>> portfolio = report.read_portfolio('Data/portfolio.csv')
>>> portfolio
... see what the output is ...
>>>
```

### Вправа 4.10: Приклад використання getattr()

`getattr()` — це альтернативний механізм для читання атрибутів. Його можна використовувати для написання надзвичайно гнучкого коду. Для початку спробуйте цей приклад:

```python
>>> import stock
>>> s = stock.Stock('GOOG', 100, 490.1)
>>> columns = ['name', 'shares']
>>> for colname in columns:
        print(colname, '=', getattr(s, colname))

name = GOOG
shares = 100
>>>
```

Уважно придивіться до того, що вихідні дані повністю визначаються іменами атрибутів, указаними у змінній `columns`.

У файлі `tableformat.py` візьміть цю ідею та розгорніть її в узагальнену функцію `print_table()`, яка друкує таблицю, що показує визначені користувачем атрибути списку довільних об’єктів. Як і у випадку з попередньою функцією `print_report()`, `print_table()` також має приймати екземпляр `TableFormatter` для керування вихідним форматом. Ось як це має працювати:

```python
>>> import report
>>> portfolio = report.read_portfolio('Data/portfolio.csv')
>>> from tableformat import create_formatter, print_table
>>> formatter = create_formatter('txt')
>>> print_table(portfolio, ['name','shares'], formatter)
      name     shares
---------- ----------
        AA        100
       IBM         50
       CAT        150
      MSFT        200
        GE         95
      MSFT         50
       IBM        100

>>> print_table(portfolio, ['name','shares','price'], formatter)
      name     shares      price
---------- ---------- ----------
        AA        100       32.2
       IBM         50       91.1
       CAT        150      83.44
      MSFT        200      51.23
        GE         95      40.37
      MSFT         50       65.1
       IBM        100      70.44
>>>
```

[Зміст](../Contents.md) \| [Попередній розділ (4.2. Успадкування)](02_Inheritance.md) \| [Наступний розділ (4.4. Визначення нових винятків)](04_Defining_exceptions.md)