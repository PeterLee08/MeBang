# C++
## 1. C++ SFINAE
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
## 2. C++ 完美转发
```c++
    void fun(int &x) { cout << "lvalue ref" << endl; }  
    void fun(int &&x) { cout << "rvalue ref" << endl; }  
    void fun(const int &x) { cout << "const lvalue ref" << endl; }  
    void fun(const int &&x) { cout << "const rvalue ref" << endl; }  
    
    template<typename T>  
    void PerfectForward(T &&t) { fun(std::forward<T>(t)); }  
```

## 3. C++ windll 
**获取dll地址**:
```C++
    CString getDllAddress(){  
        HMODULE handle = NULL;  
        BOOL bret = GetModuleHandleEx(GET_MODULE_HANDLE_EX_FLAG_FROM_ADDRESS,  
            _T("thismodule"),   //该dll上任意地址，这里取个字符常量或者弄个本地函数地址;  
            &handle  
            );  
        if (!bret) return _T("");  
    
        TCHAR  szBuffer[MAX_PATH] = { 0 };  
        GetModuleFileName(handle, szBuffer, sizeof(szBuffer));  
        return szBuffer;  
    }  
```
## 4. C++ 文本操作
**全局cout**
cout.rdbuf(ofs.rdbuf())

**文本**
```c++
    std::ifstream ifs("d:/text.txt");
    list<int> data(istream_iterator<int>{ifs}, istream_iterator<int>());

    std::ofstream ofs("d:/text1.txt");
    std::copy(begin(data), end(data), ostream_iterator<int>{ofs,"delimeter"});


    std::ifstream t("file.txt");

    //一次读入string 
    //包括空白符 
    std::string str((std::istreambuf_iterator<char>(t)), std::istreambuf_iterator<char>()); 
    //跳过空白符
    string fileData((istream_iterator<char>(inputFile)), istream_iterator<char>());
    //一次读入缓冲区
    char buf[1024] = { 0 };
	ifs.read(buf, 1024);
    //一次直接写出
    std::ofstream ofs("d:/text1.txt");  
    ofs << t.rdbuf();  
```   
** 变参模板 **
** 变参输入**
如下可以输入一行各种长度,各种类型数据(重载<<)
```C++
    template<typename T>
    std::ostream& write_line(std::ostream& os, const std::string& delimitor, const T& t)
    {
        return os << t << 'n';    
    }

    template<typename T, typename ... Args>
    std::ostream& write_line(std::ostream& os, const std::string& delimitor, const T& t, const Args ... args)
    {
        os << t << delimitor;
        return write_line(os, delimitor, args...);
    }
```
然后就可以像这样调用了
```C++
    std::ofstream ofs('/tmp/test.tab')
    if (ofs) write_line(ofs, "\t", 1, 3, "sfe", "and so on")
```

## 5.C++ 捕获异常
```c++
      try{
       //...
      }
      //MFC 异常均继承于CEXcep,而且异常都以指针形式抛出,不允许直接delete,需调用成员方法删除
      catch(CException * e)
      {
        TCHAR err[1024] = {0};
        e->GetErrorMessage(err, 1024);
        //do something with err
        e->Delete()
      }
      //boost 异常信息(具体tag一般很深因此使用diagnostic感觉更合理)
      catch (boost::exception& e){
        err = boost::diagnostic_information(e);
      }
      //标准异常均继承于std::excpetion
      catch(std::exception& e)
      {
        std::string err = e.what();
        //do something with err
      }
      catch(...)
      {
        err = "unknown error";
      }
```
## 6. ｃ＋＋　比如trace函数
```c++
    #define  __INFO_TRACE(x, y)  TraceFunction  __info##y(x)
    #define  _INFO_TRACE(x)      __INFO_TRACE(x, __LINE__)
    #define  INFO_TRACE          _INFO_TRACE(__FUCTION__)
```

## 7. c++11 并发

>std::future <-- std:promise|std::package_task|std::asyn

```c++
    std::promise<int> pr;
    std::thread t([](std::promise<int>& p){ p.set_value_at_thread_exit(9); },std::ref(pr));
    std::future<int> f = pr.get_future();
    auto r = f.get();
```
```c++
    std::packaged_task<int()> task([](){ return 7; });
    std::thread t1(std::ref(task)); 
    std::future<int> f1 = task.get_future(); 
    auto r1 = f1.get();
```
```c++
std::future<int> future = std::async(std::launch::async, [](){ 
        std::this_thread::sleep_for(std::chrono::seconds(3));
        return 8;  
    });
```

