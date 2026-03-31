# Тема №24. Статическое приведение объектов к разным типам. Динамическое определение типа 🎀

## 📝 Введение

В процессе разработки программ на C++ часто возникает необходимость работать с данными разных типов. Иногда требуется преобразовать значение одного типа в другой — например, целое число в вещественное для выполнения деления, или указатель на базовый класс в указатель на производный для доступа к специфическим методам. Этот процесс называется **приведением типов (type casting)**.

В C существует достаточно простой и грубый механизм приведения типов — запись типа в круглых скобках перед выражением. Однако такой подход имеет серьезные недостатки: он позволяет выполнить практически любое преобразование, даже если оно не имеет смысла, и не дает программисту контроля над тем, насколько такое преобразование безопасно.

<div align="center">
  <img alt="Project Demo" src="./mygif/gif24-1.gif" />
</div>

C++ предлагает более совершенную систему приведения типов, которая включает четыре специализированных оператора: `static_cast`, `dynamic_cast`, `const_cast` и `reinterpret_cast`. Каждый из этих операторов предназначен для решения определенного круга задач и позволяет сделать код не только более безопасным, но и самодокументируемым — читатель сразу понимает, зачем выполняется приведение.

Кроме того, в C++ существует механизм **динамического определения типа (RTTI — Run-Time Type Information)**. Это возможность во время выполнения программы узнать точный тип объекта, на который ссылается указатель или ссылка базового класса. Это особенно полезно при работе с иерархиями наследования, когда один и тот же интерфейс может скрывать объекты разных типов.

На этом занятии мы подробно изучим все четыре оператора приведения, разберем ситуации, в которых они применяются, а также познакомимся с оператором `typeid` и классом `type_info` для динамического определения типа объектов.

## 🧃 Явное и неявное приведение типов

В C++ существует два способа преобразования типов: неявное (автоматическое) и явное (выполняемое программистом).

**Неявное приведение** происходит автоматически в следующих ситуациях:

- При выполнении арифметических операций с операндами разных типов
- При передаче аргумента в функцию, ожидающую тип другого типа
- При присваивании значения переменной другого типа

**Явное приведение** выполняется программистом с помощью операторов приведения или C-стиля.

### 🍑 ПРИМЕР 1 (Неявное приведение)

```cpp
#include <iostream>

int main() {
    int a = 5;
    double b = 2.5;

    // Неявное приведение int к double перед сложением
    double result1 = a + b;  // a временно преобразуется в double
    std::cout << "a + b = " << result1 << std::endl;

    // Неявное приведение double к int при присваивании
    int result2 = b;  // 2.5 преобразуется в 2 (с потерей данных)
    std::cout << "b = " << b << ", result2 = " << result2 << std::endl;

    // Неявное приведение при передаче аргумента
    double divide(int x, int y) {
        return x / y;  // Результат деления int сначала int, потом в double
    }

    std::cout << "5 / 2 = " << divide(5, 2) << std::endl;  // 2.0, а не 2.5!

    return 0;
}
```

### 🥝 ПРИМЕР 2 (Проблемы неявного приведения)

```cpp
#include <iostream>

class Parent {
public:
    virtual ~Parent() {}
    void parentMethod() { std::cout << "Метод родителя" << std::endl; }
};

class Child : public Parent {
public:
    void childMethod() { std::cout << "Метод потомка" << std::endl; }
};

class AnotherClass {
public:
    void someMethod() { std::cout << "Какой-то другой класс" << std::endl; }
};

int main() {
    // Неявное приведение указателей в иерархии наследования
    Child* childPtr = new Child();
    Parent* parentPtr = childPtr;  // Неявное приведение Child* -> Parent* (безопасно)

    // parentPtr->childMethod();  // Ошибка! Parent не знает о childMethod

    // Проблема: приведение Parent* обратно к Child*
    Parent* someParent = new Parent();
    // Child* badPtr = someParent;  // Ошибка компиляции! Неявно нельзя

    // C-стиль приведения (опасно!)
    Child* unsafePtr = (Child*)someParent;  // Компилируется, но опасно!
    // unsafePtr->childMethod();  // Неопределенное поведение! someParent - не Child

    // Еще опаснее - приведение к несвязанному типу
    AnotherClass* wrongPtr = (AnotherClass*)childPtr;  // Компилируется!
    // wrongPtr->someMethod();  // Полная катастрофа!

    delete childPtr;
    delete someParent;

    return 0;
}
```

