[Зміст](../Contents.md) \| [Попередній розділ (3.5. Основний модуль)](05_Main_module.md) \| [Наступний розділ (4. Класи)](../04_Classes_objects/00_Overview.md)

# 3.6 Обговорення дизайну

У цьому розділі ми переглядаємо рішення, прийняте раніше.

### Імена файлів проти ітерованих об'єктів

Порівняйте ці дві програми, які повертають однаковий результат.

```python
# Provide a filename
def read_data(filename):
    records = []
    with open(filename) as f:
        for line in f:
            ...
            records.append(r)
    return records

d = read_data('file.csv')
```

```python
# Provide lines
def read_data(lines):
    records = []
    for line in lines:
        ...
        records.append(r)
    return records

with open('file.csv') as f:
    d = read_data(f)
```

* Якій із цих функцій ви віддаєте перевагу? Чому?
* Яка з цих функцій є більш гнучкою?

### Глибока ідея: "Качина типізація"

[Качина типізація](https://uk.wikipedia.org/wiki/%D0%9A%D0%B0%D1%87%D0%B8%D0%BD%D0%B0_%D1%82%D0%B8%D0%BF%D1%96%D0%B7%D0%B0%D1%86%D1%96%D1%8F) — це концепція комп’ютерного програмування, яка дає змогу визначити, чи можна використовувати об’єкт для певної мети. Це програма [качиного тесту](https://en.wikipedia.org/wiki/Duck_test). Лінк на англійську Вікі, бо там воно краще розписано.

> Якщо воно виглядає як качка, плаває як качка і крякає як качка, то, ймовірно, це качка.

У другій версії `read_data()`, функція очікує будь-який ітерований об'єкт. Не лише рядки файлу.

```python
def read_data(lines):
    records = []
    for line in lines:
        ...
        records.append(r)
    return records
```

Це означає, що ми можемо використовувати його з іншими *рядками*.

```python
# A CSV file
lines = open('data.csv')
data = read_data(lines)

# A zipped file
lines = gzip.open('data.csv.gz','rt')
data = read_data(lines)

# The Standard Input
lines = sys.stdin
data = read_data(lines)

# A list of strings
lines = ['ACME,50,91.1','IBM,75,123.45', ... ]
data = read_data(lines)
```

Ця конструкція має значну гнучкість.

*Питання: чи варто нам приймати цю гнучкість чи боротися з нею?*

### Найкращі практики дизайну бібліотеки

Бібліотеки коду часто краще обслуговуються, використовуючи гнучкість.
Не обмежуйте свої можливості. Велика гнучкість забезпечує велику потужність.

## Вправи

### Вправа 3.17: Від імен файлів до файлоподібних об’єктів

Тепер ви створили файл `fileparse.py`, який містить функцію `parse_csv()`. Функція працювала так:

```python
>>> import fileparse
>>> portfolio = fileparse.parse_csv('Data/portfolio.csv', types=[str,int,float])
>>>
```

Наразі функція очікує передачі імені файлу. Однак ви можете зробити код більш гнучким. Змініть функцію так, щоб вона працювала з будь-яким файлоподібним/ітерованим об’єктом. Наприклад:

```
>>> import fileparse
>>> import gzip
>>> with gzip.open('Data/portfolio.csv.gz', 'rt') as file:
...      port = fileparse.parse_csv(file, types=[str,int,float])
...
>>> lines = ['name,shares,price', 'AA,100,34.23', 'IBM,50,91.1', 'HPE,75,45.1']
>>> port = fileparse.parse_csv(lines, types=[str,int,float])
>>>
```

Що станеться в цьому новому коді, якщо передати назву файлу, як і раніше?
```
>>> port = fileparse.parse_csv('Data/portfolio.csv', types=[str,int,float])
>>> port
... look at output (it should be crazy) ...
>>>
```

Так, потрібно бути обережним. Чи можете ви додати безпекову перевірку, щоб уникнути цього?

### Вправа 3.18: Виправлення існуючих функцій

Виправте функції `read_portfolio()` і `read_prices()` у файлі `report.py`, щоб вони працювали зі зміненою версією `parse_csv()`. Це має включати лише незначні зміни. Після цього ваші програми `report.py` і `pcost.py` мають працювати так само, як і завжди.

[Зміст](../Contents.md) \| [Попередній розділ (3.5. Основний модуль)](05_Main_module.md) \| [Наступний розділ (4. Класи)](../04_Classes_objects/00_Overview.md)