# java
## 1. java 反射　reflection
```java
Class cls = Class.forName("com.test.reflect.Student");  
Method m = cls.getDeclaredMethod("func",new Class[]{int.class,String.class});  
m.invoke(cls.newInstance(),20,"chb");
// or: if m is static  
m.invoke(cls,20,"chb");
```

## 2. java 注解　Annotation
**simple**
```java 
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@interface Todo {
public enum Priority {LOW, MEDIUM, HIGH}
public enum Status {STARTED, NOT_STARTED}
String author() default "Yash";
Priority priority() default Priority.LOW;
Status status() default Status.NOT_STARTED;
}

///
@Todo(priority = Todo.Priority.MEDIUM, author = "Yashwant", status = Todo.Status.STARTED)
public void incompleteMethod1() {
//Some business logic is written
//But it’s not complete yet
}

///
Class businessLogicClass = BusinessLogic.class;
for(Method method : businessLogicClass.getMethods()) {
Todo todoAnnotation = (Todo)method.getAnnotation(Todo.class);
if(todoAnnotation != null) {
System.out.println(" Method Name : " + method.getName());
System.out.println(" Author : " + todoAnnotation.author());
System.out.println(" Priority : " + todoAnnotation.priority());
System.out.println(" Status : " + todoAnnotation.status());
}
}
```
complicate
```java

public class TableCreator
{
    public static void main(String[] args) throws Exception
    {
        if (args.length < 1)
        {
            System.out.println("arguments: annotated classes");
            System.exit(0);
        }
        for (String className : args)
        {
            Class<?> cl = Class.forName(className);
            DBTable dbTable = cl.getAnnotation(DBTable.class);
            if (dbTable == null)
            {
                System.out.println("No DBTable annotations in class " + className);
                continue;
            }
            String tableName = dbTable.name();
            // If the name is empty, use the Class name:
            if (tableName.length() < 1)
                tableName = cl.getName().toUpperCase();
            List<String> columnDefs = new ArrayList<String>();
            for (Field field : cl.getDeclaredFields())
            {
                String columnName = null;
                Annotation[] anns = field.getDeclaredAnnotations();
                if (anns.length < 1)
                    continue; // Not a db table column
                if (anns[0] instanceof SQLInteger)
                {
                    SQLInteger sInt = (SQLInteger) anns[0];
                    // Use field name if name not specified
                    if (sInt.name().length() < 1)
                        columnName = field.getName().toUpperCase();
                    else
                        columnName = sInt.name();
                    columnDefs.add(columnName + " INT" + getConstraints(sInt.constraints()));
                }
                if (anns[0] instanceof SQLString)
                {
                    SQLString sString = (SQLString) anns[0];
                    // Use field name if name not specified.
                    if (sString.name().length() < 1)
                        columnName = field.getName().toUpperCase();
                    else
                        columnName = sString.name();
                    columnDefs.add(columnName + " VARCHAR(" + sString.value() + ")" + getConstraints(sString.constraints()));
                }
                StringBuilder createCommand = new StringBuilder("CREATE TABLE " + tableName + "(");
                for (String columnDef : columnDefs)
                    createCommand.append("\n    " + columnDef + ",");
                // Remove trailing comma
                String tableCreate = createCommand.substring(0, createCommand.length() - 1) + ");";
                System.out.println("Table Creation SQL for " + className + " is :\n" + tableCreate);
            }
        }
    }

    private static String getConstraints(Constraints con)
    {
        String constraints = "";
        if (!con.allowNull())
            constraints += " NOT NULL";
        if (con.primaryKey())
            constraints += " PRIMARY KEY";
        if (con.unique())
            constraints += " UNIQUE";
        return constraints;
    }
}

/*
   * Output: Table Creation SQL for annotations.database.Member is : CREATE
   * TABLE MEMBER( FIRSTNAME VARCHAR(30)); Table Creation SQL for
   * annotations.database.Member is : CREATE TABLE MEMBER( FIRSTNAME
   * VARCHAR(30), LASTNAME VARCHAR(50)); Table Creation SQL for
   * annotations.database.Member is : CREATE TABLE MEMBER( FIRSTNAME
   * VARCHAR(30), LASTNAME VARCHAR(50), AGE INT); Table Creation SQL for
   * annotations.database.Member is : CREATE TABLE MEMBER( FIRSTNAME
   * VARCHAR(30), LASTNAME VARCHAR(50), AGE INT, HANDLE VARCHAR(30) PRIMARY
   * KEY);
   */
```