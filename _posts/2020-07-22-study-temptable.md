---
layout: post
title: mysql temp table로 쿼리 편하게 하기
date: 2020-07-22 18:19
category: Study
author: ["Jongpil Won"]
tags: [Study]
cover-image: hipster.jpg
summary: mysql temporary table을 이용하는 방법
---

# FYI - mysql temp table

### Temporary Table

- in-memory table을 만들어서, query의 join등을 쉽게 할 수 있도록 한다. (constant를 테이블화 할 수 있다.)
- join을 여러번 여러 테이블이랑 걸고 싶은데, where에 들어갈 사용자 리스트가 아주 많으면, temp table을 만들고, insert해서 join하면 편하다. (select ... union all ... 보다 짧다.)
- temp table이 크면 mysql 전체의 메모리 사용량을 줄여 성능에 이슈를 일으킬 수 있다.
- 권힌이 필요하다. `GRANT CREATE TEMPORARY TABLES ON ... TO ...`

### Temp Table 생성

```sql
create temporary table temp_xxx_xxx (
  column_1 definition,
  ...
);
```

### Temp Table에 자료 추가

```sql
insert into temp_xxx_xxx
values (...), (...), (...), ...;
```


