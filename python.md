python

其他实用方法

* ` os.access(path, mode)   判断文件操作权限`

关于输入时的 input和raw\_input也非常有意思

* _`raw_input`  直接返回输入字符串`

* `input`    和 `raw_input` 不同的是如果输入语句是一个合法python语句则input将会执行返回新的结果

        #!/usr/bin/python
        str = input("Enter your input: ");
        print "Received input is : ", str`

        Enter your input: [x*5 for x in range(2,10,2)]
        Recieved input is :  [10, 20, 30, 40]

python 实现__enter__和__exit__后使用with比try finally优雅的多
