# 网络层的路由器转发策略

## 划分子网方法
从网络的主机号借用若干位作为子网号，当然主机号也就减少了同样的位数。于是两级IP地址就变成了
**三级IP**地址：
    IP地址 = {<网络号>, <子网号>, <主机号>}

## 如何获取子网的网络地址
现在剩下的问题就是：假定有一个数据报(其目标地址是145.13.3.10)已经到达了路由器。那这个路由器
如何把它转发到对应的子网呢？

我们知道，从IP数据报的首部**无法看出**源主机或者目的主机所连接的网络是否进行了子网的划分。这
是因为32位的IP地址本身及数据报的首部都没有包含任何子网划分的信息。因此必须另外想办法，这就是
使用**子网掩码(subnet mask)**，如下:
```
    (a)两级IP地址       145       . 13        . 3         . 10
    
    (b)两级IP地址       1111 1111   1111 1111   0000 0000   0000 0000 
       的子网掩码
    
                      |<--两级IP地址的网络号-->|<--子网号-->|<--主机号-->|
    (c)三级IP地址       145       . 13        . 3         . 10         
                      |<-----子网号为3的网络的网络号------->|<--主机号-->|
                      
    (d)三级IP地址
       的子网掩码      1111 1111    1111 1111   1111 1111   0000 0000
       
    (e)子网的网络
       地址             145       . 13       . 3         . 0
```
图中c是同一地址的三级IP地址结构，也就是说，现在从原来的16位主机号中拿出8位作为子网号，而主机号
从原来的16位减少到8位。
为了使路由器能够很方便的从数据报中的IP地址中提取出所要找的子网的网络地址，路由器就要使用三级IP
地址的子网掩码，如图中d，跟收到的数据报的目的IP地址145.13.3.10进行逐位的**与**运算(AND),就立即
得出是要找的**子网的网络地址145.13.3.0**

为了更好理解，可以再看个例子：已知IP地址是141.14.72.24，子网掩码是255.255.192.0，试求网络地址。
掩码的前两个字节全是1，因此网络地址的前两个字节可写为141.14。子网掩码的第四个字节是全0，因此网络
地址的第四个字节是0，接下来再分析
```
    (a)三级IP地址       141       . 14        . 72        . 24
    
    (b)IP地址的第 
       三个字节用       141       . 14        .0100 1000  . 24
       二进制表示
    
    (c)子网掩码是       1111 1111   1111 1111  1100 0000   0000 0000
    255.255.192.0                                
    
    (d)IP地址与子
       网掩码逐位       141       . 14        .0100 0000  . 0
       相与
       
    (e)子网的网络
       地址            141       . 14       . 64         . 0
```

## 路由器转发算法
在使用子网划分后，路由器的路由表中必须包含以下三项内容：目的网络地址、子网掩码和下一跳地址

路由器的转发分组算法如下：
-1 从收到的数据包的首部提取目的IP地址D
-2 先判断是否为直接交付。对路由器直接相连的网络进行逐个检查：用各网络的子网掩码和D逐位相**与**
   (AND操作)，看结果是否和相应的网络地址匹配。若匹配，则把分组进行直接交付(当然还需要把D转换成
   物理地址，把数据报封装成帧发送出去)，转发任务结束。否则就是间接交付，执行(3)
-3 若路由表中有目的地址为D的特定主机路由，则把数据报传送给路由表中所指定的下一跳路由器；否则，
   执行(4)
-4 对路由表中的每一行(目的网络地址、子网掩码、下一跳地址)，用其中的子网掩码和D逐位相**与**，
   其结果位N。若N与该行的目的网络地址匹配，则把 数据报传送给该行指明的下一跳路由器，否则，执行
   (5)
-5 若路由表中有个默认路由，则把数据报传送给路由器中所指明的默认路由器；否则，执行(6)
-6 报告转发分组出错

以上算法的核心在于第4步！