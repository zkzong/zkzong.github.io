---
title: POI读取excel数字列自动带.0的解决办法
date: 2019-04-27
categories: POI
---

最近做项目需要用POI读取excel的数据并处理，excel中有一列的整数，读取后值自动带`.0`。

解决办法有两种：
1. 设置CellType
2. 使用DataFormatter格式化

## 使用for循环遍历
```java
for (int rowNum = 1; rowNum <= lastRowNum; rowNum++) {
    XSSFCell number = xssfRow.getCell(6);
    // 第1种方式
    number.setCellType(CellType.STRING);
    // 第二种方式
    DataFormatter dataFormatter = new DataFormatter();
    String value = dataFormatter.formatCellValue(number);

}
```

## 使用foreach循环遍历
```java
for (Cell cell : row) {
    // 第1种方式
    cell.setCellType(CellType.STRING);
    // 第二种方式
    DataFormatter dataFormatter = new DataFormatter();
    String value = dataFormatter.formatCellValue(cell);
}
```