---
layout: post
title:  "简单原理实现nginx的负载均衡"
categories: Nginx
tags: Nginx
---

* content
{:toc}

nginx 简单实践系列，主要简单实践了负载均衡，仅供参考。








一、关于负载均衡
--------

### 1.1 简介

**负载均衡（Load Balance，简称 LB）**是高并发、高可用系统必不可少的关键组件，目标是**尽力将网络流量平均分发**到多个服务器上，以提高系统整体的响应速度和可用性。

主要的作用：

*   **高并发：**负载均衡通过算法调整负载，**尽力均匀的分配**应用集群中各节点的工作量，以此提高应用集群的并发处理能力（吞吐量）。
*   **伸缩性：**添加或减少服务器数量，然后由负载均衡进行分发控制。这使得应用集群具备伸缩性。
*   **高可用：**负载均衡器可以**监控候选服务器**，当服务器不可用时，自动跳过，将请求分发给可用的服务器。这使得应用集群具备高可用的特性。
*   **安全防护：**有些负载均衡软件或硬件提供了安全性功能，如：黑白名单处理、防火墙，防 DDos 攻击等。

### 1.2 载体维度分类：硬件

硬件负载均衡，一般是在定制处理器上运行的，**独立负载均衡服务器**，**价格昂贵**。

硬件负载均衡的主流产品有:F5 和 A10。F5 拥有更广泛的附加功能和服务，而 A10 则在某些特定领域如 IPv6 转换或 SSL 检查方面表现出色。

优点：

*   功能强大：支持全局负载均衡并提供较全面的、复杂的负载均衡算法。
*   性能强悍：硬件负载均衡由于是在专用处理器上运行，因此吞吐量大，可支持单机百万以上的并发。
*   安全性高：往往具备防火墙，防 DDos 攻击等安全功能。

缺点：

*   成本昂贵：购买和维护硬件负载均衡的成本都很高。
*   扩展性差：当访问量突增时，超过限度不能动态扩容。

### 1.3 载体维度分类：软件

软件负载均衡是指，**使用软件工具或解决方案来分配网络流量或计算任务到多个服务器**，以实现资源的有效利用、提高系统性能和增强服务的可用性。

与硬件负载均衡器相比，软件负载均衡**更加灵活、成本效益更高，并且可以轻松部署在标准的服务器硬件上**。

软件负载均衡通常位于客户端和服务器之间，它接收来自客户端的请求，并根据预先设定的算法决定将这些请求转发到哪一个后端服务器。通过这种方式，它可以确保工作负载均匀分布，避免任何单一服务器过载。

优点：

*   扩展性好：适应动态变化，可以通过添加软件负载均衡实例，动态扩展到超出初始容量的能力。
*   成本低廉：软件负载均衡可以在任何标准物理设备上运行，降低了购买和运维的成本。

缺点：

*   性能略差：相比于硬件负载均衡，软件负载均衡的性能要略低一些。

### 1.4 网络通信分类：七层负载均衡

#### 1.4.1 DNS 负载均衡

**DNS（Domain Name System 域名解析系统）负载均衡一般用于互联网公司，复杂的业务系统不适合使用。大型网站一般使用 DNS 负载均衡作为第一级负载均衡手段，然后在内部使用其它方式做第二级负载均衡。**

什么是 DNS？它是 OSI 第七层网络协议。DNS 被设计为一个树形结构的分布式应用，自上而下依次为：权威服务器，顶级服务器，根域名服务器，递归服务器，本地DNS服务器。

下边简单列举下：

