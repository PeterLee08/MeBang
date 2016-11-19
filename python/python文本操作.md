
# File Operation:
-----
## python
>实际使用python处理文本问题非常简单,但是觉得相对其他语言优势不明显,但是和pandas挂钩时问题就完全不一样了.先不说pandas

    with open('file') as f:
     for i in f.readlines():
        print(i)


**方法 描述**

* **f.close()**           关闭文件，记住用open()打开文件后一定要记得关闭它，否则会占用系统的可打开文件句柄数。
* **f.fileno()**           获得文件描述符，是一个数字
* **f.flush()**           刷新输出缓存
* **f.isatty()**           如果文件是一个交互终端，则返回True，否则返回False。
* **f.read([count])**        读出文件，如果有count，则读出count个字节。
* **f.readline()**         读出一行信息。
* **f.readlines()**         读出所有行，也就是读出整个文件的信息。
* **f.seek(offset[,where])**    把文件指针移动到相对于where的offset位置
* **f.tell()**              获得文件指针位置。
* **f.truncate([size])**        截取文件，使文件的大小为size。
* **f.write(string)**         把string字符串写入文件。
* **f.writelines(list)**         把list中的字符串一行一行地写入文件，是连续写入文件，没有换行。
