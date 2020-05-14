# architect
#### architect is my destination !

### 1. socket & tcp

```json
# 查看端口范围
cat /proc/sys/net/ipv4/ip_local_port_range
# 修改随机端口范围
echo "10000 61000" > /proc/sys/net/ipv4/ip_local_port_range



cat /proc/sys/net/ipv4/tcp_timestamps # 打开
cat /proc/sys/net/ipv4/tcp_tw_reuse   # 打开 前提是tcp_timestamps开  
cat /proc/sys/net/ipv4/tcp_tw_recycle # 客户端处于nat网络时不可打开,负载均衡器上不可打开


# ab 压力测试
# ab -n 总共发起的请求数 -c 并发数 http://www.baidu.com

yum install nc -y
nc -l -4 -p 9999 -k

# 问题:
# sendfile        on;
# tcp_nopush     on;
# tcp_nodelay    on;
# nignx怎么滚动升级?
# OSI七层模型各层分别有哪些协议及它们的功能?
# tcp三次握手与四次挥手? tcp状态转换

```

### 2. 负载均衡

```json
# 四层负载均衡 &  七层负载均衡

Host------>IP network-------->VSIP/LB Device------>[ServerA/IP A ,ServerB/IP B,ServerC/IP C ]

# 1.NAT方式报文:

#--------------------------------------------------------------------------------------
# 步 骤                   说 明                                备注
#--------------------------------------------------------------------------------------
#  1               Host发送服务请求报文				  源IP为Host IP,目的IP为VSIP
#--------------------------------------------------------------------------------------
#  2         LB Device接收到请求报文后,借助持续性功能
#            或调度算法计算出应该将请求分发给哪台Server                 --
#--------------------------------------------------------------------------------------
#  3         LB Device使用DNAT技术分发报文             源IP为Host IP,目的IP为Server IP
#--------------------------------------------------------------------------------------
#  4         Server接收并处理请求报文,返回响应报文       源IP为Server IP,目的IP为Host IP
#--------------------------------------------------------------------------------------
#  5         LB Device接受响应报文,转换源IP后转发       源IP为VSIP,目的IP为Host IP
#--------------------------------------------------------------------------------------

# 2. DR(直接路由模式)方式
								LB Device
									^
									|
									|
Host-------->IP network--------> General Device------->[ServerA,VSIP/IP A,ServerB,VSIP/IP B,ServerC,VSIP/IP C ]

# DR方式报文:

#--------------------------------------------------------------------------------------
# 步 骤                   说 明                                备注
#--------------------------------------------------------------------------------------
#  1               Host发送服务请求报文				  源IP为Host IP,目的IP为VSIP
#--------------------------------------------------------------------------------------
#  2      General Device收到请求后转发给LB Device     Server上的VSIP不会响应ARP请求
#--------------------------------------------------------------------------------------
#  3      LB Device接收到请求报文后,借助持续性功能
#         或调度算法计算出应该将请求分发给哪台Server               --
#--------------------------------------------------------------------------------------
#  4      LB Device分发报文              源IP为Host IP,目的IP为VSIP,目的MAC为Server的MAC地址
#--------------------------------------------------------------------------------------
#  5      Server接收并处理请求报文,返回响应报文            源IP为VSIP,目的IP为Host IP
#--------------------------------------------------------------------------------------
#  6      General Device收到响应报文后,直接将报文转发给Host         --
#--------------------------------------------------------------------------------------


							Load Balancer
 							  Layer 4
Client -----------------------------------------------------------> Server
			 修改报头目标地址进行转发 + 修改源地址(根据需求)     转 发


								

							Load Balancer
 							  Layer 7
         独立的TCP连接                       	独立的TCP连接 
Client --------------------- Proxy Code ---------------------------> Server
														  代 理


```

### 3. web集群中的session处理

```json
# 1. session保持
	* Nginx: ip_hash
	* Haproxy: sourceip
# 2. session复制
	* 复制
# 3. session共享
	* PHP: Memcached,Redis
	* Django: session框架支持
	* Tomcat: tomcat-x-session-manager
```

### 4.  负载均衡/代理能获取到用户的真实IP吗?

```json
answer: No

# 如果用户使用了代理,然后进行伪造的话,那么是无法获取用户的真实IP的.就算获取XFF(x-forwarded-for)的最后一位也只是代理的IP地址.
# 如果非匿名代理,是可以获取到用户的真实IP地址的.

# 只要用户使用了代理,都无法获取到用户真实IP地址的;
# 正常应该这样获取: 业务层面应该是先获取x-real-ip,如果Nginx设置了,这个值永远不会为空.但是高仿之后,这个值就不对了,要从XFF获取了.
# 方案一: 不使用高仿,运维把$remote_add赋值给X-Real-IP,业务获取这个;使用高仿的时候,运维手动修改配置 X-Real-IP $http_x_forwarded_for,把XFF传给X-Real-IP.业务还是取这个,正常是取第一个.但是若用户使用代理或伪造,取到的就会不对.
# 方案二: 业务层面协调一下
# 无论高仿不高仿,只要用户使用代理,都无法获取到用户的真实IP地址.
```

### 5.  Haproxy 

```json
# 安装软件依赖包
yum install -y net-tools vim lrzsz tree screen lsof tcpdump nc mtr nmap gcc gcc-c++ make
yum install -y haproxy 


```

### 6.  Haproxy + keepalived 架构

```json
yum -y  install deltarpm
yum install -y haproxy keepalived

 
echo 1 >  /proc/sys/net/ipv4/ip_nonlocal_bind  # 监听非本地IP

```

### 7.  灾备

```json
《中华人民共和国国家标准信息安全技术信息系统灾难恢复规范 GB/T 20988-2007》
```

### 8.  DNS

```json
# 域名系统（服务）协议（DNS）是一种分布式网络目录服务，主要用于域名与 IP 地址的相互转换，以及控制因特网的电子邮件的发送。
# 硬件选型
	* CPU: 12C以上
	* 内存: 16G
	* 网络: 千兆
# 系统初始化
	* 关闭selinux:  /etc/selinux/config
	* 关闭iptables: chkconfig iptable off
	* 调整ulimit限制:  echo -e '* soft nproc 65536\n* hard nproc 65536\n* soft nofile 65536\n* hard nofile 65536\n' >> /etc/security/limits.conf
```

