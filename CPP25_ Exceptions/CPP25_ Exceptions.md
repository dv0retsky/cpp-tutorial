# Тема №25. Обработка исключений 🧸

## 📖 Введение

**Обработка исключений** — это механизм, позволяющий программе реагировать на аномальные ситуации (ошибки) во время выполнения. В реальном мире ничто не работает идеально: файл может не открыться, память — не выделиться, пользователь — ввести некорректные данные. Исключения предоставляют элегантный способ отделить код обработки ошибок от основного кода программы.

<div align="center">
  <img alt="Project Demo" src="./mygif/gif-25.1.gif" />
</div>

Представьте, что вы пишете программу для банковского терминала. В процессе работы может произойти множество ошибок: недостаточно средств на счете, неверный PIN-код, проблемы с сетью, отказ сервера. Без механизма исключений вам пришлось бы после каждой операции проверять код возврата и передавать его вверх по стеку вызовов. Код превратился бы в бесконечные if-проверки, а логика программы стала бы нечитаемой.

Исключения в C++ позволяют:

- **Сигнализировать об ошибке** — сгенерировать (бросить) исключение в месте возникновения проблемы
- **Распространять ошибку** — исключение автоматически поднимается по стеку вызовов, пока не будет перехвачено
- **Обрабатывать ошибку** — перехватить исключение в подходящем месте и принять меры
- **Гарантировать освобождение ресурсов** — при раскрутке стека вызываются деструкторы всех локальных объектов

На этом занятии мы подробно изучим механизм исключений: как их генерировать, как перехватывать, какие существуют стандартные исключения, как создавать собственные классы исключений и как писать безопасный с точки зрения исключений код.

## ⚙️ Базовый механизм исключений

В C++ исключения реализованы с помощью трех ключевых слов: `throw`, `try` и `catch`.

### 💻 ПРИМЕР 1 (Простейшее исключение)

```cpp
#include <iostream>
#include <string>

double divide(double a, double b) {
    if (b == 0) {
        throw "Деление на ноль!";  // Генерируем исключение (C-строку)
    }
    return a / b;
}

int main() {
    double x, y;
    
    std::cout << "Введите два числа: ";
    std::cin >> x >> y;
    
    try {
        double result = divide(x, y);
        std::cout << "Результат: " << result << std::endl;
    }
    catch (const char* message) {  // Перехватываем исключение типа const char*
        std::cout << "Ошибка: " << message << std::endl;
    }
    
    std::cout << "Программа продолжает работу" << std::endl;
    return 0;
}
```

### 💻 ПРИМЕР 2 (Исключения разных типов)

```cpp
#include <iostream>
#include <string>
#include <stdexcept>  // для стандартных исключений

void processValue(int value) {
    if (value < 0) {
        throw std::invalid_argument("Значение не может быть отрицательным");
    }
    else if (value == 0) {
        throw 0;  // бросаем целое число
    }
    else if (value > 100) {
        throw std::out_of_range("Значение превышает допустимый диапазон");
    }
    else if (value == 42) {
        throw "Смысл жизни найден!";  // бросаем C-строку
    }
    
    std::cout << "Обработка значения " << value << std::endl;
}

int main() {
    for (int i = -10; i <= 110; i += 10) {
        std::cout << "\nПопытка обработать значение " << i << ":" << std::endl;
        
        try {
            processValue(i);
        }
        catch (const std::invalid_argument& e) {
            std::cout << "Перехвачено invalid_argument: " << e.what() << std::endl;
        }
        catch (const std::out_of_range& e) {
            std::cout << "Перехвачено out_of_range: " << e.what() << std::endl;
        }
        catch (int code) {
            std::cout << "Перехвачен целочисленный код: " << code << std::endl;
        }
        catch (const char* message) {
            std::cout << "Перехвачено сообщение: " << message << std::endl;
        }
        catch (...) {  // catch-all обработчик
            std::cout << "Перехвачено неизвестное исключение" << std::endl;
        }
    }
    
    return 0;
}
```

## 🧱 Стандартные исключения

C++ предоставляет иерархию стандартных классов исключений в заголовочном файле `<stdexcept>`.

### 💻 ПРИМЕР 3 (Иерархия стандартных исключений)

