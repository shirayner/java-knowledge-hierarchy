





如，将用户表中所有用户的邮箱后缀换成新的后缀

```sql
update security_user t
set  t.email=replace(t.email,'@qq.com','@163.com')
where t.id in (select a.id from (select u.id from security_user u where u.email like '%@qq.com') a)
```

