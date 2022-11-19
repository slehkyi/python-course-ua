[Зміст](../Contents.md) \| [Попередній розділ (5.2. Класи та інкапсуляція)](../05_Object_model/02_Classes_encapsulation.md) \| [Наступний розділ (6.2 Налаштування ітерації за допомогою генераторів)](02_Customizing_iteration.md)

# 6.1 Протокол ітерації

У цьому розділі розглядається основний процес ітерації.

### Ітерація всюди

Багато різних об'єктів підтримують ітерацію.

```python
a = 'hello'
for c in a: # Ітерація символів в a
    ...

b = { 'name': 'Dave', 'password':'foo'}
for k in b: # Ітерація ключів у словнику
    ...

c = [1,2,3,4]
for i in c: # Ітерація елементів у списку/таплі
    ...

f = open('foo.txt')
for x in f: # Ітерація рядків у файлі
    ...
```

### Ітерація: протокол

Розглянемо оператор `for`.

```python
for x in obj:
    # інші команди
```

Що відбувається під капотом?

```python
_iter = obj.__iter__()        # Отримати об'єкт ітератора
while True:
    try:
        x = _iter.__next__()  # Отримати наступний предмет
        # інші команди ...
    except StopIteration:     # Більше немає предметів
        break
```

Усі об’єкти, які працюють із циклом for, реалізують цей низькорівневий протокол ітерації.

Приклад: ручна ітерація по списку.

```python
>>> x = [1,2,3]
>>> it = x.__iter__()
>>> it
<listiterator object at 0x590b0>
>>> it.__next__()
1
>>> it.__next__()
2
>>> it.__next__()
3
>>> it.__next__()
Traceback (most recent call last):
File "<stdin>", line 1, in ? StopIteration
>>>
```

### Підтримка ітерації

Знання про ітерацію корисні, якщо ви хочете додати її до своїх власних об’єктів. Наприклад, зробити власний контейнер.

```python
class Portfolio:
    def __init__(self):
        self.holdings = []

    def __iter__(self):
        return self.holdings.__iter__()
    ...

port = Portfolio()
for s in port:
    ...
```

## Вправи

### Вправа 6.1: Ітерація

Створіть такий список:

```python
a = [1,9,4,25,16]
```

Перегляньте цей список вручну. Викличте `__iter__()`, щоб отримати ітератор, і викликайте метод `__next__()`, щоб отримати послідовні елементи.

```python
>>> i = a.__iter__()
>>> i
<listiterator object at 0x64c10>
>>> i.__next__()
1
>>> i.__next__()
9
>>> i.__next__()
4
>>> i.__next__()
25
>>> i.__next__()
16
>>> i.__next__()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
>>>
```

Вбудована функція `next()` — це ярлик для виклику методу `__next__()` ітератора. Спробуйте використати його у файлі:

```python
>>> f = open('Data/portfolio.csv')
>>> f.__iter__()    # Note: This returns the file itself
<_io.TextIOWrapper name='Data/portfolio.csv' mode='r' encoding='UTF-8'>
>>> next(f)
'name,shares,price\n'
>>> next(f)
'"AA",100,32.20\n'
>>> next(f)
'"IBM",50,91.10\n'
>>>
```

Продовжуйте викликати `next(f)`, доки не дійдете до кінця файлу. Дивіться, що відбувається.

### Вправа 6.2: Підтримка ітерації

Іноді ви можете зробити так, щоб один із ваших власних об’єктів підтримував ітерацію, особливо якщо ваш об’єкт обертається навколо існуючого списку або іншого ітерованого. У новому файлі `portfolio.py` визначте такий клас:

```python
# portfolio.py

class Portfolio:

    def __init__(self, holdings):
        self._holdings = holdings

    @property
    def total_cost(self):
        return sum([s.cost for s in self._holdings])

    def tabulate_shares(self):
        from collections import Counter
        total_shares = Counter()
        for s in self._holdings:
            total_shares[s.name] += s.shares
        return total_shares
```