```cpp
#include <iostream>
#include <stdexcept>
#include <vector>
#include <memory>

void demonstrateExceptions() {
    // std::logic_error и его наследники
    try {
        throw std::logic_error("Логическая ошибка");
    }
    catch (const std::logic_error& e) {
        std::cout << "logic_error: " << e.what() << std::endl;
    }
    
    try {
        throw std::invalid_argument("Неверный аргумент");
    }
    catch (const std::invalid_argument& e) {
        std::cout << "invalid_argument: " << e.what() << std::endl;
    }
    
    try {
        throw std::domain_error("Ошибка домена");
    }
    catch (const std::domain_error& e) {
        std::cout << "domain_error: " << e.what() << std::endl;
    }
    
    try {
        throw std::length_error("Ошибка длины");
    }
    catch (const std::length_error& e) {
        std::cout << "length_error: " << e.what() << std::endl;
    }
    
    try {
        throw std::out_of_range("Выход за границы");
    }
    catch (const std::out_of_range& e) {
        std::cout << "out_of_range: " << e.what() << std::endl;
    }
    
    // std::runtime_error и его наследники
    try {
        throw std::runtime_error("Ошибка времени выполнения");
    }
    catch (const std::runtime_error& e) {
        std::cout << "runtime_error: " << e.what() << std::endl;
    }
    
    try {
        throw std::range_error("Ошибка диапазона");
    }
    catch (const std::range_error& e) {
        std::cout << "range_error: " << e.what() << std::endl;
    }
    
    try {
        throw std::overflow_error("Переполнение");
    }
    catch (const std::overflow_error& e) {
        std::cout << "overflow_error: " << e.what() << std::endl;
    }
    
    try {
        throw std::underflow_error("Антипереполнение");
    }
    catch (const std::underflow_error& e) {
        std::cout << "underflow_error: " << e.what() << std::endl;
    }
}

int main() {
    std::cout << "=== Демонстрация стандартных исключений ===" << std::endl;
    demonstrateExceptions();
    
    std::cout << "\n=== Примеры из реальной жизни ===" << std::endl;
    
    // at() генерирует out_of_range
    try {
        std::vector<int> vec = {1, 2, 3};
        std::cout << vec.at(10) << std::endl;  // выход за границы
    }
    catch (const std::out_of_range& e) {
        std::cout << "vector::at(): " << e.what() << std::endl;
    }
    
    // bad_alloc при нехватке памяти
    try {
        std::unique_ptr<int[]> ptr(new int[100000000000]);  // слишком большой массив
    }
    catch (const std::bad_alloc& e) {
        std::cout << "bad_alloc: " << e.what() << std::endl;
    }
    
    return 0;
}
```

## 🛠️ Собственные классы исключений

Часто удобно создавать собственные классы исключений, наследующие от `std::exception`, чтобы передавать дополнительную информацию об ошибке.

### 💻 ПРИМЕР 4 (Пользовательские исключения)

