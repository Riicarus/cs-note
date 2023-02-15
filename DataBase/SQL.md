# SQL 相关

## 内置函数
### `COUNT()`
> `COUNT(*)`, `COUNT(1)`, `COUNT(COL_NAME)` 的区别?
`COUNT(*)` 和 `COUNT(1)` 功能相同, 都是统计表中所有数据的条数. 而 `COUNT(COL_NAME)` 不会统计表中的空数据.  
由此: `COUNT(*)` = `COUNT(1)` >= `COUNT(COL_NAME)`