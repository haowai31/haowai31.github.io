---
layout: post
title: 安全协议读书笔记（一）
category: 安全协议
date: 2015-06-11
---

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default" ></script>

选书为北京邮电大学出版的《安全协议》，作者是曹天杰、张永平、汪楚娇。这本书是上课时候的教材，由于很多内容囫囵吞枣，在此梳理一遍，更新时间不定。


<!-- more -->

## 攻击模型 #  
该攻击模型来源是Dolev和Yao在1983年发的论文。
一般默认攻击者具有以下能力：

1. 可以窃听所有经过网络的消息；
2. 可以阻止和截获所有经过网络的消息；
3. 可以存储所获得或自身创造的消息；
4. 可以根据存储的消息伪造消息，并发送该消息；
5. 可以作为合法的主题参与协议的运行；

而常见的协议攻击手段包括：窃听、篡改、重放、预重放、反射、拒绝服务、类型攻击、密码分析、证书操纵、协议交互等。
下面分别对反射、类型攻击、证书操纵三种攻击方式详解。

### 反射 #

首先，反射是重放的一个特例，该攻击存在的前提是协议能够并行运行。
现在在A和B之间存在一个认证协议，通过这个协议的验证可以使得A和B互相验证，其中\\(N\_A\\)和\\(N\_B\\)分别是A和B生成的一个随机数，K是A和B之间共享的密钥，A和B通过认证对方能揭开自己加密的随机数的方式认证，对方与自己同时拥有密钥K从而进行相互认证。协议过程如下：


1. A&rarr;B: {\\(N_A\\)}K
2. B&rarr;A: {\\(N\_B\\)}K, \\(N\_A\\)
3. A&rarr;B: \\(N\_B\\)

A收到消息2的时候就可以认为此消息来源为B，同样B在收到消息3时就可以认为消息来源为A，因为A和B共享密钥K。但是，该协议就可以被反射攻击，攻击过程如下，其中第2,3,6步是攻击者C重新和A发起的另外一个并行认证协议。以下表示协议执行顺序，但是该过程包含了两个并行执行的认证协议。

1. A&rarr;C: {\\(N\_A\\)}K
2. C&rarr;A: {\\(N\_A\\)}K
3. A&rarr;C: {\\(N\_A\\)'}K, \\(N\_A\\)
4. C&rarr;A: {\\(N\_A\\)'}K, \\(N\_A\\)
5. A&rarr;C: \\(N\_A\\)'
6. C&rarr;A: \\(N\_A\\)'

消息1是A发起的一个认证协议，在此，称其为认证协议M，在消息1之后，C收到了用K加密的信息，立即向A发起了一个新的认证协议（认证协议N），发送的就是消息1收到的使用K加密的随机数\\(N\_A\\)。对于认证协议N来说，A需要向C发送一个\\(N\_A\\)的明文，才能向C证明A知道密钥K。同时也会发送一个由K加密的随机数\\(N\_A\\)'。而对于认证协议M来说，C需要向A发送\\(N\_A\\)的明文才能向A证明C保有密钥K，之后C做了两次重放，就完成了认证协议M和N的认证，而在此过程中，所有的解密工作由A完成，C在不保有密钥K的情况下完成了认证。称C完成了一次反射攻击。

同时在完成攻击之后，C同时获得，获取任意明文被密钥K加密成密文的能力，这在别的攻击中非常有用，比如随时获取明文-密文对，有助于破解密钥K。

### 类型攻击 #

由于在协议当中，各方收到的消息都是二进制串组成的，用户没办法将该二进制串区分开，哪部分是加密的，哪部分是明文都不清楚。类型攻击就是利用这一点，让用户将一个消息错误的理解成为另外一个消息，比如可以将身份标识识别为一个密钥。下面是一个例子，该协议是Otway和Rees认证协议。A和B都长期存有与服务器S的密钥\\(K\_AS\\)和\\(K\_BS\\)。A和B的相互认证需要通过S进行，之后S会向A和B发送一个会话密钥\\(K\_AB\\)，具体流程如下，其中M和\\(N_A\\)是A选择的随机数，\\(N\_B\\)是B的随机数。

1. A&rarr;B: M, A, B, {\\(N\_A\\), M, A, B}\\(K\_{AS}\\)
2. B&rarr;S: M, A, B, {\\(N\_A\\), M, A, B}\\(K\_{AS}\\), {\\(N\_A\\), M, A, B}\\(K\_{BS}\\)
3. S&rarr;B: M, {\\(N\_A\\), \\(K\_{AB}\\)}\\(K\_{AS}\\), {\\(N\_A\\), M, A, B}\\(K\_{BS}\\)
4. B&rarr;A: M, {\\(N\_A\\), \\(K\_{AB}\\)}\\(K\_{AS}\\)

可以看到，消息1和消息4的格式比较相似，这就是进行类型攻击的点。而类型攻击需要额外进行一些假设，就是想要替换的类型长度是一致的，比如这里如果想用M、A、B替换\\(K\_{AB}\\)，那么这两者长度要一致，这里假设一致。在这些假设之后，攻击就可以开始进行了。这里\\(C\_B\\)表示C假冒B进行攻击。

1. A&rarr;\\(C\_B\\): M, A, B, {\\(N\_A\\), M, A, B}\\(K\_{AS}\\)
2. 
3. 
4. \\(C\_B\\)&rarr;A: M, {\\(N\_A\\), M, A, B}\\(K\_{AS}\\)

可以看到，在消息4中，C直接把从A发送的消息1，部分重新发送给消息A，使得A误认为组合域M、A、B当作共享密钥\\(K\_{AB}\\)，而M、A、B的值都是明文，可以得到，所以在之后的会话中，C就可以继续假冒B与A进行通信了。

### 证书操纵 #
数字证书可以担保某实体是公钥的拥有者。但是如果没有验证声明拥有密钥对拥有密钥权限的实体时，就会存在潜在攻击，使得攻击者具有能力获取合法的公钥证书，即使攻击者并不保有对应的私钥。
比如，A和B分别拥有公钥\\(g^a\\)和\\(g^b\\)，对应的私钥分别是a和b。A和B分别拥有证书Cert(A)和证书Cert(B)。证书中有公钥的副本。该协议是用来进行密钥协商，这里x和y是A和B选择的随机数。该协议描述如下：

1. A&rarr;B: \\(g^x\\), Cert(A)
2. B&rarr;A: \\(g^y\\), Cert(B)

之后双方计算共享密钥\\(K\_{AB}=g^{ay+bx}\\)。A和B都使用x和y进行计算。攻击者声明自己拥有公钥\\(g^{ac}\\)，并拥有证书Cert(C)，但是其实攻击者并没有私钥ac。在声明之后，C完成了与A和B的各一次认证，同时进行，攻击过程如下：

1. A&rarr;\\(C\_B\\): \\(g^x\\), Cert(A)
2. C&rarr;B: \\(g^x\\), Cert(C)
3. B&rarr;C: \\(g^y\\), Cert(B)
4. \\(C\_B\\)&rarr;A: \\(g^{yc}\\), Cert(B)

A计算密钥\\(K\_{AB}=(g^{yc})^a\*(g^b)^x=g^{acy+bx}\\)，B将计算密钥\\(K\_{AB}=(g^{ac})^y\*(g^x)^b=g^{acy+bx}\\)。可以看到A和B计算出的密钥是一样的。但是在这里，对于A来说，A认为A和B（其实是C假冒的）保有密钥\\(K\_AB\\)，而对于B来说，B认为B和C保有这个密钥。可以看到存在很多问题。

