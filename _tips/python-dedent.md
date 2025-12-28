---
title: "python text dedent: from textwrap import dedent"
date: 2025-03-03
tags: [python]
---

```python
from textwrap import dedent

def test():
    # end first line with \ to avoid the empty line!
    s = '''\
    hello
      world
    '''
    print(repr(s))          # prints '    hello\n      world\n    '
    print(repr(dedent(s)))  # prints 'hello\n  world\n'
```
