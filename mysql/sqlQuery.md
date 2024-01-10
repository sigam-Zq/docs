# sql 两个相同in查询



```sql
SELECT * FROM table_name WHERE id IN (1,2) AND id IN (2,3);
```

这条语句会查询出 id 为 2 的记录

两者 为且的关系 