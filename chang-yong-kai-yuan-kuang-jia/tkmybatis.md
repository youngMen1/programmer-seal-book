# 1.TkMybatis的常用方法介绍

## 1.1.Maven依赖

```
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>2.1.5</version>
</dependency>
```

## 1.2.使用要求

### 泛型\(实体类\)&lt;T&gt;的类型必须符合要求

_**实体类按照如下规则和数据库表进行转换,注解全部是JPA中的注解:**_

1. 表名默认使用类名,驼峰转下划线\(只对大写字母进行处理\),如UserInfo默认对应的表名为user\_info。
2. 表名可以使用@Table\(name = “tableName”\)进行指定,对不符合第一条默认规则的可以通过这种方式指定表名。
3. 字段默认和@Column一样,都会作为表字段,表字段默认为Java对象的Field名字驼峰转下划线形式。
4. 可以使用@Column\(name = “fieldName”\)指定不符合第3条规则的字段名。
5. 使用@Transient注解可以忽略字段,添加该注解的字段不会作为表字段使用。
6. 建议一定是有一个@Id注解作为主键的字段,可以有多个@Id注解的字段作为联合主键。

## 1.3.所有的mapper继承此类将具有以下通用方法

### 1.3.1.查询方法

###### BaseSelectMapper下的通用方法

| 方法名称 | 作用 |
| :--- | :--- |
| List&lt;T&gt; selectAll\(\); | 查询全部数据 |
| T selectByPrimaryKey\(Object key\); | 通过主键查询 |
| T selectOne\(T record\); | 通过实体查询单个数据 |
| List&lt;T&gt; select\(T record\); | 通过实体查询多个数据 |
| int selectCount\(T record\); | 通过实体查询实体数量 |
| boolean existsWithPrimaryKey\(Object key\); | 通过主键查询此主键是否存在 |

###### SelectByIdsMapper下的通用方法

| 方法名称 | 作用 |
| :--- | :--- |
| List&lt;T&gt; selectByIds\(String var1\); | 通过多个主键查询数据 |

### 1.3.2.添加方法

###### BaseInsertMapper下的通用方法

| 方法名称 | 作用 |
| :--- | :--- |
| int insert\(T record\); | 全部添加 |
| int insertSelective\(T record\); | 选择性\(不为null\)的添加 |

###### MySqlMapper下的通用方法

| 方法名称 | 作用 |
| :--- | :--- |
| int insertList\(List&lt;T&gt; list\); | 批量插入 |
| int insertUseGeneratedKeys\(T record\); | 如果主键为自增可使用此方法获取添加成功的主键 |

#### 1.3.3.修改方法

###### BaseUpdateMapper下的通用方法

| 方法名称 | 作用 |
| :--- | :--- |
| int updateByPrimaryKey\(T record\); | 按照实体进行修改 |
| int updateByPrimaryKeySelective\(T record\); | 按照实体进行有选择\(不为null\)的修改 |

#### 1.3.4.删除方法



