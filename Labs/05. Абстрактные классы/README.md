# Лабораторная работа 5: Абстрактные классы

## Теоретическая часть

Абстрактные классы используются для задания общего интерфейса для группы классов. Они не предназначены для создания объектов напрямую, а служат основой для `классов-наследников`.

### Определение абстрактного класса

Абстрактный класс в C++ содержит хотя бы одну **чисто виртуальную функцию** (pure virtual function). Такая функция объявляется с `= 0`:
```cpp
class AbstractReader {
public:
    virtual void readAll(const std::string& filename) = 0;  // Чисто виртуальная функция
    virtual ~AbstractReader() = default;                    // Виртуальный деструктор
};
```

Объявление `= 0` делает метод обязательным для переопределения в производных классах. Попытка создать объект `AbstractReader` приведет к ошибке компиляции.

### Реализация наследника

Класс `CsvReader` должен наследоваться от `AbstractReader` и реализовать метод `readAll`:
```cpp
#include <fstream>
#include <iostream>
#include <vector>
#include <string>

class CsvReader : public AbstractReader {
public:
    void readAll(const std::string& filename) override {
      // your code here
    }
};
```

### Чтение JSON с помощью библиотеки `nlohmann::json`

[JSON](https://ru.wikipedia.org/wiki/JSON) - это текстовый формат файлов, который хранит данные в виде пар `ключ - значение`.
Например, записи об `автомобилях` в нём будут выглядеть примерно так:
```json
[
  { "name": "Ford", "year": 2018, "power": 123 },
  { "name": "KIA", "year": 2023, "power": 115 },
]
```

Для работы с JSON-файлами можно использовать библиотеку [nlohmann::json](https://github.com/nlohmann/json), скачав заголовочный файл [json.hpp](https://github.com/nlohmann/json/blob/develop/single_include/nlohmann/json.hpp).

Реализуем `JsonReader`, наследуемый от `AbstractReader`:
```cpp
#include "json.hpp"
#include <fstream>
#include <iostream>

using json = nlohmann::json;

class JsonReader : public AbstractReader {
public:
    void readAll(const std::string& filename) override {
        std::ifstream file(filename);
        if (!file.is_open()) {
            throw std::runtime_error("Ошибка открытия файла: " + filename);
        }
        json j;
        file >> j;
        for (const auto& e : j) {
            std::cout << "Название: " << e["name"].get<std::string>() << ", "
                      << "Год: " << e["year"].get<int>() << ", "
                      << "Мощность: " << e["power"].get<int>() << std::endl;
        }
    }
};
```

## Практическое задание

1) **Добавить абстрактный класс `AbstractReader` в код ЛР4**:
   - Включить в него чисто виртуальную функцию `readAll(const std::string& filename)`.
   - Переделать `CsvReader`, чтобы он наследовался от `AbstractReader`.

2) **Создать `JsonReader`, наследованный от `AbstractReader`**:
   - Использовать библиотеку `nlohmann::json` для чтения JSON-файлов.
   - Реализовать метод `readAll`, который считывает данные из JSON.

3) **Добавить выбор файла через диалоговое окно в `MainWindow`**:
   - Добавить кнопку "Обзор" для выбора файла.
   - Использовать [QFileDialog](https://doc.qt.io/qt-6/qfiledialog.html#details) для открытия файлов (`.csv` или `.json`).
   - В зависимости от расширения файла использовать `CsvReader` или `JsonReader`.

Пример использования `QFileDialog`:
```cpp
QString filename = QFileDialog::getOpenFileName(this, "Открыть файл", "", "Файлы CSV (*.csv);;Файлы JSON (*.json)");
if (!filename.isEmpty()) {
    std::unique_ptr<AbstractReader> reader;
    if (filename.endsWith(".csv")) {
        reader = std::make_unique<CsvReader>();
    } else if (filename.endsWith(".json")) {
        reader = std::make_unique<JsonReader>();
    }
    if (reader) {
        reader->readAll(filename.toStdString());
    }
}
```