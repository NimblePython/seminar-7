# Повторение static_cast, reinterpret_cast и dynamic_cast

Продолжаем тему операторов преобразования типов в языке C++ и рассмотрим оператор `static_cast`. Он является аналогом оператора `reinterpret_cast`, но накладывает более строгие ограничения. Лучший способ понять различия и возможности `static_cast` — это рассмотреть конкретные примеры.

Пусть в программе объявлены переменные различных типов и указатели на них:

```cpp
int main() {
    char ch {0};
    short sh {0};
    int i {0};
    double d {0};

    char* ptr_ch {&ch};
    short* ptr_sh {&sh};
    int* ptr_i {&i};
    double* ptr_d {&d};

    ch = static_cast<char>(sh);    // Преобразование допустимо
    ch = reinterpret_cast<char>(sh); // Ошибка

    d = static_cast<double>(i);    // Преобразование допустимо
    d = reinterpret_cast<double>(i); // Ошибка

    sh = static_cast<short>(ptr_i);  // Ошибка 
    sh = reinterpret_cast<short>(ptr_i);  // Допустимо

    ptr_d = static_cast<double *>(ptr_ch);  // Ошибка 
    ptr_d = reinterpret_cast<double *>(ptr_ch);  // Допустимо

    return 0;
}
```

Как видно, оператор `static_cast` позволяет выполнять преобразования типов между переменными, в отличие от `reinterpret_cast`. Однако при работе с указателями ситуация иная. С помощью `static_cast` нельзя преобразовывать указатели к обычным переменным (и наоборот), а также преобразовывать указатели на несвязанные между собой типы. В отличие от этого, `reinterpret_cast` может выполнять такие преобразования.

Что значит "несвязанные типы указателей"? Рассмотрим пример с двумя независимыми структурами:

```cpp
struct point2D {
    int x, y;
};

struct point3D {
    int x, y, z;
};
```

С точки зрения оператора `static_cast`, эти типы являются независимыми. Соответственно, приведение одного типа к другому невозможно:

```cpp
int main() {
    point2D* ptr_2d = new point2D {1, 2};
    
    point3D* ptr_3d = static_cast<point3D *>(ptr_2d);  // Ошибка
    point3D* ptr_3d_2 = reinterpret_cast<point3D *>(ptr_2d);  // Допустимо
    
    delete ptr_2d;
    return 0;
}
```

Однако, если одна структура будет наследоваться от другой:

```cpp
struct point3D : point2D { };
```

Ошибка исчезнет. Это показывает, что `static_cast` требует наличия связи между типами. Даже если цепочка наследования будет длинной:

```cpp
struct point { };

struct point2D : point {
    int x, y;
};

struct point3D : point2D {
    int x, y, z;
};
```

Оператор `static_cast` продолжит отслеживать взаимосвязь между типами:

```cpp
int main() {
    point* ptr_pt = new point;

    point3D* ptr_3d = static_cast<point3D *>(ptr_pt);  // Ошибка
    point3D* ptr_3d_2 = reinterpret_cast<point3D *>(ptr_pt);  // Допустимо
    
    delete ptr_pt;
    return 0;
}
```

Также можно выполнить обратное преобразование от дочернего класса к базовому:

```cpp
int main() {
    point3D* ptr_3d = new point3D;

    point* ptr_pt = static_cast<point *>(ptr_3d);  // Допустимо
    point* ptr_pt_2 = reinterpret_cast<point *>(ptr_3d);  // Допустимо

    delete ptr_3d;
    return 0;
}
```

Однако в обратном преобразовании с `static_cast` часто нет необходимости, так как компилятор способен выполнить его автоматически.

## Оператор `dynamic_cast`

Все рассмотренные операторы приведения типов — `const_cast`, `reinterpret_cast`, `static_cast` — работают на этапе компиляции, и их относят к статическим операторам. В отличие от них, оператор `dynamic_cast` выполняет приведение типов во время выполнения программы.

Пример, где необходим оператор `dynamic_cast`, связан с использованием полиморфизма и наследования:

```cpp
class Thing {
public:
    virtual void print() const { }
};

class Ball : public Thing {
    int radius, color;
public:
    virtual void print() const override { puts("Ball"); }
    void get_data(int& r, int& c) const { r = radius; c = color; }
};

class Mouse : public Thing {
public:
    virtual void print() const override { puts("Mouse"); }
};
```

Допустим, у нас есть тележка для товаров:

```cpp
class Cart {
public:
    void add_thing(const Thing& th) {
        th.print();

        const Ball* ptr_ball = dynamic_cast<const Ball*>(&th);

        if (ptr_ball) {
            int radius, color;
            ptr_ball->get_data(radius, color);
            puts("th is a Ball");
        } else {
            puts("th is not a Ball");
        }
    }
};
```

В метод `add_thing` можно передавать объекты разных типов, и в ходе выполнения программы проверяется, принадлежит ли объект классу `Ball`.

Пример работы программы:

```cpp
int main() {
    Cart cr;

    Ball b;
    Mouse m;

    cr.add_thing(b);
    cr.add_thing(m);

    return 0;
}
```

Результат:

```bs
Ball
th is a Ball
Mouse
th is not a Ball
```

`dynamic_cast` работает только с указателями и требует наличия хотя бы одного виртуального метода в классе для использования механизма динамической идентификации типа (RTTI). 

На практике использование `dynamic_cast` желательно минимизировать, так как оно может указывать на недостатки в архитектуре программы.
