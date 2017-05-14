# 文件系统

1. 脚本路径操作



**常用的遍历获取文件大小**

* **os.listdir(path)** 　　　　　 返回指定目录下的所有文件和目录名
* **os.path.getsize(name)** 　　 获得文件大小，如果name是目录返回0
* **os.path.isfile()** 和 **os.path.isdir()**    判断是否为文件
* **os.path.dirname(path)**　　　 截取最后一个文件分隔符/or,需谨慎

```python
    #python 遍历目录
    for i  in os.listdir(path):
        print (i)
```

* **shutil.rmtree(path[, ignore_errors[, onerror])**
* **shutil.copyfile(src, dst)**
* **shutil.move(src, dst)**
* **shutil.copytree(olddir,newdir** [,True/Flase符号链接是否还是连接否则是副本])


*使用相对低频目录的创建删除切换*

* **os.remove(path)** 　　　　　 函数用来删除一个文件
* **os.rmdir('dirname')** 　　　　    建议替换 shutil.rmtree
* **os.mkdir("newdir")**     　　  　　 建议替换os.makedirs
* **os.makedirs(path)**
* **os.rename(current_file_name, new_file_name)**
* **os.chdir("newdir")**
* **os.getcwd()**
* **os.walk(path)**　　　　　　　 在取当前下的子目录和文件分类时比较有用

```python
    for i in os.walk(r'F:topdir'):
       print (i)
```
    ('F:topdir', ['chdir1', 'chdir2'], ['level.txt'])
    ('F:topdirchdir1', [], ['leve2.txt'])
    ('F:topdirchdir2', [], [])