## 🍟 Операторы приведения типов в C++

Для решения проблем, связанных с приведением типов, в C++ были введены четыре специализированных оператора приведения. Каждый из них имеет четкую область применения и выполняет проверки на разных этапах.

### 🍓 ПРИМЕР 3 (static_cast)

`static_cast` — наиболее часто используемый оператор приведения. Он выполняет преобразования на этапе компиляции и используется для:

- Преобразования между числовыми типами **(int → double, double → int)**
- Преобразования указателей в иерархии наследования (но только там, где связь известна на этапе компиляции)
- Преобразования `void*` к конкретному типу указателя
- Вызова явных конструкторов и операторов преобразования

```cpp
#include <iostream>

class Base {
public:
    virtual ~Base() {}
    void baseMethod() { std::cout << "Base::baseMethod()" << std::endl; }
};

class Derived : public Base {
public:
    void derivedMethod() { std::cout << "Derived::derivedMethod()" << std::endl; }
};

class Unrelated {
public:
    void someMethod() { std::cout << "Unrelated::someMethod()" << std::endl; }
};

int main() {
    // 1. Преобразование числовых типов
    double pi = 3.14159;
    int intPi = static_cast<int>(pi);  // Отбрасывает дробную часть
    std::cout << "pi = " << pi << ", intPi = " << intPi << std::endl;

    // 2. Преобразование в иерархии наследования (downcast)
    Base* basePtr = new Derived();

    // static_cast для downcast - работает, но не проверяет реальный тип
    Derived* derivedPtr = static_cast<Derived*>(basePtr);
    derivedPtr->derivedMethod();  // Безопасно, если basePtr действительно указывает на Derived

    // 3. Опасное использование static_cast
    Base* anotherBase = new Base();
    // Derived* badDerived = static_cast<Derived*>(anotherBase);  // Компилируется, но опасно!
    // badDerived->derivedMethod();  // Неопределенное поведение!

    // 4. static_cast не работает с несвязанными типами
    // Unrelated* unrelated = static_cast<Unrelated*>(basePtr);  // Ошибка компиляции!

    // 5. Преобразование через void*
    void* voidPtr = static_cast<void*>(derivedPtr);
    Derived* backAgain = static_cast<Derived*>(voidPtr);
    backAgain->derivedMethod();

    delete basePtr;
    delete anotherBase;

    return 0;
}
```

### 🥥 ПРИМЕР 4 (`dynamic_cast`)

`dynamic_cast` — оператор приведения, который выполняет проверку на этапе выполнения. Он используется исключительно для полиморфных типов (классов, имеющих хотя бы одну виртуальную функцию). `dynamic_cast` проверяет, можно ли безопасно выполнить преобразование, и в случае неудачи возвращает `nullptr` для указателей или генерирует исключение для ссылок.

```cpp
#include <iostream>
#include <typeinfo>

class Animal {
public:
    virtual ~Animal() {}  // Наличие виртуальной функции делает класс полиморфным
    virtual void speak() const { std::cout << "Животное издает звук" << std::endl; }
};

class Dog : public Animal {
public:
    void speak() const override { std::cout << "Гав-гав!" << std::endl; }
    void wagTail() const { std::cout << "Собака виляет хвостом" << std::endl; }
};

class Cat : public Animal {
public:
    void speak() const override { std::cout << "Мяу!" << std::endl; }
    void purr() const { std::cout << "Кошка мурлычет" << std::endl; }
};

void interactWithAnimal(Animal* animal) {
    animal->speak();

    // Пытаемся определить, собака ли это
    Dog* dog = dynamic_cast<Dog*>(animal);
    if (dog) {
        std::cout << "Это собака! ";
        dog->wagTail();
    }

    // Пытаемся определить, кошка ли это
    Cat* cat = dynamic_cast<Cat*>(animal);
    if (cat) {
        std::cout << "Это кошка! ";
        cat->purr();
    }
}

int main() {
    Animal* animals[] = { new Dog(), new Cat(), new Animal() };

    for (Animal* animal : animals) {
        interactWithAnimal(animal);
        std::cout << "-------------------" << std::endl;
    }

    // Демонстрация dynamic_cast для ссылок
    Dog dog;
    Animal& animalRef = dog;

    try {
        Cat& catRef = dynamic_cast<Cat&>(animalRef);  // Здесь dog - это не Cat!
        catRef.purr();  // Не выполнится
    } catch (const std::bad_cast& e) {
        std::cout << "Исключение: " << e.what() << " - это не кошка!" << std::endl;
    }

    // Очистка памяти
    for (Animal* animal : animals) {
        delete animal;
    }

    return 0;
}
```

