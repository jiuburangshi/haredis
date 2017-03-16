# Redis HA - sentinel + haproxy
# 首先感谢原作者的指引：https://github.com/falsecz/haredis

- sentinel: autoswitch master/slave
- haproxy: active check for only master node
- haproxy api: disable failed nodes to prevent multimaster

### 图中只启用了一个哨兵，而本项目中，在每个redis节点都将启动一个哨兵
!["diagram"](diagram.png)

without haproxy maintenance mode
```
  A is master, B is slave
  A crashes, sentinel convert B to master
  A is recovered (still master)
  haproxy balancing between A and B, until sentinel convert A to slave
  data written to A are lost
```

witch haproxy maintenance mode via notification script
```
  A is master, B is slave
  A crashes, sentinel convert B to master
  haproxy disable A
  A is recovered (still master) but disabled in haproxy
  haproxy points only to B
```

### 按以下方法实现，可kill 任意redis节点，无论主备，只要有一台启动，都将推举出master节点

### Run redis cluster 
### 在三台不同host A, B, C上执行
### 假设: 
```
A: 10.10.10.185
B: 10.10.10.186
C: 10.10.10.106
```
```
redis-server --port 6666
redis-server --port 6667
redis-server --port 6668
```

### Set slaves
### 在单台host A上执行
```
redis-cli -h 10.10.10.186 -p 6667 SLAVEOF 10.10.10.185 6666
redis-cli -h 10.10.10.106 -p 6668 SLAVEOF 10.10.10.185 6666
```

### Run sentinel
### 分别在三台host A, B, C执行，启用哨兵模式
```
redis-server sentinel.conf  --sentinel
```

### Run haproxy
### 在某台host A上运行haproxy
```
haproxy -f haproxy.cfg -db
```

Open http://10.10.10.185:8080/ and try kill some redis


### Notice
tested on
- redis 3.0.7
- haproxy 1.5.18

[!] in production on single host you must specify different data dir before SLAVEOF command otherwise you loose data on master
