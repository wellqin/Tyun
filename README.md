# Tiops

### 基于Centos7.4 + Python-jango1.11 开发的多云堡垒机与运维系统。


#### 版本更新日志：

----------
- **release-v5.1 以及之后**

		PS：领添TiOps商业版本

		1.跳板机密码登录功能添加开关(默认关闭，不允许密码登录) (最高)    已完成
		
		2.个人登录秘钥放到个人中心里面，秘钥自带密码 (最高)    已完成
		
		3.支持主机多账号功能 (最高)      5.1前交付
		
		4.添加MFA支持(web和xhell可单独配置) (中)    web、xshell密码登录加密保  5.1前交付
		
		5.操作日志+导出操作日志(中)    6.1前交付
		
		6.用户密码复杂度可配置，包括用户账号有效期，密码有效期，密码错误账户锁定时间，xshell 最 大空闲时间 (低)  6.1前交付
		
		7.LDAP支持(低)   6.1前交付

- **release-v5.0**


		PS：此为增强版本，增加平台稳定性与优化瓶颈

		1.多云管理需要支持AWS
		  支持至少1000台主机的导入和管理（我们的长期目标是5000台）
		  主机数量多的情况，要能让用户便于查询到在平台上的资源记录或操作的日志

		2.堡垒机Xshell、putty和CRT访问堡垒机能展示权限下的业务树和主机，并可以免密登录主机
		 （不用在平台上配置系统用户和主机的绑定关系）
		  要支持100个并发终端同时登录使用
		
		3.Agent安装支持批量安装
		  安装Agent依赖包不能和客户目标机器有冲突，不能影响客户机器原有业务
		
		4.文件分发支持本地存储
		  支持500台主机的100M以内的小文件并发分发（保证成功率）
		
		5.脚本分发支持本地存储
		  支持500台主机的100M以内的脚本文件并发执行（保证成功率）
		
		6.Socket优化不能无限制开很多Socket
		  Socket后端推送状态优化
		
	
- **release-v4.0**

		1.云账号中增加已导入的主机数量显示
		
		2.云主机页面中快速查看未分配的主机
		
		3.Agent的安装效率低，无法实现批量安装
		
		4.文件分发时的临时存储点支持本地磁盘
		
		5.使用终端登录堡垒机后能看到与页面一致的主机资源（包括业务树结构）
		
		6.界面中对于大数据量的显示和使用优化
		
		7.产品中增加帮助信息显示
		
		8.内置一些常用场景的角色配置

	
	

#### 

### 主要功能：

----------


-  **首页**

        支持主机数量、主机状态、主机监控（CPU、内存、网络、磁盘IO）、访问日志查看。

- **用户中心**

        包含账号、用户、用户组管理、角色、资源授权、功能授权、配额工单模块。

- **多云管理**

        1.云账号：含三种云账号：公有云（阿里、腾讯、华为、金山、AWS、Azure）|私有云（VMware、OpenStack、ZStack）| 局域网。
        2.云主机：从云账号导入的相应主机
        3.网络：主机对应的网络

- **硬件管理**

		支持机房、机柜、服务器设备、网络设备（交换机|路由器|防火墙）管理。

- **业务树管理**

		支持将云主机资源添加到相应的业务节点进行查看，动态分配业务，一目了然。

- **运维管理**

		支持对业务树上的主机进行运维操作：
			> 初级运维：开机|关机|重启|重置密码|开启SSH会话。
              > 高级运维：批量主机的文件分发与脚本执行。
        
- **操作日志**

		包含会话、命令、文件、插件安装、文件分发、作业分发六种日志。
        
- **系统设置**

		包含自定义字段、主机登录信息、堡垒机登录信息、授权。


### 安装过程：

