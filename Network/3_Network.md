# 网络层 #

---

[toc]

---

---

## IP 协议 ##

---

### IP协议的定义和作用 ###

> IP 协议（Internet Protocol）又称互联网协议，是支持网间互联的数据包协议。

- 该协议工作在网络层，主要目的就是为了提高网络的可扩展性
- 和传输层 TCP 相比，IP 协议提供一种无连接/不可靠、尽力而为的数据包传输服务
- 网络层协议(IP)负责提供主机间的逻辑通信；运输层协议(TCP等)负责提供进程间的逻辑通信
- 作用:
    - **寻址和路由**：在IP 数据包中会携带源 IP 地址和目的 IP 地址来标识该数据包的源主机和目的主机。IP 数据包在传输过程中，每个中间节点（IP 网关、路由器）只根据网络地址进行转发
        - 如果中间节点是路由器，则路由器会根据路由表选择合适的路径。
        - IP 协议根据路由选择协议提供的路由信息对 IP 数据报进行转发，直至抵达目的主机
    - **分段与重组**：IP 数据包在传输过程中可能会经过不同的网络，在不同的网络中数据包的最大长度限制是不同的，IP 协议通过给每个 IP 数据包分配一个标识符以及分段与组装的相关信息，使得数据包在不同的网络中能够传输，被分段后的 IP 数据报可以独立地在网络中进行转发，在到达目的主机后由目的主机完成重组工作，恢复出原来的 IP 数据包

---

---

## 路由 ##

---

### 什么是路由器, 和交换机有什么区别 ###

| 路由器                                                       | 交换机                                               |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| 工作于网络层                                                 | 工作于数据链路层                                     |
| 通过数据包中的目的 IP 地址识别不同的网络从而确定数据转发的目的地址 | 利用主机的物理地址（MAC 地址）确定数据转发的目的地址 |

---

### 路由器的分组转发流程 ###

```c
从 IP 数据包中提取出目的主机的 IP 地址，找到其所在的网络
    
if(目的 IP 地址所在的网络与本路由器直接相连){
    不需要经过其它路由器直接交付
}
else if(路由表中有目的 IP 地址的特定主机路由){
    按照路由表传送到下一跳路由器中
}
else if(逐条检查路由表, 找到匹配路由){
    按照路由表转发到下一跳路由器中
}
else if(路由表中设置有默认路由){
    照默认路由转发到默认路由器中
}
else{
    无法找到合适路由，向源主机报错
}
```

---

---

## 地址解析 ##

---

### ARP 的原理和过程 ###

