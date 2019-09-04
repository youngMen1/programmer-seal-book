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

## 1.3 算法优化\(fast paxos\)