<table style="border-collapse: collapse; width: 100%; height: 126px" border="1"><tbody><tr style="height: 21px"><td style="width: 23.6364%; height: 21px">DNS服务器的种类</td><td style="width: 31.1262%; height: 21px">&#160;</td><td style="width: 45.1839%; height: 21px">&#160;</td></tr><tr style="height: 21px"><td style="width: 23.6364%; height: 21px">本地DNS服务器<br>（Local DNS Server）</td><td style="width: 31.1262%; height: 21px">通常是用户设备或网络中的默认 DNS 服务器，它接收用户的 DNS 查询请求。</td><td style="width: 45.1839%; height: 21px">首先检查自己的缓存中是否已有该域名的对应IP地址。如果有，直接返回结果；如果没有，则向上级 DNS 服务器发送查询请求。</td></tr><tr style="height: 21px"><td style="width: 23.6364%; height: 21px">递归域名服务器<br>（Recursive DNS Server）</td><td style="width: 31.1262%; height: 21px">主要任务是代表客户端执行完整的解析过程，从而简化了客户端的工作流程。</td><td style="width: 45.1839%; height: 21px">通过递归方式完成查询。这意味着如果本地没有所需的信息，它会依次向根域名服务器、顶级域名服务器以及权威域名服务器发起查询，直到找到目标域名对应的 IP 地址。</td></tr><tr style="height: 21px"><td style="width: 23.6364%; height: 21px">根域名服务器<br>（Root DNS Server）</td><td style="width: 31.1262%; height: 21px">是整个 DNS 系统的最顶层，提供顶级域名（TLD）服务器的信息。</td><td style="width: 45.1839%; height: 21px">当本地 DNS 服务器无法在缓存中找到答案时，它会向根域名服务器发出查询请求。根域名服务器不会直接提供最终的 IP 地址，而是返回负责特定顶级域名（如 .com、.org 等）的顶级域名服务器的地址。</td></tr><tr style="height: 21px"><td style="width: 23.6364%; height: 21px">顶级域名服务器<br>（Top-Level Domain DNS Server, TLD Server）</td><td style="width: 31.1262%; height: 21px">管理特定顶级域名下的所有子域名信息。</td><td style="width: 45.1839%; height: 21px">根据来自本地 DNS 服务器的请求，顶级域名服务器会提供负责特定二级域名（例如 example.com 中的 example 部分）的权威域名服务器的地址。</td></tr><tr style="height: 21px"><td style="width: 23.6364%; height: 21px">权威域名服务器<br>（Authoritative DNS Server）</td><td style="width: 31.1262%; height: 21px">存储并维护特定域名的实际 DNS 记录，包括 A 记录（指向IPv4地址）、AAAA 记录（指向 IPv6 地址）、MX 记录（邮件服务器信息）等。</td><td style="width: 45.1839%; height: 21px">这是最后一个层级的 DNS 服务器，它能提供所查询域名的确切 IP 地址或其他所需信息。一旦本地 DNS 服务器接收到权威域名服务器提供的答案，就会将该信息返回给客户端，并且通常会在本地缓存一段时间以供后续查询使用。</td></tr></tbody></table>

DNS 负载均衡的工作原理就是：**基于 DNS 查询缓存，按照负载情况返回不同服务器的 IP 地址。**

如下图，针对服务集群中的两个服务地址 127.0.0.1、127.0.0.2，在 DNS 服务器中都配置为指向域名 www.xxx.com，当用户通过域名访问时，请求到 DNS 服务器，然后随机返回一个实际请求的服务器 IP 地址。


优点：

**简易部署与管理：**无需改变应用程序的代码，仅通过配置DNS即可实现负载分发，简化了服务器集群的管理。  
**地理分布优化：**支持基于地理位置的流量分配，能够将用户请求定向到距离最近的数据中心或服务器，从而减少延迟，提高访问速度。  
**扩展性强：**可以轻松地向服务器池中添加新的服务器，以应对增加的流量需求，提高了系统的可伸缩性。  
**成本效益高：**通过更有效的资源利用减少了硬件投资和运营成本。

缺点：

**缓存问题：**由于DNS解析结果会被客户端及中间DNS服务器缓存，当某个服务器出现故障时，已缓存该服务器IP地址的用户仍然可能被导向至故障服务器，直到缓存过期。这可能导致服务中断或性能下降。  
**负载平衡精度有限：**DNS负载均衡通常基于简单的算法（如轮询）进行分配，无法实时感知各服务器的实际负载情况，导致负载分配不够精准。  
**安全性考虑：**尽管DNSSEC等技术增强了DNS的安全性，但DNS本身仍可能存在被攻击的风险，例如DDoS攻击，可能会对整个系统造成影响。  
**不支持会话持久化：**如果应用需要保持用户的会话状态，单纯的DNS负载均衡无法保证用户在多次请求间总是被导向到同一台服务器，除非结合其他机制来实现。