### 🍇 ПРИМЕР 5 (`const_cast`)

`const_cast` — единственный оператор приведения, который может удалять (или добавлять) квалификаторы `const` и `volatile`. Используется в ситуациях, когда нужно передать константный объект в функцию, которая ожидает неконстантный указатель, но при этом гарантируется, что функция не будет модифицировать объект.

```cpp
#include <iostream>
#include <cstring>

// Функция из старой C-библиотеки, которая не использует const
void legacyToUpper(char* str) {
    for (size_t i = 0; i < strlen(str); ++i) {
        str[i] = toupper(str[i]);
    }
}

// Функция, которая логирует значение, но не меняет его
void logValue(const int* value) {
    std::cout << "Значение: " << *value << std::endl;

    // Иногда нужно передать значение в старую функцию, которая требует неконстантный указатель
    // Но мы уверены, что функция не изменит данные
    int* nonConst = const_cast<int*>(value);
    std::cout << "Через nonConst: " << *nonConst << std::endl;
    // *nonConst = 100;  // Если бы мы это сделали, это было бы опасно!
}

class DataProcessor {
private:
    mutable int cacheHits;  // mutable позволяет изменять поле даже в const-объекте

public:
    DataProcessor() : cacheHits(0) {}

    int computeExpensive() const {
        // const_cast для удаления const у this
        // Используется, когда нужно изменить mutable-поле (но mutable - лучше)
        const_cast<DataProcessor*>(this)->cacheHits++;

        // Имитация сложных вычислений
        return 42;
    }

    int getCacheHits() const { return cacheHits; }
};

int main() {
    // 1. Ситуация с устаревшими функциями
    const char* constStr = "hello world";
    char buffer[100];
    strcpy(buffer, constStr);

    // legacyToUpper(constStr);  // Ошибка компиляции!
    legacyToUpper(buffer);  // Работает с буфером
    std::cout << "Буфер после преобразования: " << buffer << std::endl;

    // Используем const_cast (ТОЛЬКО если уверены!)
    char* nonConstPtr = const_cast<char*>(constStr);
    // legacyToUpper(nonConstPtr);  // Опасно! Может изменить константную строку

    // 2. Работа с const-объектами
    const int value = 100;
    logValue(&value);

    // 3. Использование const_cast для работы с mutable
    DataProcessor proc;
    for (int i = 0; i < 5; ++i) {
        proc.computeExpensive();
    }
    std::cout << "Количество обращений к кэшу: " << proc.getCacheHits() << std::endl;

    return 0;
}
```

### 🍒 ПРИМЕР 6 (`reinterpret_cast`)

`reinterpret_cast` — самый опасный оператор приведения. Он выполняет низкоуровневое переинтерпретирование битового представления. Используется крайне редко, в основном для системного программирования, работы с аппаратурой или специфических оптимизаций.

```cpp
#include <iostream>

struct Data {
    int id;
    double value;
    char name[20];
};

// Функция для записи бинарных данных в файл или отправки по сети
void writeBinary(const unsigned char* data, size_t size) {
    std::cout << "Запись " << size << " байт бинарных данных" << std::endl;
    for (size_t i = 0; i < std::min(size, size_t(10)); ++i) {
        std::cout << std::hex << static_cast<int>(data[i]) << " ";
    }
    std::cout << std::dec << std::endl;
}

int main() {
    // 1. Преобразование между несвязанными типами указателей
    int number = 0x12345678;
    int* intPtr = &number;

    // Интерпретируем int* как char* для побайтового доступа
    char* bytePtr = reinterpret_cast<char*>(intPtr);
    std::cout << "Байты числа 0x12345678: ";
    for (size_t i = 0; i < sizeof(int); ++i) {
        std::cout << std::hex << static_cast<int>(bytePtr[i]) << " ";
    }
    std::cout << std::dec << std::endl;

    // 2. Преобразование указателя в целое число и обратно
    // (полезно для низкоуровневого программирования)
    uintptr_t address = reinterpret_cast<uintptr_t>(intPtr);
    std::cout << "Адрес числа: " << address << std::endl;

    int* recoveredPtr = reinterpret_cast<int*>(address);
    std::cout << "Восстановленное значение: " << *recoveredPtr << std::endl;

    // 3. Сериализация структуры в бинарный вид
    Data myData = {42, 3.14159, "Test"};

    // Интерпретируем структуру как массив байт
    unsigned char* binaryData = reinterpret_cast<unsigned char*>(&myData);
    writeBinary(binaryData, sizeof(Data));

    // 4. Опасное использование (демонстрация)
    float pi = 3.14159f;
    // Интерпретируем float как int (получим мусор, но иногда полезно для анализа битов)
    int intBits = *reinterpret_cast<int*>(&pi);
    std::cout << "Битовое представление float " << pi << " как int: "
              << std::hex << intBits << std::dec << std::endl;

    // reinterpret_cast не может убрать const
    const int* constInt = &number;
    // int* bad = reinterpret_cast<int*>(constInt);  // Ошибка! Используйте const_cast

    return 0;
}
```