- 部署后端各个组件如下：

	  	# 安装基本环境
		yum install -y git wget dos2unix
		yum install -y gcc.x86_64 python-devel.x86_64 mysql-devel  //数据库的包依赖
	
		# 安装Java
		yum -y install java-1.8.0-openjdk-src.x86_64
	
		# 安装mariadb2，自启动并创建数据库tyun
		yum -y install mariadb-devel.i686 mariadb-server.x86_64
		systemctl start mariadb   #安装完启动服务并设置自启动
		systemctl enable mariadb
		mysql -uroot  -se "create database tyun default charset 'utf8';"
	
		# 安装tomcat，华为云AK认证war包
		mkdir -p /opt
		cd /opt
		wget http://mirrors.hust.edu.cn/apache/tomcat/tomcat-9/v9.0.16/bin/apache-	tomcat-9.0.16.tar.gz //会下载至命令的当前目录
		tar -zxvf apache-tomcat-9.0.16.tar.gz
	
		# 下载后端源代码
		cd /opt/tyun-service
		yum install python-pip -y
		pip install --upgrade pip
		pip install -I requests==2.18.4
		pip install -r requirements.txt -i https://pypi.douban.com/simple/   # 豆瓣源加速
	
		# 创建日志文件位置
		sudo mkdir -p /var/log/tiops/cmdb
		sudo mkdir -p /var/log/tiops/devops
		sudo mkdir -p /var/log/tiops/home
		sudo mkdir -p /var/log/tiops/hardware
		sudo mkdir -p /var/log/tiops/identity
		sudo mkdir -p /var/log/tiops/systemsetting
		sudo mkdir -p /var/log/tiops/tcmp
	
		# 安装 Redis server 3.2.12
		yum -y install redis
		systemctl enable redis
		systemctl start redis
		
		
		# 安装 rabbitmq 3.6.10
		cd /opt/
		yum install -y epel-release
		yum install -y erlang
		wget  http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.10/rabbitmq-server-3.6.10-1.el7.noarch.rpm
		yum install -y rabbitmq-server-3.6.10-1.el7.noarch.rpm
		systemctl start rabbitmq-server
		systemctl enable rabbitmq-server
		rabbitmq-plugins enable rabbitmq_management  # 启动rabbitmq的web端监控界面
		
		# 添加rabbitmq的admin用户，授予最高权限
		sudo rabbitmqctl add_user admin admin
		sudo rabbitmqctl set_user_tags admin administrator
		sudo rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
	
		# 启动tomcat
		cd /opt/apache-tomcat-9.0.16/webapps
		mv /opt/tyun-service/script/hwSignature.war /opt/apache-tomcat-9.0.16/webapps
		cd /opt/apache-tomcat-9.0.16/bin
		./startup.sh
	
		#------------------------下面配置文件，需要手动修改后执行--------------------------------------#
	
		cd /opt/tyun-service/etc/tiops/devops
		sed -ie "s/192.168.108.22/${influxdb_ip}/g"  devops.conf           # influx地址-influxdb conf
		sed -ie "s/192.168.108.23/${jumpserver_ip}/g"  devops.conf         # 堡垒机地址-jumpserver conf
		sed -ie "s/192.168.108.22/${service_ip}/g"  devops.conf            # 后端地址-saltstack conf
		sed -ie "s/production.gateway.tiops.tyun.cn/${jumpserver_url}/g"  devops.conf    # 堡垒机jumpserver域名-jumpserver xshell conf
		
		# [saltstack_master]
		sed -ie "s/124.74.245.93/${back_url}/g"  devops.conf               # 后端域名-saltstack conf，前面没有http
		
		cd /opt/tyun-service/etc/tiops/tcmp
		sed -ie "s/192.168.108.22/${service_ip}/g" tcmp.conf               # 后端地址
		
		cd /opt/tyun-service/etc/tiops/common
		sed -ie "s/develop.tiops.tyun.cn/${front_url}/g" common.conf       # 前端域名 不加http
		sed -ie "s/agent.tiops.tyun.cn/${back_url}/g" common.conf          # 后端域名  不加http
		
		
		cd /opt/tyun-service/etc   # 放在系统服务器/etc目录下生效
		cp -a * /etc/
	
		#------------------------上面配置文件，需要手动修改后执行--------------------------------------#
	
	
		sed -ie 's/192.168.108.22/localhost/g' /opt/tyun-service/settings.py  # 本地数据库配置
	
	
		# 初始化数据库表
		cd /opt/tyun-service
		python manage.py makemigrations tcmp cmdb identity hardware devops home systemsetting
		python manage.py migrate
		
		# 监控数据与分发脚本
		mkdir -p /srv/salt/_modules
		mkdir -p /srv/salt/script
		cp /opt/tyun-service/servicefiles/monitor_scripts/tiops_monitor.py /srv/salt/_modules
		
		cd /opt/tyun-service
		# 初始化个人中心数据
		# 先初始化个人中心（init.py文件在gitlab上），自定义字段，然后权限和资源
		#python manage.py shell -c 'from migrate import *; migrate()' # 失败则手动
		#python manage.py shell -c 'from exrendattr import *; exrendattr()' # ok
		
		mv script/init.py ./
		python manage.py shell -c 'from init import *; migrate(); exrendattr()'
		
		# 初始化权限  role_permission_info/permission_info
		python manage.py shell -c 'from identity.lib.perms import *;
		from identity.models import *;
		accounts = Account_info.objects.all();
		PermissionsManager().init__sql();
		PermissionsManager().init_perms(accounts[0])'
		
		# 初始化资源
		python manage.py shell -c 'from identity.lib.resourcemanager import *;ResourceManager().init__sql()'
		
		
		# 安装salt-master salt-minion salt-api
		yum install https://repo.saltstack.com/yum/redhat/salt-repo-2018.3.el7.noarch.rpm   # 指定2018版本的salt
		yum install -y salt-master salt-minion
		systemctl enable salt-master
		yum -y install salt-api pyOpenSSL
		systemctl enable salt-api
		
		
		cd /etc/pki/tls/certs/
		salt-call --local tls.create_self_signed_cert             # salt api基于证书通信
		openssl rsa -in localhost.key -out localhost_nopass.key   # 解码key，生成无密码key文件
		
		
		chmod 755 /etc/pki/tls/certs/localhost.crt
		chmod 755 /etc/pki/tls/certs/localhost.key
		chmod 755 /etc/pki/tls/certs/localhost_nopass.key
		
		
		echo "输入salt2018 "
		useradd -M -s /sbin/nologin saltapi
		#passwd saltapi
		echo "salt2018" |passwd --stdin saltapi  # 自动输入salt2018
		
		sed -i '/#default_include/s/#default/default/g' /etc/salt/master
		mkdir -p /etc/salt/master.d/
		cd /etc/salt/master.d/
		touch eauth.conf
		touch api.conf
		
		cat << EOF > /etc/salt/master.d/eauth.conf
		external_auth:
		  pam:
		    saltapi:   # 用户
		      - .*     # 该配置文件给予saltapi用户所有模块使用权限，出于安全考虑一般只给予特定模块使用权限
		      - '@runner'
		EOF
		
		cat << EOF > /etc/salt/master.d/api.conf
		rest_cherrypy:
		  port: 8001
		  ssl_crt: /etc/pki/tls/certs/localhost.crt
		  ssl_key: /etc/pki/tls/certs/localhost_nopass.key
		EOF
		
		# salt配置文件修改
		sed -i "s/#order_masters: False/order_masters: True/g" /etc/salt/master
		sed -i "s/#auto_accept: False/auto_accept: True/g" /etc/salt/master
		systemctl restart salt-master
		systemctl start salt-api
		
		# 部分源码加密,大约30min,会在当前项目下生成build目录
		# 先下载依赖GCC与python-devel，Cpython
		sudo yum install -y gcc.x86_64
		sudo yum install -y python-devel.x86_64
		sudo pip install Cython --install-option="--no-cython-compile"
		
		cd /opt/tyun-service
		sleep 2
		python setup.py tcmp
		python setup.py cmdb
		python setup.py identity
		python setup.py hardware
		python setup.py devops
		python setup.py home
		python setup.py systemsetting
		
		sleep 2
		cd /opt/tyun-service
		rm -rf tcmp cmdb identity hardware devops home systemsetting
		mv build/* ./
		rm -rf build/
	
		# 启动项目
		cd /opt/tyun-service
		pip install gunicorn
		gunicorn  wsgi -b 0.0.0.0:8088 -k eventlet -w 16 --access-logformat '%(h)s %(t)s "%(r)s" %(s)s %(b)s '  -p /opt/tyun-service/script/gunicorn.pid --access-logfile /opt/tyun-service/script/gunicorn_access.log --error-logfile /opt/tyun-service/script/gunicorn_error.log  --log-level debug -t 300 --daemon
		
		# 开启后台定时任务
		cd /opt/tyun-service
		python manage.py crontab add
		
		# 开启celery
		python script/celery start -d
		
		# 最后注意授权许可证
