# Глава 21. Работа с файлами

- [ ] Работа с json и xml   
- [ ] Базовые операции с файлами  
- [ ] Логирование  
- [x] Задача на кастомный Split до точки с запятой

## Основные пакеты для работы с файлами

* `os` — базовые операции: открытие, создание, удаление, переименование, изменение прав.
* `io` — инструменты для чтения/записи.
* `bufio` — буферизированные чтение/запись для производительности и удобства. Пакет позволяет, например, читать по словам.
* `fmt` — форматированный вывод в файл.
* `encoding` — для работы с JSON, CSV, XML и другими форматами.
* `log` — классический и простой пакет для логирования. 
* `log/slog` — пакет для структурированного логирования.

## Чтение файла 

Когда файл небольшой, его читают целиком: 

```go
package main

import (
	"fmt"
	"log"
	"os"
)

func main() {
	data, err := os.ReadFile("config.json")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(string(data))
}
```

Пример результата: 

```json
{
  "server": {
    "host": "localhost",
    "port": 8080,
    "timeout_seconds": 30
  },
  "database": {
    "host": "localhost",
    "port": 5432,
    "username": "admin",
    "password": "secret123",
    "name": "app_db"
  },
  "logging": {
    "level": "info",
    "file": "app.log",
    "max_size_mb": 100
  },
  "features": {
    "debug_mode": true,
    "enable_cache": false
  },
  "api_keys": ["key123", "key456", "key789"]
}
```

Однако представьте себе ситуацию, что файл большой, а памяти мало. Как быть, когда файл не помещается в память целиком? В этом случае его читают по частям:

```go
package main

import (
	"fmt"
	"io"
	"log"
	"os"
)

func main() {
	// Открываем файл на чтение
	file, err := os.Open(".log")
	if err != nil {
		log.Fatal(err)
	}
	defer func() {
		// Обязательно закрыть!
		err = file.Close()
		if err != nil {
			log.Fatal(err)
		}
	}()
	buf := make([]byte, 128) // 128B буфер
	for {
		// Читаем в буфер
		n, err := file.Read(buf)
		// Получим ошибку io.EOF,
		// когда дойдем до конца файла
		if err == io.EOF {
			break
		}
		if err != nil {
			log.Fatal(err)
		}
		if n > 0 {
			fmt.Print(string(buf[:n]))
		}
	}
}
```

Результат — содержимое файла. Пример результата: 

```
[2026-06-11 10:15:23] Server started
[2026-06-11 10:15:23] User admin logged in
[2026-06-11 10:15:23] Request processed /api/users
[2026-06-11 10:15:23] Server stopped
```

**Всегда закрывайте за собой файл!**   
В противном случае возникнет утечка памяти.

Когда нужно прочесть файл построчно, используют пакет `bufio`:

```go 
package main

import (
	"bufio"
	"fmt"
	"log"
	"os"
)

func main() {
	file, err := os.Open(".log")
	if err != nil {
		log.Fatal(err)
	}
	defer func() {
		err = file.Close()
		if err != nil {
			log.Fatal(err)
		}
	}()
	scanner := bufio.NewScanner(file)
	// Читаем построчно.
	// В случае ошибки вернется false
	// В случае конца файла также 
	// вернется false. 
	// В противном случае — true.
	for scanner.Scan() {
		line := scanner.Text()
		fmt.Println(line)
	}
	// В случае конца файла ошибка равна nil
	if err := scanner.Err(); err != nil {
		log.Fatal(err)
	}
}
```
Пример результата: 

```
[2026-06-11 10:15:23] Server started
[2026-06-11 10:15:23] User admin logged in
[2026-06-11 10:15:23] Request processed /api/users
[2026-06-11 10:15:23] Server stopped
```

По умолчанию `bufio.NewScanner` возвращает значение типа `*bufio.Scanner`, для которого определена функция `split`. Она-то и отвечает за то, что мы читаем файл построчно. В `bufio` существует несколько встроенных функций, которые задают в качестве `split`, когда хотят получить другое поведение. Делают это с помощью метода `Split`:

```go
// Встроенные функции разделения в пакете bufio:
scanner.Split(bufio.ScanLines)   // По строкам — по умолчанию
scanner.Split(bufio.ScanWords)   // По словам
scanner.Split(bufio.ScanRunes)   // По рунам
scanner.Split(bufio.ScanBytes)   // Побайтово
```

Иногда встроенных функций не хватает. В этом случае приходится написать собственную. Сигнатура для функции `split` такая: 

```go
func(data []byte, atEOF bool) (advance int, token []byte, err error)
```

Параметры:
* `data` — непрочитанные данные в буфере.
* `atEOF` — `true`, если это последняя часть данных — достигнут конец файла.

Возвращаемые значения:
* `advance` — на сколько байт продвинуться в буфере. 
* `token` — найденный токен — то, что вернёт `scanner.Text`.
* `err` — ошибка.

Рассмотрим, как считать файл по частям, разделенным символом запятой.

```go
package main

import (
	"bufio"
	"fmt"
	"log"
	"os"
)

func main() {
	file, err := os.Open("dinner.txt")
	if err != nil {
		log.Fatal(err)
	}
	defer func() {
		err = file.Close()
		if err != nil {
			log.Fatal(err)
		}
	}()
	scanner := bufio.NewScanner(file)
	scanner.Split(ScanComma)
	for scanner.Scan() {
		line := scanner.Text()
		fmt.Println(line)
	}
	if err := scanner.Err(); err != nil {
		log.Fatal(err)
	}
}

func ScanComma(data []byte, atEOF bool) (advance int,
	token []byte, err error) {
	if atEOF && len(data) == 0 {
		// Возвращаем последний токен
		return 0, nil, nil
	}
	for i := range len(data) {
		if data[i] == ',' {
			// Нашли запятую, возвращаем токен до неё
			return i + 1, data[:i], nil
		}
	}
	return len(data), data, nil
}
```