虽然 DNS 负载均衡提供了一种简单且有效的方法来分散网络流量，但它也存在一定的局限性。因此，在实际应用中，**常常需要结合使用其他技术和策略，如健康检查、自动故障转移以及会话保持机制等**，以克服其固有的不足。

#### 1.4.2 HTTP 负载均衡

HTTP 负载均衡是基于 HTTP 重定向实现的。HTTP 负载均衡属于七层负载均衡。

HTTP 重定向原理是：根据用户的 HTTP 请求计算出一个真实的服务器地址，将该服务器地址写入 HTTP 重定向响应中，返回给浏览器，由浏览器重新进行访问。


优点：

高可用性和可靠性：即使某台服务器发生故障，负载均衡器也能自动将流量分配到其他健康的服务器上，从而保证服务的连续性。  
扩展性：可以轻松添加更多的服务器到池中以应对增加的负载，提高了系统的可伸缩性。  
性能提升：通过智能地选择最佳的服务器来处理每个请求，可以减少响应时间和提高用户体验。  
安全性增强：负载均衡器可以充当额外的安全层，提供诸如 SSL 终止、DDoS 防护等功能，保护后端服务器免受攻击。  
会话持久化支持：一些高级负载均衡器支持会话持久化功能，确保用户在整个会话过程中总是被导向到同一台服务器。

缺点：

**性能较差：**每次访问需要两次请求服务器，增加了访问的延迟。  
**降低搜索排名：使用重定向后，搜索引擎会视为 SEO 作弊。**  
复杂性和成本：实现和维护 HTTP 负载均衡需要一定的技术知识和投入，包括硬件成本、软件许可证费用以及维护成本。  
单点故障风险：如果负载均衡器本身出现故障，则可能导致整个系统不可用。虽然可以通过设置主备负载均衡器来缓解这一问题，但这增加了架构的复杂度。  
潜在的瓶颈：在高并发情况下，如果负载均衡器本身的处理能力不足，可能会成为系统的瓶颈。

由于其缺点比较明显，所以这种负载均衡策略实际应用较少。

#### 1.4.3 反向代理

反向代理（Reverse Proxy）方式是指以 代理服务器 来接受网络请求，然后 将请求转发给内网中的服务器，并将从内网中的服务器上得到的结果返回给网络请求的客户端。反向代理负载均衡属于七层负载均衡。

反向代理服务的主流产品：**Nginx、Apache**。

如下图，看下 Nginx 如何实现负载均衡：


首先，在代理服务器上设定好负载均衡规则。

然后，当收到客户端请求，反向代理服务器拦截指定的域名或 IP 请求，根据负载均衡算法，将请求分发到候选服务器上。

其次，如果某台候选服务器宕机，**反向代理服务器会有容错处理**，比如分发请求失败 3 次以上，将请求分发到其他候选服务器上。

优点：

**多种负载均衡算法：**支持多种负载均衡算法，以应对不同的场景需求。  
**可以监控服务器：**基于 HTTP 协议，可以监控转发服务器的状态，如：系统负载、响应时间、是否可用、连接数、流量等，从而根据这些数据调整负载均衡的策略。

缺点：

**额外的转发开销：**反向代理的转发操作本身是有性能开销的，可能会包括创建连接，等待连接响应，分析响应结果等操作。  
**增加系统复杂度：**反向代理常用于做分布式应用的水平扩展，但反向代理服务存在以下问题，为了解决以下问题会给系统整体增加额外的复杂度和运维成本。

**反向代理服务如果自身宕机，就无法访问站点，所以需要有高可用方案**，常见的方案有：主备模式（一主一备）、双主模式（互为主备）。

反向代理服务自身也存在性能瓶颈，**随着需要转发的请求量不断攀升，需要有可扩展方案**。

### 1.5 网络通信分类：四层负载均衡

#### 1.5.1 IP 负载均衡

IP 负载均衡是在网络层通过修改请求目的地址进行负载均衡。


如上图，大致流程为：