```cpp
#include <iostream>
#include <exception>
#include <string>
#include <sstream>

// Базовое исключение для банковских операций
class BankException : public std::exception {
private:
    std::string message;
    int errorCode;
    std::string accountNumber;
    
public:
    BankException(const std::string& msg, int code, const std::string& acc)
        : message(msg), errorCode(code), accountNumber(acc) {}
    
    const char* what() const noexcept override {
        return message.c_str();
    }
    
    int getErrorCode() const { return errorCode; }
    std::string getAccountNumber() const { return accountNumber; }
};

// Конкретные типы исключений
class InsufficientFundsException : public BankException {
public:
    InsufficientFundsException(const std::string& acc, double balance, double amount)
        : BankException(
            "Недостаточно средств на счете", 
            1001, 
            acc
          ) {
        // Сохраняем дополнительную информацию
        this->balance = balance;
        this->amount = amount;
    }
    
    double getBalance() const { return balance; }
    double getAmount() const { return amount; }
    
private:
    double balance;
    double amount;
};

class InvalidAccountException : public BankException {
public:
    InvalidAccountException(const std::string& acc)
        : BankException(
            "Неверный номер счета", 
            1002, 
            acc
          ) {}
};

class AccountBlockedException : public BankException {
public:
    AccountBlockedException(const std::string& acc, const std::string& reason)
        : BankException(
            "Счет заблокирован: " + reason, 
            1003, 
            acc
          ) {}
};

// Банковский счет
class BankAccount {
private:
    std::string accountNumber;
    double balance;
    bool blocked;
    std::string blockReason;
    
public:
    BankAccount(const std::string& acc, double initialBalance)
        : accountNumber(acc), balance(initialBalance), blocked(false) {}
    
    void withdraw(double amount) {
        if (blocked) {
            throw AccountBlockedException(accountNumber, blockReason);
        }
        
        if (amount <= 0) {
            throw std::invalid_argument("Сумма снятия должна быть положительной");
        }
        
        if (amount > balance) {
            throw InsufficientFundsException(accountNumber, balance, amount);
        }
        
        balance -= amount;
        std::cout << "Снято " << amount << " со счета " << accountNumber 
                  << ". Остаток: " << balance << std::endl;
    }
    
    void deposit(double amount) {
        if (blocked) {
            throw AccountBlockedException(accountNumber, blockReason);
        }
        
        if (amount <= 0) {
            throw std::invalid_argument("Сумма пополнения должна быть положительной");
        }
        
        balance += amount;
        std::cout << "Зачислено " << amount << " на счет " << accountNumber 
                  << ". Остаток: " << balance << std::endl;
    }
    
    void block(const std::string& reason) {
        blocked = true;
        blockReason = reason;
    }
    
    void unblock() {
        blocked = false;
        blockReason.clear();
    }
    
    double getBalance() const { return balance; }
    std::string getAccountNumber() const { return accountNumber; }
};

int main() {
    try {
        BankAccount account("40817810099910000123", 5000);
        
        account.withdraw(1000);   // OK
        account.withdraw(6000);   // Должно выбросить исключение
    }
    catch (const InsufficientFundsException& e) {
        std::cout << "Ошибка: " << e.what() << std::endl;
        std::cout << "Счет: " << e.getAccountNumber() << std::endl;
        std::cout << "Баланс: " << e.getBalance() << ", запрошено: " << e.getAmount() << std::endl;
        std::cout << "Код ошибки: " << e.getErrorCode() << std::endl;
    }
    catch (const BankException& e) {
        std::cout << "Банковская ошибка: " << e.what() << std::endl;
        std::cout << "Код ошибки: " << e.getErrorCode() << std::endl;
    }
    catch (const std::exception& e) {
        std::cout << "Стандартная ошибка: " << e.what() << std::endl;
    }
    
    std::cout << "\n=== Демонстрация блокировки счета ===" << std::endl;
    
    try {
        BankAccount account2("40817810099910000124", 10000);
        account2.block("Подозрительная активность");
        account2.withdraw(100);  // Должно выбросить исключение о блокировке
    }
    catch (const AccountBlockedException& e) {
        std::cout << "Ошибка: " << e.what() << std::endl;
        std::cout << "Счет: " << e.getAccountNumber() << std::endl;
    }
    
    return 0;
}
```

## 🧹 Раскрутка стека и RAII

Одно из важнейших свойств исключений — автоматическая раскрутка стека. При возникновении исключения вызываются деструкторы всех локальных объектов, созданных на пути от `try`-блока до места генерации исключения.

### 💻 ПРИМЕР 5 (Раскрутка стека)

```cpp
#include <iostream>
#include <memory>

class Resource {
private:
    std::string name;
    
public:
    Resource(const std::string& n) : name(n) {
        std::cout << "Ресурс " << name << " захвачен" << std::endl;
    }
    
    ~Resource() {
        std::cout << "Ресурс " << name << " освобожден" << std::endl;
    }
    
    void use() const {
        std::cout << "Использую ресурс " << name << std::endl;
        if (name == "problem") {
            throw std::runtime_error("Ошибка при использовании ресурса");
        }
    }
};

void function3() {
    Resource r3("r3");
    r3.use();
    std::cout << "function3 завершается" << std::endl;
}

void function2() {
    Resource r2("r2");
    try {
        function3();
    }
    catch (const std::exception& e) {
        std::cout << "function2 перехватила: " << e.what() << std::endl;
        throw;  // пробрасываем дальше
    }
    std::cout << "function2 завершается" << std::endl;
}

void function1() {
    Resource r1("r1");
    function2();
    std::cout << "function1 завершается" << std::endl;
}

int main() {
    std::cout << "=== Демонстрация раскрутки стека ===" << std::endl;
    
    try {
        function1();
    }
    catch (const std::exception& e) {
        std::cout << "main перехватила: " << e.what() << std::endl;
    }
    
    std::cout << "\n=== Гарантии при исключениях ===" << std::endl;
    
    // Умные указатели автоматически освобождают память
    try {
        auto ptr = std::make_unique<int>(42);
        std::cout << "Значение: " << *ptr << std::endl;
        throw std::runtime_error("Ошибка");
        // ptr автоматически удалится при раскрутке стека
    }
    catch (const std::exception& e) {
        std::cout << "Перехвачено: " << e.what() << std::endl;
        // Память не утекла, хотя delete не вызывали
    }
    
    return 0;
}
```

