```C++
内存连续容器:
c.erase(remove(c.begin(), c.end(), 1963), c.end());
c.erase(remove_if(c.begin(), c.end(), badValue), c.end());
关联容器
c.erase(1963)
```


利用move(强制类型转换)forward(根据类型转换)可做成完美转发模板

虚继承内存对象会存在多个虚表指针,增加对象内存和函数跳转复杂性,能避免就避免

1. 不用临时变量交换两个数
```C++
void swap(int *a,int *b){     
    (*a)^=(*b)^=(*a)^=(*b);   
} 
``` 
即：
```C++
a ^= b;  
b ^= a;  
a ^= b;  
```
2. 乘以2的m次方
```C++
int mulTwoPower(int n,int m){//计算n*(2^m)  
    return n<<m;  
} 
``` 

3. 判断一个数的奇偶性
```C++
bool isOddNumber(int n){  
    return (n & 1) == 1;  
}  
```

4. 判断一个数是不是2的幂
```C++
bool isFactorialofTwo(int n){  
    return (n & (n - 1)) == 0;   
} 
```

5. 求两个整数的平均值
```C++
int getAverage(int x, int y){  
        return (x+y) >> 1;   
｝
```

条款31：了解你的排序选择


windows下每个dll都有自己的申请动态内存方式(尤其版本间),要在哪个dll申请哪个dll释放,否则会造成意想不到的麻烦

如果dll较多,由于dll加载的对齐方式可能启动很慢,这是最好使用工具重新调整dll的加载地址

