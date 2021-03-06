# 0 总体说明
本文档主要说明搭建ProTrade量化系统所要做的前期工作.
本说明分为以下几部分:

1. nginx安装及使用
2. Tornado安装及使用
3. supervior使用
4. MongoDB安装及使用

ProTrade系统启动项:

1. nginx
sudo nginx -c /Users/xavierxia/Desktop/XavierXia/pro_code/python/ProTrade/conf/nginx.conf
2. supervior
3. MongoDB
4. tushare



# 1 nginx
## 安装brew
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

若出现安装不上的情况，如：

Error: Could not link: /usr/local/share/doc/homebrew
就执行: sudo rm -rf /usr/local/share/doc/homebrew

### brew 升级
	sudo brew update

### Mac安装nignx
	brew search nginx
	brew install nginx

### 或者还有其他的方法安装Nginx
#### 下载 nginx,pcre,zlib库

1. ./configure --sbin-path=/usr/local/nginx/nginx --conf-path=/usr/local/nginx/nginx.conf --pid-path=/usr/local/nginx/nginx.pid --with-http_ssl_module --with-pcre=../pcre-8.38 --with-zlib=../zlib-1.2.8 --with-openssl=../openssl-1.0.2g
2. make -j8 && sudo make install
3. sudo ln -s /usr/local/nginx/sbin/nginx /usr/sbin/nginx

### 安装nginx之后的说明
#### 安装完以后，可以在终端输出的信息里看到一些配置路径：

	/usr/local/etc/nginx/nginx.conf （配置文件路径）
	/usr/local/var/www （服务器默认路径）
	/usr/local/Cellar/nginx/ （安装路径）

访问localhost:8080，成功说明安装好了

### nginx备注
	ln -s  /usr/local/sbin/nginx /usr/bin/nginx 做个软连接
### 常用的指令有： 
	nginx -V 查看版本，以及配置文件地址
	nginx -v 查看版本
	nginx -c filename 指定配置文件
	nginx -h 帮助

### nginx简单命令
#### 重新加载配置|重启|停止|退出 nginx
	nginx -s reload|reopen|stop|quit
### 打开 nginx
	nginx
### 使用指定的配置文件启动nginx
	sudo nginx -c /Users/xavierxia/Desktop/XavierXia/pro_code/python/ProTrade/conf/nginx.conf
### 测试配置是否有语法错误
	nginx -t 测试nginx.conf配置
###例如
	sudo nginx -s quit -c /Users/xavierxia/Desktop/XavierXia/pro_code/python/ProTrade/conf/nginx.conf


# 2 Tornado

## 安装
	pip install tornado

## 问题1
ImportError: No module named tornado.httpserver
解决:重新安装python2.7导致了需要重新安装tornado

# 3 supervior
## 用supervisor来管理服务和进程
###安装
	sudo pip install supervisor

###在 Mac环境中对supervior进行如下设置
#### 1. 转换成root权限
	sudo -i
	echo_supervisord_conf > /etc/supervisord.conf
