# Глава 7. Вложенные функции, лямбды, вариативные функции

Копнем тему функций глубже! Обсудим, для чего нужны вложенные функции, стоит ли злоупотреблять лямбдами и как выглядят вариадики — функции с переменным числом аргументов.

## Вложенные функции
В питоне функции могут быть вложенными (inner, nested), то есть объявленными внутри другой функции:

```python {.example_for_playground}
def outer_f():
    def inner_f():
        print("This is inner function")
    
    inner_f()

outer_f()
inner_f()
```

В данном примере вызов `outer_f()` завершится успешно, а вызов `inner_f()` строчкой ниже приведет к исключению:

```
NameError: name 'inner_f' is not defined
```

Пример показывает, что вложенные функции скрыты от доступа откуда-либо, кроме функции-обертки. Таким образом достигается инкапсуляция вспомогательного кода: он исключается из внешней области видимости.

Итак, вложенные функции нельзя вызывать извне. Но сами они имеют доступ к аргументам и локальным переменным функции-обертки!

Рассмотрим функцию `has_permissions()`, которая принимает путь к директории. Внутри нее есть вложенная функция `get_permissions_str()`, которая принимает имя пользователя и проверяет, имеет ли пользователь доступ к директории.

В этом примере используются строки вида `f"text {val} text"`: так называемые f-строки, о которых вы узнаете [в главе 10.](/courses/python/chapters/python_chapter_0100#block-formatting) Это литералы форматированных строк: перед строкой идет символ `f`, а в саму строку в фигурные скобки можо вставлять любые значения.

```python {.example_for_playground}
def has_permissions(directory):
    def get_permissions_str(user):
        if user == "root":
            return f"Permission to {directory} granted for {user}"

        # Maybe add additional checks here
        return f"Permission to {directory} declined for {user}"

    return get_permissions_str


has_permissions_tmp = has_permissions("/tmp")
print(has_permissions_tmp("sandbox_user"))

has_permissions_logs = has_permissions("/var/logs")
print(has_permissions_logs("root"))
```
```
Permission to /tmp declined for sandbox_user
Permission to /var/logs granted for root
```

В этом примере вложенная функция `get_permissions_str()` работает с параметрами `directory` и `user`. Мы видим, что она имеет доступ к параметрам и переменным внешней функции.

Функция `has_permissions()` возвращает свою внутреннюю функцию. Ее можно присвоить переменной, чтобы затем пользоваться переменной как вызываемым объектом.

Это и происходит в последних 4-х строках кода примера: мы вызвали функцию `has_permissions()` с аргументом `"/tmp"`. Она вернула свою внутреннюю функцию, и мы присвоили ее переменной `has_permissions_tmp`. Теперь эту переменную можно использовать как функцию: в ней хранится вложенная функция `get_permissions_str()`. Поэтому если вызвать `has_permissions_tmp` с аргументом `"sandbox_user"`, то отработает код вложенной функции, в котором запомнено значение `"/tmp"` параметра внешней функции.

Знание о вложенных функциях нам пригодится, когда в [главе про декораторы](/courses/python/chapters/python_chapter_0270/) мы будем обсуждать **замыкания** (closures) — вложенные функции, которые ссылаются на переменные, объявленные в теле внешней функции. Рассмотрим это на небольшом примере:

```python {.example_for_playground}
def f_outer(initial_val):
    items = [initial_val]

    def f_inner(val):
        items.append(val)
        print("Items:", items)

    return f_inner

f = f_outer(1)
f(2)
f(3)
```
```
Items: [1, 2]
Items: [1, 2, 3]
```

Здесь вложенная функция `f_inner()` замыкает список `items`, объявленный в области видимости внешней функции `f_outer()`. Поэтому она может добавлять в этот список новые элементы с помощью метода `append()`. На строке `f = f_outer(1)` создается вызываемый объект `f`, которому присваивается возвращаемое из функции `f_outer()` замыкание `f_inner()`. Оно впоследствии и вызывается для значений 2 и 3.

Так как вложенная функция «видит» аргументы и переменные внешней функции, то будьте внимательны к именованию переменных во вложенной функции. В коде вложенной функции они могут затенить переменные из внешней области видимости. Более подробно про это мы расскажем в главе [«Области видимости».](/courses/python/chapters/python_chapter_0220/)

Напишите функцию `calc_gcd(a, b)`, которая находит наибольший общий делитель (GCD, greatest common divisor) чисел `a` и `b`. Функция должна возвращать два значения: GCD и количество вызовов, которые были выполнены перед возвратом ответа. {.task_text}

Например, `calc_gcd(25, 15)` вернет 5 и 4, а `calc_gcd(8, 3)` вернет 1 и 5.  {.task_text}

Для подсчета количества вызовов заведите вложенную рекурсивную функцию и вызывайте ее. В своем решении реализуйте алгоритм Евклида. Он заключается в следующем: {.task_text}
- GCD равен `a`, если `a` и `b` совпадают.
- GCD равен GCD от `a - b` и `b`, если `a` больше `b`.
- GCD равен GCD от `a` и `b - a`, если `a` меньше `b`.

```python {.task_source #python_chapter_0070_task_0010}
```
Внутри функции `calc_gcd()` заведите вложенную функцию. Она должна принимать на вход значения `a` и `b`, а также третий параметр: счетчик вызовов. Вложенная функция должна инкрементировать его и возвращать в качестве одного из значений в `return`. {.task_hint}
```python {.task_answer}
def calc_gcd(a, b):
    calls = 0
    
    def calc_gcd_inner(a, b, calls):
        calls += 1
        if a == b:
            return a, calls
        if a > b:
            return calc_gcd_inner(a - b, b, calls)
        return calc_gcd_inner(a, b - a, calls)

    return calc_gcd_inner(a, b, calls)
```

Кстати, автор питона Гвидо ван Россум в своей известной [презентации «Introduction to Python»](https://people.csail.mit.edu/rudolph/Teaching/Lectures/guido-intro-1.pdf) предложил вот такой лаконичный вариант поиска наибольшего общего делителя:

```python {.example_for_playground}
def gcd(a, b):
    "greatest common divisor"
    while a != 0:
        a, b = b % a, a # parallel assignment
    return b
```
В примере от Гвидо применяется мощный инструмент питона: parallel assignment.

```python
a = x
b = y
```

Это прием, позволяющий запись выше сократить до одной строки:

```python
a, b = x, y
```

Parallel assignment особенно популярен, когда требуется выполнить swap — поменять значения двух переменных:

```python
x, y = y, x
```

## Лямбда-функции
Лямбда-функции (их также называют анонимными, безымянными) — это компактные функции-однострочники. Они могут иметь много параметров, но содержат только одно вычисляемое и возвращаемое выражение. Для возврата значения из лямбды `return` не используется. 

Если для объявления обычной именованной функции используется ключевое слово `def`, то для объявления лямбды — ключевое слово `lambda`:

```
lambda параметры: выражение
```

Тело лямбда-функции должно состоять из единственного выражения. В нем не может быть никаких операторов (`assert`, `pass`, `raise`, `+=` и прочих). Наличие оператора внутри лямбда-функции приведет к исключению типа `SyntaxError`.

Пример объявления и вызова лямбда-функции, которая повторяет строку `s` `n` раз:

```python {.example_for_playground}
mult_str = lambda s, n:  s * n
print(mult_str("*", 5))
```
```
*****
```

Здесь функциональному объекту `mult_str` присваивается безымянная лямбда-функция. На следующей строке функциональный объект вызывается. Эта лямбда-функция эквивалентна обычной функции, просто позволяет сэкономить немного места:

```python
def mult_str(s, n):
    return s * n
```

Ключевые слова, такие как `return` и `pass`, в лямбде использовать **нельзя.** Список параметров лямбды теоретически может быть и пустым:

```python {.example_for_playground}
x = lambda: 5

print(x())
```
```
5
```

Вместо присваивания лямбды функциональному объекту ее можно сразу вызвать.

```python {.example_for_playground}
(lambda s, n:  print(s * n))("I", 3)
```

Код выглядит странно, но выводит в консоль ожидаемый результат:

```
III
```

Напишите лямбда-функцию, которая принимает два числа `a` и `b`, возвращает их сумму и разность. Присвойте ее функциональному объекту `calc`. {.task_text}

Чтобы лямбда корректно вернула пару значений, явным образом положите их в кортеж: оберните возвращаемые значения в круглые скобки, необходимые для конструирования кортежа. {.task_text}

```python  {.task_source #python_chapter_0070_task_0020}
```
Синтаксис: `<variable> = lambda <arg1, arg2, ...>: (ret_val1, ret_val2, ...)`. {.task_hint}
```python {.task_answer}
calc = lambda a, b: (a + b, a - b)
```

Напишите лямбда-функцию, которая просто выводит в консоль `"Press any key to continue"`. Присвойте ее функциональному объекту `to_next_step`.  {.task_text}

Вызовите этот объект.  {.task_text}

```python  {.task_source #python_chapter_0070_task_0030}
```
Синтаксис: `<variable> = lambda: <function_call()>`.{.task_hint}
```python {.task_answer}
to_next_step = lambda : print("Press any key to continue")
to_next_step()
```

Теперь вы узнаете лямбду, если увидите ее в чужом коде! Но стоит ли лямбдами злоупотреблять? В PEP8 [сказано следующее:](https://peps.python.org/pep-0008/#programming-recommendations) всегда используйте определение функций через `def` вместо того, чтобы присваивать лямбду функциональному объекту. PEP8 настаивает на тотальном избегании лямбд, такие дела.

Например, вот эту лямбду PEP8 считает нужным превратить в полноценную функцию:

```python
f = lambda x: 2 * x
```

```python
def f(x): return 2 * x
```

С чем это связано? Да, лямбды действительно позволяют сэкономить немного места. Но взамен лишают вас понятного стека вызовов, усложняют процесс отладки. Потому что в стек вызовов вместо имени конкретной функции попадает весьма расплывчатое `<lambda>`. Так что выбирая между компактностью и понятностью, отдавайте предпочтение понятности.

Что выведет этот код?  {.task_text}

```python  {.example_for_playground}
def f(items):
    predicate = lambda x: x > 0

    for item in items:
        if predicate(item):
            return item

    return 0

x = f([-10, 0, 10, 20])
print(x)
```

```consoleoutput {.task_source #python_chapter_0070_task_0040}
```
Функция `f` принимает на вход список. Внутри нее создается функциональный объект `predicate`, которому присваивается лямбда. Лямбда возвращает `True`, если переданный ей аргумент больше нуля. Иначе она возвращает `False`. Затем в цикле к каждому элементу списка `items` применяется этот предикат. И если элемент оказывается больше нуля, мы возвращаем его из функции `f`. Если такой элемент не найден, функция возвращает ноль. {.task_hint}
```python {.task_answer}
10
```

## Вариативные функции {#block-variadic}
[Вариативные функции](/courses/python/chapters/python_chapter_0260) (variadic functions) — это функции с переменным числом аргументов. Они реализованы в питоне и способны принимать произвольное количество позиционных и именованных аргументов:

```python
def f(*args, **kwargs):
    ...

f(1, 2, 3, k1="A", k2="B")
```

`args` и `kwargs` — популярные имена для вариативных позиционных и именованных аргументов. Символы звездочек `*` и `**` перед аргументами означают, что внутри переменной содержится некоторая коллекция, которую можно распаковать. 

`*args` — это список всех переданных в функцию позиционных аргументов (в нашем случае это 1, 2, 3). А `**kwargs` (от слова keyworded) хранит все именованные аргументы (`k1`, `k2`). Распаковку и упаковку коллекций мы обсудим в [одной из следующих глав.](/courses/python/chapters/python_chapter_0250/)

`**kwargs` является словарем — коллекцией, хранящей ключи и значения. Более подробно словари будут рассмотрены в одной из следующих глав. А пока кратко приведем варианты обхода словаря, содержащего именованные аргументы `kwargs`:

```python
for key in kwargs:
    print(key)
```

```python
for key, value in kwargs.items():
    print(key, value)
```

Имплементируйте функцию `f()`, которая принимает вариативные позиционные и именованные аргументы. {.task_text}

В теле функции проитерируйтесь по позиционным и именованным аргументам, чтобы вывести их по одному в консоль. Для именованных аргументов нужно выводить их ключи. {.task_text}

Например, при вызове `f(1, 2, k1="a", k2="b")` в консоль должно быть выведено 4 значения, каждое c новой строки: "1", "2", "k1", "k2". {.task_text}

```python  {.task_source #python_chapter_0070_task_0050}
```
Внутри функции `f()` сначала проитерируйтесь циклом `for` по позиционным аргументам для вывода их по одному в консоль, затем — по именованным. {.task_hint}
```python {.task_answer}
def f(*args, **kwargs):
    for a in args:
        print(a)

    for k in kwargs:
        print(k)
```

## Функции как объекты первого класса

В питоне функция является объектом первого класса. Это означает, что функцию можно присвоить переменной, передать в качестве аргумента другой функции или вернуть из функции как результат. Например, можно создать список, каждый элемент которого — функция. Или завести функцию, которая принимает аргумент — функцию-колбэк или функцию-обработчик данных.

Что выведет этот код?  {.task_text}

```python  {.example_for_playground}
def calc(x, y):
    return x * y - y

def f(a, b, action):
    return action(a, b)

g = f
print(g(2, 5, calc))
```

```consoleoutput {.task_source #python_chapter_0070_task_0060}
```
Функция `f` принимает три аргумента, последний из которых — функция. Объекту `g` присваивается функция `f`. Затем этот объект, являющийся функцией, вызывается с аргументами 2, 5 и `calc`. В консоль выводится значение выражения `2 * 5 - 5`, то есть 5.{.task_hint}
```python {.task_answer}
5
```

Понимание того факта, что функции являются объектами первого класса, пригодится в главах про [декораторы](/courses/python/chapters/python_chapter_0270/) и [функции высших порядков.](/courses/python/chapters/python_chapter_0280/)

## Резюмируем
- Функции в питоне могут быть вложенными. Таким образом достигается инкапсуляция кода.
- Вложенные функции имеют доступ к аргументам и переменным своей внешней функции.
- Лямбда-функции — это функции-однострочники вида `lambda параметры: выражение`.
- PEP8 не рекомендует использовать лямбды, потому что они запутывают стек вызовов: вместо явного имени функции в него попадает абстрактная `<lambda>`.
- Функции в питоне — это объекты первого класса.
