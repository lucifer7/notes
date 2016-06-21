### MySQL optimizer
Explain SQL,
| PREDICATES| INDEX |
| ------ | ------ |
| ... where id = 1       |  const |
| ..  where id in (1, 2) |  range |
| ... where id > 1       | range  |