#### 2. 修复/etc/supervisord.conf
	[include]
	files = /etc/supervisord.d/*.ini
#### 3. 创建目录
	mkdir -p /etc/supervisord.d
#### 4. 将进程管理的脚本(ProTrade里面的tornado.ini文件)放到supervisord.d
#### 5. supervisord 启动
	sudo /Users/xavierxia/anaconda2/bin/supervisord -c /etc/supervisord.conf
	sudo supervisord -c /etc/supervisord.conf
将在.bash_profile supervisord的命令路径即可: sudo supervisord -c /etc/supervisord.conf 

#### 6. 查看进程及相关操作
	sudo supervisorctl
#### 进程关闭
	supervisor> stop tornadoes:tornado-8000
	或者
	sudo supervisorctl stop all

#### 进程开启
	supervisor> start tornadoes:tornado-8000
	或者
	sudo supervisorctl start all

	supervisorctl status
	supervisorctl update

#### 查看程序状态
	supervisor> status
#### 查看该程序的日志
	supervisor> tail -f tornadoes:tornado-8000

#### 问题1
	Error: Another program is already listening on a port that one of 	our HTTP servers is configured to use.  Shut this program down 	first before starting supervisord.
	解决办法：
	sudo unlink /tmp/supervisor.sock
	sudo unlink /var/run/supervisor.sock
#### 问题2
	tornadoes:tornado-8000   FATAL  unknown error making dispatchers for 'tornado-8000': EACCES
	supervisor> start tornadoes:tornado-8000
	tornadoes:tornado-8000: ERROR (spawn error)
	supervisor> status
	tornadoes:tornado-8000  FATAL  unknown error making dispatchers for 'tornado-8000': EACCES
	解决办法：
	原因是权限问题，解决方法是，把 Supervisor 的日志文件，和新增的配置文件中的日志文件，设置正确的用户和组，如本例通过 tornado-8000 设置日志文件用户和组权限，另外把日志文件权限设置为 777.

# 4 MongoDB

#### 安装PyMongodb
	pip install pymongo
#### 升级
	pip install --upgrade pymongo

#### 下载mongoDB
	curl -O https://fastdl.mongodb.org/osx/mongodb-osx-x86_64-3.4.6.tgz
#### 解压
	tar -zxvf mongodb-osx-x86_64-3.4.6.tgz
	mkdir -p mongodb
	cp -R -n mongodb-osx-x86_64-3.4.6/ mongodb
#### 增加环境变量
	open .bash_profile
	#MONGODB_HOME=/Users/xavierxia/mongodb
	#export PATH=“$MONGODB_HOME/bin:$PATH”
#### 数据存放路径
	mkdir -p /data/db #默认
	mongodb/bin/mongod --dbpath /Users/xavierxia/data/db/
#### 进入mongoDB shell终端,需要开启 mongodb之后再执行该语句
	mongodb/bin/mongo

#####问题1: pymongo版本不兼容,会出现问题
	import pymongo
	conn = pymongo.Connection("localhost", 27017)
	Traceback (most recent call last):
  		File "<stdin>", line 1, in <module>
	AttributeError: 'module' object has no attribute 'Connection'
##### 解决方法
	pip install pymongo==2.7.2

##### 问题2: 终端下命令失效
	在.bash_profile中添加 export PATH=/bin:/usr/bin:/sbin:/usr/sbin 
	source .bash_profile
	
##### 问题3: mongodb数据的可视化
	下载Robo 3T

##### 问题4:可以用ipython的方式模拟本系统操作数据,如下:
	In [3]: from pymongo import MongoClient
	In [4]: client = MongoClient('localhost', 27017)
	In [5]: db = client['finance']
	In [6]: collection = db.stock
	In [9]: doc = collection.find_one({"dCode":"600000","dktype":"d"},	{"dData":1,"_id":0})

# 5 Anaconda
### 到官网下载anaconda2和3
	1.先安装 anaconda2到目录,如/Users/xavierxia/anaconda2/
	2.之后安装anaconda3到目录,如/Users/xavierxia/anaconda2/envs下面(envs为新建文件夹)
	在/Users/xavierxia/anaconda2/envs/生成test_py3和test_py2
##### 3.基于 python3.6 创建一个名为test_py3 的环境
	conda create --name test_py3 python=3.6 
##### 4.基于 python2.7 创建一个名为test_py2 的环境
	conda create --name test_py2 python=2.7

#### 激活 test 环境
	source activate test_py2 # linux/mac

#### 切换到python3
	source activate  test_py3
#### 列出当前使用的环境
	conda info --envs

##### 默认显示anaconda2的python版本
	将anaconda的bin目录加入PATH，根据版本不同，也可能是~/anaconda3/bin
	echo 'export PATH="~/anaconda2/bin:$PATH"' >> ~/.bashrc
#### 更新bashrc以立即生效
	source ~/.bashrc

#### 安装包
	conda install pymongo