1\. 客户端请求 192.168.137.10，由负载均衡服务器接收到报文。  
2\. 负载均衡服务器根据算法选出一个服务节点 192.168.0.1，然后将报文请求地址改为该节点的 IP。  
3\. 真实服务节点收到请求报文，处理后，返回响应数据到负载均衡服务器。  
4\. 负载均衡服务器将响应数据的源地址改负载均衡服务器地址，返回给客户端。

IP 负载均衡在内核进程完成数据分发，**较反向代理负载均衡有更好的从处理性能**。但是，由于所有请求响应都要经过负载均衡服务器，**集群的吞吐量受制于负载均衡服务器的带宽**。

#### 1.5.2 数据链路层通过修改 mac 地址来实现负载均衡

数据链路层负载均衡是指，**在通信协议的数据链路层修改 mac 地址**进行负载均衡。


在 Linux 平台上最好的链路层负载均衡开源产品是 LVS (Linux Virtual Server)。

LVS 是基于 Linux 内核中 netfilter 框架实现的负载均衡系统。netfilter 是内核态的 Linux 防火墙机制，可以在数据包流经过程中，根据规则设置若干个关卡（hook 函数）来执行相关的操作。

LVS 的工作流程大致如下：

1\. 客户端请求：当客户端发起一个请求到目标服务时，首先会到达 LVS 配置的虚拟 IP 地址（VIP），这个 VIP 对外代表了一个或多个实际提供服务的服务器集群。  
2\. 流量分发的四种模式：  
 1）IPVS 模块：LVS 的核心是运行在 Linux 内核中的 IPVS（IP Virtual Server）模块，它负责根据预设的负载均衡算法（如轮询、最少连接数等）将收到的请求分发给后端的真实服务器（Real Server, RS）。  
 2）直接路由模式（DR：Direct Routing）：在此模式下，LVS 简单地重写数据包的目标 MAC 地址为选定真实服务器的 MAC 地址，并将数据包发送回交换机，由交换机转发给真实服务器。这种方式不需要修改数据包的 IP 头部信息，因此效率很高。  
 3）NAT 模式（Network Address Translation）：在这种模式下，LVS 不仅改变目标 MAC 地址，还会修改目标 IP 地址为目标服务器的实际 IP 地址，并且处理来自真实服务器的响应，将其源 IP 地址改为 VIP 后再返回给客户端。此方法适用于真实服务器位于私有网络的情况。  
 4）隧道模式（Tunnel：Tunneling）：通过将原始请求封装在 IP 隧道中发送给真实服务器，允许真实服务器位于不同的物理位置，只要它们能够与 LVS 通信即可。  
3\. 真实服务器处理请求：被选中的真实服务器接收到请求后进行处理，并直接将响应发送回客户端（在直接路由和隧道模式下），或者通过 LVS 返回给客户端（在 NAT 模式下）。  
4\. 健康检查：LVS通常结合外部工具（如keepalived）来进行真实服务器的健康检查。如果某个真实服务器不可用，LVS将停止向其分配新的请求，直到它恢复正常。

### 1.6 负载均衡算法简介

负载均衡器的实现可以分为两个部分：首先通过负载均衡算法在候选服务器列表选出一个服务器;然后，将请求数据发送到该服务器上。

负载均衡算法是负载均衡服务核心中的核心。

负载均衡产品多种多样，但是各种负载均衡算法原理是共性的。负载均衡算法有很多种，分别适用于不同的应用场景，常用的有：轮询、随机、最小活跃数、源地址哈希、一致性哈希等等，下文将逐个简单介绍。

*   **随机算法（Random）**

随机算法将请求随机分发到候选服务器。更适合服务器硬件相同的场景。当调用量较小的时候，可能负载并不均匀，调用量越大，负载越均衡。

*   **加权随机算法（Weighted Random）**

加权随机算法在随机算法的基础上，按照概率调整权重，进行负载分配。

*   **轮询算法（Round Robin）**

轮询算法的策略是：将请求依次分发到候选服务器。

该算法适合场景：各服务器处理能力相近，且每个事务工作量差异不大。如果存在较大差异，那么处理较慢的服务器就可能会积压请求，最终无法承担过大的负载。

*   **加权轮询算法（Weighted Round Robbin）**

