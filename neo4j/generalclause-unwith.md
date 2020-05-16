# Neo4j Unwind解包子句

## 例

以下是示例Cypher查询，展开了一个列表。

```
UNWIND [a, b, c, d] AS x 
RETURN x 
```
