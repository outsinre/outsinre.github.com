---
layout: post
title: Big Data Notes
---
接触big data时间不长，但是随着现在big data的兴起, 越来越多的人投入这个领域. 加上以前是做分布式系统开发的, 所以很自然地就进入了领域，一路弄下来，庆幸的是统计没白学, 这个领域我的感觉，就是cs的应用统计学，这块占了80％,剩下20％是分布式算法.
---

information retrieval 没啥高深的算法，基本数据结构就是inverted list, skip list。然后加上几个matching model，用的最多的估计还是vector space或者OKAPI 25。
machine learning 实践中真正好使的都是最基本的算法吧。比较鬼扯的那些个灌水算法，加上一堆乱七
八糟regularizer或者推convergence bound的，估计也没人care，因为实际上一碰上真
实数据全不work。要么就是仅仅在小规模数据上work，碰上大数据就要算一光年或者要
1TB内存...呵呵。

---
本来我是带着娱乐的态度来回帖的，但是既然碰到了大牛，请educate我。

1. 请告诉我任意一个数据结构，比inverted list 更重要，并且广泛地应用到了实际的text retrieval system中。
2. 请告诉我任意一个document retrieval model，比vector space model 或者 Okapi BM25, Statistically significantly better for general purpose document retrieval. Either implemented in Lucene or Lemur.
3. 请告诉我任意一个clustering algorithm，other than Kmeans，will be your safe first choice of clustering when you see some arbitrary data.
4. 对于Classification，Old Stuff Like KNN works well in many cases. Kernel algorithms are good publishing baselines, 但它们NxN 的需求极大得限制了它在大规模数据上得使用。Other algorithms like MinHash, LSH, KD-trees etc are all old. 

我的论点是，工业界真正使用的算法，没有那么多fancy的东西，因为确实大多数recent publish的work都不怎么work。都是tune parameters和选择性得测试data set搞出来灌水的。一旦你拿出那些算法在大规模真实数据上一跑，大部分都不怎么work。或者tune了N久比传统算法好不了多少，还不稳定。举例来说一个work的，page rank algorithm，这还是实现在真实系统里的。你要是实现过你就知道，比起kleinberg的HITS algorithm没有什么优势，但是Google实现的好，关键是加了很多有用的不被学术界所齿的heuristics，所以效果不错。

如果你确实认为近年的research极大得促进了科技得进步，改善了人类的生活，请告诉我近三年有什么publish在NIPS/ICML/WWW/KDD/COLT上的work被大规模的应用到了实际系统中，I am glad to know。我去学习。btw，deep learning去年NIPS很火，技术被google买了，那东西是彻底的刁丝翻身，NN这种没有理论得东西被statistical ML领域的人鄙视多少年了。Again，The true fact is我很菜。 我的的知识很落伍。很久没跟进最新的paper了。你要是能educate我，是个好事儿，我正好去学习。偷偷implement
一下这些牛逼算法赚个大的。

After Ph.D., you may make significant contribution to the area, you may not.
Most likely not. But you will gain the ability to tell whether something is
really working or it is just "claimed working". Working algorithms are usually very very simple. 忽悠algorithms are usually intentionally made complex and not working. 我觉得如果连这个都没练出来，那几百篇paper是白读了。

What's the shortest lie in computer science? "It works".

What's the shortest truth in computer science? "It sucks".

没有任何冒犯做research的人的意思，我也干这个，我就是想说，虽然不时会有一些比较牛逼的算法出现，（比如像SVM，就是work）。但残酷的现实就是，绝大部分的research work都没有什么significant contribution，除了发paper没啥用。这个估计读了phd的都有感受。所以灌完水拿了个phd. ，要去工业界，不用认为自己就牛逼得不得了，好像比没读phd的高几等。
---

coding:
- JOIN: nested join, hash join, sort-merge join
- Number: Fibonacci, prime，随机取文件某一行
- String: strstr, wordcount
- Tree: height, lca, balance tree
- Heap: 查找最大的k个数
- DP: 最大连续子串和
- array: find a key in rotated array, 去除重复字符
- linkedlist: 是否有环，插入结点，删除重复结点　
- 递归回溯：变化很多，这方面需要大量练习

知识性：
多线程，mutex/semaphore
java GC
C++ virtual, smart pointer
regex使用
数据库：知道btree, 索引
search engine: 倒排表,拉链，稀疏索引，空间向量模型，tf*idf, 
large scale data: hash, consistent hash, bloom filter, bitmap, 外排序，
partition
分布式：CAP理论，gossip，Paxos, GFS设计思想
network: socket, tcp3次握手, asyschnoized io, epoll, select, 惊群