### 9.  

```json

```

### 10.  jenkins install

```json
# docker  installation
yum install -y yum-utils device-mapper-persistent-data lvm2 
yum install -y java-1.8.0 java-1.8.0-openjdk-devel

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum clean all 

yum makecache fast

yum -y install docker-ce

systemctl start docker

docker version

# docker-compose installation

curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

docker-compose --version


# gitlab-ce.tar.gz jenkins.docker.gz jenkins.tar.gz sonarqube.tar.gz

docker load -i gitlab-ce.tar.gz

docker-cpmpose up -d gitlab

docker ps


# jenkins installation 
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

yum install jenkins


# jenkins docker installation 

# 获取镜像
docker pull jenkins/jenkins:lts
docker pull jenkins/jenkins
# 启动服务
docker run -it --name jenkinsci -v $HOME/jenkins:/var/jenkins_home -p 8080:8080 -p 50000:50000 -p 45000:45000 jenkins/jenkins-lts 


# gitlab install
# https://www.cnblogs.com/zhangycun/p/10963094.html
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-10.0.0-ce.0.el7.x86_64.rpm
rpm -i gitlab-ce-10.0.0-ce.0.el7.x86_64.rpm
vim  /etc/gitlab/gitlab.rb
	external_url "http://服务器IP:port" 默认 8080


gitlab-ctl reconfigure
gitlab-ctl restart  

rpm -e gitlab-ce # 卸载

初始账户: root 密码:5iveL!fe
```

### 11.  jenkins 

```json
# SSH clone code
ssh-keygen -t rsa -C assasin0308@sina.com
# 粘贴公钥至gitlab

# apache-maven installation
wget http://archive.apache.org/dist/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz
tar zxvf apache-maven-3.5.2-bin.tar.gz

vim /etc/profile
export M3_HOME=/usr/local/apache-maven-3.5.2
export PATH=$PATH:$M3_HOME/bin
source /etc/profile

mvn -version
mvn clean compile
mvn package


# Sonarqube代码质量平台
# jenkins与Sonarqube集成   

https://www.cnblogs.com/ding2016/p/8065241.html 
官方文档：https://docs.sonarqube.org/display/SONAR/Installing+the+Server
官方下载：https://www.sonarqube.org/downloads/




# start sonarquebe
docker-compose up -d sonarqube

安装 Sonarqube Scanner for jenkins 插件
系统设置---> Sonarqube servers---> add 
Global Tools Configuration ---> Sonarqube Scanner ---> add
配置--->增加构建步骤---> Execute Sonarqube Scanner--->Analysis propertites ---> ....
	sonar.projectKey=demo
	sonar.projectName=demo
	sonar.ProjectVersion=1.0
	sonar.source=./account-email/src
	sonar.java.binaries=./account-email/target

#----------------------------------------------------------------
    sonar.sourceEncoding=UTF-8
    sonar.language=java
    sonar.modules=java-module

    java-module.sonar.projectName=Java module

    # 正确的配置
    java-module.sonar.sources=src
    java-module.sonar.projectBaseDir=.
    sonar.java.binaries=bin
#-----------------------------------------------------------------

```

### 12.  SaltStack installation & configuration

```json
# 四大功能: 远程执行 配置管理  云管理 事件驱动

#-------------------------------------------------------------------------------------
# 安装  https://www.cnblogs.com/xintiao-/p/10380656.html
	wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
	wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
	yum clean all # 清空缓存
	yum makecache # 生成yum缓存
	# 查看salt包
	yum list salt

	# 安装salt-master
	yum install salt-master -y
	# 安装salt-minion
	yum install salt-minion -y

# 配置 - /etc/salt/master
interface: 0.0.0.0  #绑定到本地的0.0.0.0地址
publish_port: 4505　　#管理端口，命令发送
user: root　　　　　　#运行salt进程的用户
worker_threads: 5　　#salt运行线程数，线程越多处理速度越快，不要超过cpu个数
ret_port: 4506　　#执行结果返回端口
pidfile: /var/run/salt-master.pid #pid文件位置
log_file: /var/log/salt/master　　#日志文件地址

#自动接收minion的key
auto_accept: False

# 配置 - /etc/salt/minion
master: master
master_port: 4506
user: root
id: slave
acceptance_wait_time: 10
log_file: /var/log/salt/minion

systemctl start salt-minion
systemctl start salt-master

#-------------------------------------------------------------------------------------


# on 192.168.2.102  salt-master +  salt-minion
1. yum install salt-master salt-minion -y 
3. systemctl start salt-master

5. vim /etc/salt/minion
master: 192.168.2.102
6. systemctl start salt-minion
8. tree /etc/salt/pki/
/etc/salt/pki/
├── master
│   ├── master.pem
│   ├── master.pub
│   ├── minions
│   ├── minions_autosign
│   ├── minions_denied
│   ├── minions_pre
│   │   ├── 192.168.2.102
│   │   └── 192.168.2.104
│   └── minions_rejected
└── minion
    ├── minion.pem
    └── minion.pub 

9. salt-key -A # 同意所有
10. salt '*' test.ping 或 salt \* test.ping # 验证      master使用zeromq向minion发消息 订阅/发布
11. lsof -n -i:4505  # 检查所有与master 4505端口的连接
12. salt '*' cmd.run 'w'  # 在所有机器执行w命令




# on 192.168.2.104  salt-minion
2. yum install  salt-minion -y 

4. vim /etc/salt/minion
master: 192.168.2.102
7. systemctl start salt-minion


```

### 13.  SaltStack 一