## 🍉 Динамическое определение типа (RTTI)

**RTTI (Run-Time Type Information)** — механизм, позволяющий получить информацию о типе объекта во время выполнения программы. В C++ RTTI реализован с помощью оператора `typeid` и класса `type_info`.

### 🍊 ПРИМЕР 7 (typeid и type_info)

```cpp
#include <iostream>
#include <typeinfo>

class Base {
public:
    virtual ~Base() {}  // Для полиморфного использования typeid
};

class Derived1 : public Base {};
class Derived2 : public Base {};

// Шаблонная функция для демонстрации typeid
template <typename T>
void printTypeInfo(const T& value) {
    std::cout << "Тип: " << typeid(T).name()
              << ", значение: " << typeid(value).name()
              << std::endl;
}

void identifyObject(Base* obj) {
    if (obj) {
        const std::type_info& typeInfo = typeid(*obj);
        std::cout << "Объект имеет тип: " << typeInfo.name() << std::endl;

        if (typeInfo == typeid(Derived1)) {
            std::cout << "  -> Это Derived1!" << std::endl;
        } else if (typeInfo == typeid(Derived2)) {
            std::cout << "  -> Это Derived2!" << std::endl;
        } else if (typeInfo == typeid(Base)) {
            std::cout << "  -> Это просто Base!" << std::endl;
        }
    }
}

class NoVirtual {
    // Нет виртуальных функций - typeid будет статическим
};

int main() {
    // 1. typeid для встроенных типов
    int i = 42;
    double d = 3.14;
    const char* str = "hello";

    std::cout << "typeid(int).name(): " << typeid(int).name() << std::endl;
    std::cout << "typeid(i).name(): " << typeid(i).name() << std::endl;
    std::cout << "typeid(d).name(): " << typeid(d).name() << std::endl;
    std::cout << "typeid(str).name(): " << typeid(str).name() << std::endl;
    std::cout << "typeid(const char*).name(): " << typeid(const char*).name() << std::endl;

    // 2. typeid в иерархии наследования
    Base base;
    Derived1 derived1;
    Derived2 derived2;

    Base* ptr1 = &base;
    Base* ptr2 = &derived1;
    Base* ptr3 = &derived2;

    std::cout << "\nИдентификация объектов:" << std::endl;
    identifyObject(ptr1);
    identifyObject(ptr2);
    identifyObject(ptr3);

    // 3. Сравнение type_info
    if (typeid(base) == typeid(derived1)) {
        std::cout << "base и derived1 одного типа" << std::endl;
    } else {
        std::cout << "base и derived1 разных типов" << std::endl;
    }

    // 4. typeid для неполиморфных типов
    NoVirtual nv1, nv2;
    NoVirtual* nvPtr = &nv1;

    // Для неполиморфных типов typeid определяется на этапе компиляции
    std::cout << "\nНеполиморфный тип: " << typeid(nv1).name() << std::endl;
    std::cout << "Через указатель: " << typeid(*nvPtr).name() << std::endl;

    // 5. Дополнительная информация из type_info
    const std::type_info& ti = typeid(d);
    std::cout << "\nИмя типа: " << ti.name() << std::endl;
    std::cout << "Хэш-код: " << ti.hash_code() << std::endl;

    // 6. typeid с нулевым указателем (осторожно!)
    Base* nullPtr = nullptr;
    try {
        // Разыменование нулевого указателя для typeid - исключение!
        const std::type_info& bad = typeid(*nullPtr);
    } catch (const std::bad_typeid& e) {
        std::cout << "\nИсключение: " << e.what() << std::endl;
    }

    return 0;
}
```