Цей клас має бути шаром навколо списку, але з деякими додатковими методами, такими як властивість `total_cost`. Змініть функцію `read_portfolio()` у `report.py` так, щоб вона створювала екземпляр `Portfolio` так:

```python
# report.py
...

import fileparse
from stock import Stock
from portfolio import Portfolio

def read_portfolio(filename):
    '''
    Read a stock portfolio file into a list of dictionaries with keys
    name, shares, and price.
    '''
    with open(filename) as file:
        portdicts = fileparse.parse_csv(file,
                                        select=['name','shares','price'],
                                        types=[str,int,float])

    portfolio = [ Stock(d['name'], d['shares'], d['price']) for d in portdicts ]
    return Portfolio(portfolio)
...
```

Спробуйте запустити програму `report.py`. Ви побачите, що вона вражаюче падає через те, що екземпляри `Portfolio` не піддаються ітерації.

```python
>>> import report
>>> report.portfolio_report('Data/portfolio.csv', 'Data/prices.csv')
... crashes ...
```

Виправте це, змінивши клас `Portfolio` для підтримки ітерації:

```python
class Portfolio:

    def __init__(self, holdings):
        self._holdings = holdings

    def __iter__(self):
        return self._holdings.__iter__()

    @property
    def total_cost(self):
        return sum([s.shares*s.price for s in self._holdings])

    def tabulate_shares(self):
        from collections import Counter
        total_shares = Counter()
        for s in self._holdings:
            total_shares[s.name] += s.shares
        return total_shares
```

Після внесення цієї зміни ваша програма `report.py` має працювати знову. Поки ви це робите, виправте свою програму `pcost.py`, щоб використовувати новий об’єкт `Portfolio`. Ось так:

```python
# pcost.py

import report

def portfolio_cost(filename):
    '''
    Computes the total cost (shares*price) of a portfolio file
    '''
    portfolio = report.read_portfolio(filename)
    return portfolio.total_cost
...
```

Перевірте, щоб переконатися, що він працює:

```python
>>> import pcost
>>> pcost.portfolio_cost('Data/portfolio.csv')
44671.15
>>>
```

### Вправа 6.3: Створення більш правильного контейнера

Створюючи клас-контейнер, ви часто хочете робити більше, ніж просто ітерацію. Змініть клас `Portfolio`, щоб він мав інші спеціальні методи, подібні до цього:

```python
class Portfolio:
    def __init__(self, holdings):
        self._holdings = holdings

    def __iter__(self):
        return self._holdings.__iter__()

    def __len__(self):
        return len(self._holdings)

    def __getitem__(self, index):
        return self._holdings[index]

    def __contains__(self, name):
        return any([s.name == name for s in self._holdings])

    @property
    def total_cost(self):
        return sum([s.shares*s.price for s in self._holdings])

    def tabulate_shares(self):
        from collections import Counter
        total_shares = Counter()
        for s in self._holdings:
            total_shares[s.name] += s.shares
        return total_shares
```

Тепер спробуйте кілька експериментів, використовуючи цей новий клас:

```
>>> import report
>>> portfolio = report.read_portfolio('Data/portfolio.csv')
>>> len(portfolio)
7
>>> portfolio[0]
Stock('AA', 100, 32.2)
>>> portfolio[1]
Stock('IBM', 50, 91.1)
>>> portfolio[0:3]
[Stock('AA', 100, 32.2), Stock('IBM', 50, 91.1), Stock('CAT', 150, 83.44)]
>>> 'IBM' in portfolio
True
>>> 'AAPL' in portfolio
False
>>>
```

Одне важливе зауваження щодо цього: зазвичай код вважається «Python», якщо він розмовляє звичайним словником того, як зазвичай працюють інші частини Python. Для об’єктів-контейнерів підтримка ітерації, індексування, утримування та інших типів операторів є важливою частиною цього.

[Зміст](../Contents.md) \| [Попередній розділ (5.2. Класи та інкапсуляція)](../05_Object_model/02_Classes_encapsulation.md) \| [Наступний розділ (6.2 Налаштування ітерації за допомогою генераторів)](02_Customizing_iteration.md)