```json
# 配置规则一
  YAML文件使用固定的缩进风格表示数据层级关系,Slat需要每个缩进级别由两个空格组成;
  不能使用tab
# 配置规则二 冒号后加空格
YAML   my_key: my_value 


1. vim /etc/salt/master

    416 file_roots:
    417   base:
    418     - /srv/salt/base
    419   dev:
    420     - /srv/salt/dev
    421   test:
    422     - /srv/salt/test
    423   prod:
    424     - /srv/salt/prod

2. mkdir -p /srv/salt/{base,dev,test,prod}
3. systemctl restart salt-master # 重启
4. salt '*' test.ping # 测试
#-------------------------------------------------
5. 安装apapche
	cd /srv/salt/base/
	vim apache.sls # 以 .sls 文件结尾
	apache-install:
      pkg.installed:
        - name: httpd

	apache-service:
      service.running:
        - name: httpd
        - enbale: True
6. salt '192.168.2.104*' state.sls apache
# 执行效果如下:
#---------------------------------------------------------------------------
192.168.2.104:
----------
          ID: apache-install
    Function: pkg.installed
        Name: httpd
      Result: True
     Comment: Package httpd is already installed.
     Started: 07:07:47.137344
    Duration: 5760.31 ms
     Changes:   
----------
          ID: apache-service
    Function: service.running
        Name: httpd
      Result: True
     Comment: Started Service httpd
     Started: 07:07:53.047608
    Duration: 2547.25 ms
     Changes:   
              ----------
              httpd:
                  True

Summary
------------
Succeeded: 2 (changed=1)
Failed:    0
------------
Total states run:     2
#---------------------------------------------------------------------------	
7. 将 apache.sls归档至 web 目录
/srv/salt/base/
└── web
    └── apache.sls
8. salt '192.168.2.104*' state.sls web.apache  # web.apache
# 执行效果同上

# salt 高级状态 

9. vim top.sls
    base:
      '192.168.2.104':
        - web.apache
      '192.168.2.102':
        - web.apache
10. salt '*' state.highstate
# 执行效果如下:
#---------------------------------------------------------------------------
192.168.2.104:
----------
          ID: apache-install
    Function: pkg.installed
        Name: httpd
      Result: True
     Comment: Package httpd is already installed.
     Started: 07:12:02.969437
    Duration: 3837.104 ms
     Changes:   
----------
          ID: apache-service
    Function: service.running
        Name: httpd
      Result: True
     Comment: Started Service httpd
     Started: 07:12:06.807070
    Duration: 404.687 ms
     Changes:   
              ----------
              httpd:
                  True

Summary
------------
Succeeded: 2 (changed=1)
Failed:    0
------------
Total states run:     2
192.168.2.102:
----------
          ID: apache-install
    Function: pkg.installed
        Name: httpd
      Result: True
     Comment: Package httpd is already installed.
     Started: 23:19:41.248380
    Duration: 4942.367 ms
     Changes:   
----------
          ID: apache-service
    Function: service.running
        Name: httpd
      Result: True
     Comment: Started Service httpd
     Started: 23:19:46.266833
    Duration: 3536.332 ms
     Changes:   
              ----------
              httpd:
                  True

Summary
------------
Succeeded: 2 (changed=1)
Failed:    0
------------
Total states run:     2
#---------------------------------------------------------------------------



# 参考:
https://www.unixhot.com/docs/saltstack/
# salt所有状态:
https://www.unixhot.com/docs/saltstack/ref/states/all/index.html#all-salt-states




# saltstack 配置管理 lamp状态管理
1. lamp.sls
/srv/salt/base/
├── top.sls
└── web
    ├── apache.sls
    └── lamp.sls
2. vim  lamp.sls
#---------------------------------------------------------------------------
lamp-install:  # 状态id
  pkg.installed:
    - pkgs:
      - httpd
      - php
      - php-pdo
      - php-mysql

apache-config:
  file.managed:
    - name: /etc/httpd/conf/httpd.conf
    - source: salt://web/files/httpd.conf  # base路径
    - user: root
    - group: root
    - mode: 644
    - require:
      - pkg: lamp-install

apache-auth:
  pkg.installed:
    - name: httpd-tools
    - require_in:
      - cmd: apache-auth
  cmd.run:
    - name: htpasswd -bc /etc/httpd/conf/htpasswd_file admin admin
    - unless: test -f /etc/httpd/conf/htpasswd_file

apache-conf:
  file.recurse:
    - name: /etc/httpd/conf.d
    - source: salt://web/files/apache-conf.d
    - watch_in:
      - service: lamp-service

/etc/php.ini:
  file.managed:
    - source: salt://web/files/php.ini
    - user: root
    - group: root
    - mode: 644
    - watch_in:
      - service: lamp-service

lamp-service:
  service.running:
    - name: httpd
    - enable: True
    - reload: True
    - watch:
      - file: apache-config
      - file: apache-conf
      
#---------------------------------------------------------------------------

2. salt '192.168.2.104*' state.highstate test=True

3. vim append.sls  追加方式
/srv/salt/base/
├── top.sls
└── web
    ├── apache.sls
    ├── append.sls
    └── lamp.sls

/etc/profile:
  file.append:
    - text:
      - "#aaa"

4. salt '*' state.sls web.append
# 执行效果如下:
#---------------------------------------------------------------------------
192.168.2.102:
----------
          ID: /etc/profile
    Function: file.append
      Result: True
     Comment: File /etc/profile is in correct state
     Started: 23:50:48.453648
    Duration: 7.42 ms
     Changes:   

Summary
------------
Succeeded: 1
Failed:    0
------------
Total states run:     1
192.168.2.104:
----------
          ID: /etc/profile
    Function: file.append
      Result: True
     Comment: File /etc/profile is in correct state
     Started: 07:43:19.090493
    Duration: 6.02 ms
     Changes:   

Summary
------------
Succeeded: 1
Failed:    0
------------
Total states run:     1
#---------------------------------------------------------------------------
5. 包管理--- 功能模块
# 参考:
https://docs.saltstack.com/en/latest/ref/states/all/salt.states.pkg.html

状态模块: pkg
功 能: 管理软件包状态
常用方法:
	pkg.installed # 确保软件包已安装,若没安装则进行安装
	pkg.latest    # 确保软件包是最新版本,如果不是进行升级
	pkg.remove    # 确保软件包已卸载,如果之前已安装.进行卸载
	pkg.purge     # 出remove外,也会删除其配置

状态模块: file
功 能: 管理文件状态
常用方法:
	file.managed # 保证文件存在并且为对应的状态
	file.recurse # 保证目录存在并且为对应的状态
	file.absent  # 确保文件不存在,如果存在则删除

状态模块: service
功 能: 管理服务状态
常用方法:
	service.running # 确保服务处于运行状态,若没运行则启动
	service.enabled # 确保服务开机自启动
	service.disabled # 确保服务开机不自启动
	service.dead     # 确保服务当前没有运行,若运行则停止运行

状态模块: requisites
功 能: 处理状态间关系
常用方法:
	require # 我依赖某个状态
	require_in # 我被某个状态依赖
	watch # 我关注某个状态
	watch_in # 我被某个状态关注

# unless: 状态判断 条件为真(0),则不执行
# cmd.run: 执行模块 


```

