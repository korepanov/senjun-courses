# Глава 1
## Введение 
Язык [Go](https://ru.wikipedia.org/wiki/Go) - компилируемый многопоточный язык программирования, разрабатываемый внутри компании Google с 2007 года. Непосредственным проектированием занимались Роберт Гризмер, Роб Пайк и Кен Томпсон. Официально язык представлен в ноябре 2009 года. Поддержка официального компилятора предоставляется для многих операционных систем, в том числе для Windows, Linux и macOS. Язык имеет свою философию, основанную на минималистичности языковых средств и необходимости обработки ошибок по месту их возникновения. 

Первая из особенностей заключается в том, что с точки зрения разработчиков компилятора языка Go многие возможности большинства из языков программирования не используются в реальной разработке. Поэтому хорошо бы иметь язык, который не обладает излишествами. Программы, не усложненные дополнительными языковыми конструкциями, получаются легко читаемыми. 
Вторая особенность состоит в том, что в Go отсутствует обработка ошибок по принципу `try-catch`, чтобы разработчики задумывались каждый раз над смыслом возможной ошибки и обрабатывали ее на месте. 

Приятная возможность языка Go -  это горутины - легковесные потоки, которые может создавать программа. Горутина требует существенно меньше ресурсов, чем процесс на уровне операционной системы.

Цель данного курса - познакомить читателя с языком Go на достаточном уровне, чтобы иметь твердую базу для уверенного использования языка.

## Базовый синтаксис 

Знакомство с языком Go мы начнем, рассматривая базовый синтаксис и выполняя небольшие учебные задания. Цель данного раздела - первое вводное знакомство с языком с тем, чтобы потом рассмотреть многие его частности более подробно. Итак, поехали! Вот, как выглядит hello world на языке Go: 
```golang
    package main // имя текущего пакета 

    import "fmt" // импортируемый пакет fmt 

    func main() { // точка вход в программу 
        fmt.Println("Hello world!") 
    }
```

Первая строка программы - имя текущего пакета. Каждая программа на языке Go состоит из множества файлов, объединяемых в так называемые пакеты. Главный пакет называется main, также, как функция определяющая точку входа в программу. После директивы import идет список пакетов, которые необходимо включить. В данном случае мы включили пакет fmt, чтобы использовать функцию Println из него, которая печатает на экран текст и делает перенос на следующую строку. Текст после символов слешей `//` является однострочным комментарием и игнорируется компилятором. Многострочные комментарии также, как и в языке C, выделяются следующим образом: 
```golang 
/*
    Многострочный
    комментарий
*/
```

### Ключевые особенности

- не используешь - не подключай/не объявляй!
- отсутствие синтаксических излишеств
- кроссплатформенность
- горутины
- отсутствие исключений
- сборщик мусора, обеспечивающий безопасную работу с памятью

В Go существует следующее правило: не используешь - не подключай/не объявляй! Пакет, который подключен, но не используется, является программной ошибкой. Такая программа скомпилирована не будет. Аналогично неиспользуемая переменная также является ошибкой. 

## Встроенные типы данных
- `int8` —  1 байт. Целое число от -128 до 127.
- `int16` — 2 байта. Целое число от -32768 до 32767.
- `int32` — 4 байта. Целое число от -2147483648 до 2147483647.
- `int64` — 8 байт. Целое число от –9 223 372 036 854 775 808 до 9 223 372 036 854 775 807
- `uint8` — 1 байт. Целое число от 0 до 255
- `uint16` — 2 байта. Целое число от 0 до 65535
- `uint32` — 4 байта. Целое число от 0 до 4294967295
- `uint64` — 8 байт. Целое число от 0 до 18 446 744 073 709 551 615
- `byte` — 1 байт. Синоним типа `uint8`, представляет целое число от 0 до 255
- `rune` — 4 байта. Синоним типа `int32`, представляет целое число от -2147483648 до 2147483647
- `int` —  4/8 байт. В зависимости о платформы соответствует либо `int32`, либо `int64`
- `uint` — 4/8 байт. В зависимости о платформы соответствует либо `uint32`, либо `uint64`
- `float32` — 4 байта. Число с плавающей точкой от -3.4e+38 до 3.4e+38
- `float64` — 8 байт. Число с плавающей точкой от -1.7e+308 до 1.7e+308
- `complex64` — 8 байт. Комплексное число, где вещественная и мнимая части представляют числа `float32`
- `complex128` — 16 байт. Комплексное число, где вещественная и мнимая части представляют числа `float64`
- `bool` — 1 байт. Логический тип или тип `bool` может иметь одно из двух значений: `true` (истина) или `false` (ложь)
- `string` - строка.

В таблице выше перечислены встроенные типы языка Go. "Встроенные" означает такие, которые уже есть на уровне самого языка, встроенные в компилятор. Помимо встроенных типов данных, существуют составные типы, которые может определить сам пользователь (прикладной программист) языка Go, а также некоторые другие особенные типы, о которых мы поговорим позже в этом курсе.

Переменные определяются с использованием следующего синтаксиса: 
```
var <имя переменной> <тип переменной>
```

Например: 
```golang    
var i int
```

При объявлении переменной по умолчанию для всех численных переменных задается нулевое значение, для типа bool - false, для строки - пустая строка.

Также допустимо короткое объявление переменной, с неявным типом, например: 
```golang    
var i = 5
```
При таком объявлении компилятор сам определит тип переменной.

Более короткая запись:
```golang 
i := 5
```

Последнее работает только внутри функций.

В Go также используются указатели в стиле C. Например, чтобы объявить переменную типа указатель на `string`, нужно поставить перед словом `string` символ `*`. Чтобы взять адрес переменной, необходимо перед именем переменной поставить символ `&`, а чтобы разыменовать указатель и получить значение необходимо поставить `*`. Арифметика указателей отсутствует во избежание ошибок.

Пример использования указателей: 
```golang 
var s string

s = "Hello!"

var p *string // переменная p типа указатель на string 
p = &s        // взять адрес переменной s и присвоить в переменную p

fmt.Println(*p) //получить значение строки по адресу p и вывести на экран 
```

## Задание 1
Дана следующая программа: 

```golang
package main

import "fmt"

func swap(p *string, p2 *string) {
	// ваш код здесь 
}

func main() {
	var s string
	var s2 string

	s = ", gophers!"
	s2 = "Hello"

	swap(&s, &s2)

	fmt.Println(s + s2)
}

```  
Допишите функцию `swap`, которая принимает на вход два указателя на тип `string`, и меняет местами значения соответствующих  переменных. Программа должна вывести сообщение `Hello, gophers!`

## Решение 

```golang
package main

import "fmt"

func swap(p *string, p2 *string) {
	var s = *p // получаем строку по указателю p и присваиваем ее в s
	*p = *p2   // присваиваем строке по указателю p значение строки по указателю p2
	*p2 = s    // присваиваем строке по указателю p2 значение строки s
}

func main() {
	var s string
	var s2 string

	s = ", gophers!"
	s2 = "Hello"

	swap(&s, &s2)

	fmt.Println(s + s2)
}
```

##  Резюме
1. Язык Go - компилируемый многопоточный язык программирования. 
2. В языке Go существует 18 встроенных типов. Переменные могут объявляться как с явным, так и с неявным типом. 
3. В языке Go используются указатели в стиле C. Арифметика указателей отсутствует.
4. Каждая программа на языке Go состоит из множества файлов, объединяемых в так называемые пакеты. Один из таких пакетов - пакет fmt. Главный пакет называется main, также, как функция определяющая точку входа в программу.