设计型：
queue/stack实现
LRU
trie　tree
设计游戏
四则运算求值

我感觉把我上面说的练熟，还是很大可能性遇到的，虽然不是很全面，但我觉得不应该
把太多时间花在难题上，充实知识体系，符合职位要求更重要。
---
请问有没有python实现的数据结构和算法课？

入门算法推荐berkeley的 shewchuck 的 数据结构 还有一个大作业 是一个棋牌游戏的人工智能 那个课是semester的 用java的

顺便说一下 我个人感觉python实际使用起来兼容性很差 即使都是2.x 比如我使用NLTK的经历就很痛苦 每一个package弄下来都要debug 不能直接使用 逻辑bug没有 都是不兼容的bug 有些网上也找不到答案 只能看进去该函数 比如生成wordnet的网络图 我就是调用了另一个函数bird书里边的例子我都run不了而且python非常慢 我用的还是pyDEV 经常不知怎么的就死机了  

java大部分好技术都是java的 比如hadoop lucene weka 等等 

我觉得python发展很混乱 而且现在都又去学swift和node了 

python面试如果你不是行家 很快就可以看出来 语言本身的小技巧挺多的 所以用不好
python而用它面试属于丢人现眼了
---
老赵，你现在转战这里了？ 我觉得python vs java 的话，能够上 java 还是上 java,
python 在 hadoop world 是个怪胎. 所谓大家写的python  大部分都在后台被变成了
jython or cython. 由于隔着这1层纱，很多 performance 的问题根本没法debug.  
python 是入门用的, 几十行的function call 用python写还行。几百行，上千行的lib
还是得用jvm 写才行。
---
big data有两套生态啊，python的scipy和java的hadoop这些我用python就不用hadoop了，直接上scipy和numpy建模容易，虽然运行很慢，但是写起来来快，不适合生产，但是适合建模用hadoop我就不用python，直接上java我们是用python来快速modelling，然后对sample做test没问题之后，再由我等java编程师转译成java代码，测试无问题后下放生产基本上对python的使用是用完就扔掉，下次要用再写，基本上都不重用java则是大量重用代码，以造可复用的轮子为主要目的. scala用来写一些类库挺好，基本上除了java以外，其他所有语言都不宜写太多，都不适合搞软件工程其实我java代码也写得不多，一般超过200行就分类了
---
千万不要被任何FP欺骗上当，function programming从来不是，现在不是，将来也不是编程的主流，相信我
这一句就行了。老老实实用传统语言，有你吃香的喝辣的，FP就是没事找事。作为开发
者，记得一句话: We are coming here to solve problem, not to create problem。
---
我发表一下看法：
过去二十年CS深受OO和互联网的影响。所有的数据和业务逻辑都被封装在大大小小的模
块里面，这样保证了能够传输，移植和复用等问题。

但是被小心翼翼封装在json或者packet里面的数据已经没法流动起来了。rdbms虽然能
够处理transaction但是对于高维稀疏而且schema多变的数据也无能为力，以致于现在
最靠谱的数据共享方式还是文件或者文件的变种。

所谓的大数据工具，是一种海量拆包的工具，只不过是在反过来做过去20年各种无谓的
encapsulation。可以认为互联网每天成千上百个pb的数据里面，真正有价值的部分只
是几十个tb，而其中能够分析也不过这当中的一个百分比。无论是tableau也好还是
gnip也好还是sumologic也好，做的都是这些器，这些东西十几年前都做过了，只不过
现在从pc软件变成了web service。
---
靠谱

