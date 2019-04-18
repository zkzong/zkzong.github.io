```
// 查找所有记录
PageHelper.startPage(1, 0);

// 统计总数
PageHelper.startPage(1, -1);

// 查找第一页，每页5条
PageHelper.startPage(1, 5);
```

spring需要配置xml
spring boot不需要
