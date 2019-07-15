# Polymorphism

```cpp
#include <iostream>

#include <utility>
#include <memory>

struct vtable {
    int  (*get_x  ) (void* This);
    int  (*get_y  ) (void* This);
    int  (*get_z  ) (void* This);

    void (*deleter) (void* This);
};

template <class type>
vtable const get_instance_of = {
    [](void* This) { return ((type*) This)->get_x(); },
    [](void* This) { return ((type*) This)->get_y(); },
    [](void* This) { return ((type*) This)->get_z(); },
    [](void* This) { delete ((type*) This); }
};

struct point {
    int get_x() const { return *x * 3; }
    int get_y() const { return *y * 2; }
    int get_z() const { return *z * 4; }

    std::unique_ptr <int> x, y, z;

    point(int x, int y, int z) : x(new int(x)), y(new int(y)), z(new int(z)) {}
    point(int val) : point(val, val, val) {}

    point(point&& point_m) : x(std::move(point_m.x)), y(std::move(point_m.y)), z(std::move(point_m.z)) {}

    ~point() {
        x.reset(nullptr);
        y.reset(nullptr);
        z.reset(nullptr);
    }
};

struct base {
    vtable const * caller;
    void* This;

    template <class type>
    base(type object)
        : This(new type(std::move(object))), caller(&get_instance_of <type>) {}

    base(base &&object)
        : This(std::exchange(object.This, nullptr)), caller(std::exchange(object.caller, nullptr)) {}

    decltype(auto) get_x() const { return caller->get_x(This); }
    decltype(auto) get_y() const { return caller->get_y(This); }
    decltype(auto) get_z() const { return caller->get_z(This); }

    ~base() {
        if (This) {
            caller->deleter(This);
        }
    }
};

int main() {
    base element0(point{ 1, 2, 3 });
    base element1(std::move(element0));

    std::cout << element1.get_x() << '\n';
    std::cout << element1.get_y() << '\n';
    std::cout << element1.get_z() << '\n';
    return 0;
}
```