不过oo跟互联网还不是一个时代，互联网更靠后一些
oop很早就显现出了替代其他各种paradigms的架势
随之而来的是软件工程这个学科的兴起
然后逐步替换并淘汰掉c为代表的硬件/命令式编程
开始剥离出抽象的逻辑代码而非命令代码
最早做出垮平台的是fortran，字节码那些都是fortran先搞出来的
然后oop优化最早是smalltalk，以及后来的strongtalk搞出来的理论
再然后lars bak等人根据strongtalk的经验
address了sun的一个项目组用c++用疯了的问题
这就是oak以及后来的java还有官方jvm hotspot的第一版
然后就是java瞄准了网络时代，sun提出了the network is the computer
java上各种socket等的编程也远比c什么容易很多，封装得更彻底
最早c/c++什么用corba，简直不是人用的
然后java在corba基础之上搞出了rmi
再后来是ejb，ejb就是分布式系统的一个典型应用
然后ejb太过于复杂，加上m$被一脚踢出了java阵营
所以迫不及待需要一个更高level的通信协议，这就是后来xml
然后基于xml演化出了uddi, wsdl&soap这个web service的第一代
但是web service还是太过于复杂，于是有个phd提出了restful构架
简化了web service，然后就有了网络上各种产品比如json的今天
同时server side以ejb为代表的j2ee大规模应用，但是ejb本身也太过于复杂
rod johnson和gavin king为代表的aussie建议简化，同时orm也被提出
建议对传统db做封装，因为sql太不统一了，而且关系型数据跟oo概念有很大出入
这就有了spring和hibernate，然后spring和hibernate横行，就有了分布式今天的基础
架构
再然后，google等web公司逐步兴起，又提出了很多新概念，比如nosql和map reduce这些
后来，yahoo根据google的各种概念，作出了java版的google系统的翻版
然后贡献给了apache并开源了，其他公司就都跑去抄yahoo的这个东西，不要钱比什么
都重要
这就是hadoop，hadoop在一定程度上拓宽了传统db的范畴
这几个基本上构成了今天分布式的基础架构
再然后，这一套完成之后，人们开始想办法针对这一套架构做优化
简单说就是如何引入脚本来简化某些领域的开发，就像以前sql对db一样
这就有了ruby以及jruby，js以及rhino和nashorn，python和jython
同时jvm自身也在摸索一些更为合理的编程方式，这就是scala，groovy还有clojure
再然后，也就是一年前，更多的专业脚本语言被提出，要搬到jvm上去
这就是renjin，也就是r在jvm上的impl，以及hadoop自身发展出的类似sql的ql
比如cassandra用的cql，同时java本身也在拓展jvm的性能
java引入了script engine，随着java版本的逐步完善，以后让jvm直接执行脚本
比如python, ruby, js,groovy这些，会变得更为方便和便捷
但是jvm毕竟还是java的一部分，不懂jvm还是不行
另外很多人还在用并行计算的思维来思考分布式计算，都是hpc那些，这个也不对
不懂分布式就很难理解分布式所带来的各种问题，hadoop等都在尝试着让分布式变得更
简单
cloud也在努力使分布式变得更为简单，但是要做到无脑就用的程度
还是太遥远了，因为各种东西都很不完善，至少现阶段，还是要会java才行
否则都是toy，各种兼容性的问题，不胜其烦，生产系统可没办法这样搞
多来几个生产bugs，编程师就要准备打包滚蛋了

不过这些都是empirical东西
真正的big data和分布式理论要超越这些具体的impl
理论上用什么都可以做出来，用汇编都行，但是实践是另外一回事
实际干活还是以堆轮子为首选，否则没办法维护
---
jvm也是c写的，最终什么都是c，但是c和汇编都太底层了
跟人的思维接不上，人毕竟是人，不可能完全用机器的思维方式去思考和书写语言
整个计算机系统就是层层封装的结果

并行计算跟分布式计算是两回事
并行计算很多时候对于单机更有意义，共享内存这些
分布式计算一定涉及网络连接，分布式计算不在乎甚至有意识地破坏某些nodes
以测试整个系统的健壮程度，比如chaos monkey，就是要让某些nodes fail掉
看看系统work不work，并行计算用得比较多的是hpc，而不是分布式系统
分布式系统因为nodes上各种乱七八糟的系统什么良莠不齐
所以找到一个统一的平台非常重要，否则每个node都要求定制软件，工作量太大
jvm是目前能找到的最好平台
其他语言要么效率比不过jvm，要么就是兼容性比不过java

hpc上的mpi这些到还真是用c比较多，物理系什么都很喜欢写pbsscript
然后提交hpc排队，执行后看结果，并行计算和分布式计算有一些共性和重叠
但是毕竟不是一个东西，不同的topics

从效率上说，效率提升不只比单线程的效率
是多线程，多进程的效率提升，能并行处理的部分越多，可以提升的空间就越大
要并行处理，就需要decoupling，割裂多个模块，使之可以并行
这个要看需求，不同需求决定了dependency的多寡，一般科学计算依赖较强
web的相互依赖较弱，所以一般这种都最先用在web上

还有就是效率本身，java和c的差异主要体现在内存的管理上
那效率也是由综合因素决定的，也不仅仅取决于内存操作的执行效率
还同样取决于网络，which是分布式计算一定要涉及的部分
网络的io比起内存，那是要慢太多了
一般来说操作效率cpu>>内存>>硬盘>>网络
这里面还有什么l1 cache，l2 cache这些就都不细说了，n年不搞这些了
但是网络的latency远远高过硬盘，内存耗时这些应该是个共识
工作时候，网络的io是要尽量减少的，所以就算你辛辛苦苦用c实现了
提升了内存效率，但是还是改变不了网络的延迟，该慢还是慢，那你用c做有意义么？
木桶原理，决定高度的是最短的一块，分布式最短也就是最慢的一般都是网络操作
但是分布式又离不开网络操作，否则就不是分布式了