加权轮询算法在轮询算法的基础上，增加了权重属性来调节转发服务器的请求数目。

性能高、处理速度快的节点应该设置更高的权重，使得分发时优先将请求分发到权重较高的节点上。

*   **最少连接数（Least Connections）**

最少连接数算法将请求分发到连接数/请求数最少的候选服务器（目前处理请求最少的服务器）。

它是根据候选服务器当前的请求连接数，动态分配。

适用于**对系统负载较为敏感或请求连接时长相差较大的场景**。

由于每个请求的连接时长不一样，如果采用简单的轮循或随机算法，都可能出现某些服务器当前连接数过大，而另一些服务器的连接过小的情况，这就造成了负载并非真正均衡。虽然，轮询或随机算法都可以通过加权重属性的方式进行负载调整，**但加权方式难以应对动态变化**。

*   **加权最少连接数（Weighted Least Connections）**

结合了加权轮询和最少连接数的优点，在选择服务器时既考虑了服务器的当前连接数也考虑了其权重，**使得更强大的服务器可以处理更多的请求**。

*   **源 IP 地址哈希（Source IP Hash）**

根据客户端 IP 地址计算哈希值，并根据哈希结果选择服务器。

这种方式可以**确保来自同一客户端的请求总是被发送到相同的服务器上**，有助于保持会话状态的一致性，用来实现会话粘滞（Sticky Session）。

*   **一致性哈希（Consistent Hash）**

一致性哈希算法的目标是：相同的请求尽可能落到同一个服务器上。

一致性哈希可以很好的解决稳定性问题，可以将所有的存储节点排列在首尾相接的 Hash 环上，每个 key 在计算 Hash 后会顺时针找到临接的存储节点存放。而当有节点加入或退出时，仅影响该节点在 Hash 环上顺时针相邻的后续节点。

‘相同的请求’是指：一般在使用一致性哈希时，需要指定一个 key 用于 hash 计算，可能是：用户 ID、请求方 IP、请求服务名称、参数列表构成的串等等。

‘尽可能’是指：服务器可能发生上下线，少数服务器的变化不应该影响大多数的请求。当某台候选服务器宕机时，原本发往该服务器的请求，会基于虚拟节点，平摊到其它候选服务器，不会引起剧烈变动。

优点：加入和删除节点只影响哈希环中顺时针方向的相邻的节点，对其他节点无影响。

缺点：加减节点会造成哈希环中部分数据无法命中。当使用少量节点时，节点变化将大范围影响哈希环中数据映射，不适合少量数据节点的分布式方案。普通的一致性哈希分区在增减节点时需要增加一倍或减去一半节点才能保证数据和负载的均衡。

二、Nginx 的负载均衡简单测试
-----------------

### 2.1 轮询【默认方式】

轮询算法是默认的一种方式。

**当 nginx 接收到请求后，逐一分配到所配置的服务列表进行访问并返回。**

**当某个服务异常时，自动跳过，并不会导致请求失败。**

实例配置如下：

```nginx
# 反向代理配置upstream server_shanky {    server www.shanky.cn:5001;    server www.shanky.cn:5021;    server www.shanky.cn:5031;} server {    listen 8888;    server_name www.shanky.cn;     access_log  /var/log/nginx/access.log;    error_log  /var/log/nginx/error.log;     location / {            index index.html index.htm;            proxy_pass http://server_shanky; # 指定反向代理服务器列表    }}
```

刷新同一地址，请求的服务不同。

### 2.2 weight【权重】

权重用于调整不同服务器之间请求分配比例。

它允许管理员根据每台服务器的实际处理能力和性能，来手动控制流量分配，从而优化资源利用和提高系统的整体效率。

通过合理设置权重，可以根据实际需求动态调整服务器之间的负载分布，不仅提高了系统的灵活性和可靠性，还能更好地应对突发流量和资源限制等问题。

配置示例：

```nginx
# 反向代理配置upstream server_shanky {    server www.shanky.cn:5001 weight=1; # 默认为 1，可不配置    server www.shanky.cn:5021 weight=3;    server www.shanky.cn:5031 weight=6;}
```

