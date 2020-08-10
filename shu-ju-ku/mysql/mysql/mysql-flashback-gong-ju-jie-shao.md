# MySQL Flashback 工具介绍

* DML Flashback

  * 独立工具，通过伪装成slave拉取binlog来进行处理
    * [MyFlash](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2FMeituan-Dianping%2FMyFlash)
      「大众点点评」
    * [binlog2sql](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fdanfengcao%2Fbinlog2sql)
      「大众点评\(上海\)」
    * [mysqlbinlog\_flashback](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2F58daojia-dba%2Fmysqlbinlog_flashback)
      \(更倾向于阿里RDS\) 「58到家」
  * patch形式集成到官方工具mysqlbinlog
  * 简单脚本。先用mysqlbinlog解析出文本格式的binlog，再根据回滚原理用正则进行匹配并替换

* DDL Flashback



