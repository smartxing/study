## cat 查看文件的内容
    cat命令是一个显示文本文件内容的便捷工具，如果一个日志文件比较小，可
    以直接使用cat命令将其内容打印出来，进行查看，但是，对于较大的日志文
    件，请不要这样做，打开一个过大的文件可能会占用过多的系统资源，从而
    影响系统对外的服务。
## more 分页显示文件
    cat的缺点在于，一旦执行后，便无法再进行交互和控制，而more命令可以分
    页的展现文件内容，按enter键显示文件下一行，按空格键便显示下一页，按
    f键显示下一屏内容，按b键显示上一屏内容。
    另一个命令less提供比more更加丰富的功能，支持内容查找，并且能够高亮
    显示。
## tail 显示文件尾
    使用tail命令能够查看到文件最后几行，这对于日志文件非常有效，因为日
    志文件常常是追加写入的，新写入的内容处于文件的末尾位置。
## head 显示文件头
    与tail命令类似，但是不同的是head命令用于显示文件开头的一组行
## sort 内容排序
    一个文件中包含有众多的行，经常需要对这些行中的某一列进行排序操作，
    sort命令的作用便是对数据进行排序。
    sort -n 数值排序 -r 降序 -k 2 第二列 -t :设定间隔符号 demo.txt

## wc 字符统计
    wc命令可以用来统计指定文件中的字符数，字数，行数，并输出统计结果。
## uniq 查看重复出现的行
    uniq命令可以用来显示文件中行重复的次数，或者显示仅出现一次的行，以
    及仅仅显示重复出现的行，并且，uniq的去重针对的只是连续的两行，因此
    它常常与sort结合起来使用。
## curl URL访问工具
    要想在命令行下通过HTTP协议访问网页文档，就不得不用到一个工具，这便
    是curl，它支持HTTP，HTTPS，FTP，FTPS，Telnet等多种协议，常被用来在
    命令行下抓取网页和监控WEB服务器状态。
## 查看端口被占用情况
    lsof -i :8080/netstat -anp|grep port  查看端口被占用

## sed 
    删除某行
    sed '1d' ab              #删除第一行 
    sed '$d' ab              #删除最后一行
    sed '1,2d' ab           #删除第一行到第二行
    sed '2,$d' ab           #删除第二行到最后一行

    显示某行
    sed -n '1p' ab           #显示第一行 
    sed -n '$p' ab           #显示最后一行
    sed -n '1,2p' ab        #显示第一行到第二行
    sed -n '2,$p' ab        #显示第二行到最后一行

    使用模式进行查询
    sed -n '/ruby/p' ab    #查询包括关键字ruby所在所有行
    sed -n '/\$/p' ab        #查询包括关键字$所在所有行，使用反斜线\屏蔽特殊含义
    sed -i '/匹配字符串/d'  filename  （注：若匹配字符串是变量，则需要“”，而不是‘’。记得好像是）

    替换匹配行中的某个字符串
    sed -i '/匹配字符串/s/替换源字符串/替换目标字符串/g' filename
## cut 
    cut -f1 -d " " 根据 " " 切分 取第一列
## awk
    awk '{print $1}' demo.log | head -10 打印文件指定列  
    可以参考[手册](http://www.itshouce.com.cn/linux/linux-awk.html)      

##  系统情况
```html
load: top ,uptime
cpu: top
磁盘: df -h
网卡流量：sar -n DEV 1 4
io状况： iostat -d -k 2
内存：free -m
bug级别命令 
vmstat 2 1 CPU使用率，内存使用，虚拟内存交换情况,IO读写情况

```

## 最长用的shell命令
cat demo.log | cut -f1 -d " " | sort | uniq -c | sort -k 1 -n -r | head -10

在指定时间内kill服务
```
#!/bin/bash
xxx_stop() {
    count=`ps -ef |grep xxxxxx |grep -v "grep" |wc -l`
    if [ $count -gt 0 ];then
            pid_old=`ps -ef | grep xxxxxx|grep java | grep -v "grep" |awk '{print $2}'`
            ps -ef | grep xxxxxx  |grep java| grep -v "grep" | kill -15 `awk '{print $2}'`
            timer=100
            while [ $count -gt 0 ]
            do
                    echo '任务正在进行中请稍后... timer'  $timer  'taskcount:' $count
                    pid_new=`ps -ef | grep xxxxxx |grep java | grep -v "grep" |awk '{print $2}'`
                    let timer=timer-1;
                    sleep 3
                    if [[ $timer -lt 1 ]] || [[ $pid_old -ne $pid_new ]];then
                            echo '任务结束超时，强制关闭任务'+ $timer
                            pid=`ps -ef | grep xxxxxx |grep java| grep -v "grep" | awk '{print $2}'`
                            kill -9 $pid
                            break;
                    fi
                    count=`ps -ef |grep xxxxxx |grep java |grep -v "grep" |wc -l`
            done
            echo '任务正常退出....'
    else
        echo '任务不存在，无需kill'
    fi
}
```