## 🍍 Задачи для практической работы

### 🌟 Задача 1. Статическое приведение числовых типов

Создайте программу, которая демонстрирует использование `static_cast` для преобразования между различными числовыми типами.

**Требования:**

- Объявите переменные следующих типов: `int`, `double`, `float`, `char`, `bool`.
- Присвойте им произвольные значения.

С помощью `static_cast` выполните следующие преобразования:

- `int` → `double` и обратно `double` → `int`
- `double` → `float` и обратно `float` → `double`
- `char` → `int` (получить код символа)
- `int` → `char` (получить символ по коду)
- `bool` → `int` и `int` → `bool`
- Для каждого преобразования выведите исходное значение, результат преобразования и объясните, что произошло с данными (были ли потери, округления и т.д.).
- Особое внимание уделите преобразованию `double` → `int` с дробной частью и преобразованию `int` → `char`, если число выходит за пределы диапазона символов.

### 🌟 Задача 2. Приведение в иерархии наследования

Создайте иерархию классов, моделирующую геометрические фигуры.

**Требования:**


Создайте базовый класс Shape с виртуальными методами:

- virtual double area() const = 0 — вычисление площади
- virtual void print() const — вывод информации о фигуре

Создайте два производных класса:

- Circle с полем radius (double)
- Rectangle с полями width и height (double)
- В каждом производном классе реализуйте метод area().

**В функции main():**

- Создайте массив указателей на Shape, содержащий объекты разных типов
- Используя static_cast, преобразуйте указатель на Shape в указатель на Circle или Rectangle (только когда точно знаете тип)
- Вызовите специфические методы, например getRadius() для круга
- Продемонстрируйте, что произойдет, если выполнить неправильное приведение (компиляция пройдет, но поведение будет неопределенным)

### 🌟 Задача 3. Безопасный downcast с dynamic_cast

Используя иерархию классов из предыдущей задачи, модифицируйте программу для безопасного определения типа с помощью dynamic_cast.

**Требования:**

- Убедитесь, что базовый класс Shape имеет виртуальный деструктор (для работы RTTI).

Добавьте в производные классы специфические методы:

- Circle::getRadius() — возвращает радиус
- Rectangle::getWidth() и getHeight() — возвращают размеры
- В функции main() создайте контейнер (например, std::vector<Shape*>) с разными фигурами.

Напишите функцию processShape(Shape* shape), которая:

- Пытается преобразовать shape в Circle* с помощью dynamic_cast
- Если успешно, выводит радиус и площадь круга
- Если нет, пытается преобразовать в Rectangle*
- Если успешно, выводит размеры и площадь прямоугольника
- Если ни то, ни другое, выводит сообщение о неизвестной фигуре
- Продемонстрируйте работу функции для всех фигур в контейнере.
- Добавьте обработку ссылок с dynamic_cast и перехват исключения std::bad_cast.

### 🌟 Задача 4. Удаление константности с const_cast

Создайте класс StringProcessor, который работает со строками, и продемонстрируйте ситуации, где может потребоваться const_cast.

**Требования:**

- Напишите функцию void toUpperCase(char* str) из старой C-библиотеки, которая преобразует строку в верхний регистр (используйте toupper).
- Напишите функцию void printString(const char* str), которая выводит строку.

**В функции main():**

- Создайте константную строку: const char* constStr = "hello world"
- Создайте неконстантный массив символов и скопируйте в него строку
- Попробуйте передать constStr в функцию toUpperCase — должно вызвать ошибку компиляции
- Используйте const_cast, чтобы временно снять константность и передать строку в функцию (НО обязательно объясните в комментариях, почему это опасно)
- Создайте класс с полем mutable и покажите, как const_cast можно использовать для изменения такого поля в константном методе (альтернатива mutable)
- Объясните, в каких случаях использование const_cast допустимо, а в каких — опасно.

### 🌟 Задача 5. Низкоуровневое представление данных с reinterpret_cast

Создайте программу, которая исследует внутреннее представление различных типов данных в памяти с помощью reinterpret_cast.