`dinner.txt`:
```
apple,banana,orange,grape 
```

Результат: 
```
apple
banana
orange
grape 
```

В файле `input.txt` находится фрагмент кода на языке C++. Необходимо вывести код частями. Части отделяются друг от друга точкой с запятой `;`. Эта точка с запятой не должна находиться в кавычках. Когда точка с запятой внутри кавычек, то она является просто частью строки. Считайте, что текст анализируемой программы всегда правильный. Также в этой программе используются только двойные кавычки, и не встречаются кавычки внутри кавычек. {.task_text}

Воспользуйтесь `bufio.NewScanner`. В качестве функции `split` возьмите `ScanInstruction`, тело которой вам необходимо реализовать.  {.task_text}

```go {.task_source #golang_chapter_0210_task_0010}
package main

import (
	"bufio"
	"fmt"
	"log"
	"os"
)

func main() {
	// ваш код здесь 
}

func ScanInstruction(data []byte, atEOF bool) (advance int,
	token []byte, err error) {
	// ваш код здесь 
}
```

Код для `main` аналогичен примеру для чтения текста по частям, разделенными запятой. Работа с последним токеном в `ScanInstruction` аналогична `ScanComma`. {.task_hint}


```go {.task_answer}
package main

import (
	"bufio"
	"fmt"
	"log"
	"os"
)

func main() {
	file, err := os.Open("input.txt")
	if err != nil {
		log.Fatal(err)
	}
	defer func() {
		err = file.Close()
		if err != nil {
			log.Fatal(err)
		}
	}()
	scanner := bufio.NewScanner(file)
	scanner.Split(ScanInstruction)
	for scanner.Scan() {
		line := scanner.Text()
		fmt.Print(line)
	}
	if err := scanner.Err(); err != nil {
		log.Fatal(err)
	}
}

func ScanInstruction(data []byte, atEOF bool) (advance int,
	token []byte, err error) {
	if atEOF && len(data) == 0 {
		// Возвращаем последний токен
		return 0, nil, nil
	}
	inQuote := false // внутри кавычек
	for i := range len(data) {
		if data[i] == '"' {
			// Переключаем состояние кавычек
			inQuote = !inQuote
		} else if !inQuote && data[i] == ';' {
			// Нашли точку с запятой вне кавычек
			return i + 1, data[:i], nil
		}
	}
	return len(data), data, nil
}
```

## Запись в файл 

Бывает необходимость как писать в существующий файл, так и в новый. Рассмотрим оба варианта.  
Запись в новый файл: 

```go
package main

import (
	"fmt"
	"log"
	"os"
)

func main() {
	file, err := os.Create("pc.txt")
	exitOnError(err)
	defer func() {
		err := file.Close()
		exitOnError(err)
	}()
	// Запись строки
	_, err = file.WriteString("CPU AMD Ryzen 5 7600X\n")
	exitOnError(err)
	// Запись байтов
	data := []byte("Memory 16 ГБ (2x8 ГБ) DDR5-6000 МГц\n")
	_, err = file.Write(data)
	exitOnError(err)
	storage := "SSD 2 ТБ M.2 NVMe"
	os := "Ubuntu 22.04"
	// Форматированная запись
	fmt.Fprintf(file, "Storage: %s, OS: %s\n", storage, os)
}

func exitOnError(err error) {
	if err != nil {
		log.Fatal(err)
	}
}
```

Запись множества мелких записей в файл с буфером и без него:
```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"log"
	"math/rand"
	"os"
	"time"
)

const measureNumber = 10000

func main() {
	file, err := os.OpenFile("sensor.txt", os.O_WRONLY, 0644)
	exitOnError(err)
	defer func() {
		err := file.Close()
		exitOnError(err)
	}()
	withoutBuffer(file)
	// Чистим файл
	err = file.Truncate(0)
	exitOnError(err)
	_, err = file.Seek(0, 0)
	exitOnError(err)
	withBuffer(file)
}

func withoutBuffer(file *os.File) {
	// Вызываем trackTime
	// и откладываем выполнение
	// возвращаемой ею функции
	defer trackTime("withoutBuffer")()
	systemInfo(file)
}

func withBuffer(file *os.File) {
	defer trackTime("withBuffer")()
	writer := bufio.NewWriter(file)
	systemInfo(writer)
	// Пишем буферизованный результат
	// в файл
	err := writer.Flush()
	exitOnError(err)
}

func systemInfo(writer io.Writer) {
	r := rand.New(rand.NewSource(42))
	for i := range measureNumber {
		fmt.Fprintf(writer, "%d\tsensor value:\t%.2f\n",
			i+1, r.Float64())
	}
}

// Функция trackTime замеряет время работы
// функции с именем funcName
func trackTime(funcName string) func() {
	start := time.Now()
	return func() {
		elapsed := time.Since(start)
		fmt.Printf("%s: %v\n", funcName, elapsed)
	}
}

func exitOnError(err error) {
	if err != nil {
		log.Fatal(err)
	}
}
```