解释：例如示例中的权重总量为 10，那么如果 nginx 接到十个请求，那么三个服务就分别接到 1、3、6 个请求。但是首次请求进入那个服务是不固定的，权重为 6 的服务概率最大。

实际效果就类似于：（但是顺序是随机的）

```nginx
# 反向代理配置upstream server_shanky {    server www.shanky.cn:5001;    server www.shanky.cn:5021;    server www.shanky.cn:5021;    server www.shanky.cn:5021;    server www.shanky.cn:5031;    server www.shanky.cn:5031;    server www.shanky.cn:5031;    server www.shanky.cn:5031;    server www.shanky.cn:5031;    server www.shanky.cn:5031;}
```

此策略比较适合**服务器的硬件配置差别比较大**的情况。

### 2.3 ip\_hash【基于客户端 IP 来分配】

指定负载均衡器按照基于客户端 IP 的分配方式，这个方法**确保了相同的客户端的请求一直发送到相同的服务器**，以保证 session 会话。**这样每个访客都固定访问一个后端服务器，可以解决 session 不能跨服务器的问题。**

Nginx 使用的 Jenkins hash 函数对输入数据（如 IP 地址）进行哈希计算，生成一个 32 位无符号整数作为哈希值。对于 IP 地址 127.0.0.1，经过 Jenkins hash 计算后，得到的哈希值为 879045407。

如果有三个服务且权重均为1，则通过哈希值对3取余，得到的 0、1、2 分别对应三个服务。

如果有多个权重不同的服务，则通过对权重总和的值取余，得到序号，对应到不同的服务上。

实例配置：

```nginx
# 反向代理配置upstream server_shanky {    ip_hash; # 保证每个客户端访问同一个后端服务    server www.shanky.cn:5001;    server www.shanky.cn:5021;    server www.shanky.cn:5031;}
```

_注意：在 nginx 版本 1.3.1 之前，不能在 ip\_hash 中使用权重（weight）。_

_ip\_hash **不能与 backup（配置备用服务）同时使用**。_

_此策略适合有状态服务，比如 session。_

_当有服务器需要剔除，必须**手动进行停服操作**。_

### 2.4 least\_conn【最少连接】

**把请求转发给连接数较少的后端服务。**

轮询算法是把请求平均的转发给各个后端，使它们的负载大致相同；但是，**有些请求占用的时间很长，会导致其所在的后端负载较高**。这种情况下，least\_conn 这种方式就可以达到更好的负载均衡效果。

```nginx
# 反向代理配置upstream server_shanky {    least_conn; # 把新的请求，转发到当前连接数最少的服务    server www.shanky.cn:5001;    server www.shanky.cn:5021;    server www.shanky.cn:5031;}
```

**此负载均衡策略适合请求处理时间长短不一造成服务器过载的情况。**

### 2.5 第三方策略：fair【按照最短响应时间分配请求】

此方式需要单独额外的配置，不能直接在 upstream 模块添加关键字 fair。

**大概的思路就是，下载补充包，通过 ./configure 命令，将模块添加到已安装的 nginx，重启下服务就可以使用关键字 fair 了。**

#### 2.5.1 配置步骤和示例