### 14.  SaltStack 二

```json
# Tomcat 安装管理

1. vim tomcat.sls

jdk-install:
  pkg.installed:
    - name: java-1.8.0-openjdk

tomcat-install:
  file.managed:
    - name: /usr/local/src/apache-tomcat-8.0.46.tar.gz
    - source: salt://web/apache-tomcat-8.0.46.tar.gz
    - user: root
    - group: root
    - mode: 755
  cmd.run:
    - name: cd /usr/local && tar zxf apache-tomcat-8.0.46.tar.gz && mv apache-tomcat-8.0.46 /usr/local/ && ln -s /usr/local/apache-tomcat-8.0.46 /usr/local/tomcat
    - unless: test -L /usr/local/tomcat && test -d /usr/local/apache-tomcat-8.0.46


2. salt '192.168.2.104*' state.sls web.tomcat
# 执行效果如下:
#---------------------------------------------------------------------------
192.168.2.104:
----------
          ID: jdk-install
    Function: pkg.installed
        Name: java-1.8.0-openjdk
      Result: True
     Comment: Package java-1.8.0-openjdk is already installed.
     Started: 09:17:51.683726
    Duration: 796.258 ms
     Changes:   
----------
          ID: tomcat-install
    Function: file.managed
        Name: /usr/local/apache-tomcat-8.0.46.tar.gz
      Result: True
     Comment: File /usr/local/apache-tomcat-8.0.46.tar.gz is in the correct state
     Started: 09:17:52.481986
    Duration: 163.66 ms
     Changes:   
----------
          ID: tomcat-install
    Function: cmd.run
        Name: cd /usr/local/ && tar zxf apache-tomcat-8.0.46.tar.gz && mv apache-tomcat-8.0.46 /usr/local/tomcat  && ln -s /usr/local/apache-tomcat-8.0.46 /usr/local/tomcat
      Result: True
     Comment: Command "cd /usr/local/ && tar zxf apache-tomcat-8.0.46.tar.gz && mv apache-tomcat-8.0.46 /usr/local/tomcat  && ln -s /usr/local/apache-tomcat-8.0.46 /usr/local/tomcat" run
     Started: 09:17:52.646182
    Duration: 207.545 ms
     Changes:   
              ----------
              pid:
                  8610
              retcode:
                  0
              stderr:
              stdout:

Summary
------------
Succeeded: 3 (changed=1)
Failed:    0
------------
Total states run:     3
#---------------------------------------------------------------------------
3. salt '*' state.sls web.tomcat

```

### 15.  SaltStack 三

```json
# salt数据系统: Grains  &   Pillar

# Minion启动时收集(静态数据)
# grains应用场景:
	grains可以再salt系统中用于配置管理模块
	Grains可以以target中使用,用来匹配Minion,如匹配操作系统,使用-G选项
	# salt -G "os:CentOS" cmd.run 'uptime'
	# 执行效果如下:
	#---------------------------------------------------------------------------
	192.168.2.104:
     09:47:17 up  2:45,  1 user,  load average: 0.00, 0.01, 0.06
	192.168.2.102:
     22:59:03 up 40 min,  3 users,  load average: 0.00, 0.07, 0.22
	#---------------------------------------------------------------------------
	Grains用于信息查询
	Grains保存着收集到的客户端的详细信息

1. salt '192.168.2.104*' grains.ls 
2. salt '192.168.2.104*' grains.items
3. salt '192.168.2.104*' grains.item fqdn_ip4
```

### 16.  SaltStack 四 

```json
# Apache监听本地IP地址  结合jinjia模板

	变量使用Grains: {{ grains['fqdn_ip4'][0]}}
	变量使用执行模块: {{salt['network.hw_addr']('eth0')}}
	变量使用Pillar: {{pillar['apache']['PORT']}}

# lamp-jinjia.sls

lamp-install:
  pkg.installed:
    - pkgs:
      - httpd
      - php
      - php-pdo
      - php-mysql

apache-config:
  file.managed:
    - name: /etc/httpd/conf/httpd.conf
    - source: salt://web/files/httpd.conf
    - user: root
    - group: root
    - mode: 644
    - template: jinja
    - defaults:
      PORT: 80 
      IPADDR: {{ grains['fqdn_ip4'][0] }}
    - require:
      - pkg: lamp-install

apache-auth: 
  pkg.installed:
    - name: httpd-tools
    - require_in:
      - cmd: apache-auth
  cmd.run:
    - name: htpasswd -bc /etc/httpd/conf/htpasswd_file admin admin
    - unless: test -f /etc/httpd/conf/htpasswd_file

apache-conf:
  file.recurse:
    - name: /etc/httpd/conf.d
    - source: salt://web/files/apache-conf.d
    - watch_in:
      - service: lamp-service

/etc/php.ini:
  file.managed:
    - source: salt://web/files/php.ini
    - user: root
    - group: root
    - mode: 644
    - watch_in:
      - service: lamp-service

lamp-service:
  service.running:
    - name: httpd
    - enable: True
    - reload: True
    - watch:
      - file: apache-config

```

### 17.  SaltStack 五

