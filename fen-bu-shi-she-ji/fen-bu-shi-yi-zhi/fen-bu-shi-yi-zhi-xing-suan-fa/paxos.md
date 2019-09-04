# 1 Paxos算法

## 1.1 基本定义

算法中的参与者主要分为三个角色，同时每个参与者又可兼领多个角色:

⑴proposer 提出提案，提案信息包括提案编号和提议的value;

⑵acceptor 收到提案后可以接受\(accept\)提案;

⑶learner 只能"学习"被批准的提案;

算法保重一致性的基本语义:

⑴决议\(value\)只有在被proposers提出后才能被批准\(未经批准的决议称为"提案\(proposal\)"\);

⑵在一次Paxos算法的执行实例中，只批准\(chosen\)一个value;

⑶learners只能获得被批准\(chosen\)的value;

有上面的三个语义可演化为四个约束:

⑴P1:一个acceptor必须接受\(accept\)第一次收到的提案;

⑵P2a:一旦一个具有value v的提案被批准\(chosen\)，那么之后任何acceptor 再次接受\(accept\)的提案必须具有value v;

⑶P2b:一旦一个具有value v的提案被批准\(chosen\)，那么以后任何 proposer 提出的提案必须具有value v;

⑷P2c:如果一个编号为n的提案具有value v，那么存在一个多数派，要么他们中所有人都没有接受\(accept\)编号小于n的任何提案，要么他们已经接受\(accpet\)的所有编号小于n的提案中编号最大的那个提案具有value v;

## 1.2 基本算法\(basic paxos\)

算法\(决议的提出与批准\)主要分为两个阶段:

1. prepare阶段： 

\(1\). 当Porposer希望提出方案V1，首先发出prepare请求至大多数Acceptor。Prepare请求内容为序列号&lt;SN1&gt;;

\(2\). 当Acceptor接收到prepare请求&lt;SN1&gt;时，检查自身上次回复过的prepare请求&lt;SN2&gt;

a\). 如果SN2&gt;SN1，则忽略此请求，直接结束本次批准过程;

b\). 否则检查上次批准的accept请求&lt;SNx，Vx&gt;，并且回复&lt;SNx，Vx&gt;；如果之前没有进行过批准，则简单回复&lt;OK&gt;;

1. accept批准阶段： 

\(1a\). 经过一段时间，收到一些Acceptor回复，回复可分为以下几种:

a\). 回复数量满足多数派，并且所有的回复都是&lt;OK&gt;，则Porposer发出accept请求，请求内容为议案&lt;SN1，V1&gt;;

b\). 回复数量满足多数派，但有的回复为：&lt;SN2，V2&gt;，&lt;SN3，V3&gt;……则Porposer找到所有回复中超过半数的那个，假设为&lt;SNx，Vx&gt;，则发出accept请求，请求内容为议案&lt;SN1，Vx&gt;;

c\). 回复数量不满足多数派，Proposer尝试增加序列号为SN1+，转1继续执行;

\(1b\). 经过一段时间，收到一些Acceptor回复，回复可分为以下几种:

a\). 回复数量满足多数派，则确认V1被接受;

b\). 回复数量不满足多数派，V1未被接受，Proposer增加序列号为SN1+，转1继续执行;

\(2\). 在不违背自己向其他proposer的承诺的前提下，acceptor收到accept 请求后即接受并回复这个请求。

## 1.3 算法优化\(fast paxos\)

Paxos算法在出现竞争的情况下，其收敛速度很慢，甚至可能出现活锁的情况，例如当有三个及三个以上的proposer在发送prepare请求后，很难有一个proposer收到半数以上的回复而不断地执行第一阶段的协议。因此，为了避免竞争，加快收敛的速度，在算法中引入了一个Leader这个角色，在正常情况下同时应该最多只能有一个参与者扮演Leader角色，而其它的参与者则扮演Acceptor的角色，同时所有的人又都扮演Learner的角色。

在这种优化算法中，只有Leader可以提出议案，从而避免了竞争使得算法能够快速地收敛而趋于一致，此时的paxos算法在本质上就退变为两阶段提交协议。但在异常情况下，系统可能会出现多Leader的情况，但这并不会破坏算法对一致性的保证，此时多个Leader都可以提出自己的提案，优化的算法就退化成了原始的paxos算法。

一个Leader的工作流程主要有分为三个阶段：

\(1\).学习阶段 向其它的参与者学习自己不知道的数据\(决议\);

\(2\).同步阶段 让绝大多数参与者保持数据\(决议\)的一致性;

\(3\).服务阶段 为客户端服务，提议案;

![img](/static/image/20130902213600156.png)

## 1.3.1 学习阶段

当一个参与者成为了Leader之后，它应该需要知道绝大多数的paxos实例，因此就会马上启动一个主动学习的过程。假设当前的新Leader早就知道了1-134、138和139的paxos实例，那么它会执行135-137和大于139的paxos实例的第一阶段。如果只检测到135和140的paxos实例有确定的值，那它最后就会知道1-135以及138-140的paxos实例。

## 1.3.2 同步阶段

此时的Leader已经知道了1-135、138-140的paxos实例，那么它就会重新执行1-135的paxos实例，以保证绝大多数参与者在1-135的paxos实例上是保持一致的。至于139-140的paxos实例，它并不马上执行138-140的paxos实例，而是等到在服务阶段填充了136、137的paxos实例之后再执行。这里之所以要填充间隔，是为了避免以后的Leader总是要学习这些间隔中的paxos实例，而这些paxos实例又没有对应的确定值。

## 1.3.4 服务阶段

Leader将用户的请求转化为对应的paxos实例，当然，它可以并发的执行多个paxos实例，当这个Leader出现异常之后，就很有可能造成paxos实例出现间断。

## 1.3.5 问题

\(1\).Leader的选举原则

\(2\).Acceptor如何感知当前Leader的失败，客户如何知道当前的Leader

\(3\).当出现多Leader之后，如何kill掉多余的Leader

\(4\).如何动态的扩展Acceptor

# 2. Zookeeper

2.1 整体架构

在Zookeeper集群中，主要分为三者角色，而每一个节点同时只能扮演一种角色，这三种角色分别是：

\(1\). Leader 接受所有Follower的提案请求并统一协调发起提案的投票，负责与所有的Follower进行内部的数据交换\(同步\);

\(2\). Follower 直接为客户端服务并参与提案的投票，同时与Leader进行数据交换\(同步\);

\(3\). Observer 直接为客户端服务但并不参与提案的投票，同时也与Leader进行数据交换\(同步\);

20130902213637765.png

## 2.2 QuorumPeer的基本设计

20130902213801484.png

Zookeeper对于每个节点QuorumPeer的设计相当的灵活，QuorumPeer主要包括四个组件：客户端请求接收器\(ServerCnxnFactory\)、数据引擎\(ZKDatabase\)、选举器\(Election\)、核心功能组件\(Leader/Follower/Observer\)。其中：

\(1\). ServerCnxnFactory负责维护与客户端的连接\(接收客户端的请求并发送相应的响应\);

\(2\). ZKDatabase负责存储/加载/查找数据\(基于目录树结构的KV+操作日志+客户端Session\);

\(3\). Election负责选举集群的一个Leader节点;

\(4\). Leader/Follower/Observer一个QuorumPeer节点应该完成的核心职责;

## 2.3 QuorumPeer工作流程

20130902214130031.png

### 2.3.1 Leader职责

20130902214225640.png

Follower

确认

: 

等待所有的

Follower

连接注册，若在规定的时间内收到合法的

Follower

注册数量，则确认成功；否则，确认失败。

