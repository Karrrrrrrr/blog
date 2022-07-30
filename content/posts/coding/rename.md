---
title: "批量重命名"

date: "2022-07-30T17:11:17+08:00" 

categories: ["代码"]

tags: ["Python"]

---

```python
from os import system, popen

originPostFix = 'js'
toPostFix = 'ts'
ans = popen(f'find . -name "*.{originPostFix}" ')
for i in ans.read().split('\n'):
    if not i:
        continue
    system(f'mv {i} {i.replace(originPostFix, toPostFix)}')
```
