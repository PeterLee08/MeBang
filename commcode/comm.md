# C++
## 1. SFINAE
**exampe 1: 通配不同类型处理**
```c++
    
    struct Test {
        typedef int foo;
    };

    template <typename T> 
    void f(typename T::foo) {} // Definition #1

    template <typename T> 
    void f(T) {}               // Definition #2

    int main() {
        f<Test>(10); // Call #1.
        f<int>(10);  // Call #2. Without error (even though there is no int::foo) thanks to SFINAE.
    }
```
**example 2:SFINAE can be used to determine if a type contains a certain typedef**
```c++

    #include <iostream>

    template <typename T>
    struct has_typedef_foobar {
        // Types "yes" and "no" are guaranteed to have different sizes,
        // specifically sizeof(yes) == 1 and sizeof(no) == 2.
        typedef char yes[1];
        typedef char no[2];

        template <typename C>
        static yes& test(typename C::foobar*);

        template <typename>
        static no& test(...);

        // If the "sizeof" of the result of calling test<T>(nullptr) is equal to sizeof(yes),
        // the first overload worked and T has a nested type named foobar.
        static const bool value = sizeof(test<T>(nullptr)) == sizeof(yes);
    };

    struct foo {    
        typedef float foobar;
    };

    int main() {
        std::cout << std::boolalpha;
        std::cout << has_typedef_foobar<int>::value << std::endl;
        std::cout << has_typedef_foobar<foo>::value << std::endl;
    }

```
or in c++11
```c++
    #include <iostream>
    #include <type_traits>

    template <typename... Ts> using void_t = void;

    template <typename T, typename = void>
    struct has_typedef_foobar : std::false_type {};

    template <typename T>
    struct has_typedef_foobar<T, void_t<typename T::foobar>> : std::true_type {};

    struct foo {
    using foobar = float;
    };

    int main() {
    std::cout << std::boolalpha;
    std::cout << has_typedef_foobar<int>::value << std::endl;
    std::cout << has_typedef_foobar<foo>::value << std::endl;
    }
```
or in c++17
```c++
    #include <iostream>
    #include <type_traits>

    template <typename T>
    using has_typedef_foobar_t = decltype(T::foobar);

    struct foo {
    using foobar = float;
    };

    int main() {
    std::cout << std::boolalpha;
    std::cout << std::is_detected<has_typedef_foobar_t, int>::value << std::endl;
    std::cout << std::is_detected<has_typedef_foobar_t, foo>::value << std::endl;
    }
```

**example 3: 判断是否包含属性**
```c++
    
    class A {
    public:
            std::string toString() const {
                    return std::string("toString from class A");
            }
    };

    class B {
    };

    template<typename T>
    struct HasToStringFunction {
            template<typename U, std::string (U::*)() const >
            struct matcher;

            template<typename U>
            static char helper(matcher<U, &U::toString> *);

            template<typename U>
            static int helper(...);

            enum { value = sizeof(helper<T>(NULL)) == 1 };
    };

    template <bool>
    struct ToStringWrapper {};

    template<>
    struct ToStringWrapper<true> {
            template<typename T>
            static std::string toString(T &x) {
                    return x.toString();
            }
    };

    template<>
    struct ToStringWrapper<false> {
            template<typename T>
            static std::string toString(T &x) {
                    return std::string(typeid(x).name());
            }
    };

    template<typename T>
    std::string toString(const T &x) {
            return ToStringWrapper<HasToStringFunction<T>::value>::toString(x);
    }

    int main() {
            A a;
            B b;

            std::cout << toString(a) << std::endl;
            std::cout << toString(b) << std::endl;
    }
```
**example 4: isclass**
```c++
    template <typename T>
    class is_class {
        template <typename U>
        static char helper(int U::*);
        template <typename U>
        static int helper(...);
    public:
        static const bool value = sizeof(helper<T>(0)) == 1;
    };
```
**example 5: isbaseof**
```c++
    template <typename T1, typename T2>
    struct is_same {
        static const bool value = false;
    };

    template <typename T>
    struct is_same<T, T> {
        static const bool value = true;
    };

    template<typename Base, typename Derived, bool = (is_class<Base>::value && is_class<Derived>::value)>
    class is_base_of {
        template <typename T>
        static char helper(Derived, T);
        static int helper(Base, int);
        struct Conv {
            operator Derived();
            operator Base() const;
        };
    public:
        static const bool value = sizeof(helper(Conv(), 0)) == 1;
    };

    template <typename Base, typename Derived>
    class is_base_of<Base, Derived, false> {
    public:
        static const bool value = is_same<Base, Derived>::value;
    };

    template <typename Base>
    class is_base_of<Base, Base, true> {
    public:
        static const bool value = true;
    };
```
## 2.完美转发
```c++
    void fun(int &x) { cout << "lvalue ref" << endl; }  
    void fun(int &&x) { cout << "rvalue ref" << endl; }  
    void fun(const int &x) { cout << "const lvalue ref" << endl; }  
    void fun(const int &&x) { cout << "const rvalue ref" << endl; }  
    
    template<typename T>  
    void PerfectForward(T &&t) { fun(std::forward<T>(t)); }  
```
