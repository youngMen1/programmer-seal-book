# 1.SQL优化

## 1.1.一些常见的 SQL 实践

#### 1.1.1.负向条件查询不能使用索引

```
select from order where status!=0 and status!=1
not in/not exists # 都不是好习惯
```

可以优化为`in`查询：

```
select from order where status in(2,3)
```

## 1.2.并非周知的 SQL 实践

## 1.3.小众但有用的 SQL 实践