### 💻 ПРИМЕР 6 (Проблемы с сырыми указателями и их решение)

```cpp
#include <iostream>
#include <memory>
#include <fstream>

// Плохой пример - утечка памяти при исключении
void badExample() {
    int* ptr = new int(100);
    std::cout << "Выделена память: " << *ptr << std::endl;
    
    // Если здесь возникнет исключение, память утечет
    throw std::runtime_error("Ошибка");
    
    delete ptr;  // Этот код никогда не выполнится
    std::cout << "Память освобождена" << std::endl;
}

// Хороший пример - использование умных указателей
void goodExampleWithSmartPtr() {
    auto ptr = std::make_unique<int>(100);
    std::cout << "Выделена память: " << *ptr << std::endl;
    
    throw std::runtime_error("Ошибка");
    // Умный указатель автоматически освободит память при раскрутке стека
}

// Хороший пример - RAII класс для файла
class FileHandler {
private:
    std::ofstream file;
    
public:
    FileHandler(const std::string& filename) {
        file.open(filename);
        if (!file.is_open()) {
            throw std::runtime_error("Не удалось открыть файл");
        }
        std::cout << "Файл открыт" << std::endl;
    }
    
    ~FileHandler() {
        if (file.is_open()) {
            file.close();
            std::cout << "Файл закрыт" << std::endl;
        }
    }
    
    void write(const std::string& data) {
        file << data;
    }
};

void processFile() {
    FileHandler file("test.txt");
    file.write("Данные");
    throw std::runtime_error("Ошибка при обработке");
    // Файл закроется автоматически в деструкторе
}

int main() {
    std::cout << "=== Демонстрация проблем с сырыми указателями ===" << std::endl;
    
    try {
        // badExample();  // Раскомментируйте, чтобы увидеть утечку
    }
    catch (const std::exception& e) {
        std::cout << "Перехвачено: " << e.what() << std::endl;
    }
    
    std::cout << "\n=== Решение с умными указателями ===" << std::endl;
    
    try {
        goodExampleWithSmartPtr();
    }
    catch (const std::exception& e) {
        std::cout << "Перехвачено: " << e.what() << std::endl;
    }
    
    std::cout << "\n=== RAII для файлов ===" << std::endl;
    
    try {
        processFile();
    }
    catch (const std::exception& e) {
        std::cout << "Перехвачено: " << e.what() << std::endl;
    }
    
    return 0;
}
```

## 🛡️ Спецификации исключений и noexcept

В современном C++ спецификации исключений (`throw`()) устарели. Вместо них используется `noexcept`, который указывает, что функция не генерирует исключения.

### 💻 ПРИМЕР 7 (noexcept)

```cpp
#include <iostream>
#include <vector>

// Функция гарантирует, что не бросит исключение
void safeFunction() noexcept {
    std::cout << "Эта функция не бросает исключений" << std::endl;
    // Если здесь возникнет исключение, программа вызовет std::terminate
}

// Функция может бросить исключение
void riskyFunction() {
    std::cout << "Эта функция может бросить исключение" << std::endl;
    throw std::runtime_error("Ошибка");
}

// noexcept зависит от условия
void conditionalFunction(bool throwException) noexcept(false) {
    if (throwException) {
        throw std::runtime_error("Ошибка");
    }
}

// Проверка noexcept в compile-time
template <typename Func>
void checkNoexcept(Func f, const char* name) {
    std::cout << name << " noexcept? " 
              << (noexcept(f()) ? "да" : "нет") << std::endl;
}

class MyClass {
public:
    // Деструкторы по умолчанию noexcept
    ~MyClass() {
        std::cout << "Деструктор MyClass" << std::endl;
    }
    
    // Перемещающие операции обычно noexcept
    MyClass(MyClass&& other) noexcept {
        std::cout << "Перемещающий конструктор" << std::endl;
    }
    
    MyClass& operator=(MyClass&& other) noexcept {
        std::cout << "Перемещающее присваивание" << std::endl;
        return *this;
    }
    
    // Копирующие операции могут бросать исключения
    MyClass(const MyClass& other) {
        std::cout << "Копирующий конструктор" << std::endl;
    }
};

int main() {
    std::cout << "=== Проверка noexcept ===" << std::endl;
    
    checkNoexcept([] { safeFunction(); }, "safeFunction");
    checkNoexcept([] { riskyFunction(); }, "riskyFunction");
    
    std::cout << "\n=== Вектор и noexcept ===" << std::endl;
    
    std::vector<MyClass> vec;
    MyClass obj;
    
    std::cout << "Добавление элемента:" << std::endl;
    vec.push_back(obj);  // Использует копирование
    
    std::cout << "\nДобавление с перемещением:" << std::endl;
    vec.push_back(std::move(obj));  // Использует перемещение (noexcept)
    
    // noexcept важен для вектора: если перемещение noexcept,
    // вектор будет использовать его при реаллокации, иначе - копирование
    
    std::cout << "\n=== noexcept и terminate ===" << std::endl;
    
    try {
        // riskyFunction();  // Нормально перехватывается
        // safeFunction();   // Не бросает исключений
    }
    catch (...) {
        std::cout << "Перехвачено" << std::endl;
    }
    
    return 0;
}
```