1）先下载第三方包：[https://github.com/gnosek/nginx-upstream-fair](https://github.com/gnosek/nginx-upstream-fair "https://github.com/gnosek/nginx-upstream-fair")


需要将此文件夹放到 linux 系统中，本示例复制到路径：/usr/local/nginx-upstream-fair-master。

2）进入 nginx 源文件所在的文件夹（即 nginx 安装包解压后的文件夹），本示例的是：/usr/local/nginx-1.20.0。

```nginx
# 下载 nginx 包并解压wget http://nginx.org/download/nginx-1.20.0.tar.gztar -zxf nginx-1.20.1.tar.gz && cd nginx-1.20.0/# 其他一些可能用到的依赖包（按需安装）yum -y install gcc gcc-c++ openssl openssl-devel zlib zlib-devel pcre pcre-devel make cmake gperftools perl-devel  gd-devel libxml2 libxml2-dev libxslt-devel  redhat-rpm-config.noarch
```

_注意：如果源文件找不到了，需要重新下载一个相同版本安装包使用。_

3）执行命令添加模块：

```nginx
# 手动添加模块./configure --add-module=/usr/local/nginx-upstream-fair-master# 重新进行编译[root@www nginx-1.20.0]# make# 添加成功，注意 configure arguments 配置：[root@www sbin]#  nginx -Vnginx version: nginx/1.20.0built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) configure arguments: --add-module=/usr/local/nginx-upstream-fair-master
```

4）然后先停止 nginx 服务，再将 nginx 进行复制：

```nginx
# 先停止 nginx 服务[root@www sbin]# ./nginx -s stop# 复制 nginx[root@www nginx-1.20.0]# cp -rp objs/nginx /usr/local/nginx/sbin/nginxcp: overwrite ‘/usr/local/nginx/sbin/nginx’? yes[root@www nginx-1.20.0]#
```

最后，修改 nginx.conf 配置文件，开启服务测试一下。

**配置示例：**

```nginx
# 反向代理配置upstream server_shanky {    fair; # 按照最短响应时间分配请求    server www.shanky.cn:5001;    server www.shanky.cn:5021;    server www.shanky.cn:5031;}
```

#### 2.5.2 make 执行时报错：‘ngx\_http\_upstream\_srv\_conf\_t’ has no member named ‘default\_port’

ngx\_http\_upstream\_srv\_conf\_t’没有名为‘default\_port’的成员。

```nginx
# 具体提示：/usr/local/nginx-upstream-fair-master/ngx_http_upstream_fair_module.c: In function ‘ngx_http_upstream_init_fair_rr’:/usr/local/nginx-upstream-fair-master/ngx_http_upstream_fair_module.c:543:28: error: ‘ngx_http_upstream_srv_conf_t’ has no member named ‘default_port’     if (us->port == 0 && us->default_port == 0) {                            ^/usr/local/nginx-upstream-fair-master/ngx_http_upstream_fair_module.c:553:51: error: ‘ngx_http_upstream_srv_conf_t’ has no member named ‘default_port’     u.port = (in_port_t) (us->port ? us->port : us->default_port);
```

```nginx
# 解决，执行如下命令即可：# Linux: sed -i 's/default_port/no_port/g' /usr/local/nginx-upstream-fair-master/ngx_http_upstream_fair_module.c# macOs: sed -i '' 's/default_port/no_port/g' /file/ngx_http_upstream_fair_module.c
```

_参考：[https://blog.csdn.net/weixin\_46152207/article/details/121786162](https://blog.csdn.net/weixin_46152207/article/details/121786162 "https://blog.csdn.net/weixin_46152207/article/details/121786162")_

### 2.6 第三方策略：url\_hash【按访问url的hash结果来分配请求】

此方式需要单独额外的配置，不能直接在 upstream 模块添加关键字 url\_hash。

**大概的思路就是，下载补充包，通过 ./configure 命令，将模块添加到已安装的 nginx，重启下服务就可以使用关键字 fair 了。**

1）先下载第三方包：[https://github.com/evanmiller/nginx\_upstream\_hash](https://github.com/evanmiller/nginx_upstream_hash "https://github.com/evanmiller/nginx_upstream_hash")

需要将此文件夹放到 linux 系统中，本示例复制到路径：/usr/local/nginx\_upstream\_hash-master。

2）进入 nginx 源文件所在的文件夹（即 nginx 安装包解压后的文件夹），本示例的是：/usr/local/nginx-1.20.0。

_注意：如果源文件找不到了，需要重新下载一个相同版本安装包使用。_

```nginx
# 下载 nginx 包并解压wget http://nginx.org/download/nginx-1.20.0.tar.gztar -zxf nginx-1.20.1.tar.gz && cd nginx-1.20.0/# 其他一些可能用到的依赖包（按需安装）yum -y install gcc gcc-c++ openssl openssl-devel zlib zlib-devel pcre pcre-devel make cmake gperftools perl-devel  gd-devel libxml2 libxml2-dev libxslt-devel  redhat-rpm-config.noarch
```

3）执行命令添加模块：

```nginx
# 手动添加模块[root@www nginx-1.20.0]# ./configure --add-module=/usr/local/nginx_upstream_hash-master# 重新进行编译[root@www nginx-1.20.0]# make# 添加成功，查看配置，注意 configure arguments 配置：[root@www nginx-1.20.0]# nginx -Vnginx version: nginx/1.20.0built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) configure arguments: --add-module=/usr/local/nginx_upstream_hash-master
```

4）然后先停止 nginx 服务，再将 nginx 进行复制：

```nginx
# 先停止 nginx 服务[root@www sbin]# ./nginx -s stop# 复制 nginx[root@www nginx-1.20.0]# cp -rp objs/nginx /usr/local/nginx/sbin/nginxcp: overwrite ‘/usr/local/nginx/sbin/nginx’? yes[root@www nginx-1.20.0]#
```

最后，修改 nginx.conf 配置文件，开启服务测试一下。

**配置示例：**

```nginx
# 反向代理配置upstream server_shanky {    hash $request_uri; # 相同的 url 定向到同一后端服务器    # hash $remote_addr; # 也可根据客户端 IP 映射    # hash $args; # 也可根据客户端携带的参数进行映射    server www.shanky.cn:5001;    server www.shanky.cn:5021;    server www.shanky.cn:5031;}
```

配置完成后就可以实现，相同的 url 请求到同一个服务。

另外，当某一个服务异常停止后，原来通过 hash 对应到此服务的请求，会自动对应到其他服务，并不会造成请求失败。

### 2.7 第三方策略：consistent\_hash【采用一致性哈希算法结果来分配请求】

此方式需要单独额外的配置，不能直接在 upstream 模块添加关键字 consistent\_hash。

**大概的思路就是，下载补充包，通过 ./configure 命令，将模块添加到已安装的 nginx，重启下服务就可以使用关键字 consistent\_hash了。**

1）先下载第三方包：[https://github.com/replay/ngx\_http\_consistent\_hash](https://github.com/replay/ngx_http_consistent_hash "https://github.com/replay/ngx_http_consistent_hash")

文件很简单，如下图：


需要将此文件夹放到 linux 系统中，**本示例复制到路径：/usr/local/ngx\_http\_consistent\_hash-master。**

2）进入 nginx 源文件所在的文件夹（即 nginx 安装包解压后的文件夹），本示例的是：/usr/local/nginx-1.20.0。

_注意：如果源文件找不到了，需要重新下载一个相同版本安装包使用。_

```nginx
# 下载 nginx 包并解压wget http://nginx.org/download/nginx-1.20.0.tar.gztar -zxf nginx-1.20.1.tar.gz && cd nginx-1.20.0/# 其他一些可能用到的依赖包（按需安装）yum -y install gcc gcc-c++ openssl openssl-devel zlib zlib-devel pcre pcre-devel make cmake gperftools perl-devel  gd-devel libxml2 libxml2-dev libxslt-devel  redhat-rpm-config.noarch
```

3）执行命令添加模块：

```nginx
# 手动添加模块./configure --add-module=/usr/local/ngx_http_consistent_hash-master# 重新进行编译[root@www nginx-1.20.0]# make# 添加成功，查看配置，注意 configure arguments 配置：[root@www nginx-1.20.0]# nginx -Vnginx version: nginx/1.20.0built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) configure arguments: --add-module=/usr/local/ngx_http_consistent_hash-master
```

4）然后先停止 nginx 服务，再将 nginx 进行复制：

```nginx
# 先停止 nginx 服务[root@www sbin]# ./nginx -s stop# 复制 nginx[root@www nginx-1.20.0]# cp -rp objs/nginx /usr/local/nginx/sbin/nginxcp: overwrite ‘/usr/local/nginx/sbin/nginx’? yes[root@www nginx-1.20.0]#
```

最后，修改 nginx.conf 配置文件，开启服务测试一下。

**配置示例：**

```nginx
# 反向代理配置upstream server_shanky {    consistent_hash $request_uri; # 一致性哈希算法    # consistent_hash $remote_addr; # 也可根据客户端 IP 映射    # consistent_hash $args; # 也可根据客户端携带的参数进行映射    server www.shanky.cn:5001;    server www.shanky.cn:5021;    server www.shanky.cn:5031;}
```
