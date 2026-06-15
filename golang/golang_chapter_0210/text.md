# Глава 21. Работа с файлами

- [ ] Работа с json и xml   
- [ ] работа с файлами  
- [ ] логирование  
- [ ] Задача на кастомный Split до точки с запятой

## Работа с файлами

### Основные пакеты для работы с файлами

* `os` — базовые операции: открытие, создание, удаление, переименование, изменение прав.
* `io` — некоторые инструменты для чтения/записи.
* `bufio` — буферизированный ввод/вывод для производительности и удобства. Пакет позволяет, например, читать по словам.
* `fmt` — форматированный вывод в файл.
* `encoding` — для JSON, CSV, XML и других форматов.

### Чтение файла 

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
    "name": "myapp_db"
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
	// В случае ошибки вернутся false.
	for scanner.Scan() {
		line := scanner.Text()
		fmt.Println(line)
	}
	// В случае конца файла ошибка равно nil.
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

