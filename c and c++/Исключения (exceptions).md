___

```cpp
#include <iostream>
#include <stdexcept>

int divide(int a, int b) {
    if (b == 0) {
        throw std::invalid_argument("Division by zero");
    }
    return a / b;
}

int main() {
    try {
        int result = divide(10, 0);
        // Может выбросить исключение
        std::cout << "Result: " << result << std::endl;
    } catch (const std::invalid_argument& e) {
        std::cerr << "Error: " << e.what() << std::endl;
    } catch (...) {
        std::cerr << "An unknown error occurred" << std::endl;
    }

    return 0;
}
```

---
### Ключевые слова throw, try, catch

хз, мне кажется все тут понятно, даже писать лень

---

### Отличие между исключениями и ошибками (рантаймами)

ну это тоже жесткий очев, я бы тока это запомнил:
***Иерархия стандартных исключений:***

- `std::exception` — базовый класс для всех стандартных исключений.
    - `std::logic_error` — ошибки, которые можно выявить до выполнения программы (например, неверные аргументы).
        - `std::invalid_argument`
        - `std::out_of_range`
    - `std::runtime_error` — ошибки, которые возникают во время выполнения программы.
        - `std::overflow_error`
        - `std::underflow_error`

---

### Последовательность и правила обработки исключений

> Компилятор(?) идет по `catch` по порядку и берет первый, который больше подходит
> Если бросили исключение типа наследник, то по ссылке на родителя его можно принять
> Если `catch(void*)`, то поймаем любой указатель

Можно бросить исключение дальше из `catch`

---

### Спецификации исключений 

```cpp
void foo() noexcept; // Функция не выбрасывает исключений
void bar() noexcept(true); // То же, что и noexcept

void baz() noexcept(false); 
// Функция может выбрасывать исключения
```


---