```json    
# salt的生产实践
	不推荐使用 file 目录模块模块进行代码部署
	不建议salt管理项目的配置文件,建议分层管理,salt只管理应用服务
	若有固定的文件服务器,可以使用 source: salt://  http://  ftp://
	SLS版本化
	使用版本化 Job Cache保存job的输出到MySQL中
	/var/cache/salt/master/jobs/
	默认保留24小时: keep_jobs: 24

# 远程执行
salt <target> <function> <arguments>
salt '*' cmd.run 'df -h'

# 将返回的数据持久化保存 
salt '*' test.ping --return redis_return
salt '*' test.ping --return mongo_return,redis_return,cassandra_return,elasticsearch_return ******

# return ---> mysql
https://www.unixhot.com/docs/saltstack/ref/returners/all/salt.returners.mysql.html#module-salt.returners.mysql

#  yum -y install mysql-python
1. vim /etc/salt/master  
# Which returner(s) will be used for minion's result:
#return: mysql
master_job_cache: mysql
mysql.host: '192.168.2.103'
mysql.user: 'root'
mysql.pass: '*********'
mysql.db: 'salt'
mysql.port: 3306

2. systemctl restart salt-master
3. salt '*' test.ping
4. salt '*' cmd.run 'df -h'


# 目标选择 
salt '*' test.ping  # *匹配
salt 'linux-node[1-2].com' test.ping 
salt 'linux-node[1,2].com' test.ping 
sale -E 'linux-node(1|2).com' test.ping  # 正则匹配 -E

base:
  'linux-node(1|2).com':
    - match: pcre
    - web.lamp

salt -L 'linux-node1.com,linux-node2.com,linux-node3.com' test.ping  # List 
salt -G 'os:CentOs' test.ping # -G grains
salt -S 192.168.2.103/24 test.ping # -S ip
salt -S 192.168.2.104 test.ping 


salt '*' -b 10 test.ping # 批处理
salt '*' -G 'os:CentOS' --batch-size 25% apache.signal restart 

# 执行模块

salt '*' network.active_tcp  # 活跃的tcp连接
salt '*' network.arp 

salt '*' network.connect archlinux.org 80 # 测试端口
salt '*' network.connect archlinux.org 80 timeout=3
salt '*' network.connect archlinux.org 80 timeout=3 family=ipv4
salt '*' network.connect archlinux.org port=53 proto=udp timeout=3
 
salt '*' network.dig archinux.org # 域名解析
salt '*' network.netstat 
salt '*' network.ping  www.baidu.com 

salt '*' state.show_top

# 日常管理
# include  包含状态
include:
  - web.httpd  # include httd.sls
   
# 测试
salt-run manage.status
salr-run manage.versions

# 修改minion_id
	停止minion服务
	salt-key -d minionid 删除minion
	rm -r /etc/salt/minion_id
	rm -f /etc/salt/pki
	修改配置文件id
	启动minion服务

```

### 18.  SaltStack 六     

```json
# salt本地管理 无master架构

# file_client: remote   # 将 remote 改为 local
salt-call --local state.sls web.tomcat

#  zabbix-agent 案例
/srv/salt/
├── base     # 公共的
│   ├── init  # ---初始化
│   │   ├── files
│   │   │   └── epel-7.repo
│   │   └── yum-repo.sls
│   ├── logstash   # ---logstash
│   ├── top.sls
│   ├── web
│   │   ├── apache.sls
│   │   ├── apache-tomcat-8.0.46.tar.gz
│   │   ├── append.sls
│   │   ├── lamp.sls
│   │   └── tomcat.sls
│   └── zabbix   # ---zabbix
│       ├── files
│       │   └── zabbix_agentd.conf
│       └── zabbix-agent.sls
├── dev
├── prod
└── test

1. vim yum-repo.sls
/etc/yum.repos.d/epel-7.repo
  file.managed:
    - source: salt://init/files/epel-7.repo
    - user: root
    - group: root
    - mode: 644

2. vim zabbix-agent.sls
#include:
  #- init: yum-repo

zabbix-agent:
  pkg.installed:
    - name: zabbix40-agent
    #- require:
    #  - file: /etc/yum.repos.d/epel.repo
  file.managed:
    - name: /etc/zabbix_agentd.conf
    - source: salt://zabbix/files/zabbix_agentd.conf
    - user: root
    - group: root
    - mode: 644
    - template: jinja
    - defaults:
      ZABBIX_SERVER: 192.168.2.103
      AGENT_HOSTNAME: {{ grains['fqdn'] }}
    - require:
      - pkg: zabbix-agent
  service.running:
    - name: zabbix-agent
    - enable: True
    - watch:
      - file: zabbix-agent 
      - pkg: zabbix-agent

zabbix_agent.conf.d:
  file.directory:
    - name: /etc/zabbix_agentd.conf.d
    - watch_in:
      - service: zabbix-agent
    - require:
      - pkg: zabbix-agent
      - file: zabbix-agent
3. zabbix_agentd.conf
Server={{ ZABBIX_SERVER }}
Hostname={{ AGENT_HOSTNAME }}

Include=/etc/zabbix_agentd.conf.d/ # 去掉注释
4. salt '*' state.sls zabbix.zabbix-agent test=True
5. salt '*' state.sls zabbix.zabbix-agent 
# 执行效果如下:
#---------------------------------------------------------------------------
192.168.2.102:
----------
          ID: zabbix-agent
    Function: pkg.installed
        Name: zabbix40-agent
      Result: True
     Comment: Package zabbix40-agent is already installed.
     Started: 23:38:15.756715
    Duration: 15205.207 ms
     Changes:   
----------
          ID: zabbix-agent
    Function: file.managed
        Name: /etc/zabbix_agentd.conf
      Result: True
     Comment: The file /etc/zabbix_agentd.conf is in the correct state
     Started: 23:38:30.963639
    Duration: 20.409 ms
     Changes:   
----------
          ID: zabbix_agent.conf.d
    Function: file.directory
        Name: /etc/zabbix_agentd.conf.d
      Result: True
     Comment: The directory /etc/zabbix_agentd.conf.d is in the correct state
     Started: 23:38:30.987142
    Duration: 1.938 ms
     Changes:   
----------
          ID: zabbix-agent
    Function: service.running
      Result: True
     Comment: Service zabbix-agent is already enabled, and is in the desired state
     Started: 23:38:30.989642
    Duration: 1308.499 ms
     Changes:   

Summary
------------
Succeeded: 4
Failed:    0
------------
Total states run:     4
192.168.2.104:
----------
          ID: zabbix-agent
    Function: pkg.installed
        Name: zabbix40-agent
      Result: True
     Comment: Package zabbix40-agent is already installed.
     Started: 05:25:14.603753
    Duration: 13347.962 ms
     Changes:   
----------
          ID: zabbix-agent
    Function: file.managed
        Name: /etc/zabbix_agentd.conf
      Result: True
     Comment: The file /etc/zabbix_agentd.conf is in the correct state
     Started: 05:25:28.106326
    Duration: 120.26 ms
     Changes:   
----------
          ID: zabbix_agent.conf.d
    Function: file.directory
        Name: /etc/zabbix_agentd.conf.d
      Result: True
     Comment: The directory /etc/zabbix_agentd.conf.d is in the correct state
     Started: 05:25:28.227367
    Duration: 0.364 ms
     Changes:   
----------
          ID: zabbix-agent
    Function: service.running
      Result: True
     Comment: Service zabbix-agent is already enabled, and is in the desired state
     Started: 05:25:28.227845
    Duration: 1109.733 ms
     Changes:   

Summary
------------
Succeeded: 4
Failed:    0
------------
Total states run:     4
#---------------------------------------------------------------------------

6. 系统初始化
	DNS   file.managed
	防火墙  service.dead
	limit设置  file.managed
	SSH useDNS设置,修改端口 file.managed
	systemctl 内核参数调优 systemctl
	关闭不需要的服务 service
	时间同步  file.managed cron
	基础软件包 pkg.installed
	include:
      - init.yum-repo

    base-pkg:
      pkg.installed:
        - pkg:
          - screen
          - lrzsz
          - vim
	yum源 file.managed


```