## 🧪 Задачи для практической работы

### ✅ Задача 1. Деление с обработкой исключений

Напишите программу, которая запрашивает у пользователя два числа и выполняет их деление. Обработайте возможные исключения:

**Требования:**

- Запросите у пользователя два целых числа.
- Реализуйте функцию `double` `divide(int a, int b)`, которая:
- Если делитель равен нулю, генерирует исключение `std::invalid_argument` с сообщением "Деление на ноль"
- Иначе возвращает результат деления
- В функции `main()`:
- Оберните вызов `divide()` в `try`-блок
- Перехватите `std::invalid_argument` и выведите сообщение об ошибке
- Перехватите любые другие исключения (...) и выведите "Неизвестная ошибка"

Программа должна продолжать работу после обработки исключения (запрашивать новые числа в цикле до тех пор, пока пользователь не введет корректные данные или не выберет выход)

### ✅ Задача 2. Калькулятор с проверкой входных данных

Создайте программу-калькулятор, которая выполняет четыре арифметических операции и обрабатывает все возможные ошибки.

**Требования:**

- Запросите у пользователя два числа и операцию (+, -, *, /).
- Реализуйте функции для каждой операции.
- Обработайте следующие исключительные ситуации:
- Деление на ноль — `std::runtime_error`
- Некорректная операция — `std::invalid_argument`
- Переполнение при умножении — собственное исключение `OverflowException` (наследуйте от `std::exception`)
- Ввод нечисловых данных — `std::ios_base`::failure (при использовании потоков)
- Реализуйте повторный ввод при ошибках.
- Добавьте возможность завершения программы по команде "exit".

### ✅ Задача 3. Работа с динамической памятью и исключения

Создайте класс `DynamicArray`, который управляет динамическим массивом и корректно обрабатывает исключения.

**Требования:**

- Класс должен содержать:
- Указатель на массив целых чисел
- Размер массива
- Конструктор принимает размер и выделяет память. Если размер <= 0, генерируйте `std::invalid_argument`.
- Метод `get(int index)` возвращает элемент по индексу. Если индекс вне допустимого диапазона, генерируйте `std::out_of_range`.
- Метод `set(int index, int value)` устанавливает значение. Аналогичная проверка границ.
- Метод `resize(int newSize)` изменяет размер массива. При выделении памяти может возникнуть `std::bad_alloc`.
- Деструктор освобождает память.
- В функции `main()` продемонстрируйте работу с классом и обработку всех возможных исключений. Убедитесь, что при возникновении исключения в конструкторе или методе resize не происходит утечки памяти.

### ✅ Задача 4. RAII и исключения

Создайте несколько классов, демонстрирующих принцип `RAII` (Resource Acquisition Is Initialization) и их поведение при исключениях.

**Требования:**

- Создайте класс `FileWrapper`, который:
- В конструкторе открывает файл (используйте `std::ofstream` или `std::ifstream`)
- В деструкторе закрывает файл
- Имеет метод `write(const std::string& data)` для записи
- Создайте класс `DatabaseConnection`, который:
- В конструкторе "подключается к базе данных" (имитация: вывод сообщения)
- В деструкторе "отключается"
- Имеет метод `query(const std::string& sql)`, который может генерировать исключение `std::runtime_error`
- В функции `main()`:
- Создайте функцию, которая открывает файл, подключается к БД и выполняет запросы
- Покажите, что при возникновении исключения в любом месте все ресурсы корректно освобождаются (деструкторы вызываются)
- Продемонстрируйте разницу между использованием `RAII`-классов и сырых указателей/дескрипторов

