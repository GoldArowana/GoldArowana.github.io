---
title: 权限管理设计模式
date: 2021-07-09 13:03:33
tags:
    - authority
    - design patterns
categories:
    - design patterns
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co74.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co74.jpg
---

```flow
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes
or No?|approved:>http://www.google.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something...|request

st->op1(right)->cond
cond(yes, right)->c2
cond(no)->sub1(left)->op1
c2(yes)->io->e
c2(no)->op2->e
```


{% plantuml %}
Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response

Alice -> Bob: Another atuhentication Request
Alice <-- Bob: Another authentication Response
{% endplantuml %}