### 19.  SaltStack 七

```json
# salt Redis部署

/srv/salt/prod/
├── modules
│   ├── apache
│   ├── haproxy
│   ├── keepalived
│   ├── mysql
│   └── redis
│       └── redis-install.sls
└── redis-cluster
    ├── files
    │   └── redis-master.conf
    └── redis-master.sls


1. vim redis-install.sls
redis-install:
  pkg.installed:
    - name: redis
2. redis-master.sls
include:
  - modules.redis.redis-install

redis-master-config:
  file.managed:
    - name: /etc/redis.conf
    - source: salt://redis-cluster/files/redis-master.conf
    - user: root
    - group: root
    - mode: 644
    - template: jinja
    - defaults:
      REDIS_MEM: 100M

redis-master-service:
  service.running:
    - name: redis
    - enable: True
    - watch:
      - file: redis-master-config

3. salt '*' state.sls redis-cluster.redis-master  saltenv=prod  # 默认 base环境
# 执行效果如下:
#---------------------------------------------------------------------------
192.168.2.102:
----------
          ID: redis-install
    Function: pkg.installed
        Name: redis
      Result: None
     Comment: The following packages are set to be installed/updated: redis
     Started: 00:33:34.239727
    Duration: 1015.057 ms
     Changes:   
----------
          ID: redis-master-config
    Function: file.managed
        Name: /etc/redis.conf
      Result: None
     Comment: The file /etc/redis.conf is set to be changed
     Started: 00:33:35.257619
    Duration: 288.224 ms
     Changes:   
              ----------
              newfile:
                  /etc/redis.conf
----------
          ID: redis-master-service
    Function: service.running
        Name: redis
      Result: None
     Comment: Service is set to be started
     Started: 00:33:35.601150
    Duration: 226.642 ms
     Changes:   

Summary
------------
Succeeded: 3 (unchanged=3, changed=1)
Failed:    0
------------
Total states run:     3
192.168.2.104:
----------
          ID: redis-install
    Function: pkg.installed
        Name: redis
      Result: True
     Comment: Package redis is already installed.
     Started: 06:20:39.511771
    Duration: 999.409 ms
     Changes:   
----------
          ID: redis-master-config
    Function: file.managed
        Name: /etc/redis.conf
      Result: None
     Comment: The file /etc/redis.conf is set to be changed
     Started: 06:20:40.513646
    Duration: 299.917 ms
     Changes:   
              ----------
              diff:
                  --- 
                  +++ 
                  @@ -58,7 +58,7 @@
                   # IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
                   # JUST COMMENT THE FOLLOWING LINE.
                   # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                  -bind 127.0.0.1
                  +bind 0.0.0.0
                   
                   # Protected mode is a layer of security protection, in order to avoid that
                   # Redis instances left open on the internet are accessed and exploited.
                  @@ -125,7 +125,7 @@
                   
                   # By default Redis does not run as a daemon. Use 'yes' if you need it.
                   # Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
                  -daemonize no
                  +daemonize yes
                   
                   # If you run Redis from upstart or systemd, Redis can interact with your
                   # supervision tree. Options:
                  @@ -534,7 +534,7 @@
                   # limit for maxmemory so that there is some free RAM on the system for slave
                   # output buffers (but this is not needed if the policy is 'noeviction').
                   #
                  -# maxmemory <bytes>
                  + maxmemory 100M
                   
                   # MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
                   # is reached. You can select among five behaviors:
              mode:
                  0644
              user:
                  root
----------
          ID: redis-master-service
    Function: service.running
        Name: redis
      Result: None
     Comment: Service is set to be started
     Started: 06:20:41.150092
    Duration: 239.849 ms
     Changes:   

Summary
------------
Succeeded: 3 (unchanged=2, changed=1)
Failed:    0
------------
Total states run:     3
#---------------------------------------------------------------------------
```

### 20.  SaltStack 八

```json
# salt-ssh systemctl stop salt-minion
1. yum install -y salt-ssh

2. vim /etc/salt/roster 
# Sample salt-ssh config file
#web1:
#  host: 192.168.42.1 # The IP addr or DNS hostname
#  user: fred         # Remote executions will be executed as user fred
#  passwd: foobarbaz  # The password to use for login, if omitted, keys are used
#  sudo: True         # Whether to sudo to root, not enabled by default
#web2:
#  host: 192.168.42.2
node1:
  host: 192.168.2.104
  user: root
  passwd: 19920308shibin
  port: 22

node2:
  host: 192.168.2.103
  user: root
  passwd: 19920308shibin
  port: 22

3.  salt-ssh '*' test.ping -i
4. salt-ssh '*' -r 'w'
# 执行效果如下:
#---------------------------------------------------------------------------
node2:
    ----------
    retcode:
        0
    stderr:
    stdout:
        root@192.168.2.103's password: 
         01:09:20 up  2:20,  3 users,  load average: 0.10, 0.16, 0.21
        USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
        root     tty1                      07:48     ?     0.38s  0.38s -bash
        root     pts/0    192.168.2.101    23:45    8.00s  0.74s  0.02s /usr/bin/python /usr/bin/salt-ssh * -r w
        root     pts/1    192.168.2.101    23:54   22:08   0.39s  0.39s -bash
node1:
    ----------
    retcode:
        0
    stderr:
    stdout:
        root@192.168.2.104's password: 
         06:56:25 up 11:47,  3 users,  load average: 0.00, 0.04, 0.05
        USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
        root     tty1                      19:09    1:24m  2.27s  2.27s -bash
        root     pts/0    192.168.2.101    05:32   21:37   1.39s  1.39s -bash
        root     pts/1    192.168.2.101    05:42   55:21   1.09s  1.09s -bash

#--------------------------------------------------------------------------- 
```

