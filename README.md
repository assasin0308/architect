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

```

### 12.  

```json

```

### 13.  

```json

```

### 14.  

```json

```

### 15.  

```json

```

### 16.  

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