而且割裂的平台会使得很多优化手段无法使用
一个统一的平台远比几个不同平台各搞各的要容易优化
python的包很多都是fortran, c++还有python自身写的
乱七八糟，不仅垮平台很难实现，还同时导致综合执行效率降低
c++的有些优化手段，fortran就用不了，反之亦然，因为毕竟不是一个语言
很多特性不一样，相比之下，java所有的包都是jvm上的
所有代码有一个工整的执行格式，那么优化手段就多了

jvm本身在lars bak手下制作的时候，lars bak注册了23项专利
大部分都是优化专利，google后来雇用了lars bak，搞了v8引擎
结果就被oracle告了，就因为lars bak的专利
lars bak从strongtalk时代开始搞oop的优化，老鸟中的老鸟，巨牛无比
其他没有办法过他手优化的，比如ruby和python，效率就偏低，就慢
所以现在都争着往jvm上搬，什么jruby, jython, js这些，都有jvm的版本
到了jvm，就能用上lars bak的东西了，就快不少
现在lars bak在搞dart

大多数人，如果没有经过一定的代码优化理论训练
就算能用c或者汇编写一个hadoop这么大的东西出来
其执行效率还是会低于jvm和hadoop，更不要说开发和维护的效率了

分布式是一个非常大的topic，能涵盖你所知道的全部
我不认为有谁能够一个人搞定全部，这么想的基本上都属于盲目自大的
所以搞分布式学会利用别人做好的轮子非常重要，否则事倍功半
有些东西根本不是一个人一天两天能写出来的，比如os, jvm, db这些
这些都是群策群力多年积累下来的东西，大多数人穷其一生
能做其中一个，并得到市场的认同
就牛得不得了了，更不要说上面各种类库，spring, hibernate, hadoop
这些要是有人能写一个出来，都不要写完整了，你能参与其中
你就已经很牛了，这些都是apache top level档次的projects
能做其中一个，做到创始人的话，应该就能在wikipedia上有一个term来描述你的生平
你这辈子其实不用打工了，到处演讲卖书就好了
甚至到一些大学里面混个什么荣誉博士，问题不大
【 在 gtrr35 (GTR-R35) 的大作中提到: 】
: 赵老师啊，问你个简单的问题。
: 平行计算就是为了效率。既然这么讲求效率，为什么还用java作为平台语言，搞个
: Hadoop？难道不是应该用汇编和C吗？比如用C去implement MPI不就挺好吗？
---
c把跨平台，gc这些常见问题都给搞定了的话，那就是java了
java本身就是c like的语言，跟c++,obj c什么一样，都是c家族
不过可能是唯一一个不把c作为key letter放进去的c like语言
跟python这种非c like语言不太一样
java最早设计出来就是把那些c/c++工程中常见的问题
给统一用一个类库或者环境给搞定，所以最早有说法说是java类库最全最多

分布式并不是适合数据传递较少，相反，是传递较多才用分布式
但是尽量减少网络的io，是一个常见的优化手段
但是再少，都比并行计算用得多呀，毕竟网络是分布式的一个主要特征
而并行计算不用网络也没啥，弄点共享内存什么的，hpc一样可以搞并行计算

小程序没问题，用什么写都有可能，但是一旦项目变大
再在跨平台gc这种问题上折腾的话，实在是吃不消，力不从心，太累
大多数时候都是拿个轮子，直接抄来用了，java哲学比较讨人喜欢
如非万不得已，不要重复造轮子
【 在 gtrr35 (GTR-R35) 的大作中提到: 】
: 收藏了！
: 我是把分布式计算和并行计算混为一谈，因为我用到的都是后者。
: 按照你的说法，其实分布式计算很适合节点间数据传递较少的任务。对于这些任务，网
: 速不一定是瓶颈。比如一个大程序，是同时cpu intensive和memory intensive的，但
: 是可以分割，并且每个分割出来的小任务，也都是cpu&memory intensive的，但是同时
: 节点间的数据传递有限。这样的任务，网络的latency就不是瓶颈了吧。而且用C也比
: java效率高吧？
: 跨平台的确是个问题。要不是有这个问题，肯定现在最流行的是C，并且比第二的语言
: 可能高出一个数量的使用度。问个外行话，能不能先在所有的unix和linux系统下先
把C
: 跨平台了？
---
[Problem Solving with Algorithms and Data Structures](http://interactivepython.org/runestone/static/pythonds/index.html).
[zhaoce](http://www.mitbbs.com/article_t1/DataSciences/6829_0_1.html).
[Algorithmic](http://www.mitbbs.com/article/JobHunting/32600683_0.html).