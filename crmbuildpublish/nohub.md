# 程序后台运行的方法


```bash

nohup ./run > log.log 2>&1 & 

```

参数说明：
./run : 你需要后台运行的程序
>: 日志文件追加到文件中
log.log: 运行的日志，或你的文件的输出内容

& 是一个描述符，如果1或2前不加&，会被当成一个普通文件。

1>&2 意思是把标准输出重定向到标准错误.

2>&1 意思是把标准错误输出重定向到标准输出。

&>filename 意思是把标准输出和标准错误输出都重定向到文件filename中