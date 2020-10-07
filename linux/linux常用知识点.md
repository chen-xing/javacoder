#linux常用知识
##网络抓包
>fconfig查看网卡 ip host 
tcpdump -i eth1  host hostname and port 80  -w /tmp/data.pcap
开源软件RawCap也可以抓到(下载地址:http://www.netresec.com/?page=RawCap).将抓到的包保存为pcap后缀,用wireshark打开,就可以继续分析了.
win7以管理员身份运行，ctrl+c停止后pcap文件才有内容。今天刚好用，开始不会ctrl+c也是这种情况

##压缩解压
>1.压缩命令：
　　命令格式：tar  -zcvf   压缩文件名.tar.gz   被压缩文件名
      可先切换到当前目录下。压缩文件名和被压缩文件名都可加入路径。
 
>2.解压缩命令：
　　命令格式：tar  -zxvf   压缩文件名.tar.gz
　　解压缩后的文件只能放在当前的目录。

##解压war包
>jar -xvf project.war
>需要说明的是可以指定解压到指定的目录，但是需要全路径，常用做法是把war包拷贝到当前目录后然后执行解压