### 21.  SaltStack 九

```json
# salt-api
https://docs.saltstack.com/en/latest/topics/netapi/writing.html
1. yum install salt-api
2. yum install pyOpenSSL
3. salt-call --local tls.create_self_signed_cert
[ERROR   ] You should upgrade pyOpenSSL to at least 0.14.1 to enable the use of X509 extensions
local:
    Created Private Key: "/etc/pki/tls/certs/localhost.key." Created Certificate: "/etc/pki/tls/certs/localhost.crt."
4. vim  /etc/salt/master.d/api.conf
rest_cherrypy:
  host: 192.168.2.103
  port: 8000
  ssl_cert: /etc/pki/tls/certs/localhost.crt
  ssl_key: /etc/pki/tls/certs/localhost.key
5. useradd -M -s /sbin/nologin saltapi  # 创建用户 -M 不创建home目录 -s 仅做验证
6. echo "saltapi" | passwd saltapi --stdin # 非交互式创建密码
7. vim /etc/salt/master.d/auth.conf
external_auth:
  pam:
    saltapi:
      - .*
      - '@wheel'
      - '@runner'
      - '@jobs'
8. systemctl restart salt-master
9. systemctl start salt-api
10. curl -sSk http://192.168.2.103:8000/login \
	-H 'Accept: application/x-yaml' \
	-H 'X-Auth-Token: xxxxxxxxxxxxxxxxxxxx' \
	-d username='saltapi' \
	-d password='saltapi' \
	-d eauth='pam'

```

### 22.  SaltStack 十

```json
# salt-master高可用  多master
minion配置可写为列表:
master:
  - 192.168.2.103
  - 192.168.2.104

保证两台master配置相同  # nfs文件共享

# nfs 搭建 on 192.168.2.103
yum install nfs-utils rpcbind
 vim /etc/exports
/etc/salt/pki/master  192.168.2.104 *(rw,sync,no_root_squash,no_all_squash)
/srv/salt  192.168.2.104 *(rw,sync,no_root_squash,no_all_squash)
systemctl start nfs
# on 192.168.2.104
mount -t nfs  192.168.2.103:/etc/salt/pki/master /etc/salt/pki/master
mount -t nfs  192.168.2.103:/srv/salt /srv/salt
systemctl start salt-master

```

### 23.  docker

