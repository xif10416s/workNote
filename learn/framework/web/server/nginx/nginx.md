##启动　　

cd usr/local/nginx/sbin
./nginx

##重启

　　更改配置重启nginx　　

kill -HUP 主进程号或进程号文件路径
或者使用
cd /usr/local/nginx/sbin
./nginx -s reload

判断配置文件是否正确　

nginx -t -c /usr/local/nginx/conf/nginx.conf
或者
cd  /usr/local/nginx/sbin
./nginx -t

##关闭

　　查询nginx主进程号

　　ps -ef | grep nginx

　　从容停止   kill -QUIT 主进程号

　　快速停止   kill -TERM 主进程号

　　强制停止   kill -9 nginx

　　若nginx.conf配置了pid文件路径，如果没有，则在logs目录下

　　kill -信号类型 '/usr/local/nginx/logs/nginx.pid'



