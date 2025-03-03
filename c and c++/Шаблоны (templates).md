___

### Объявление шаблонов

- шаблоны функций

```cpp
// вместо typename можно писать class (смысл не меняется)
template <typename T>
void swap(T& x, T&y) {
	T t = x;
	x = y;
	y = t;
}
```

- шаблоны классов

```cpp
template <typename T>
class C {
	// реализация всякая от типа Т
};

int main() {
	int x=3, y=4;
	swap(x,y);
	// swap<int>(x,y); явное указание шаблонного параметра
	C<int> c; // обязательно нужно указывать шаблонный параметр
}
```

Можно объявлять значения по умолчанию: `template <typename T = int>`, теперь можно писать так `C c(12);`

---

### Специализации шаблонов

> Как и в случае перегрузки функций, может быть несколько шаблонов с одинаковым именем, но с разным набором шаблонных параметров

Специализация шаблона позволяет определить специфическое поведение для определённого типа данных.

```cpp
template <typename T>
void print(T value) {
    std::cout << "Generic template: "
	    << value << "\n";
}

template <>
void print<int>(int value) {
    std::cout << "Specialized template for int: "
		<< value << "\n";
}
```

Компилятор в первую очередь рассматривает точные указания типов

---

### Алиасы для типов (`typedef`)

Ключевое слово `typedef` используется для создания псевдонимов (алиасов) для существующих типов данных.

1. Создание псевдонима для простого типа

```cpp
typedef unsigned long ulong;
```

2. Создание псевдонима для указателя

```cpp
typedef int* intPtr;
```

3. Создание псевдонима для структуры

```cpp
typedef struct {
	 int x;
	 int y; 
} Point;
```

4. Создание псевдонима для функции

```cpp
typedef void (*FuncPtr)(int);

void printNumber(int num) {
    std::cout << num << std::endl;
}

int main() {
    FuncPtr func = printNumber;
    func(5);
    return 0;
}
```

5. Использование с шаблонами (шаблонные `typedef`-ы)

```cpp
template <typename T>
using myType<T> = VeryLongTypename <int, vector<T>> ;
```

---
### Использование `typename` с целью сообщить, что указано название типа

При работе с шаблонами, компилятор может не знать, является ли имя типом или полем, пока не увидит полное определение шаблона, в таких случаях используется `typename`, чтобы явно указать, что имя является типом.

```cpp
template <typename T>
class C {
	// сделали просто так
	typedef T type;
};

template <typename T>
int f() {
	C<T>::type x;
	/*
	это ошибка компиляции
	потому что появляется неоднозначность, ведь
	type это либо поле либо название типа

	решение вопроса неоднозначности:
	КОМПИЛЯТОР будет считать что это поле
	*/

	typename C<T>::type x;
	/*
	теперь компилятор точно знает, что это название типа
	и ошибка компиляции исчезнет
	*/
}
```

---

### remove_const, remove_reference, etc.

```cpp
template <typename T>
struct remove_const<const T> {
	typedef T type;
};

template <typename T>
struct remove_const<T> {
	typedef T type;
};

int main() {
typename remove_const<const int>::type x = 10;
// имеет тип int

typename remove_const<int>::type y = 12;
// тоже имеет тип int
}

```

---

### Правила вывода шаблонных типов

- ссылка отбрасывается при выводе шаблонных типов, а константность нет

```cpp
// случай где функция принимает ссылку Т
template <typename T>
void f(T& x);

int main() {
	int x;
	f(x); 
	// T = int, x имеет тип: int&
	
	int& y = x;
	f(y);
	// T = int, y: int&
	
	const int z = 0;
	f(z);
	// T = const int, z: const int&
	
	const int& t = z;
	f(t);
	// T = const int, t: const int&
}

// случай где функция принимает просто Т
// тут уже  отбрасываются не только ссылки,
// но и константы
template <typename T>
void f(T x);

int main() {
	int x;
	f(x); 
	// T = int, x: int
	
	int& y = x;
	f(y);
	// T = int, y: int
	
	const int z = 0;
	f(z);
	// T = int, z: int
	
	const int& t = z;
	f(t);
	// T = int, t: int
}
```