**Требования:**


Создайте структуру Student с полями:

- int id
- double averageGrade
- char name[32]
- Заполните структуру тестовыми данными.
- Напишите функцию printMemory(const unsigned char* data, size_t size), которая выводит побайтовое представление данных в шестнадцатеричном формате.
- Используя reinterpret_cast, получите доступ к памяти структуры как к массиву байт и выведите её содержимое.
- Проделайте то же самое для простых типов (int, double) и объясните порядок байт (little-endian/big-endian) на вашей системе.
- Покажите, как можно записать структуру в бинарный файл и прочитать обратно, используя reinterpret_cast.
- Важно: Добавьте комментарии, объясняющие, почему reinterpret_cast опасен и когда его применение оправдано.

### 🌟 Задача 6. Динамическая идентификация типов с typeid

Создайте систему обработки событий, где тип события определяется во время выполнения с помощью typeid.

**Требования:**


Создайте иерархию классов событий:

- Базовый класс Event с виртуальным деструктором
- MouseEvent с полями x, y (int)
- KeyboardEvent с полем keyCode (int)
- WindowEvent с полем windowId (int)
- Создайте класс EventHandler, который содержит метод handleEvent(Event* event).

**В методе handleEvent:**

- Используйте `typeid` для определения точного типа события
- Выведите имя типа с помощью `typeid(*event).name()`
- В зависимости от типа, выполните соответствующую обработку (вывод информации о событии)

**В функции main():**

- Создайте вектор указателей на `Event`, содержащий события разных типов
- Передайте каждое событие в обработчик
- Сравните `type_info` для объектов одного типа и разных типов
- Продемонстрируйте, что произойдет с `typeid`, если базовый класс не имеет виртуальных функций (будет определяться статический тип).

### 🌟 Задача 7. Комбинированное использование всех операторов приведения

Создайте комплексный пример, демонстрирующий совместное использование всех четырех операторов приведения в реалистичном сценарии.

**Требования:**


Создайте следующую иерархию:

- Абстрактный базовый класс `IMediaItem` (интерфейс)
- Классы `AudioTrack`, `VideoTrack`, `Playlist` (наследуют `IMediaItem`)
- Добавьте в классы специфические поля и методы.

Реализуйте класс `MediaPlayer`, который:

- Хранит список указателей на `IMediaItem*`
- Имеет метод `play(IMediaItem* item)`, использующий dynamic_cast для определения типа и вызова соответствующего метода воспроизведения
- Имеет метод `exportMetadata(const IMediaItem* item)`, который с помощью `const_cast` временно снимает константность для вызова устаревшей функции логирования
- Имеет метод `saveToFile(IMediaItem* item)`, который с помощью reinterpret_cast интерпретирует объект как массив байт для бинарной сериализации
- Использует `static_cast` для преобразования в иерархии, когда тип точно известен (например, при работе с собственными внутренними структурами)
- В функции `main()` продемонстрируйте работу всех методов и объясните, какой оператор приведения где используется и почему.

### 🌟 Задача 8. Полиморфный сериализатор с RTTI

Разработайте систему сериализации объектов, которая использует RTTI для определения типа и сохранения/загрузки объектов в файл.

**Требования:**


Создайте абстрактный базовый класс Serializable с чисто виртуальными методами:

- virtual void serialize(std::ostream& os) const = 0
- virtual void deserialize(std::istream& is) = 0
- virtual std::string getTypeName() const = 0
- Создайте несколько производных классов (например, `Person`, `Product`, `Order`), реализующих эти методы.

Реализуйте класс ObjectFactory, который:

- Хранит отображение из имени типа в функцию создания объекта
- Умеет создавать объект по имени типа

Реализуйте класс `Serializer`, который:

- При сохранении объекта: сначала записывает в файл имя типа (полученное через `typeid` или пользовательский метод), затем данные объекта через `serialize`
- При загрузке: читает имя типа, создает объект через фабрику, затем вызывает `deserialize`
- В `main()` создайте несколько объектов разных типов, сохраните их в файл, затем загрузите обратно и проверьте, что все данные восстановлены корректно.
- Используйте `dynamic_cast` при загрузке для проверки типов (если потребуется).

<div align="center">
  <img alt="Project Demo" src="./mygif/gif24-2.gif" />
</div>

---

<div align="center"> Made with ❤️ by <b>dv0retsky</b> </div>