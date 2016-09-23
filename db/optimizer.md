### MySQL optimizer
#### Explain SQL plan
Explain SQL,
| PREDICATES| INDEX |
| ------ | ------ |
| ... where id = 1       |  const |
| ..  where id in (1, 2) |  range |
| ... where id > 1       | range  |

#### Batch Insert
1. SQL modify
Use 
> INSERT t() VALUES(),(),()
Instead of:
> INSERT t() VALUES();
> INSERT t() VALUES();

2. MySQL URL config
use:
> rewriteBatchedStatements=true&useCompress=true

[SQL performance, batch writing](http://java-persistence-performance.blogspot.kr/2013/05/batch-writing-and-dynamic-vs.html)

