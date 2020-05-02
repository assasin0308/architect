# architect
#### architect

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
```

### 2.

```json

```

### 3.

```json

```

### 4.

```json

```

### 5.

```json

```

### 6.

```json

```

### 7.

```json

```

### 8.

```json

```

### 9.

```json

```

### 10.

```json

```

### 11.

```json

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

