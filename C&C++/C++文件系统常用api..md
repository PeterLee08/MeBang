# 文件系统

2. C++路径操作
3. windows下dll路径问题

#C++

enum file_type { status_error, file_not_found, regular_file, directory_file,
                       symlink_file, block_file, character_file, fifo_file, socket_file,
                       type_unknown
                     };
namespace boost::filesystem

* **void         copy(const path& from, const path& to[, system::error_code& ec])**   //函数都有两种重载异常和错误码
* **bool         create_directories(const path& p);**      //windows下有bug,正确返回时未必系统准备就绪
* **void         copy_file(const path& from, const path& to);**  	//需要先建立文件夹
* **bool         exists(const path& p)**
* **uintmax_t    file_size(const path& p)**
* **bool         is_directory(file_status s)**
* **bool         is_empty(const path& p)**
* **bool         is_regular_file(file_status s)**
* **bool         is_symlink(const path& p)**
* **std::time_t  last_write_time(const path& p)**
* **bool         remove(const path& p)**
* **uintmax_t    remove_all(const path& p)**
* **path& remove_filename()**		//效率上有优势 much more efficient than *this = parent_path()
* **void copy_directory(const path& from, const path& to)**			// 不建议使用

**不常用                                                                       **
* **void swap(path& lhs, path& rhs)**			//交换文件内容
* **void         create_symlink(const path& to, const path& new_symlink)**
* **void         create_hard_link(const path& to, const path& new_hard_link)**
* **uintmax_t    hard_link_count(const path& p)**
* **void         resize_file(const path& p, uintmax_t size)**
* **file_status  status(const path& p)**
* **path         temp_directory_path()**		//环境变量决定

>遍历目录 模仿python
```c++
    for(boost::filesystem::recursive_directory_iterator end, dir("./); dir != end; ++dir){
      cout << *dir << endl;
    }
```

**路径名: **
* **class  boost::filesystem::path
* **template &lt;class String&gt;**
* **String string(const codecvt_type& cvt=codecvt()) const;**			// native format
* **const wstring       wstring(const codecvt_type& cvt=codecvt()) const**
* **const u16string     u16string() const;**
* **const u32string     u32string() const**
* **const value_type*   c_str() const;   // native().c_str()**

**路径操作:**

* **path  parent_path() const**
* **path  filename() const;**
* **path  stem() const;**
* **path  extension() const**
* **bool empty() const**

---

## windows 获取dll路径通用函数
>windows提供了获取exe路径的函数,能满足大部分获取可执行程序的需求,但当显示加载,或者跨语言调用时
可执行程序未必和动态库在同一位置

在知道明确句柄的情况下可以使用
GetMuleFileName

>在不知道句柄是该如何处理呢

此外推荐另外一种方式,根据字符常量地址获取dll路径
**VirtualQuery**和__VirtualQueryEx__
```C++
    HMODULE getDllModule(LPCVOID pcaller)
    {
      HMODULE  hmdl = null;
      MEMORY_BASIC_INFORMATION  mbi;
      if(VirtualQuery(pcaller, &mbi, sizeof(MEMORY_BASIC_INFORMATION)) ==  sizeof(MEMORY_BASIC_INFORMATION)  ){
        hmdl = (HMODULE)mbi.AllocationBase;
      }
      return hmdl;
    }
```
如果要获取dll名字(路径)
```C++
    CString getDllModuleName(LPCVOID pcaller){
      HMODULE hmdl = getDllModule(pcaller);
      if(!hmdl) return "";
      const int max_size = 512;
      TCHAR tzPath[max_size] = {0};
      ::GetModuleFileName(hmdl, tzPath, max_size);

      return tzPath;
    }
```

另外一种方式:
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