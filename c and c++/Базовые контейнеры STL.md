___

> ***Контейнеры делятся на две группы:***
> 
> 1) vector, deque, list - "последовательные" контейнеры, отображение в виде последовательности
> 2) map, set, multiset (ну и все с unordered_) - ассоциативные контейнеры, отображение в виде множества

---
### `std::vector`

Массив, который меняет размер по мере надобности

***Методы:***
- `.push_back()` - работает за $O(1)$ амортизированно
- `.pop_back()` 
- `.emplace_back()`
- `.front()`
- `.back()`

- `[]`
- `.at()` -  в отличие от `[]` бросает исключение `std::out_of_range`, тоже две версии для константного и неконстантного вектора. Если превзошли границу -> кинуть исключение. Но оператор `[]` работает быстрее.

- `.size()`
- `.capacity()`
- `.shrink_to_fit()` - размер вектора делает равным `size`
- `.resize()` - меняет размер, если принимает меньше, чем лежало - уничтожает

```cpp
template <class T, class Alloc = std::allocator<T>>
class MyVec {
public:
	void push_back(const T& x) {
		if (size < capacity) {
			traits::construct(alloc, arr + size, x);
			++size;
			return;
		}

		// выделение сырой памяти
		T* newArr = traits::allocate(alloc, capacity*2);
		
		// нужно вызывать конструкторы, а не операторы присваивания
		for (size_t i = 0; i < capacity; ++i) {
			// тут не очень правильная реализация
			// мб попозже  с помощью мув семантики
			traits::construct(alloc, newArr+i, arr[i]);
		}

		traits::construct(alloc, newArr+capacity, x);
		
		for (size_t i = 0; i < capacity; ++i) {
			traits::destroy(alloc, arr + i);
		}
		
		traits::deallocate(arr, capacity);		
		arr = newArr;
		capacity <<= 1; // *=2
		++size;
	}

	template <typename... Args>
	void emplace_back(const Args&... args) { ... }

	T& operator[](size_t i) {
		return arr[i];
	}
	// перегрузка для константного вектора
	const T& operator[](size_t i) const {
		return arr[i];
	}

	// конструктор копирования
	// ну реализовать его очев
	traits::select_on_container_copy_construction(alloc);
	// вот создали новый аллокатор
	// создаем новый массив, у нового аллокатора просим
	// аллоцировать память (allocate)
	// потом также construct и создаем копии всех элементов
	// из старого массива и все

	// деструктор:
	// сначала дестрой, потом деаллокейт
	

private:
	T* arr; // сам массив
	size_t size; // сколько используется
	size_t capacity; // сколько эл-тов выделено
	Alloc alloc;
	using traits = std::allocator_traits<Alloc>;
}
```

***Функция swap()***
Она меняет местами содержимое двух векторов и принимает в качестве параметра второй вектор (так быстрее, чем базовый `swap()`, который бы поменял местами сами векторы, а не их значения)


---

***Вопросы на понимание по вектору:***

Скока объем?

```cpp
vector<int> v(100);
v.shrink_to_fit;
sizeof(v);

// выведет 2*size_t + T
// т.е. суммарный размер полей
```

Чо будет?

```cpp
int main() {
	vector<int> v(100);
	delete[] &(v[0]);
}

// упадет конечно
// рантайм какой-нибудь или сигсегв
// произойдет двойное удаление по одному и 
// тому же указателю 
// (наш delete[], а потом деструктор)
```
