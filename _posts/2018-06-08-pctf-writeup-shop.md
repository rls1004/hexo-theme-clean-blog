---
layout: post
title: "[Plaid CTF] shop"
image: /img/hello_world.jpeg
---

## 요약
`GOT Overwrite`

## 프로그램 분석
4개의 명령어를 사용할 수 있다.

```
[a] allocate item (0x400A5B)
[l] list item (0x400D0E)
[c] check out and buy (0x400B78)
[n] shop name (0x400D6A)
```
명령어 목록:

| cmd | desc | addr |
| :------ |:--- | :--- |
| a | allocate item | 0x400A5B |
| l | list item | 0x400D0E |
| c | check out and buy | 0x400B78 |
| n | shop name | 0x400D6A |


#### [a] allocate item (0x400A5B)
- **item 구조**

item size : 0x130
item+0 ~ +7 : &head (\*0x6021F0)
item+8 ~ +11 : random bytes (0-9,a-f)
item+12 ~ +43 : name
item+44 ~ +299 : description
item+300 ~ +304 : price
