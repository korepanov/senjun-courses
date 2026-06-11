# Глава 21. Работа с файлами

- [ ] Работа с json и xml   
- [ ] работа с файлами  
- [ ] логирование  

## Работа с файлами

### Основные пакеты для работы с файлами

* `os` — базовые операции: открытие, создание, удаление, переименование, изменение прав.
* `io` — некоторые инструменты для чтения/записи.
* `bufio` — буферизированный ввод/вывод для производительности и удобства. Пакет позволяет, например, читать по словам.
* `fmt` — форматированный вывод в файл.
* `encoding` — для JSON, CSV, XML и других форматов.

### Чтение файла 

Чтобы открыть файл и прочесть часть фиксированного размера, используют `os.Open` и `Read`: 

```go
package main

import (
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
		// обязательно закрыть!
		err = file.Close()
		if err != nil {
			log.Fatal(err)
		}
	}()
	buf := make([]byte, 4096) // 4KB буфер
	n, err := file.Read(buf)
	if err != nil {
		log.Fatal(err)
	}
	if n > 0 {
		fmt.Print(string(buf[:n]))
	}
}
```

Если размер файла меньше 4KB, то мы получим все содержимое файла. Пример результата: 

```
[2026-06-11 10:15:23] Server started
[2026-06-11 10:15:23] User admin logged in
[2026-06-11 10:15:23] Request processed /api/users
[2026-06-11 10:15:23] Server stopped
```

**Всегда закрывайте за собой файл!**   
В противном случае возникнет утечка памяти.

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

Здесь возникает вопрос. Как читать большие файлы? Если файл не поместится в память целиком, то мы не сможем прочесть его через `os.ReadFile`. Следовательно, нужно читать файл по частям. 