```json

1. installation

https://cr.console.aliyun.com/cn-hangzhou/new


wget https://download.docker.com/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
yum install docker-ce python-pip docker-compose #单机编排工具

systemctl enable docker.service
systemctl start docker.service

docker pull centos
docker pull busybox
docker pull mysql
docker pull nginx
docker pull alpine
docker pull aclstack/mem
docker pull aclstack/cpu
docker pull progrium/consul
docker pull sebp/elk
docker pull fluent/fluentd

2. 配置镜像加速器

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://hr1upp6v.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

3. vim /usr/lib/systemd/system/docker.service
# ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock 修改为
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://192.168.2.103 -H unix:///var/run/docker.sock
# ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0 -H unix:///var/run/docker.sock  
4. docker -H 192.168.2.103 info  # 验证
5. docker search lnmp # 搜索镜像
6. docker run hello-world
7. docker images # 查看所有镜像
#-------------------------------------------------------
             交互模式       别名     镜像名    执行的命令
8. docker run -it --name assasin_centos  centos     bash 

exit/ 
docker run assasin_centos
docker attach assasin_centos
Ctrl + p  q
docker exec -it assasin_centos bash
#-------------------------------------------------------
9. docker ps -a # 查看正在运行的镜像
10. docker rm -f  0b436ff2011d # -f 强制删除
11. docker run -it --name mini_os alpine sh


12. 镜像制作:
	docker run -it --name my_nginx centos bash
[root@1d05506fe6ac yum.repos.d]  rpm -ivh https://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm 
[root@1d05506fe6ac yum.repos.d] yum install nginx -y 
[root@1d05506fe6ac yum.repos.d] yum install php-fpm -y 
[root@1d05506fe6ac yum.repos.d] yum install supervisor  # 多进程管理工具
[root@1d05506fe6ac yum.repos.d] vi /etc/supervisor.conf # file=supervisord.d/*.ini
[root@1d05506fe6ac yum.repos.d] vi php_nginx.ini

[supervisord]
nodaemon=true # 后台模式运行

[program:nginx]
command=/usr/sbin/nginx -g "daemon off;" # 非后台模式启动

[program:phpfpm]
command=/usr/sbin/php-fpm -F -c /etc/php.ini 
autostart = true
startsecs = 3
autorestart = true
startretries = 3
user = root
redirect_stderr = false
stdout_logfile_maxbytes = 50MB
stdout_logfile_backups = 20

[root@1d05506fe6ac yum.repos.d] supervisord 
[root@1d05506fe6ac yum.repos.d]  退出镜像

                                     镜像名/ID  仓库名
	docker commit -m 'assasin nginx' my_nginx nginx:v1

#---------------------------------------------------------------------------
                     端口映射    容器名称         容器名称/ID        命令
13.  docker run -it -p 80:80  --name nginx_v1  1d05506fe6ac   supervisord 

# 提交镜像至官方仓库
 #我的dockerId: assasinshibin xxxxxxxxxxx
 docker login   
 docker tag  1d05506fe6ac 仓库名/镜像名:版本号
 docker push 仓库名/镜像名:版本号
#---------------------------------------------------------------------------
14.  mysql 
# 错误: docker run -it --name mysql -p 8888:3306 mysql
# 报错信息: You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD 
# 正确: docker run -it --name mysql -p 8888:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql

docker run -it --name mysql -p 8888:3306 --rm mysql  # 停止后自动删除 仅限于测试
docker run -it --name mysql -p 8888:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
# 注意:随机密码 在打印信息中
docker run -it --name mysql -p 8888:3306 -e MYSQL_RANDOM_ROOT_PASSWORD=true mysql

#--------------------------------------------------------------------------- 

15. docker run -it --rm -m 50M aclstack/mem mem  # 每秒请求10M内存 go实现
16. docker run --cpuset-cpus 0 -it --rm -c 2048 aclstack/cpu cpu  # 在0号CPU执行 -c 指定权重 默认1024

#---------------------------------------------------------------------------

17. docker run -it -P --name nginx nginx  # -P 端口随机映射
18. docker run -it -p 80:80 --name nginx nginx  # -p tcp端口
19. docker run -it -p 80:80/udp --name nginx nginx  # udp端口
20. docker run -it -p 192.168.1.103:80:80 --name nginx nginx # 指定IP映射
#---------------------------------------------------------------------------
21. 单台容器互联 测试
docker run -it --rm --name busybox1 busybox  # 终端1
ip a

docker run -it --rm --link busybox1:busybox1 --name busybox2 busybox  # 终端2
ping busybox1
#--------------------------------------------------------------------------------
22. docker network ls # docker 三种网络模式
23. docker run -it --net=host nginx # host模式 与宿主机一致
24. docker network create --driver bridge my_net # 自定义网络
# 返回cb380ad40d60f643a4c001d64f7d3d9b0ccdc0dca20294e162c55cbd13c41149
25. dcoker run -it --rm --network-my_net busybox # 创建网络
26. docker network create --driver bridge --subnet 172.22.16.0/24 --gateway 172.22.16.1 my_net2 # 创建指定网络
27. docker run --it --rm --network-my_net2 --ip 172.22.16.88 busybox # 指定IP创建
28. docker run --it --rm --network-my_net2 --ip 172.22.16.88 --name assasin busybox # assasin 必须ping不通
29. docker run --it --rm --network-my_net2 --ip 172.22.16.88 --name shibin busybox
#----------------------------------------------------------------------------------
30. docker network connect my_net2 assasin
#跨主机互通
31. docker run -d -p 8500:8500 --name sonsul progrium/consul -server -bootsrap  # mode-3
32. vim /etc/docker/daemon.json
{
    "registry-mirrors": ["https://hr1upp6v.mirror.aliyuncs.com"],
    "dns":["192.168.2.103","223.55.5"],
    "data-root" :"/data/docker",
    "cluster-store": "consul://192.168.2.105:8500",  // dockerd --help | grep cluster
    "cluster-advertise" "192.168.2.103:2375"
}
33. systemctl daemon-reload
34. 
35. 
36. 
37. 
38.
40. 


#----------------------------------------------------------------------------------
# docker-compose
pip install docker-compose
# 编写容器编排文件
docker-compose.yml

web1:
  image: nginx
  volumes:
    - /opt/index1.html:/usr/share/nginx/html/index.html
  expose:
    - 80

web2:
  image: nginx
  volumes:
    - /opt/index2.html:/usr/share/nginx/html/index.html
  expose:
    - 80

haproxy:
  image: haproxy
  volumes:
    - /opt/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg
  links:
    - web1
    - web2
  ports:
    - "7777:1080"
    - "80:80"

# haproxy.cfg
global
  log 127.0.0.1 local0
  log 127.0.0.1 local1 notice
   maxconn 4096

defaults
  log global
  mode http
  option httplog
  option dontlognull
  timeout connect 5000ms
  timeout client 5000ms
  timeout server 5000ms

listen stats
  bind 0.0.0.0:1080
  mode http
  stats enable
  stats hide-version
  stats uri /stats
  stats auth admin:admin

frontend balance
  bind 0.0.0.0:80
  default_backend web_backends

...... 未完

# 启动:  docker-compose -f docker-compose.yml up -d # 后台启动
# 停止一台: docker stop opt_web1_1
# 启动: docker start opt_web1_1
#----------------------------------------------------------------------------------



```

### 24.  

```json

```

### 25.  

```json

```

### 26.  

```json

```

### 27.  

```json

```

### 28.  

```json

```

### 29.  

```json

```

### 30.  

```json

```

### 31.  

```json

```

### 32.  

```json

```

### 33.  

```json

```

### 34.  

```json

```

### 35.  

```json

```

### 36.  

```json

```

### 37.  

```json

```

### 38.  

```json

```

### 39.  

```json

```

### 40.  

```json

```

### 41.  

```json

```

### 42.  

```json

```

### 43.  

```json

```

### 44.   

```json

```

### 45.  

```json

```

### 46.  

```json

```

### 47.  

```json

```

### 48.  

```json

```

### 49.  

```json

```

### 50.  

```json

```

### 51.  

```json

```

### 52.  

```json

```

### 53. 

```json

```

### 54. 

```json

```

### 55. 

```json

```

### 56. 

```json

```

### 57. 

```json

```

### 58. 

```json

```

### 59. 

```json

```

### 60. 

```json

```

### 61. 

```json

```

### 62. 

```json

```

### 63. 

```json

```

### 44. 

```json

```

### 65. 

```json

```

### 66. 

```json

```

### 67. 

```json

```

### 68. 

```json

```

### 69. 

```json

```

### 70. 

```json

```

### 71. 

```json

```

### 72. 

```json

```

### 73. 

```json

```

### 74. 

```json

```

### 75. 

```json

```

### 76. 

```json

```

### 77. 

```json

```

### 78. 

```json

```

### 79. 

```json

```

### 70. 

```json

```

### 71. 

```json

```

### 72. 

```json

```

### 73. 

```json

```

### 74. 

```json

```

### 75. 

```json

```

### 76. 

```json

```

### 77. 

```json

```

### 78. 

```json

```

### 79. 

```json

```

### 80. 

```json

```

### 81. 

```json

```

### 82. 

```json

```

### 83. 

```json

```

### 84. 

```json

```

### 85. 

```json

```

### 86. 

```json

```

### 87. 

```json

```

### 88. 

```json

```

### 89. 

```json

```

### 90. 

```json

```

### 91. 

```json

```

### 92. 

```json

```

### 93. 

```json

```

### 94. 

```json

```

### 95. 

```json

```

### 96. 

```json

```

### 97. 

```json

```

### 98. 

```json

```

### 99. 

```json

```

### 100. 

```json

```

