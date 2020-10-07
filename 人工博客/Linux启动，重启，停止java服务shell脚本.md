# Linux启动，重启，停止java服务shell脚本

### 1、脚本代码

```
#!/bin/bash
app='web-0.0.1-SNAPSHOT.jar'
cmd=$1
pid=`ps -ef|grep java|grep $app|awk '{print $2}'`

startup(){
  nohup java -Xms384m -Xmx384m  -jar web-0.0.1-SNAPSHOT.jar --server.port=8001  >nohup.out &
  tail -f blog-admin.out
}

if [ ! $cmd ]; then
  echo "Please specify args 'start|restart|stop'"
  exit
fi

if [ $cmd == 'start' ]; then
  if [ ! $pid ]; then
    startup
  else
    echo "$app is running! pid=$pid"
  fi
fi

if [ $cmd == 'restart' ]; then
  if [ $pid ]
    then
      echo "$pid will be killed after 3 seconds!"
      sleep 3
      kill -9 $pid
  fi
  startup
fi

if [ $cmd == 'stop' ]; then
  if [ $pid ]; then
    echo "$pid will be killed after 3 seconds!"
    sleep 3
    kill -9 $pid
  fi
  echo "$app is stopped"
fi
```



### 2、使用方式

+ 修改app的名称以及startup的启动命令
+ 执行`chmod +x server.sh`添加运行权限
+ 执行`./server.sh start`或者`./server.sh restart`即可启动项目
+ 执行`./server.sh stop`停止项目运行