### ✅ Задача 5. Иерархия собственных исключений

Создайте иерархию классов исключений для системы управления библиотекой.

**Требования:**

Создайте базовый класс `LibraryException`, наследующий от `std::exception`, с полем `errorCode`.

Создайте производные классы:
- `BookNotFoundException` (книга не найдена)
- `BookAlreadyBorrowedException` (книга уже выдана)
- `ReaderNotFoundException` (читатель не найден)
- `MaxBooksReachedException` (читатель достиг лимита книг)
- `InvalidReturnDateException` (некорректная дата возврата)

Каждый класс должен содержать дополнительную информацию, специфичную для ошибки (например, ISBN книги, ID читателя, текущее количество книг и т.д.).

Создайте класс `Library`, который управляет книгами и читателями, и использует эти исключения.

В функции `main()` продемонстрируйте различные сценарии и обработку исключений, включая полиморфный перехват по базовому классу.

### ✅ Задача 6. Безопасный стек с исключениями

Реализуйте шаблонный класс `SafeStack`, который обеспечивает строгую гарантию безопасности исключений.

Требования:

- Класс должен содержать:
- Динамический массив элементов
- Емкость и текущий размер
- Методы:
- `void` `push(const T& value)` — добавляет элемент. При нехватке памяти увеличивает емкость. Обеспечьте строгую гарантию: если в процессе увеличения емкости возникнет исключение, стек должен остаться в исходном состоянии.
- `void` `pop()` — удаляет верхний элемент. Если стек пуст, генерирует `std::out_of_range`.
- T& `top()` — доступ к верхнему элементу. Если стек пуст, генерирует `std::out_of_range`.
- `bool` `empty()` const
- `size_t` `size()` const
- Реализуйте "копируй и обменивай" (copy-and-swap) идиому для оператора присваивания.
- В функции `main()` продемонстрируйте, что при возникновении исключений состояние стека не повреждается.

### ✅ Задача 7. Парсинг чисел с обработкой ошибок

Напишите программу, которая читает числа из файла, суммирует их и обрабатывает все возможные ошибки.

Требования:

- Программа запрашивает имя файла у пользователя.
- Пытается открыть файл. Если файл не открывается, генерируется и обрабатывается исключение.
- Читает файл построчно, каждую строку пытается преобразовать в число.
- Обработайте следующие исключения:
- `std::ifstream`::failure при ошибках чтения
- `std::invalid_argument` при невозможности преобразовать строку в число
- `std::out_of_range` при выходе числа за пределы диапазона
- `std::bad_alloc` при нехватке памяти
- Подсчитайте:
- Количество успешно прочитанных чисел
- Количество строк с ошибками
- Сумму чисел
- Выведите статистику и сообщения об ошибках.
- Гарантируйте, что файл будет закрыт даже при возникновении исключений (используйте `RAII`).

### ✅ Задача 8. Комплексная задача: система бронирования билетов

Разработайте систему бронирования билетов на концерты с комплексной обработкой исключений.

**Требования:**

Создайте класс `Concert` с полями:
- Название
- Дата
- Общее количество мест
- Количество свободных мест
- Цена билета

Создайте иерархию исключений:
- `BookingException` (базовое)
- `ConcertNotFoundException` (концерт не найден)
- `NoSeatsAvailableException` (нет свободных мест)
- `InvalidBookingDataException` (некорректные данные при бронировании)
- `PaymentFailedException` (ошибка оплаты)

Создайте класс `BookingSystem`:
- Хранит вектор концертов
- Метод `addConcert()` — добавляет концерт
- Метод `findConcertsByDate()` — поиск по дате
- Метод `bookTickets()` — бронирование билетов (проверяет наличие мест, генерирует исключения)
- Метод `processPayment()` — обработка платежа (имитация, может генерировать `PaymentFailedException`)
- Метод `generateReport()` — формирование отчета

Все методы должны обеспечивать гарантии безопасности исключений:
- При ошибке в `bookTickets()` состояние системы не должно измениться
- При ошибке оплаты забронированные места должны освобождаться

В функции `main()` реализуйте интерактивный интерфейс:
- Просмотр концертов
- Бронирование билетов
- Отмена бронирования
- Вывод отчета
- Обработайте все возможные исключения и предоставьте пользователю понятные сообщения об ошибках.

<div align="center">
  <img alt="Project Demo" src="./mygif/gif-25.2.gif" />
</div>

---

<div align="center"> Made with ❤️ by <b>dv0retsky</b> </div>