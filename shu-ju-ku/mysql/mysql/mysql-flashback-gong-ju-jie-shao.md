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

  * [Percona Data Recovery Tool for InnoDB](https://link.jianshu.com/?t=https%3A%2F%2Fwww.percona.com%2Fdocs%2Fwiki%2Finnodb-data-recovery-tool_start.html)--&gt;[code](https://link.jianshu.com/?t=https%3A%2F%2Flaunchpad.net%2Fpercona-data-recovery-tool-for-innodb)
  * [undrop-for-innodb](https://link.jianshu.com/?t=https%3A%2F%2Ftwindb.com%2Fundrop-tool-for-innodb%2F)--&gt;[code](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fchhabhaiya%2Fundrop-for-innodb)「2017-01-01 已经闭源并收费」

DDL Falshback 要求离线，比 DML Flashback 要求更多，生产环境数据可能覆盖导致恢复不全

因为 5.6 开始，有单独的 purge 线程，数据恢复可能性更低了



