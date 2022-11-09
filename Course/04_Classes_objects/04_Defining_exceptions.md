[Зміст](../Contents.md) \| [Попередній розділ (4.3. Спеціальні методи)](03_Special_methods.md) \| [Наступний розділ (5. Об'єктна модель)](../05_Object_model/00_Overview.md)

# 4.4 Визначення винятків

Винятки, визначені користувачем, визначаються класами.

```python
class NetworkError(Exception):
    pass
```

**Винятки завжди успадковуються від `Exception`.**

Зазвичай це порожні класи. Використовуйте `pass` для тіла.

Ви також можете створити ієрархію винятків.

```python
class AuthenticationError(NetworkError):
     pass

class ProtocolError(NetworkError):
    pass
```

## Вправи

### Вправа 4.11: Визначення спеціального винятку

Для бібліотек часто є хорошою практикою визначати власні винятки.

Це полегшує розрізнення винятків Python, створених у відповідь на поширені помилки програмування, від винятків, навмисно створених бібліотекою, щоб сигналізувати про певну проблему використання.

Змініть функцію `create_formatter()` з останньої вправи так, щоб вона викликала настроюваний виняток `FormatError`, коли користувач вводить неправильну назву формату.

Наприклад:

```python
>>> from tableformat import create_formatter
>>> formatter = create_formatter('xls')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "tableformat.py", line 71, in create_formatter
    raise FormatError('Unknown table format %s' % name)
FormatError: Unknown table format xls
>>>
```

[Зміст](../Contents.md) \| [Попередній розділ (4.3. Спеціальні методи)](03_Special_methods.md) \| [Наступний розділ (5. Об'єктна модель)](../05_Object_model/00_Overview.md)