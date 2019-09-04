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



