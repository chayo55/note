liunx常用命令：
cat more等查看文件信息以及文件内容
mv cp rm操作文件，移动复制删除
vim编辑软件，dd快捷删除行，yy复制当前行，nyy复制当前开始的n行，p粘贴，yyp复制当前行到下一行
tar解压软件，tar -xvf file.conf /root  参数x解压，c为压缩，v为输出解压缩过程，f指定文件
| 管道命令，将前面命令的结果传给后面命令作为输入
ps  process status 进程状态命令，查看 -e参数等同于-A，显示全部进程，-f显示程序间关系，结合管道命令 使用grep文本搜索命令过滤
top 查看详细进程信息，时间等实时监控
kill 命令杀死命令，9 为强制杀死
chmod 改变文件的权限
chown 改变文件的所有者
network 网络管理
systemctl 服务管理器指令 管理各软件服务，systemctl start mariadb，在centos7之前是service start mariadb.service
redis-server redis.conf 启动redis服务实例
redis-cli -p 6379 , redis客户端连接
start nginx 启动nginx