***Ссылки отбрасываются:***
- Когда параметр функции **не является ссылкой или указателем**
- Когда параметр функции - **универсальная ссылка**, и аргумент - **rvalue**

***Ссылки не отбрасываются:***
- Когда параметр функции - **ссылка или указатель** (`T& param` или `T* param`)
- Когда параметр функции - **универсальная ссылка**, и аргумент - **lvalue**

***Константы не отбрасываются:***
- Когда параметр функции **является  ссылкой или указателем**

---

### non-type template parameters (параметры шаблонов, не являющиеся типами)

брух
```cpp
template <typename X, template <typename> class T>
// принимает контейнер типа Т
// из элементов типа Х
void f(const T<X>& t); 

...
array<int> a;
f<int, array>(a);
```

---

### Variadic templates (шаблоны с переменным количеством шаблонных аргументов)

Это возможность , которая позволяет создавать шаблоны, принимающие произвольное количество аргументов. Они полезны для реализации функций или классов, которые должны работать с разным количеством входных данных, например, для создания кортежей (`std::tuple`)

```cpp
template <typename... Args>
// ф-ция принимающая произвольное количество
// аргументов произвольных типов
void f(Args... args);

void f() {
    std::cout << "end\n";
}

template <typename Head, typename... Tail>
void f(Head x, Tail... t) {
	std::cout << x;
	f(t...); // Рекурсивный вызов с оставшимися аргументами
	// случай, когда элементы кончились нужно
	// разбирать отдельно (сделано выше)
	// т.е. реализовать функцию без аргументов
}

// чтобы не создавались копии принимаем ссылку
// а лучше универсальную ссылку
void f(Head&& x, Tail&&... t) {
	std::cout << x;
	f(t...); // Все аргументы передаются как lvalue
	// f(std::forward<Tail>(t)...); // "идеальная передача"
}
```

С помощью оператора `sizeof` можно узнать количество аргументов в пакете параметров:

```cpp
template <typename... Args>
void printCount(Args... args) {
    std::cout << sizeof...(Args);
}

int main() {
    printCount(1, 2.5, "hello"); // Выведет 3
    return 0;
}
```

---

### Функторы (объекты-функции)

Объекты классов, для которых перегружен оператор `()`. Они ведут себя как функции, но могут сохранять состояние.

```cpp
// в примере используется функтор
// сравнения элементов cmp (compare)
template <typename T>
struct Cmp {
    bool operator()(const T& a, const T& b) const {
        return a < b;
    }
};

template <typename T, typename Cmp>
void qsort(T* begin, T* end, Cmp cmp) {
	...
	cmp(*x, *y);
	// у объекта типа Cmp должен быть перегружен
	// оператор () для двух аргументов
}
```

Или можно не писать свой сравниватель и просто использовать библиотечный

```cpp
template <typename T, typename Cmp = std::less<T>>
void qsort(T* begin, T* end, Cmp cmp = Cmp()) {
    if (begin >= end) return;
    T* pivot = partition(begin, end, cmp);
    qsort(begin, pivot, cmp);
    qsort(pivot + 1, end, cmp);
}
```

---

### CRTP (Curious Recurring Template Pattern) или как обойтись без слова virtual

Идиома, которая позволяет имитировать динамический полиморфизм (как при использовании виртуальных функций) без накладных расходов на виртуальные таблицы (vtable). 
Основная идея CRTP заключается в том, что класс-наследник передаёт себя как шаблонный параметр в базовый класс. Это позволяет базовому классу вызывать методы наследника напрямую, без использования виртуальных функций.

```cpp
template <typename T>
struct Base {
	void f() {
		static_cast<T&>(this)->f();
	}
};

struct Derived : Base<Derived> {
	void f() {
		// делает что-то полезное
	}
};

int main() {
	Derived d;
	Base* b = d;
	b.f();
}
```

---