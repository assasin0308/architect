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


6-- 10:25
```

### 14.  SaltStack 二

```json

```

### 15.  SaltStack 三

```json

```

### 16.  SaltStack 四

```json

```

### 17.  

```json

```

### 18.  

```json

```

### 19.  

```json

```

### 20.  

```json

```

### 21.  

```json

```

### 22.  

```json

```

### 23.  

```json

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

