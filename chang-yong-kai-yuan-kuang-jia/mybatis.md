# 1.Mybatis知识总结
## 1.1.Mybatis中<![CDATA[]]>的作用
在使用mybatis 时我们sql是写在xml 映射文件中，如果写的sql中有一些特殊的字符的话，在解析xml文件的时候会被转义，但我们不希望他被转义，所以我们要使用<![CDATA[ ]]>来解决。




