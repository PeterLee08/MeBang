
# File Operation:
-----

# C++

>最有意思的感觉应该是,  
[**想跟踪一段数据,但数据流要经过多个动态库,如何将数据流信息打印到一个文件,又不想大改动架构,增加参数,用log输出又增加了自己不想要的信息?**]()  
将全局变量cout 将cout的rdbuff绑定到指定文件, 然后不同模块间cout输出自己想要的信息(**cout.rdbuf(ofs.rdbuf())**),这样就可以很多事情可以不用去关心 --- cout是C++为数不多的全局变量

空白符制表符等分割数据一次读取,可迭代数据一次写入
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
